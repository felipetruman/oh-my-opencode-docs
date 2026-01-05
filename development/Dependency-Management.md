---
layout: default
title: Dependency Management
parent: Development
nav_order: 1
---

# Dependency Management

> **Relevant source files**
> * [bun.lock](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock)
> * [package.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json)
> * [src/cli/config-manager.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts)
> * [src/shared/jsonc-parser.test.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.test.ts)
> * [src/shared/jsonc-parser.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.ts)

This document covers the dependency structure, package management strategy, and dynamic plugin installation workflow for oh-my-opencode. It focuses on the project's use of Bun as a package manager, the categorization of dependencies, and the automated installation of external authentication plugins.

For information about the build process that uses these dependencies, see [Build System](/code-yeongyu/oh-my-opencode/12.1-build-system). For CI/CD automation that validates dependencies, see [CI/CD Pipeline](/code-yeongyu/oh-my-opencode/12.2-cicd-pipeline).

## Package Manager: Bun

oh-my-opencode uses [Bun](https://bun.sh) as its package manager and runtime, rather than npm or yarn. This choice provides several advantages:

* **Fast installation**: Bun's native implementation provides faster dependency resolution and installation
* **Built-in TypeScript support**: No need for ts-node or tsx for running TypeScript directly
* **Compatible lockfile**: [bun.lock L1-L132](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L1-L132)  uses a JSON format that tracks exact versions and integrity hashes

The project explicitly uses Bun commands in build scripts [package.json L26-L31](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L31)

 and the installation workflow [src/cli/config-manager.ts L294-L306](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L294-L306)

**Sources:** [package.json L1-L78](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L1-L78)

 [bun.lock L1-L132](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L1-L132)

## Dependency Categories

### Dependency Structure Overview

```

```

**Sources:** [package.json L52-L77](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L52-L77)

### Production Dependencies

These dependencies are required at runtime when oh-my-opencode is executing as an OpenCode plugin or CLI tool:

| Package | Version | Purpose |
| --- | --- | --- |
| `@opencode-ai/plugin` | ^1.0.162 | Plugin API types and utilities |
| `@opencode-ai/sdk` | ^1.0.162 | Core OpenCode SDK for session management |
| `@ast-grep/cli` | ^0.40.0 | CLI for AST-based code search (25 languages) |
| `@ast-grep/napi` | ^0.40.0 | Native bindings for AST manipulation |
| `@clack/prompts` | ^0.11.0 | Interactive CLI prompts for installation |
| `@code-yeongyu/comment-checker` | ^0.6.1 | Native binary for detecting code comments |
| `@openauthjs/openauth` | ^0.4.3 | OAuth flow implementation for auth plugins |
| `commander` | ^14.0.2 | CLI command parsing and routing |
| `hono` | ^4.10.4 | Lightweight HTTP server for OAuth callbacks |
| `jsonc-parser` | ^3.3.1 | JSONC parsing with comment/trailing comma support |
| `picocolors` | ^1.1.1 | Terminal color output |
| `picomatch` | ^4.0.2 | Glob pattern matching for conditional rules |
| `xdg-basedir` | ^5.1.0 | Cross-platform config directory resolution |
| `zod` | ^4.1.8 | Runtime type validation for configurations |

**Sources:** [package.json L52-L66](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L52-L66)

### Development Dependencies

These dependencies are used only during development and build processes:

| Package | Version | Purpose |
| --- | --- | --- |
| `bun-types` | latest | Type definitions for Bun runtime APIs |
| `typescript` | ^5.7.3 | TypeScript compiler for type checking and declaration generation |
| `@types/picomatch` | ^3.0.2 | Type definitions for picomatch |

The build process [package.json L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L26)

 uses TypeScript for type checking (`tsc --noEmit`) and declaration file generation (`tsc --emitDeclarationOnly`), while Bun handles the actual JavaScript bundling.

**Sources:** [package.json L68-L72](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L68-L72)

### Trusted Dependencies

oh-my-opencode explicitly marks certain dependencies as trusted [package.json L73-L77](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L73-L77)

 allowing them to run post-install scripts. This is a security feature in Bun that prevents arbitrary code execution during installation:

```

```

These packages require trust because they:

* **Download platform-specific native binaries** during installation (AST-grep and comment-checker have separate packages for darwin-arm64, linux-x64-gnu, win32-x64-msvc, etc.)
* **Execute post-install scripts** to set up native addons

The lockfile [bun.lock L36-L70](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L36-L70)

 shows the optional dependencies for each platform variant, ensuring the correct native binary is installed.

**Sources:** [package.json L73-L77](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L73-L77)

 [bun.lock L36-L70](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L36-L70)

## Key Dependencies Deep Dive

### OpenCode SDK Integration

The `@opencode-ai/sdk` and `@opencode-ai/plugin` packages form the foundation of the plugin system:

```

```

The version constraint `^1.0.162` allows patch and minor updates but locks the major version. Both packages are maintained in sync as [package.json L58-L59](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L58-L59)

 shows the same version.

**Sources:** [package.json L58-L59](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L58-L59)

 [bun.lock L80-L82](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L80-L82)

### AST-Grep Native Bindings

AST-grep provides both CLI and programmatic access to code manipulation:

```

```

The `--external @ast-grep/napi` flag in the build command [package.json L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L26)

 ensures the native binding is not bundled, preventing issues with native module resolution.

**Sources:** [package.json L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L26)

 [package.json L53-L54](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L53-L54)

 [bun.lock L36-L70](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L36-L70)

### Authentication and OAuth Stack

The OAuth implementation for Gemini (Antigravity) and ChatGPT (Codex) auth plugins uses a minimal HTTP server:

```

```

The `google-auth.ts` module is exported separately [package.json L19-L22](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L19-L22)

 to allow external auth plugins to reuse the OAuth server implementation.

**Sources:** [package.json L57](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L57-L57)

 [package.json L61](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L61-L61)

 [package.json L19-L22](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L19-L22)

 [bun.lock L78](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L78-L78)

### Configuration Parsing

JSONC support is critical for user-friendly configuration files with comments:

```

```

The implementation [src/shared/jsonc-parser.ts L9-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.ts#L9-L24)

 wraps the library's API to provide strict error handling for configuration files.

**Sources:** [package.json L62](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L62-L62)

 [src/shared/jsonc-parser.ts L1-L67](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.ts#L1-L67)

## Version Management Strategy

### Version Constraint Patterns

oh-my-opencode uses semantic versioning constraints:

| Constraint | Example | Allows | Used For |
| --- | --- | --- | --- |
| `^` (caret) | `^1.0.162` | Minor and patch | Most dependencies |
| `latest` | `latest` | All updates | Development tools (bun-types) |

The caret (`^`) constraint is used for all production dependencies, allowing automatic patch and minor updates but preventing breaking changes from major version bumps.

**Sources:** [package.json L52-L66](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L52-L66)

### Dynamic Version Resolution

The CLI installation process fetches the latest versions of auth plugins dynamically:

```

```

This function [src/cli/config-manager.ts L17-L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L17-L26)

 queries the NPM registry to get the latest published version, ensuring users always install the newest auth plugin versions during setup.

**Sources:** [src/cli/config-manager.ts L17-L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L17-L26)

### Lockfile and Integrity

The `bun.lock` file [bun.lock L1-L132](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L1-L132)

 maintains:

* **Exact versions**: Every dependency and transitive dependency pinned
* **Integrity hashes**: SHA-512 checksums for package verification
* **Platform constraints**: OS and CPU architecture requirements
* **Optional dependencies**: Platform-specific native binaries

Example lockfile entry structure:

```

```

**Sources:** [bun.lock L36](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock#L36-L36)

## Dynamic Plugin Installation

### Auth Plugin Installation Flow

The `oh-my-opencode install` command dynamically installs authentication plugins based on detected subscriptions:

```

```

**Sources:** [src/cli/config-manager.ts L243-L271](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L243-L271)

 [src/cli/config-manager.ts L294-L306](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L294-L306)

### Auth Plugin Version Management

The `addAuthPlugins` function [src/cli/config-manager.ts L243-L271](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L243-L271)

 implements two different strategies:

#### Antigravity (Gemini) - Latest Version

```

```

This fetches the latest version from NPM and pins it explicitly.

#### Codex (ChatGPT) - GitHub Hotfix

```

```

The ChatGPT plugin uses a special GitHub reference defined in [src/cli/config-manager.ts L15](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L15-L15)

:

```

```

This hotfix is applied via `setupChatGPTHotfix()` [src/cli/config-manager.ts L273-L292](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L273-L292)

 which creates a `package.json` in the OpenCode config directory [src/cli/config-manager.ts L10](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L10-L10)

 with a direct GitHub dependency.

**Sources:** [src/cli/config-manager.ts L243-L292](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L243-L292)

### Provider Configuration Injection

After installing plugins, the system injects provider configuration into `opencode.json`:

```

```

The Antigravity configuration [src/cli/config-manager.ts L308-L349](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L308-L349)

 defines complete model specifications including:

* Thinking support flags
* Attachment capabilities
* Context and output token limits
* Input/output modalities (text, image, PDF)

The Codex configuration [src/cli/config-manager.ts L351-L362](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L351-L362)

 specifies OpenAI models with thinking flags for reasoning models (o3, o4-mini).

**Sources:** [src/cli/config-manager.ts L308-L391](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L308-L391)

## Installation Workflow

### Complete Dependency Installation Process

```

```

The `runBunInstall()` function [src/cli/config-manager.ts L294-L306](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L294-L306)

 spawns a Bun subprocess in the OpenCode config directory:

```

```

This installs the auth plugins locally in the OpenCode configuration directory, separate from the main oh-my-opencode installation.

**Sources:** [src/cli/config-manager.ts L294-L306](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L294-L306)

 [src/cli/config-manager.ts L7-L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L7-L11)

## Dependency Update Strategy

### Automated Updates in CI

The CI workflow [referenced in section 12.2] runs tests and typechecks on every commit, ensuring compatibility with the current dependency versions. The schema generation step [package.json L27](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L27-L27)

 also runs during CI, validating that configuration type definitions remain consistent.

### Manual Version Updates

To update dependencies:

1. **Update package.json constraints** if needed
2. **Run `bun update`** to resolve new versions
3. **Verify lockfile changes** in `bun.lock`
4. **Run test suite** with `bun test` [package.json L31](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L31-L31)
5. **Run typecheck** with `bun run typecheck` [package.json L30](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L30-L30)

The project pins OpenCode SDK versions precisely (`^1.0.162`) to ensure compatibility, as the SDK evolves rapidly during development.

**Sources:** [package.json L25-L31](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L25-L31)

 [package.json L58-L59](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L58-L59)