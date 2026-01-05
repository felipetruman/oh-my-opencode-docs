---
layout: default
title: CI CD Pipeline
parent: Development
nav_order: 1
---

# CI/CD Pipeline

> **Relevant source files**
> * [.github/workflows/ci.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml)
> * [.github/workflows/publish.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml)
> * [script/generate-changelog.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts)
> * [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts)

## Purpose and Scope

This document describes the continuous integration and deployment workflows implemented via GitHub Actions. It covers automated testing, type checking, build verification, and the release publication process. For information about the local build system and tooling, see [Build System](/code-yeongyu/oh-my-opencode/11.1-experimental-features). For the manual release process and versioning strategy, see [Release Process](/code-yeongyu/oh-my-opencode/11.3-model-configuration).

The CI/CD system consists of two primary workflows:

* **CI Workflow** - Automated quality checks on every push and pull request
* **Publish Workflow** - Manual release workflow for publishing to npm

---

## Overview

### CI/CD Pipeline Architecture

```

```

**Sources:** [.github/workflows/ci.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml)

 [.github/workflows/publish.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml)

---

## CI Workflow

The CI workflow provides automated quality assurance for all code changes.

### Trigger Conditions

The workflow activates on:

* Push events to `master` or `dev` branches [.github/workflows/ci.yml L4-L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L4-L7)
* Pull requests targeting the `master` branch [.github/workflows/ci.yml L6-L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L6-L7)

Concurrency control prevents multiple simultaneous runs per branch, automatically canceling in-progress runs when new commits arrive [.github/workflows/ci.yml L9-L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L9-L11)

### Job Dependencies and Execution

```

```

**Sources:** [.github/workflows/ci.yml L14-L74](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L14-L74)

### Test Job

Executes the test suite using Bun:

| Step | Command | Purpose |
| --- | --- | --- |
| Setup Bun | `oven-sh/setup-bun@v2` | Install latest Bun runtime |
| Install dependencies | `bun install` | Install packages with `@ast-grep/napi` allowed |
| Run tests | `bun test` | Execute test suite |

The `BUN_INSTALL_ALLOW_SCRIPTS` environment variable permits installation scripts only for `@ast-grep/napi` [.github/workflows/ci.yml L25-L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L25-L26)

**Sources:** [.github/workflows/ci.yml L14-L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L14-L29)

### Typecheck Job

Validates TypeScript type correctness:

| Step | Command | Purpose |
| --- | --- | --- |
| Setup Bun | `oven-sh/setup-bun@v2` | Install latest Bun runtime |
| Install dependencies | `bun install` | Install packages |
| Type check | `bun run typecheck` | Run TypeScript compiler in check-only mode |

**Sources:** [.github/workflows/ci.yml L31-L46](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L31-L46)

### Build Job

Performs a full build and validates output artifacts [.github/workflows/ci.yml L48-L74](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L48-L74)

:

1. **Dependencies:** Requires successful completion of both `test` and `typecheck` jobs [.github/workflows/ci.yml L50](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L50-L50)
2. **Build Execution:** Runs `bun run build` which invokes the build script [.github/workflows/ci.yml L68](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L68-L68)
3. **Output Verification:** Checks for required artifacts: * `dist/index.js` - Bundled JavaScript output * `dist/index.d.ts` - TypeScript declarations

**Sources:** [.github/workflows/ci.yml L48-L74](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L48-L74)

### Schema Auto-Commit

On pushes to `master`, the workflow automatically commits schema changes [.github/workflows/ci.yml L75-L86](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L75-L86)

:

```

```

This ensures `assets/oh-my-opencode.schema.json` stays synchronized with code changes, particularly when Zod schemas are modified. The commit uses the `github-actions[bot]` identity [.github/workflows/ci.yml L81-L82](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L81-L82)

**Sources:** [.github/workflows/ci.yml L75-L86](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L75-L86)

### Draft Release Creation

On pushes to `dev`, the workflow generates or updates a draft release [.github/workflows/ci.yml L88-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L88-L134)

:

1. **Changelog Generation:** Executes `script/generate-changelog.ts` to create release notes
2. **Draft Management:** Creates new draft release tagged as `next` or updates existing draft
3. **Release Notes:** Populated with commit history and contributor attribution

The draft release provides a preview of upcoming changes before the formal release process.

