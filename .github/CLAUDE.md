# .github/ — Claude Code Guidance

## Overview

This directory contains GitHub Actions workflows and Dependabot configuration for the job-root-cache composite actions.

## Cleanup Workflow (`workflows/cleanup.yml`)

A reusable workflow (`workflow_call`) that deletes the job setup cache for the current run. Called by consuming workflows after their jobs complete to clean up the `job-setup-cache-<run_id>-<os>` cache entry.

- Runs with `if: always()` to ensure cleanup happens regardless of job success/failure
- Requires `GH_TOKEN` (via `secrets.GITHUB_TOKEN`) and `GH_REPO` (via `github.repository`)
- Cache key pattern: `job-setup-cache-${{ github.run_id }}-${{ runner.os }}`

## Release Workflow (`workflows/release.yml`)

Delegates to the reusable workflow in [`brianespinosa/release-action`](https://github.com/brianespinosa/release-action). All release logic (semver versioning via git-cliff, changelog generation, GitHub release creation, alias tag management, and Dependabot major version PR title rewriting) lives there.

**Semver mapping** (handled by git-cliff via conventional commits):
- `feat` → minor bump
- `fix` → patch bump
- `!` after type/scope (e.g. `feat(ci)!:`) or `BREAKING CHANGE` footer → major bump
- `chore`, `ci`, `docs`, `style` → no bump

**REQUIRED: All PRs must be merged via squash merge.** The reusable workflow rewrites Dependabot major-version PR titles from `fix(deps):` to `feat(deps)!:` — this only takes effect when squash merging (the PR title becomes the commit message).

## Dependabot (`dependabot.yml`)

- Targets `github-actions` ecosystem only
- Single update entry with `fix(deps):` conventional commit prefix for all updates
- Major version bumps get their PR title rewritten to `feat(deps)!:` by the release workflow
