# Agent Skills

Reusable [Agent Skills](https://agentskills.io) for common git workflows. Each skill is a folder with a `SKILL.md` file — the same format works in [Cursor](https://cursor.com/docs/context/skills), [Claude Code](https://code.claude.com/docs/en/custom-skills), and other tools that adopt the open standard.

## Skills

| Skill | Use when |
|-------|----------|
| [commit-and-push](commit-and-push/SKILL.md) | Group changes by concern, commit with conventional messages, and push |
| [create-feature-branch](create-feature-branch/SKILL.md) | Start a new feature, fix, or chore branch from an updated base |
| [fetch-and-merge](fetch-and-merge/SKILL.md) | Fetch from remote and merge upstream into the current branch |
| [create-pr](create-pr/SKILL.md) | Create an Azure DevOps pull request with a concise auto-generated description |

## Installation

### Claude Code (every project)

Per [Claude's skills documentation](https://support.claude.com/en/articles/12512176-what-are-skills), skills are folders of instructions that Claude loads when relevant. In [Claude Code](https://code.claude.com/docs/en/custom-skills), **personal skills** live in `~/.claude/skills/` and apply to **all projects** on your machine.

1. Clone this repository:

```bash
git clone https://github.com/jeroensmink98/agent-skills.git
cd agent-skills
```

2. Install each skill into your personal skills folder (symlink or copy):

```bash
mkdir -p ~/.claude/skills

ln -s "$(pwd)/commit-and-push" ~/.claude/skills/commit-and-push
ln -s "$(pwd)/create-feature-branch" ~/.claude/skills/create-feature-branch
ln -s "$(pwd)/fetch-and-merge" ~/.claude/skills/fetch-and-merge
ln -s "$(pwd)/create-pr" ~/.claude/skills/create-pr
```

To copy instead of symlink:

```bash
cp -R commit-and-push create-feature-branch fetch-and-merge create-pr ~/.claude/skills/
```

3. Start Claude Code in any git repository (`claude`). Skills are picked up automatically. If you install while a session is already open, start a new session or wait for Claude Code to detect the new files under `~/.claude/skills/`.

4. Verify — invoke directly or ask in natural language:

```
/commit-and-push
/create-feature-branch
/fetch-and-merge
/create-pr
```

The folder name becomes the slash command; the `description` in each `SKILL.md` helps Claude auto-load the skill when your request matches.

**Project-only install** (shared with teammates via git): place skills under `.claude/skills/` in a repository instead. See [Where skills live](https://code.claude.com/docs/en/custom-skills#where-skills-live) in the Claude Code docs.

### Cursor (every project)

Install into your personal Cursor skills folder:

```bash
git clone https://github.com/jeroensmink98/agent-skills.git
cd agent-skills

mkdir -p ~/.cursor/skills

ln -s "$(pwd)/commit-and-push" ~/.cursor/skills/commit-and-push
ln -s "$(pwd)/create-feature-branch" ~/.cursor/skills/create-feature-branch
ln -s "$(pwd)/fetch-and-merge" ~/.cursor/skills/fetch-and-merge
ln -s "$(pwd)/create-pr" ~/.cursor/skills/create-pr
```

Or install per-project under `.cursor/skills/` in a repository.

### Windows

The skill files are the same on every OS. Only the personal skills folder path and install commands differ.

| Tool | Personal skills folder |
|------|------------------------|
| Claude Code | `%USERPROFILE%\.claude\skills\` (e.g. `C:\Users\You\.claude\skills\`) |
| Cursor | `%USERPROFILE%\.cursor\skills\` |

**Recommended — copy (PowerShell, no admin required):**

```powershell
git clone https://github.com/jeroensmink98/agent-skills.git
cd agent-skills

New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
Copy-Item -Recurse commit-and-push, create-feature-branch, fetch-and-merge, create-pr "$env:USERPROFILE\.claude\skills\"

New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.cursor\skills"
Copy-Item -Recurse commit-and-push, create-feature-branch, fetch-and-merge, create-pr "$env:USERPROFILE\.cursor\skills\"
```

Copy only the block for the tool you use (Claude Code, Cursor, or both).

**Git Bash** (Git for Windows) — same commands as macOS/Linux:

```bash
git clone https://github.com/jeroensmink98/agent-skills.git
cd agent-skills

mkdir -p ~/.claude/skills
cp -R commit-and-push create-feature-branch fetch-and-merge create-pr ~/.claude/skills/
```

**Symlinks (optional)** — use if you want `git pull` in the clone to update installed skills automatically. On Windows this requires [Developer Mode](https://learn.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development) or an elevated terminal:

```powershell
# PowerShell — run from the cloned agent-skills directory
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\commit-and-push" -Target "$(Get-Location)\commit-and-push"
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\create-feature-branch" -Target "$(Get-Location)\create-feature-branch"
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\fetch-and-merge" -Target "$(Get-Location)\fetch-and-merge"
New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\skills\create-pr" -Target "$(Get-Location)\create-pr"
```

Usage is identical: run `claude` or open Cursor in any repo, then `/commit-and-push` or ask in natural language.

## MCP servers (Claude Code)

This repo includes [`mcp.json.example`](mcp.json.example) with two MCP servers:

| Server | Purpose | Requires |
|--------|---------|----------|
| `fetch` | Fetch web pages for Claude | [uv](https://docs.astral.sh/uv/) (`uvx` on PATH) |
| `ado` | Azure DevOps (repos, pipelines, work items) — **required for `/create-pr`** | [Node.js](https://nodejs.org/) (`npx` on PATH), Azure DevOps PAT |

Claude Code reads MCP config from **`%USERPROFILE%\.claude.json`** (user scope, all projects) or **`.mcp.json`** in a project root (project scope, shareable via git). This is **not** `%USERPROFILE%\.claude\settings.json` and not Claude Desktop's `%APPDATA%\Claude\claude_desktop_config.json`.

### Windows (PowerShell)

**1. Prerequisites** — install Node.js and uv, then confirm they work:

```powershell
node --version
npx --version
uvx --version
```

**2. Set your Azure DevOps PAT** (replace placeholders; use a fine-scoped PAT with the access your org requires):

```powershell
# Current session only
$env:ADO_PAT = "your-azure-devops-pat"

# Persist for your Windows user account (recommended)
[System.Environment]::SetEnvironmentVariable("ADO_PAT", "your-azure-devops-pat", "User")
```

Restart the terminal after setting a persistent variable so `claude` inherits it.

**3. Customize the example** — copy `mcp.json.example` and replace:

- `YOUR_ORGANIZATION` — Azure DevOps org name (e.g. `contoso`)
- `YOUR_PROJECT` — default project name in `ado_mcp_project`

**4. Install — pick one scope**

*User scope (every project)* — merge the `mcpServers` block into your global config:

```powershell
# Open (or create) Claude Code's user MCP config
notepad $env:USERPROFILE\.claude.json
```

Paste the contents of `mcp.json.example` into the file. If `.claude.json` already exists, merge only the `mcpServers` entries — do not delete existing keys like `preferences`.

*Project scope (one repo, team-shared)* — in the project where you want these servers:

```powershell
Copy-Item .\mcp.json.example .\.mcp.json
notepad .\.mcp.json   # set YOUR_ORGANIZATION and YOUR_PROJECT
```

Commit `.mcp.json` if teammates should use the same server list (secrets stay in `ADO_PAT`, not in the file).

**5. Verify** — start Claude Code and check servers:

```powershell
claude
# then inside Claude Code:
/mcp
```

You should see `fetch` and `ado` connected. The first time a project-scoped `.mcp.json` is used, Claude Code prompts you to approve the servers.

### macOS / Linux

Same flow; paths are `~/.claude.json` (user) or `.mcp.json` (project). Set `ADO_PAT` in your shell profile:

```bash
export ADO_PAT="your-azure-devops-pat"
```

### Cursor users

Cursor uses a different config file and env syntax. In Cursor, set `ADO_TOKEN` to `"${env:ADO_PAT}"` instead of `"${ADO_PAT}"`, and add the servers under **Cursor Settings → MCP** or your project's `.cursor/mcp.json`.

## Usage

Invoke by name in chat, for example:

- `/commit-and-push` or "commit and push my changes"
- "Create a feature branch for user authentication"
- "Fetch and merge remote into my current branch"
- `/create-pr` or "open a PR into develop" (requires ado MCP)

## Learn more

- [What are skills?](https://support.claude.com/en/articles/12512176-what-are-skills) — Claude Help Center overview
- [Extend Claude with skills](https://code.claude.com/docs/en/custom-skills) — Claude Code setup, paths, and slash commands
- [Connect Claude Code to tools via MCP](https://code.claude.com/docs/en/mcp) — MCP scopes, `.mcp.json`, and env var expansion
- [Agent Skills open standard](https://agentskills.io) — portable skill format across AI tools
