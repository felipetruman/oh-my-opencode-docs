---
layout: default
title: Hooks Integration
parent: Compatibility
nav_order: 1
---

# Hooks Integration

> **Relevant source files**
> * [README.ja.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ja.md)
> * [README.ko.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md)
> * [README.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md)
> * [README.zh-cn.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.zh-cn.md)
> * [src/shared/config-path.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts)

This document describes how oh-my-opencode executes Claude Code's hook system to provide compatibility for users migrating from Claude Code. For information about oh-my-opencode's native hook system, see [Hook Reference](/code-yeongyu/oh-my-opencode/13.4-hook-reference). For other Claude Code compatibility features, see [Configuration Migration](/code-yeongyu/oh-my-opencode/9.1-configuration-migration) and [Compatibility Toggles](/code-yeongyu/oh-my-opencode/9.3-compatibility-toggles).

**Purpose**: The hooks integration layer enables oh-my-opencode to execute custom scripts defined in Claude Code's `settings.json` format, allowing users to run automated actions at specific points in the agent workflow without modifying their existing Claude Code configurations.

**Scope**: This document covers the four Claude Code hook types (PreToolUse, PostToolUse, UserPromptSubmit, Stop), their configuration file locations, execution flow, matcher syntax, and integration with OpenCode's event system.

---

## Hook Types and Event Points

Claude Code defines four hook event types that oh-my-opencode supports. Each hook type executes at a specific point in the agent workflow and has distinct capabilities for intercepting or modifying agent behavior.

| Hook Type | Execution Point | Capabilities | Common Use Cases |
| --- | --- | --- | --- |
| `PreToolUse` | Before tool execution | Block tool call, modify tool input | Validate arguments, inject environment variables, prevent dangerous operations |
| `PostToolUse` | After tool execution | Add warnings/context to tool result | Run formatters (eslint, prettier), log tool usage, inject additional information |
| `UserPromptSubmit` | When user submits a prompt | Block submission, inject messages | Validate prompt content, add context, enforce policies |
| `Stop` | When session becomes idle | Inject follow-up prompts | Auto-continue workflows, schedule tasks, trigger cleanup |

