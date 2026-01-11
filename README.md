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

This creates a PR with:
- The `smart-code-review` plugin embedded in `.claude/plugins/`
- A GitHub Actions workflow file

After merging, add the `CLAUDE_CODE_OAUTH_TOKEN` secret.

## Installation Options

### Option A: Embed Plugin in Repository (Recommended)

This approach commits the plugin to your repo, avoiding any external dependencies at runtime.

**Step 1:** Copy the plugin to your repository:

```bash
mkdir -p .claude/plugins
git clone --depth 1 https://github.com/lostendfound/claude-plugins-marketplace.git /tmp/marketplace
cp -r /tmp/marketplace/smart-code-review .claude/plugins/
rm -rf /tmp/marketplace
```

**Step 2:** Create `.github/workflows/claude-code-review.yml`:

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
          # Plugin is auto-discovered from .claude/plugins/
          prompt: "/smart-code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
```

**Step 3:** Commit both the plugin and workflow, then add `CLAUDE_CODE_OAUTH_TOKEN` secret.

### Option B: Clone Plugin at Runtime

If you don't want to commit the plugin to your repo, you can clone it during the workflow:

```yaml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]

jobs:
  claude-review:
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

      - name: Setup smart-code-review plugin
        run: |
          mkdir -p .claude/plugins
          git clone --depth 1 https://github.com/lostendfound/claude-plugins-marketplace.git /tmp/marketplace
          cp -r /tmp/marketplace/smart-code-review .claude/plugins/
          rm -rf /tmp/marketplace

      - name: Run Claude Code Review
        uses: anthropics/claude-code-action@v1
        with:
          claude_code_oauth_token: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
          prompt: "/smart-code-review:code-review ${{ github.repository }}/pull/${{ github.event.pull_request.number }}"
```

## For Local Claude Code

Add the marketplace and install the plugin:

```bash
claude /plugin marketplace add https://github.com/lostendfound/claude-plugins-marketplace.git
claude /plugin install smart-code-review@lostendfound-plugins
```

> **Note:** For GitHub Actions, we recommend the embedded approach above instead of using `plugin_marketplaces`, which can cause conflicts on self-hosted runners.

## Creating New Plugins

1. Create a new directory under the root (e.g., `my-plugin/`)
2. Add `.claude-plugin/plugin.json` with plugin metadata
3. Add commands in `commands/` directory
4. Update `.claude-plugin/marketplace.json` to include your plugin
5. Commit and push

## License

MIT
