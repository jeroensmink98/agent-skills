# Agent Skills

Reusable [Agent Skills](https://agentskills.io) for common git workflows. Each skill is a folder with a `SKILL.md` file — the same format works in [Cursor](https://cursor.com/docs/context/skills), [Claude Code](https://code.claude.com/docs/en/custom-skills), and other tools that adopt the open standard.

## Skills

| Skill | Use when |
|-------|----------|
| [commit-and-push](commit-and-push/SKILL.md) | Group changes by concern, commit with conventional messages, and push |
| [create-feature-branch](create-feature-branch/SKILL.md) | Start a new feature, fix, or chore branch from an updated base |
| [fetch-and-merge](fetch-and-merge/SKILL.md) | Fetch from remote and merge upstream into the current branch |

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
```

To copy instead of symlink:

```bash
cp -R commit-and-push create-feature-branch fetch-and-merge ~/.claude/skills/
```

3. Start Claude Code in any git repository (`claude`). Skills are picked up automatically. If you install while a session is already open, start a new session or wait for Claude Code to detect the new files under `~/.claude/skills/`.

4. Verify — invoke directly or ask in natural language:

```
/commit-and-push
/create-feature-branch
/fetch-and-merge
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
```

Or install per-project under `.cursor/skills/` in a repository.

## Usage

Invoke by name in chat, for example:

- `/commit-and-push` or "commit and push my changes"
- "Create a feature branch for user authentication"
- "Fetch and merge remote into my current branch"

## Learn more

- [What are skills?](https://support.claude.com/en/articles/12512176-what-are-skills) — Claude Help Center overview
- [Extend Claude with skills](https://code.claude.com/docs/en/custom-skills) — Claude Code setup, paths, and slash commands
- [Agent Skills open standard](https://agentskills.io) — portable skill format across AI tools
