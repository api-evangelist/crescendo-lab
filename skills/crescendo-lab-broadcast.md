---
name: send-broadcast-campaign
description: Send a 1-to-many SMS broadcast via MAAC Go, then track delivery and cost.
api: MAAC Go API
base_url: https://sms.cresclab.com/api
operations:
  - createBroadcast
  - listBroadcasts
  - getMetrics
auth: Bearer sk_live_/sk_test_ in Authorization header
generated: '2026-07-18'
method: generated
source: openapi/crescendo-lab-maacgo-openapi.yaml
---

# Send a broadcast campaign

Send bulk SMS to many recipients and measure the result.

## Steps

1. **Check budget** — `GET /wallet/balance` before a high-volume send.
2. **Create** — `POST /broadcast` (`createBroadcast`) with `{ name, body, recipients[], scheduled_at?, team? }`.
   - `recipients`: 1–10,000 numbers (E.164 or `09xxxxxxxx`).
   - `body`: same NCC rules as a send — `【brand】` prefix + `STOP`/`退訂`.
   - Omit `scheduled_at` to send now, or pass ISO 8601 to schedule (funds reserve immediately).
3. **Handle the response mode**:
   - ≤30 recipients → `200` inline with `delivered` / `failed` / `cost_cents` / `refund_cents`.
   - >30 recipients → `202` `status: "queued"`; cron dispatches in batches of 40 every 5 min.
4. **Track** — `GET /broadcast` (`listBroadcasts`) for per-broadcast delivery stats; individual DLR webhooks (`sms.delivered` / `sms.failed`) flip each message.
5. **Report** — `GET /sms/metrics` (`getMetrics`) for daily volume, delivery, and cost totals.

## Rules & error handling

- `402 insufficient_balance` → open `topup_url`, top up, retry.
- `400` → `recipients_required` / `invalid_phones` / `too_many_recipients` / `ncc_blocked`; fix and resend.
- Failed messages auto-refund the wallet.
- Cross-links: conventions/crescendo-lab-conventions.yml, errors/crescendo-lab-problem-types.yml.
