---
layout: default
title: Release Process
parent: Development
nav_order: 1
---

# Release Process

> **Relevant source files**
> * [.github/workflows/ci.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml)
> * [.github/workflows/publish.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml)
> * [script/generate-changelog.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts)
> * [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts)

This document describes the manual release workflow used to publish new versions of oh-my-opencode to npm. The release process includes version bumping, changelog generation from git commits, npm publishing with provenance, and GitHub release creation.

For information about the automated CI pipeline that runs on every push, see [CI/CD Pipeline](/code-yeongyu/oh-my-opencode/11.2-keyword-modes). For information about the build system that compiles the plugin, see [Build System](/code-yeongyu/oh-my-opencode/11.1-experimental-features).

## Overview

The release process is entirely manual and triggered via GitHub Actions workflow dispatch. It follows a linear pipeline:

1. **Pre-checks**: Run tests and type checking
2. **Version determination**: Fetch latest npm version and calculate new version
3. **Build**: Compile TypeScript and generate schema
4. **Publish**: Push to npm registry with provenance
5. **Git operations**: Commit version bump, create tag, push to repository
6. **GitHub release**: Create release with auto-generated changelog
7. **Branch sync**: Force-update `master` branch to match release tag

**Release Pipeline Flow**

```

```

Sources: [.github/workflows/publish.yml L1-L141](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L1-L141)

 [script/publish.ts L1-L184](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L1-L184)

## Manual Release Workflow

The publish workflow is defined in `.github/workflows/publish.yml` and is triggered via GitHub's web interface or API. It accepts two inputs:

| Input | Type | Required | Options | Description |
| --- | --- | --- | --- | --- |
| `bump` | choice | Yes | `major`, `minor`, `patch` | Semantic version component to increment |
| `version` | string | No | Any valid semver | Override calculated version (skips bump logic) |

The workflow only runs on the canonical repository (`code-yeongyu/oh-my-opencode`) to prevent forks from publishing unauthorized versions.

**Workflow Input and Job Dependencies**

```

```

Sources: [.github/workflows/publish.yml L4-L25](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L4-L25)

 [.github/workflows/publish.yml L61-L85](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L61-L85)

### Permissions

The workflow requires specific GitHub permissions:

* `contents: write` - For creating commits, tags, and releases
* `id-token: write` - For npm provenance attestation via OIDC

Sources: [.github/workflows/publish.yml L22-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L22-L24)

## Version Bumping Strategy

Version determination is handled by [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts)

 and follows this logic:

1. **Fetch previous version**: Query npm registry at `https://registry.npmjs.org/oh-my-opencode/latest`
2. **Calculate new version**: Apply semantic versioning bump or use override
3. **Check existence**: Verify the new version doesn't already exist on npm
4. **Update package.json**: Modify version field via string replacement

**Version Calculation Flow**

```

```

Sources: [script/publish.ts L11-L42](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L11-L42)

 [script/publish.ts L153-L180](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L153-L180)

### Version Bump Implementation

The `bumpVersion()` function implements semantic versioning rules:

```

```

| Bump Type | Example | Effect |
| --- | --- | --- |
| `major` | `1.5.3` → `2.0.0` | Breaking changes |
| `minor` | `1.5.3` → `1.6.0` | New features |
| `patch` | `1.5.3` → `1.5.4` | Bug fixes |

Sources: [script/publish.ts L24-L34](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L24-L34)

## Changelog Generation

Changelog generation extracts commit messages and contributor information from git history. The process is implemented in both [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts)

 (for releases) and [script/generate-changelog.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts)

 (for draft releases).

**Changelog Generation Pipeline**

```

```

Sources: [script/publish.ts L44-L107](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L44-L107)

 [script/generate-changelog.ts L16-L71](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L16-L71)

### Commit Filtering Rules

Commits are filtered by prefix to exclude non-user-facing changes:

| Prefix | Excluded | Reason |
| --- | --- | --- |
| `ignore:` | ✓ | Internal notes |
| `test:` | ✓ | Test code changes |
| `chore:` | ✓ | Maintenance tasks |
| `ci:` | ✓ | CI configuration |
| `release:` | ✓ | Automated release commits |

Sources: [script/publish.ts L51](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L51-L51)

 [script/generate-changelog.ts L23](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L23-L23)

### Contributor Attribution

Contributors are identified via the GitHub API and filtered against a team list:

```

```

The resulting changelog includes a section thanking external contributors with their usernames and commit titles.

