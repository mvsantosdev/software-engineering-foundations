# Read-only audit

Goal: produce `AUDIT.md` without changing the project — no files, migrations, or installs.

## Preparation

- Identify the stack, database, test tooling, the single test command, and which environments exist (production and test).
- Use read-only commands only.
- If a test could touch production, do not run it: record "not verified" and why.

## Stage 1 — Integration tests

- Is a test tool configured? Is there a test database separate from production? What is the command to run everything?
- Run the suite (only if it does not touch production) and report how many tests pass and how many fail.
- List the critical flows — signup, login, checkout, subscription, webhooks, and any route that touches money, permissions, or another user's data.
- Classify each flow: integration test hitting the real route / unit test with mocks only / no test.

## Stage 2 — Client trust

- Pricing, discount, shipping, permission, admin role, and plan-limit rules that exist only in the frontend.
- Secrets in frontend code or in variables that reach the browser (`NEXT_PUBLIC_`, `VITE_`, `EXPO_PUBLIC_`, or equivalent). Classify each as public by design (publishable/anon key) or secret (service_role, AI keys, gateway secret), and list the ones that need rotation because they were already published.
- Database exposed via API (Supabase or equivalent):
  - tables in exposed schemas with RLS disabled;
  - for each policy, who can read, insert, update, and delete, in plain language;
  - overly permissive policies: `USING (true)`, `WITH CHECK (true)`, rules that do not compare against `auth.uid()`;
  - sensitive fields (payment status, role, balance) the client can change directly;
  - use of the `service_role` key in the browser.

## Stage 3 — Race conditions

- List every snippet that "reads, decides, then writes" over a limited resource: order creation, stock decrement, coupon redemption, balance, credits, slots, invitations, counters.
- For each one: what happens if two identical requests arrive in the same millisecond; and whether the database protects it — unique index, atomic operation with an affected-rows check, or `SELECT ... FOR UPDATE`.
- Flag protection that depends on an in-memory lock, an expiring Redis lock, or a cache read, and say whether the database alone would solve it.

## Stage 4 — Idempotency

- Incoming webhooks: is the signature verified against the raw request body before any processing? Is there a processed-events table with the event ID as a unique key? What happens if the same event arrives twice?
- Calls to the payment gateway (charge, subscription, refund): do they use an idempotency key generated once per business operation?
- Endpoints that create orders or payments: do they accept `Idempotency-Key` from the client and return the same response when the key repeats?

## Stage 5 — Separation of concerns

- List the project's longest files and, in each one, what is UI, what is business logic, and what is database or external-service access.
- Point out cross-dependencies, duplicated code, and global state that make separation harder.
- Name the number-one candidate for refactoring.

## Deliverable — `AUDIT.md` at the root

1. At the top, the five highest-risk findings, in order.
2. One table per stage: **finding | risk (high/medium/low) | file:line | suggested fix**.
3. What could not be verified, with the reason. No guessing.
4. On the last line, which of the five fixes to run first and why.
