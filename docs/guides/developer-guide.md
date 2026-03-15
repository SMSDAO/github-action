# Developer Guide

## Overview

This guide covers local development, build system internals, CI/CD pipeline details, debugging techniques, and diagnostic tooling for the Renovate GitHub Action.

## Project Structure

```
src/
├── index.ts       # Entry point: reads input, runs Renovate container
├── input.ts       # Parses action inputs and filters env vars
├── docker.ts      # Resolves Docker image name and version tag
└── renovate.ts    # Constructs docker run command and executes it

tools/
├── compile.js     # esbuild compilation script
├── cjs-shim.ts    # CJS compatibility shim for ESM bundles
└── tsconfig.json  # TypeScript config for tooling scripts
```

## Development Setup

```bash
# Install Node.js >= 20.9.0 (latest LTS recommended; see .node-version for pinned version)
node --version  # >= 20.9.0

# Install pnpm 10 (via Corepack — recommended — or npm)
corepack enable && corepack prepare pnpm@10 --activate
# or: npm install -g pnpm@10

# Install dependencies (frozen lockfile enforced)
HUSKY=0 pnpm install --frozen-lockfile

# Build
pnpm build

# Type check
pnpm typecheck

# Lint + format check
pnpm lint

# Auto-fix lint + format
pnpm lint:fix
```

## Build System

### Compilation

The action is bundled with [esbuild](https://esbuild.github.io/) into a single `dist/index.js` file:

```js
// tools/compile.js
await build({
  entryPoints: ['./src/index.ts'],
  bundle: true,
  platform: 'node',
  target: 'node20',
  minify: !!env['CI'], // Minify only in CI
  tsconfig: 'tsconfig.dist.json',
  sourcemap: true,
  format: 'esm',
  outdir: './dist/',
  inject: ['tools/cjs-shim.ts'], // CJS compatibility
});
```

The `dist/` directory is `.gitignore`d on `main` but committed on the `release` branch by semantic-release.

### TypeScript Configuration

- `tsconfig.json` — base config (strictest + node20, used for linting/type-checking)
- `tsconfig.dist.json` — distribution config (extends base, entry-point scoped)
- `tools/tsconfig.json` — config for tooling scripts

## CI/CD Pipeline

### Workflow Jobs (`build.yml`)

| Job                         | Trigger            | Description                                            |
| --------------------------- | ------------------ | ------------------------------------------------------ |
| `prepare`                   | push/PR            | Installs deps and saves `node_modules` cache           |
| `commitlint`                | push/PR            | Validates commit messages against Conventional Commits |
| `lint`                      | push/PR            | ESLint + Prettier + TypeScript type-check              |
| `e2e`                       | push/PR (non-fork) | Builds action and runs full Renovate e2e test          |
| `renovate-config-validator` | push/PR            | Validates example config files                         |
| `release`                   | push to `main`     | Merges into `release` branch, runs semantic-release    |

### Dependency Caching

The `setup-node` composite action uses a two-level cache strategy:

1. Restore `node_modules` from cache keyed on `hash(.node-version, pnpm-lock.yaml)`
2. If cache hit → skip install entirely
3. If cache miss → run `pnpm install --frozen-lockfile`, then save cache

### Release Process

1. CI merges `main` into the `release` branch (creating it if needed)
2. `semantic-release` analyses commits using Conventional Commits preset
3. Bumps version in `package.json`, generates release notes, commits `dist/` to `release`
4. Creates a GitHub Release and tag (e.g. `v1.0.0`)

## Debugging

### Enable Debug Logging

```yaml
- uses: SMSDAO/github-action@main
  with:
    token: ${{ secrets.RENOVATE_TOKEN }}
  env:
    LOG_LEVEL: debug
```

### Inspect Docker Command

Set `LOG_LEVEL=debug` and look for the `docker run` command in the action logs. To reproduce locally:

```bash
docker run -t --rm \
  --env RENOVATE_TOKEN=... \
  --env LOG_LEVEL=debug \
  --volume /tmp:/tmp \
  ghcr.io/renovatebot/renovate:42 \
  --repositories=myorg/myrepo
```

### Environment Variable Filtering

To pass a custom environment variable into the Renovate container, either:

1. Use an existing prefix (`RENOVATE_*`) — automatically forwarded
2. Override `env-regex` to include your variable:

```yaml
- uses: SMSDAO/github-action@main
  with:
    env-regex: '^(?:RENOVATE_\w+|LOG_LEVEL|MY_CUSTOM_VAR)$'
```

### Validate Config Locally

```bash
# With Docker
docker run --rm \
  ghcr.io/renovatebot/renovate:42 \
  renovate-config-validator /path/to/renovate.json
```

## Testing Changes

Since this action runs Docker, end-to-end tests require a real GitHub token. The CI `e2e` job handles this using `secrets.RENOVATE_TOKEN` (falls back to `GITHUB_TOKEN`).

For local iteration without Docker:

1. Mock `@actions/exec` in a test file
2. Instantiate `Input` and `Renovate` classes with mocked inputs
3. Assert the constructed `docker run` command string

## Adding New Inputs

1. Add the input definition to `action.yml`
2. Add a getter method in `src/input.ts`
3. Use the value in `src/renovate.ts` where appropriate
4. Update `docs/deployment.md` configuration table
5. Update `docs/env-vars.md` if a corresponding env var is added

## Dependency Updates

This repo uses its own Renovate config (`.github/renovate.json`) to keep its dependencies current. Do not manually update `pnpm-lock.yaml` — let Renovate create PRs.
