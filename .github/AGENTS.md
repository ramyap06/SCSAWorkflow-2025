# Custom Agents

This file registers custom agents available in this workspace.

## Available Agents

### SPAC
- **File**: `.github/agents/spac.agent.md`
- **Description**: General SPAC development assistant aligned with CONTRIBUTING.md workflows: implementation, testing, docs, visualization, and PR readiness.
- **Use When**:
  - You want one primary agent for SPAC coding tasks
  - You want guidance that follows project terminology and contribution standards
  - You want to coordinate coding + tests + docs + commit preparation

## Available Skills

### Commit Message Generator
- **File**: `.github/skills/spac-commit/SKILL.md`
- **Slash Command**: `/spac-commit`
- **Description**: Generate Conventional Commit messages from staged changes, or propose multiple commit options for unstaged changes with separation advice.
- **Use When**:
  - You have staged changes and need a compliant commit message
  - You have unstaged changes and want commit splitting recommendations
  - You want scope suggestions based on function and file names, with approval before apply
