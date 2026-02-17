# Secretary Skill

AI agent skill for secretary duties — email triage, daily briefings, calendar management, meeting reminders, and health/life support.

## Features

- **📧 Email Digest** — Periodic unread email check, filtering, prioritization, and summary
- **🌅 Morning Briefing** — Daily overview: calendar, weather, GitHub activity
- **🌙 Evening Briefing** — Day review + tomorrow's preview
- **⏰ Meeting Reminders** — Automatic pre-meeting notifications (configurable minutes before)
- **💚 Health & Life Support**
  - Break reminders at configurable times
  - Shopping list management
  - Long work session detection
  - Late night mode (gentle sleep nudges)

## Prerequisites

- **[himalaya](https://github.com/pimalaya/himalaya)** — CLI email client (installed & authenticated)
- **[gcalcli](https://github.com/insanum/gcalcli)** — Google Calendar CLI (installed & authenticated)

## Setup

1. Copy `config.example.yml` → `config.yml`
2. Fill in your values (email account, notification channel, timing preferences)
3. Verify tools work:
   ```bash
   himalaya list --account <your-account> --folder INBOX
   gcalcli agenda
   ```
4. Run onboarding: see `references/onboarding.md`

## Configuration

See `config.example.yml` for all options. Key settings:

| Key | Description | Default |
|-----|-------------|---------|
| `email.check_interval_hours` | Email check frequency | `4` |
| `calendar.reminder_minutes` | Minutes before meeting to remind | `5` |
| `notifications.morning_report_time` | Morning briefing time | `"08:00"` |
| `notifications.evening_report_time` | Evening briefing time | `"23:00"` |
| `health.break_reminder_times` | Break reminder schedule | `["12:00","15:00","18:00"]` |
| `health.night_mode_start` | Late night mode trigger | `"00:00"` |
| `health.long_work_threshold_hours` | Continuous work alert threshold | `2` |

## Templates

- `templates/email-digest.md` — Email digest format
- `templates/morning-brief.md` — Morning briefing format
- `templates/evening-brief.md` — Evening briefing format
- `templates/meeting-reminder.md` — Meeting reminder format
- `templates/break-reminder.md` — Break reminder format

## License

MIT
