---
name: create-pr
description: Create an Azure DevOps pull request using the ado MCP server. Use when the user runs /create-pr, asks to open a PR, or wants to create a pull request for the current branch with a concise auto-generated description.
---

# Create Pull Request (Azure DevOps)

Create a PR from the current branch via the **ado** MCP server (`repo_pull_request_write`). Ask for the target branch when missing, draft a short useful description from local git context, confirm, then create.

## Prerequisites

- **ado MCP** configured and connected (`/mcp` shows `ado` as connected). See [mcp.json.example](../mcp.json.example).
- Local git repo with an Azure DevOps remote and the source branch **pushed** to origin.
- `ADO_PAT` set with permissions to create pull requests.

If the ado MCP is unavailable, stop and tell the user to configure it first — do not fall back to REST APIs or `gh`.

## Git safety

- NEVER update git config
- NEVER force-push
- Do not create a PR for uncommitted work without telling the user — mention unpushed or uncommitted changes
- Do not include secrets, tokens, or `.env` contents in the PR description

## Workflow

### 1. Gather local context (run in parallel)

```bash
git branch --show-current
git status -sb
git remote -v
git log -10 --oneline
```

If the user named a target branch, also run:

```bash
git log --oneline <target>..HEAD
git diff --stat <target>...HEAD
```

Parse the Azure DevOps remote URL to extract **organization**, **project**, and **repository** when possible:

| Remote format | Example |
|---------------|---------|
| HTTPS | `https://dev.azure.com/{org}/{project}/_git/{repo}` |
| SSH | `git@ssh.dev.azure.com:v3/{org}/{project}/{repo}` |

Strip a `refs/heads/` prefix from branch names when comparing locally; ADO APIs need `refs/heads/{branch}`.

### 2. Resolve target branch

| Situation | Action |
|-----------|--------|
| User specified target (e.g. "PR into `develop`") | Use it |
| Default branch obvious from `origin/HEAD` or repo convention | Suggest it, still confirm |
| Unclear | **Ask the user** which branch to merge into |

Use **AskQuestion** when available with likely options (`main`, `master`, `develop`, or branches from ado `repo_branch` `list` if helpful). Do not guess when multiple long-lived branches exist.

Normalize to `refs/heads/{branch}` for MCP calls.

### 3. Resolve repository and check for existing PR

Use ado MCP:

1. **`repo_repository`** `get` or `list` — confirm `repositoryId` (name or GUID) and `project` if not parsed from remote.
2. **`repo_pull_request`** `list` — filter by `sourceRefName` = current branch and `status` = `Active`. If one exists, report its ID and URL and **stop** unless the user asked to create another.

### 4. Draft title and description

Keep the description **short and scannable** (aim for under ~30 lines). No walls of text.

**Title** — pick the best fit:

- Single clear commit → use its conventional subject
- Multiple commits → summarize the theme, e.g. `feat: add user authentication flow`
- Fallback → branch name turned into a readable title (`feature/add-auth` → `Add auth`)

**Description template:**

```markdown
## Summary

- <what changed and why, 1–2 bullets>
- <user-visible or reviewer-relevant impact, if any>

## Changes

- <area/file group>: <brief note>
- <area/file group>: <brief note>

## Commits

- `<short-hash>` <subject>
- `<short-hash>` <subject>

## Notes

- <unpushed commits, breaking changes, follow-ups, or "none">
```

Rules:

- **Summary**: 1–3 bullets max; explain *why*, not every file touched.
- **Changes**: 3–6 bullets from `git diff --stat` / commit messages; group by concern.
- **Commits**: list commits on `target..HEAD` (cap at 10; say "+ N more" if needed).
- **Notes**: call out draft/WIP, missing tests, or required reviewer focus; omit section if empty.
- Skip boilerplate ("This PR…", "Please review…").
- Do not paste full diffs.

Show the draft title, target branch, and description to the user. **Ask for confirmation** before creating unless they explicitly said to create without confirming.

### 5. Create the PR (ado MCP)

Call **`repo_pull_request_write`** with:

```json
{
  "action": "create",
  "repositoryId": "<name-or-guid>",
  "project": "<project>",
  "sourceRefName": "refs/heads/<current-branch>",
  "targetRefName": "refs/heads/<target-branch>",
  "title": "<title>",
  "description": "<markdown description>",
  "isDraft": false
}
```

Set `isDraft: true` only when the user asked for a draft PR.

Optional: if the user provided work item IDs, set `workItems` (space-separated).

### 6. Report result

Return:

1. PR ID, title, and web URL (from MCP response)
2. Source → target branches
3. Draft vs active
4. Anything left for the user (push branch, add reviewers, link work items)

---

## Examples

**User:** `/create-pr`

→ Gather git context, ask "Which branch should this merge into?", draft description, confirm, create.

**User:** `/create-pr into develop`

→ Use `develop` as target, skip branch question, draft, confirm, create.

**User:** `create a draft PR to main`

→ Target `main`, `isDraft: true`, draft description, confirm, create.

---

## Troubleshooting

| Problem | Action |
|---------|--------|
| Branch not on remote | Tell user to push first (`git push -u origin HEAD`) |
| ado MCP not connected | Point to MCP setup in README / `mcp.json.example` |
| Active PR already exists | Show existing PR link; do not duplicate |
| Cannot parse remote | Ask user for project and repository name |
| Empty `target..HEAD` | Warn that branches may be identical; confirm before creating |
