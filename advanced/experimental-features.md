---
layout: default
title: Experimental Features
parent: Advanced Topics
nav_order: 1
---

# Experimental Features

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

This page documents advanced context management and optimization features that are under active development. These features provide fine-grained control over token usage, context window management, and session compaction strategies. While stable enough for production use, their behavior and configuration options may change in future releases based on user feedback and performance data.

For standard reliability features like session recovery and context monitoring, see [Reliability System](../reliability/). For MCP integration and external service connections, see [MCP Integration](/code-yeongyu/oh-my-opencode/8-mcp-integration).

---

## Configuration Location

Experimental features are configured in the `experimental` section of `oh-my-opencode.json`:

```json
{
  "experimental": {
    "aggressive_truncation": false,
    "preemptive_compaction": true,
    "preemptive_compaction_threshold": 0.80,
    "truncate_all_tool_outputs": true,
    "dcp_for_compaction": false,
    "dynamic_context_pruning": {
      "enabled": false,
      "notification": "detailed",
      "turn_protection": {
        "enabled": true,
        "turns": 3
      },
      "protected_tools": ["task", "todowrite", "todoread"],
      "strategies": {
        "deduplication": { "enabled": true },
        "supersede_writes": { "enabled": true, "aggressive": false },
        "purge_errors": { "enabled": true, "turns": 5 }
      }
    }
  }
}
```

**Sources:** [src/config/schema.ts L163-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L163-L176)

 [README.md L116](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L116-L116)

---

## Feature Architecture Overview

The experimental features form a layered context management system that intercepts tool outputs, prunes conversation history, and triggers compaction before token limits are reached.

```mermaid
flowchart TD

ToolExec["Tool Execution"]
ToolResult["Tool Result"]
ContextCheck["Context Window Check"]
Truncator["Tool Output Truncator<br>(tool-output-truncator.ts)"]
Monitor["Context Window Monitor<br>(context-window-monitor.ts)"]
Compaction["Preemptive Compaction<br>(preemptive-compaction.ts)"]
AggressiveTrunc["Aggressive Truncation<br>experimental.aggressive_truncation"]
TruncateAll["Truncate All Tools<br>experimental.truncate_all_tool_outputs"]
ThresholdCheck["Threshold Check<br>experimental.preemptive_compaction_threshold"]
DCP["Dynamic Context Pruning<br>experimental.dynamic_context_pruning"]
DCPCompact["DCP for Compaction<br>experimental.dcp_for_compaction"]
TruncateOutput["Truncate Tool Output"]
PruneHistory["Prune Conversation History"]
SummarizeSession["Summarize Session"]

ToolResult -.-> Truncator
Truncator -.-> ContextCheck
ContextCheck -.-> Monitor
Truncator -.-> AggressiveTrunc
Truncator -.-> TruncateAll
AggressiveTrunc -.-> TruncateOutput
TruncateAll -.-> TruncateOutput
Monitor -.-> ThresholdCheck
ThresholdCheck -.-> Compaction
Compaction -.-> DCPCompact
DCP -.-> PruneHistory
Compaction -.-> SummarizeSession
Monitor -.-> DCP
DCP -.-> PruneHistory

subgraph Actions ["Actions"]
    TruncateOutput
    PruneHistory
    SummarizeSession
end

subgraph Strategies ["Strategies"]
    AggressiveTrunc
    TruncateAll
    ThresholdCheck
    DCP
    DCPCompact
    DCPCompact -.-> DCP
end

subgraph subGraph1 ["Experimental Processing"]
    Truncator
    Monitor
    Compaction
end

subgraph subGraph0 ["Event Flow"]
    ToolExec
    ToolResult
    ContextCheck
    ToolExec -.-> ToolResult
end
```

**Sources:** [src/hooks/tool-output-truncator.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/tool-output-truncator.ts)

 [src/hooks/preemptive-compaction.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/preemptive-compaction.ts)

 [src/hooks/context-window-monitor.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/context-window-monitor.ts)

---

## Aggressive Truncation

Aggressive truncation reduces tool output size more aggressively than the default behavior, prioritizing context preservation over output completeness.

### Behavior

When `aggressive_truncation` is enabled, tool outputs are truncated to maintain **50% headroom** in the context window, with a hard cap of **50,000 tokens** per tool output. This is more conservative than the standard truncation strategy.

### Configuration

```json
{
  "experimental": {
    "aggressive_truncation": true,
    "truncate_all_tool_outputs": true
  }
}
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `aggressive_truncation` | boolean | `false` | Enable more aggressive truncation of tool outputs |
| `truncate_all_tool_outputs` | boolean | `true` | Truncate all tool outputs, not just whitelisted tools (grep, glob, LSP, AST-grep) |

### Implementation Details

The truncator hook processes tool results in the `tool.result` event handler:

```mermaid
flowchart TD

