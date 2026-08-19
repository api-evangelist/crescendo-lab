---
name: attribute-sms-cost-by-team
description: Tag MAAC Go sends with a team and report which team sent and spent how much against the shared wallet.
api: MAAC Go API
base_url: https://sms.cresclab.com/api
operations:
  - createTeam
  - listTeams
  - smsByTeam
  - getMetrics
  - deleteTeam
auth: Bearer sk_live_/sk_test_ in Authorization header
generated: '2026-08-13'
method: generated
source: openapi/crescendo-lab-teams-api-openapi.yml + openapi/crescendo-lab-sms-api-openapi.yml
---

# Attribute SMS cost by team

MAAC Go bills one prepaid wallet per account. Teams do **not** split that wallet — they are a reporting tag that answers "which team sent what, and what did it cost". Use this when several groups share one account and finance needs a breakdown.

**This capability is REST-only.** No MCP tool creates, lists or deletes a team (see `mcp/crescendo-lab-tool-crosswalk.yml`). An agent on the MCP server can only *tag* a send via the `team` field on `send_sms` / `send_broadcast`; everything below needs a direct REST call.

## Steps

1. **Create the team** — `POST /teams` (`createTeam`) with a name.
   - Creation is idempotent on `name`: re-posting an existing name does not duplicate it, and posting the name of an archived team un-archives it.
   - You can skip this entirely — passing a new `team` value on a send auto-creates the team.
2. **List teams** — `GET /teams` (`listTeams`) to confirm the tag you are about to use already resolves to the team you mean.
3. **Tag every send** — pass `team` on `POST /sms/send` (`sendSms`) and `POST /broadcast` (`createBroadcast`). An untagged send is unattributable after the fact; there is no backfill operation.
4. **Report** — `GET /sms/by-team` (`smsByTeam`) for the per-team message log, and `GET /sms/metrics` (`getMetrics`) with `days=N` for the aggregate (success rate, failure rate, latency, total cost) to reconcile against the wallet ledger.
5. **Retire a team** — `DELETE /teams` (`deleteTeam`) archives it. Archiving is reversible by re-creating the same name in step 1.

## Rules & error handling

- Attribution is set at send time only. If cost reporting matters, make the `team` field mandatory in your own wrapper before the call reaches MAAC Go.
- The wallet is shared and debited atomically on successful send; carrier-level permanent failures auto-refund it, so a team's cost total can move after the fact. Reconcile against `wallet_events`, not against a snapshot.
- `402 insufficient_balance` is an account-level condition, not a team-level one — one team exhausting the wallet blocks every other team.
- See `errors/crescendo-lab-problem-types.yml`, `rate-limits/crescendo-lab-rate-limits.yml` and `conventions/crescendo-lab-conventions.yml`.
