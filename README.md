# SMTP2GO

Email delivery platform with a REST API for sending transactional emails, managing SMTP accounts, tracking delivery, viewing email statistics, and suppressing addresses.

- **Website:** https://www.smtp2go.com/
- **Developer Docs:** https://developers.smtp2go.com/docs/introduction-guide
- **API Reference:** https://developers.smtp2go.com/reference/general-api-resources
- **GitHub Org:** https://github.com/smtp2go-oss
- **Pricing:** https://www.smtp2go.com/pricing/
- **Status Page:** https://smtp2gostatus.com/
- **Blog:** https://www.smtp2go.com/blog/
- **LinkedIn:** https://www.linkedin.com/company/smtp2go
- **X:** https://x.com/smtp2go

## Overview

SMTP2GO provides reliable email delivery infrastructure through both SMTP relay and a REST API (base URL: `https://api.smtp2go.com/v3`). The platform supports transactional email, SMS messaging, template management, sender domain verification, webhook event delivery, suppression list management, and comprehensive reporting.

## API Capabilities

- Send individual, MIME-encoded, and batch emails
- Send SMS to up to 100 recipients per request
- Manage sender domains and single sender emails
- Manage API keys and SMTP users
- Configure webhooks for event notifications
- Create and manage email templates
- Manage suppression lists (blocks by address or domain)
- Access delivery activity, bounce, spam, and unsubscribe reports
- Manage subaccounts for resellers and agencies
- Configure dedicated IP addresses

## Authentication

Every request requires an API key passed in the request body. Keys are 32 characters with an `api-` prefix. Keys are managed under **Sending > API Keys** in the SMTP2GO dashboard.

## SDKs

Official open-source SDKs available from the `smtp2go-oss` GitHub organization:

- Python
- Node.js
- Go
- Ruby
- PHP
- Laravel
- .NET
- Rust
- Django

## Pricing Summary

| Plan | Monthly Cost | Included Emails | Overage |
|------|-------------|-----------------|---------|
| Free | $0 | 1,000 | None (capped) |
| Starter | $10 | 10,000 | $1.00/1k |
| Professional | $75 | 100,000 | $0.85/1k |
| Premier | Custom | 3,000,000+ | Custom |

Annual billing saves approximately 16.7% on Starter and Professional plans.

## Repository Contents

- `apis.yml` — APIs.json 0.19 provider index
- `plans/smtp2go-plans-pricing.yml` — API Commons Plans 0.1 pricing detail
- `rate-limits/smtp2go-rate-limits.yml` — API Commons Rate Limits 0.1
- `finops/smtp2go-finops.yml` — FinOps Framework FOCUS 1.0 cost guidance

## Maintainer

**Kin Lane** — kin@apievangelist.com
