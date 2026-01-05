---
layout: default
title: Event Handling
parent: Architecture
nav_order: 1
---

# Event Handling

> **Relevant source files**
> * [assets/oh-my-opencode.schema.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json)
> * [src/config/schema.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts)
> * [src/hooks/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts)
> * [src/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts)

## Purpose and Scope

This document describes the event-driven architecture of oh-my-opencode, explaining how events flow from OpenCode through the plugin to registered hooks. It covers the event types the plugin subscribes to, the dispatch pattern used to invoke hooks, and the data structures passed through the event pipeline.

For information about individual hooks and their behaviors, see [Hook System](../reliability/). For the plugin's initialization and lifecycle management, see [Plugin Lifecycle](/code-yeongyu/oh-my-opencode/3.1-plugin-lifecycle).

---

## Event-Driven Architecture Overview

The oh-my-opencode plugin implements an event-driven architecture where OpenCode emits events at various lifecycle points, and the plugin acts as a dispatcher, routing these events to specialized hooks that implement specific behaviors. The plugin registers multiple event handler functions that OpenCode invokes when specific events occur.

**Event Handler Registration Pattern**

The plugin returns an object with handler functions from its main entry point:

```mermaid
flowchart TD

EventEmitter["Event Emitter"]
PluginReturn["Plugin Return Object<br>src/index.ts:322-575"]
EventHandler["event handler<br>Line 480-540"]
ToolBefore["tool.execute.before<br>Line 542-561"]
ToolAfter["tool.execute.after<br>Line 563-574"]
ChatMessage["chat.message<br>Line 333-336"]
MessagesTransform["experimental.chat.messages.transform<br>Line 338-344"]
ConfigHandler["config<br>Line 346-478"]
SessionRecovery["sessionRecovery"]
TodoContinuation["todoContinuationEnforcer"]
ContextMonitor["contextWindowMonitor"]
ToolTruncator["toolOutputTruncator"]
Others["... 20+ other hooks"]

EventEmitter -.->|"Fires events"| PluginReturn
EventHandler -.->|"Dispatches to"| SessionRecovery
EventHandler -.->|"Dispatches to"| TodoContinuation
EventHandler -.-> ContextMonitor
ToolBefore -.-> ToolTruncator
ToolAfter -.-> ToolTruncator
ChatMessage -.-> Others

subgraph subGraph2 ["Hook Instances"]
    SessionRecovery
    TodoContinuation
    ContextMonitor
    ToolTruncator
    Others
end

subgraph subGraph1 ["OhMyOpenCodePlugin Entry Point"]
    PluginReturn
    EventHandler
    ToolBefore
    ToolAfter
    ChatMessage
    MessagesTransform
    ConfigHandler
    PluginReturn -.->|"Dispatches to"| EventHandler
    PluginReturn -.->|"Dispatches to"| ToolBefore
    PluginReturn -.->|"Dispatches to"| ToolAfter
    PluginReturn -.->|"Dispatches to"| ChatMessage
    PluginReturn -.-> MessagesTransform
    PluginReturn -.-> ConfigHandler
end

subgraph subGraph0 ["OpenCode Core"]
    EventEmitter
end
```

**Sources:** [src/index.ts L211-L576](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L211-L576)

---

## OpenCode Event Types

OpenCode emits events at different points in the session and tool execution lifecycle. The plugin subscribes to the following event categories:

### Event Type Taxonomy

| Event Handler | Purpose | Typical Event Types | Frequency |
| --- | --- | --- | --- |
| `event` | Generic event dispatcher for session lifecycle events | `session.created`, `session.deleted`, `session.error`, `session.idle`, `message.updated` | Per session lifecycle |
| `tool.execute.before` | Pre-tool execution interception | N/A (tool-specific) | Before every tool call |
| `tool.execute.after` | Post-tool execution enhancement | N/A (tool-specific) | After every tool call |
| `chat.message` | User message interception | N/A (message-specific) | Per user message |
| `experimental.chat.messages.transform` | Message array transformation | N/A (message batch) | Before sending to LLM |
| `config` | Configuration finalization | N/A (initialization) | Once per session start |

