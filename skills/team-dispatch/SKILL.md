---
name: team-dispatch
description: An orchestration playbook for safely dispatching specs, PRs, and subsystems to multiple subagents in parallel and coordinating them. Use when one task should be split across several agents — feature development dispatch, PR/code-review fan-out, large refactors/migrations, multi-angle bug investigation, docs/onboarding generation. Matches requests like "run this as a team", "split this across agents", "dispatch in parallel", "review fan-out", "distribute this brief".
---

# team-dispatch — a subagent team-orchestration playbook

The standard procedure for splitting one task across multiple agents **without collisions**, then having a lead integrate, verify, and report in one place. Codebase-agnostic — identical in any repo.

Running this skill, you are the **engineering lead (PM)**. Don't do everything solo — dispatch and integrate (trivial 1–2 file changes are the exception; see "When not to use").

## When to use / when not to

Use it:
- Splitting a spec/ticket/brief across roles (backend, frontend, devops, qa, security, …) for implementation
- Reviewing a PR, branch diff, or subsystem along several dimensions at once (correctness, security, performance, tests)
- Applying the same change across N files/modules (refactor, migration)
- Digging into an unexplained bug from several angles simultaneously
- Reading several subsystems to produce docs or an onboarding map

Don't (over-engineering):
- An obvious change to 1–3 files → just do it yourself
- A simple fact lookup or single-file read → search directly
- A large fan-out (10+) for a light check or read-only survey → go small; match scale to the task

## The backbone (five steps, common to every mode)

1. **Recon & mode selection** — Look at the nature of the work and pick a mode below. If needed, skim directly first (file lists, channels, diff scope) to build the work-list.
2. **Freeze the contract** — Pin the interface every parallel owner will share **before anything else**: types/function signatures, file ownership, routes and response shapes, enums, acceptance criteria (AC). Write it into a single scratchpad file and link it in every owner prompt. **Skip this step and owners break each other through interface drift.**
3. **Assign a single owner per file** — Each file/directory is written by **exactly one owner**. If two would overlap, split the work or make one read-only. (Reason: subagents share the same working tree → double-assigning a file = silent overwrites.) If parallel writes are unavoidable, isolate owners in separate **worktrees**.
4. **Fan out in waves** — Respect hard dependencies: **the foundation (types, contracts, shared modules) must land first** so dependent owners can typecheck. Once it lands, spawn the rest **concurrently in one message** (independent work in parallel). No dependencies → everything in parallel.
5. **Integration re-verification & report** — Each owner's isolated green ≠ integration green. **The lead re-runs the full gate suite in the working tree itself** (typecheck, tests, lint, regressions). Then the four-part report (① what was done ② files changed/created ③ verification results ④ remaining risks). Shut down finished agents.

## Per-mode handles

- **A. Development dispatch** — write · shared tree, single owners · waves. Owners per role (backend/frontend/devops/qa/security). Contract freeze matters most here. **Spec gate (human approval)**: immediately after freezing the contract and **before** spawning any write owner, show the human a contract summary (interfaces, file ownership, acceptance criteria) and get approval. No write spawns before approval — the same principle as the incident flow's "go ahead and fix" gate. Parallel implementation running to completion on top of a wrong contract is the most expensive failure mode.
- **B. Review fan-out** — read-only (zero collisions) · no file ownership needed · minimal barriers. Run dimension-specific reviewers in parallel, then do NOT report findings as-is — pass them through the **adversarial verification procedure** below before synthesis. The safest first introduction for a team.
  1. Collect all reviewers' findings and dedupe by file and line.
  2. Attach a **dedicated refuter agent** to each finding (never the agent that found it). Prompt: "Refute this finding. If you cannot reconstruct a concrete failure scenario — specific input → wrong output — from the actual code, judge it refuted."
  3. Report only the findings that survive refutation as CONFIRMED; drop the rest. Default is one verifier per finding; expand to a 3-verifier majority vote only when the user asks for a thorough/exhaustive review (cost scales with size).
- **C. Refactor / migration** — write · **worktree isolation or single owner per file**. Discover targets → transform → verify per file. Single-owner discipline is what prevents the parallel-edit disaster.
- **D. Bug investigation** — read-only fan-out (call sites, data, timeline, logs — each agent takes a different angle) → synthesis → narrow to **one fix owner** for the write. In an incident/outage context, use `references/incident-templates.md` for the contract and report formats (the user brings the code themselves; no git automation).
- **E. Docs / onboarding** — read-only · purely parallel. One reader per subsystem → structured map → synthesis.

## Non-negotiable discipline (checklist)

- [ ] Was the contract frozen **first** (interfaces pinned before any owner spawned)?
- [ ] (Mode A) Was the frozen contract **approved by the human** before write owners spawned?
- [ ] (Modes B/D) Was nothing reported to the human **without adversarial verification**?
- [ ] Does every file have **exactly one** owner (zero overlap, or isolated)?
- [ ] Were hard dependencies split into waves (foundation first)?
- [ ] Were write owners run as a named (visible) team (no critical coding in hidden background agents)?
- [ ] Was the scale matched to the task (no oversized fan-out for light work)?
- [ ] Did the lead distrust isolated green and **re-run the integration gate itself**?
- [ ] Were finished agents shut down?

## Owner prompt template (given to each agent)

```
You are the <role> owner on the <task name> team. Task #<id>.
Read the contract first (single source of truth): <scratchpad contract file path>
Files you own (modify these only): <file list>
Mission: <specific work>
Contract compliance: no unilateral changes to fields/signatures/enums. If something must
change, agree with the lead first via SendMessage.
Verification (required before completion): <typecheck/test/gate commands>
Boundaries: no changes outside owned files. <areas that must not be touched>.
On completion: mark done via TaskUpdate + report via SendMessage: ① deliverable summary
② the exact interface ③ verification results.
```

## Integration report template (lead → human)

```
① What was done — what and why, team composition / wave summary
② Files changed/created — new/modified list (confirm it matches ownership)
③ Verification results — integration gates the lead re-ran itself (pass/fail per command)
④ Remaining risks — follow-up scope, unresolved items, commit/push status
```

## Cautions

- Contract changes go through the lead only: agree, then broadcast to owners. Owners never change the shared interface unilaterally.
- If there is any security/privacy surface, always include a read-only review owner in the final wave.
- Commit and push only when the user explicitly asks. The lead never does so on its own.
