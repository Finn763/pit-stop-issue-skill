# Fix and PR (detail for SKILL.md)

## 1. One item = one worktree + one branch

- Fresh worktree from the upstream tip: `<ops>/wt-<slug>`, branch
  `fix/<issue-or-slug>`. Never reuse the audit snapshot; never fix on `main`.
- Expected tree afterwards: **only** the files the fix owns.

## 2. RED first (TDD, no exceptions)

Write the regression test, run it, watch it fail **for the right reason**
(correct file/line in the traceback, expected error kind). Test lives where the
host puts it (`tests/<mirrored path>/`, same style as neighbors). ≤2 invariant
tests per fix; behavior contracts, never snapshots. A test that would hang
(e.g. an infinite loop) goes through a reproduction with a timeout instead.

## 3. Minimal fix, then GREEN + neighbors

Smallest change that kills the root cause; touch siblings the bug class demands.
Then: the new test green, the file's suite green, and the adjacent suites green.
Use the host's runner and its isolation model (per-file subprocess isolation if
it has one; otherwise expect pollution and rerun per file).

## 4. Pre-existing failures are not yours

Anything failing identically on the **pristine** upstream tip (a separate clean
worktree used only for baseline runs) is environmental/host-specific: document it
in the PR, do not fix it in. Typical shapes: POSIX-only syscalls/tests on a
Windows host, unquoted Windows paths through `shlex`, module symlinks resolving
to another worktree, timing/flake tests.

## 5. Push + open, only on the word

Commit with the host's message style (conventional prefix + English body). Push
the branch to **your fork** (pushing the upstream remote directly is expected to
fail). Open a **cross-fork PR** against upstream `main` with: why it matters,
what it does, how to test (command + RED→GREEN receipt), neighbor results, type
checklist. Keep it to the fix and its test.

## 6. Accept per PR (three things, checked not claimed)

`gh pr view` state is OPEN; the returned `headRefOid` equals the local branch
`rev-parse HEAD`; `git status --short` is empty. Anything else is not done.

## 7. Failure modes this stops

- Hanging regression test → subprocess script with a hard timeout.
- Test-file edits colliding between two of your own PRs → distinct new test
  file names per PR.
- Shell quoting breaking commit/PR text → message in a file (`commit -F`,
  `--body-file`), never long inline strings.
- Long test/subagent logs → write to `<ops>/prscan/` log files; tail, don't paste.
- Windows `git worktree add` path mangling, stale locks, `git stash` bans →
  follow the host playbook for worktrees, locks, and restores.