ToolResult["tool.result Event"]
CheckEnabled["Check if Hook Enabled"]
GetMetrics["Get Context Usage Metrics"]
CalculateLimit["Calculate Truncation Limit<br>remaining_tokens × 0.5<br>max 50k tokens"]
CheckSize["Check Output Size"]
Truncate["Truncate if Exceeds Limit"]
LogWarning["Log Truncation Warning"]

ToolResult -.-> CheckEnabled
CheckEnabled -.-> GetMetrics
GetMetrics -.-> CalculateLimit
CalculateLimit -.-> CheckSize
CheckSize -.-> Truncate
Truncate -.-> LogWarning
```

The truncation logic maintains the beginning and end of the output with an ellipsis separator, preserving critical context clues for the agent.

**Sources:** [src/hooks/tool-output-truncator.ts L30-L120](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/tool-output-truncator.ts#L30-L120)

 [src/config/schema.ts L164-L171](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L164-L171)

---

## Preemptive Compaction

Preemptive compaction triggers session summarization before the hard context limit is reached, preventing API errors and maintaining conversation flow.

### Trigger Mechanism

Compaction is triggered when context usage exceeds the configured threshold percentage:

```mermaid
flowchart TD

SessionIdle["session.idle Event"]
GetUsage["Get Context Usage<br>used_tokens / total_tokens"]
CheckThreshold["Usage ≥ Threshold?<br>(default: 80%)"]
Continue["Continue Session"]
CheckDCP["DCP Enabled?<br>experimental.dcp_for_compaction"]
RunDCP["Run Dynamic Context Pruning"]
Summarize["Trigger Session Summarization"]
CheckAgain["Still Over Threshold?"]
InjectContext["Inject Critical Context<br>(AGENTS.md, current dir)"]

SessionIdle -.-> GetUsage
GetUsage -.-> CheckThreshold
CheckThreshold -.->|"No"| Continue
CheckThreshold -.->|"Yes"| CheckDCP
CheckDCP -.->|"Yes"| RunDCP
CheckDCP -.->|"No"| Summarize
RunDCP -.-> CheckAgain
CheckAgain -.->|"Yes"| Summarize
CheckAgain -.->|"No"| Continue
Summarize -.-> InjectContext
InjectContext -.-> Continue
```

### Configuration

```json
{
  "experimental": {
    "preemptive_compaction": true,
    "preemptive_compaction_threshold": 0.80
  }
}
```

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `preemptive_compaction` | boolean | `true` | Enable preemptive compaction before hard limits |
| `preemptive_compaction_threshold` | number | `0.80` | Percentage of context window to trigger compaction (0.5-0.95) |

### Context Preservation

The `CompactionContextInjector` preserves critical information during summarization:

1. **AGENTS.md content** - Agent delegation instructions
2. **Current directory info** - Working directory context
3. **Recent tool outputs** - Last N turns of tool results

**Sources:** [src/hooks/preemptive-compaction.ts L20-L180](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/preemptive-compaction.ts#L20-L180)

 [src/hooks/compaction-context-injector.ts L10-L85](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/compaction-context-injector.ts#L10-L85)

---

## Dynamic Context Pruning (DCP)

Dynamic Context Pruning intelligently removes redundant or outdated information from the conversation history without full summarization, preserving the semantic flow while reducing token usage.

### Pruning Strategies

DCP implements three pruning strategies that can be enabled independently:

```mermaid
flowchart TD

FindErrors["Find Errored Tool Calls"]
HistoryAnalysis["Analyze Conversation History"]
FindDupes["Find Duplicate Tool Calls<br>(same tool + same args)"]
FindWrites["Find Write Operations"]
RemoveDupes["Remove All But Last"]
FindReads["Find Subsequent Reads"]
CheckMatch["Same File?"]
PruneWrite["Prune Write Input"]
CheckAge["Age > N Turns?<br>(default: 5)"]
PurgeError["Purge Error Input"]
Protection["Apply Turn Protection<br>(keep last 3 turns)"]
ProtectedTools["Check Protected Tools<br>task, todowrite, todoread, etc."]

subgraph subGraph3 ["DCP Strategy Selection"]
    HistoryAnalysis
    Protection
    ProtectedTools
    HistoryAnalysis -.-> FindDupes
    HistoryAnalysis -.-> FindWrites
    HistoryAnalysis -.-> FindErrors
    RemoveDupes -.-> Protection
    PruneWrite -.-> Protection
    PurgeError -.-> Protection
    Protection -.-> ProtectedTools

subgraph subGraph2 ["Strategy 3: Purge Errors"]
    FindErrors
    CheckAge
    PurgeError
    FindErrors -.-> CheckAge
    CheckAge -.-> PurgeError
