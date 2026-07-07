# team-dispatch

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
No git, troubleshooting, or uninstall? See [INSTALL.md](INSTALL.md).

## Quick start

Ask Claude Code something like:

> "Review this project with team-dispatch — split it into correctness, security, and test coverage."

Review fan-out (mode B) is the safest first run: it is entirely read-only.

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
