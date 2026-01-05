---
layout: default
title: LSP Tools
parent: Tools
nav_order: 1
---

# Enhanced Base Tools

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

This page documents oh-my-opencode's improvements to OpenCode's base tool system. Enhanced base tools address critical limitations in OpenCode's built-in `grep` and `glob` tools (timeout handling, output truncation) and extend LSP capabilities beyond analysis to include refactoring operations. For structural code search using AST-aware patterns, see [Search and Analysis Tools](/code-yeongyu/oh-my-opencode/5.2-ast-grep-tools). For custom tools like `interactive_bash` and `look_at`, see [Custom Tools](/code-yeongyu/oh-my-opencode/5.4-session-management-tools).

---

## Overview

OpenCode provides several base tools that agents use to interact with codebases. However, these tools have limitations that can degrade agent performance or cause sessions to hang. oh-my-opencode replaces problematic base tools with enhanced versions and extends LSP capabilities with refactoring operations.

### Tool Enhancement Strategy

```mermaid
flowchart TD

BaseGrep["grep<br>(no timeout)"]
BaseGlob["glob<br>(no timeout)"]
BaseLSP["LSP Tools<br>(analysis only)"]
EnhGrep["grep<br>+ timeout<br>+ output truncation<br>+ context-aware limits"]
EnhGlob["glob<br>+ timeout<br>+ output truncation<br>+ context-aware limits"]
LSPRefactor["LSP Refactoring Tools<br>+ lsp_rename<br>+ lsp_prepare_rename<br>+ lsp_code_actions<br>+ lsp_code_action_resolve"]
NoHang["Sessions don't hang"]
ContextProtect["Context window protected"]
Refactor["Can refactor code"]

BaseGrep -.->|"Replaced by"| EnhGrep
BaseGlob -.->|"Replaced by"| EnhGlob
BaseLSP -.->|"Extended with"| LSPRefactor
EnhGrep -.-> NoHang
EnhGrep -.-> ContextProtect
EnhGlob -.-> NoHang
EnhGlob -.-> ContextProtect
LSPRefactor -.-> Refactor

subgraph subGraph2 ["Agent Experience"]
    NoHang
    ContextProtect
    Refactor
end

subgraph subGraph1 ["oh-my-opencode Enhanced Tools"]
    EnhGrep
    EnhGlob
    LSPRefactor
end

subgraph subGraph0 ["OpenCode Base Tools"]
    BaseGrep
    BaseGlob
    BaseLSP
end
```

**Sources:** README.md:586-587, Diagram 5 from system architecture

---

## Enhanced grep Tool

The enhanced `grep` tool addresses critical issues in OpenCode's base implementation by adding timeout protection and output truncation.

### Problems with Base grep

| Issue | Impact | Agent Behavior |
| --- | --- | --- |
| No timeout mechanism | Infinite hang on large searches | Session becomes unresponsive |
| Unlimited output | Context window overflow | Agent loses important context |
| No early termination | Resource exhaustion | System slowdown |

### Enhancement Features

**Timeout Protection**

The enhanced grep implements configurable timeouts to prevent infinite hangs:

* Default timeout prevents indefinite blocking
* Graceful termination when timeout exceeded
* Error messages indicate timeout occurred

**Output Truncation**

Dynamic output limiting based on context window usage:

* Monitors remaining context window capacity
* Truncates output to maintain 50% headroom
* Maximum cap of 50,000 tokens per result
* Preserves most relevant matches at start of output

**Context-Aware Behavior**

```mermaid
flowchart TD

Start["Agent calls grep"]
CheckContext["Check context<br>window usage"]
Execute["Execute search<br>with timeout"]
CheckOutput["Output size check"]
Truncate["Apply truncation<br>if needed"]
Return["Return results"]

subgraph subGraph0 ["grep Execution Flow"]
    Start
    CheckContext
    Execute
    CheckOutput
    Truncate
    Return
    Start -.-> CheckContext
    CheckContext -.-> Execute
    Execute -.-> CheckOutput
    CheckOutput -.->|"Too large"| Truncate
    CheckOutput -.->|"Acceptable"| Return
    Truncate -.-> Return
end
```

**Sources:** README.md:699-701, Diagram 5

---

## Enhanced glob Tool

The enhanced `glob` tool mirrors grep enhancements, providing timeout protection and output management for file pattern matching operations.

### Problems with Base glob

| Issue | Impact | Agent Behavior |
| --- | --- | --- |
| No timeout on large directories | Session hangs during recursive glob | Agent cannot proceed |
| Unlimited file list output | Context window overflow | Agent loses workspace awareness |
| No result limiting | Performance degradation | Slow response times |

### Enhancement Features

**Timeout Protection**

* Prevents infinite recursion in deep directory trees
* Configurable timeout per glob operation
* Graceful fallback when timeout exceeded

**Output Management**

* Limits number of matching files returned
* Truncates file lists when too large
* Maintains context window budget
* Prioritizes most relevant matches

