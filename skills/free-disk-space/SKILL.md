---
name: free-disk-space
description: Reclaim disk space on macOS by auditing and clearing build artifacts, package-manager caches, stale git worktrees, and offloading large media to an external drive. Use whenever the user complains about low storage, a full disk, "out of space" errors, or asks to clean up, offload, or free up their machine — even if they don't name specific folders.
---

# Free Disk Space (macOS)

Audit the disk, clear what's safely regenerable, and confirm before touching anything that isn't.

## Ground rules

- Use `trash` (if the CLI is available) instead of `rm -rf`. Accumulate in Trash; ask the user to empty it once at the end.
- Prefer a tool's own cleanup command over deleting its directories (`go clean -cache`, `brew cleanup --prune=all`, `pnpm store prune`, `npm cache clean --force`, `cargo clean`).
- Measure with `df -h /System/Volumes/Data` (not plain `df /`).
- **Do not clear GUI app data.** `~/Library/Caches/<App>` and `~/Library/Application Support/<App>` often hold logins, cookies, and history. Leave them alone unless the user names a specific app and accepts the risk. Prefer build cleanup, package-manager prune, stale worktrees, and external-drive offload instead.

## Workflow

### 1. Assess

`df -h /System/Volumes/Data; ls /Volumes`. Note free space and whether an external drive is mounted.

### 2. Survey

Targeted `du -sh ... | sort -hr` — not a full scan of `~`. Usual big ones:

- **Build artifacts**: `target/` (every worktree too), `node_modules/`, Xcode DerivedData
- **Package-manager caches**: go-build, Homebrew, pnpm/npm/pip stores — clean via CLI
- **Large media**: CapCut, Screen Studio, Movies, Downloads
- Report app caches / Application Support sizes only; don't auto-delete them

### 3. Clear safe tier without asking

Build artifacts and package-manager / language-tool caches only. Everything else (app data, media, model weights, toolchains): summarize sizes and ask, or propose offload.

### 4. Audit git worktrees

For each worktree: dirty? unique/unpushed commits? Clean + fully merged/pushed → `git worktree remove` and `git branch -d` (never `-D`). Keep and report anything dirty or ahead.

### 5. Offload keepers

`rsync -a` → verify sizes → trash original → symlink old path. Symlink individual folders, not whole special dirs like `~/Movies`. Warn the app needs the drive mounted.

### 6. Wrap up

What freed immediately, what's in Trash, what you left alone. Ask user to empty Trash and re-check `df`.
