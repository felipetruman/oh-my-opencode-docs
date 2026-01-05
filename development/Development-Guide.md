---
layout: default
title: Development Guide
parent: Development
nav_order: 1
---

# Development Guide

> **Relevant source files**
> * [.github/workflows/ci.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml)
> * [.github/workflows/publish.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml)
> * [bun.lock](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock)
> * [package.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json)
> * [script/generate-changelog.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts)
> * [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts)
> * [src/cli/config-manager.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts)
> * [src/shared/jsonc-parser.test.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.test.ts)
> * [src/shared/jsonc-parser.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.ts)

This document provides an overview of the development workflow, build system, and contribution process for oh-my-opencode contributors. It covers the local development environment setup, build pipeline architecture, testing procedures, and the path from code changes to published releases.

For detailed information on specific topics, see:

* Build system configuration and TypeScript compilation: [Build System](/code-yeongyu/oh-my-opencode/11.1-experimental-features)
* Continuous integration workflows and automated testing: [CI/CD Pipeline](/code-yeongyu/oh-my-opencode/11.2-keyword-modes)
* Publishing workflow and version management: [Release Process](/code-yeongyu/oh-my-opencode/11.3-model-configuration)
* Dependency rationale and management: [Dependency Management](/code-yeongyu/oh-my-opencode/11.4-parallel-execution-patterns)

---

## Prerequisites

The project requires the following tools:

| Tool | Purpose | Version |
| --- | --- | --- |
| Bun | Runtime and package manager | latest |
| TypeScript | Type checking | ^5.7.3 |
| Git | Version control | any |
| Node.js | npm publishing (CI only) | 24+ |

**Platform Support**: The project builds on macOS, Linux, and Windows. Native dependencies are automatically installed for the current platform during `bun install`.

**Sources**: [package.json L1-L70](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L1-L70)

---

## Development Workflow Overview

```

```

**Sources**: [package.json L22-L28](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L22-L28)

 [.github/workflows/ci.yml L1-L135](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L1-L135)

 [.github/workflows/publish.yml L1-L141](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L1-L141)

---

## Build System Architecture

The build system uses Bun as the primary build tool, with TypeScript for type generation and a custom schema builder.

```

```

**Key Build Commands**:

| Command | Script | Purpose |
| --- | --- | --- |
| `bun run build` | Runs all build steps | Full compilation pipeline |
| `bun run clean` | `rm -rf dist` | Remove build artifacts |
| `bun run typecheck` | `tsc --noEmit` | Type checking only |
| `bun run prepublishOnly` | `clean && build` | Pre-publish hook |

**External Dependencies**: The build externalizes `@ast-grep/napi` to avoid bundling native binaries. Platform-specific binaries are resolved at runtime via optional dependencies.

**Sources**: [package.json L22-L28](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L22-L28)

 [package.json L50-L69](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L50-L69)

---

## Project File Structure

```

```

**Sources**: [package.json L1-L70](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L1-L70)

 [.github/workflows/ci.yml L1-L135](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L1-L135)

 [.github/workflows/publish.yml L1-L141](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L1-L141)

---

## Testing Strategy

The project uses Bun's built-in test runner for unit tests.

**Test Execution**:

```

```

