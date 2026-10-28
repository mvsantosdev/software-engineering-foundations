# Software Engineering Foundations for AI-Built Apps

An [OpenCode](https://opencode.ai) skill that audits and fixes apps built with AI (vibe coding) against five foundations. It turns the checklist into a workflow: read-only audit first, then fixes in the order that keeps you safe.

## Why

AI-generated apps tend to ship with the same five defects:

1. **No integration tests** — everything is mocked, so nothing is actually tested.
2. **Client trust** — prices, permissions, and plan limits enforced only in the browser; secrets in the bundle; RLS disabled.
3. **Race conditions** — two requests read, decide, and write over the same stock, coupon, or balance.
4. **Missing idempotency** — a repeated payment webhook credits the customer twice.
5. **No separation of concerns** — one giant file where every change breaks something else.

## What it does

- **Audit, read-only.** Walks the five foundations, runs the existing suite when it is safe, and writes `AUDIT.md` with risk, `file:line`, and a suggested fix for each finding.
- **Fix in the right order.** Integration tests first, then client trust, then race conditions and idempotency, then refactoring.
- **Keep humans in the loop.** Plans before edits, SQL reviewed before migrations, production untouched, no test weakened to pass.

## Install

Project-local:

```sh
mkdir -p .opencode/skills
cp -r software-engineering-foundations .opencode/skills/
```

Global:

```sh
mkdir -p ~/.config/opencode/skills
cp -r software-engineering-foundations ~/.config/opencode/skills/
```

Windows (PowerShell):

```powershell
New-Item -ItemType Directory -Force .opencode/skills | Out-Null
Copy-Item -Recurse software-engineering-foundations .opencode/skills/
```

OpenCode discovers `.opencode/skills/<id>/SKILL.md` automatically. No dependencies to install.

## Use

Invoke it explicitly with `@software-engineering-foundations`, or just describe the problem:

- "Audit this app I built with AI." → runs the read-only audit and writes `AUDIT.md`.
- "My payment webhook credits the customer twice." → goes straight to idempotency.
- "Review this before I launch." → audit plus client-trust and race-condition fixes.

## Structure

```text
software-engineering-foundations/
├── SKILL.md
└── references/
    ├── audit.md    # five-stage read-only audit and report format
    └── fixes.md    # five fix procedures with acceptance criteria
```

`SKILL.md` carries the order and guardrails. References load only when their foundation is being worked on, so a fix does not drag the other four into context.

## Origin and attribution

Condensed and adapted from ["Fundamentos de engenharia de software para vibe coders"](https://manodeyvin.com.br/p/fundamentos-de-engenharia-de-software) by **Mano Deyvin**. The five foundations and the technical positions come from that post. This repository's contribution is the English adaptation, the audit/fix workflow, and the acceptance criteria.

This is an independent adaptation. It is not affiliated with, reviewed by, or endorsed by the original author. If you reuse this material, keep the attribution and link back to the source.

## License

MIT — see [LICENSE](LICENSE). The license covers this adaptation and does not grant rights to the original article, which remains its author's.
