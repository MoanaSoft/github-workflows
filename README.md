# MoanaSoft GitHub Workflows

Reusable GitHub Actions workflows maintained independently by MoanaSoft.

This repository is intentionally separate from `ESI-France/github-workflows`.
Patterns and fixes may be ported through **separate reviews and pull requests**,
but no workflow can reference or invoke resources from the other organization.

## Reusable workflows

- `.github/workflows/php-package-ci.yml` — PHP/Composer packages.
- `.github/workflows/laravel-ci.yml` — Laravel applications (PHP tests, formatting, static analysis, optional frontend build, dependency audit).

These shared workflows accept `workflow_call` **only**. They never run on `push`,
`pull_request` or a schedule.

## Manual-only CI policy (October 2026)

Application repositories define a small caller workflow triggered **only** by
`workflow_dispatch`. The operator explicitly selects a commit/ref and starts a
run when an independent CI check is warranted. No other trigger is enabled.

The caller should set minimal read-only permissions, a bounded concurrency
group with `cancel-in-progress: true`, and reference the MoanaSoft reusable
workflow using a **verified immutable 40-character commit SHA**, not `@main`.
The called workflow limits each job to 10–15 minutes.

Example for a backend-only Laravel application:

```yaml
name: Manual CI
on:
  workflow_dispatch:
permissions:
  contents: read
concurrency:
  group: manual-ci-${{ github.repository }}-${{ github.ref }}
  cancel-in-progress: true
jobs:
  qa:
    uses: MoanaSoft/github-workflows/.github/workflows/laravel-ci.yml@<PINNED_40_CHAR_SHA>
    with:
      php-version: "8.4"
      run-lint: true
      run-analysis: true
      run-tests: true
      run-frontend: false
      run-security: true
```

Replace the illustrative SHA placeholder with a real commit from MoanaSoft's
`main` branch before using the example. Configure optional private Composer
GitHub App authentication in the calling repository where required; never put
application tokens, private keys or production environment files in this
public repository.

The Laravel reusable workflow creates a disposable CI `.env`, forces an
in-memory SQLite testing connection in its PHP job and runs PHPUnit with
`--fail-on-warning` by default. Applications requiring other services or
test databases need a reviewed adaptation rather than a silent fallback.

**Quota rule:** provisioning a manual workflow does not trigger a run. Only
an operator's explicit manual dispatch can consume GitHub Actions minutes.

## Coordinating the two organizations

Reusable workflow implementations and revision histories are independent.
For a shared generic improvement, prepare separate MoanaSoft and ESI France
changes with their own review, tests, release SHA and rollback. Do not copy
organization-specific secrets or internal infrastructure details to this
public repository. No cross-organization runtime dependency is permitted.