**Sources**: [README.md L593-L597](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L593-L597)

 [README.ko.md L553-L558](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md#L553-L558)

---

## Configuration File Hierarchy

Hook configurations are loaded from multiple locations with a defined precedence order. Files are read in priority sequence, with later files overriding or merging with earlier ones.

### Configuration File Locations

```mermaid
flowchart TD

UserSettings["~/.claude/settings.json<br>(User-wide hooks)"]
ProjectSettings["./.claude/settings.json<br>(Project hooks)"]
LocalSettings["./.claude/settings.local.json<br>(Git-ignored local overrides)"]
Loader["ClaudeCodeHooksLoader"]
PreTool["PreToolUse Hooks"]
PostTool["PostToolUse Hooks"]
PromptSubmit["UserPromptSubmit Hooks"]
Stop["Stop Hooks"]

UserSettings -.-> Loader
ProjectSettings -.-> Loader
LocalSettings -.-> Loader
Loader -.-> PreTool
Loader -.-> PostTool
Loader -.-> PromptSubmit
Loader -.-> Stop

subgraph subGraph3 ["Hook Execution"]
    PreTool
    PostTool
    PromptSubmit
    Stop
end

subgraph subGraph2 ["Hook Loading"]
    Loader
end

subgraph subGraph1 ["Project-Level Configuration"]
    ProjectSettings
    LocalSettings
end

subgraph subGraph0 ["User-Level Configuration"]
    UserSettings
end
```

**Priority Order** (highest to lowest):

1. `./.claude/settings.local.json` - Local overrides (git-ignored)
2. `./.claude/settings.json` - Project-specific hooks
3. `~/.claude/settings.json` - User-wide defaults

**File Format**: All configuration files use JSON format with the following structure:

```
{
  "hooks": {
    "PreToolUse": [...],
    "PostToolUse": [...],
    "UserPromptSubmit": [...],
    "Stop": [...]
  }
}
```

**Sources**: [README.md L586-L591](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L586-L591)

 [README.ja.md L555-L559](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ja.md#L555-L559)

---

## Hook Execution Flow

The following diagram illustrates how hooks integrate with OpenCode's event system and the agent workflow.

### Hook-to-Event Mapping

```mermaid
flowchart TD

ToolExecute["session.tool.execute<br>event"]
PromptEvent["session.message.created<br>event (user message)"]
IdleEvent["session.idle<br>event"]
PreToolDispatch["PreToolUse<br>Hook Dispatcher"]
PostToolDispatch["PostToolUse<br>Hook Dispatcher"]
PromptDispatch["UserPromptSubmit<br>Hook Dispatcher"]
StopDispatch["Stop<br>Hook Dispatcher"]
ToolMatcher["Tool Name Matcher<br>(regex pattern)"]
AlwaysMatch["Always Match<br>(no matcher)"]
CommandHook["Command Hook<br>(run shell command)"]
ScriptHook["Script Hook<br>(execute script file)"]
BlockTool["Block Tool<br>(prevent execution)"]
ModifyInput["Modify Input<br>(transform args)"]
InjectWarning["Inject Warning<br>(add context)"]
InjectPrompt["Inject Prompt<br>(continue session)"]

ToolExecute -.-> PreToolDispatch
ToolExecute -.-> PostToolDispatch
PromptEvent -.-> PromptDispatch
IdleEvent -.-> StopDispatch
PreToolDispatch -.-> ToolMatcher
PostToolDispatch -.-> ToolMatcher
PromptDispatch -.-> AlwaysMatch
StopDispatch -.-> AlwaysMatch
ToolMatcher -.-> CommandHook
ToolMatcher -.-> ScriptHook
AlwaysMatch -.-> CommandHook
AlwaysMatch -.-> ScriptHook
CommandHook -.-> BlockTool
CommandHook -.-> ModifyInput
CommandHook -.-> InjectWarning
CommandHook -.-> InjectPrompt
ScriptHook -.-> BlockTool
ScriptHook -.-> ModifyInput
ScriptHook -.-> InjectWarning
ScriptHook -.-> InjectPrompt

subgraph Results ["Results"]
    BlockTool
    ModifyInput
    InjectWarning
    InjectPrompt
end

subgraph subGraph3 ["Hook Types"]
    CommandHook
    ScriptHook
end

subgraph subGraph2 ["Matcher Evaluation"]
    ToolMatcher
    AlwaysMatch
end

subgraph subGraph1 ["Hook Execution Layer"]
    PreToolDispatch
    PostToolDispatch
    PromptDispatch
    StopDispatch
end

subgraph subGraph0 ["OpenCode Events"]
    ToolExecute
    PromptEvent
    IdleEvent
end
```

**Execution Sequence**:

1. **Event Trigger**: OpenCode event fires (`session.tool.execute`, `session.message.created`, `session.idle`)
2. **Hook Dispatch**: Appropriate hook dispatcher intercepts the event
3. **Matcher Evaluation**: For tool hooks, check if tool name matches the configured pattern
4. **Hook Execution**: Execute matching command or script hooks
5. **Result Processing**: Apply hook results (block, modify, inject content)

**Sources**: [README.md L593-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L593-L610)

---

## Hook Matchers and Filtering

Tool-related hooks (PreToolUse, PostToolUse) support pattern matching to selectively execute only for specific tools.

### Matcher Syntax

The `matcher` field accepts a regular expression pattern that is tested against the tool name:

```
{
  "matcher": "Write|Edit",
  "hooks": [...]
}
```

**Matcher Behavior**:

* **Pattern Matching**: Uses JavaScript regex with case-sensitive matching
* **Pipe Operator**: `|` acts as logical OR (matches any alternative)
* **No Matcher**: If `matcher` is omitted, hook executes for all tool calls
* **Multiple Matchers**: Define separate hook objects for different patterns

### Common Matcher Patterns

| Pattern | Matches | Use Case |
| --- | --- | --- |
| `"Write\|Edit"` | `Write`, `Edit` tools | Run formatter after file modifications |
| `".*"` | All tools | Global logging or validation |
| `"^bash$"` | Exact match for `bash` | Bash-specific environment setup |
| `"lsp_.*"` | All LSP tools | LSP call monitoring |
| `"Read\|List"` | `Read`, `List` tools | Audit file access |

**Sources**: [README.md L599-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L599-L610)

---

## Hook Configuration Examples

### Example 1: Auto-Format on File Write

Run ESLint auto-fix after any file write or edit operation:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "eslint --fix $FILE"
          }
        ]
      }
    ]
  }
}
```

**Available Variables**:

* `$FILE`: Path to the modified file
* `$CWD`: Current working directory
* `$TOOL`: Name of the tool that was executed

**Sources**: [README.md L599-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L599-L610)

### Example 2: Git Auto-Commit

Automatically commit changes after file writes:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "git add $FILE && git commit -m 'Auto-commit: $FILE'"
          }
        ]
      }
    ]
  }
}
```

### Example 3: Prevent Dangerous Bash Commands

