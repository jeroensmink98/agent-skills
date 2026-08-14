---
name: create-feature-branch
description: Create and check out a new feature branch from the latest default branch. Use when the user asks to create a feature branch, start a new branch, branch off main/master, or begin work on a new feature or fix in isolation.
---

# Create Feature Branch

Create a new branch from an up-to-date base, with a clear name and upstream ready for work.

## Git safety (non-negotiable)

- NEVER update git config
- NEVER force-push
- NEVER use interactive git (`-i` flags)
- NEVER discard uncommitted work without explicit user approval
- Do not commit or push unless the user asked for that separately

## Workflow

### 1. Inspect (run in parallel)

```bash
git status -sb
git branch -a
git remote -v
```

Note:

- Current branch and uncommitted changes
- Default branch name (`main`, `master`, or `develop` — prefer what `origin/HEAD` points to)
- Whether the working tree is clean

### 2. Handle uncommitted changes

| State | Action |
|-------|--------|
| Clean working tree | Continue |
| User asked to include WIP on new branch | Stash is **not** needed — create branch with changes in place |
| Uncommitted changes and user did not mention them | Stop and ask: stash, commit, or discard before branching |
| Only untracked files user likely wants | Mention them; proceed if harmless |

### 3. Determine branch name

Use the name the user gave. If none, derive from context:

```
feature/<short-kebab-description>   # new functionality
fix/<short-kebab-description>       # bug fixes
chore/<short-kebab-description>     # tooling, deps, housekeeping
```

Rules:

- Lowercase, kebab-case, no spaces
- Keep under ~50 characters
- No leading/trailing hyphens

Examples: `feature/add-user-auth`, `fix/login-redirect-loop`, `chore/upgrade-deps`

### 4. Update base branch and create new branch

Replace `<base>` with the default branch (usually `main` or `master`):

```bash
git fetch origin
git checkout <base>
git pull --ff-only origin <base>
git checkout -b <new-branch-name>
```

If `git pull --ff-only` fails (local base has diverged), report the situation and ask how to proceed — do not rebase or reset without explicit approval.

### 5. Optional — push and set upstream

Only when the user asked to push the new branch or set upstream:

```bash
git push -u origin HEAD
```

---

## Branch-from-current (alternative)

When the user explicitly wants to branch from the **current** branch (not default):

```bash
git fetch origin
git checkout -b <new-branch-name>
```

Mention that the new branch includes whatever commits are on the current branch.

---

## Output after completion

Report:

1. Base branch used and whether it was updated
2. New branch name and current `git status -sb`
3. Uncommitted or stashed changes, if any
4. Whether upstream was set (and remote URL if pushed)