Sources: [script/publish.ts L68-L107](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L68-L107)

 [script/generate-changelog.ts L5-L71](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L5-L71)

## npm Publishing

The plugin is published to the npm registry with provenance attestation. The publishing step includes validation to prevent duplicate versions and ensures the build artifacts exist.

**npm Publish Workflow**

```

```

Sources: [.github/workflows/publish.yml L104-L127](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L104-L127)

 [script/publish.ts L109-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L109-L117)

### Provenance Attestation

When running in CI, the publish step includes `--provenance` to generate OIDC-signed attestations linking the package to its source code and build process:

```

```

This requires:

* npm version 9.5.0+ (upgraded in workflow: [.github/workflows/publish.yml L80-L81](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L80-L81) )
* `id-token: write` permission ([.github/workflows/publish.yml L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L24-L24) )
* GitHub OIDC provider configuration

The `--ignore-scripts` flag prevents `prepublishOnly` from re-running the build, as it was already executed in the workflow.

Sources: [.github/workflows/publish.yml L119-L127](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L119-L127)

 [script/publish.ts L109-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L109-L117)

## Git Operations and GitHub Releases

After successful npm publishing, the release script performs git operations and creates a GitHub release.

**Git and Release Operations**

```

```

Sources: [script/publish.ts L119-L151](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L119-L151)

 [.github/workflows/publish.yml L128-L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L140)

### Tag and Release Creation

The script creates an annotated git tag and corresponding GitHub release:

```

```

If a tag or release already exists (e.g., from a retry), the operation is skipped to maintain idempotency.

Sources: [script/publish.ts L133-L150](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L133-L150)

### Master Branch Synchronization

After release creation, the `master` branch is force-updated to match the release tag:

```

```

This ensures the `master` branch always points to the latest stable release, while active development continues on `dev`.

Sources: [.github/workflows/publish.yml L133-L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L133-L140)

## Draft Releases

The CI workflow automatically creates or updates a draft release on the `dev` branch to preview upcoming changes. This draft is deleted when an actual release is published.

**Draft Release Lifecycle**

```

```

Sources: [.github/workflows/ci.yml L88-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L88-L134)

 [script/generate-changelog.ts L1-L93](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L1-L93)

### Draft Release Tag

The draft release uses a special tag name `next` that is not version-based. This allows continuous updates without creating multiple draft releases.

| Property | Value |
| --- | --- |
| Tag name | `next` |
| Title | "Upcoming Changes 🍿" |
| State | Draft (not visible to public) |
| Target | Current `dev` branch SHA |

When a real release is published, the draft is deleted via: [.github/workflows/publish.yml L128-L131](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L131)

Sources: [.github/workflows/ci.yml L115-L132](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L115-L132)

 [.github/workflows/publish.yml L128-L131](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L131)

## Environment Variables

The publish script and workflow rely on several environment variables:

| Variable | Source | Purpose | Used In |
| --- | --- | --- | --- |
| `BUMP` | workflow input | Version bump type (`major`/`minor`/`patch`) | [script/publish.ts L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L6-L6) |
| `VERSION` | workflow input | Version override | [script/publish.ts L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L7-L7) |
| `CI` | GitHub Actions | Enable provenance and git operations | [script/publish.ts L112-L120](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L112-L120) |
| `GITHUB_TOKEN` | GitHub Actions | Authenticate gh CLI | [.github/workflows/publish.yml L125](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L125-L125) |
| `NPM_CONFIG_PROVENANCE` | workflow env | Enable npm provenance | [.github/workflows/publish.yml L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L126-L126) |
| `GH_TOKEN` | GitHub Actions | Authenticate GitHub API calls | [.github/workflows/ci.yml L113-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L113-L134) |

Sources: [script/publish.ts L6-L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L6-L7)

 [.github/workflows/publish.yml L121-L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L121-L126)

 [.github/workflows/ci.yml L113-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L113-L134)

## Build Verification

Before publishing, the workflow verifies that build artifacts exist:

```

```

This catches build failures early, before attempting to publish an incomplete package. The verification runs immediately after the build step and before the publish script execution.

Sources: [.github/workflows/publish.yml L114-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L114-L117)

## Concurrency Control

The publish workflow uses concurrency groups to prevent simultaneous releases:

```

```

This ensures that if multiple releases are triggered simultaneously, only one executes while others are queued. This prevents version conflicts and race conditions in the npm registry.

Sources: [.github/workflows/publish.yml L20](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L20-L20)