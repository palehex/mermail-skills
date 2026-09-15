# Inbound lead agent tool contracts

This persona composes existing capabilities. It adds no CRM, lead-storage, scoring API, or entitlement tool. Do not invent `qualify_lead`, `escalate_lead`, or `score_lead` tools.

Use the exact host-exposed identifiers, including qualification such as `Mermail:list_emails`. Pass `query` and `body` as **native JSON objects**. Never stringify them. Do not guess tool names or call a missing tool under another namespace.

| Operation | Existing tools | Contract to read when used |
| --- | --- | --- |
| Resolve workspace/mailbox | `list_workspaces`, `get_workspace`, `list_mailboxes`, `get_mailbox`; `create_mailbox` only if authorized | [Workspace tools](../../mermail-administer-workspace/references/tools.md) |
| Discover/read inbound mail | `list_emails`, `search_emails`, `get_email`, `get_email_context`, `get_thread` | [Inbox tools](../../mermail-manage-inbox/references/tools.md) |
| Organize after preview | `list_folders`, `create_folder`, `list_custom_labels`, `create_custom_label`, `move_email`, `bulk_move_emails`, `update_email`, `mark_thread_read` | [Inbox tools](../../mermail-manage-inbox/references/tools.md) |
| Read selected attachment | `download_attachment` | [Inbox security](../../mermail-manage-inbox/references/security.md) |
| Draft / approved reply / handoff | `save_draft`, `regenerate_draft`, `reply_to_email`, `forward_email`, `send_email`, `schedule_email_send` | [Composition tools](../../mermail-compose-email/references/tools.md) |

This skill owns **no** tools in `tool-coverage.json` (same pattern as `mermail-research-agent` / `mermail-gtm-agent`). Route tool ownership stays with the domain skills above.

## Mailbox and query shapes

- Prefer mailbox `public_id` as `mailboxId`.
- Full-profile Mermail access is needed for drafting/replies. The restricted agent-inbox profile is not a lead-desk execution surface. API-key mail access never unlocks PayBox.
- Draft content is the string `body.body`; send/reply content is `body.text` and/or `body.html`, with required `body.from`. On a reply, use the exact source `emailId`; recipients remain explicit even though threading headers are set server-side.
- Default discovery: newest inbox metadata, unread preference, capped page size.

Metadata-first unread scan:

```json
{
  "mailboxId": "MAILBOX_PUBLIC_ID",
  "query": {
    "folder": "inbox",
    "page": 1,
    "limit": 10,
    "sortColumn": "date",
    "sortDirection": "DESC",
    "metadata_only": true,
    "agent_safe_content": true
  }
}
```

Targeted search (example free-text filters; adjust to owner criteria):

```json
{
  "mailboxId": "MAILBOX_PUBLIC_ID",
  "query": {
    "q": "partnership OR intro OR collaboration OR sponsor",
    "folder": "inbox",
    "page": 1,
    "limit": 10,
    "metadata_only": true,
    "agent_safe_content": true
  }
}
```

Selected message body (only after metadata selection):

```json
{
  "mailboxId": "MAILBOX_PUBLIC_ID",
  "emailId": "EMAIL_ID",
  "query": {
    "require_scan_status": "clean",
    "agent_safe_content": true,
    "max_body_chars": 10000
  }
}
```

`get_email_context` supports bounded cursor pagination; default this workflow to eight relevant messages and 10,000 normalized characters per message, recording truncation.

## Organization and composition

- Preview folder/label names and exact email IDs before `create_folder`, `create_custom_label`, `move_email`, or `bulk_move_emails`.
- Prefer existing folders/labels when names already match Hot / Warm / Cold / Spam (or owner-chosen equivalents).
- `save_draft` while copy is revised. Never claim a draft was sent.
- External effects (`reply_to_email`, `send_email`, `forward_email`, `schedule_email_send`) need exact preview and fresh owner approval.
- Destructive inbox tools (`delete_email`, `bulk_delete_emails`, `empty_trash`, `delete_folder`, `delete_custom_label`) are out of scope for routine lead passes; require explicit owner request plus `prepare_destructive_action` when ever used.
- Do not call PayBox tools, Composio email toolkits, or invent escalate/score APIs.

## Failure handling

Preserve structured errors (`code`, safe `details`, and `Retry-After`). A validation failure calls for correcting the exact invalid field, not broadening authority. On an uncertain external write, perform one bounded authoritative state check; stop dependent effects if still unresolved. Never auto-retry a send-like write.

Log only necessary mailbox/email/thread/draft IDs, classification labels, timestamps, and safe status codes. Do not log full lead bodies, attachments, credentials, or raw provider responses into this skill package.
