---
name: omc-review
description: Evaluate finished work for defects, risk, and simplification before it ships
---

# Review

Use this skill to evaluate work that already exists. Review never authors the
change it is judging.

This is the canonical review workflow. `merge-readiness` routes here, and
`ai-slop-cleaner` is an opt-in lane within it.

## Goal
Find what is actually wrong, ranked by severity, with enough detail to act on.

## Workflow
1. Establish what changed and what it was meant to do.
2. Read the change against that intent.
3. Check correctness first, then risk, then simplification.
4. Verify each candidate finding before reporting it.
5. Report findings most-severe first.
6. **hexlog (fork `omc-hexlog`)**: if `.hexlog/flow.md` exists at the repo root, invoke `Skill("hexlog-flow")` for phase `revisao`, target: the plan/spec basename of the reviewed change when known, otherwise `hex:target:review-{change-slug}`. `attachment({project, text})` of the full review report, verbatim; then verdict `review` = `approve` (no blocking finding) | `request-changes` (blocking findings to fix) | `reject` (evidence: that hash and the top findings, file:line); then evaluate the phase gate `review-approved`. A re-review after fixes supersedes the previous `review` verdict. The gate records the outcome only; review stays advisory. Skip silently when the file is absent.

## What to check
- **Correctness** — logic defects, edge cases, error paths, concurrency
- **Risk** — security boundaries, destructive operations, data integrity
- **Reuse** — existing utilities or patterns the change should have used
- **Simplification** — code that could be deleted or collapsed
- **Coverage** — behavior that ships untested

## Rules
- Separate lanes: the reviewer must not be the author's same active context.
- Verify before reporting. A plausible-sounding finding that does not reproduce is noise.
- State severity honestly; do not pad the list to look thorough.
- "No findings" is a valid result when the work is sound.
- Advisory by default — review informs, it does not gate. Hard gates (release,
  security, destructive operations) stay separate and fail closed.

## Output
- Findings, most-severe first, each with file, line, and concrete failure scenario
- What was checked and found clean
- Anything that could not be assessed
