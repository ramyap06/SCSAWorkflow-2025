---
name: spac-commit
description: "Use when generating Conventional Commit messages from staged changes, or proposing multiple commit options for unstaged changes with commit separation advice per CONTRIBUTING.md. Triggers: commit message, conventional commits, staged changes, unstaged changes, split commits, commit scope."
---

# Commit Message Generator Skill

Generate commit messages according to CONTRIBUTING.md.

## Scope and Intent

This skill is focused only on commit-message planning and commit separation.

- If staged changes exist: propose one high-quality Conventional Commit message.
- If there are no staged changes: propose multiple commit options for unstaged changes and recommend logical separation.

## Rules from CONTRIBUTING.md

Follow exactly:

`<type>(scope): <short description>`

- `scope` is optional in Conventional Commits, but recommended in this repo.
- For multi-line commits, keep a blank line between subject and body.

Types and Version Impact:
- BREAKING CHANGE (→ major version): Incompatible API changes
  - Add `BREAKING CHANGE: <description>` in the message body
- feat (→ minor version): New backward-compatible feature
- fix (→ patch version): Bug fix
- docs: Documentation updates only
- style: Code style/formatting (no logic change)
- refactor: Code refactoring without new features or fixes
- test: Adding/updating tests
- chore: Miscellaneous tasks (dependencies, config, etc.)

Message quality:
- Keep description under 50 chars when possible
- Use present-tense imperative verbs
- Keep scope meaningful and precise
- Include `BREAKING CHANGE: <description>` in body when needed
- Include more details in body when helpful

## Scope Selection Strategy

Prefer more specific scopes first:

1. function/method name (most specific)
2. file/module name
3. broader SPAC terminology fallback

SPAC terminology fallback options (from CONTRIBUTING.md):
- cells
- features
- tables
- associated-tables
- annotation
- visualization
- utils
- transformation
- core

Examples:
- `fix(histogram): ...`
- `test(visualization): ...`
- `feat(annotation): ...`

## Workflow

### A) Staged changes exist

1. Inspect staged diff.
2. Infer best `type` + `scope` + short description.
3. Generate message option(s).
4. Ask permission before committing.

### B) No staged changes

1. Inspect unstaged diff.
2. Group changes into logical commit units.
3. Provide multiple options when reasonable, e.g.:
   - function-focused split
   - file-focused split
4. For each proposed commit:
   - suggest scope
   - ask approval or custom scope
5. Ask permission before staging/committing.

If multiple groupings are reasonable, present at least two options.

## Broader Contribution Awareness

When proposing commit splits, consider CONTRIBUTING workflow quality:
- keep commits narrow and modular
- separate implementation, tests, and docs when practical
- align with PR readability and reviewability
- explain briefly why a proposed grouping/scope is recommended

## Output Templates

### Staged

Proposed commit (with optional body):
`fix(histogram): prevent external figure closure`

```
- ...
- ...
```

Proceed to commit?

### Unstaged (multi-option)

Option A (function-focused):
1. `fix(histogram): ...`
2. `test(histogram): ...`

Option B (file-focused):
1. `fix(visualization): ...`

Choose A/B (or custom) options.

Ready to apply selected commits?

### Unstaged (single-option)

Suggested commits:
1. `fix(histogram): ...`
2. `test(histogram): ...`

Ready to proceed?
