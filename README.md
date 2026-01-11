# Claude Plugins Marketplace

A collection of custom Claude Code plugins for code review and development workflows.

## Available Plugins

| Plugin | Description |
|--------|-------------|
| [smart-code-review](./smart-code-review) | Intelligent code review with re-review support for new commits |
| [setup-code-review](./setup-code-review) | Install smart code review workflow to any repository via PR |

## Quick Start

To add smart code review to any repository, run:

```
/setup-code-review:install
```

This creates a PR with the workflow file. After merging, add the `CLAUDE_CODE_OAUTH_TOKEN` secret.

## Installation

### For GitHub Actions

Add the marketplace to your workflow:

```yaml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]

jobs:
  claude-review:
    runs-on: ubuntu-latest  # or self-hosted
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
            https://github.com/YOUR_USERNAME/claude-plugins-marketplace.git
          plugins: |
            smart-code-review@lostendfound-plugins
          prompt: "/smart-code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
```

### For Local Claude Code

Add the marketplace globally:

```bash
claude /plugin marketplace add https://github.com/YOUR_USERNAME/claude-plugins-marketplace.git
claude /plugin install smart-code-review@lostendfound-plugins
```

## Creating New Plugins

1. Create a new directory under the root (e.g., `my-plugin/`)
2. Add `.claude-plugin/plugin.json` with plugin metadata
3. Add commands in `commands/` directory
4. Update `.claude-plugin/marketplace.json` to include your plugin
5. Commit and push

## License

MIT
