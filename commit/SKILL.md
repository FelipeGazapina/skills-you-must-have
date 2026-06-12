---
name: commit
description: Stage and commit current changes with a conventional commit message (feat|fix|chore|style). Use when you want to commit your work.
allowed-tools: Bash(git *) Read Glob Grep
---

# Commit

Stage current changes and create one or more commits with conventional commit messages.

## Rules

- Commit message format: `type: summary`
  - **type**: `feat` (new feature), `fix` (bug fix), `chore` (refactor, config, tooling, cleanup), `style` (UI/CSS-only changes, no logic)
  - **summary**: max 300 characters, lowercase first letter, no period at the end, in English
- Analyze the **full diff** to determine the correct type and write a meaningful summary
- If changes span **unrelated concerns**, split into separate commits (e.g., feature code in one, config/tooling in another)
- If changes are all related, use a single commit with the **dominant** type
- **NEVER** add `Co-Authored-By` or any co-author trailer to the commit message
- **NEVER** commit files that contain secrets (`.env`, credentials, API keys). Warn the user if detected.
- **NEVER** use `git add -A` or `git add .` — add specific files by name
- **NEVER** use `--no-verify` or `--no-gpg-sign`
- If a pre-commit hook fails, report the error and stop — do NOT retry
- If the diff is empty, tell the user and stop

## Steps

### 1. Understand the Changes

```bash
git status
git diff HEAD
git log --oneline -5
```

If the diff is empty and there are no untracked files, say "Nada para commitar." and stop.

Check recent commit messages to match the repo's style.

### 2. Analyze

- Read the diff carefully to understand what changed and why
- Use Read/Glob/Grep if the diff alone isn't enough to understand the intent
- Determine if the changes should be **one commit or multiple**:
  - Same concern (e.g., all part of one feature) → single commit
  - Mixed concerns (e.g., feature + tooling, bugfix + config) → split into separate commits
- For each commit, determine the type:
  - `feat`: new functionality, new endpoints, new UI elements, new models/controllers
  - `fix`: bug fixes, corrections to existing behavior
  - `style`: UI/CSS-only changes, theme updates, no logic changes
  - `chore`: refactoring, config changes, dependency updates, tooling, cleanup, skill files
- Draft each summary: focus on the **why** not the **what**, max 300 chars

### 3. Stage & Commit

For each commit:

1. Add files by name — never `git add .`
2. Skip `.env*`, `credentials*`, `master.key`, or any file that looks like it contains secrets
3. For new untracked files, include them only if they are part of the current work
4. Create the commit:

```bash
git commit -m "type: summary here"
```

### 4. Confirm

Run `git log --oneline -N` (where N = number of commits created) to show the results.
