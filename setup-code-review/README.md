# Setup Code Review Plugin

A Claude Code plugin that installs the smart code review workflow to any repository via pull request.

## Usage

Run in any GitHub repository:

```
/setup-code-review:install
```

This will:
1. Create a new branch `add-claude-code-review`
2. Add the workflow file at `.github/workflows/claude-code-review.yml`
3. Create a pull request with setup instructions

## What Gets Installed

The workflow includes:
- Automatic code review on every PR
- Smart re-review support for new commits
- CLAUDE.md compliance checking
- 80% confidence threshold to reduce false positives

## After Merging

You'll need to add the `CLAUDE_CODE_OAUTH_TOKEN` secret:

1. Go to **Settings > Secrets and variables > Actions**
2. Click **New repository secret**
3. Name: `CLAUDE_CODE_OAUTH_TOKEN`
4. Value: Your Claude Code OAuth token
