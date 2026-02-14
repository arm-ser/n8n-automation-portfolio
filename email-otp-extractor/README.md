# Email 2FA OTP Extractor Template

An n8n workflow that monitors email inboxes via IMAP for 2FA/OTP verification emails, extracts codes and sign-in links using a regex-based scoring system, deduplicates using PostgreSQL, and forwards formatted alerts to your notification channel.

## What It Does

```
IMAP Monitor (Inbox + Finance folder)
        │
  Extract Email Content
        │
  Filter OTP Emails (pattern matching)
        │
  Check if Already Processed (PostgreSQL)
        │
  Should Process? ──No──→ [end]
        │ Yes
  Extract OTP Codes & Sign-in Links
        │
  Store as Processed (PostgreSQL)
        │
  Format Message
        │
  Send to Notification Channel
```

**Also supports:** Webhook trigger for manual/external email forwarding.

## Key Features

- **Multi-mailbox monitoring** — watches multiple IMAP folders simultaneously
- **Smart OTP detection** — regex scoring system identifies 2FA emails vs regular mail
- **Code extraction** — pulls 4-8 digit OTP codes from email body
- **Link extraction** — finds sign-in/verification URLs
- **Deduplication** — PostgreSQL tracks processed message IDs to prevent duplicate alerts
- **Webhook support** — can also receive emails via HTTP webhook
- **Formatted output** — clean, readable notifications with extracted codes highlighted

## Prerequisites

| Requirement | Purpose |
|------------|---------|
| **IMAP email account(s)** | Email monitoring (Gmail, Outlook, self-hosted, etc.) |
| **PostgreSQL database** | Deduplication tracking |
| **Matrix account** (or alternative) | Alert delivery |
| **n8n instance** | v1.0+ recommended |

## Setup Instructions

1. **Import** — Go to n8n → Workflows → Import from File → select `Email_OTP_Extractor_Template.json`
2. **Create PostgreSQL database table:**
   ```sql
   CREATE TABLE processed_emails (
     id SERIAL PRIMARY KEY,
     message_id TEXT UNIQUE,
     processed_at TIMESTAMP DEFAULT NOW()
   );
   ```
3. **Create PostgreSQL credential** — Settings → Credentials → Add → Postgres → enter your DB connection details
4. **Create IMAP credential(s)** — Settings → Credentials → Add → IMAP → enter your email server details
5. **Update IMAP mailbox names** — edit the IMAP monitor nodes and set your folder names (e.g., `INBOX`, `Folders/Finance`)
6. **Create Matrix credential** (or your preferred notification service)
7. **Update Matrix room ID** — edit the Matrix node and set your room ID
8. **Activate** — toggle the workflow active

## Node Breakdown

| Node | Purpose |
|------|---------|
| Monitor Inbox | IMAP trigger watching main inbox — **set your mailbox name** |
| Monitor Finance Emails | IMAP trigger watching finance folder — **set your mailbox name** |
| Webhook | HTTP endpoint for external email forwarding |
| Extract Email Content | Set node standardizing email fields (subject, body, message_id) |
| Filter OTP Emails | Code node with regex patterns to identify 2FA/OTP emails |
| Check if Already Processed | PostgreSQL SELECT to check if message_id exists |
| Should Process Email? | If node — skips already-processed emails |
| Extract OTP and Links | Code node using regex scoring to find codes and verification URLs |
| Store Processed Email | PostgreSQL INSERT to mark email as processed |
| Format Matrix Message | Code node converting markdown to HTML for Matrix |
| Send to Matrix | Delivers the formatted OTP alert |

## Customization Guide

### Add More Email Folders
Duplicate an IMAP Monitor node, set it to watch a different folder, and connect it to the "Extract Email Content" node.

### Modify OTP Detection Patterns
Edit the `Filter OTP Emails` Code node to add patterns for services not yet covered. The node uses a scoring system — emails scoring above the threshold are classified as OTP emails.

### Change Notification Service
Replace the Matrix node with Slack, Discord, Telegram (great for mobile OTP alerts), or Pushover. Map the formatted message to the message body field.

### Database Alternatives
If you don't want to use PostgreSQL, you can:
- Replace with **SQLite** (n8n has a built-in SQLite node)
- Use a **Google Sheet** as a simple deduplication log
- Remove deduplication entirely (may get duplicate alerts)

## Security Considerations

- This workflow processes sensitive authentication data (OTP codes, sign-in links)
- Ensure your n8n instance is secured (HTTPS, authentication enabled)
- Ensure your notification channel is private
- Consider auto-deleting processed entries from the database after 24 hours
- The webhook endpoint should be protected or disabled if not needed
