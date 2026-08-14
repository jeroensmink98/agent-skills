---
name: commit-and-push
description: Group uncommitted changes by concern, create conventional commits, and push. Use when the user runs /commit-and-push, asks to commit and push, or wants changes committed with minimal-effort logical grouping.
---

# Commit and Push

End-to-end workflow: inspect changes → group by concern → conventional commit per group → push.

## Git safety (non-negotiable)

- NEVER update git config
- NEVER skip hooks (`--no-verify`, etc.) unless the user explicitly requests it
- NEVER force-push to `main`/`master`; warn if push would require it
- NEVER commit likely secrets (`.env`, credentials, tokens)
- NEVER use interactive git (`-i` flags)
- Avoid `git commit --amend` unless the user explicitly asked and HEAD is unpushed
- Only push when this skill is invoked (push is part of the workflow here)

## Workflow

### 1. Inspect (run in parallel)

```bash
git status
git diff          # unstaged
git diff --cached # staged
git log -10 --oneline
```

Also check whether the branch tracks a remote (`git status -sb`).

### 2. Partition into commit groups (minimal effort)

Split only where concerns clearly differ. Prefer **fewer, coherent commits** over many tiny ones.

**Default grouping signals** (first match wins; merge small same-type groups):

| Signal | Group together |
|--------|----------------|
| Same directory tree | `.github/`, `scripts/`, `infra/`, `docs/` |
| Same change kind | All doc-only edits → one `docs:` commit |
| Same feature/fix | Files that implement one bugfix or one feature |
| Config/meta | `AGENTS.md`, `.cursor/`, root README when touched for the same housekeeping pass |

**Do not commit** (leave out and mention to user):

- Personal scratch files (`notities.md`, `notes.md`, `.todo`) unless user included them
- Untracked build artifacts, `.env`, credentials

**Do not blend** unrelated types in one subject (e.g. pipeline code + unrelated script in one commit).

### 3. Plan (brief)

Before committing, show a short plan:

```
Commit 1 — docs: …
  - docs/README.md
  - AGENTS.md

Commit 2 — ci: …
  - .github/workflows/ci.yml
```

Then execute without waiting for confirmation unless the user said to ask first.

### 4. Commit each group

For each group, in order (docs/chore before feat/fix when order matters):

```bash
git add -- <paths>
git commit -m "$(cat <<'EOF'
<type>[optional scope]: <imperative subject>

<optional body — why, not what>

EOF
)"
```

After each commit, verify with `git status`. If a hook fails, fix and make a **new** commit — never amend a failed commit.

### 5. Push

```bash
git push -u origin HEAD   # first push of branch
# or
git push                  # when upstream already set
```

If push is rejected (non-fast-forward), report and ask how to proceed — do not force-push.

---

## Conventional commit format

```
<type>[optional scope]: <description>

[optional body]
```

| Part | Rules |
|------|-------|
| **type** | `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert` |
| **scope** | Optional — ticket (`ABC-123`) or area (`api`, `scripts`) |
| **description** | Imperative, lowercase start, no trailing period, ~72 chars |
| **body** | Optional; explain *why* when not obvious |

**Type quick pick:** `docs` documentation · `ci` pipelines/workflows · `feat` new behavior · `fix` bug · `chore` housekeeping · `refactor` restructure without behavior change

### Examples

```
docs: clarify setup instructions in README
ci: add lint workflow for pull requests
feat(auth): add JWT-based login endpoint
chore: add utility script for local dev setup
```

### Anti-patterns

- ❌ `Fixed stuff` / `WIP` / past tense
- ❌ `feat: Add thing.` — capitalized or period at end
- ❌ One commit mixing unrelated areas (e.g. CI config + personal notes)

---

## Grouping examples

**Three unrelated areas → three commits:**
- `scripts/setup.sh` + `scripts/README.md` → `feat: add local setup script`
- `docs/README.md` only → `docs: update project README`
- `AGENTS.md` housekeeping → `docs: update AGENTS.md`

**Same feature across files → one commit:**
- `.github/workflows/ci.yml` + `docs/ci.md` for the same change → single `ci:` or `feat:` commit

**Single typo in one file → one commit, don't over-split.**

---

## Output after completion

Report:

1. Each commit hash + subject
2. Branch name and push result (remote updated / already up to date)
3. Any files left uncommitted and why
