# Installing team-dispatch (Windows / macOS / Linux)

The **team-dispatch orchestration skill** plus **six role agents** for Claude Code — split one task across parallel subagents without collisions.

## Prerequisites

- Claude Code installed (check with `claude --version` in a terminal)
- Nothing else — this pack is markdown files only; install it as a plugin (one command,
  recommended) or by copying two folders into place
- Optional: the playbook's "named team" experience (visible teammates coordinated via
  SendMessage/TaskUpdate) requires Claude Code's agent-teams / teammate mode to be enabled.
  Without it the skill still works — owners just run as background subagents instead of
  visible panes, so expect less live visibility, not missing functionality.

## Install — Option 1: Plugin (recommended)

Inside Claude Code, run:

```
/plugin marketplace add rokaproj/team-dispatch-pack
/plugin install team-dispatch@rokaproj
```

Then run `/reload-plugins` (or restart Claude Code).

Why this is the safe path:

- **Nothing of yours is overwritten.** The plugin is installed into Claude Code's
  plugin cache (`~/.claude/plugins/`), not into `~/.claude/agents/` or `~/.claude/skills/`.
- **Everything is namespaced** — the skill appears as `team-dispatch:team-dispatch`,
  the agents as `team-dispatch:tech-lead`, `team-dispatch:backend-engineer`, and so on.
- **Your same-named agents win.** If you already have your own `tech-lead` in
  `~/.claude/agents/`, it takes precedence over the plugin's version.
- **Clean removal** — `/plugin uninstall team-dispatch@rokaproj` (or `/plugin disable`
  to keep it around) leaves your own files untouched.

## Install — Option 2: Manual copy

Clone the repository, then copy the two folders into your Claude Code user directory.
Agents are then available under their plain names (`tech-lead`, not `team-dispatch:tech-lead`).

> **Caution:** unlike the plugin install, the copy overwrites any existing agents
> with the same names (backend-engineer, frontend-engineer, devops-engineer,
> qa-engineer, security-reviewer, tech-lead). If you have customized versions,
> back up `~/.claude/agents/` first.

### macOS / Linux (bash, zsh)

```bash
git clone https://github.com/rokaproj/team-dispatch-pack.git
cd team-dispatch-pack
mkdir -p ~/.claude/skills ~/.claude/agents
cp -r skills/* ~/.claude/skills/
cp agents/*.md ~/.claude/agents/
echo "Installed. Restart Claude Code to pick up the skill and agents."
```

### Windows (PowerShell)

```powershell
git clone https://github.com/rokaproj/team-dispatch-pack.git
cd team-dispatch-pack
New-Item -ItemType Directory -Force -Path "$HOME\.claude\skills", "$HOME\.claude\agents" | Out-Null
Copy-Item ".\skills\*" "$HOME\.claude\skills\" -Recurse -Force
Copy-Item ".\agents\*.md" "$HOME\.claude\agents\" -Force
Write-Host "Installed. Restart Claude Code to pick up the skill and agents."
```

No git? Download the repo as a ZIP from GitHub (**Code → Download ZIP**), extract it, and run the same copy commands from the extracted folder.

## Verify the install

1. Restart Claude Code (open a new session), or `/reload-plugins` for the plugin install.
2. Run `/skills` — **team-dispatch** should appear in the list
   (as `team-dispatch:team-dispatch` for the plugin install).
3. Ask Claude "list the available agents" — tech-lead, backend-engineer, and the other four
   should be included (prefixed with `team-dispatch:` for the plugin install).

If the skill doesn't show up, check that the path is exactly
`~/.claude/skills/team-dispatch/SKILL.md` — an extra nested folder from extraction is the usual cause.

## First runs

The skill matches automatically on requests like these:

- **Code review (safest introduction — read-only)**
  > "Review this project with team-dispatch, split into correctness / security / test coverage."
- **Incident or bug investigation** — after you've cloned/checked out the problem branch yourself:
  > "I've checked out this branch. Use team-dispatch to find the problem. Symptom: X. Repro: Y."
  → It reports findings and stops. Code changes start only after you say "go ahead and fix" (human gate).
- **Feature development dispatch**
  > "Implement this spec with team-dispatch as a team."
  → It freezes the interface contract and shows it to you for approval **before** any agent writes code.

## Built-in rules

- Investigation and review stages are read-only — no code changes before approval
- Exactly one write owner per file — parallel edit collisions are prevented at the root
- `git clone` / `commit` / `push` are always done by you — the skill works only inside the local working tree
- The lead re-runs the integration gates (tests, lint, typecheck) itself and delivers a four-part report

## Uninstall

- **Plugin install:** `/plugin uninstall team-dispatch@rokaproj` — your own files are untouched.
- **Manual copy:** delete `~/.claude/skills/team-dispatch/` and the six agent
  `.md` files from `~/.claude/agents/`.
