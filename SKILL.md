---
name: secretary
description: AI secretary skill for email monitoring, calendar briefings, and meeting reminders. Uses himalaya (email) and gcalcli (calendar) as backends. Handles morning/evening reports, unread email digests, and pre-meeting notifications.
---

# Secretary

An AI agent skill for performing secretary duties: email triage, daily briefings, calendar management, and proactive reminders.

## Prerequisites

- **himalaya** — CLI email client ([github.com/pimalaya/himalaya](https://github.com/pimalaya/himalaya))
- **gcalcli** — Google Calendar CLI ([github.com/insanum/gcalcli](https://github.com/insanum/gcalcli))
- Both must be installed and authenticated before use. See [references/onboarding.md](references/onboarding.md) for setup.

## Quick Start

1. Copy `config.example.yml` to `config.yml` and fill in your values.
2. Verify tools are working:
   ```bash
   himalaya list --account <account>
   gcalcli agenda
   ```
3. You're ready. The sections below describe each duty.

---

## Email Digest

Check for unread emails, filter for importance, and send a summary notification.

### Procedure

1. **Fetch unread emails**
   ```bash
   himalaya list --account <config.email.account> --folder INBOX -s "unseen"
   ```

2. **Filter and prioritize**
   - Flag emails from `config.email.master_address` as high priority
   - Group by: needs-reply, FYI, newsletters/automated
   - Skip known noise (automated notifications, marketing)

3. **Format the digest**
   Use the template at [templates/email-digest.md](templates/email-digest.md).
   Include for each email:
   - Sender, subject, received time
   - 1-line summary of content
   - Priority tag (🔴 urgent / 🟡 normal / ⚪ low)

4. **Send notification**
   Post the formatted digest to `config.notifications.channel`.

### Cadence

Run every `config.email.check_interval_hours` hours. Can also be triggered on demand.

---

## Morning Briefing

Delivered at `config.notifications.morning_report_time` in `config.notifications.timezone`.

### Procedure

1. **Get today's calendar**
   ```bash
   gcalcli agenda "today" "tomorrow" --calendar <config.calendar.calendar_id> --details location --details description --tsv
   ```

2. **Weather (optional)**
   If `config.briefing.include_weather` is true, fetch weather for `config.briefing.weather_city`.

3. **GitHub activity (optional)**
   If `config.briefing.include_github` is true, check PRs/issues for repos in `config.briefing.github_repos`.

4. **Format the briefing**
   Use the template at [templates/morning-brief.md](templates/morning-brief.md).
   Include:
   - Date and greeting
   - Today's schedule (time, title, location)
   - Weather summary (if enabled)
   - Open PRs/issues (if enabled)
   - Action items / reminders

5. **Send to channel**
   Post to `config.notifications.channel`.

6. **Schedule meeting reminders**
   For each meeting today, create a reminder `config.calendar.reminder_minutes` minutes before start.
   Use cron jobs or heartbeat checks to deliver reminders like:
   > ⏰ Meeting in 5 min: "Weekly Standup" (Zoom link: ...)

---

## Evening Briefing

Delivered at `config.notifications.evening_report_time` in `config.notifications.timezone`.

### Procedure

1. **Review today's events**
   ```bash
   gcalcli agenda "today 00:00" "today 23:59" --calendar <config.calendar.calendar_id> --tsv
   ```

2. **Summarize the day**
   - Events that occurred
   - Emails received / replied count
   - Any pending action items

3. **Preview tomorrow**
   ```bash
   gcalcli agenda "tomorrow" "tomorrow +1d" --calendar <config.calendar.calendar_id> --tsv
   ```

4. **Format the briefing**
   Use the template at [templates/evening-brief.md](templates/evening-brief.md).

5. **Send to channel**
   Post to `config.notifications.channel`.

---

## Meeting Reminders

Proactive notifications before calendar events.

### How it works

- After the morning briefing, schedule reminders for each event.
- `config.calendar.reminder_minutes` before each meeting, send a notification with:
  - Event title and time
  - Location / video call link (extracted from description)
  - Attendees (if available)
- Use cron scheduling or heartbeat polling depending on your agent platform.

---

## Onboarding

For first-time setup, run an interactive onboarding flow:

1. Check if himalaya and gcalcli are installed
2. Guide through himalaya account configuration (IMAP/SMTP)
3. Guide through gcalcli OAuth authentication
4. Create `config.yml` by asking each field interactively
5. Verify connectivity with a test email fetch and calendar query

Full onboarding instructions: [references/onboarding.md](references/onboarding.md)

---

## Templates

- [templates/email-digest.md](templates/email-digest.md) — Email digest notification format
- [templates/morning-brief.md](templates/morning-brief.md) — Morning briefing format
- [templates/evening-brief.md](templates/evening-brief.md) — Evening briefing format

## Configuration Reference

See `config.example.yml` for all available options. Key sections:

| Section | Purpose |
|---------|---------|
| `email` | himalaya account, addresses, check interval |
| `calendar` | gcalcli calendar ID, reminder timing |
| `notifications` | Channel, report times, timezone |
| `briefing` | Optional weather/GitHub integration |