**Sources:** [.github/workflows/ci.yml L88-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L88-L134)

 [script/generate-changelog.ts L16-L35](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L16-L35)

---

## Publish Workflow

The publish workflow handles manual releases to npm with full version management, changelog generation, and GitHub release creation.

### Workflow Inputs

Manual execution via `workflow_dispatch` with two inputs [.github/workflows/publish.yml L5-L18](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L5-L18)

:

| Input | Type | Required | Options | Description |
| --- | --- | --- | --- | --- |
| `bump` | choice | Yes | `major`, `minor`, `patch` | Semantic version component to increment |
| `version` | string | No | - | Override version (bypasses bump calculation) |

### Job Pipeline

```

```

**Sources:** [.github/workflows/publish.yml L26-L141](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L26-L141)

### Build Process

The publish workflow performs a manual build rather than using `bun run build` [.github/workflows/publish.yml L104-L112](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L104-L112)

:

```

```

This explicit build command ensures:

* ESM format output
* External handling of `@ast-grep/napi` binary dependency
* Declaration file generation
* JSON schema regeneration

**Sources:** [.github/workflows/publish.yml L104-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L104-L117)

### Publish Script Integration

The workflow invokes `script/publish.ts` with environment variables [.github/workflows/publish.yml L119-L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L119-L126)

:

| Environment Variable | Purpose |
| --- | --- |
| `BUMP` | Semantic version component (`major`, `minor`, `patch`) |
| `VERSION` | Optional version override |
| `CI` | Enables CI-specific behavior (provenance, git operations) |
| `GITHUB_TOKEN` | Authentication for GitHub API operations |
| `NPM_CONFIG_PROVENANCE` | Enables npm provenance for supply chain security |

**Sources:** [.github/workflows/publish.yml L119-L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L119-L126)

### Post-Publish Operations

After successful npm publication, the workflow performs cleanup and branch synchronization [.github/workflows/publish.yml L128-L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L140)

:

1. **Draft Release Deletion:** Removes the `next` draft release created by CI workflow
2. **Master Branch Sync:** Force-resets `master` branch to the newly tagged version

```

```

**Sources:** [.github/workflows/publish.yml L128-L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L140)

---

## Supporting Scripts

### publish.ts

The `script/publish.ts` script orchestrates the complete publishing workflow [script/publish.ts L1-L183](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L1-L183)

#### Version Management

```

```

The `bumpVersion()` function implements semantic versioning logic [script/publish.ts L24-L34](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L24-L34)

:

* `major`: Increments major version, resets minor and patch to 0
* `minor`: Increments minor version, resets patch to 0
* `patch`: Increments patch version

**Sources:** [script/publish.ts L11-L42](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L11-L42)

 [script/publish.ts L153-L170](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L153-L170)

#### Changelog Generation

The script generates release notes by analyzing git history [script/publish.ts L44-L66](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L44-L66)

:

```

```

Commit filtering rules [script/publish.ts L51](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L51-L51)

:

* **Excluded prefixes:** `ignore:`, `test:`, `chore:`, `ci:`, `release:`
* **Included:** All other commits with commit hash and message

**Sources:** [script/publish.ts L44-L66](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L44-L66)

#### Contributor Attribution

The `getContributors()` function queries the GitHub API to identify external contributors [script/publish.ts L68-L107](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L68-L107)

:

```

```

**Sources:** [script/publish.ts L68-L107](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L68-L107)

#### NPM Publishing

The `buildAndPublish()` function publishes to npm with provenance support [script/publish.ts L109-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L109-L117)

:

```

```

The `--ignore-scripts` flag prevents `prepublishOnly` from re-running since the build already completed in the workflow. The `--provenance` flag enables npm's supply chain attestation when running in GitHub Actions with OIDC.

**Sources:** [script/publish.ts L109-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L109-L117)

#### Git Operations

The `gitTagAndRelease()` function handles version control operations [script/publish.ts L119-L151](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L119-L151)

:

| Operation | Command | Purpose |
| --- | --- | --- |
| Configure git | `git config user.email/name` | Set bot identity |
| Commit version | `git commit -m "release: v$VERSION"` | Record version bump |
| Create tag | `git tag v$VERSION` | Tag release commit |
| Push changes | `git push origin HEAD --tags` | Sync to remote |
| GitHub release | `gh release create v$VERSION` | Create public release |

