---
name: manage-contacts
description: Build and read the MAAC Go address book with NCC consent tracking before running a broadcast.
api: MAAC Go API
base_url: https://sms.cresclab.com/api
operations:
  - createContact
  - listContacts
  - createBroadcast
auth: Bearer sk_live_/sk_test_ in Authorization header
generated: '2026-08-13'
method: generated
source: openapi/crescendo-lab-contacts-api-openapi.yml + openapi/crescendo-lab-broadcast-api-openapi.yml
---

# Manage contacts with consent tracking

The MAAC Go address book is the consent record for Taiwanese marketing SMS. Under NCC rules a marketing send has to be justifiable, so build the list here rather than carrying an ad-hoc array into every broadcast.

**This capability is REST-only.** There is no MCP tool for contacts (see `mcp/crescendo-lab-tool-crosswalk.yml`), so an agent working through the `maacgo` MCP server cannot read or write the contact book — it has to call the REST API directly with the same bearer key.

## Steps

1. **Add contacts** — `POST /contacts` (`createContact`).
   - Phones accept E.164 (`+886912345678`) or TW local (`0912345678`).
   - The endpoint dedupes duplicate phones *within* the submitted batch and skips `(user_id, phone)` pairs that already exist, so re-posting the same list is safe and does not create duplicates.
2. **Read the book back** — `GET /contacts` (`listContacts`) with `page` + `limit` (`limit` max 500, default 50). The response carries `total`; page until you have consumed it rather than assuming one page.
3. **Broadcast to the list** — `POST /broadcast` (`createBroadcast`) with the phones you just confirmed.
   - 30 recipients or fewer dispatch inline; more than 30 queue asynchronously and you should expect the result over webhooks, not in the response body.
   - Personalize with `{{var}}` placeholders and a per-recipient `vars` map.

## Rules & error handling

- Marketing-class bodies must carry a `【brand】` prefix (4–8 characters) **and** a `STOP` or `退訂` opt-out, or the send returns `400 ncc_blocked` with `MISSING_OPT_OUT` / `NO_SIGNATURE` in `issues[]`.
- Check the wallet before a large broadcast — a send that exceeds the balance returns `402 insufficient_balance` with a `topup_url`. Do not retry until the user confirms the top-up.
- New accounts (under 24h old) sit under a probation send cap; established accounts are capped at 1,000/hour. Exhaustion is `429 rate_limited`. See `rate-limits/crescendo-lab-rate-limits.yml`.
- See `errors/crescendo-lab-problem-types.yml` and `conventions/crescendo-lab-conventions.yml`.