Block bash execution for specific commands:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "bash",
        "hooks": [
          {
            "type": "script",
            "script": "./scripts/validate-bash-command.sh",
            "block_on_failure": true
          }
        ]
      }
    ]
  }
}
```

### Example 4: Auto-Continue Workflows

Inject continuation prompt when session goes idle:

```python
{
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Continue with next task from todo list'"
          }
        ]
      }
    ]
  }
}
```

**Sources**: [README.md L599-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L599-L610)

---

## Hook Execution Mechanisms

Hooks support two execution mechanisms: command execution and script execution.

### Command Hooks

Execute shell commands directly with variable substitution:

```json
{
  "type": "command",
  "command": "prettier --write $FILE"
}
```

**Characteristics**:

* **Inline Execution**: Command runs in shell context
* **Variable Expansion**: `$VAR` syntax substituted before execution
* **Exit Code**: Non-zero exit codes can block operations (PreToolUse)
* **Output Capture**: stdout/stderr captured for injection (PostToolUse)

### Script Hooks

Execute external script files:

```json
{
  "type": "script",
  "script": "./hooks/validate.sh",
  "args": ["$FILE", "$TOOL"]
}
```

**Characteristics**:

* **File-Based**: Script must exist at specified path (relative to config file)
* **Arguments**: Supports passing variables as script arguments
* **Reusability**: Share scripts across multiple hook definitions
* **Error Handling**: Script exit codes determine success/failure

**Sources**: [README.md L599-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L599-L610)

---

## Integration with OpenCode Event System

The hooks integration layer bridges Claude Code's hook system with OpenCode's native event architecture.

### Event-to-Hook Translation

```mermaid
flowchart TD

A1["session.tool.execute<br>(before)"]
A2["session.tool.execute<br>(after)"]
A3["session.message.created"]
A4["session.idle"]
B1["PreToolUseHookDispatcher"]
B2["PostToolUseHookDispatcher"]
B3["UserPromptSubmitDispatcher"]
B4["StopHookDispatcher"]
C1["PreToolUse Hooks"]
C2["PostToolUse Hooks"]
C3["UserPromptSubmit Hooks"]
C4["Stop Hooks"]

A1 -.-> B1
A2 -.-> B2
A3 -.-> B3
A4 -.-> B4
B1 -.-> C1
B2 -.-> C2
B3 -.-> C3
B4 -.-> C4

subgraph subGraph2 ["Claude Code Hooks"]
    C1
    C2
    C3
    C4
end

subgraph subGraph1 ["Hook Dispatcher Layer"]
    B1
    B2
    B3
    B4
end

subgraph subGraph0 ["OpenCode SDK Events"]
    A1
    A2
    A3
    A4
end
```

**Event Registration**: The hooks system registers listeners for OpenCode events during plugin initialization. When events fire, the dispatcher evaluates matcher patterns and executes configured hooks.

**Execution Context**: Hooks execute with access to:

* **Tool Context**: Tool name, arguments, execution state
* **Session Context**: Current directory, session ID, user prompt
* **Environment Variables**: Inherited from OpenCode process

**Sources**: [README.md L586-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L586-L610)

---

## Disabling Claude Code Hooks

The hooks integration can be disabled while preserving other Claude Code compatibility features.

### Configuration

Disable hooks in `oh-my-opencode.json`:

```json
{
  "claude_code": {
    "hooks": false
  }
}
```

### Compatibility Matrix

| Feature | Disabled When `hooks: false` | Unaffected |
| --- | --- | --- |
| Hook Execution | `~/.claude/settings.json` hooks | oh-my-opencode native hooks |
| Hook Execution | `./.claude/settings.json` hooks | OpenCode event listeners |
| Hook Execution | `./.claude/settings.local.json` hooks | Other plugins' hooks |
| Configuration Loading | Claude Code hook configs | oh-my-opencode hook configs |

### Alternative: Disable Specific Hook Types

To disable only the Claude Code hooks feature while keeping other hooks active, add to `disabled_hooks`:

```json
{
  "disabled_hooks": ["claude-code-hooks"]
}
```

**Effect**: This disables the Claude Code compatibility hook execution but preserves:

* oh-my-opencode's native hooks (todo-continuation-enforcer, session-recovery, etc.)
* Other Claude Code compatibility features (commands, skills, agents, MCPs)

**Sources**: [README.md L641-L665](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L641-L665)

 [README.ko.md L602-L625](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md#L602-L625)

---

## Data Flow Example

The following diagram illustrates a complete hook execution flow for a file write operation:

### PostToolUse Hook Example: ESLint Auto-Fix

```mermaid
sequenceDiagram
  participant p1 as Agent
  participant p2 as OpenCode
  participant p3 as HookDispatcher
  participant p4 as ESLint
  participant p5 as FileSystem

  p1->>p2: Execute Write tool<br/>(file: "src/app.ts")
  p2->>p5: Write file content
  p5-->>p2: Write complete
  p2->>p3: Emit tool.execute (after)
  p3->>p3: Match "Write" against "Write|Edit"
  p3->>p3: Load PostToolUse hooks
  p3->>p4: Execute: eslint --fix src/app.ts
  p4->>p5: Read src/app.ts
  p4->>p5: Write fixed src/app.ts
  p4-->>p3: Exit code: 0
  p3->>p2: Inject warning:<br/>"ESLint auto-fix applied"
  p2->>p1: Tool result + hook output