### Integration with Tool Output Truncator

Both enhanced grep and glob work in conjunction with the Tool Output Truncator hook (see [Tool Enhancement Hooks](/code-yeongyu/oh-my-opencode/7.3-todo-continuation-enforcer)):

```mermaid
flowchart TD

ToolCall["Agent calls<br>grep or glob"]
EnhTool["Enhanced tool<br>with timeout"]
TruncHook["Tool Output<br>Truncator Hook"]
FinalResult["Final truncated<br>result"]
Monitor["Context Window<br>Monitor"]
Budget["50% headroom<br>50k token cap"]

Monitor -.-> TruncHook
Budget -.-> TruncHook

subgraph subGraph1 ["Context Management"]
    Monitor
    Budget
end

subgraph subGraph0 ["Tool Execution Pipeline"]
    ToolCall
    EnhTool
    TruncHook
    FinalResult
    ToolCall -.-> EnhTool
    EnhTool -.-> TruncHook
    TruncHook -.-> FinalResult
end
```

**Sources:** README.md:699-702, README.md:586-587

---

## LSP Refactoring Tools

OpenCode provides LSP tools for analysis (hover, goto definition, find references). oh-my-opencode extends this with refactoring capabilities that enable agents to modify code structurally.

### Available LSP Tools

#### Analysis Tools (OpenCode Base)

| Tool | Description | Use Case |
| --- | --- | --- |
| `lsp_hover` | Type info, docs, signatures at cursor position | Understanding code semantics |
| `lsp_goto_definition` | Jump to symbol definition | Code navigation |
| `lsp_find_references` | Find all symbol usages | Impact analysis |
| `lsp_document_symbols` | File symbol outline | Understanding file structure |
| `lsp_workspace_symbols` | Search symbols by name | Finding declarations |
| `lsp_diagnostics` | Get errors/warnings | Pre-build validation |
| `lsp_servers` | List available LSP servers | Tool availability |

#### Refactoring Tools (oh-my-opencode)

| Tool | Description | Use Case |
| --- | --- | --- |
| `lsp_prepare_rename` | Validate rename operation before execution | Safety check |
| `lsp_rename` | Rename symbol across entire workspace | Refactoring |
| `lsp_code_actions` | Get available quick fixes and refactorings | Discovering fixes |
| `lsp_code_action_resolve` | Apply selected code action | Executing refactoring |

### lsp_rename Tool

Enables workspace-wide symbol renaming with LSP accuracy.

**Input Schema:**

* `path`: File containing the symbol
* `position`: Cursor position on symbol (line, character)
* `newName`: New name for the symbol

**Workflow:**

```mermaid
flowchart TD

Start["Agent identifies<br>symbol to rename"]
Prepare["lsp_prepare_rename<br>(optional validation)"]
Check["Check if rename<br>is valid"]
Execute["lsp_rename with<br>new name"]
Changes["LSP calculates<br>all changes"]
Apply["Apply changes<br>across workspace"]
Complete["Return success<br>+ files modified"]
Error["Return error"]

Check -.-> Error

subgraph subGraph0 ["Rename Operation"]
    Start
    Prepare
    Check
    Execute
    Changes
    Apply
    Complete
    Start -.-> Prepare
    Prepare -.->|"Valid"| Check
    Check -.->|"Invalid"| Execute
    Execute -.-> Changes
    Changes -.-> Apply
    Apply -.-> Complete
end
```

**Example Use Cases:**

* Renaming a function used across multiple files
* Changing a class name and updating all references
* Refactoring variable names for clarity

### lsp_code_actions Tool

Retrieves available quick fixes and refactorings for a code location.

**Input Schema:**

* `path`: File path
* `range`: Text range (start/end line and character)
* `diagnostics`: Optional diagnostics to filter actions

**Output:**

* List of available code actions (quick fixes, refactorings, source actions)
* Each action includes: kind, title, edit details

**Action Kinds:**

* `quickfix`: Fix errors or warnings
* `refactor`: Structural refactorings
* `refactor.extract`: Extract method/variable
* `refactor.inline`: Inline variable/method
* `refactor.rewrite`: Rewrite code structure
* `source`: Source-level actions (organize imports, etc.)

### lsp_code_action_resolve Tool

Executes a selected code action from `lsp_code_actions`.

**Input Schema:**

* `codeAction`: The code action object to execute

**Workflow:**

```mermaid
flowchart TD

Detect["Agent detects<br>code smell"]
Query["lsp_code_actions<br>at location"]
List["List available<br>fixes"]
Select["Agent selects<br>appropriate action"]
Resolve["lsp_code_action_resolve"]
Apply["Apply workspace<br>edits"]
Verify["Verify changes"]

subgraph subGraph0 ["Code Action Execution"]
    Detect
    Query
    List
    Select
    Resolve
    Apply
    Verify
    Detect -.-> Query
    Query -.-> List
    List -.-> Select
    Select -.-> Resolve
    Resolve -.-> Apply
    Apply -.-> Verify
end
```

