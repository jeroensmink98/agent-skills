# Agent Skills

Reusable [Cursor Agent Skills](https://cursor.com/docs/context/skills) for common git workflows.

## Skills

| Skill | Use when |
|-------|----------|
| [commit-and-push](commit-and-push/SKILL.md) | Group changes by concern, commit with conventional messages, and push |
| [create-feature-branch](create-feature-branch/SKILL.md) | Start a new feature, fix, or chore branch from an updated base |
| [fetch-and-merge](fetch-and-merge/SKILL.md) | Fetch from remote and merge upstream into the current branch |

## Installation

Copy or symlink skill directories into your Cursor skills folder:

```bash
# Personal skills (all projects)
ln -s "$(pwd)/commit-and-push" ~/.cursor/skills/commit-and-push
ln -s "$(pwd)/create-feature-branch" ~/.cursor/skills/create-feature-branch
ln -s "$(pwd)/fetch-and-merge" ~/.cursor/skills/fetch-and-merge
```

Or install per-project under `.cursor/skills/` in a repository.

## Usage

Invoke by name in chat, for example:

- "Use commit-and-push to commit my changes"
- "Create a feature branch for user authentication"
- "Fetch and merge remote into my current branch"
