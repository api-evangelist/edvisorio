---
name: Create a student and build a quote (Edvisor.io)
description: Register a prospective student in an Edvisor agency account and generate, price, and publish a quote for school offerings.
api: graphql/edvisorio-api-v2-graphql.graphql
endpoint: https://api-v2.edvisor.io/graphql
operations: [createStudent, offeringsList, generateQuotePrice, createStudentQuote, publishStudentQuote]
---

# Create a student and build a quote

Edvisor.io is a GraphQL API at `https://api-v2.edvisor.io/graphql`. Send GraphQL over
HTTP POST with `Content-Type: application/json` and an `Authorization: Bearer <api_key>`
header. All calls must be made server-side. Responses are always HTTP 200 — check the
top-level `errors[]` array for failures (see `errors/edvisorio-problem-types.yml`).

## Steps

1. **Create the student.** Call the `createStudent` mutation with the prospect's
   details (name, email, agency association). Keep the returned `studentId`.
2. **Find offerings.** Call the `offeringsList` query with a `filter` and
   `pagination: { limit, offset }` to find candidate school programs/offerings.
3. **Price it.** Call the `generateQuotePrice` query with the chosen offerings and
   student parameters to compute live pricing (fees, currency).
4. **Create the quote.** Call the `createStudentQuote` mutation, associating it with
   the `studentId` and the selected offering options.
5. **Publish.** Call the `publishStudentQuote` mutation so the quote becomes visible
   to the student.

## Rules

- Pagination is offset/limit via `PaginationInput` (`conventions/edvisorio-conventions.yml`).
- There is no idempotency-key contract; avoid blind retries of `createStudent` /
  `createStudentQuote` to prevent duplicates. Use `upsertStudent` when you have a
  stable business key.
- Prefer `createStudentQuote` over the deprecated `createStudentQuoteDeprecated`.
