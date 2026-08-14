---
name: fetch-and-merge
description: Fetch from remote and merge the upstream tracking branch into the current local branch. Use when the user asks to fetch, sync with remote, pull latest changes, merge remote into local, or update the current branch from origin.
---

# Fetch and Merge

Fetch latest refs from remote, then merge the upstream tracking branch into the current local branch.

## Git safety (non-negotiable)

- NEVER update git config
- NEVER skip hooks unless the user explicitly requests it
- NEVER force-push or `git reset --hard` without explicit user approval
- NEVER use interactive git (`-i` flags)
- Prefer `--ff-only` when fast-forward is possible; use a merge commit when not
- Do not discard uncommitted work — stop if merge would overwrite local changes

## Workflow

### 1. Inspect (run in parallel)

```bash
git status -sb
git branch -vv
git remote -v
```

Identify:

- Current branch name
- Its upstream (e.g. `origin/feature/foo`)
- Uncommitted or staged changes
- Whether upstream exists

If there is no upstream, report and ask which remote branch to merge (or offer to set upstream).

### 2. Guard uncommitted changes

| State | Action |
|-------|--------|
| Clean working tree | Continue |
| Uncommitted changes | Stop and ask user to commit, stash, or discard before merging |
| Merge already in progress | Report conflict state; do not start a second merge |

### 3. Fetch

```bash
git fetch origin
```

Use a different remote name only when the repo uses something other than `origin`.

### 4. Merge remote into local

Let `<upstream>` be the remote tracking ref (e.g. `origin/main` or the branch's `@{upstream}`).

**Preferred — merge upstream of current branch:**

```bash
git merge --ff-only @{upstream}
```

If `--ff-only` fails (local branch has unique commits and remote has new commits):

1. Report that a fast-forward is not possible
2. Ask whether to create a merge commit or rebase (do not rebase without explicit approval)
3. If merge commit is OK (default when user said "merge"):

```bash
git merge @{upstream}
```

**Explicit remote branch** (when user named a branch or no upstream is set):

```bash
git merge --ff-only origin/<branch>
# or, if not fast-forwardable and user approved merge:
git merge origin/<branch>
```

### 5. Handle merge conflicts

If merge stops with conflicts:

1. Run `git status` and list conflicted files
2. Do **not** auto-resolve unless the user asked
3. Tell the user which files need resolution and that they can ask for help resolving
4. Do not commit the merge until conflicts are resolved

After user resolves conflicts:

```bash
git add -- <resolved-files>
git commit   # completes the merge (use default merge message)
```

### 6. Verify

```bash
git status -sb
git log -5 --oneline
```

---

## Fetch-only variant

When the user asked only to **fetch** (not merge):

```bash
git fetch origin
git status -sb
```

Report how many commits the local branch is ahead/behind upstream. Offer to merge if behind.

---

## Output after completion

Report:

1. Remote fetched and upstream ref merged
2. Fast-forward vs merge commit
3. Current ahead/behind status (`git status -sb`)
4. Any conflicts or reasons the merge was skipped
5. Suggested next step if blocked (stash, commit, resolve conflicts)