### LSP Server Configuration

LSP tools require LSP servers to be configured in OpenCode. oh-my-opencode respects all OpenCode LSP configuration and adds additional configuration options.

**Configuration Location:**

* `~/.config/opencode/oh-my-opencode.json` (user)
* `.opencode/oh-my-opencode.json` (project)

**Example Configuration:**

```json
{
  "lsp": {
    "typescript-language-server": {
      "command": ["typescript-language-server", "--stdio"],
      "extensions": [".ts", ".tsx"],
      "priority": 10
    },
    "pylsp": {
      "command": ["pylsp"],
      "extensions": [".py"],
      "disabled": false
    }
  }
}
```

**Configuration Options:**

| Option | Type | Description |
| --- | --- | --- |
| `command` | string[] | Command and arguments to start LSP server |
| `extensions` | string[] | File extensions this server handles |
| `priority` | number | Server priority when multiple match |
| `env` | object | Environment variables for server process |
| `initialization` | object | LSP initialization options |
| `disabled` | boolean | Disable this server |

**Sources:** README.md:536-549, README.md:812-839

---

## Tool Permission System

Enhanced tools respect OpenCode's permission system, which can be configured per-agent.

### Permission Levels

```mermaid
flowchart TD

Ask["ask<br>(prompt user)"]
Allow["allow<br>(auto-approve)"]
Deny["deny<br>(reject)"]
Edit["edit<br>(lsp_rename, etc.)"]
Read["read<br>(grep, glob, lsp_hover)"]
Bash["bash<br>(command execution)"]

Edit -.-> Ask
Edit -.-> Allow
Edit -.-> Deny
Read -.-> Ask
Read -.-> Allow
Read -.-> Deny
Bash -.-> Ask
Bash -.-> Allow
Bash -.-> Deny

subgraph subGraph1 ["Tool Categories"]
    Edit
    Read
    Bash
end

subgraph subGraph0 ["Permission Hierarchy"]
    Ask
    Allow
    Deny
end
```

### Configuring Tool Permissions

Permissions are set per-agent in the configuration:

```json
{
  "agents": {
    "explore": {
      "permission": {
        "edit": "deny",
        "bash": "ask",
        "webfetch": "allow"
      }
    }
  }
}
```

**Permission Categories:**

| Category | Affected Tools | Default |
| --- | --- | --- |
| `edit` | `lsp_rename`, `lsp_code_action_resolve`, file write operations | `ask` |
| `bash` | Command execution tools | `ask` |
| `webfetch` | Web request tools | `ask` |
| `external_directory` | Access outside project root | `deny` |

**Sources:** README.md:773-796, Diagram 5

---

## Tool Registration Flow

Enhanced tools are registered during plugin initialization, replacing or extending base OpenCode tools.

```mermaid
flowchart TD

PluginLoad["OhMyOpenCodePlugin.load()"]
ConfigLoad["Load configuration"]
ToolRegistry["Tool registration"]
UnregisterBase["Unregister base<br>grep/glob"]
RegisterEnh["Register enhanced<br>grep/glob"]
RegisterLSP["Register LSP<br>refactoring tools"]
ApplyPerms["Apply permission<br>configuration"]
AgentCall["Agent uses tools"]
EnhExec["Enhanced tool<br>execution"]
PermCheck["Permission check"]
Result["Return result"]

ToolRegistry -.-> UnregisterBase
ApplyPerms -.-> AgentCall

subgraph Runtime ["Runtime"]
    AgentCall
    EnhExec
    PermCheck
    Result
    AgentCall -.-> EnhExec
    EnhExec -.-> PermCheck
    PermCheck -.-> Result
end

subgraph subGraph1 ["Tool Registration"]
    UnregisterBase
    RegisterEnh
    RegisterLSP
    ApplyPerms
    UnregisterBase -.-> RegisterEnh
    RegisterEnh -.-> RegisterLSP
    RegisterLSP -.-> ApplyPerms
end

subgraph subGraph0 ["Plugin Initialization"]
    PluginLoad
    ConfigLoad
    ToolRegistry
    PluginLoad -.-> ConfigLoad
    ConfigLoad -.-> ToolRegistry
end
```

**Sources:** Diagram 1, Diagram 5

---

## Summary

Enhanced base tools provide critical improvements to OpenCode's tool system:

| Enhancement | Problem Solved | Agent Benefit |
| --- | --- | --- |
| grep timeout | Infinite hangs | Reliable search operations |
| glob timeout | Infinite recursion | Safe file system traversal |
| Output truncation | Context overflow | Maintained context awareness |
| LSP refactoring | Analysis-only limitation | Structural code modifications |
| Permission system | Unrestricted tool access | Safe agent operations |

These enhancements enable agents to work more reliably and safely while maintaining full IDE-level capabilities for code navigation and refactoring.

**Sources:** README.md:522-589, Diagram 5, README.md:812-839