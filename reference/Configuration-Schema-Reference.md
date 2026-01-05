---
layout: default
title: Configuration Schema Reference
parent: Reference
nav_order: 1
---

# Configuration Schema Reference

> **Relevant source files**
> * [README.ja.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ja.md)
> * [README.ko.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md)
> * [README.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md)
> * [README.zh-cn.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.zh-cn.md)
> * [assets/oh-my-opencode.schema.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json)
> * [src/config/schema.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts)
> * [src/hooks/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts)
> * [src/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts)
> * [src/shared/config-path.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts)

This page provides a complete reference for all configuration options available in `oh-my-opencode.json`. The configuration system uses Zod schemas for runtime validation and generates JSON Schema for IDE autocomplete support.

For information about how configuration is loaded and merged, see [Configuration System](/code-yeongyu/oh-my-opencode/3.2-configuration-system). For agent-specific settings and behavior, see [Agent Reference](/code-yeongyu/oh-my-opencode/12.2-cicd-pipeline).

## Configuration File Locations

oh-my-opencode loads configuration from multiple locations with a defined priority order. Project-level configuration takes precedence over user-level configuration. Both JSON and JSONC (JSON with Comments) formats are supported.

```

```

**Sources:** [src/index.ts L125-L217](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L125-L217)

 [src/shared/config-path.ts L1-L48](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts#L1-L48)

 [src/shared/jsonc-parser.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.ts)

### JSONC Support

Configuration files support JSONC (JSON with Comments) format:

* Line comments: `// comment`
* Block comments: `/* comment */`
* Trailing commas: `{ "key": "value", }`

When both `.jsonc` and `.json` files exist in the same location, the `.jsonc` file takes priority.

```

```

**Sources:** [README.md L715-L743](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L715-L743)

 [src/index.ts L129](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L129-L129)

 [src/shared/jsonc-parser.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.ts)

### Platform-Specific Paths

| Platform | User Config Path | Priority |
| --- | --- | --- |
| **Linux/macOS** | `~/.config/opencode/oh-my-opencode.{jsonc,json}` | Standard |
| **Windows** | `~/.config/opencode/oh-my-opencode.{jsonc,json}` | Preferred |
| **Windows** | `%APPDATA%\opencode\oh-my-opencode.{jsonc,json}` | Fallback |

The Windows implementation checks for the cross-platform path first (`~/.config`) before falling back to `%APPDATA%` for backward compatibility with existing installations. Within each location, `.jsonc` files are checked before `.json` files.

**Sources:** [src/shared/config-path.ts L13-L33](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts#L13-L33)

 [src/index.ts L191-L198](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L191-L198)

### Backward Compatibility Migration

The configuration system automatically migrates legacy agent names during load:

* `omo` / `OmO` → `Sisyphus`
* `OmO-Plan` / `omo-plan` → `Planner-Sisyphus`
* `omo_agent` → `sisyphus_agent`

Migrated configurations are written back to disk to update the file.

```

```

**Sources:** [src/index.ts L62-L123](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L62-L123)

## Schema Structure Overview

The configuration schema is defined using Zod and exported as both TypeScript types and JSON Schema. The JSON Schema is served at `https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json` for IDE autocomplete.

```

```

**Sources:** [script/build-schema.ts L1-L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/build-schema.ts#L1-L29)

 [src/config/index.ts L1-L22](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/index.ts#L1-L22)

 [README.md L717-L722](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L717-L722)

## Top-Level Properties

### disabled_mcps

**Type:** `string[]`
**Valid values:** `"websearch_exa"`, `"context7"`, `"grep_app"`
**Default:** `[]` (all enabled)

Disables specific built-in MCP services. Built-in MCPs are loaded by default and provide external capabilities to agents.

```

```

| MCP | Description | Used By |
| --- | --- | --- |
| `context7` | Official library documentation lookup | librarian |
| `websearch_exa` | Real-time web search via Exa AI | librarian |
| `grep_app` | GitHub code search across public repositories | librarian, explore |

**Sources:** [assets/oh-my-opencode.schema.json L11-L21](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L11-L21)

 [README.md L795-L808](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L795-L808)

### disabled_agents

**Type:** `string[]`
**Valid values:** `"Sisyphus"`, `"oracle"`, `"librarian"`, `"explore"`, `"frontend-ui-ux-engineer"`, `"document-writer"`, `"multimodal-looker"`
**Default:** `[]` (all enabled)

Completely disables specific agents, preventing them from being registered or invoked. Disabled agents cannot be called via `call_omo_agent` or task delegation.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L22-L36](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L22-L36)

 [README.md L797-L805](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L797-L805)

### disabled_hooks

**Type:** `string[]`
**Valid values:** See table below
**Default:** `[]` (all enabled)

Disables specific lifecycle hooks. Hooks intercept events and modify behavior at various stages of the plugin lifecycle.

```

```

**Available Hooks:**

| Hook Name | Event | Purpose |
| --- | --- | --- |
| `todo-continuation-enforcer` | session.idle | Forces completion of TODO items |
| `context-window-monitor` | message.updated | Monitors token usage |
| `session-recovery` | session.error | Recovers from session errors |
| `session-notification` | session.idle | Desktop notifications |
| `comment-checker` | tool.execute.after | Validates code comments |
| `grep-output-truncator` | tool.execute.after | Truncates grep output |
| `tool-output-truncator` | tool.execute.after | Truncates tool outputs |
| `directory-agents-injector` | tool.execute.before | Injects AGENTS.md context |
| `directory-readme-injector` | tool.execute.before | Injects README.md context |
| `empty-task-response-detector` | tool.execute.after | Detects empty task results |
| `think-mode` | chat.message | Enables extended thinking |
| `anthropic-auto-compact` | session.error | Auto-compacts on token limits |
| `rules-injector` | tool.execute.before | Injects conditional rules |
| `background-notification` | session.idle | Background task notifications |
| `auto-update-checker` | session.created | Checks for plugin updates |
| `startup-toast` | session.created | Shows startup message |
| `keyword-detector` | chat.message | Activates specialized modes |
| `agent-usage-reminder` | tool.execute.after | Reminds about agent delegation |
| `non-interactive-env` | tool.execute.before | Handles non-interactive environments |
| `interactive-bash-session` | session.* | Manages tmux sessions |
| `empty-message-sanitizer` | chat.message | Sanitizes empty messages |

**Sources:** [assets/oh-my-opencode.schema.json L37-L65](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L37-L65)

 [README.md L784-L792](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L784-L792)

### google_auth

**Type:** `boolean`
**Default:** `false`

Enables the built-in Google Antigravity OAuth authentication for Gemini models. When `true`, the plugin registers a Google provider with single-account authentication.

```

```

**Note:** The external `opencode-antigravity-auth` plugin is recommended over the built-in auth as it provides multi-account load balancing and more models. When using the external plugin, set this to `false`.

```

```

**Sources:** [README.md L723-L747](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L723-L747)

 [assets/oh-my-opencode.schema.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json)

## Agent Configuration

The `agents` object allows customization of built-in agents. Each agent key corresponds to an agent name, and the value is an `AgentOverrideConfig` object.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L66-L1255](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L66-L1255)

 [README.md L749-L805](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L749-L805)

### Configurable Agents

| Agent Name | Default Model | Purpose |
| --- | --- | --- |
| `build` | (OpenCode default) | Default build agent (demoted to subagent) |
| `plan` | (OpenCode default) | Planning agent (demoted to subagent) |
| `Sisyphus` | `anthropic/claude-opus-4-5` | Primary orchestrator |
| `Planner-Sisyphus` | (inherits from plan) | Planning mode for Sisyphus |
| `oracle` | `openai/gpt-5.2` | Architecture and debugging |
| `librarian` | `anthropic/claude-sonnet-4-5` | Research and documentation |
| `explore` | `opencode/grok-code` | Codebase exploration |
| `frontend-ui-ux-engineer` | `google/gemini-3-pro-preview` | Frontend development |
| `document-writer` | `google/gemini-3-pro-preview` | Technical writing |
| `multimodal-looker` | `google/gemini-3-flash` | Visual content analysis |

**Sources:** [README.md L489-L496](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L489-L496)

 [README.md L749-L765](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L749-L765)

### Agent Properties

#### model

**Type:** `string`
**Required:** No

Specifies the LLM model to use for the agent. Model names follow the provider/model format (e.g., `anthropic/claude-opus-4-5`, `openai/gpt-5.2`).

```

```

**Sources:** [assets/oh-my-opencode.schema.json L72-L74](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L72-L74)

 [README.md L751-L759](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L751-L759)

#### temperature

**Type:** `number`
**Range:** `0` to `2`
**Required:** No

Controls randomness in model responses. Lower values (0.0-0.3) produce more deterministic outputs, while higher values (0.7-2.0) increase creativity and variation.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L75-L79](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L75-L79)

#### top_p

**Type:** `number`
**Range:** `0` to `1`
**Required:** No

Nucleus sampling parameter. Controls diversity by limiting the cumulative probability of considered tokens. Lower values make output more focused.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L80-L84](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L80-L84)

