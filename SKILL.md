---
name: pit-stop-issue-skill
description: Use when asked to audit a large repository piece by piece and turn the findings into fixes, run a multi-PR contribution campaign against an upstream repo, or convert audit findings into separate worktrees/branches/pull-requests. Triggers: subsystem audit, audit in waves, fix every finding, contribute upstream, per-module review, backlog of fixes to PRs.
---

# Audit Subsystems to PRs

Split a repo into subsystems, audit them in parallel read-only waves, freeze the
triaged backlog, then land each kept finding as its own worktree → branch → RED test →
minimal fix → verified commit → fork PR. Finding without a `path:line` anchor is
not a finding; a PR without a failing-then-passing test is not a fix.

## When to Use

- A repo too big for one audit pass (thousands of files across distinct areas).
- Contributing fixes upstream where each fix must stand alone (own branch/PR).
- Turning a pile of audit output into an actionable, deduped backlog.

When NOT to use: single-file fixes (edit directly), read-only questions, repos
you own (no fork dance needed), emergencies (fix first, audit later).

## The Run

Six stages, in order. Details live in `references/`; the core below is the contract.

1. **Slice** — partition the tree into audit units: one top-level dir or one giant
   module each (see `references/audit-waves.md`). Record the unit list.
2. **Sweep** — per wave, dispatch N parallel read-only auditor subagents,
   each on ONE unit, FIND-ONLY (Load → Find → Propose, then STOP).
   Every finding carries `path:line` + tag + strength + symptom + evidence +
   impact + minimal fix + cost. See `references/audit-waves.md`.
3. **Triage** — keep only defensible defects; drop style/speculation. Then
   **dedup every survivor against upstream open issues/PRs** (`gh search`).
   See `references/triage-dedup.md`.
4. **Freeze** — write the kept items to a backlog file (one line each: anchor,
   defect, lane). This file is the wave's contract.
5. **Fix** — one item at a time: fresh worktree + branch → RED test proving the
   bug → minimal fix → GREEN → neighbor tests → push fork → cross-fork PR.
   **REQUIRED SUB-SKILL:** test-driven-development for every fix.
   See `references/fix-and-pr.md`.
6. **Accept** — per PR, verify three things: PR is OPEN, `head.sha == local
   branch sha`, worktree clean (`git status --short` empty). See
   `references/fix-and-pr.md`.

## Iron Laws

- **Find-only means find-only.** The audit agent must not enter Fix (no edits, tests, installs, network).
- **Dedup before fixing.** A finding already filed upstream becomes a `Closes #N`
  fix; a finding with an open fixing PR is hands-off. Fixing blind creates collisions.
- **Test-heavy work serializes.** Test/build runs go through one serial lock; scan in parallel, fix in series.
- **Pre-existing failures stay out.** A failure also present on the pristine base is
  environmental — document it, never fix it inside your PR.
- **Push/PR only on an explicit word.** Changes wait in the worktree; nothing is
  pushed or opened until the human says so.
- **User instructions override this skill;** where a skill demands more, follow it.

## Red Flags — STOP, You Are Rationalizing

- "The finding is obvious, skip the RED test" → the test proves the line, not your confidence.
- "Strong enough, dedup can wait" → dedup IS the work; collisions waste real reviewer time.
- "Full test suite in one process to save time" → use per-file isolation or eat phantom failures.
- "One PR for several findings to save effort" → one finding = one worktree/branch/PR.
- "This audit claim must be right, fix it" → verify the anchor file/line yourself first;
  auditors mislabel paths.
- "Tests pass, ship it" → also confirm neighbors green and the three acceptance items.

## Real-World Impact

Battle-tested on a ~17k-file monorepo: 5 audit waves (25 subsystems) →
backlog with dedup → 17 merged-ready PRs (4 closed existing issues, rest novel),
each with RED→GREEN evidence and neighbor receipts. Validation basis for this skill.
