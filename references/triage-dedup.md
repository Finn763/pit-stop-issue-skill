# Triage and Dedup (detail for SKILL.md)

## 1. Triage verdicts (with the human where judgment is needed)

- **Drop**: style/yagni/shrink/speculative; theoretical races; behavior call with
  no stated contract; anything whose impact can't be demonstrated.
- **Keep**: named-control defeats with a traced path, real crashes/hangs, silent
  data/control loss, protocol breaches — each with a minimal, testable fix.
- Medium items you keep anyway must carry their caveat into the backlog.

## 2. Dedup every survivor BEFORE fixing (non-negotiable)

For each kept finding, search upstream open issues AND open PRs by file/function
name and symptom (e.g. `gh search issues/prs --repo <upstream> --state open`).

- **REFERENCE lane** — an open issue describes the exact defect: fix it and use
  `Closes #N`. Highest acceptance, do these first.
- **NOVEL lane** — no match: new PR, no `Closes`. Re-verify occupancy right
  before pushing (branch-head check per the host's playbook).
- **OCCUPIED** — an open fixing PR exists, or a cluster of related issues/PRs
  owns the area: hands off. Do not race a live PR.
- **HOLD** — needs evidence (backend contract, schema, measurement) or a design
  decision: record the trigger that would reopen it, never a speculative fix.
- **DROP** — intentional design (existing test pins it; docs declare it), or a
  claim that inverts the code on re-read. Record one line saying why.

## 3. Freeze the backlog

One file, e.g. `<ops>/audit/BACKLOG.md`, with sections per lane. Each item:
anchor `path:line`, one-line defect, lane (+ `Closes #N` / occupant / caveat).
This file is the wave's contract: Fix reads from it, never from raw reports.
Append per wave; also append the delivered PR numbers and the holds with reasons.

## 4. Common triage traps

- An audit "Strong" that an open PR already fixes → OCCUPIED, not a trophy.
- Two similar items in one module → fix the traced one; HOLD the inferred one.
- An untestable-in-CI fix (missing optional dep, entangled JS workspace) →
  either prove it another way or HOLD it. Never ship on vibes.