**Sources:** [script/publish.ts L119-L151](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L119-L151)

### generate-changelog.ts

The `script/generate-changelog.ts` script generates changelog content for draft releases [script/generate-changelog.ts L1-L92](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L1-L92)

#### Latest Release Detection

```

```

The script uses `gh release list` with filters [script/generate-changelog.ts L7-L13](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L7-L13)

:

* `--exclude-drafts` - Ignore draft releases
* `--exclude-pre-releases` - Ignore pre-releases
* `--limit 1` - Get only the most recent release

**Sources:** [script/generate-changelog.ts L7-L14](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L7-L14)

#### Commit Analysis

Identical to `publish.ts` commit filtering, with the same exclusion patterns [script/generate-changelog.ts L16-L35](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L16-L35)

:

```

```

**Sources:** [script/generate-changelog.ts L16-L35](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L16-L35)

#### Contributor Detection

Uses GitHub API to compare commit ranges and extract contributor information [script/generate-changelog.ts L37-L71](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L37-L71)

:

```

```

Contributors are grouped by username with their commit titles listed beneath. Team members (`actions-user`, `github-actions[bot]`, `code-yeongyu`) are excluded [script/generate-changelog.ts L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L5-L5)

**Sources:** [script/generate-changelog.ts L37-L71](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L37-L71)

---

## Concurrency and Permissions

### Concurrency Control

Both workflows implement concurrency management [.github/workflows/ci.yml L9-L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L9-L11)

 [.github/workflows/publish.yml L20](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L20-L20)

:

| Workflow | Group | Behavior |
| --- | --- | --- |
| CI | `${{ github.workflow }}-${{ github.ref }}` | Cancel in-progress runs on new push |
| Publish | `${{ github.workflow }}-${{ github.ref }}` | Queue sequential executions |

This prevents race conditions and resource waste when multiple commits are pushed rapidly.

### Required Permissions

#### CI Workflow Permissions

[.github/workflows/ci.yml L51-L52](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L51-L52)

 [.github/workflows/ci.yml L92-L93](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L92-L93)

| Permission | Scope | Purpose |
| --- | --- | --- |
| `contents: write` | `build` job | Auto-commit schema changes |
| `contents: write` | `draft-release` job | Create/update draft releases |

#### Publish Workflow Permissions

[.github/workflows/publish.yml L22-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L22-L24)

| Permission | Scope | Purpose |
| --- | --- | --- |
| `contents: write` | All jobs | Git tag creation, release publishing |
| `id-token: write` | `publish` job | OIDC authentication for npm provenance |

The `id-token: write` permission enables GitHub's OIDC provider to generate short-lived tokens for npm's provenance verification, providing cryptographic proof of the package's build origin.

**Sources:** [.github/workflows/ci.yml L9-L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L9-L11)

 [.github/workflows/ci.yml L51-L52](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L51-L52)

 [.github/workflows/ci.yml L92-L93](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L92-L93)

 [.github/workflows/publish.yml L20-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L20-L24)

---

## Environment Configuration

### Node.js and Bun Setup

All workflows use consistent runtime versions:

| Runtime | Version | Purpose |
| --- | --- | --- |
| Bun | `latest` | Build and test execution |
| Node.js | `24` | npm publishing (publish workflow only) |

The publish workflow upgrades npm to the latest version for OIDC trusted publishing support [.github/workflows/publish.yml L80-L81](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L80-L81)

**Sources:** [.github/workflows/ci.yml L19-L21](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L19-L21)

 [.github/workflows/publish.yml L32-L34](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L32-L34)

 [.github/workflows/publish.yml L72-L81](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L72-L81)

### Dependency Installation

Installation uses a restricted allow-list for post-install scripts:

```

```

This security measure prevents arbitrary code execution during installation, permitting only the `@ast-grep/napi` package which requires native compilation.

**Sources:** [.github/workflows/ci.yml L24-L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L24-L26)

 [.github/workflows/publish.yml L36-L39](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L36-L39)

 [.github/workflows/publish.yml L86-L89](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L86-L89)

### Repository Conditions

The publish workflow includes a repository check [.github/workflows/publish.yml L64](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L64-L64)

:

```

```

This prevents accidental publishing from forks, ensuring only the canonical repository can publish to npm.

**Sources:** [.github/workflows/publish.yml L61-L64](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L61-L64)