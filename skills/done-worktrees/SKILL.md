---
name: done-worktrees
description: Use this skill when the human has decide we're done working on the feature, and we're ready to merge into main.
---

Review remaining uncommitted code. If all changes are related and can fit into an atomic commit, create a single commit. If work should be broken into multiple atomic commits, do so and commit all.

If we're on a worktree, merge back into the main/master. Then, clean up the worktree.

If work on this feature was tied to a GitHub issue, close that issue as completed.
