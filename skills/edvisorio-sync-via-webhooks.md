---
name: Keep an external CRM in sync with Edvisor.io (webhooks + polling)
description: Subscribe to Edvisor webhook events and reconcile student/school/offering/quote changes into an external system using the GraphQL API.
api: graphql/edvisorio-api-v2-graphql.graphql
endpoint: https://api-v2.edvisor.io/graphql
operations: [student, studentsList, offering, schoolQuoteList, invoiceList]
---

# Keep an external CRM in sync with Edvisor.io

Two-way sync combines the GraphQL API (pull) with webhooks (push). See
`asyncapi/edvisorio-webhooks.yml` for the full event catalog.

## Steps

1. **Register a webhook** (via Edvisor support / account setup) pointing at your HTTPS
   endpoint and subscribe to the events you care about, e.g. `student:create`,
   `student:update`, `student:delete`, `studentQuote:create`, `offering:update`.
2. **Acknowledge deliveries.** Return a `2xx` HTTP status immediately. Any non-2xx
   (including 3xx) is treated as a failed delivery and Edvisor retries every 15
   minutes for up to 3 days.
3. **Read the payload.** Each event body carries `created` (timestamp), `user` (actor),
   and `data.before` / `data.after`. Use `after` to upsert into your system.
4. **Reconcile via the API.** For a fuller record, re-fetch the changed entity:
   `student` (by id) or `studentsList`, `offering`, `schoolQuoteList`, `invoiceList`.
5. **Backfill / repair.** On startup or after downtime, page through `studentsList`
   with `pagination: { limit, offset }` to catch anything missed while your endpoint
   was unavailable.

## Rules

- Webhook events use the `resource:action` name format (e.g. `student:update`).
- There is no documented webhook signature header — restrict your endpoint by network
  or a shared secret path segment as appropriate.
- All API reads require the `Authorization: Bearer <api_key>` header, server-side only.