**Test File Convention**: Tests are co-located with source files using the `.test.ts` suffix (e.g., [src/tools/grep/downloader.test.ts L1-L104](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/grep/downloader.test.ts#L1-L104)

).

**Test Structure Example**:

```

```

**CI Integration**: Tests run automatically on all pushes to `master` and `dev` branches, and on all pull requests ([.github/workflows/ci.yml L14-L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L14-L29)

).

**Sources**: [src/tools/grep/downloader.test.ts L1-L104](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/grep/downloader.test.ts#L1-L104)

 [.github/workflows/ci.yml L14-L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L14-L29)

 [package.json L28](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L28-L28)

---

## Continuous Integration

Two GitHub Actions workflows automate quality checks and releases:

### CI Workflow (.github/workflows/ci.yml)

**Trigger**: Push to `master`/`dev` or pull request to `master`

**Jobs**:

1. **test**: Executes `bun test` with trusted dependencies allowlist
2. **typecheck**: Runs `tsc --noEmit` for type validation
3. **build**: Compiles project and verifies output artifacts exist
4. **draft-release**: (dev branch only) Generates draft release with changelog

**Trusted Dependencies**: The workflow explicitly allows install scripts only for `@ast-grep/napi` via `BUN_INSTALL_ALLOW_SCRIPTS` environment variable ([.github/workflows/ci.yml L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L26-L26)

 [.github/workflows/ci.yml L43](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L43-L43)

).

**Auto-commit Schema**: The build job automatically commits `assets/oh-my-opencode.schema.json` changes on master pushes ([.github/workflows/ci.yml L76-L86](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L76-L86)

).

**Sources**: [.github/workflows/ci.yml L1-L135](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L1-L135)

### Publish Workflow (.github/workflows/publish.yml)

**Trigger**: Manual `workflow_dispatch` with version bump selection

**Inputs**:

* `bump`: Choice of `major`, `minor`, or `patch`
* `version`: Optional explicit version override

**Jobs**:

1. **test**: Pre-release test validation
2. **typecheck**: Pre-release type validation
3. **publish**: Version bump, npm publish, GitHub release creation, master branch sync

**OIDC Publishing**: Uses npm provenance via `NPM_CONFIG_PROVENANCE: true` for supply chain security ([.github/workflows/publish.yml L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L126-L126)

).

**Sources**: [.github/workflows/publish.yml L1-L141](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L1-L141)

---

## Release Process Flow

```

```

**Version Deduplication**: The publish script checks if the target version already exists on npm and skips publishing to prevent conflicts ([script/publish.ts L153-L160](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L153-L160)

 [script/publish.ts L167-L170](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L167-L170)

).

**Draft Release Management**: After publishing, the workflow deletes any existing "next" draft release ([.github/workflows/publish.yml L128-L131](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L131)

).

**Sources**: [script/publish.ts L1-L184](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L1-L184)

 [.github/workflows/publish.yml L61-L141](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L61-L141)

---

## Native Dependencies

The project includes native dependencies that require special handling:

### Trusted Dependencies System

The following packages execute install scripts and are explicitly allowlisted in [package.json L65-L69](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L65-L69)

:

```

```

**Installation**: Always use `BUN_INSTALL_ALLOW_SCRIPTS` to restrict script execution:

```

```

### Ripgrep Runtime Installation

Unlike other native dependencies, ripgrep is **not** installed at package install time. Instead, [src/tools/grep/downloader.ts L125-L173](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/grep/downloader.ts#L125-L173)

 downloads and installs ripgrep to `~/.cache/oh-my-opencode/bin/` on first use.

**Platform Mapping** ([src/tools/grep/downloader.ts L21-L27](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/grep/downloader.ts#L21-L27)

):

| Platform Key | Ripgrep Platform | Archive Format |
| --- | --- | --- |
| `arm64-darwin` | `aarch64-apple-darwin` | tar.gz |
| `arm64-linux` | `aarch64-unknown-linux-gnu` | tar.gz |
| `x64-darwin` | `x86_64-apple-darwin` | tar.gz |
| `x64-linux` | `x86_64-unknown-linux-musl` | tar.gz |
| `x64-win32` | `x86_64-pc-windows-msvc` | zip |

**Sources**: [package.json L65-L69](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L65-L69)

 [src/tools/grep/downloader.ts L19-L173](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/grep/downloader.ts#L19-L173)

 [bun.lock L25-L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L25-L29)

---

## Local Development Workflow

### Initial Setup

```

```

### Development Cycle

```

```

### Testing the Plugin Locally

To test the plugin in OpenCode without publishing:

1. Build the plugin: `bun run build`
2. In your OpenCode configuration, reference the local path: ``` ```

### Before Committing

```

```

**Sources**: [package.json L22-L28](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L22-L28)

 [.github/workflows/ci.yml L23-L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L23-L29)

---

## Dependency Upgrade Strategy

**Lock File**: [bun.lock L1-L116](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L1-L116)

 ensures reproducible builds. Commit lock file changes after dependency updates.

**Native Dependencies**: When upgrading `@ast-grep/cli`, `@ast-grep/napi`, or `@code-yeongyu/comment-checker`, verify CI workflows still execute install scripts correctly.

**OpenCode SDK**: The plugin depends on `@opencode-ai/plugin` ([package.json L54](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L54-L54)

), which includes `@opencode-ai/sdk` transitively. This should track OpenCode's release cycle.

**Upgrade Checklist**:

1. Update version in [package.json L49-L59](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L49-L59)
2. Run `bun install` to update [bun.lock L1-L116](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L1-L116)
3. Run `bun run typecheck` and `bun test`
4. Test in local OpenCode installation
5. Commit both `package.json` and `bun.lock`

**Sources**: [package.json L49-L69](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L49-L69)

 [bun.lock L1-L116](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L1-L116)

---

## Key Development Files

| File | Purpose | When to Edit |
| --- | --- | --- |
| `package.json` | Dependencies, scripts, metadata | Add dependencies, change version, update scripts |
| `tsconfig.json` | TypeScript compilation options | Change target/module, add path mappings |
| `.github/workflows/ci.yml` | CI automation | Change test strategy, add build steps |
| `.github/workflows/publish.yml` | Release automation | Modify release process, change npm config |
| `script/publish.ts` | Version & release logic | Change versioning strategy, modify changelog format |
| `script/build-schema.ts` | JSON Schema generation | Change schema output format |

**Sources**: [package.json L1-L70](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L1-L70)

 [.github/workflows/ci.yml L1-L135](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L1-L135)

 [.github/workflows/publish.yml L1-L141](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L1-L141)

 [script/publish.ts L1-L184](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L1-L184)