---
name: execute
description: Carry an approved task through to working, verified code
---

# Execute

Use this skill when the work is understood and the job is to build it.

This is the canonical execution workflow. `autopilot`, `ralph`, `ultragoal`,
`ultrapilot`, `pipeline`, and `swarm` route here.

## Goal
Take a task from agreed intent to working code, with evidence that it works.

## Workflow
1. Confirm the task is clear enough to build. If it is not, plan first.
2. Break the work into independent units; run genuinely independent units in parallel.
3. Implement the smallest correct change per unit, reusing existing utilities and patterns.
4. Verify as you go, not only at the end.
5. Report what changed, what was verified, and what remains.

**hexlog (fork `omc-hexlog`)**: if `.hexlog/flow.md` exists at the repo root, invoke `Skill("hexlog-flow")` for phase `execucao` at these steps; skip silently when the file is absent, and skip all of it when execute runs inside ralph, autopilot or team (they register). Target: the plan/spec basename when the task came from one; otherwise `hex:target:execute-{task-slug}`.
- Step 3/4, per unit, once its sequence ends: one `deviation` for each non-`none` `## Deviations` an executor reported, and one `deviation` `verification-failure` with `attempts[]` in order when a unit needed a retry after a failed check (`decidedBy: executor` or `orchestrator`). hexlog-audit:RG-4 hexlog-audit:RG-6
- Step 5, before reporting: `timeline` of the target must show every deviation above. `timeline` is paginated (limit 50, `nextCursor`): page with `since` = `nextCursor` until it is null before treating anything as missing. Register any gap now, saying in `source` that it is late. hexlog-audit:RG-9
- Step 5: milestone `phase-completed`. Do not register `completion-verified`: approval is recorded by `review` (gate `review-approved`) or `verify` (gate `verified`).
- User cancel: milestone `cancelled`. A stop that needs the user: milestone `escalated`. Register a `deviation` at the same moment with the reason and what was tried (`outcome.status: user-stop`, `decidedBy: user`). hexlog-audit:RG-8
- General rule: load `references/audit-types.md` of the `hexlog-flow` skill before recording an audit type. hexlog-audit:RG-10

## Scale
Match the machinery to the task:
- **Single unit** — implement directly, verify, done.
- **Several independent units** — delegate to `executor` agents in parallel.
- **Long-running or unbounded** — keep a durable task list and continue until the list is empty.
- **Needs coordinated parallel workers** — use `team`.

Do not spin up coordination for work that one focused pass would finish.

## Rules
- Prefer deletion over addition when behavior is preserved.
- Do not add dependencies without an explicit request.
- Keep diffs small and reversible.
- Placeholder TODOs, `test.skip`, and stub tests are blockers, not progress.
- Authoring and approval are separate passes — do not self-approve; hand off to `review` or `verify`.

## Completion
Before claiming done:
- No pending tasks
- Tests pass, or failures are reported plainly
- Verification evidence collected

## Output
- Files changed
- What was implemented
- Evidence it works
- What is still open
