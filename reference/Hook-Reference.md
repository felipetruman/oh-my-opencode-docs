---
layout: default
title: Hook Reference
parent: Reference
nav_order: 1
---

# Hook Reference

> **Relevant source files**
> * [assets/oh-my-opencode.schema.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json)
> * [src/config/schema.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts)
> * [src/hooks/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts)
> * [src/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts)

This page provides a comprehensive reference for all lifecycle hooks available in oh-my-opencode. Hooks are modular components that intercept and enhance behavior at specific points in the OpenCode execution flow. For information about the overall hook system architecture, see [Hook System Overview](../reliability/). For detailed implementation guides on specific hook categories, see [Context Management Hooks](/code-yeongyu/oh-my-opencode/7.1-session-recovery), [Session Management Hooks](/code-yeongyu/oh-my-opencode/7.2-message-validation), [Tool Enhancement Hooks](/code-yeongyu/oh-my-opencode/7.3-todo-continuation-enforcer), [Context Injection Hooks](/code-yeongyu/oh-my-opencode/7.4-context-management-hooks), and [User Experience Hooks](/code-yeongyu/oh-my-opencode/7.5-context-injection-hooks).

---

## Hook System Architecture

The hook system operates by registering handlers at specific OpenCode plugin extension points. Each hook can process events, modify tool inputs/outputs, or inject context at various stages of the agent workflow.

```

```

Sources: [src/index.ts L211-L576](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L211-L576)

---

## Hook Lifecycle Events

Hooks attach to specific OpenCode event types. The following diagram shows when each event fires and which hooks process them:

```

```

Sources: [src/index.ts L480-L574](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L480-L574)

 [README.md L44-L146](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L44-L146)

---

## Complete Hook Reference Table

The following table lists all available hooks with their identifiers, trigger conditions, and primary purposes:

| Hook Name | Category | Trigger Event(s) | Enabled by Default | Primary Purpose |
| --- | --- | --- | --- | --- |
| `todo-continuation-enforcer` | Session Management | `session.idle` | ✓ | Forces agents to complete TODOs before stopping |
| `context-window-monitor` | Context Management | `message.updated`, `tool.execute.after` | ✓ | Tracks token usage and reminds agents of headroom |
| `session-recovery` | Session Management | `session.error` | ✓ | Auto-recovers from missing tool results, thinking errors |
| `session-notification` | UX | `session.idle` | ✓ | Sends OS notifications when agents go idle |
| `comment-checker` | Tool Enhancement | `tool.execute.before`, `tool.execute.after` | ✓ | Prevents excessive code comments |
| `grep-output-truncator` | Tool Enhancement | `tool.execute.after` | ✓ | Truncates grep output based on context window |
| `tool-output-truncator` | Tool Enhancement | `tool.execute.after` | ✓ | Dynamically truncates tool outputs to preserve context |
| `directory-agents-injector` | Context Injection | `tool.execute.before`, `tool.execute.after`, `event` | ✓ | Injects AGENTS.md from directory hierarchy |
| `directory-readme-injector` | Context Injection | `tool.execute.before`, `tool.execute.after`, `event` | ✓ | Injects README.md from project root |
| `empty-task-response-detector` | Tool Enhancement | `tool.execute.after` | ✓ | Warns when Task tool returns empty response |
| `think-mode` | UX | `event` | ✓ | Auto-enables extended thinking for complex queries |
| `anthropic-auto-compact` | Context Management | `session.error`, `event` | ✓ | Auto-compacts Claude sessions at token limits |
| `preemptive-compaction` | Context Management | `event` | ✓ | Proactively compacts at 80% threshold |
| `compaction-context-injector` | Context Management | (internal) | ✓ | Injects context for summarization |
| `rules-injector` | Context Injection | `tool.execute.before`, `tool.execute.after`, `event` | ✓ | Injects conditional rules from .claude/rules/ |
| `background-notification` | UX | `event` | ✓ | Notifies when background tasks complete |
| `auto-update-checker` | UX | `event` | ✓ | Checks for oh-my-opencode updates |
| `startup-toast` | UX | `session.created` | ✓ | Shows welcome message on load |
| `keyword-detector` | UX | `chat.message` | ✓ | Detects keywords to activate specialized modes |
| `agent-usage-reminder` | UX | `tool.execute.after`, `event` | ✓ | Reminds users to leverage specialized agents |
| `non-interactive-env` | Tool Enhancement | `tool.execute.before` | ✓ | Validates permissions in non-interactive environments |
| `interactive-bash-session` | Tool Enhancement | `tool.execute.after`, `event` | ✓ | Manages tmux sessions for interactive_bash tool |
| `empty-message-sanitizer` | Session Management | `experimental.chat.messages.transform` | ✓ | Prevents API errors from empty messages |
| `claude-code-hooks` | Compatibility | Multiple | ✓ | Executes Claude Code-compatible hooks |