```

**Flow Breakdown**:

1. **Tool Execution**: Agent calls Write tool, OpenCode writes file
2. **Event Emission**: OpenCode emits `tool.execute` event (after phase)
3. **Matcher Evaluation**: Hook dispatcher checks if "Write" matches pattern
4. **Hook Execution**: ESLint command runs with `$FILE` substituted
5. **Result Injection**: ESLint output appended to tool result
6. **Agent Response**: Agent receives combined tool result and hook output

**Sources**: [README.md L599-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L599-L610)

---

## Security Considerations

Hook execution introduces security implications that users should understand:

### Command Injection Risks

**Variable Substitution**: Variables like `$FILE` are substituted directly into commands without sanitization. Malicious file paths could inject commands.

**Mitigation**:

* Use script hooks with proper argument quoting instead of command hooks
* Validate file paths in PreToolUse hooks before executing PostToolUse commands
* Avoid using user-controlled input in hook commands

### File System Access

**Hook Privileges**: Hooks execute with the same permissions as the OpenCode process, granting full file system access.

**Best Practices**:

* Store hook scripts in version-controlled directories
* Use `.claude/settings.local.json` for machine-specific hooks (git-ignored)
* Audit hook configurations before sharing projects

### Execution Isolation

**No Sandboxing**: Hooks execute directly in the shell without isolation or resource limits.

**Recommendations**:

* Test hooks in isolated environments before deployment
* Set timeout limits in hook scripts
* Monitor hook execution logs for unexpected behavior

**Sources**: [README.md L586-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L586-L610)

---

## Troubleshooting

### Hooks Not Executing

**Symptom**: Configured hooks don't run when expected.

**Diagnostic Steps**:

1. Verify hook configuration file exists at expected path
2. Check JSON syntax validity (use `jsonlint` or similar)
3. Confirm matcher pattern matches actual tool name
4. Check if `claude_code.hooks` is set to `false` in config
5. Verify hook isn't in `disabled_hooks` array

### Hook Execution Errors

**Symptom**: Hooks execute but fail or produce unexpected results.

**Debugging**:

1. Check shell command/script has execute permissions
2. Verify script paths are relative to configuration file directory
3. Test commands manually with variable substitution
4. Review hook execution logs in OpenCode output
5. Validate exit codes match expected behavior

### Variable Substitution Issues

**Symptom**: Variables like `$FILE` aren't being replaced.

**Causes**:

* Variable only available in specific hook contexts (e.g., `$FILE` only in tool hooks)
* Shell escaping issues with special characters
* Variable case sensitivity

**Solutions**:

* Use double quotes around variables in commands
* Escape special characters in file paths
* Verify variable availability for hook type

**Sources**: Based on common hook configuration patterns

---

## Summary

The Claude Code hooks integration provides a compatibility layer enabling oh-my-opencode users to execute custom scripts at strategic points in the agent workflow. The system supports four hook types (PreToolUse, PostToolUse, UserPromptSubmit, Stop), loads configurations from a hierarchical file structure, and integrates seamlessly with OpenCode's event system.

**Key Capabilities**:

* Execute shell commands or scripts before/after tool execution
* Block operations or modify tool behavior
* Inject context and warnings into tool results
* Auto-continue workflows when sessions go idle
* Pattern-based filtering for selective hook execution

**Integration Points**:

* Claude Code settings.json format compatibility
* OpenCode event system bridging
* oh-my-opencode's native hook architecture
* Session lifecycle and tool execution flow

For managing hook features, see [Compatibility Toggles](/code-yeongyu/oh-my-opencode/9.3-compatibility-toggles). For configuring other Claude Code features, see [Configuration Migration](/code-yeongyu/oh-my-opencode/9.1-configuration-migration).