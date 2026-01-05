---
layout: default
title: Tool Reference
parent: Reference
nav_order: 1
---

# Tool Reference

> **Relevant source files**
> * [README.ja.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ja.md)
> * [README.ko.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md)
> * [README.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md)
> * [README.zh-cn.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.zh-cn.md)
> * [src/shared/config-path.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts)
> * [src/tools/background-task/constants.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/constants.ts)
> * [src/tools/background-task/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/index.ts)
> * [src/tools/background-task/types.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/types.ts)
> * [src/tools/call-omo-agent/constants.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/constants.ts)
> * [src/tools/interactive-bash/constants.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/interactive-bash/constants.ts)
> * [src/tools/look-at/constants.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/look-at/constants.ts)
> * [src/tools/look-at/tools.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/look-at/tools.ts)

This page provides a complete reference for all tools available in oh-my-opencode. Tools are organized by category with their exact parameters, descriptions, and implementation details. For architectural context, see page 5 (Tool System).

All tool descriptions and parameters are sourced directly from the codebase to ensure accuracy.

## Tool Organization

Oh-my-opencode provides 20+ tools across six functional categories. Tools are registered through the OpenCode plugin system and made available to agents based on their permission settings.

**Tool Registration Architecture**

```

```

Sources: [src/tools/index.ts L1-L69](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L1-L69)

## LSP Tools

Language Server Protocol tools provide IDE-like capabilities for code navigation, analysis, and refactoring. All LSP tools require an active LSP server for the target file's language.

| Tool Name | Purpose | Key Parameters | Output Type |
| --- | --- | --- | --- |
| `lsp_hover` | Get type information, documentation, and signatures at cursor position | `path` (string)`line` (number)`character` (number) | Hover information with type signatures and docs |
| `lsp_goto_definition` | Jump to symbol definition | `path` (string)`line` (number)`character` (number) | Location(s) of definition |
| `lsp_find_references` | Find all usages of symbol across workspace | `path` (string)`line` (number)`character` (number)`includeDeclaration` (boolean) | Array of reference locations |
| `lsp_document_symbols` | Get structured outline of file symbols | `path` (string) | Hierarchical symbol tree |
| `lsp_workspace_symbols` | Search symbols by name across entire project | `query` (string) | Array of matching symbols with locations |
| `lsp_diagnostics` | Get errors, warnings, and hints for file | `path` (string) | Array of diagnostic messages with severity |
| `lsp_servers` | List available LSP servers and their status | None | Array of server information |
| `lsp_prepare_rename` | Validate that rename operation is safe | `path` (string)`line` (number)`character` (number) | Range and validation result |
| `lsp_rename` | Rename symbol across entire workspace | `path` (string)`line` (number)`character` (number)`newName` (string) | Workspace edits to apply |
| `lsp_code_actions` | Get available quick fixes and refactorings | `path` (string)`startLine` (number)`startChar` (number)`endLine` (number)`endChar` (number) | Array of available code actions |
| `lsp_code_action_resolve` | Apply selected code action | `action` (object) | Result of applying action |

**Access Restrictions**: No special restrictions. Available to all agents unless explicitly denied via tool permissions.

Sources: [src/tools/index.ts L1-L13](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L1-L13)

 [README.md L310-L321](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L310-L321)

## AST Tools

Abstract Syntax Tree tools provide syntax-aware code search and replacement capabilities across 25+ programming languages.

| Tool Name | Purpose | Key Parameters | Output Type | Notes |
| --- | --- | --- | --- | --- |
| `ast_grep_search` | Search code patterns using AST rules | `pattern` (string)`language` (string)`paths` (array, optional)`rule` (object, optional) | Array of matches with context | Supports complex AST queries and pattern matching |
| `ast_grep_replace` | Replace code patterns preserving AST structure | `pattern` (string)`replacement` (string)`language` (string)`paths` (array) | Array of files modified | Safer than regex replacement |

**Supported Languages**: JavaScript, TypeScript, Python, Rust, Go, Java, C, C++, C#, Ruby, PHP, Swift, Kotlin, Dart, Lua, Bash, and 10+ more.

**Access Restrictions**: No special restrictions. Available to all agents.

Sources: [src/tools/index.ts L15-L18](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L15-L18)

 [README.md L322-L323](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L322-L323)

## Search and File Discovery Tools

Enhanced file system tools with performance optimizations and timeout protection.

| Tool Name | Purpose | Key Parameters | Output Type | Implementation |
| --- | --- | --- | --- | --- |
| `grep` | Search file contents with pattern matching | `pattern` (string)`paths` (array, optional)`file_pattern` (string, optional)`case_sensitive` (boolean) | Array of matches with line numbers | Uses ripgrep binary for performance |
| `glob` | Find files matching path patterns | `patterns` (array)`exclude` (array, optional) | Array of matching file paths | Standard glob pattern support |

**grep Implementation Details**:

