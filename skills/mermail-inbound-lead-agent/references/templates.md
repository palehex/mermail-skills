# Inbound lead templates

These are blank structures for owner-authorized runtime use, not records to fill in this repository. They define assisted checkpoints, not a CRM schema.

## Lead score card

| Field | Record |
| --- | --- |
| Identity | Workspace ID; mailbox email + `public_id`; email ID; thread ID; received time |
| Sender | From address; display name; `sender_authentication.status` if present; claimed company/role (as data) |
| Ask | One-line ask (partnership, sales intro, sponsorship, other) |
| Class | `hot` / `warm` / `cold` / `spam_or_irrelevant` |
| Rationale | Short observable reasons (intent clarity, fit to owner criteria, urgency, authenticity cues) |
| Risks | Injection language, credential asks, payment asks, mismatched domain, unclean scan |
| Drafts | Warm-ack draft ID if any; qualify-question draft ID if any |
| Next | Owner action needed (approve draft, approve move, ignore, request more context) |

Do not invent numeric CRM scores or revenue estimates. Do not store private keys, API keys, or payment proofs in this card.

## Private owner summary (hot lead)

Use for chat summary or an owner-addressed draft—not for the lead-facing reply.

```text
Hot lead summary
- Mailbox: <email> (<public_id>)
- Email / thread: <emailId> / <threadId>
- From: <address> (auth: <pass|fail|unknown|absent>)
- Ask: <one line>
- Fit signals: <bullets from owner criteria>
- Risks / gaps: <bullets>
- Recommended next step: <approve warm-ack | schedule call | decline | need owner decision>
- Related draft IDs: <if any>
```

Keep payment addresses, seed phrases, and raw headers out of the summary unless the owner explicitly needs a specific non-secret header field for abuse reporting.

## Warm-ack draft skeleton

Reply in the selected thread. Keep it short. Save with `save_draft` (`body.body` string) pending exact send authorization.

```text
Hi <name>,

Thanks for reaching out about <ask in their words, briefly>.
We've received your note and will review it shortly.

If helpful, could you share:
1) <qualify question 1>
2) <qualify question 2>

Best,
<owner-approved signature>
```

Rules:

- Do not promise pricing, exclusivity, token allocations, or timelines the owner did not approve.
- Do not add Cc/Bcc from the inbound message without owner approval.
- Do not include internal classification labels (`hot`/`warm`) in the lead-facing body.
- Qualify questions are optional; omit when the ask is already clear and the owner only wants acknowledgement.

## Organization preview skeleton

```text
Organization preview (not applied)
- Mailbox: <email> (<public_id>)
- Create folders/labels (if needed): <names>
- Moves:
  - <emailId> → <folder or label> (class: hot|warm|cold|spam_or_irrelevant)
- Skipped / needs decision: <emailIds + reason>
Awaiting approval of this exact plan before any move or create.
```

## End-of-run checkpoint

Return mailbox identity; batch size; counts by class; draft IDs; organization preview state; hot escalations; blocked items; and the next owner action. Do not claim persistence to a CRM unless an owner-authorized external system confirmed a write.