end

subgraph subGraph1 ["Strategy 2: Supersede Writes"]
    FindWrites
    FindReads
    CheckMatch
    PruneWrite
    FindWrites -.-> FindReads
    FindReads -.->|"Yes"| CheckMatch
    CheckMatch -.-> PruneWrite
end

subgraph subGraph0 ["Strategy 1: Deduplication"]
    FindDupes
    RemoveDupes
    FindDupes -.->|"Yes"| RemoveDupes
end
end
```

### Configuration

```json
{
  "experimental": {
    "dynamic_context_pruning": {
      "enabled": false,
      "notification": "detailed",
      "turn_protection": {
        "enabled": true,
        "turns": 3
      },
      "protected_tools": [
        "task",
        "todowrite", 
        "todoread",
        "lsp_rename",
        "lsp_code_action_resolve",
        "session_read",
        "session_write",
        "session_search"
      ],
      "strategies": {
        "deduplication": {
          "enabled": true
        },
        "supersede_writes": {
          "enabled": true,
          "aggressive": false
        },
        "purge_errors": {
          "enabled": true,
          "turns": 5
        }
      }
    }
  }
}
```

### Strategy Details

#### 1. Deduplication Strategy

Removes duplicate tool calls where the same tool is invoked with identical arguments. Only the most recent invocation is preserved.

**Example:** If `lsp_goto_definition` is called three times for the same symbol, the first two calls are pruned.

#### 2. Supersede Writes Strategy

Prunes write operation inputs when the file is subsequently read. The rationale: if the file was read later, the agent already has the current state.

**Modes:**

* **Normal mode** (`aggressive: false`): Only prune if the exact same file is read
* **Aggressive mode** (`aggressive: true`): Prune any write if ANY subsequent read occurs (⚠️ may lose context)

#### 3. Purge Errors Strategy

Removes tool call inputs that resulted in errors after a configurable number of turns have passed. The agent has likely moved on from these failed attempts.

**Default:** Purge after 5 turns

### Turn Protection

Turn protection prevents pruning of recent tool outputs to maintain conversation coherence. By default, the last 3 turns are protected from all pruning strategies.

### Protected Tools

Certain tools are never pruned due to their critical role in workflow management:

* `task` - Task delegation tool
* `todowrite` / `todoread` - Todo management
* `lsp_rename` / `lsp_code_action_resolve` - Refactoring operations
* `session_read` / `session_write` / `session_search` - Session management

### Notification Levels

| Level | Behavior |
| --- | --- |
| `off` | Silent pruning, no notifications |
| `minimal` | Toast notification with pruned count |
| `detailed` | Detailed breakdown of pruned items by strategy |

**Sources:** [src/config/schema.ts L127-L161](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L127-L161)

 [README.md L116](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L116-L116)

---

## DCP for Compaction

When enabled, DCP runs as the first step during preemptive compaction, potentially avoiding full summarization if sufficient context can be freed.

### Execution Flow

```mermaid
sequenceDiagram
  participant p1 as Context Window Monitor
  participant p2 as Preemptive Compaction Hook
  participant p3 as Dynamic Context Pruning
  participant p4 as Session Summarization

  p1->>p2: Threshold exceeded (>80%)
  p2->>p2: Check dcp_for_compaction
  alt DCP Enabled
    p2->>p3: Run pruning strategies
    p3->>p3: Apply deduplication
    p3->>p3: Apply supersede writes
    p3->>p3: Apply purge errors
    p3-->>p2: Return pruned count
    p2->>p2: Recalculate usage
  alt Still over threshold
    p2->>p4: Fall back to summarization
  else Below threshold
  else Below threshold
    p2-->>p1: Continue without summarization
  end
  else DCP Disabled
    p2->>p4: Summarize session
  end
  p4->>p2: Inject critical context
  p2-->>p1: Compaction complete
```

### Configuration

```json
{
  "experimental": {
    "dcp_for_compaction": true,
    "preemptive_compaction": true,
    "preemptive_compaction_threshold": 0.80,
    "dynamic_context_pruning": {
      "enabled": true,
      "strategies": {
        "deduplication": { "enabled": true },
        "supersede_writes": { "enabled": true },
        "purge_errors": { "enabled": true, "turns": 5 }
      }
    }
  }
}
```

### Benefits

* **Avoids summarization overhead** - Pruning is faster than full summarization
* **Preserves conversation flow** - No semantic loss from summarization
* **Granular control** - Can configure exactly which content to prune
* **Fallback safety** - Automatically falls back to summarization if pruning is insufficient

**Sources:** [src/hooks/preemptive-compaction.ts L80-L120](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/preemptive-compaction.ts#L80-L120)

 [src/config/schema.ts L175](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L175-L175)

---

## Hook Integration Points

The experimental features integrate with the oh-my-opencode plugin lifecycle through specific hook registration points:

```mermaid
flowchart TD

