# Environment Variables

This action reads environment variables from the runner and forwards matching ones into the Renovate Docker container.

## Required Variables

| Variable         | Description                                                                                         |
| ---------------- | --------------------------------------------------------------------------------------------------- |
| `RENOVATE_TOKEN` | GitHub token used by Renovate to authenticate. Required unless passed via the `token` action input. |

## Optional Variables

| Variable                                  | Description                                                   | Default |
| ----------------------------------------- | ------------------------------------------------------------- | ------- |
| `LOG_LEVEL`                               | Logging verbosity: `debug`, `info`, `warn`, `error`           | `info`  |
| `RENOVATE_CONFIG_FILE`                    | Path to the Renovate configuration file inside the container  | —       |
| `GITHUB_COM_TOKEN`                        | Token for github.com when running on GitHub Enterprise Server | —       |
| `NODE_OPTIONS`                            | Node.js options passed into the container                     | —       |
| `HTTPS_PROXY` / `HTTP_PROXY` / `NO_PROXY` | Proxy settings (case-insensitive)                             | —       |

## Environment Variable Filtering

By default, only variables matching this regex are forwarded to the Renovate container:

```
^(?:RENOVATE_\w+|LOG_LEVEL|GITHUB_COM_TOKEN|NODE_OPTIONS|(?:HTTPS?|NO)_PROXY|(?:https?|no)_proxy)$
```

You can override this with the `env-regex` action input:

```yaml
- uses: SMSDAO/github-action@main
  with:
    token: ${{ secrets.RENOVATE_TOKEN }}
    env-regex: '^(?:RENOVATE_\w+|LOG_LEVEL|MY_CUSTOM_VAR)$'
```

## Example `.env` / Secrets Setup

Copy `.env.example` to `.env` for local development:

```bash
cp .env.example .env
```

Then edit `.env` to set your values. **Never commit `.env` files with real secrets.**

For GitHub Actions, store secrets in your repository's **Settings → Secrets and variables → Actions**.

## CI/CD Variables

These variables are set automatically by GitHub Actions and do not need to be configured:

| Variable       | Description                            |
| -------------- | -------------------------------------- |
| `GITHUB_TOKEN` | Short-lived token for the workflow run |
| `GITHUB_REF`   | Branch/tag ref for the current run     |
| `GITHUB_SHA`   | Commit SHA for the current run         |