### Event Properties Structure

Events passed to the generic `event` handler follow this structure:

```yaml
{
  event: {
    type: string,           // e.g., "session.created", "session.error"
    properties: {
      info?: { id?: string, title?: string, parentID?: string },
      sessionID?: string,
      messageID?: string,
      error?: unknown,
      // ... other type-specific properties
    }
  }
}
```

**Sources:** [src/index.ts L480-L540](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L480-L540)

---

## Event Dispatcher Pattern

The plugin uses a sequential dispatch pattern where events are forwarded to multiple hooks in a specific order. Each hook is invoked with `await`, ensuring synchronous processing.

### Main Event Dispatcher

```mermaid
flowchart TD

Event["OpenCode Event"]
Dispatcher["async event handler"]
H1["autoUpdateChecker?.event"]
H2["claudeCodeHooks.event"]
H3["backgroundNotificationHook?.event"]
H4["sessionNotification"]
H5["todoContinuationEnforcer?.handler"]
H6["contextWindowMonitor?.event"]
H7["directoryAgentsInjector?.event"]
H8["directoryReadmeInjector?.event"]
H9["rulesInjector?.event"]
H10["thinkMode?.event"]
H11["anthropicAutoCompact?.event"]
H12["preemptiveCompaction?.event"]
H13["agentUsageReminder?.event"]
H14["interactiveBashSession?.event"]

Event -.-> Dispatcher
Dispatcher -.-> H1

subgraph subGraph1 ["Hook Invocation Sequence"]
    H1
    H2
    H3
    H4
    H5
    H6
    H7
    H8
    H9
    H10
    H11
    H12
    H13
    H14
    H1 -.-> H2
    H2 -.-> H3
    H3 -.-> H4
    H4 -.-> H5
    H5 -.-> H6
    H6 -.-> H7
    H7 -.-> H8
    H8 -.-> H9
    H9 -.-> H10
    H10 -.-> H11
    H11 -.-> H12
    H12 -.-> H13
    H13 -.-> H14
end

subgraph subGraph0 ["Event Dispatcher (src/index.ts:480-540)"]
    Dispatcher
end
```

**Note:** The `?` suffix indicates conditional hooks that may be `null` if disabled via configuration. The dispatcher uses optional chaining (`?.`) to safely skip disabled hooks.

**Hook Enablement Logic**

Hooks are conditionally instantiated based on configuration:

```javascript
const isHookEnabled = (hookName: HookName) => !disabledHooks.has(hookName);

const contextWindowMonitor = isHookEnabled("context-window-monitor")
  ? createContextWindowMonitorHook(ctx)
  : null;
```

**Sources:** [src/index.ts L213-L214](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L213-L214)

 [src/index.ts L233-L235](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L233-L235)

 [src/index.ts L480-L494](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L480-L494)

---

## Session Event Handling

Session events manage the lifecycle of conversation sessions, including creation, deletion, and error handling.

### Session Creation and Tracking

When a session is created, the plugin tracks whether it's a main session or a background (child) session:

```mermaid
flowchart TD

SessionCreated["session.created event"]
CheckParent["Has parentID?"]
SetMain["setMainSession(id)<br>src/features/claude-code-session-state.ts"]
Skip["Skip tracking<br>(background session)"]

SessionCreated -.-> CheckParent
CheckParent -.->|"No"| SetMain
CheckParent -.->|"Yes"| Skip
```

**Code Implementation:**

[src/index.ts L499-L506](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L499-L506)

 processes `session.created` events:

```typescript
if (event.type === "session.created") {
  const sessionInfo = props?.info as
    | { id?: string; title?: string; parentID?: string }
    | undefined;
  if (!sessionInfo?.parentID) {
    setMainSession(sessionInfo?.id);
  }
}
```

