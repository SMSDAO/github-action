# Admin Guide

## Overview

This guide covers administration of the Renovate GitHub Action deployment, including access governance, billing controls, security settings, and audit logging.

## Access Governance

This action does not implement application-level RBAC — access control is enforced through GitHub's native permission model and Renovate's self-hosted configuration options. The table below maps common organizational roles to the appropriate GitHub permission level and Renovate config scope:

| Organizational Role     | GitHub Permission  | Renovate Scope                                                             |
| ----------------------- | ------------------ | -------------------------------------------------------------------------- |
| **Org Admin**           | Organization owner | Can manage secrets, tokens, and workflow permissions for all repos         |
| **Repo Admin**          | Repository admin   | Can configure `renovate.json`, approve/merge PRs, manage branch protection |
| **Developer**           | Repository write   | Can review and merge Renovate PRs, adjust per-repo config                  |
| **Read-only / Auditor** | Repository read    | Can view Renovate PRs and logs; no ability to modify config                |

For team-level access control, use [GitHub Teams](https://docs.github.com/en/organizations/organizing-members-into-teams/about-teams) to assign repository permissions to groups of users.

## Repository Scoping (User Management)

Admins control which repositories Renovate manages via the `RENOVATE_AUTODISCOVER_FILTER` and related Renovate configuration options:

```js
// renovate.json / global config
{
  autodiscoverFilter: ['myorg/*'],   // Repos to manage
  onboarding: false,                 // Skip per-repo onboarding
  requireConfig: 'optional',        // Config not required per repo
}
```

For organization-wide deployment, set `RENOVATE_REPOSITORIES` to control which repos are managed:

```yaml
env:
  RENOVATE_REPOSITORIES: '["myorg/frontend","myorg/api-service"]'
```

## Billing and Metering Controls

Monitor API usage to avoid GitHub rate limit exhaustion:

| Control           | Environment Variable               | Description              |
| ----------------- | ---------------------------------- | ------------------------ |
| Rate limit buffer | `RENOVATE_PR_HOURLY_LIMIT`         | Max PRs opened per hour  |
| Concurrent PRs    | `RENOVATE_PR_CONCURRENT_LIMIT`     | Max open PRs at one time |
| Branch limit      | `RENOVATE_BRANCH_CONCURRENT_LIMIT` | Max concurrent branches  |

```yaml
env:
  RENOVATE_PR_HOURLY_LIMIT: '5'
  RENOVATE_PR_CONCURRENT_LIMIT: '20'
```

## API Monitoring

Track GitHub API rate limit usage in Renovate logs (set `LOG_LEVEL=debug`):

```
DEBUG Rate limit remaining: 4243/5000
DEBUG X-RateLimit-Reset: 1710000000
```

For persistent monitoring, integrate with your observability stack via structured logs:

```yaml
env:
  LOG_LEVEL: debug
  LOG_FORMAT: json
```

## Audit Logging

All Renovate runs produce structured log output. Capture and retain logs using GitHub Actions log retention or ship to an external log aggregator:

```yaml
- name: Run Renovate
  uses: SMSDAO/github-action@main
  with:
    token: ${{ secrets.RENOVATE_TOKEN }}
  env:
    LOG_LEVEL: info

- name: Upload Renovate logs
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: renovate-logs-${{ github.run_number }}
    path: /tmp/renovate-log-*.ndjson
    retention-days: 30
```

## Security Configuration

### Secret Scanning

Enable GitHub's secret scanning and push protection for all repositories:

- Navigate to **Repository → Settings → Security → Code security and analysis**
- Enable **Secret scanning** and **Push protection**

The `.github/workflows/security.yml` in this action provides automated:

- Dependency review on pull requests
- CodeQL analysis for JavaScript/TypeScript

### Secure Token Management

Store all tokens as GitHub Actions secrets, never in code:

```yaml
# Good practice
token: ${{ secrets.RENOVATE_TOKEN }}

# Never do this
token: ghp_actualtoken...  # ❌
```

### Dependency Pinning

Pin action versions to commit SHAs to prevent supply-chain attacks:

```yaml
# Pinned to SHA (recommended)
uses: actions/checkout@8e8c483db84b4bee98b60c0593521ed34d9990e8 # v6.0.1

# Version tag only (less secure)
uses: actions/checkout@v4  # ❌ for production
```

## Configuration Settings

Key global Renovate configuration options:

```js
module.exports = {
  // Runs on all repos in the org
  autodiscover: true,
  autodiscoverFilter: ['myorg/*'],

  // Onboarding
  onboarding: false,
  requireConfig: 'optional',

  // Safety limits
  prConcurrentLimit: 20,
  prHourlyLimit: 5,

  // Useful for avoiding conflicts with Renovate App
  branchPrefix: 'self-hosted-renovate/',

  // Extend from shared presets
  extends: ['config:base'],
};
```
