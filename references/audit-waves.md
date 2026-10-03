# Audit Waves — Slice and Sweep (detail for SKILL.md)

## 1. Slice the tree

One audit unit = one top-level directory, or one giant module (a ~1 MB file is a
unit by itself). Drop units with no code for the goal (docs, translations,
assets, vendored data). Units too large for a ~25-minute pass may be split or
partially covered — record coverage honestly (`Search surface`).

Typical unit examples: `cron/`, `gateway/`, `tools/`, `agent/`, `cli.py`,
`hermes_state*.py`, `apps/desktop/src`, `apps/desktop/electron`.

## 2. Prep one read-only snapshot

Fetch the upstream tip, then create a detached worktree pinned to it
(e.g. `<ops>/audit-main`). All audit subagents read from this snapshot; none
write to it. Rationale: findings stay anchored to one commit; dirty trees poison
`Load`.

## 3. Dispatch a wave (parallel, bounded)

Wave = N subagents (pick N for your host; 5 worked). Each gets ONE unit and the
same contract:

- Read the pit-stop (or equivalent) SKILL + audit + guardrails references first.
- Phases: Load → Find → Propose only, then **STOP**. Forbidden: Fix, edits,
  tests, builds, installs, network, repo scripts, commit, push, secrets/`.env`.
- Timebox ~25 min. Unpinned root cause → `needs-evidence` or drop, never invent.
- Bias to real defects (security with a traced path, observability, traced
  null/type deref, TOCTOU/shared state, protocol correctness). Mark style-class
  (`delete/stdlib/native/yagni/shrink`) and "unlikely to be accepted upstream".
- Every finding: `path:line` (unique anchor; first hit + `same text at N, M, …`
  for repeats), tag, strength (`Strong/Worth exploring/Speculative`), symptom,
  evidence, impact, minimal fix, cost. Plus a NOT-doing list with triggers.
- Write a full markdown report to `<ops>/audit[/<wave>]/<unit>.md`.
- Return a pure-ASCII summary (≤15 lines): `SUB=<unit> STATUS=ok|empty|error`,
  strength/tag counts, top 5 as `STRENGTH|tag|file:line|one-line symptom`.
  Fixed shape because the parent parses it.

## 4. Guardrails for the sweep

- The skill has no native find-only mode: the user instruction to stop after
  Propose overrides it (user instructions beat skills).
- Verify every anchor file/line yourself before fixing — auditors mislabel paths.
- Cost: 5 parallel agents over a huge tree is expensive; one wave at a time,
  triage before the next.
- Do not run the repo's tests during the sweep; verification belongs to stage 5.
