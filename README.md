# Secretary Skill

An AI agent skill for performing secretary duties: email triage, daily briefings, calendar management, and proactive reminders.

Built for [OpenClaw](https://github.com/openclaw/openclaw) agents.

## Features

- 📧 **Email Digest** — Check unread emails, filter by importance, send summaries
- ☀️ **Morning Briefing** — Daily schedule, weather, and action items
- 🌙 **Evening Briefing** — Day review and tomorrow preview
- ⏰ **Meeting Reminders** — Proactive notifications before calendar events

## Prerequisites

| Tool | Purpose | Installation |
|------|---------|-------------|
| [himalaya](https://github.com/pimalaya/himalaya) | CLI email client (IMAP/SMTP) | `brew install himalaya` |
| [gcalcli](https://github.com/insanum/gcalcli) | Google Calendar CLI | `pip install gcalcli` |

Both tools must be installed and authenticated before use.

## Quick Start

1. **Clone this skill** into your agent's skills directory:
   ```bash
   git clone https://github.com/zyom45/secretary-skill.git skills/secretary
   ```

2. **Copy and configure** the config file:
   ```bash
   cp config.example.yml config.yml
   # Edit config.yml with your values
   ```

3. **Verify tools are working**:
   ```bash
   himalaya list --account <your-account>
   gcalcli agenda
   ```

4. **You're ready.** The skill provides procedures for email digests, briefings, and reminders.

## Configuration

See `config.example.yml` for all available options:

| Section | Purpose |
|---------|---------|
| `email` | himalaya account, addresses, check interval |
| `calendar` | gcalcli calendar ID, reminder timing |
| `notifications` | Channel, report times, timezone |
| `briefing` | Optional weather/GitHub integration |

## Skill Structure

```
secretary/
├── SKILL.md              # Full skill instructions
├── config.example.yml    # Configuration template
├── references/
│   └── onboarding.md     # First-time setup guide
└── templates/
    ├── email-digest.md   # Email notification format
    ├── morning-brief.md  # Morning briefing format
    └── evening-brief.md  # Evening briefing format
```

## Usage with OpenClaw

This skill is auto-discovered when placed in your agent's skills directory. The agent will read `SKILL.md` when handling secretary-related tasks.

For scheduled briefings, set up cron jobs:
- Morning briefing at configured time
- Email digest every N hours
- Meeting reminders based on calendar

## License

MIT

## Contributing

Issues and PRs welcome at [zyom45/secretary-skill](https://github.com/zyom45/secretary-skill).
