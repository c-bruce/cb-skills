---
name: worktree
description: Set up or remove a git worktree for a task. Use when the user asks to start an issue, work on a PR, or review a PR in a worktree or in parallel, or says a worktree is finished.
---

# Worktree

Each task gets its own worktree, opened in its own editor window when the config names an editor. The
current session stays where it is; the user starts a new Claude Code session in the worktree.

Run every command from the main checkout: the first path in `git worktree list`. Run commands that belong
inside the new worktree in a subshell, `(cd <worktree> && <command>)`, so this session stays in the main
checkout.

## Config

Read both config files before you create or remove a worktree. Each is optional.

- `~/.claude/worktree.md`: the user's preferences, for every repo.
- `.claude/worktree.md` in the main checkout: this repo's setup.

[CONFIG.md](CONFIG.md) defines every key, its default, and how the two files combine. Follow the free-text
body of each file as well as its keys.

## Tasks

| Task | Worktree | Checks out |
|---|---|---|
| Start an issue | `<issue>-<slug>` | new branch `<branch_prefix><issue>-<slug>` from `base` |
| Work on the user's own PR | `pr-<number>` | the PR's head branch |
| Review a PR | `review-<number>` | the PR head, detached |

Find the task from the number, not the user's wording: GitHub issues and PRs share one number sequence.
`gh pr view <number> --json author,headRefName` succeeds only for a PR. A PR is the user's own when its
author matches `gh api user -q .login`. When the user asks to work on someone else's PR without saying
review, ask which they mean.

`<slug>` is 2-4 kebab-case words from the issue title (`gh issue view <issue> --json title`). If a branch
matching `<branch_prefix><issue>-*` already exists, use it and its slug.

## Create

1. `git fetch origin`
2. Create the worktree under `worktree_dir`. If it already exists, go to step 6.
   - Issue: `git worktree add --no-track -b <branch> <worktree_dir>/<name> <base>`, or
     `git worktree add <worktree_dir>/<name> <branch>` for an existing branch. `--no-track` keeps the
     branch from tracking `base`, so its first push needs `git push -u origin HEAD`.
   - Own PR: `git worktree add <worktree_dir>/pr-<number> <branch>`. With no local branch, use
     `git worktree add --track -b <branch> <worktree_dir>/pr-<number> origin/<branch>`.
   - Review: `git fetch origin pull/<number>/head`, then
     `git worktree add --detach <worktree_dir>/review-<number> FETCH_HEAD`.

   A branch lives in one worktree at a time. If git reports the branch is checked out elsewhere, stop and
   tell the user which worktree holds it.

   When `worktree_dir` is inside the repo and `git check-ignore` does not match it, add it to
   `.git/info/exclude`.
3. Copy the `copy` files from the main checkout into the worktree at the same relative paths.
4. Run `install` in the worktree. Done when it exits 0.
5. If `ports` is set, take a slot and write the port file, as [CONFIG.md](CONFIG.md#ports) describes.
6. If `open` is set, run it.
7. Report:
   - the worktree path
   - the branch, or the commit for a review
   - the files copied in step 3
   - the ports and the start command from step 5
   - every `warn_if_changed` message whose glob matches a file in `git diff --name-only <base>...HEAD`
   - for a review, the PR's title and author

## Remove

`git worktree remove <worktree_dir>/<name>`. If git refuses because of uncommitted or untracked changes,
show them to the user, say whether the branch matches its remote, and let the user decide whether to force
it. Delete the branch only when the user asks. Removing the worktree frees its port slot.