#### prompt

**Type:** `string`
**Required:** No

Overrides the agent's system prompt. This completely replaces the built-in prompt with custom instructions.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L85-L87](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L85-L87)

 [src/config/schema.ts L78](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L78-L78)

#### prompt_append

**Type:** `string`
**Required:** No

Appends additional instructions to the agent's built-in system prompt instead of replacing it. Useful for adding project-specific guidelines while preserving default behavior.

```

```

**Sources:** [src/config/schema.ts L79](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L79-L79)

#### tools

**Type:** `Record<string, boolean>`
**Required:** No

Enables or disables specific tools for the agent. The key is the tool name, and the value is a boolean indicating availability.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L88-L96](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L88-L96)

#### disable

**Type:** `boolean`
**Default:** `false`
**Required:** No

When `true`, completely disables the agent. This is equivalent to adding the agent name to `disabled_agents` but is scoped to the individual agent configuration.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L97-L99](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L97-L99)

 [README.md L761-L763](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L761-L763)

#### description

**Type:** `string`
**Required:** No

Custom description shown in the agent list and used for agent selection contexts. Overrides the built-in description.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L100-L102](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L100-L102)

#### mode

**Type:** `"subagent" | "primary" | "all"`
**Required:** No

Controls when the agent appears in agent selection:

