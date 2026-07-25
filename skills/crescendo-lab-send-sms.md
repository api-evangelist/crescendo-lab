---
name: send-transactional-sms
description: Send a 1-to-1 transactional SMS (OTP, order or reminder) via MAAC Go and confirm delivery.
api: MAAC Go API
base_url: https://sms.cresclab.com/api
operations:
  - sendSms
  - getSms
auth: Bearer sk_live_/sk_test_ in Authorization header
generated: '2026-07-18'
method: generated
source: openapi/crescendo-lab-maacgo-openapi.yaml
---

# Send a transactional SMS

Use this for OTP codes, order notifications, and appointment reminders in Taiwan.

## Steps

1. **Send** — `POST /sms/send` (`sendSms`) with `{ to, body, type }`.
   - `to`: E.164 (`+886912345678`) or TW local (`0912345678`).
   - `body`: include a `【brand】` prefix; 70 chars = 1 segment (Chinese), 160 (ASCII).
   - `type`: `otp`, `notification`, or `marketing`.
   - Optional `team`: cost-attribution tag (auto-created if new).
2. **Read the response** — capture `message_id`, `status`, `segments`, `cost_cents`, `balance_cents`.
3. **Confirm delivery** — prefer the `sms.delivered` webhook (HMAC-SHA256, `X-Cresclab-Signature`). As a fallback poll `GET /sms/{id}` (`getSms`) no more than once per 5s.

## Rules & error handling

- On `402 insufficient_balance`, open the returned `topup_url`, wait for top-up, then retry — do not loop.
- On `400 ncc_blocked`, read `issues[]` and revise copy; never bypass NCC compliance (no blocklisted shorteners; marketing needs a `STOP`/`退訂` opt-out).
- On `429 rate_limited`, back off per `retry_after`.
- See errors/crescendo-lab-problem-types.yml and conventions/crescendo-lab-conventions.yml.
