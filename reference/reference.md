---
layout: default
title: Reference
parent: Reference
nav_order: 1
---

# Reference

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

This section provides comprehensive reference documentation for all configurable aspects of oh-my-opencode. It serves as a lookup guide for configuration options, agents, tools, and hooks available in the plugin.

**Scope**: This page provides an overview of the reference material and quick-access tables for common configuration tasks. For detailed documentation of specific components:

* Configuration options: see [Configuration Schema Reference](/code-yeongyu/oh-my-opencode/12.1-build-system)
* Agent specifications: see [Agent Reference](/code-yeongyu/oh-my-opencode/12.2-cicd-pipeline)
* Tool capabilities: see [Tool Reference](/code-yeongyu/oh-my-opencode/12.3-release-process)
* Hook behaviors: see [Hook Reference](/code-yeongyu/oh-my-opencode/12.4-dependency-management)

## Configuration System Overview

The oh-my-opencode plugin uses a hierarchical JSON-based configuration system with validation, migration, and IDE support.

### Configuration Loading Flow

**Diagram: Configuration File Loading and Validation**

```

```

**Sources**: [src/index.ts L125-L217](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L125-L217)

 [src/shared/jsonc.ts L10-L30](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc.ts#L10-L30)

 [src/shared/deep-merge.ts L1-L20](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/deep-merge.ts#L1-L20)

 [src/config/schema.ts L178-L204](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L178-L204)

### Configuration File Structure

The configuration system uses a two-tier hierarchy with project-level configs overriding user-level configs:

| Priority | Location (Linux/macOS) | Location (Windows) |
| --- | --- | --- |
| 1 | `.opencode/oh-my-opencode.json(c)` | `.opencode/oh-my-opencode.json(c)` |
| 2 | `~/.config/opencode/oh-my-opencode.json(c)` | `~/.config/opencode/oh-my-opencode.json(c)` or `%APPDATA%/opencode/oh-my-opencode.json(c)` |

**JSONC Support**: Both `.json` and `.jsonc` (JSON with Comments) formats are supported. The `.jsonc` file takes priority if both exist.

**Sources**: [src/index.ts L189-L217](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L189-L217)

 [src/shared/config-path.ts L1-L48](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts#L1-L48)

 [src/shared/jsonc.ts L10-L30](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc.ts#L10-L30)

### Configuration Schema Structure

**Diagram: OhMyOpenCodeConfig Type Structure**

```

```

**Sources**: [src/config/schema.ts L178-L204](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L178-L204)

 [src/mcp/types.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L1-L5)

### Top-Level Configuration Options

| Property | Type | Description | Default | Reference |
| --- | --- | --- | --- | --- |
| `disabled_mcps` | `McpName[]` | MCP servers to disable | `[]` | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `disabled_agents` | `AgentName[]` | Built-in agents to disable | `[]` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `disabled_hooks` | `HookName[]` | Hooks to disable | `[]` | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `disabled_commands` | `BuiltinCommandName[]` | Built-in commands to disable | `[]` | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `agents` | `AgentOverrides` | Agent configuration overrides | `{}` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `claude_code` | `ClaudeCodeConfig` | Claude Code compatibility toggles | `{}` | [#9](/code-yeongyu/oh-my-opencode/9-claude-code-compatibility) |
| `google_auth` | `boolean` | Enable built-in Google auth | `false` | [#2.2](../getting-started/Authentication-Setup.md) |
| `sisyphus_agent` | `SisyphusAgentConfig` | Sisyphus orchestrator config | `{}` | [#4.1](/code-yeongyu/oh-my-opencode/4.1-sisyphus-orchestrator) |
| `comment_checker` | `CommentCheckerConfig` | Comment checker customization | `{}` | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `experimental` | `ExperimentalConfig` | Experimental feature flags | `{}` | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `auto_update` | `boolean` | Enable automatic updates | `true` | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |

**Sources**: [src/config/schema.ts L178-L204](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L178-L204)

## System Components Quick Reference

### Built-in MCPs

**Diagram: MCP Registration and Configuration**

```

```

**Sources**: [src/index.ts L519-L528](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L519-L528)

 [src/mcp/index.ts L1-L40](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L1-L40)

 [src/mcp/types.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L1-L5)

| MCP Name | Purpose | Tools Provided | Used By | Configuration Function |
| --- | --- | --- | --- | --- |
| `websearch_exa` | Web search with 2025+ filter | `search` | librarian | `createBuiltinMcps()` |
| `context7` | Official documentation lookup | `resolve-library-id`, `get-docs` | librarian | `createBuiltinMcps()` |
| `grep_app` | GitHub code search | `search` | librarian, explore | `createBuiltinMcps()` |

**Sources**: [src/mcp/index.ts L1-L40](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L1-L40)

 [src/mcp/types.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L1-L5)

### Built-in Agents

**Diagram: Agent Creation and Configuration Flow**

```

```

**Sources**: [src/index.ts L62-L94](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L62-L94)

 [src/index.ts L404-L486](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L404-L486)

 [src/agents/index.ts L1-L350](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/index.ts#L1-L350)

 [src/features/claude-code-agent-loader.ts L1-L100](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/claude-code-agent-loader.ts#L1-L100)

| Agent Name | Default Model | Mode | Write Access | Created By | Reference |
| --- | --- | --- | --- | --- | --- |
| `Sisyphus` | `anthropic/claude-opus-4-5` | primary | Yes | `createBuiltinAgents()` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `Planner-Sisyphus` | `anthropic/claude-opus-4-5` | subagent | No | `createBuiltinAgents()` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `OpenCode-Builder` | From `config.agent.build` | subagent | Yes | Runtime | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `oracle` | `openai/gpt-5.2` | subagent | No (LSP only) | `createBuiltinAgents()` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `librarian` | `anthropic/claude-sonnet-4-5` | subagent | No | `createBuiltinAgents()` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `explore` | `opencode/grok-code` | subagent | No | `createBuiltinAgents()` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `frontend-ui-ux-engineer` | `google/gemini-3-pro-high` | subagent | Yes | `createBuiltinAgents()` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `document-writer` | `google/gemini-3-flash` | subagent | Yes | `createBuiltinAgents()` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `multimodal-looker` | `google/gemini-3-flash` | subagent | No | `createBuiltinAgents()` | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `build` | From OpenCode config | primary/subagent | Yes | OpenCode | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |
| `plan` | From OpenCode config | primary/subagent | No | OpenCode | [#13.2](/code-yeongyu/oh-my-opencode/13.2-agent-reference) |

**Sources**: [src/agents/index.ts L1-L350](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/index.ts#L1-L350)

 [src/index.ts L404-L486](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L404-L486)

### Built-in Hooks

**Diagram: Hook Initialization and Event Dispatch**

```

```

**Sources**: [src/index.ts L219-L332](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L219-L332)

 [src/hooks/index.ts L1-L25](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts#L1-L25)

 [src/config/schema.ts L45-L68](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L45-L68)

| Hook Name | Creation Function | Primary Event | Purpose | Reference |
| --- | --- | --- | --- | --- |
| `todo-continuation-enforcer` | `createTodoContinuationEnforcer()` | `session.idle` | Auto-inject continue prompt | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `context-window-monitor` | `createContextWindowMonitorHook()` | `message.updated` | Track token usage, warn at 70% | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `session-recovery` | `createSessionRecoveryHook()` | `session.error` | Abort + fix + resume | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `session-notification` | `createSessionNotification()` | `session.idle` | OS notifications | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `comment-checker` | `createCommentCheckerHooks()` | `tool.execute.after` | Warn about comments | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `tool-output-truncator` | `createToolOutputTruncatorHook()` | `tool.execute.after` | Dynamic truncation | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `directory-agents-injector` | `createDirectoryAgentsInjectorHook()` | `tool.execute.before` | Inject AGENTS.md | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `directory-readme-injector` | `createDirectoryReadmeInjectorHook()` | `tool.execute.before` | Inject README.md | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `empty-task-response-detector` | `createEmptyTaskResponseDetectorHook()` | `tool.execute.after` | Detect empty task output | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `think-mode` | `createThinkModeHook()` | `chat.message` | Switch to thinking mode | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `anthropic-context-window-limit-recovery` | `createAnthropicContextWindowLimitRecoveryHook()` | `session.error` | Auto-compact on limit | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `preemptive-compaction` | `createPreemptiveCompactionHook()` | `message.updated` | Compact at 80% | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `rules-injector` | `createRulesInjectorHook()` | `tool.execute.before` | Inject .claude/rules/ | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `background-notification` | `createBackgroundNotificationHook()` | `session.idle` | Background task alerts | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `auto-update-checker` | `createAutoUpdateCheckerHook()` | `session.created` | Check npm for updates | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `startup-toast` | (part of `auto-update-checker`) | `session.created` | Show version toast | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `keyword-detector` | `createKeywordDetectorHook()` | `chat.message` | Detect ultrawork/search | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `agent-usage-reminder` | `createAgentUsageReminderHook()` | `chat.message` | Remind to use agents | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `non-interactive-env` | `createNonInteractiveEnvHook()` | `tool.execute.before` | Set CI env vars | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `interactive-bash-session` | `createInteractiveBashSessionHook()` | Multiple | tmux session tracking | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `empty-message-sanitizer` | `createEmptyMessageSanitizerHook()` | `experimental.chat.messages.transform` | Remove empty messages | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |
| `thinking-block-validator` | `createThinkingBlockValidatorHook()` | `experimental.chat.messages.transform` | Validate thinking blocks | [#13.4](/code-yeongyu/oh-my-opencode/13.4-hook-reference) |

**Sources**: [src/index.ts L238-L332](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L238-L332)

 [src/hooks/index.ts L1-L25](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts#L1-L25)

 [src/config/schema.ts L45-L68](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L45-L68)

## Agent Configuration Structure

**Diagram: AgentOverrideConfigSchema Type Hierarchy**

```

```

**Sources**: [src/config/schema.ts L4-L103](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L4-L103)

### Agent Override Properties

| Property | Type | Constraints | Description |
| --- | --- | --- | --- |
| `model` | `string` | - | LLM model identifier (e.g., `"anthropic/claude-opus-4-5"`) |
| `temperature` | `number` | `0.0 - 2.0` | Sampling temperature for creativity control |
| `top_p` | `number` | `0.0 - 1.0` | Nucleus sampling parameter |
| `prompt` | `string` | - | Complete replacement system prompt |
| `prompt_append` | `string` | - | Append to default system prompt |
| `tools` | `Record<string, boolean>` | - | Per-tool enable/disable (e.g., `{ "edit": false }`) |
| `disable` | `boolean` | - | Completely disable the agent |
| `description` | `string` | - | Agent description shown in UI |
| `mode` | `enum` | `'subagent' \| 'primary' \| 'all'` | Agent operational mode |
| `color` | `string` | Hex color pattern | UI color (e.g., `"#00CED1"`) |
| `permission` | `object` | See below | Fine-grained permission control |

**Sources**: [src/config/schema.ts L74-L89](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L74-L89)

### Permission System

The permission system provides three levels of access control:

| Permission Level | Behavior |
| --- | --- |
| `ask` | Prompt user for approval each time |
| `allow` | Always allow without prompting |
| `deny` | Always deny the operation |

The `bash` permission can be granular, specified as either:

* A single permission string: `"ask"`, `"allow"`, or `"deny"`
* A per-command mapping: `{ "npm": "allow", "rm": "ask", "git": "allow" }`

**Sources**: [assets/oh-my-opencode.schema.json L115-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L115-L176)

## Experimental Features

**Diagram: ExperimentalConfigSchema Structure**

```

```

**Sources**: [src/config/schema.ts L127-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L127-L176)

| Feature | Type | Default | Description | Reference |
| --- | --- | --- | --- | --- |
| `aggressive_truncation` | `boolean` | `undefined` | More aggressive tool output truncation | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `auto_resume` | `boolean` | `undefined` | Automatically resume after recovery | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `preemptive_compaction` | `boolean` | `undefined` | Trigger compaction before hard limit | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `preemptive_compaction_threshold` | `number` | `undefined` | Percentage (0.5-0.95) to trigger compaction | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `truncate_all_tool_outputs` | `boolean` | `true` | Apply truncation to all tools, not just whitelist | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `dcp_for_compaction` | `boolean` | `undefined` | Use DCP before summarization | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |
| `dynamic_context_pruning` | `object` | See schema | Advanced context pruning with strategies | [#13.1](/code-yeongyu/oh-my-opencode/13.1-configuration-schema-reference) |

**Sources**: [src/config/schema.ts L163-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L163-L176)

### Dynamic Context Pruning Configuration

The `dynamic_context_pruning` experimental feature provides advanced context management with multiple pruning strategies:

| DCP Property | Type | Default | Description |
| --- | --- | --- | --- |
| `enabled` | `boolean` | `false` | Enable dynamic context pruning |
| `notification` | `'off' \| 'minimal' \| 'detailed'` | `'detailed'` | Notification verbosity level |
| `turn_protection.enabled` | `boolean` | `true` | Protect recent tool outputs |
| `turn_protection.turns` | `number` | `3` | Number of recent turns to protect |
| `protected_tools` | `string[]` | `['task', 'todowrite', ...]` | Tools never pruned |
| `strategies.deduplication.enabled` | `boolean` | `true` | Remove duplicate tool calls |
| `strategies.supersede_writes.enabled` | `boolean` | `true` | Prune writes when file subsequently read |
| `strategies.supersede_writes.aggressive` | `boolean` | `false` | Prune any write if ANY subsequent read |
| `strategies.purge_errors.enabled` | `boolean` | `true` | Prune errored tool inputs after N turns |
| `strategies.purge_errors.turns` | `number` | `5` | Turns before purging errors |

**Sources**: [src/config/schema.ts L127-L161](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L127-L161)

## Version Management and Auto-Update

**Diagram: Auto-Update System Architecture**

```

```

**Sources**: [src/hooks/auto-update-checker/checker.ts L15-L264](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/checker.ts#L15-L264)

 [src/hooks/auto-update-checker/cache.ts L49-L93](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/cache.ts#L49-L93)

 [src/hooks/auto-update-checker/constants.ts L13-L41](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/constants.ts#L13-L41)

### Version Detection Constants

The auto-update system uses platform-specific paths defined in constants:

**OPENCODE_CONFIG_PATHS** (checked in order):

```

```

**OPENCODE_CACHE_DIR**:

* Linux/macOS: `~/.cache/opencode/`
* Windows: `%LOCALAPPDATA%/opencode/`

**Sources**: [src/hooks/auto-update-checker/constants.ts L13-L41](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/constants.ts#L13-L41)

### Cache Directories

| Purpose | Location (Linux/macOS) | Location (Windows) |
| --- | --- | --- |
| Plugin cache | `~/.cache/opencode/` | `%LOCALAPPDATA%/opencode/` |
| Installed packages | `~/.cache/opencode/node_modules/` | `%LOCALAPPDATA%/opencode/node_modules/` |
| Version tracking | `~/.cache/opencode/version` | `%LOCALAPPDATA%/opencode/version` |

**Sources**: [src/hooks/auto-update-checker/constants.ts L13-L27](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/constants.ts#L13-L27)

## Configuration Schema Generation

**Diagram: Schema Build Pipeline**

```

```

**Sources**: [script/build-schema.ts L1-L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/build-schema.ts#L1-L29)

 [src/config/schema.ts L178-L204](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L178-L204)

 [assets/oh-my-opencode.schema.json L1-L10](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1-L10)

### Schema Reference Usage

Configuration files reference the schema for IDE support:

```

```

The schema provides:

* Autocomplete for all configuration properties
* Validation of enum values (agent names, hook names, etc.)
* Type checking for numbers, booleans, and strings
* Pattern validation for colors, model identifiers, etc.

**Sources**: [assets/oh-my-opencode.schema.json L1-L10](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1-L10)

 [README.md L709-L713](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L709-L713)

## Navigation Guide

This reference section is organized into four main subsections:

1. **[Configuration Schema Reference](/code-yeongyu/oh-my-opencode/12.1-build-system)**: Complete documentation of all configuration options, including examples and validation rules
2. **[Agent Reference](/code-yeongyu/oh-my-opencode/12.2-cicd-pipeline)**: Detailed specifications for each agent, including default models, capabilities, and use cases
3. **[Tool Reference](/code-yeongyu/oh-my-opencode/12.3-release-process)**: Comprehensive listing of all tools with their arguments, permissions, and usage examples
4. **[Hook Reference](/code-yeongyu/oh-my-opencode/12.4-dependency-management)**: Complete hook catalog with trigger conditions, purposes, and configuration options

For conceptual information about how these components work together, see:

* Agent system architecture: [#4](../agents/)
* Tool system overview: [#5](../tools/)
* Hook system architecture: [#7](../reliability/)
* Configuration system: [#3.2](/code-yeongyu/oh-my-opencode/3.2-configuration-system)