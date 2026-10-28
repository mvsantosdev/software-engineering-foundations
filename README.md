# Software Engineering Foundations for AI-Built Apps

An agent skill for apps built with AI (vibe coding), covering five foundations. It turns the checklist into a workflow: read-only audit first, then fixes in the order that keeps you safe. Works with any harness supported by the [Agent Skills CLI](https://github.com/vercel-labs/skills) — OpenCode, Claude Code, Codex, Cursor, GitHub Copilot, Gemini CLI, and 75+ more.

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

Requires Node.js 22.20 or later. The [Agent Skills CLI](https://github.com/vercel-labs/skills) detects your installed agents and installs to the right directory for each one.

```sh
# Project-local (default), shared with the team
npx skills add mvsantosdev/software-engineering-foundations

# Global, available in every project
npx skills add mvsantosdev/software-engineering-foundations -g
```

Non-interactive, targeting specific harnesses:

```sh
npx skills add mvsantosdev/software-engineering-foundations -g -a claude-code -a opencode -y
```

Useful flags:

| Flag | Effect |
| --- | --- |
| `--list` | Show the skills in the repo without installing |
| `-a, --agent` | Target specific harnesses; omit to choose interactively |
| `-g, --global` | Install for the user instead of the project |
| `--copy` | Copy files instead of symlinking (needed on Windows without Developer Mode) |
| `-y, --yes` | Skip prompts, for CI |

Update or remove later:

```sh
npx skills update software-engineering-foundations
npx skills remove software-engineering-foundations
```

### Manual install

Copy this directory into any agent's skills folder — `.opencode/skills/`, `.claude/skills/`, `.agents/skills/`, or the equivalent. The directory name becomes the skill ID.

```sh
mkdir -p .opencode/skills
cp -r software-engineering-foundations .opencode/skills/
```

On Windows (PowerShell):

```powershell
New-Item -ItemType Directory -Force .opencode/skills | Out-Null
Copy-Item -Recurse software-engineering-foundations .opencode/skills/
```

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
