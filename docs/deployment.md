# Deployment Guide

## Prerequisites

- A GitHub repository where you want automated dependency updates
- A GitHub personal access token (PAT) or GitHub App token with `repo` scope
- Docker available on your GitHub Actions runner

## Basic Usage

Create a workflow file `.github/workflows/renovate.yml` in your target repository:

```yaml
name: Renovate

on:
  schedule:
    # Run every day at 02:00 UTC
    - cron: '0 2 * * *'
  workflow_dispatch:

jobs:
  renovate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Run Renovate
        uses: SMSDAO/github-action@main
        with:
          token: ${{ secrets.RENOVATE_TOKEN }}
          configurationFile: renovate.json
```

## Configuration Options

| Input                     | Description                            | Required                      | Default                              |
| ------------------------- | -------------------------------------- | ----------------------------- | ------------------------------------ |
| `token`                   | GitHub token for Renovate              | No (use `RENOVATE_TOKEN` env) | —                                    |
| `configurationFile`       | Path to Renovate config file           | No                            | —                                    |
| `renovate-version`        | Renovate Docker image version          | No                            | `42`                                 |
| `renovate-image`          | Renovate Docker image name             | No                            | `ghcr.io/renovatebot/renovate`       |
| `mount-docker-socket`     | Mount Docker socket in container       | No                            | —                                    |
| `docker-socket-host-path` | Host path for Docker socket            | No                            | `/var/run/docker.sock`               |
| `docker-cmd-file`         | Override Docker command                | No                            | —                                    |
| `docker-network`          | Docker network to use                  | No                            | —                                    |
| `docker-user`             | Docker user                            | No                            | —                                    |
| `docker-volumes`          | Docker volume mounts                   | No                            | `/tmp:/tmp`                          |
| `env-regex`               | Regex for env vars passed to container | No                            | `^(?:RENOVATE_\w+\|LOG_LEVEL\|...)$` |

## Self-Hosted Runner Setup

If you are using a self-hosted runner, ensure Docker is installed and the runner user has Docker permissions:

```bash
sudo usermod -aG docker $RUNNER_USER
```

## Pinning the Renovate Version

For reproducible runs, pin to a specific Renovate version:

```yaml
- uses: SMSDAO/github-action@main
  with:
    token: ${{ secrets.RENOVATE_TOKEN }}
    renovate-version: '42.64.1'
```

## GitHub App Authentication

For higher API rate limits, use a [GitHub App](https://docs.github.com/en/apps) instead of a PAT:

```yaml
- name: Generate GitHub App token
  id: app-token
  uses: actions/create-github-app-token@v1
  with:
    app-id: ${{ secrets.APP_ID }}
    private-key: ${{ secrets.APP_PRIVATE_KEY }}

- name: Run Renovate
  uses: SMSDAO/github-action@main
  with:
    token: ${{ steps.app-token.outputs.token }}
```
