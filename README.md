# team-dispatch

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Markdown only](https://img.shields.io/badge/dependencies-none%20%C2%B7%20markdown%20only-blue.svg)](#whats-in-the-box)
[![Made for Claude Code](https://img.shields.io/badge/made%20for-Claude%20Code-d97757.svg)](https://claude.com/claude-code)

A team-orchestration playbook for [Claude Code](https://claude.com/claude-code): fan one task out to multiple subagents in parallel **without them colliding**, then integrate and verify the results in one place.

Markdown only — one skill plus six role agents. No scripts, no binaries, no dependencies.

## Why this exists

Parallel subagents fail in predictable ways:

- Two agents write the same file and one silently overwrites the other.
- Owners drift apart on interfaces (types, response shapes, field names) and break each other at integration time.
- Every agent reports green in isolation, but the combined working tree doesn't even typecheck.
- Review fan-outs report plausible-but-wrong findings that nobody tried to refute.

team-dispatch encodes the discipline that prevents each of these: **freeze the contract first, exactly one writer per file, fan out in dependency waves, adversarially verify findings, and have the lead re-run the full integration gate before reporting.**

## The five modes

| Mode | Work type | Access | Key discipline |
|------|-----------|--------|----------------|
| A | Feature development dispatch | write | Contract freeze + human spec gate before any write owner spawns |
| B | Review fan-out (PR / diff / subsystem) | read-only | Adversarial verification — only findings that survive refutation are reported |
| C | Refactor / migration across many files | write | Single owner per file, or worktree isolation |
| D | Bug investigation from multiple angles | read-only → 1 fix owner | Wide read-only fan-out, then narrow to a single writer |
| E | Docs / onboarding maps | read-only | Pure parallel readers → synthesized map |

## What's in the box

```
team-dispatch-pack/
├─ README.md
├─ INSTALL.md                          ← install instructions (copy two folders)
├─ LICENSE                             ← MIT
├─ skills/
│  └─ team-dispatch/
│     ├─ SKILL.md                      ← the orchestration playbook
│     └─ references/
│        └─ incident-templates.md      ← incident investigation & report templates
└─ agents/
   ├─ tech-lead.md                     ← lead: decompose, freeze contracts, integrate, decide
   ├─ backend-engineer.md
   ├─ frontend-engineer.md
   ├─ devops-engineer.md
   ├─ qa-engineer.md
   └─ security-reviewer.md
```

## Install

Prerequisites: [Claude Code](https://claude.com/claude-code) installed (`claude --version`). This pack is markdown only — installing means copying two folders into `~/.claude/`.

> **Caution:** this overwrites any existing agents with the same names
> (tech-lead, backend-engineer, frontend-engineer, devops-engineer,
> qa-engineer, security-reviewer). Back up `~/.claude/agents/` first if
> you have customized versions.

**macOS / Linux**

```bash
git clone https://github.com/rokaproj/team-dispatch-pack.git
cd team-dispatch-pack
mkdir -p ~/.claude/skills ~/.claude/agents
cp -r skills/* ~/.claude/skills/
cp agents/*.md ~/.claude/agents/
```

**Windows (PowerShell)**

```powershell
git clone https://github.com/rokaproj/team-dispatch-pack.git
cd team-dispatch-pack
New-Item -ItemType Directory -Force -Path "$HOME\.claude\skills", "$HOME\.claude\agents" | Out-Null
Copy-Item ".\skills\*" "$HOME\.claude\skills\" -Recurse -Force
Copy-Item ".\agents\*.md" "$HOME\.claude\agents\" -Force
```

Restart Claude Code, then verify: run `/skills` — **team-dispatch** should appear.
No git? Skill not showing up? Want to uninstall? See [INSTALL.md](INSTALL.md).

## Usage

There is nothing to run — team-dispatch is a skill, so Claude Code activates it on its own when a request matches ("run this as a team", "split this across agents", "review fan-out", …). To be explicit, just name it in the prompt, like the examples below.

### Review fan-out — the safest first run (Mode B, read-only)

> "Review this project with team-dispatch — split it into correctness, security, and test coverage."

The lead spawns one read-only reviewer per dimension, dedupes their findings, then attaches a dedicated refuter agent to each finding. Only findings that survive refutation are reported — nothing plausible-but-unverified reaches you.

### Feature development dispatch (Mode A)

> "Implement this spec with team-dispatch as a team."

The lead decomposes the spec, freezes the interface contract (types, routes, file ownership, acceptance criteria) and **shows it to you for approval before any agent writes code**. Owners then implement in dependency waves — foundation first — and the lead re-runs typecheck/tests/lint on the combined tree before reporting.

### Refactor / migration (Mode C)

> "Migrate every module from X to Y with team-dispatch."

Targets are discovered first, then transformed with exactly one write owner per file (or worktree isolation), then verified per file.

### Bug investigation (Mode D)

> "Use team-dispatch to find why `<symptom>`. Repro: `<steps>`."

Read-only agents investigate the same bug from different angles (call sites, data, timeline, logs) and the lead reports a synthesis. Code changes start only after you say "go ahead and fix" — then a single fix owner is spawned. For incidents/outages, the pack includes report templates in `skills/team-dispatch/references/incident-templates.md`.

### Docs / onboarding map (Mode E)

> "Map this codebase for onboarding with team-dispatch."

One reader per subsystem, purely parallel and read-only, synthesized into a structured map.

### What to expect on every run

- Obvious 1–3 file changes are done solo — the skill declines to over-orchestrate small tasks.
- Every run ends with a four-part report: ① what was done ② files changed/created ③ verification results (gates the lead re-ran itself) ④ remaining risks.
- `git commit` / `push` never happen unless you explicitly ask.

## Non-negotiables (the checklist the lead runs)

- Contract frozen **before** any owner spawns (interfaces, file ownership, acceptance criteria in one file)
- (Mode A) The frozen contract is **approved by the human** before write owners spawn
- Exactly **one** write owner per file — zero overlap, or isolate with worktrees
- Hard dependencies split into waves — foundation lands first
- (Modes B/D) No finding reported without **adversarial verification**
- The lead **re-runs the integration gate itself** (typecheck / tests / lint) — isolated green is not integration green
- `git commit` / `push` only when the user explicitly asks

## License

[MIT](LICENSE)
