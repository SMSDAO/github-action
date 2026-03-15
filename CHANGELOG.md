# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

For historical changelogs, see the [GitHub releases page](https://github.com/renovatebot/github-action/releases/).

## [Unreleased]

### Added

- `docs/` directory with architecture, deployment, environment variables, and user guides
- `docs/guides/admin-guide.md` — RBAC, billing controls, audit logging, security configuration
- `docs/guides/developer-guide.md` — build system internals, CI/CD pipeline, debugging, contributing
- `docs/assets/ui/` — Neo-Glow dashboard screenshots: user, admin, and developer views
- `typecheck` script (`tsc --noEmit`) to `package.json` for explicit TypeScript validation
- Typecheck step in CI `lint` job
- Security scanning workflow (`.github/workflows/security.yml`)
- `README.md`: added `Documentation` section linking to all docs and `UI Preview` section embedding all three dashboard screenshots

### Changed

- `build.yml`: CI now runs `lint` and `commitlint` on all push and pull_request events (not just forks)
- `build.yml`: `release` job now initializes the `release` branch automatically if it does not exist, eliminating the manual branch bootstrap requirement
- `build.yml`: `release` job now uses `if: always()` guard with failure/cancellation checks so it only runs when all upstream jobs succeed or are skipped cleanly
- `.env.example`: Replaced placeholder Next.js variables with correct Renovate-specific environment variable documentation

### Fixed

- CI `release` job failing with `"A branch or tag with the name 'release' could not be found"` when the `release` branch has not been initialized in the repository
- `build.yml` `merge` step not writing `commit` output to `GITHUB_OUTPUT` (was using unset shell variable)

## [v1.0.0] - 2026-03-15

### Added

- Enterprise stabilization: deterministic pnpm installs with frozen lockfile
- Full documentation suite in `docs/`
- Explicit TypeScript type-checking in CI
- Automated `release` branch initialization in CI
