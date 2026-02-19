# Shared Workflows

Reusable GitHub Actions workflows for CI and application security scanning. Call them from any repo with a few lines of YAML.

## Workflows

### Build & Test

Standard Node.js build and test pipeline.

```yaml
jobs:
  build:
    uses: MenaceLabs/shared-workflows/.github/workflows/build-and-test.yml@main
    with:
      node-version: '20'
      build-command: 'npm run build'
      test-command: 'npm test'        # optional, skipped if empty
      artifact-path: 'dist'           # optional, skipped if empty
```

| Input | Default | Description |
|-------|---------|-------------|
| `node-version` | `'20'` | Node.js version |
| `build-command` | `'npm run build'` | Build command to run |
| `test-command` | `''` | Test command (empty to skip) |
| `artifact-path` | `''` | Build output to upload as artifact (empty to skip) |

### AppSec Pipeline

Four security checks running in parallel, all zero-cost.

```yaml
jobs:
  appsec:
    uses: MenaceLabs/shared-workflows/.github/workflows/appsec.yml@main
    with:
      node-version: '20'
    secrets: inherit
```

| Job | Tool | What it does |
|-----|------|-------------|
| Dependency Audit | `npm audit` + [OSV-Scanner](https://github.com/google/osv-scanner) | Checks dependencies for known vulnerabilities |
| Secret Detection | [Gitleaks](https://github.com/gitleaks/gitleaks) | Scans git history for leaked credentials |
| SAST Analysis | [Bearer](https://github.com/Bearer/bearer) | Static application security testing |
| License Compliance | [license-checker](https://github.com/davglass/license-checker) | Flags copyleft licenses (GPL-3.0, AGPL-3.0) |

Each job can be toggled off individually:

| Input | Default | Description |
|-------|---------|-------------|
| `node-version` | `'20'` | Node.js version |
| `run-sast` | `true` | Run SAST scanning |
| `run-dependency-audit` | `true` | Run dependency audit |
| `run-secret-scan` | `true` | Run secret detection |
| `run-license-check` | `true` | Run license compliance check |

All jobs use `continue-on-error: true`, so they report findings without blocking your pipeline.

## Secrets

| Secret | Required by | Notes |
|--------|------------|-------|
| `GITLEAKS_LICENSE` | Secret Detection | Optional. Gitleaks runs without it but some features are limited. Pass via `secrets: inherit`. |

## Full Example

A calling repo's workflow file:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read

jobs:
  build:
    uses: MenaceLabs/shared-workflows/.github/workflows/build-and-test.yml@main
    with:
      node-version: '20'
      build-command: 'npm run build'
      artifact-path: 'dist'

  appsec:
    uses: MenaceLabs/shared-workflows/.github/workflows/appsec.yml@main
    with:
      node-version: '20'
    secrets: inherit
```
