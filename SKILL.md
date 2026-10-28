---
name: software-engineering-foundations
description: Audit and fix AI-built (vibe-coded) apps across five foundations — integration tests, client trust and RLS, race conditions, idempotency, and separation of concerns. Use when reviewing an AI-generated app, or when investigating duplicate orders, negative stock, coupons used twice, repeated webhooks, double-credited balances, secrets in the frontend, prices calculated client-side, oversized files, or missing tests.
---

Five predictable failures in apps built with AI, in the order they should be fixed: tests first (they lock behavior), then the open door (client trust), then money (race conditions and idempotency), and finally tidiness (separation of concerns).

## Execution rules

- Work on a separate branch; main receives reviewed merges.
- Show the plan and wait for approval before changing any file.
- New migration: present the SQL and wait for human review before running it. Production stays untouched.
- Prefer the database: a unique index, an atomic operation, or a row lock beats Redis, a queue, or an in-memory lock. If new infrastructure is truly needed, explain why the database is not enough.
- Explain each change in one sentence the app owner understands. If they don't get it, the change is not ready.
- A test failing on a real bug stays failing and goes on a bug list.

## The five foundations

| Foundation | Signal | Correct protection |
| --- | --- | --- |
| Integration tests | No test hits the real route; everything is mocked; the customer finds the breakage | Real route and test database; mock only external services |
| Client trust | Price, discount, permission, or limit only in the frontend; secret in a public env var; RLS off or `USING (true)` | Server recalculates everything; policies keyed to `auth.uid()`; secrets server-side only |
| Race conditions | Duplicate order, negative stock, coupon used twice, overdrawn balance | Unique index → conditional atomic UPDATE → `SELECT ... FOR UPDATE` |
| Idempotency | Repeated webhook, duplicate email, double-credited balance, duplicate subscription | Processed-events table with a unique ID; `Idempotency-Key` per operation |
| Separation of concerns | Giant file; changing one thing breaks another | UI, business rules, and data access separated in small steps |

## Workflow

### 1. Audit (read-only)

Read `references/audit.md` and run the five stages in order. Do not edit code, run migrations, or install anything. Write `AUDIT.md` at the project root.

Done when all five stages have findings (or "not verified" with a reason), each finding has a risk, `file:line`, and fix, and the top lists the five highest risks.

### 2. Pick the first cut

Present the five highest risks and the fix order. Wait for the human to confirm what goes in now.

### 3. Fix

Read the matching section in `references/fixes.md`, one at a time, in this order:

1. **Integration tests** — lock behavior before any change.
2. **Never trust the client** — close the door: secrets, RLS, server-side rules.
3. **Race conditions** — money and limited resources.
4. **Idempotency** — webhooks and billing.
5. **Separation of concerns** — last, with tests protecting you.

For each fix: plan → approval → one step or migration at a time → tests at the end. Done when the single test command runs and you report what passes and what fails.

### 4. Close out

Update `AUDIT.md` with what was fixed, what remains, and the single command to run the tests.

## Shortcuts

- Diagnosis only: do step 1 and stop.
- Known problem: go straight to that foundation's section in `references/fixes.md`, but run the existing tests first.

Source: "Fundamentos de engenharia de software para vibe coders" by Mano Deyvin — <https://manodeyvin.com.br/p/fundamentos-de-engenharia-de-software>.