* `"subagent"`: Only available for delegation (via `call_omo_agent`)
* `"primary"`: Only available as main agent selection
* `"all"`: Available in both contexts

```

```

**Sources:** [assets/oh-my-opencode.schema.json L103-L110](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L103-L110)

#### color

**Type:** `string` (hex color)
**Pattern:** `^#[0-9A-Fa-f]{6}$`
**Required:** No

Custom color for the agent in the UI. Must be a 6-digit hex color code.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L111-L114](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L111-L114)

#### permission

**Type:** `PermissionConfig` object
**Required:** No

Fine-grained permission control for the agent. See [Permission System](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/Permission System)

 below.

**Sources:** [assets/oh-my-opencode.schema.json L115-L177](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L115-L177)

## Permission System

The permission system provides fine-grained control over what actions agents can perform. Each permission can be set to `"ask"` (prompt user), `"allow"` (auto-approve), or `"deny"` (block).

```

```

**Sources:** [README.md L773-L795](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L773-L795)

 [assets/oh-my-opencode.schema.json L115-L177](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L115-L177)

### Permission Properties

#### edit

**Type:** `"ask" | "allow" | "deny"`
**Default:** `"ask"`

Controls file editing permissions. Applies to tools like `write_file`, `edit_file`, and LSP refactoring operations.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L118-L125](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L118-L125)

#### bash

**Type:** `"ask" | "allow" | "deny"` OR `Record<string, "ask" | "allow" | "deny">`
**Default:** `"ask"`

Controls bash command execution. Can be a global permission or per-command granular control.

**Global Permission:**

```

```

**Per-Command Permission:**

```

```

**Sources:** [assets/oh-my-opencode.schema.json L126-L151](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L126-L151)

 [README.md L788-L790](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L788-L790)

