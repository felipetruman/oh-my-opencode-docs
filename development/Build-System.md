---
layout: default
title: Build System
parent: Development
nav_order: 1
---

# Build System

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

The build system transforms the TypeScript source code in `src/` into distributable JavaScript artifacts in `dist/`, generates TypeScript declaration files, and produces the JSON schema for configuration validation. This page covers the build scripts, compilation process, output artifacts, and schema generation.

For information about CI/CD pipeline integration, see [CI/CD Pipeline](/code-yeongyu/oh-my-opencode/12.2-cicd-pipeline). For release process and npm publishing, see [Release Process](/code-yeongyu/oh-my-opencode/12.3-release-process).

---

## Build Overview

The build system uses Bun as both the package manager and JavaScript runtime, with TypeScript for type checking and declaration generation. The build process produces three main entry points plus a JSON schema.

**Build System Architecture**

```

```

Sources: [package.json L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L26)

---

## Build Scripts

The `package.json` defines five npm scripts for build and validation tasks.

| Script | Command | Purpose |
| --- | --- | --- |
| `build` | Composite script | Runs all build steps in sequence |
| `build:schema` | `bun run script/build-schema.ts` | Generates JSON schema from TypeScript types |
| `clean` | `rm -rf dist` | Removes build artifacts |
| `prepublishOnly` | `bun run clean && bun run build` | Automatic pre-publish cleanup and rebuild |
| `typecheck` | `tsc --noEmit` | Type checking without emitting files |
| `test` | `bun test` | Runs test suite |

### Main Build Script

The `build` script executes four sequential operations:

```

```

**Build Script Breakdown:**

1. **Main bundle compilation**: `bun build src/index.ts src/google-auth.ts --outdir dist --target bun --format esm --external @ast-grep/napi` * Compiles plugin entry and OAuth server * Target: Bun runtime * Format: ES modules * External: `@ast-grep/napi` (native addon, cannot be bundled)
2. **Declaration generation**: `tsc --emitDeclarationOnly` * Generates `.d.ts` files for TypeScript consumers * Controlled by `tsconfig.json`
3. **CLI compilation**: `bun build src/cli/index.ts --outdir dist/cli --target bun --format esm` * Separate bundle for CLI executable * No declaration files needed (not exposed as library)
4. **Schema generation**: `bun run build:schema` * Generates `dist/oh-my-opencode.schema.json` * See [Schema Generation](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/Schema Generation)  section

Sources: [package.json L26-L27](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L27)

---

## Build Process Flow

**Complete Build Pipeline**

```

```

Sources: [package.json L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L26)

 [.github/workflows/publish.yml L104-L113](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L104-L113)

---

## Output Artifacts

The build process produces the following artifacts in the `dist/` directory:

### Published Files

The `package.json` `files` field specifies that only `dist/` is included in the npm package:

```

```

### Entry Points

The `package.json` defines three module exports:

| Export | Types | Import | Description |
| --- | --- | --- | --- |
| `.` | `./dist/index.d.ts` | `./dist/index.js` | Main plugin entry |
| `./google-auth` | `./dist/google-auth.d.ts` | `./dist/google-auth.js` | OAuth server for Gemini |
| `./schema.json` | N/A | `./dist/oh-my-opencode.schema.json` | Configuration schema |

**Entry Point Configuration:**

```

```

### CLI Binary

The CLI is exposed as an executable through the `bin` field:

```

```

This allows users to run `bunx oh-my-opencode` or `npx oh-my-opencode` after installation.

**Distribution Artifact Structure:**

```

```

Sources: [package.json L11-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L11-L24)

 [package.json L8-L10](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L8-L10)

---

## Schema Generation

The `build:schema` script generates a JSON Schema document from Zod validation schemas defined in the codebase. This schema is used for IDE autocomplete and validation in configuration files.

### Schema Build Process

**Schema Generation Flow:**

```

```

The schema is written to two locations:

