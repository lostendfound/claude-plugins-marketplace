---
allowed-tools:
  - Bash(gh repo view:*)
  - Bash(gh pr create:*)
  - Bash(gh pr list:*)
  - Bash(git checkout:*)
  - Bash(git branch:*)
  - Bash(git add:*)
  - Bash(git commit:*)
  - Bash(git push:*)
  - Bash(git status:*)
  - Bash(git pull:*)
  - Bash(mkdir:*)
  - Read
  - Write
  - Glob
description: Install smart code review workflow to the current repository via PR
---

# Install Smart Code Review

Install the smart code review GitHub Actions workflow to the current repository by creating a pull request.

## Instructions

Follow these steps precisely:

### Step 1: Check Prerequisites

1. Verify we're in a git repository with a GitHub remote
2. Check if `.github/workflows/claude-code-review.yml` already exists
   - If it exists, inform the user and ask if they want to update it
3. Get the repository's default branch name using `gh repo view --json defaultBranchRef`

### Step 2: Create Feature Branch

1. Ensure we're on the default branch and it's up to date
2. Create a new branch: `add-claude-code-review`
3. If branch already exists, ask user if they want to overwrite it

### Step 3: Create Workflow File

Create the file `.github/workflows/claude-code-review.yml` with this content:

```yaml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]

jobs:
  claude-review:
    if: github.event.pull_request.draft == false
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
      issues: read
      id-token: write

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 1

      - name: Run Claude Code Review
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          plugin_marketplaces: |
            https://github.com/lostendfound/claude-plugins-marketplace.git
          plugins: |
            smart-code-review@lostendfound-plugins
          prompt: "/smart-code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
```

### Step 4: Commit and Push

1. Stage the workflow file: `git add .github/workflows/claude-code-review.yml`
2. Commit with message: `feat: add Claude smart code review workflow`
3. Push the branch to origin

### Step 5: Create Pull Request

Create a PR using `gh pr create` with:

**Title**: `Add Claude Smart Code Review workflow`

**Body**:
```markdown
## Summary

This PR adds an automated code review workflow using Claude Code with the smart-code-review plugin.

## Features

- **Automatic code review** on every PR
- **Smart re-review support** - when you push fixes, Claude will verify if previous issues were addressed
- **CLAUDE.md compliance** - checks code against your project's guidelines
- **80% confidence threshold** - reduces false positives

## Setup Required

After merging, you need to add the `CLAUDE_CODE_OAUTH_TOKEN` secret to your repository:

1. Go to **Settings > Secrets and variables > Actions**
2. Click **New repository secret**
3. Name: `CLAUDE_CODE_OAUTH_TOKEN`
4. Value: Your Claude Code OAuth token

## How It Works

1. Open a PR → Claude reviews and comments
2. Push fixes → Claude re-reviews and reports which issues are fixed
3. All issues fixed → Claude confirms completion

---
Generated with [Claude Code](https://claude.ai/code)
```

### Step 6: Return to Default Branch

Switch back to the default branch after creating the PR.

### Step 7: Output Result

Tell the user:
1. The PR was created successfully
2. Provide the PR URL
3. Remind them to add the `CLAUDE_CODE_OAUTH_TOKEN` secret after merging

## Error Handling

- If workflow file already exists and user doesn't want to update, exit gracefully
- If branch creation fails, provide clear error message
- If PR creation fails, check if PR already exists for that branch