#### webfetch

**Type:** `"ask" | "allow" | "deny"`
**Default:** `"ask"`

Controls web request permissions. Applies to HTTP requests made via MCPs or built-in tools.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L152-L159](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L152-L159)

#### doom_loop

**Type:** `"ask" | "allow" | "deny"`
**Default:** `"ask"`

Controls whether the agent can override infinite loop detection. When denied, agents are blocked if they appear to be in an endless cycle.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L160-L167](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L160-L167)

#### external_directory

**Type:** `"ask" | "allow" | "deny"`
**Default:** `"ask"`

Controls access to files outside the project root directory. Useful for restricting agents to the current workspace.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L168-L175](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L168-L175)

## Sisyphus Agent Configuration

The Sisyphus agent system controls whether the specialized Sisyphus orchestrator replaces OpenCode's default `build` and `plan` agents. Configuration is via the `sisyphus_agent` object (or legacy `omo_agent`).

### sisyphus_agent

**Type:** `object`
**Properties:**

* `disabled`: `boolean` (default: `false`)
* `default_builder_enabled`: `boolean` (default: `false`)
* `planner_enabled`: `boolean` (default: `true`)
* `replace_plan`: `boolean` (default: `true`)

#### disabled

When `false` (default), Sisyphus orchestration is enabled:

* **Sisyphus** becomes the default agent (primary mode)
* **OpenCode-Builder** (if enabled) runs in subagent mode
* **Planner-Sisyphus** (if enabled) runs in subagent mode
* Original `build` and `plan` are demoted to subagent mode

When `true`, standard OpenCode agents remain primary.

```

```

#### default_builder_enabled

When `true`, registers the `OpenCode-Builder` agent (OpenCode's default build agent with renamed identity). Defaults to `false` to reduce agent clutter.

```

```

#### planner_enabled

When `true` (default), registers the `Planner-Sisyphus` agent based on OpenCode's plan agent with OhMyOpenCode's planning prompt.

```

```

#### replace_plan

When `true` (default), the original `plan` agent is demoted to subagent mode when Sisyphus is enabled. When `false`, `plan` remains available as a primary agent.

```

```

**Sources:** [src/config/schema.ts L115-L120](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L115-L120)

 [src/index.ts L415-L477](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L415-L477)

 [README.md L807-L844](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L807-L844)

### Agent Registration Flow

```

```

**Sources:** [src/index.ts L415-L486](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L415-L486)

### Customizing Sisyphus Agents

Sisyphus-related agents can be customized in the `agents` object:

```

```

**Sources:** [README.md L826-L844](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L826-L844)

 [src/index.ts L420-L477](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L420-L477)

## Claude Code Compatibility

The `claude_code` object provides toggles to disable specific Claude Code compatibility features. All toggles default to `true` (enabled).

```

```

**Sources:** [README.md L654-L676](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L654-L676)

 [assets/oh-my-opencode.schema.json L1297-L1334](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1297-L1334)

### Claude Code Toggles

| Toggle | When `false`, stops loading from... | Unaffected |
| --- | --- | --- |
| `mcp` | `~/.claude/.mcp.json``./.mcp.json``./.claude/.mcp.json` | Built-in MCPs (context7, websearch_exa, grep_app) |
| `commands` | `~/.claude/commands/*.md``./.claude/commands/*.md` | `~/.config/opencode/command/``./.opencode/command/` |
| `skills` | `~/.claude/skills/*/SKILL.md``./.claude/skills/*/SKILL.md` | - |
| `agents` | `~/.claude/agents/*.md``./.claude/agents/*.md` | Built-in agents (oracle, librarian, etc.) |
| `hooks` | `~/.claude/settings.json``./.claude/settings.json``./.claude/settings.local.json` | - |

**Sources:** [README.md L668-L675](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L668-L675)

### Toggle Properties

#### mcp

**Type:** `boolean`
**Default:** `true`

Controls loading of MCP configurations from Claude Code-style `.mcp.json` files. Does not affect built-in MCPs (context7, websearch_exa, grep_app).

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1300-L1303](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1300-L1303)

#### commands