### Session Deletion

[src/index.ts L508-L513](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L508-L513)

 handles `session.deleted` events to clean up main session tracking:

```typescript
if (event.type === "session.deleted") {
  const sessionInfo = props?.info as { id?: string } | undefined;
  if (sessionInfo?.id === getMainSessionID()) {
    setMainSession(undefined);
  }
}
```

### Session Error Recovery

Session errors trigger a sophisticated recovery mechanism that coordinates between multiple components:

```mermaid
sequenceDiagram
  participant p1 as OpenCode
  participant p2 as Event Dispatcher
  participant p3 as SessionRecoveryHook
  participant p4 as TodoContinuationEnforcer
  participant p5 as OpenCode Client

  p1->>p2: session.error event
  p2->>p3: isRecoverableError(error)?
  alt Error is recoverable
    p3->>p4: markRecovering(sessionID)
    note over p4: Blocks todo continuation
    p3->>p3: handleSessionRecovery(messageInfo)
    p3->>p3: Manipulate message storage
    p3->>p4: markRecoveryComplete(sessionID)
  alt Recovery successful && main session
    p2->>p5: session.prompt("continue")
  end
  end
```

**Code Implementation:**

[src/index.ts L515-L539](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L515-L539)

 implements the session error handler:

```typescript
if (event.type === "session.error") {
  const sessionID = props?.sessionID as string | undefined;
  const error = props?.error;

  if (sessionRecovery?.isRecoverableError(error)) {
    const messageInfo = {
      id: props?.messageID as string | undefined,
      role: "assistant" as const,
      sessionID,
      error,
    };
    const recovered =
      await sessionRecovery.handleSessionRecovery(messageInfo);

    if (recovered && sessionID && sessionID === getMainSessionID()) {
      await ctx.client.session
        .prompt({
          path: { id: sessionID },
          body: { parts: [{ type: "text", text: "continue" }] },
          query: { directory: ctx.directory },
        })
        .catch(() => {});
    }
  }
}
```

**Coordination Between Recovery and Continuation:**

[src/index.ts L244-L248](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L244-L248)

 wires up callbacks to coordinate state:

```
if (sessionRecovery && todoContinuationEnforcer) {
  sessionRecovery.setOnAbortCallback(todoContinuationEnforcer.markRecovering);
  sessionRecovery.setOnRecoveryCompleteCallback(todoContinuationEnforcer.markRecoveryComplete);
}
```

**Sources:** [src/index.ts L244-L248](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L244-L248)

 [src/index.ts L515-L539](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L515-L539)

---

## Tool Execution Events

Tool execution events enable interception and enhancement of tool calls before and after execution.

### Tool Execution Event Flow

```mermaid
flowchart TD

Agent["Agent decides to use tool"]
Before["tool.execute.before<br>src/index.ts:542-561"]
Execute["OpenCode executes tool"]
After["tool.execute.after<br>src/index.ts:563-574"]
Response["Tool result returned to agent"]
B1["claudeCodeHooks"]
B2["nonInteractiveEnv"]
B3["commentChecker"]
B4["directoryAgentsInjector"]
B5["directoryReadmeInjector"]
B6["rulesInjector"]
B7["Special: task tool handler"]
A1["claudeCodeHooks"]
A2["toolOutputTruncator"]
A3["contextWindowMonitor"]
A4["commentChecker"]
A5["directoryAgentsInjector"]
A6["directoryReadmeInjector"]
A7["rulesInjector"]
A8["emptyTaskResponseDetector"]
A9["agentUsageReminder"]
A10["interactiveBashSession"]

Before -.-> B1
B7 -.-> Execute
After -.-> A1
A10 -.-> Response

subgraph subGraph2 ["After Hooks (Sequential)"]
    A1
    A2
    A3
    A4
    A5
    A6
    A7
    A8
    A9
    A10
    A1 -.-> A2
    A2 -.-> A3
    A3 -.-> A4
    A4 -.-> A5
    A5 -.-> A6
    A6 -.-> A7
    A7 -.-> A8
    A8 -.-> A9
    A9 -.-> A10
end

subgraph subGraph1 ["Before Hooks (Sequential)"]
    B1
    B2
    B3
    B4
    B5
    B6
    B7
    B1 -.-> B2
    B2 -.-> B3
    B3 -.-> B4
    B4 -.-> B5
    B5 -.-> B6
    B6 -.-> B7
end

subgraph subGraph0 ["Tool Call Lifecycle"]
    Agent
    Before
    Execute
    After
    Response
    Agent -.-> Before
    Execute -.-> After
end
```

