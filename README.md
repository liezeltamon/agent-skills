# Agent skills

Reusable, platform-neutral skills for coding and research agents.

## Available skills

- `commit-local-changes`: inspect a complete Git working tree, propose atomic
  Conventional Commits, and create local commits only after explicit approval.

## Local discovery

Codex discovers personal skills under `$HOME/.agents/skills`. Link an individual
skill from this checkout into that directory:

```bash
mkdir -p "$HOME/.agents/skills"
ln -s /path/to/agent-skills/commit-local-changes \
  "$HOME/.agents/skills/commit-local-changes"
```

Keep project-specific skills in the project's `.agents/skills` directory.
