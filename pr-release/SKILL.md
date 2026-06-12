---
name: pr-release
description: Create a branch from pending commits, push, open a documented PR via gh, and request review from project contributors.
argument-hint: "[optional: PR title or scope override]"
allowed-tools: Bash(git *) Bash(gh *) Read Glob Grep
---

# PR Release

Create a release branch from pending commits, push it, open a pull request with a structured summary via `gh`, and request review from project contributors. Use when you're ready to ship your work.

## Arguments

$ARGUMENTS — Optional. A PR title or scope override (e.g., "fix Turbo Drive on quotation pages"). If provided, use it as the PR title instead of auto-generating one. If empty, derive the title from the commit messages.

## Rules

- **Branch naming**: `<type>/<summary>` where type is `fix`, `feat`, or `chore` — determined by analyzing the pending commits. Summary is lowercase, hyphenated, max 50 chars (e.g., `fix/turbo-drive-stimulus-quotations`).
- **NEVER push directly to `main`** — always create a branch first.
- **NEVER force-push** (`--force`, `--force-with-lease`) unless the user explicitly asks.
- **Commit before branching.** If there are uncommitted changes, tell the user to run `/commit` first and stop.
- **PR body format**: Use the structured template defined in Step 4. Always include a Summary section and a list of commits.
- **Reviewers**: Request review from all contributors found via `gh api` — skip the current user.
- If there are no pending commits ahead of the remote, say "Nenhum commit pendente para criar PR." and stop.
- If `gh` is not authenticated, tell the user to run `gh auth login` and stop.

## Steps

### 1. Check Prerequisites

```bash
git status
git branch --show-current
git log --oneline origin/main..HEAD
gh auth status
```

- If there are uncommitted changes, say "Há mudanças não commitadas. Rode `/commit` primeiro." and stop.
- If `origin/main..HEAD` is empty (no pending commits), say "Nenhum commit pendente para criar PR." and stop.
- If `gh auth status` fails, say "gh CLI não autenticado. Rode `gh auth login`." and stop.

### 2. Analyze Commits

```bash
git log --oneline origin/main..HEAD
git diff origin/main..HEAD --stat
```

- Read the commit messages to determine the **dominant type** (`fix`, `feat`, or `chore`)
- Draft a **branch name**: `<type>/<hyphenated-summary>` (max 50 chars total)
- Draft a **PR title**: use $ARGUMENTS if provided, otherwise generate one — concise, under 70 chars, in English
- Collect all commit messages for the PR body

### 3. Create Branch & Push

Check the current branch first:

- **Already on a feature branch** (not `main`): skip branch creation, just push the current branch.
- **On `main`**: create a new branch and push it.

```bash
git checkout -b <branch-name>
git push -u origin <branch-name>
```

### 4. Create PR

Build the PR body from the commits, then create via `gh`:

```bash
gh pr create --base main --title "<PR title>" --body "$(cat <<'EOF'
## Summary

<2-4 bullet points explaining what changed and why>

## Commits

<list of all commits with hashes>

## How to test

<brief verification steps>

---
*Created with `/pr-release`*
EOF
)"
```

### 5. Request Reviewers

```bash
gh repo view --json nameWithOwner --jq '.nameWithOwner'
gh api repos/<owner>/<repo>/contributors --jq '.[].login'
```

- Filter out the current user (`gh api user --jq '.login'`)
- Request review from remaining contributors:

```bash
gh pr edit <PR-number> --add-reviewer <reviewer1>,<reviewer2>
```

If no other contributors exist, skip this step and note it in the summary.

### 6. Summary

**Branch**: `<branch-name>`

**PR**: `<PR-URL>`

**Title**: `<PR title>`

**Commits**: `<N>` commits included

**Reviewers**: `<list>` (or "none — solo project")
