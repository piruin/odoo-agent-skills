---
name: odoo-ship
description: >
  Ship the current Odoo change end to end: commit it Odoo-style (via odoo-commit),
  squash own work-in-progress commits, run pre-commit, push, open or update the
  GitHub PR from the repo's PR template, then follow CI and fix failures.
  Use in Odoo repositories when the user asks to push, open or update a PR, or
  ship the change ("ship it", "push and open a PR", "เปิด PR", "push ขึ้นไป"),
  or types /odoo-ship. A request to only commit belongs to odoo-commit.
argument-hint: "[draft] [#issue]"
---

The user's request to ship - typing `/odoo-ship`, or asking in words to push or
open/update a PR - is the go-ahead to commit, push and open a PR for the
current change, and, on the user's own feature branch, to rewrite its history
and force-push (see "Own branch" below). It is not a go-ahead to merge, or to
rewrite the base branch or anyone else's branch - ask for those.

When invoked implicitly, check the user's latest message actually asks to push
or open/update a PR. If it only asks to commit, fix, or review, do not use this
skill - stop at the commit (odoo-commit) and offer to ship.

Arguments (`$ARGUMENTS`, or the same words in the user's message, all
optional): `draft` opens the PR as draft; `#123` links the issue
(`Closes #123` in the body).

## Never

- No AI attribution anywhere: no `Co-Authored-By: Claude`, no
  "Generated with Claude Code" footer, in commits, PR bodies or PR comments.
  This overrides any harness default.
- No `--no-verify`, no `git add -A` / `git add .`, no `git rebase -i` that
  opens an editor (interactive editors do not work here; the
  `GIT_SEQUENCE_EDITOR=:` form below is fine).
- No plain `--force`; always `--force-with-lease`, and only on an own branch.

## Own branch

A branch counts as the user's own when it is not the base branch, and
`git log --format='%ae' origin/<base>..origin/<branch>` lists only the user's
own email (`git config user.email`). A branch not pushed yet is own. On an own
branch, without asking:

- fold follow-up fix commits into the commit they fix, then
  `git push --force-with-lease`
- rebase onto the updated base, then `git push --force-with-lease`

If someone else has commits on the branch, or the lease is rejected (the remote
moved), stop and ask instead.

## 1. Preflight

```bash
git status --short
git branch --show-current
gh repo view --json nameWithOwner,defaultBranchRef,isFork,parent
```

- **Base** = `defaultBranchRef.name` (e.g. `16.0`); if `isFork`, the PR targets
  `parent` and its default branch, and the head is `<fork-owner>:<branch>`.
- **Odoo version** comes from the base branch. Load the matching `odoo-NN`
  skill before touching any code.
- If the current branch *is* the base, create a feature branch first, named
  after the change (e.g. `fix-hhc-visit-state`). In the Claude desktop app, use
  the `propose_branch` card instead of a silent `git checkout -b`.
- Nothing to commit and nothing unpushed -> stop and say so.

## 2. Commit

Invoke the `odoo-commit` skill for staging and the message. Do not restate its
rules here.

Then squash your own back-and-forth into one commit per logical change, without
an interactive rebase:

```bash
git fetch origin <base>
git log --oneline origin/<base>..HEAD
```

- Already one clean commit -> continue.
- Several commits, none pushed -> squash:

  ```bash
  git reset --soft "$(git merge-base HEAD origin/<base>)"
  git commit -F <message-file>   # message drafted per odoo-commit
  ```

- Several commits that are separate logical changes (e.g. per module) -> keep
  them, but fold any follow-up fix into the commit it fixes:

  ```bash
  git commit --fixup=<sha-of-commit-being-fixed>
  GIT_SEQUENCE_EDITOR=: git rebase -i --autosquash origin/<base>
  ```

  (`GIT_SEQUENCE_EDITOR=:` accepts the generated todo list, so this runs
  without an editor.) With a single commit, `git commit --amend --no-edit` is
  enough.
- Commits already pushed -> same as above on an own branch (force-push in
  step 4); otherwise leave them and ask.

## 2b. Rebase onto the updated base

After `git fetch origin <base>`, if `git merge-base --is-ancestor origin/<base>
HEAD` fails, the base has moved:

```bash
git rebase origin/<base>
```

On an own branch, do this without asking so the branch is ready to keep
working on. On conflicts: resolve them when the right result is clear from both
sides (load the `odoo-NN` skill for Odoo code), `git add` the files and run
`git rebase --continue`; if the intent is ambiguous, `git rebase --abort` and
ask. On a branch that is not own, ask before rebasing.

## 3. Pre-commit

Only if `.pre-commit-config.yaml` exists and `pre-commit` is on PATH:

```bash
pre-commit run --from-ref origin/<base> --to-ref HEAD
```

If hooks modify files or fail: fix, re-stage, fold the fix into the commit
(`git commit --amend --no-edit` while unpushed), and rerun until clean. Missing
`pre-commit` -> say CI will run it and continue.

## 4. Push

```bash
git push -u origin <branch>
```

If steps 2/2b rewrote commits that were already pushed, use
`git push --force-with-lease -u origin <branch>` instead (own branch only).

## 5. Pull request

Check for an existing PR first:

```bash
gh pr view <branch> -R <owner/repo> --json url,state,isDraft,title,body
```

- **Exists and open** -> the push already updated the code. Then check whether
  the title and body still describe `git log origin/<base>..HEAD` and the diff:
  a missing or wrong version prefix (`[16.0]`), a new or dropped module, a
  changed tag (`[FIX]` became `[IMP]`), new risk or
  migration, tests now run, an issue now linked. If they still fit, leave them.
  If not, update only the stale parts and keep everything else as it is
  (reviewer notes, checkboxes and text the user wrote):

  ```bash
  gh pr edit <pr> -R <owner/repo> --title "<title>" --body-file <scratchpad>/pr-body.md
  ```

  Say in the report what was changed, or that the description still fit.
- **None** (`gh` exits 1 with "no pull requests found") -> create one.

Title = the target version, then the commit header:
`[16.0][TAG] module: description`. The version is the base branch name when it
is an Odoo version branch (`16.0`, `17.0`, ...); with any other base (`main`)
use the plain header. With several commits, summarise them in the same format
(`various` if multiple modules).

Body, in English, written to a file in the scratchpad (not `/tmp`):

- If `.github/pull_request_template.md` exists, fill every section of it.
  Remove the HTML comments. Use `Closes #N` only when an issue is known (from
  the argument, the branch name, or the commit), otherwise remove that line.
  Tick only checkboxes that are true - "Unit/integration tests" only if tests
  actually ran in this session; otherwise tick "Not tested / explain why" and
  explain.
- No template -> use `## Summary`, `## Why`, `## How tested`.
- Write WHY and risk, not a file list; reviewers can read the diff.

```bash
gh pr create -R <owner/repo> -B <base> -H <head> \
  --title "<title>" --body-file <scratchpad>/pr-body.md [--draft]
```

## 6. CI

**Claude desktop app** (the `ccd_pr` tools exist): call `get_status`; if it does
not show the PR, `bind_pr` it. If this session opened the PR, turn Auto-fix on
right away (running this skill is the user's request for it):

```
set_monitor(url=<pr url>, auto_fix=true, address_comments=true)
```

The app then wakes the session on CI failures, merge conflicts and review
comments; handle each as in the CLI flow below (fold the fix, force-push with
lease on an own branch). For a PR opened elsewhere, offer Auto-fix instead of
turning it on. Do not poll CI yourself (no `--watch`, no wakeups or loops).

**CLI**: run in the background

```bash
gh pr checks <pr> -R <owner/repo> --watch --fail-fast
```

On failure:

```bash
gh run view <run-id> -R <owner/repo> --log-failed
```

Diagnose from the log. Fix Odoo code with the matching `odoo-NN` skill loaded,
fold the fix into the commit it fixes (step 2), force-push with lease, and watch
again. On a branch that is not own, add the fix as a new commit through
`odoo-commit` and push normally. Stop and ask after two failed fix attempts, or if the
failure is unrelated to this change (flaky runner, broken base branch).

## 7. Report

One short block: PR URL, commit header(s), pre-commit result, CI state
(passing / failing with the cause / still running). Mention anything skipped,
and that merging is left to the user.