* Downloads platform-specific ripgrep binary on first use
* Binary cached in `~/.cache/oh-my-opencode/bin/`
* Provides timeout protection (default: 30 seconds)
* Replaces OpenCode's default grep which lacks timeout

**Access Restrictions**: No special restrictions. Output may be truncated by tool-output-truncator hook.

Sources: [src/tools/index.ts L20-L21](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L20-L21)

 [README.md L362](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L362-L362)

## Background Tools

Tools for managing parallel background task execution. These enable true multi-agent parallelism.

| Tool Name | Purpose | Key Parameters | Output Type | Description Source |
| --- | --- | --- | --- | --- |
| `background_task` | Start agent execution in background | `description` (string)`prompt` (string)`agent` (string) | Task ID (returns immediately) | `BACKGROUND_TASK_DESCRIPTION` constant |
| `background_output` | Check status and retrieve task results | `task_id` (string)`block` (boolean, optional)`timeout` (number, optional) | Task status, output, and completion state | `BACKGROUND_OUTPUT_DESCRIPTION` constant |
| `background_cancel` | Cancel running background task | `taskId` (string, optional)`all` (boolean, optional) | Cancellation confirmation | `BACKGROUND_CANCEL_DESCRIPTION` constant |

**Background Task Lifecycle**:

```

```

**Tool Descriptions** (as defined in code):

* `background_task`: "Run agent task in background. Returns task_id immediately; notifies on completion. Use `background_output` to get results. Prompts MUST be in English."
* `background_output`: "Get output from background task. System notifies on completion, so block=true rarely needed."
* `background_cancel`: "Cancel running background task(s). Use all=true to cancel ALL before final answer."

**Access Restrictions**:

* Available to all agents except multimodal-looker
* Task creation requires valid agent name
* Only parent session can query/cancel its own tasks
* Parent session notified on completion via background-notification hook