### Before Execution

The `tool.execute.before` handler receives `input` and `output` parameters where hooks can modify the tool arguments before execution:

```typescript
"tool.execute.before": async (input, output) => {
  await claudeCodeHooks<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.before\"" undefined  file-path="\"tool.execute.before\"">Hii</FileRef>;
  await nonInteractiveEnv?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.before\"" undefined  file-path="\"tool.execute.before\"">Hii</FileRef>;
  await commentChecker?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.before\"" undefined  file-path="\"tool.execute.before\"">Hii</FileRef>;
  await directoryAgentsInjector?.["tool.execute.before"]?.(input, output);
  await directoryReadmeInjector?.["tool.execute.before"]?.(input, output);
  await rulesInjector?.["tool.execute.before"]?.(input, output);

  // Special handling for task tool
  if (input.tool === "task") {
    const args = output.args as Record<string, unknown>;
    const subagentType = args.subagent_type as string;
    const isExploreOrLibrarian = ["explore", "librarian"].includes(subagentType);

    args.tools = {
      ...(args.tools as Record<string, boolean> | undefined),
      background_task: false,
      ...(isExploreOrLibrarian ? { call_omo_agent: false } : {}),
    };
  }
}
```

**Task Tool Filtering:**

[src/index.ts L550-L560](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L550-L560)

 implements special logic to restrict tools available to subagents:

* All subagents have `background_task` disabled (prevents recursive background tasks)
* `explore` and `librarian` agents have `call_omo_agent` disabled (prevents delegation loops)

### After Execution

The `tool.execute.after` handler processes tool results, enabling truncation, validation, and monitoring:

```javascript
"tool.execute.after": async (input, output) => {
  await claudeCodeHooks<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
  await toolOutputTruncator?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
  await contextWindowMonitor?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
  await commentChecker?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
  await directoryAgentsInjector?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
  await directoryReadmeInjector?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
  await rulesInjector?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
  await emptyTaskResponseDetector?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
  await agentUsageReminder?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
  await interactiveBashSession?.<FileRef file-url="https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/\"tool.execute.after\"" undefined  file-path="\"tool.execute.after\"">Hii</FileRef>;
}
```

**Sources:** [src/index.ts L542-L574](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L542-L574)

---

## Message Events

Message events enable interception and transformation of chat messages before they reach the LLM.

### Chat Message Event

The `chat.message` handler processes user messages as they arrive:

```mermaid
flowchart TD

UserMessage["User types message"]
Handler["chat.message handler<br>src/index.ts:333-336"]
ClaudeCode["claudeCodeHooks"]
Keyword["keywordDetector"]
LLM["Message sent to LLM"]

UserMessage -.-> Handler
Handler -.-> ClaudeCode
ClaudeCode -.-> Keyword
Keyword -.-> LLM
```

[src/index.ts L333-L336](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L333-L336)

 implements the handler:

```javascript
"chat.message": async (input, output) => {
  await claudeCodeHooks["chat.message"]?.(input, output);
  await keywordDetector?.["chat.message"]?.(input, output);
}
```

**Keyword Detection Use Case:**

