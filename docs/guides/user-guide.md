# User Guide

## Getting Started

This guide walks you through setting up automated dependency updates for your GitHub repositories using the Renovate GitHub Action.

### Step 1: Create a Renovate Token

1. Go to **GitHub → Settings → Developer settings → Personal access tokens**
2. Generate a token with `repo` scope (or use a GitHub App for better rate limits)
3. Add it as a repository secret named `RENOVATE_TOKEN`

### Step 2: Create a Renovate Configuration

Add a `renovate.json` file to the root of your repository:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:base"]
}
```

For more configuration options, see the [Renovate documentation](https://docs.renovatebot.com/).

### Step 3: Add the Workflow

Create `.github/workflows/renovate.yml` in your repository:

```yaml
name: Renovate

on:
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:

jobs:
  renovate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: SMSDAO/github-action@main
        with:
          token: ${{ secrets.RENOVATE_TOKEN }}
          configurationFile: renovate.json
```

### Step 4: Trigger the First Run

Manually trigger the workflow from **Actions → Renovate → Run workflow** to verify it works.

## Common Configuration Examples

### Update npm dependencies only

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:base"],
  "enabledManagers": ["npm"]
}
```

### Auto-merge minor and patch updates

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:base"],
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch"],
      "automerge": true
    }
  ]
}
```

### Limit to specific repositories

Set `RENOVATE_REPOSITORIES` in your workflow:

```yaml
env:
  RENOVATE_REPOSITORIES: '["myorg/myrepo"]'
```

## Troubleshooting

### Renovate creates no pull requests

- Check that your token has write access to the target repositories
- Verify your `renovate.json` is valid: run `renovate-config-validator renovate.json`
- Set `LOG_LEVEL=debug` for verbose output

### Rate limit errors

- Use a GitHub App instead of a PAT for higher rate limits
- Reduce the frequency of scheduled runs

### Docker socket errors

If you need Docker-in-Docker support, enable socket mounting:

```yaml
- uses: SMSDAO/github-action@main
  with:
    token: ${{ secrets.RENOVATE_TOKEN }}
    mount-docker-socket: 'true'
```
