---
name: done-worktrees
description: Finish a feature in a git worktree. Use when the user says work is done, asks to merge a worktree back to main/master, clean up a worktree, or close the related issue.
---

Review remaining uncommitted code. If all changes are related and can fit into an atomic commit, create a single commit. If work should be broken into multiple atomic commits, do so and commit all.

If we're on a worktree, merge back into the main/master. Then, clean up the worktree.

If work on this feature was tied to a GitHub issue, close that issue as completed.
