# Smart Code Review Plugin

An intelligent code review plugin for Claude Code that supports re-reviewing PRs after new commits are pushed.

## Features

- **Smart eligibility checking**: Detects when new commits have been pushed since the last review
- **Full review mode**: Comprehensive code review for new PRs
- **Re-review mode**: Verifies if previous issues were fixed AND scans for new issues
- **Confidence scoring**: Only reports issues with 80%+ confidence to reduce false positives
- **CLAUDE.md compliance**: Checks code against project guidelines

## Review Modes

### Full Review (Initial)
When no previous Claude review exists on the PR:
- Runs 5 parallel agents to check CLAUDE.md compliance, bugs, git history, previous PR comments, and code comments
- Scores each issue for confidence
- Posts a standard code review comment

### Re-Review (Follow-Up)
When a previous Claude review exists AND new commits have been pushed:
- Parses previous review to extract issues
- Verifies if each issue was addressed (FIXED/STILL PRESENT)
- Scans new commits for new issues
- Posts a follow-up comment with status summary

### Skip
When a previous Claude review exists but NO new commits have been pushed:
- Skips the review (already reviewed, no changes)

## Usage

### In GitHub Actions Workflow

```yaml
- name: Run Claude Code Review
  uses: anthropics/claude-code-action@v1
  with:
    claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    plugin_marketplaces: |
      https://github.com/YOUR_USERNAME/claude-plugins-marketplace.git
    plugins: |
      smart-code-review@lostendfound-plugins
    prompt: "/smart-code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
```

### Manual Invocation
```
/smart-code-review:code-review owner/repo/pull/123
```

## Configuration

The plugin uses an 80% confidence threshold by default. Issues below this threshold are filtered out to reduce noise.