* **`dist/oh-my-opencode.schema.json`**: Included in npm package for runtime access
* **`assets/oh-my-opencode.schema.json`**: Committed to git repository for GitHub-hosted schema URL

### Schema URL in Configurations

Generated configurations reference the schema via the `$schema` property:

```

```

This enables IDE features like autocomplete and validation when editing `oh-my-opencode.json` files.

Sources: [package.json L27](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L27-L27)

 [src/cli/config-manager.ts L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L140-L140)

---

## TypeScript Configuration

The build process relies on `tsconfig.json` for type checking and declaration generation. Key compiler options:

* **`declaration: true`**: Generate `.d.ts` files
* **`emitDeclarationOnly: true`** (when using `tsc --emitDeclarationOnly`): Only emit declarations, not JavaScript
* **`outDir: "dist"`**: Output directory for compiled files
* **`target: "ES2022"`**: Modern JavaScript features
* **`module: "ES2022"`**: ES module system
* **`moduleResolution: "bundler"`**: Bundler-compatible module resolution

The `tsc --noEmit` command in the `typecheck` script validates types without generating any output files, used in CI for validation.

Sources: [package.json L30-L31](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L30-L31)

---

## External Dependencies

### @ast-grep/napi

The `@ast-grep/napi` package is a native Node.js addon that provides AST parsing functionality. It requires special handling in the build process:

1. **Build exclusion**: Marked as `--external` in Bun build command to prevent bundling
2. **Trusted dependency**: Listed in `trustedDependencies` to allow install scripts
3. **Install-time compilation**: The package compiles native code during `bun install`

**Install Command with Trusted Dependencies:**

```

```

Environment variable in CI:

```

```

This explicitly allows only `@ast-grep/napi` to run install scripts, preventing arbitrary code execution from other dependencies.

Sources: [package.json L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L26)

 [package.json L73-L77](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L73-L77)

 [.github/workflows/ci.yml L24-L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L24-L26)

---

## Build Verification

The CI workflow includes verification steps to ensure build outputs are correctly generated.

### Verification Steps in CI

```

```

These checks ensure:

* Main plugin JavaScript bundle exists
* TypeScript declarations were generated
* CLI executable was created

**Build Verification Flow:**

```

```

The publish workflow includes additional verification steps to debug the build environment before publishing.

Sources: [.github/workflows/ci.yml L70-L73](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L70-L73)

 [.github/workflows/publish.yml L115-L122](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L115-L122)

---

## Pre-Publish Hook

The `prepublishOnly` script ensures a clean build before publishing to npm:

```

```

This lifecycle hook:

1. Removes all previous build artifacts with `bun run clean`
2. Performs a fresh build with `bun run build`
3. Runs automatically before `npm publish` (or `bun publish`)

However, in the CI publish workflow, this is skipped with `--ignore-scripts` to avoid rebuilding after the build has already been verified:

```

```

Sources: [package.json L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L29-L29)

 [script/publish.ts L113](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L113-L113)

---

## Debug Build Information

The publish workflow includes debug output to help diagnose build issues:

```

```

After the build completes, artifact contents are verified:

```

```

This diagnostic information appears in GitHub Actions logs when builds fail, helping identify environment or configuration issues.

Sources: [.github/workflows/publish.yml L91-L102](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L91-L102)

 [.github/workflows/publish.yml L115-L122](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L115-L122)

---

## Summary

The build system transforms TypeScript source into distributable artifacts through a four-step process:

1. **Bundle compilation** with Bun (targeting Bun runtime, ESM format)
2. **Declaration generation** with TypeScript compiler
3. **CLI bundling** as separate executable
4. **Schema generation** from Zod schemas

The system handles native dependencies (@ast-grep/napi) through explicit exclusion and trusted dependency configuration, ensures build correctness through verification checks, and maintains a clean build state through pre-publish hooks.

All build outputs are written to `dist/` and published to npm, with the schema additionally copied to `assets/` for GitHub-hosted URL access.