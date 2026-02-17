## ☀️ Good Morning — {{date}}

{{#if weather}}
### 🌤 Weather in {{weather_city}}
{{weather_summary}} — High: {{high}}°C / Low: {{low}}°C
{{/if}}

### 📅 Today's Schedule

{{#if events}}
| Time | Event | Location |
|------|-------|----------|
{{#each events}}
| {{time}} | {{title}} | {{location}} |
{{/each}}
{{else}}
No events scheduled today. 🎉
{{/if}}

{{#if github_activity}}
### 🐙 GitHub Activity

{{#each github_repos}}
- **{{repo}}**: {{open_prs}} open PRs, {{open_issues}} open issues
{{/each}}
{{/if}}

{{#if reminders}}
### 📌 Reminders
{{#each reminders}}
- {{text}}
{{/each}}
{{/if}}

---
Have a productive day!
