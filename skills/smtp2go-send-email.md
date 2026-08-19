---
name: smtp2go-send-email
description: Use when sending transactional, test, scheduled, template, HTML, or plain text email through SMTP2GO using the direct API with an SMTP2GO_API_KEY environment variable.
---

# SMTP2GO Send Email

## Overview

Send email through SMTP2GO's standard `/email/send` endpoint using the direct HTTP API and `SMTP2GO_API_KEY` from the environment. Treat every send as an external side effect: verify recipients and content before executing the request.

## Workflow

1. Confirm the message details:
   - optional `sender`: `Name <email@example.com>`
   - `to`: array of up to 100 recipients in `Name <email@example.com>` format
   - `subject`
   - one of `text_body`, `html_body`, or `template_id`
   - optional `cc`, `bcc`, `custom_headers`, `attachments`, `inlines`, `template_data`, `schedule`, `fastaccept`

2. If `sender` is not given, query both authenticated discovery endpoints using `SMTP2GO_API_KEY`:
   - `POST https://api.smtp2go.com/v3/allowed_senders/view` ([API reference](https://developers.smtp2go.com/reference/view-allowed-senders))
   - `POST https://api.smtp2go.com/v3/single_sender_emails/view` ([API reference](https://developers.smtp2go.com/reference/view-all-single-sender-emails))
   - Send `{}` as the JSON body and use the same API-key and content-type headers as the send request.
   - Show the user the available allowed senders and single sender email addresses, including verification/status information returned by the API.
   - Recommend a verified single sender email that is permitted by the allowed-sender settings. Explain the recommendation briefly, then ask the user to choose or confirm it.
   - If neither endpoint returns a usable sender, explain that a sender must be configured or verified before sending.

3. If any other required detail is missing, ask a concise question before sending.

4. Always show a complete email preview and ask for explicit approval immediately before sending. Include sender, `to`, `cc`, `bcc`, subject, body or template details, attachments, and schedule when present. Do this even if the user previously said to send. Skip the preview and send immediately only when the user explicitly instructs both that no preview is needed and that the email should be sent directly.

5. Verify `SMTP2GO_API_KEY` is set in the shell environment before any discovery or send request. If it is missing, ask the user to provide or export it; do not ask them to paste the key into chat.

6. Send the request to the regionless API unless the user requests a region-specific endpoint:
   - `https://api.smtp2go.com/v3/email/send`
   - US: `https://us-api.smtp2go.com/v3/email/send`
   - EU: `https://eu-api.smtp2go.com/v3/email/send`
   - AU: `https://au-api.smtp2go.com/v3/email/send`

7. Report `request_id`, `email_id`, `schedule_id`, and any failures from the response. Do not claim final delivery unless webhooks or activity data confirm it.

## Request Shape

Use JSON with these core fields:

```json
{
  "sender": "Sender Name <sender@example.com>",
  "to": ["Recipient Name <recipient@example.com>"],
  "subject": "Subject line",
  "text_body": "Plain text body",
  "html_body": "<p>Optional HTML body</p>"
}
```

`sender`, `to`, and `subject` are required. `html_body` or `text_body` is required unless `template_id` is provided.

## Direct API Execution

Prefer a small Python stdlib request so the API key is read from the environment inside the process and is not placed directly in the command arguments:

```python
import json
import os
import urllib.request

payload = {
    "sender": "Sender Name <sender@example.com>",
    "to": ["Recipient Name <recipient@example.com>"],
    "subject": "Subject line",
    "text_body": "Plain text body",
}

api_key = os.environ["SMTP2GO_API_KEY"]
request = urllib.request.Request(
    "https://api.smtp2go.com/v3/email/send",
    data=json.dumps(payload).encode("utf-8"),
    headers={
        "accept": "application/json",
        "content-type": "application/json",
        "X-Smtp2go-Api-Key": api_key,
    },
    method="POST",
)

with urllib.request.urlopen(request, timeout=30) as response:
    print(response.read().decode("utf-8"))
```

`curl` is acceptable for quick one-off sends, but avoid verbose mode and never echo the key:

```bash
curl -sS https://api.smtp2go.com/v3/email/send \
  -H "accept: application/json" \
  -H "content-type: application/json" \
  -H "X-Smtp2go-Api-Key: ${SMTP2GO_API_KEY}" \
  --data '{"sender":"Sender Name <sender@example.com>","to":["Recipient Name <recipient@example.com>"],"subject":"Subject line","text_body":"Plain text body"}'
```

If network access is blocked by the runtime sandbox, request permission to run the direct API call rather than falling back to the SMTP2GO MCP.

## Options

- `custom_headers`: array of `{ "header": "...", "value": "..." }`; use this for `Reply-To`. Do not set `Content-Type`, `Content-Transfer-Encoding`, or `MIME-Version`.
- `attachments`: array with `filename` plus either `fileblob` base64 data or `url`; include `mimetype` when known.
- `inlines`: same shape as attachments; reference as `<img src="cid:filename"/>` in `html_body`.
- `template_id` and `template_data`: use instead of manually writing body content when the user specifies a template.
- `schedule`: future timestamp within the next 3 days, for example `2026-05-19 13:15:00 +1200`.
- `fastaccept`: only set when the user accepts background sending semantics.

## Safety Checks

- Re-read all recipients before sending; be especially careful with `cc` and `bcc`.
- Never send to a production/customer list unless the user explicitly confirms the exact audience.
- Show the actual body content in the preview when practical; summarize very long bodies and offer the full draft. Include all recipients, sender, subject, body or template details, attachments, and schedule.
- Treat an instruction to "send" as authorization to prepare the email, not permission to bypass the preview. Only explicit "no preview" plus direct-send wording bypasses it.
- Redact API keys and sensitive tokens from final responses and logs.
