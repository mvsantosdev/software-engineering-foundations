# Fixes, one at a time

Apply in the order given in `SKILL.md`. For each section: show the plan, wait for approval, one step or migration at a time, tests at the end.

## 1. Integration tests

Locks behavior before anything else is touched.

1. List the critical flows: signup, login, checkout, webhooks, and every route that touches money, permissions, or another user's data.
2. Write the behavior statements in plain language using `describe` / `it` ("when..., then..."), covering:
   - happy path;
   - expected errors: declined payment, invalid field, user without permission;
   - repetition: double click, duplicate webhook, two simultaneous requests;
   - limits: zero stock, expired plan, zero or negative amount;
   - unauthorized access: user A trying to read or change user B's data.
3. Show the statements and wait for confirmation. The app owner is the source of truth for expected behavior.
4. Set up the environment with the tool the project already uses; if there is none, suggest the simplest one for the stack and wait for approval. A separate test database — local Supabase, container, or test-only project. Never point at production.
5. Hit the real routes (API routes, server actions, edge functions). Mock only external services: gateway, email, third-party APIs. Database, rules, and policies run for real. Simultaneous repetition uses `Promise.all`.
6. Optional, only with approval: a single smoke E2E test (Playwright) from signup to checkout.
7. Run everything and report what passes and what fails. A real bug becomes a bug-list item; do not adjust the test.
8. Hand over the single command to run the suite after every AI change.

Canonical repetition test:

```js
describe('payment webhook', () => {
  it('when the same event arrives twice, unlocks the plan only once', async () => {
    await post(webhook, paidEvent)
    await post(webhook, paidEvent)
    expect(await subscriptions(user)).toBe(1)
  })

  it('when both copies arrive together, unlocks only once', async () => {
    await Promise.all([post(webhook, paidEvent), post(webhook, paidEvent)])
    expect(await subscriptions(user)).toBe(1)
  })
})
```

Acceptance: the suite runs with one command, critical flows hit the real route, and repetition tests prove a single effect. RLS tests only count against a real database.

## 2. Never trust the client

Everything in the browser can be read and changed by the user. Frontend validation serves the experience; the server decides.

1. List every calculation or decision that exists only on the client: price, discount, shipping, permission, admin role, plan limit. Move it to the server (API route, server action, edge function) and recalculate while ignoring client-sent values.
2. Secrets: remove every secret key from the frontend and classify the public-by-design ones. Rotate anything already published, and keep `service_role`, AI keys, and gateway secrets server-side only.
3. Database exposed via API:
   - enable RLS on every table in exposed schemas;
   - write the minimal policy per operation, comparing against `auth.uid()`;
   - keep sensitive fields (payment status, role, balance) changeable only by the server;
   - confirm the secret key never reaches the browser.
4. Deliver the report first: finding, risk, `file:line`, fix. Then apply one fix at a time, with a separate migration for policies, waiting for approval on each.

Acceptance: tampering with the price in DevTools does not change the amount charged; user A cannot read or change B's data; the client cannot write sensitive fields; no secrets in the bundle.

## 3. Race conditions

Two requests read the same value and both write over it.

1. Scan the code and list every "read, decide, write" over a limited resource: order, stock, coupon, balance, credits, slots, invitations, counters.
2. For each occurrence, state in one sentence what happens with two identical requests in the same millisecond.
3. Fix it in the database, in this order of preference:
   - unique index preventing the duplicate record;
   - atomic operation, such as `UPDATE stock SET qty = qty - 1 WHERE id = X AND qty > 0`, checking affected rows;
   - transaction with `SELECT ... FOR UPDATE` when the rule is more complex.
4. On the frontend, disable the button while submitting — UX comfort. Real protection lives on the server and in the database.
5. Write a test that fires two simultaneous requests and proves only one takes effect.

If duplicates already exist that block the constraint, say so and propose a cleanup. Nothing is deleted without authorization.

Acceptance: the simultaneous test passes, stock never goes negative, a coupon is never used twice, and no solution depends on an in-memory lock, Redis, or cache.

## 4. Idempotency

Repeating an operation must produce the same result as running it once.

1. List every received webhook and every endpoint that charges, unlocks a plan, credits a balance, or sends transactional email.
2. Webhooks:
   - verify the signature against the raw request body before any processing;
   - create a processed-events table with the event ID as a unique key;
   - on receipt, try to write the ID first; if it already exists, return 200 and stop;
   - do not depend on arrival order: fetch current state from the gateway API when needed;
   - return 2xx fast and move heavy work to async processing if the project already has that structure.
3. Your calls to the gateway (charge, subscription, refund): send an idempotency key (UUID v4) created once per business operation and reused on retry.
4. App endpoints that create orders or payments: accept `Idempotency-Key` from the client and return the same response when the key repeats.
5. Test the same event twice, and twice at the same time, proving a single effect.

Idempotency keys are event or operation IDs — email, tax IDs, and other personal data stay out. Prices and billing rules remain unchanged.

Acceptance: a duplicate webhook does not credit twice, a charge retry does not create a second charge, and both sequential and concurrent tests pass.

## 5. Separation of concerns

Split without changing behavior.

1. Map the file: every function, component, and snippet, classified as UI, business rule, or data access and external services.
2. Point out cross-dependencies, duplicated code, and global state that get in the way of separation.
3. Propose the new folder and file structure following the project's existing pattern. No new framework or library.
4. Build a plan in small steps. Each step moves one piece, keeps the same public names, and leaves the app running.
5. Before step 1, check whether tests cover current behavior. If not, write characterization tests — they record today's behavior, even if it looks wrong.
6. Execute one step at a time and run the tests at the end of each.

Bugs found go in a separate list; refactoring does not fix behavior or rename routes, public props, or database columns.

Acceptance: the original file shed responsibilities, tests stay green, no route or public contract changed, and each step was reviewed in isolation.
