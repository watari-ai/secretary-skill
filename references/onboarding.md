# Secretary Skill — Onboarding Guide

This guide walks through setting up the prerequisites for the secretary skill.

---

## 1. himalaya (Email CLI)

### Installation

```bash
# macOS (Homebrew)
brew install himalaya

# or via Cargo
cargo install himalaya
```

### Account Configuration

himalaya uses a config file at `~/.config/himalaya/config.toml`.

```toml
[accounts.<account-name>]
default = true
email = "your-email@example.com"
display-name = "Your Name"

backend.type = "imap"
backend.host = "imap.gmail.com"
backend.port = 993
backend.login = "your-email@example.com"
backend.auth.type = "password"
backend.auth.raw = "<app-password>"

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.gmail.com"
message.send.backend.port = 465
message.send.backend.login = "your-email@example.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.raw = "<app-password>"
```

### Getting a Google App Password

1. Go to [Google Account Security](https://myaccount.google.com/security)
2. Enable 2-Step Verification if not already enabled
3. Go to [App Passwords](https://myaccount.google.com/apppasswords)
4. Select "Mail" and your device, then generate
5. Use the 16-character password in the config above

### Verify

```bash
himalaya list --account <account-name>
```

You should see your inbox messages.

---

## 2. gcalcli (Google Calendar CLI)

### Installation

```bash
# macOS (Homebrew)
brew install gcalcli

# or via pip
pip install gcalcli
```

### OAuth Setup

gcalcli needs Google OAuth credentials:

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a project (or use existing)
3. Enable the **Google Calendar API**
4. Go to **Credentials** → **Create Credentials** → **OAuth 2.0 Client ID**
5. Application type: **Desktop app**
6. Download the JSON file

Set the credentials:

```bash
# Option A: Place the file
mkdir -p ~/.gcalcli
mv client_secret_*.json ~/.gcalcli/oauth

# Option B: Environment variable
export GCALCLI_OAUTH_CLIENT_ID="your-client-id"
export GCALCLI_OAUTH_CLIENT_SECRET="your-client-secret"
```

### First-Time Authentication

```bash
gcalcli agenda
```

This opens a browser for OAuth consent. Authorize access, and gcalcli stores the token locally.

### Verify

```bash
gcalcli agenda "today" "tomorrow"
```

You should see today's events.

---

## 3. Creating config.yml

Use this interactive flow to create the secretary skill config:

### Step-by-step Questions

1. **Email account name**
   > What is your himalaya account name? (as configured in himalaya's config.toml)

2. **Master email address**
   > What is your (the human's) email address? (used to prioritize emails from you)

3. **Agent email address** (optional)
   > Does the agent have its own email address? (leave blank if not)

4. **Email check interval**
   > How often should I check email? (default: 4 hours)

5. **Calendar ID**
   > Which Google Calendar should I use? (blank for default, or provide calendar ID)

6. **Reminder timing**
   > How many minutes before a meeting should I remind you? (default: 5)

7. **Notification channel**
   > Which Slack/Discord channel should I send notifications to? (channel ID or name)

8. **Morning report time**
   > When should I send the morning briefing? (default: 08:00)

9. **Evening report time**
   > When should I send the evening briefing? (default: 23:00)

10. **Timezone**
    > What timezone are you in? (default: Asia/Tokyo)

11. **Weather**
    > Should I include weather in briefings? (default: yes)
    > Which city? (default: Tokyo)

12. **GitHub integration** (optional)
    > Should I include GitHub activity? (default: no)
    > Which repos? (e.g., owner/repo1, owner/repo2)

13. **Calendar reminder timing**
    > 何分前にミーティングのリマインドが欲しいですか？（デフォルト: 5分前）

14. **Break reminder schedule**
    > 休憩の声掛けはどのタイミングが良いですか？
    > - 時刻リストで指定（デフォルト: 12:00, 15:00, 18:00）
    > - または間隔で指定（例: 3時間ごと）

15. **Night mode start time**
    > 何時以降を深夜モードにしますか？深夜モードでは「そろそろ休みませんか？」と声掛けします。（デフォルト: 0:00）

16. **Long work threshold**
    > 連続作業何時間で休憩を促しますか？（デフォルト: 2時間）

### After Completion

Save the answers as `config.yml` in the skill directory, then verify:

```bash
# Test email
himalaya list --account <account>

# Test calendar
gcalcli agenda

# All good — secretary is ready!
```
