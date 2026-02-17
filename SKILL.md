---
name: secretary
description: AI secretary skill for email monitoring, calendar briefings, meeting reminders, and health/life support. Uses himalaya (email) and gcalcli (calendar). Handles morning/evening reports, email digests, pre-meeting notifications, break reminders, and shopping list management.
---

# Secretary

AI agent skill for secretary duties: email triage, daily briefings, calendar management, meeting reminders, and health/life support.

## Prerequisites

- **himalaya** — CLI email client ([github.com/pimalaya/himalaya](https://github.com/pimalaya/himalaya))
- **gcalcli** — Google Calendar CLI ([github.com/insanum/gcalcli](https://github.com/insanum/gcalcli))
- Both must be installed and authenticated. See [references/onboarding.md](references/onboarding.md).

## Quick Start

1. Copy `config.example.yml` → `config.yml`, fill in values.
2. Verify: `himalaya list --account <account>` / `gcalcli agenda`
3. Ready.

---

## Email Digest

Check unread emails, filter, summarize.

### Procedure

1. **Fetch unread**
   ```bash
   himalaya list --account <config.email.account> --folder INBOX -s "unseen"
   ```

2. **Filter and prioritize**
   - `config.email.master_address` → high priority
   - Group: needs-reply / FYI / newsletters
   - Skip noise (automated, marketing)

3. **Format** using [templates/email-digest.md](templates/email-digest.md)

4. **Send** to `config.notifications.channel`

### Cadence
Every `config.email.check_interval_hours` hours, or on demand.

---

## Morning Briefing

At `config.notifications.morning_report_time`.

### Procedure

1. **Get today's calendar**
   ```bash
   gcalcli agenda "today" "tomorrow" --details location --details description --tsv
   ```

2. **Weather** (if `config.briefing.include_weather`): fetch for `config.briefing.weather_city`

3. **GitHub** (if `config.briefing.include_github`): check `config.briefing.github_repos`

4. **Format** using [templates/morning-brief.md](templates/morning-brief.md)

5. **Send** to `config.notifications.channel`

6. **Schedule meeting reminders** → see next section

---

## Evening Briefing

At `config.notifications.evening_report_time`.

### Procedure

1. **Review today**: `gcalcli agenda "today 00:00" "today 23:59" --tsv`
2. **Summarize**: events, emails, pending items
3. **Preview tomorrow**: `gcalcli agenda "tomorrow" "tomorrow +1d" --tsv`
4. **Format** using [templates/evening-brief.md](templates/evening-brief.md)
5. **Send** to `config.notifications.channel`

---

## Meeting Reminders

Proactive notifications `config.calendar.reminder_minutes` minutes before each event.

### Step 1: Get Today's Events (in Morning Briefing)

```bash
gcalcli agenda "today" "tomorrow" --tsv --details location --details description
```

**TSV output columns**: `start_date  start_time  end_date  end_time  title  location  description`

### Step 2: Parse Events and Skip All-Day

```python
# Pseudo-code for parsing
for line in tsv_output:
    start_date, start_time, end_date, end_time, title, location, desc = line.split('\t')
    # Skip all-day events (no start_time or start_time is empty)
    if not start_time or start_time.strip() == '':
        continue
    # Extract video link from description (Zoom/Meet/Teams)
    video_link = extract_url(desc, patterns=['zoom.us', 'meet.google.com', 'teams.microsoft.com'])
```

### Step 3: Create Cron Jobs for Each Event

For each event, calculate reminder time = start_time − `config.calendar.reminder_minutes`:

```bash
# Example: event at 14:00, reminder_minutes=5 → remind at 13:55
openclaw cron add \
  --schedule "55 13 * * *" \
  --task "Send meeting reminder to config.notifications.channel" \
  --systemEvent \
  --message "⏰ 5分後: **Weekly Standup** (14:00)\n📍 https://zoom.us/j/123456"
```

**Concrete example with multiple events**:
```bash
# 10:00 朝会 → 09:55 にリマインド
openclaw cron add --schedule "55 9 17 2 *" \
  --task "message(action=send, target=config.notifications.channel, message='⏰ 5分後に始まります: **朝会** (10:00)\n📍 会議室A')" \
  --systemEvent

# 14:00 1on1 → 13:55 にリマインド
openclaw cron add --schedule "55 13 17 2 *" \
  --task "message(action=send, target=config.notifications.channel, message='⏰ 5分後に始まります: **1on1** (14:00)\n🔗 https://meet.google.com/xxx')" \
  --systemEvent
```

### Step 4: Notification Format

Use [templates/meeting-reminder.md](templates/meeting-reminder.md). Include:
- Event title and start time
- Location or video call link
- Attendees (if available in description)

### Notes
- Cron jobs are date-specific (day + month) so they fire only today
- Clean up old cron jobs on next morning briefing
- If agent platform doesn't support `openclaw cron`, use heartbeat polling: check time every heartbeat, fire reminder if within `reminder_minutes` window

---

## Health & Life Support

Proactive wellness and daily life assistance. Config: `config.health.*`

### 1. Break Reminders

**When**: At times specified in `config.health.break_reminder_times` (configurable; default: `["12:00", "15:00", "18:00"]`)

**How to implement** (cron): Read `config.health.break_reminder_times` and create a cron job for each entry:
```bash
# For each time in config.health.break_reminder_times:
openclaw cron add --schedule "<min> <hour> * * *" \
  --task "Send break reminder to config.notifications.channel" \
  --systemEvent
```

**Or via heartbeat**: Check current time each heartbeat. If within ±10 min of a `config.health.break_reminder_times` entry and no reminder sent yet for that slot today, send one.

**Tone**: Natural, caring, not pushy. See [templates/break-reminder.md](templates/break-reminder.md) for examples.

Track sent reminders in `memory/heartbeat-state.json`:
```json
{
  "breakRemindersSent": {
    "2026-02-17": ["12:00", "15:00"]
  }
}
```

### 2. Shopping List Management

**File**: `config.health.shopping_list_path` (default: `data/shopping-list.txt`)

**Format** (one item per line, `[x]` = done):
```
[ ] 牛乳
[ ] パン
[x] 卵
```

**Manage with**: `tools/shopping.sh add/done/clear`

**Evening check** (~17:00-18:00, during break reminder or heartbeat):
1. Read `shopping-list.txt`
2. If unchecked items exist, ask: 「今日買い物に行く予定はありますか？残りリスト: 牛乳、パン」
3. If empty/all done, skip

### 3. Long Work Session Detection

**Trigger**: `tools/work.sh status` shows >`config.health.long_work_threshold_hours` hours continuous work, or last Slack message from master was >2 hours ago without break.

**Action**: Gentle nudge — 「もう{config.health.long_work_threshold_hours}時間以上作業してますね。少し休憩しませんか？☕」

**Implementation**: Check in heartbeat. Compare current time with `tools/work.sh status` output or last activity timestamp.

### 4. Late Night Mode

**When**: After `config.health.night_mode_start` (configurable; default: `"00:00"`)

**Action**: If master is active (sending messages), gently suggest sleep:
- 「もう`config.health.night_mode_start`を過ぎてますよ。そろそろ休みませんか？😴」
- Repeat at most once per hour
- Don't nag — one reminder is enough

**Implementation**: In heartbeat, check if current time > `night_mode_start` and master sent a message in last 30 min. Track in `heartbeat-state.json`:
```json
{
  "lastNightReminder": "2026-02-17T00:30:00"
}
```

---

## Onboarding

First-time setup: see [references/onboarding.md](references/onboarding.md).

---

## Templates

- [templates/email-digest.md](templates/email-digest.md) — Email digest
- [templates/morning-brief.md](templates/morning-brief.md) — Morning briefing
- [templates/evening-brief.md](templates/evening-brief.md) — Evening briefing
- [templates/meeting-reminder.md](templates/meeting-reminder.md) — Meeting reminder
- [templates/break-reminder.md](templates/break-reminder.md) — Break reminder

## Configuration Reference

See `config.example.yml`. Key sections:

| Key | Purpose | Default |
|-----|---------|---------|
| `email.check_interval_hours` | メールチェック間隔 | `4` |
| `calendar.reminder_minutes` | ミーティング前リマインド | `5` |
| `notifications.morning_report_time` | 朝ブリーフィング時刻 | `"08:00"` |
| `notifications.evening_report_time` | 夕ブリーフィング時刻 | `"23:00"` |
| `health.break_reminder_times` | 休憩声掛け時刻リスト | `["12:00","15:00","18:00"]` |
| `health.night_mode_start` | 深夜モード開始時刻 | `"00:00"` |
| `health.long_work_threshold_hours` | 連続作業警告閾値 | `2` |

All timing/frequency parameters are configurable. Set during onboarding — see [references/onboarding.md](references/onboarding.md).
