---
name: smtp2go-verify-sender-domain
description: Use when an SMTP2GO account cannot send yet because no sender is verified — adds a sender domain, surfaces the DNS records to publish, verifies it, and confirms the account can send from it. Falls back to verifying a single From address.
api: openapi/_original/smtp2go-openapi-original.yml
operations:
  - add-sender-domain
  - view-sender-domains
  - verify-a-sender-domain
  - edit-tracking-domain
  - add-a-single-sender-email
  - view-all-single-sender-emails
generated: '2026-08-13'
method: generated
source: openapi/_original/smtp2go-openapi-original.yml
---

# Verify an SMTP2GO sender

## When to use this

SMTP2GO rejects every send from an unverified sender. If a send failed, or the account is new, verify the sender before doing anything else. Prefer verifying the **domain** — it gives SPF and DKIM alignment. Verify a **single sender email** only when the user does not control the domain's DNS.

## Ground rules

- Base URL: `https://api.smtp2go.com/v3` (regional alternates `us-api`, `eu-api`, `au-api`).
- Every call is a `POST` with a JSON body. There are no GETs, no path parameters, no query parameters.
- Authenticate with the `X-Smtp2go-Api-Key` header. Read the key from `SMTP2GO_API_KEY` in the environment; never ask the user to paste it into chat.
- The key needs the relevant endpoints in its permission list. A permission failure comes back as HTTP 400 with `data.error_code` = `E_ApiResponseCodes.ENDPOINT_PERMISSION_DENIED` — not 403. Check with `view-api-key-permissions` (`POST /api_keys/permissions`), which every key may call.
- There is no idempotency key. Do not blind-retry a mutating call; re-read state with a view operation first.

## Steps

1. **See what is already verified.**
   - `POST /domain/view` (`view-sender-domains`) — send `{}`. Domains delegated from a master account appear here too, flagged with `from_master`.
   - `POST /single_sender_emails/view` (`view-all-single-sender-emails`) — send `{}`.
   - If the address the user wants to send from is already covered and verified, stop and say so.

2. **Add the domain.**
   - `POST /domain/add` (`add-sender-domain`) with the domain to verify.
   - The user must own the domain; SMTP2GO requires DNS proof.

3. **Show the DNS records.**
   - Re-read `POST /domain/view` and present the records the response returns, exactly as returned. Do not compose SPF/DKIM/CNAME values yourself — publish what the API says.
   - Tell the user DNS propagation can lag; verification will fail until the records resolve.

4. **Verify.**
   - `POST /domain/verify` (`verify-a-sender-domain`).
   - On failure, re-read `/domain/view` and report the specific record still outstanding. Do not loop — ask the user to confirm the records are live, then retry once.

5. **Optional: set the tracking subdomain.**
   - `POST /domain/tracking` (`edit-tracking-domain`) sets the subdomain used for open and click tracking.
   - `POST /domain/returnpath` (`edit-return-path-domain`) sets the return-path subdomain.

6. **Fallback: verify a single address.**
   - If the user cannot edit DNS, `POST /single_sender_emails/add` (`add-a-single-sender-email`) and tell them to click the confirmation email SMTP2GO sends to that address.
   - Confirm with `POST /single_sender_emails/view`.

7. **Confirm sendability.** Report which sender is now usable and, if the user asked to send, hand off to the provider's own `smtp2go-send-email` skill rather than composing a send here.

## Gotchas

- A master-account key can act on a child account by including `subaccount_id` in the body on these endpoints.
- Verification state is not pushed anywhere — there is no webhook for "domain verified". Poll `/domain/view`.
- Deleting a domain (`POST /domain/remove`) immediately stops sends from it. Confirm with the user before calling it.