PluginInit["OhMyOpenCodePlugin Initialization<br>(src/index.ts)"]
LoadConfig["Load Plugin Config<br>experimental section"]
CreateTruncator["createToolOutputTruncatorHook<br>if 'tool-output-truncator' enabled"]
CreateMonitor["createContextWindowMonitorHook<br>if 'context-window-monitor' enabled"]
CreateCompaction["createPreemptiveCompactionHook<br>pass experimental config"]
CreateInjector["createCompactionContextInjector"]
EventHandler["event handler"]
ToolResultHandler["tool.result handler"]

PluginInit -.-> LoadConfig
LoadConfig -.-> CreateTruncator
LoadConfig -.-> CreateMonitor
LoadConfig -.-> CreateCompaction
LoadConfig -.-> CreateInjector
CreateTruncator -.-> ToolResultHandler
CreateMonitor -.-> EventHandler
CreateCompaction -.-> EventHandler

subgraph subGraph1 ["Hook Registration"]
    EventHandler
    ToolResultHandler
end

subgraph subGraph0 ["Hook Creation"]
    CreateTruncator
    CreateMonitor
    CreateCompaction
    CreateInjector
    CreateInjector -.-> CreateCompaction
end
```

The hook creation occurs in the plugin initialization sequence:

| Hook | File | Line | Config Dependency |
| --- | --- | --- | --- |
| Tool Output Truncator | [src/index.ts L251-L253](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L251-L253) | Uses `experimental.aggressive_truncation` and `experimental.truncate_all_tool_outputs` |  |
| Context Window Monitor | [src/index.ts L238-L240](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L238-L240) | None (always enabled unless disabled via `disabled_hooks`) |  |
| Preemptive Compaction | [src/index.ts L273-L277](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L273-L277) | Uses `experimental.preemptive_compaction`, `experimental.preemptive_compaction_threshold`, `experimental.dcp_for_compaction` |  |
| Compaction Context Injector | [src/index.ts L272](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L272-L272) | None (internal callback) |  |

**Sources:** [src/index.ts L219-L277](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L219-L277)

 [src/hooks/index.ts L1-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts#L1-L24)

---

## Complete Configuration Example

A production-ready configuration enabling all experimental features with conservative settings:

```json
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "experimental": {
    "aggressive_truncation": false,
    "preemptive_compaction": true,
    "preemptive_compaction_threshold": 0.75,
    "truncate_all_tool_outputs": true,
    "dcp_for_compaction": true,
    "dynamic_context_pruning": {
      "enabled": true,
      "notification": "detailed",
      "turn_protection": {
        "enabled": true,
        "turns": 3
      },
      "protected_tools": [
        "task",
        "todowrite",
        "todoread",
        "lsp_rename",
        "lsp_code_action_resolve",
        "session_read",
        "session_write",
        "session_search"
      ],
      "strategies": {
        "deduplication": {
          "enabled": true
        },
        "supersede_writes": {
          "enabled": true,
          "aggressive": false
        },
        "purge_errors": {
          "enabled": true,
          "turns": 5
        }
      }
    }
  }
}
```

### Recommended Settings by Use Case

| Use Case | `preemptive_compaction_threshold` | `dcp_for_compaction` | `aggressive_truncation` | `supersede_writes.aggressive` |
| --- | --- | --- | --- | --- |
| **Long-running refactors** | `0.70` | `true` | `false` | `false` |
| **Rapid prototyping** | `0.80` | `true` | `true` | `true` |
| **Production debugging** | `0.75` | `true` | `false` | `false` |
| **Documentation writing** | `0.85` | `false` | `false` | N/A |

**Sources:** [src/config/schema.ts L163-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L163-L176)

 [assets/oh-my-opencode.schema.json L1282-L1532](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1282-L1532)

---

## Disabling Experimental Features

To disable specific experimental hooks, use the `disabled_hooks` configuration:

```json
{
  "disabled_hooks": [
    "tool-output-truncator",
    "context-window-monitor"
  ]
}
```

Available hook names for experimental features:

* `tool-output-truncator` - Disables tool output truncation
* `context-window-monitor` - Disables context usage monitoring (⚠️ not recommended)
* No specific hook name for DCP (controlled via `experimental.dynamic_context_pruning.enabled`)

**Note:** Disabling `context-window-monitor` will prevent preemptive compaction from functioning, as it depends on context usage metrics.

**Sources:** [src/config/schema.ts L45-L68](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L45-L68)

 [src/index.ts L221-L222](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L221-L222)