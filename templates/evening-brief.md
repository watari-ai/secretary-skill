## 🌙 Evening Report — {{date}}

### 📅 Today's Events

{{#if events}}
{{#each events}}
- {{time}} — {{title}} {{#if completed}}✅{{/if}}
{{/each}}
{{else}}
No events today.
{{/if}}

### 📬 Email Summary
- Received: {{emails_received}}
- Replied: {{emails_replied}}
- Pending: {{emails_pending}}

{{#if action_items}}
### 📌 Pending Action Items
{{#each action_items}}
- {{text}}
{{/each}}
{{/if}}

### 📅 Tomorrow's Preview

{{#if tomorrow_events}}
{{#each tomorrow_events}}
- {{time}} — {{title}}
{{/each}}
{{else}}
No events scheduled for tomorrow.
{{/if}}

---
Good night! 🌟
