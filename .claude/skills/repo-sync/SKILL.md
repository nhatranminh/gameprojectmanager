---
name: repo-sync
description: Sync local working copy with the remote repository on request. Triggers on phrases like "sync", "sync repo", "sync with my repo", "đồng bộ repo", "update repo", "cập nhật repo". Pulls latest changes, summarizes what changed, and flags anything that needs the user's attention (conflicts, uncommitted work, docs that look stale).
---

# Repo Sync

Bring the local branch up to date with `origin` and report what changed, in a way that's safe by default and never discards work silently.

## When to trigger

Any message where the user asks to "sync", "sync repo", "update repo", "pull latest", "đồng bộ", "cập nhật repo" — with no other target specified. If the user names a different target (e.g. "sync with Notion"), this skill does not apply.

## Steps

1. **Check for local changes first.**
   Run `git status`. If there are uncommitted changes or untracked files:
   - Do NOT discard or stash them silently.
   - Tell the user what's uncommitted and ask whether to stash, commit, or leave as-is before continuing — unless it's obviously safe to proceed (e.g. only files unrelated to the sync).

2. **Fetch and pull.**
   - `git fetch origin <current-branch>`
   - Determine how many commits local is behind/ahead of `origin/<current-branch>`.
   - If behind and no local changes conflict: `git pull origin <current-branch>`.
   - If diverged (both ahead and behind): stop and ask the user how to reconcile (merge vs rebase) rather than picking for them.

3. **Summarize what changed.**
   After a successful pull, produce a short summary:
   - New commits (author, one-line message) since the last known state.
   - Files touched, grouped by area if the diff is large.
   - Anything notable: new dependencies, migration files, config changes, new scripts.

4. **Flag stale docs (best-effort, don't auto-edit).**
   If the pulled changes touch things typically documented in `CLAUDE.md` / `README.md` (new commands, new top-level directories, changed build/test steps), mention it and ask if the user wants those docs updated. Do not edit docs automatically unless asked.

5. **Report, don't over-explain.**
   Keep the final report to a few lines: commits behind → now up to date, what changed, anything needing attention. No need to restate every git command run.

## Explicitly out of scope

- Pushing local commits (only pull/sync inbound).
- Force operations (`reset --hard`, `push --force`) — never use these as part of this skill.
- Editing documentation files without explicit confirmation.
- Syncing with non-git systems (Notion, issue trackers, etc.) — that's a separate skill if needed.