**Type:** `boolean`
**Default:** `true`

Controls loading of slash commands from Claude Code directories (`~/.claude/commands/`, `./.claude/commands/`). Does not affect OpenCode command directories.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1304-L1307](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1304-L1307)

#### skills

**Type:** `boolean`
**Default:** `true`

Controls loading of skills from Claude Code directories (`~/.claude/skills/`, `./.claude/skills/`).

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1308-L1311](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1308-L1311)

#### agents

**Type:** `boolean`
**Default:** `true`

Controls loading of custom agent definitions from Claude Code directories (`~/.claude/agents/*.md`). Does not affect built-in agents.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1312-L1315](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1312-L1315)

#### hooks

**Type:** `boolean`
**Default:** `true`

Controls loading of hooks from Claude Code settings files (`~/.claude/settings.json`, `./.claude/settings.json`, `./.claude/settings.local.json`).

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1316-L1319](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1316-L1319)

## LSP Configuration

The `lsp` object allows configuration of additional LSP servers beyond those configured in OpenCode's main configuration. Each key is an LSP server name, and the value is an LSP server configuration object.

```

```

**Sources:** [README.md L810-L833](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L810-L833)

 [assets/oh-my-opencode.schema.json L1335-L1418](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1335-L1418)

### LSP Server Properties

#### command

**Type:** `string[]`
**Required:** Yes (unless `disabled` is `true`)

Command array to start the LSP server. The first element is the executable, followed by arguments.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1341-L1346](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1341-L1346)

#### extensions

**Type:** `string[]`
**Required:** Yes (unless `disabled` is `true`)

File extensions that this LSP server handles. Extensions should include the leading dot.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1347-L1352](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1347-L1352)

#### priority

**Type:** `number`
**Required:** No
**Default:** `0`

Priority for server selection when multiple servers match the same extension. Higher values take precedence.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1353-L1355](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1353-L1355)

#### env

**Type:** `Record<string, string>`
**Required:** No

Environment variables to set when starting the LSP server.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1356-L1362](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1356-L1362)

#### initialization

**Type:** `object`
**Required:** No

LSP initialization options passed to the server during the initialize request.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1363-L1365](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1363-L1365)

#### disabled

**Type:** `boolean`
**Default:** `false`
**Required:** No

When `true`, disables this LSP server. Useful for temporarily disabling servers without removing configuration.

```

```

**Sources:** [assets/oh-my-opencode.schema.json L1366-L1368](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1366-L1368)

## Comment Checker Configuration

The `comment_checker` object configures the comment checker hook that warns agents about excessive code comments.

### comment_checker

**Type:** `object`
**Properties:**

* `custom_prompt`: `string` (optional)

#### custom_prompt

**Type:** `string`
**Required:** No

Custom warning message to display when excessive comments are detected. Use `{{comments}}` as a placeholder for the detected comments XML.

```

```

**Sources:** [src/config/schema.ts L122-L125](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L122-L125)

## Auto Update Configuration

### auto_update

**Type:** `boolean`
**Default:** `true`

Controls whether the plugin automatically checks for and installs updates. When `true`, the auto-update-checker hook verifies the installed version against npm registry on session start.

```

```

To disable update checks entirely, add `"auto-update-checker"` to `disabled_hooks`:

```

```

**Sources:** [src/config/schema.ts L190](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L190-L190)

 [src/index.ts L281-L286](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L281-L286)

## Experimental Features

The `experimental` object contains features that are under development and may change or be removed in future versions. Use with caution in production environments.

```

```

**Sources:** [src/config/schema.ts L163-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L163-L176)

 [README.md L835-L851](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L835-L851)

### Experimental Properties

#### aggressive_truncation

**Type:** `boolean`
**Default:** `false`

When `true`, enables aggressive truncation of tool outputs when approaching token limits. Truncates more aggressively than the default truncation strategy, with fallback to summarization if still over limit.

```

```

This feature is more aggressive than the built-in `tool-output-truncator` hook. It will truncate tool outputs to fit within the remaining context window, even if that means significant data loss.

**Sources:** [src/config/schema.ts L164](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L164-L164)