The `keywordDetector` hook scans user messages for special keywords like "ultrawork", "search", or "analyze" to activate different agent modes or behaviors.

### Experimental Message Transform

The `experimental.chat.messages.transform` handler operates on the entire message array before it's sent to the LLM:

```mermaid
flowchart TD

Messages["Message Array<br>(all conversation history)"]
Transform["experimental.chat.messages.transform<br>src/index.ts:338-344"]
Sanitizer["emptyMessageSanitizer"]
LLM["Transformed messages to LLM"]

Messages -.-> Transform
Transform -.-> Sanitizer
Sanitizer -.-> LLM
```

[src/index.ts L338-L344](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L338-L344)

 shows the implementation:

```javascript
"experimental.chat.messages.transform": async (
  input: Record<string, never>,
  output: { messages: Array<{ info: unknown; parts: unknown[] }> }
) => {
  await emptyMessageSanitizer?.["experimental.chat.messages.transform"]?.(input, output as any);
}
```

**Empty Message Sanitizer:**

This hook removes messages with empty `parts` arrays to prevent LLM errors caused by malformed messages.

**Sources:** [src/index.ts L333-L344](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L333-L344)

---

## Event Flow Examples

### Example 1: Tool Execution with Truncation

```mermaid
sequenceDiagram
  participant p1 as Agent
  participant p2 as OpenCode
  participant p3 as Event Dispatcher
  participant p4 as toolOutputTruncator
  participant p5 as contextWindowMonitor

  p1->>p2: Use grep tool
  p2->>p3: tool.execute.before(grep, args)
  p3->>p3: Validate/modify args
  p3->>p2: Continue execution
  p2->>p2: Execute grep
  p2->>p3: tool.execute.after(grep, result)
  p3->>p4: Truncate large output
  p4-->>p3: Modified result
  p3->>p5: Update token count
  p5-->>p3: OK
  p3->>p2: Modified result
  p2->>p1: Truncated grep output
```

### Example 2: Session Error Recovery

```mermaid
sequenceDiagram
  participant p1 as Agent
  participant p2 as OpenCode
  participant p3 as Event Dispatcher
  participant p4 as sessionRecovery
  participant p5 as todoContinuation

  p1->>p2: Executes tool
  p2->>p2: Error occurs
  p2->>p3: event(session.error)
  p3->>p4: isRecoverableError?
  p4-->>p3: true
  p4->>p5: markRecovering()
  note over p5: Blocks continuation prompts
  p4->>p4: handleSessionRecovery()
  note over p4: Manipulates message storage
  p4->>p5: markRecoveryComplete()
  p4-->>p3: recovered = true
  p3->>p2: session.prompt("continue")
  p2->>p1: Resumes conversation
```

**Sources:** [src/index.ts L515-L539](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L515-L539)

 [src/index.ts L542-L574](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L542-L574)

---

## Event Handler Initialization

Event handlers are initialized during plugin setup, with hooks instantiated based on configuration:

```mermaid
flowchart TD

PluginInit["OhMyOpenCodePlugin<br>src/index.ts:211"]
LoadConfig["loadPluginConfig<br>Line 212"]
CheckEnabled["isHookEnabled<br>Line 214"]
CreateHook1["createSessionRecoveryHook"]
CreateHook2["createToolOutputTruncatorHook"]
CreateHook3["createContextWindowMonitorHook"]
CreateHookN["... other hooks"]
ReturnHandlers["Return handler object<br>Line 322-575"]

PluginInit -.-> LoadConfig
LoadConfig -.-> CheckEnabled
CheckEnabled -.-> CreateHook1
CheckEnabled -.-> CreateHook2
CheckEnabled -.-> CreateHook3
CheckEnabled -.-> CreateHookN
CreateHook1 -.-> ReturnHandlers
CreateHook2 -.-> ReturnHandlers
CreateHook3 -.-> ReturnHandlers
CreateHookN -.-> ReturnHandlers

subgraph subGraph0 ["Hook Instantiation"]
    CreateHook1
    CreateHook2
    CreateHook3
    CreateHookN
end
```

