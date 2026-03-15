# Architecture

## Overview

This repository implements a **GitHub Action** that runs [Renovate](https://github.com/renovatebot/renovate) as a self-hosted bot using Docker. Renovate automates dependency updates across your repositories.

## Directory Structure

```
github-action/
├── .github/
│   ├── actions/
│   │   └── setup-node/      # Composite action: install Node + pnpm with caching
│   └── workflows/
│       └── build.yml        # CI/CD pipeline: lint, e2e tests, release
├── docs/                    # Project documentation
├── example/                 # Example Renovate configuration files
├── src/                     # Action source code (TypeScript)
│   ├── index.ts             # Entry point
│   ├── input.ts             # Input parsing (action inputs + env vars)
│   ├── docker.ts            # Docker image/version resolution
│   └── renovate.ts          # Docker container execution logic
├── tools/
│   ├── compile.js           # esbuild compilation script
│   ├── cjs-shim.ts          # CommonJS shim for ESM bundles
│   └── tsconfig.json        # TypeScript config for tools
├── action.yml               # GitHub Action manifest
├── package.json             # Node.js project manifest
└── pnpm-lock.yaml           # Deterministic dependency lock file
```

## Component Relationships

```
action.yml
  └── runs: node20, main: dist/index.js
        └── src/index.ts          (entry point)
              ├── input.ts         (parses action inputs + env filtering)
              ├── docker.ts        (resolves image:tag)
              └── renovate.ts      (constructs + executes docker run)
```

## Build System

- **Runtime**: Node.js 20/22/24
- **Package manager**: pnpm 10 (lockfile: `pnpm-lock.yaml`)
- **Bundler**: esbuild → produces `dist/index.js` (ESM format)
- **Type checking**: TypeScript 5 via `tsc --noEmit`
- **Linting**: ESLint 9 with `typescript-eslint` + `eslint-config-prettier`
- **Formatting**: Prettier

## Release Strategy

Releases use [semantic-release](https://semantic-release.gitbook.io/) with the [conventional commits](https://www.conventionalcommits.org/) preset.

- CI merges `main` into the `release` branch
- semantic-release analyses commits on `release`, bumps the version, and publishes a GitHub Release
- Compiled `dist/` files are committed to the `release` branch (not `main`)