Sources: [src/config/schema.ts L44-L66](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L44-L66)

 [src/index.ts L230-L304](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L230-L304)

 [README.md L792-L793](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L792-L793)

---

## Hook Configuration

### Disabling Hooks

Individual hooks can be disabled via the `disabled_hooks` array in configuration files:

```

```

**Configuration Locations:**

* User: `~/.config/opencode/oh-my-opencode.json`
* Project: `.opencode/oh-my-opencode.json`

**Hook Enablement Logic:**

```

```

Sources: [src/index.ts L213-L214](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L213-L214)

 [README.md L846-L854](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L846-L854)

---

## Context Management Hooks

These hooks manage token usage and context window limits.

### context-window-monitor

**Trigger Events:** `message.updated`, `tool.execute.after`

**Purpose:** Implements [Context Window Anxiety Management](https://agentic-patterns.com/patterns/context-window-anxiety-management/) pattern. Tracks cumulative token usage per session and reminds agents when 70%+ capacity is reached.

**Key Functions:**

* Parses token counts from message metadata
* Maintains per-session token accumulation
* Injects reminder messages at threshold

**Factory:** `createContextWindowMonitorHook(ctx)`

Sources: [src/index.ts L233-L235](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L233-L235)

 [README.md L689-L691](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L689-L691)

---

### preemptive-compaction

**Trigger Events:** `message.updated`

**Purpose:** Proactively triggers session summarization before hitting hard token limits. Default threshold: 80% of model capacity.

**Configuration Options:**

```

```

**Features:**

* Model-aware limit detection via `getModelLimit()`
* Coordinates with `compaction-context-injector`
* Anthropic 1M token context support

**Factory:** `createPreemptiveCompactionHook(ctx, options)`

Sources: [src/index.ts L275-L279](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L275-L279)

 [src/config/schema.ts L112-L115](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L112-L115)

---

### anthropic-auto-compact

**Trigger Events:** `session.error`, `event`

**Purpose:** Automatically recovers from Anthropic token limit errors by summarizing and compacting the session. Detects specific error patterns like "maximum context length" and "prompt is too long".

**Recovery Flow:**

1. Detect recoverable token limit error
2. Delete invalid message from storage
3. Trigger session summarization
4. Retry with compacted context

**Experimental Options:**

```

```

**Factory:** `createAnthropicAutoCompactHook(ctx, options)`

Sources: [src/index.ts L271-L273](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L271-L273)

 [README.md L692](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L692-L692)

---

### compaction-context-injector

**Trigger Events:** Internal (called by compaction hooks)

**Purpose:** Injects context-aware instructions when triggering session summarization. Ensures summaries preserve critical information for ongoing work.

**Factory:** `createCompactionContextInjector()`

Sources: [src/index.ts L274](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L274-L274)

---

## Session Management Hooks

These hooks handle session lifecycle, error recovery, and state persistence.

### session-recovery

**Trigger Events:** `session.error`

**Purpose:** Automatically recovers from session errors including:

* Missing tool call results
* Thinking block violations
* Empty message errors
* Orphaned function calls

**Recovery Strategies:**

1. Detect recoverable error pattern
2. Manipulate message storage to fix inconsistency
3. Optionally auto-resume with extracted user prompt

**State Coordination:** Integrates with `todo-continuation-enforcer` to prevent prompt injection conflicts during active recovery.

```

```

**Experimental Options:**

```

```

**Factory:** `createSessionRecoveryHook(ctx, options)`

Sources: [src/index.ts L236-L248](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L236-L248)

 [src/index.ts L515-L539](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L515-L539)

 [README.md L693](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L693-L693)

---

### todo-continuation-enforcer

**Trigger Events:** `session.idle`

**Purpose:** Prevents agents from stopping mid-task. When session goes idle with incomplete TODOs, automatically injects continuation prompt.

**Coordination:** Respects recovery state from `session-recovery` to avoid conflicting prompts during error handling.

**Storage Integration:** Reads TODO list from `~/.claude/todos/` in Claude Code-compatible format.

**Factory:** `createTodoContinuationEnforcer(ctx)`

Sources: [src/index.ts L230-L232](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L230-L232)

 [src/index.ts L245-L247](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L245-L247)

 [README.md L687](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L687-L687)

---

### session-notification

**Trigger Events:** `session.idle`

**Purpose:** Sends OS-level desktop notifications when agent sessions go idle. Cross-platform support (macOS, Linux, Windows).

**Notification Content:**

* Session title
* Last agent action
* Call to action

**Factory:** `createSessionNotification(ctx)`

Sources: [src/index.ts L239-L241](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L239-L241)

 [README.md L697](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L697-L697)

---

### empty-message-sanitizer

**Trigger Events:** `experimental.chat.messages.transform`

**Purpose:** Prevents API errors from empty chat messages by sanitizing message content before transmission. Removes messages with no text/image content.

**Factory:** `createEmptyMessageSanitizerHook()`

Sources: [src/index.ts L302-L304](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L302-L304)

 [src/index.ts L338-L344](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L338-L344)

 [README.md L699](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L699-L699)

---

## Tool Enhancement Hooks

These hooks intercept and enhance tool execution behavior.

### tool-output-truncator

**Trigger Events:** `tool.execute.after`

**Purpose:** Dynamically truncates verbose tool outputs to preserve context window. Applies to:

* `grep` - capped at 50k tokens
* `glob` - capped at 50k tokens
* LSP tools (`lsp_*`)
* `ast_grep_search`

**Truncation Strategy:**

* Maintains 50% context window headroom
* Respects model-specific limits
* Aggressive mode available via experimental config

**Configuration:**

```

```

**Factory:** `createToolOutputTruncatorHook(ctx, options)`

Sources: [src/index.ts L253-L255](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L253-L255)

 [README.md L700-L702](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L700-L702)

---

### comment-checker

**Trigger Events:** `tool.execute.before`, `tool.execute.after`

**Purpose:** Prevents agents from adding excessive code comments. Analyzes code changes and warns about unnecessary comments while preserving valid patterns:

* BDD-style comments (Given/When/Then)
* Directive comments (eslint-disable, prettier-ignore)
* Documentation comments (JSDoc, TSDoc)
* Section markers

**Detection Logic:**

* Compares before/after code
* Identifies newly added comments
* Classifies comment types
* Issues warnings for low-value comments

**Factory:** `createCommentCheckerHooks()`

Sources: [src/index.ts L250-L252](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L250-L252)

 [README.md L687-L688](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L687-L688)

---

### empty-task-response-detector

**Trigger Events:** `tool.execute.after`

**Purpose:** Detects when the `task` tool returns empty responses, indicating potential agent failures. Warns user to avoid infinite waiting.

**Detection Criteria:**

* Tool name: `task`
* Output contains no meaningful content
* Output is shorter than threshold

**Factory:** `createEmptyTaskResponseDetectorHook(ctx)`

Sources: [src/index.ts L262-L264](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L262-L264)

 [README.md L698](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L698-L698)

---

### non-interactive-env

**Trigger Events:** `tool.execute.before`

**Purpose:** Validates tool execution in non-interactive environments (CI/CD, headless). Enforces permission checks for destructive operations when user cannot approve interactively.

**Affected Tools:**

* `bash` - command execution
* `edit` - file modifications
* `webfetch` - network requests

**Factory:** `createNonInteractiveEnvHook(ctx)`

Sources: [src/index.ts L296-L298](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L296-L298)

---

### interactive-bash-session

**Trigger Events:** `tool.execute.after`, `session.deleted`

**Purpose:** Manages tmux sessions for the `interactive_bash` tool. Tracks session state, persists information, and handles cleanup.

**State Management:**

* Creates/attaches to tmux sessions
* Tracks session-to-tmux mappings
* Persists state to JSON files
* Cleans up orphaned sessions

**Storage Location:** Per-session JSON files in storage directory

**Factory:** `createInteractiveBashSessionHook(ctx)`

Sources: [src/index.ts L299-L301](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L299-L301)

 [README.md L147-L148](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L147-L148)

---

## Context Injection Hooks

These hooks inject contextual information at strategic points.

### directory-agents-injector

**Trigger Events:** `tool.execute.before`, `tool.execute.after`, `event`

**Purpose:** Automatically injects `AGENTS.md` files from directory hierarchy when reading files. Walks from file directory to project root, collecting all `AGENTS.md` files.

**Injection Pattern:**

```markdown
project/AGENTS.md          # Project-wide context
├── src/AGENTS.md          # src-specific context
    └── components/AGENTS.md   # Component-specific context
```

**Features:**

* Path-based hierarchy traversal
* One-time injection per session
* Cumulative context accumulation

**Factory:** `createDirectoryAgentsInjectorHook(ctx)`

Sources: [src/index.ts L256-L258](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L256-L258)

 [README.md L551-L561](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L551-L561)

---

### directory-readme-injector

**Trigger Events:** `tool.execute.before`, `tool.execute.after`, `event`

**Purpose:** Injects project root `README.md` to provide high-level project context. Similar to AGENTS.md injection but specifically for project documentation.

**Injection Timing:** First file read operation in session

**Factory:** `createDirectoryReadmeInjectorHook(ctx)`

Sources: [src/index.ts L259-L261](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L259-L261)

---

### rules-injector

**Trigger Events:** `tool.execute.before`, `tool.execute.after`, `event`

**Purpose:** Injects conditional coding rules from `.claude/rules/` based on glob pattern matching. Supports both always-applied rules and context-specific rules.

**Rule File Format:**

```

```

**Search Paths:**

1. User rules: `~/.claude/rules/`
2. Project directory → root (upward walk)

**Supported Formats:** `.md`, `.mdc`

**Factory:** `createRulesInjectorHook(ctx)`

Sources: [src/index.ts L280-L282](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L280-L282)

 [README.md L562-L577](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L562-L577)

---

## User Experience Hooks

These hooks enhance user interaction and workflow.

### keyword-detector

**Trigger Events:** `chat.message`

**Purpose:** Detects keywords in user prompts to automatically activate specialized modes:

| Keyword(s) | Mode Activated | Effect |
| --- | --- | --- |
| `ultrawork`, `ulw` | Maximum Performance | Parallel agent orchestration |
| `search`, `find`, `찾아`, `検索` | Maximized Search | Parallel explore + librarian |
| `analyze`, `investigate`, `분석`, `調査` | Deep Analysis | Multi-phase expert consultation |

**Implementation:** Scans incoming message content for keyword patterns and modifies agent configuration dynamically.

**Factory:** `createKeywordDetectorHook()`

Sources: [src/index.ts L290-L292](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L290-L292)

 [README.md L682-L685](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L682-L685)

---

### think-mode

**Trigger Events:** `event`

**Purpose:** Auto-detects when extended thinking is needed and dynamically adjusts model settings. Triggers on phrases like "think deeply", "ultrathink", or complex reasoning requests.

**Model Adjustments:**

* Enables extended thinking mode
* Increases thinking token budget
* Adjusts temperature for reasoning

**Factory:** `createThinkModeHook()`

Sources: [src/index.ts L265-L267](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L265-L267)

 [README.md L688-L689](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L688-L689)

---

### auto-update-checker

**Trigger Events:** `config`, `event`

**Purpose:** Checks for new oh-my-opencode versions on npm and notifies users. Optionally auto-updates the plugin.

**Configuration:**

```

```

**Features:**

* Version comparison with npm registry
* Desktop toast notifications
* Automatic plugin update (when enabled)
* Cache invalidation after update

**Factory:** `createAutoUpdateCheckerHook(ctx, options)`

Sources: [src/index.ts L283-L289](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L283-L289)

 [README.md L694](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L694-L694)

---

### startup-toast

**Trigger Events:** `session.created` (main session only)

**Purpose:** Displays welcome message when OhMyOpenCode loads. Shows version info and feature highlights.

**Display Conditions:**

* Only for main sessions (not subagents)
* First session creation
* Controlled by `auto-update-checker` options

**Message Content:** "oMoMoMo..." with version number

Sources: [src/index.ts L285-L286](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L285-L286)

 [README.md L695](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L695-L695)

---

### background-notification

**Trigger Events:** `session.idle`

**Purpose:** Notifies parent sessions when background agent tasks complete. Integrated with desktop notification system.

**Notification Flow:**

1. Background task completes
2. `BackgroundManager` marks completion
3. Hook detects idle parent session
4. Desktop notification sent with task results

**Factory:** `createBackgroundNotificationHook(backgroundManager)`

Sources: [src/index.ts L308-L310](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L308-L310)

 [README.md L696](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L696-L696)

---

### agent-usage-reminder

**Trigger Events:** `tool.execute.after`, `event`

**Purpose:** Reminds users to leverage specialized agents when directly calling search tools. Suggests delegating to background agents for better results.

**Trigger Tools:**

* `grep`
* `glob`
* `ast_grep_search`

**Reminder Message:** "Consider using @librarian or @explore agents via background_task for more thorough analysis"

**Factory:** `createAgentUsageReminderHook(ctx)`

Sources: [src/index.ts L293-L295](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L293-L295)

 [README.md L691-L692](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L691-L692)

---

## Claude Code Compatibility Hook

### claude-code-hooks

**Trigger Events:** Multiple (all hook types)

**Purpose:** Executes hooks from Claude Code `settings.json` files for backward compatibility. Supports PreToolUse, PostToolUse, UserPromptSubmit, and Stop hooks.

**Search Paths:**

1. `~/.claude/settings.json` (user)
2. `./.claude/settings.json` (project)
3. `./.claude/settings.local.json` (local, git-ignored)

**Supported Hook Types:**

* **PreToolUse:** Runs before tool execution, can block or modify input
* **PostToolUse:** Runs after tool execution, can add warnings/context
* **UserPromptSubmit:** Runs on prompt submission, can block or inject messages
* **Stop:** Runs on session idle, can inject follow-up prompts

**Configuration Toggle:**

```

```

**Factory:** `createClaudeCodeHooksHook(ctx, options)`

Sources: [src/index.ts L268-L270](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L268-L270)

 [README.md L595-L623](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L595-L623)

---

## Hook Execution Order

Within each event type, hooks execute in the order they are registered in the plugin. The following diagram shows typical execution sequences:

```

```

Sources: [src/index.ts L480-L574](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L480-L574)

---

## Advanced Hook Patterns

### State Coordination Between Hooks

Some hooks coordinate state to prevent conflicts:

```

```

This prevents the continuation enforcer from injecting prompts during active error recovery.

Sources: [src/index.ts L244-L248](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L244-L248)

---

### Hook-Specific Experimental Flags

Many hooks respect experimental configuration flags:

```

```

These flags enable preview features or more aggressive behavior for specific hooks.

Sources: [src/config/schema.ts L109-L118](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L109-L118)

 [src/index.ts L237-L254](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L237-L254)

---

## Hook Factory Pattern

All hooks follow a factory pattern for creation:

```

```

Hooks return `null` or partial handlers depending on which lifecycle events they handle.

Sources: [src/hooks/index.ts L1-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts#L1-L24)

 [src/index.ts L230-L304](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L230-L304)