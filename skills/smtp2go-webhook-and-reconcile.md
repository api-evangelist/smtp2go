---
name: smtp2go-webhook-and-reconcile
description: Use when an SMTP2GO integration needs delivery outcomes — registers an event webhook for email/SMS events, then reconciles what actually happened using activity search, bounce statistics and the suppression list.
api: openapi/_original/smtp2go-openapi-original.yml
operations:
  - add-webhook
  - view-webhook
  - edit-webhook
  - search-activity
  - view-suppressions
  - add-a-suppression
  - email-bounces
  - email-summary
generated: '2026-08-13'
method: generated
source: openapi/_original/smtp2go-openapi-original.yml
---

# Wire SMTP2GO webhooks and reconcile delivery

## When to use this

A 200 from `POST /email/send` means SMTP2GO accepted the message, **not** that it was delivered. Use this skill to set up the realtime signal and to answer "did it actually arrive?".

## Ground rules

- Base URL `https://api.smtp2go.com/v3`; every call is a `POST` with a JSON body; auth is the `X-Smtp2go-Api-Key` header sourced from `SMTP2GO_API_KEY`.
- `POST /activity/search` is rate-limited to **60 requests per minute** and returns at most 1,000 items alongside a total that may be larger. It is not a substitute for webhooks — use it to reconcile, not to poll a queue.
- On 429, back off exponentially. There are no `RateLimit-*` or `Retry-After` headers to read; the status code is the whole signal.
- Free accounts may register **1** webhook; paid accounts up to **10**.

## Steps

### Register the webhook

1. `POST /webhook/view` (`view-webhook`) with `{}` to see what already exists and avoid burning the free-plan slot.
2. `POST /webhook/add` (`add-webhook`) with the receiving URL and the events to subscribe to.
   - Email events: `processed`, `delivered`, `open`, `click`, `bounce`, `spam`, `unsubscribe`, `resubscribe`, `reject`.
   - SMS events: `sms_sending`, `sms_submitted`, `sms_delivered`, `sms_failed`, `sms_rejected`, `opt-out`.
   - A webhook can be scoped to selected SMTP users, API keys or authenticated IPs.
3. `POST /webhook/edit` (`edit-webhook`) to change the URL, event list or custom headers later. `POST /webhook/remove` deletes it.

### Tell the consumer what to expect

- Delivery is an HTTP or HTTPS `POST`, JSON or form-encoded per the webhook's setting.
- **There is no signature.** SMTP2GO does not sign webhook payloads. The documented protections are basic-auth credentials in the URL, a secret in the path or query string, or allow-listing the A record of `webhooks.smtp2go.com`. Say this plainly — do not tell a user to verify a signature that does not exist.
- Timeout is 10 seconds on response headers. Failures retry for up to 48 hours, max 35 attempts (5 in the first 30 minutes, hourly for 24 hours, then 6-hourly, then 12-hourly, then one final attempt). Failed deliveries are inspectable and replayable under Settings > Webhooks > Failed Notifications.
- Design the receiver to be idempotent: retries mean the same event can arrive many times, and `open`/`click` legitimately fire more than once per message.

### Reconcile

4. `POST /activity/search` (`search-activity`) to look up what happened to specific mail. Filter by subject, event types, date window and `limit`; `only_latest` collapses to the most recent event per message. `custom_headers` and `sender_full` can be requested to pull extra context out of the raw headers.
5. `POST /stats/email_summary` (`email-summary`) and `POST /stats/email_bounces` (`email-bounces`) for aggregate posture. Note that `spam_rejects` and `bounce_rejects` are deprecated in favour of `rejects`.
6. `POST /suppression/view` (`view-suppressions`) when a recipient stopped receiving mail. Spam complaints and unsubscribes add addresses indefinitely, and any send to a suppressed address is rejected.
7. `POST /suppression/add` (`add-a-suppression`) only on explicit user instruction — it permanently blocks that recipient. `POST /suppression/remove` reverses it, but only do that with the recipient's consent.

## Diagnosing a `reject`

A `reject` event means one of three things, and the fix differs:

- The recipient is on the suppression list → check `view-suppressions`.
- The sender is not verified → run the `smtp2go-verify-sender-domain` skill.
- The sending credential (API key, SMTP user or authenticated IP) is in **Sandbox Mode** → the account holder must switch it off in the app. See `sandbox/smtp2go-sandbox.yml`.

## Reporting

Always report the `request_id` from the API response and the `email_id` for a specific message. Never claim final delivery from a send response alone — cite a `delivered` webhook event or an activity-search result.
