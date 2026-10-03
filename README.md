# Pit-Stop Issue Skill — Audit Subsystems to PRs

[中文文档](README.zh-CN.md) | English

Agent skill: audit a large repository piece by piece and turn the findings into fixes. Subsystem audit in parallel read-only waves → triaged backlog → each kept finding lands as its own worktree → branch → RED test → minimal fix → verified commit → fork PR. Finding without a `path:line` anchor is not a finding; a PR without a failing-then-passing test is not a fix.

Use when asked to audit a large repository piece by piece and turn the findings into fixes, run a multi-PR contribution campaign against an upstream repo, or convert audit findings into separate worktrees/branches/pull-requests. Triggers: subsystem audit, audit in waves, fix every finding, contribute upstream, per-module review, backlog of fixes to PRs.

## How it works

Six stages, in order. The skill file (`SKILL.md`) is the contract; details live in `references/`.

1. **Slice** — partition the tree into audit units: one top-level dir or one giant module each.
2. **Sweep** — per wave, dispatch N parallel read-only auditor subagents, each on ONE unit, FIND-ONLY (Load → Find → Propose, then STOP). Every finding carries `path:line` + tag + strength + symptom + evidence + impact + minimal fix + cost.
3. **Triage** — keep only defensible defects; drop style/speculation. Then **dedup every survivor against upstream open issues/PRs**.
4. **Freeze** — write the kept items to a backlog file (one line each: anchor, defect, lane). This file is the wave's contract.
5. **Fix** — one item at a time: fresh worktree + branch → RED test proving the bug → minimal fix → GREEN → neighbor tests → push fork → cross-fork PR.
6. **Accept** — per PR, verify three things: PR is OPEN, `head.sha == local branch sha`, worktree clean.

## Highlights

- **Find-only means find-only**: the audit agent must not enter Fix (no edits, tests, installs, network).
- **Dedup before fixing**: a finding already filed upstream becomes a `Closes #N` fix; a finding with an open fixing PR is hands-off.
- **Test-heavy work serializes**: test/build runs go through one serial lock; scan in parallel, fix in series.
- **Pre-existing failures stay out**: a failure also present on the pristine base is environmental — document it, never fix it inside your PR.
- **Push/PR only on an explicit word**: changes wait in the worktree; nothing is pushed or opened until the human says so.

## Directory layout

```
pit-stop-issue-skill/
├── SKILL.md                  # skill contract (frontmatter + six stages + iron laws)
└── references/
    ├── audit-waves.md        # slice + sweep procedure
    ├── triage-dedup.md       # triage + upstream dedup procedure
    └── fix-and-pr.md         # fix + accept procedure
```

## Quick start

Drop this directory where your agent loads skills from (e.g. alongside your other `SKILL.md` skills), then invoke it with one of its triggers:

```
audit this repo subsystem by subsystem / run a multi-PR contribution campaign
```

When NOT to use: single-file fixes (edit directly), read-only questions, repos you own (no fork dance needed), emergencies (fix first, audit later).

## Real-world impact

Battle-tested on a ~17k-file monorepo: 5 audit waves (25 subsystems) → backlog with dedup → 17 merged-ready PRs (4 closed existing issues, rest novel), each with RED→GREEN evidence and neighbor receipts.

## Docs

| File | Purpose |
|---|---|
| `SKILL.md` | Full skill contract: stages, iron laws, red flags |
| `references/audit-waves.md` | Slice + sweep procedure |
| `references/triage-dedup.md` | Triage + upstream dedup procedure |
| `references/fix-and-pr.md` | Fix + accept procedure |

## License

MIT.