#### auto_resume

**Type:** `boolean`
**Default:** `false`

When `true`, automatically resumes execution after successful recovery from thinking block errors or thinking disabled violations. Without this, recovery stops after fixing the error.

```

```

**Warning:** This can lead to unexpected behavior if the agent was in an error state for a reason. Use only when you trust the recovery mechanism.

**Sources:** [src/config/schema.ts L165](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L165-L165)

#### preemptive_compaction

**Type:** `boolean`
**Default:** `true`

Enables preemptive session compaction before hitting hard token limits. When enabled, the session is automatically summarized when token usage exceeds `preemptive_compaction_threshold`.

```

```

**Sources:** [src/config/schema.ts L167](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L167-L167)

#### preemptive_compaction_threshold

**Type:** `number`
**Range:** `0.5` to `0.95`
**Default:** `0.80`

Percentage of context window usage that triggers preemptive compaction. For example, `0.80` triggers compaction when 80% of the context window is used.

```

```

**Sources:** [src/config/schema.ts L169](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L169-L169)

#### truncate_all_tool_outputs

**Type:** `boolean`
**Default:** `true`

When `true`, truncates outputs from all tools (not just whitelisted tools like grep/glob). When `false`, only truncates specific high-output tools.

```

```

**Sources:** [src/config/schema.ts L171](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L171-L171)

#### dcp_for_compaction

**Type:** `boolean`
**Default:** `false`

When `true`, runs Dynamic Context Pruning (DCP) as the first strategy when token limit is exceeded, before falling back to full summarization. DCP attempts to remove redundant messages intelligently.

```

```

**Sources:** [src/config/schema.ts L175](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L175-L175)

#### dynamic_context_pruning

**Type:** `object`
**Default:** `{ enabled: false }`

Configures Dynamic Context Pruning (DCP), an experimental system that removes redundant or superseded messages from the context window to prevent token limit errors.

```

```

##### enabled

**Type:** `boolean`
**Default:** `false`

Master toggle for DCP. When `false`, all DCP strategies are disabled.

##### notification

**Type:** `"off" | "minimal" | "detailed"`
**Default:** `"detailed"`

Controls notification verbosity when DCP prunes messages:

* `"off"`: No notifications
* `"minimal"`: Shows count of pruned messages
* `"detailed"`: Shows detailed information about each pruned message

##### turn_protection

**Type:** `object`
**Properties:**

* `enabled`: `boolean` (default: `true`)
* `turns`: `number` (default: `3`, range: `1` to `10`)

Prevents pruning of recent tool outputs. When enabled, messages from the last N turns are protected from all pruning strategies.

##### protected_tools

**Type:** `string[]`
**Default:** `["task", "todowrite", "todoread", "lsp_rename", "lsp_code_action_resolve", "session_read", "session_write", "session_search"]`

List of tool names whose outputs should never be pruned, regardless of duplication or age.

##### strategies

**Type:** `object`

Configuration for individual pruning strategies.

###### deduplication

**Type:** `object`
**Properties:**

* `enabled`: `boolean` (default: `true`)

Removes duplicate tool calls (same tool name with same arguments). Keeps only the most recent occurrence.

###### supersede_writes

**Type:** `object`
**Properties:**

* `enabled`: `boolean` (default: `true`)
* `aggressive`: `boolean` (default: `false`)

Prunes file write operations when the file is subsequently read:

* When `aggressive: false`, only prunes writes to the exact file that was later read
* When `aggressive: true`, prunes any write if there's any subsequent read operation

###### purge_errors

**Type:** `object`
**Properties:**

* `enabled`: `boolean` (default: `true`)
* `turns`: `number` (default: `5`, range: `1` to `20`)

Removes tool call inputs that resulted in errors after N turns have passed. Keeps the error message but removes the input to save tokens.

**Sources:** [src/config/schema.ts L127-L161](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L127-L161)

## Complete Configuration Example

Here is a comprehensive example demonstrating all major configuration sections:

```

```

**Sources:** [README.md L717-L851](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L717-L851)

 [assets/oh-my-opencode.schema.json L1-L1442](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1-L1442)