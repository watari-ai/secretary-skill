## 📬 Email Digest

**Checked:** {{timestamp}}
**Unread:** {{unread_count}} messages

{{#if urgent_emails}}
### 🔴 Needs Reply

{{#each urgent_emails}}
- **{{sender}}** — {{subject}}
  {{summary}} _({{received}})_
{{/each}}
{{/if}}

{{#if normal_emails}}
### 🟡 FYI

{{#each normal_emails}}
- **{{sender}}** — {{subject}}
  {{summary}} _({{received}})_
{{/each}}
{{/if}}

{{#if low_emails}}
### ⚪ Low Priority

{{#each low_emails}}
- {{sender}} — {{subject}} _({{received}})_
{{/each}}
{{/if}}

{{#unless unread_count}}
✅ Inbox clear — no unread messages.
{{/unless}}
