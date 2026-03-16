# job-root-cache

![Last Updated](https://img.shields.io/github/last-commit/brianespinosa/job-root-cache?label=Last%20Updated&cacheSeconds=120)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

Composite actions that save, restore, and clean up a cache of the root directory across jobs in a single workflow run. Avoids re-running checkout and install for every parallel job.

- **`brianespinosa/job-root-cache/save`** — saves the current working directory to a run-scoped cache after checkout and install
- **`brianespinosa/job-root-cache/restore`** — restores the cached directory and sets up the correct Node version from `.nvmrc`
- **`brianespinosa/job-root-cache/cleanup`** (reusable workflow) — deletes the cache entry at the end of the workflow run

## Usage

```yaml ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

concurrency:
  group: ${{ github.workflow }}-${{ github.event.pull_request.number || github.ref }}
  cancel-in-progress: true

jobs:
  setup:
    runs-on: ubuntu-latest
    steps:
      - uses: brianespinosa/checkout-setup-node-install@main
      - uses: brianespinosa/job-root-cache/save@main

  lint:
    runs-on: ubuntu-latest
    needs: [setup]
    steps:
      - uses: brianespinosa/job-root-cache/restore@main
      - run: yarn lint

  test:
    runs-on: ubuntu-latest
    needs: [setup]
    steps:
      - uses: brianespinosa/job-root-cache/restore@main
      - run: yarn test

  job-cache-cleanup:
    runs-on: ubuntu-latest
    needs: [lint, test]
    if: always()
    uses: brianespinosa/job-root-cache/.github/workflows/cleanup.yml@main
```

## Assumptions

- A `.nvmrc` file is present in the repository root
- `brianespinosa/checkout-setup-node-install` or an equivalent action is used in the setup job before saving the cache
- All jobs in a workflow run use a single OS (cache key is scoped to `runner.os`)