**Hook Instantiation Pattern:**

Each hook is conditionally created based on whether it's enabled in configuration:

```javascript
const sessionRecovery = isHookEnabled("session-recovery")
  ? createSessionRecoveryHook(ctx, { experimental: pluginConfig.experimental })
  : null;
```

If a hook is disabled, it's set to `null`, and the event dispatcher uses optional chaining (`?.`) to skip it.

**Sources:** [src/index.ts L211-L304](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L211-L304)

 [src/index.ts L322-L575](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L322-L575)

---

## Configuration Handler

While not strictly an "event," the `config` handler is a special initialization hook that finalizes OpenCode's configuration before session start.

### Configuration Handler Responsibilities

```mermaid
flowchart TD

ConfigHandler["config handler<br>src/index.ts:346-478"]
ModelLimits["Parse model context limits<br>Line 347-370"]
Agents["Register builtin agents<br>Line 372-419"]
Permissions["Set global permissions<br>Line 446-450"]
MCPs["Load MCP servers<br>Line 452-459"]
Commands["Merge commands/skills<br>Line 461-477"]

ConfigHandler -.-> ModelLimits
ConfigHandler -.-> Agents
ConfigHandler -.-> Permissions
ConfigHandler -.-> MCPs
ConfigHandler -.-> Commands

subgraph subGraph0 ["Configuration Tasks"]
    ModelLimits
    Agents
    Permissions
    MCPs
    Commands
end
```

**Model Context Limit Caching:**

[src/index.ts L356-L370](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L356-L370)

 extracts context limits from provider configurations and caches them for use by compaction hooks:

```javascript
if (providers) {
  for (const [providerID, providerConfig] of Object.entries(providers)) {
    const models = providerConfig?.models;
    if (models) {
      for (const [modelID, modelConfig] of Object.entries(models)) {
        const contextLimit = modelConfig?.limit?.context;
        if (contextLimit) {
          modelContextLimitsCache.set(`${providerID}/${modelID}`, contextLimit);
        }
      }
    }
  }
}
```

**Agent Registration:**

[src/index.ts L372-L419](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L372-L419)

 conditionally registers agents based on whether Sisyphus is enabled, demoting `build` and `plan` agents to subagent mode when Sisyphus is active.

**Sources:** [src/index.ts L346-L478](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L346-L478)

---

## Summary

The event handling system in oh-my-opencode provides a robust foundation for intercepting and enhancing OpenCode's behavior:

| Event Handler | Primary Purpose | Dispatch Pattern | Hook Count |
| --- | --- | --- | --- |
| `event` | Session lifecycle management | Sequential await | 14 hooks |
| `tool.execute.before` | Pre-execution validation and argument modification | Sequential await | 7 hooks |
| `tool.execute.after` | Post-execution enhancement and monitoring | Sequential await | 10 hooks |
| `chat.message` | User message interception | Sequential await | 2 hooks |
| `experimental.chat.messages.transform` | Message array sanitization | Sequential await | 1 hook |
| `config` | Configuration finalization | Synchronous | N/A |

**Key Design Patterns:**

1. **Sequential Dispatch:** All hooks are invoked sequentially with `await`, ensuring predictable execution order
2. **Optional Chaining:** Disabled hooks are `null`, and dispatchers use `?.` to skip them
3. **Configuration-Driven:** Hook enablement is controlled via `disabled_hooks` configuration array
4. **Coordination:** Related hooks share state (e.g., session recovery and todo continuation)
5. **Type-Specific Handling:** Event dispatcher includes special logic for specific event types (e.g., `session.error`)

**Sources:** [src/index.ts L211-L576](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L211-L576)

 [src/config/schema.ts L44-L66](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L44-L66)