Sources: [src/tools/background-task/constants.ts L1-L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/constants.ts#L1-L8)

 [src/tools/background-task/types.ts L1-L17](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/types.ts#L1-L17)

## Agent Delegation Tool

Tool for invoking specialized agents synchronously or asynchronously.

| Tool Name | Purpose | Key Parameters | Output Type | Description Source |
| --- | --- | --- | --- | --- |
| `call_omo_agent` | Invoke specialized agent with prompt | `agent` (string)`prompt` (string)`run_in_background` (boolean, required) | Agent response (sync) or task ID (async) | `CALL_OMO_AGENT_DESCRIPTION` constant |

**Tool Description** (as defined in code):
"Spawn explore/librarian agent. run_in_background REQUIRED (true=async with task_id, false=sync). Available: {agents}. Prompts MUST be in English. Use `background_output` for async results."

**Allowed Agents** (from `ALLOWED_AGENTS` constant):

* `explore` - Codebase exploration
* `librarian` - Documentation research

**Note**: Only `explore` and `librarian` are allowed by `call_omo_agent`. Other agents listed in README are invoked through different mechanisms.

**Agent Invocation Flow**:

```

```

**Access Restrictions**:

* Only `explore` and `librarian` agents can be invoked via this tool
* These agents cannot recursively call `call_omo_agent` (prevents infinite recursion)
* `run_in_background` parameter is required (not optional)
* Prompts must be in English

Sources: [src/tools/call-omo-agent/constants.ts L1-L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/constants.ts#L1-L8)

## Special Tools

Specialized tools for multimodal analysis, interactive sessions, and custom extensions.

| Tool Name | Purpose | Key Parameters | Output Type | Description Source |
| --- | --- | --- | --- | --- |
| `look_at` | Analyze visual content (PDF, images, diagrams) | `path` (string)`prompt` (string, optional) | Extracted text and visual analysis | `LOOK_AT_DESCRIPTION` constant |
| `interactive_bash` | Execute tmux commands for persistent sessions | `tmux_command` (string) | Command output or error | `INTERACTIVE_BASH_DESCRIPTION` constant |
| `slashcommand` | Execute custom slash commands | `command` (string)Additional params vary | Command-specific output | Loaded from Claude Code compatible paths |
| `skill` | Invoke custom skills | `skill_name` (string)Additional params vary | Skill-specific output | Loaded from Claude Code compatible paths |

**Tool Descriptions** (as defined in code):

* `look_at`: "Analyze media files (PDFs, images, diagrams) via Gemini 2.5 Flash in separate context. Saves main context tokens."
* `interactive_bash`: "Execute tmux commands. Use 'omo-{name}' session pattern. Blocked (use bash instead): capture-pane, save-buffer, show-buffer, pipe-pane."

**look_at Tool Details**:

* Internally delegates to agent specified in `MULTIMODAL_LOOKER_AGENT` constant ("multimodal-looker")
* Supports PDF, PNG, JPG, GIF, WebP formats
* Extracts text, analyzes diagrams, interprets visual content
* Saves tokens by avoiding full file content in parent context

**interactive_bash Tool Details**:

```

```

**interactive_bash Requirements**:

* tmux must be installed and in PATH
* Path discovered on plugin load via background check
* Default timeout: 60,000ms (60 seconds) as defined in `DEFAULT_TIMEOUT_MS`
* Sessions tracked per OpenCode session via interactive-bash-session hook
* Automatic cleanup on session deletion
* Blocked tmux subcommands defined in `BLOCKED_TMUX_SUBCOMMANDS` array

Sources: [src/tools/interactive-bash/constants.ts L1-L17](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/interactive-bash/constants.ts#L1-L17)

**slashcommand and skill Loading Paths**:

* User commands: `~/.claude/commands/*.md`
* Project commands: `./.claude/commands/*.md`, `./.opencode/command/*.md`
* User skills: `~/.claude/skills/*/SKILL.md`
* Project skills: `./.claude/skills/*/SKILL.md`

Sources: [src/tools/interactive-bash/constants.ts L1-L17](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/interactive-bash/constants.ts#L1-L17)

 [src/tools/look-at/constants.ts L1-L4](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/look-at/constants.ts#L1-L4)

 [src/tools/call-omo-agent/constants.ts L1-L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/constants.ts#L1-L8)

## Tool Access Control Matrix

Different agents have different tool access based on their role and permission configuration.

| Agent | LSP Tools | AST Tools | Search Tools | Background Tools | call_omo_agent | Special Tools |
| --- | --- | --- | --- | --- | --- | --- |
| **Sisyphus** | ✓ All | ✓ All | ✓ All | ✓ All | ✓ explore, librarian only | ✓ All |
| **explore** | ✓ All | ✓ All | ✓ All | ✓ All | ✗ Cannot invoke | ✓ Limited |
| **librarian** | ✓ All | ✓ All | ✓ All | ✓ All | ✗ Cannot invoke | ✓ Limited |
| **oracle** | ✓ All | ✓ All | ✓ All | ✓ All | ✓ explore, librarian only | ✓ All |
| **frontend-ui-ux-engineer** | ✓ All | ✓ All | ✓ All | ✓ All | ✓ explore, librarian only | ✓ All |
| **document-writer** | ✓ All | ✓ All | ✓ All | ✓ All | ✓ explore, librarian only | ✓ All |
| **multimodal-looker** | ✓ All | ✓ All | ✓ All | ✗ Denied | ✗ Cannot invoke | ✗ look_at denied |

**Access Control Rules**:

* `call_omo_agent` only accepts `explore` and `librarian` as valid agent names (per `ALLOWED_AGENTS` constant)
* `explore` and `librarian` cannot recursively invoke `call_omo_agent` (prevents infinite recursion)
* `multimodal-looker` cannot use `background_task`, `call_omo_agent`, or `look_at` (prevents self-recursion)
* All other agents have full tool access by default
* Additional restrictions can be configured via `permission` settings in agent configuration

**Tool Permission Configuration**:

Agent-specific tool restrictions are defined in agent factory functions and can be overridden in configuration. The permission system operates at the OpenCode plugin level.

Sources: [src/tools/call-omo-agent/constants.ts L1-L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/constants.ts#L1-L8)

 [README.md L489-L506](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L489-L506)

## Tool Implementation Patterns

**Built-in Tool Registration**:

```

```

**Dynamic Tool Creation**:

```

```

**Tool Execution Pipeline** (applies to all tools):

1. **Pre-execution hooks** - `tool.execute.before` event * Record tool use * Cache input * Validate arguments * External PreToolUse scripts
2. **Tool execution** - Actual tool logic runs * May spawn processes (grep, bash) * May call APIs (LSP) * May delegate to agents (look_at, call_omo_agent)
3. **Post-execution hooks** - `tool.execute.after` event * Record tool result * Truncate output (tool-output-truncator) * Inject context (directory injectors) * External PostToolUse scripts
4. **Return to agent** - Result delivered to calling agent

Sources: [src/tools/index.ts L42-L68](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L42-L68)

 Diagram 5 from high-level architecture

## Tool Output Truncation

Several tools have dynamic output truncation to prevent context window exhaustion:

| Tool | Truncation Strategy | Target Size | Hook Responsible |
| --- | --- | --- | --- |
| `grep` | Line-based truncation with match preservation | 50% context headroom, max 50k tokens | grep-output-truncator |
| `glob` | File list truncation | 50% context headroom | tool-output-truncator |
| `lsp_*` | Response truncation for large results | 50% context headroom | tool-output-truncator |
| `ast_grep_search` | Match truncation | 50% context headroom | tool-output-truncator |

**Truncation Algorithm**:

1. Calculate remaining context window space
2. Set target at 50% of remaining (maintains headroom)
3. Cap at 50k tokens maximum
4. Truncate with "..." marker and summary

Truncation can be disabled via configuration:

```

```

Sources: [README.md L473-L475](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L473-L475)

 Diagram 3 from high-level architecture