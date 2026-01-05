---
layout: default
title: Task Execution And Polling
parent: Background Execution
nav_order: 1
---

# Task Execution and Polling

> **Relevant source files**
> * [.opencode/background-tasks.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.opencode/background-tasks.json)
> * [src/features/background-agent/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/index.ts)
> * [src/features/background-agent/manager.test.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.test.ts)
> * [src/features/background-agent/manager.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts)
> * [src/features/background-agent/types.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/types.ts)
> * [src/tools/background-task/tools.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/tools.ts)
> * [src/tools/call-omo-agent/tools.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/tools.ts)

This page documents how background tasks execute and are monitored in the oh-my-opencode system. It covers the task lifecycle, the dual monitoring strategy combining event-driven and polling-based approaches, and the completion detection mechanisms.

For information about task creation and lifecycle management, see [Background Manager](/code-yeongyu/oh-my-opencode/6.1-background-manager). For notification delivery to parent sessions, see [Notification System](/code-yeongyu/oh-my-opencode/6.3-notification-system). For the user-facing tools that interact with tasks, see [Background Task Tools](/code-yeongyu/oh-my-opencode/5.3-background-task-tools).

## Task Lifecycle Overview

Background tasks progress through a defined set of states during their execution. The lifecycle begins when a task is launched and ends when it reaches a terminal state.

### Task States

The system recognizes four possible task states:

| State | Description | Terminal | Triggers Notification |
| --- | --- | --- | --- |
| `running` | Task is actively executing | No | No |
| `completed` | Task finished successfully | Yes | Yes |
| `error` | Task encountered an error | Yes | Yes |
| `cancelled` | Task was cancelled by user or system | Yes | No |

**State Transition Diagram**

```mermaid
stateDiagram-v2
    [*] --> running : "launch()"
    running --> completed : "session.idle + no todos"
    running --> error : "promptAsync error"
    running --> cancelled : "session.abort() / session.deleted"
    completed --> [*] : "session.idle + no todos"
    error --> [*] : "promptAsync error"
    cancelled --> [*]
```

Sources: [src/features/background-agent/types.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/types.ts#L1-L6)

 [src/features/background-agent/manager.ts L88-L103](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L88-L103)

### Initial Task State

When a task is launched via `BackgroundManager.launch()`, it is created with the `running` state. The task structure includes execution context and progress tracking:

```yaml
{
  id: "bg_${crypto.randomUUID().slice(0, 8)}",
  sessionID: string,          // Child session ID
  parentSessionID: string,    // Parent session ID
  parentMessageID: string,    // Originating message
  description: string,        // Short task description
  prompt: string,            // Full agent prompt
  agent: string,             // Agent type (explore/librarian/etc)
  status: "running",
  startedAt: Date,
  progress: {
    toolCalls: 0,
    lastUpdate: Date
  }
}
```

After creation, the system calls `client.session.promptAsync()` to start execution in a fire-and-forget manner, then immediately starts monitoring the task.

Sources: [src/features/background-agent/manager.ts L69-L138](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L69-L138)

## Dual Monitoring Strategy

The system employs two complementary monitoring mechanisms to track task execution: event-driven monitoring for immediate updates and polling-based monitoring for reliable state checking.

### Why Dual Monitoring?

```mermaid
flowchart TD

Events["Event-Based Monitoring<br>(Real-time updates)"]
Polling["Polling-Based Monitoring<br>(Reliable fallback)"]
EventPro["Pros:<br>- Immediate updates<br>- Low overhead<br>- Captures tool calls"]
EventCon["Cons:<br>- May miss events<br>- Depends on SDK<br>- Not guaranteed"]
PollPro["Pros:<br>- Guaranteed check<br>- Catches missed events<br>- Gets full status"]
PollCon["Cons:<br>- 2-second delay<br>- Higher overhead<br>- API calls"]
Both["Combined Approach"]
Result["Result:<br>Fast + Reliable"]

Events -.-> EventPro
Events -.-> EventCon
Polling -.-> PollPro
Polling -.-> PollCon
EventPro -.-> Both
PollPro -.-> Both
Both -.-> Result
```

Sources: [src/features/background-agent/manager.ts L193-L257](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L193-L257)

 [src/features/background-agent/manager.ts L284-L459](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L284-L459)

### Event-Based Monitoring

The `BackgroundManager.handleEvent()` method processes events from the OpenCode SDK's event stream. Three event types are monitored:

**1. message.part.updated Events**

Tracks tool execution progress in real-time:

```mermaid
sequenceDiagram
  participant p1 as OpenCode SDK
  participant p2 as BackgroundManager
  participant p3 as BackgroundTask

  p1->>p2: "message.part.updated<br/>{type: 'tool', tool: 'bash'}"
  p2->>p2: "findBySession(sessionID)"
  p2->>p3: "progress.toolCalls += 1"
  p2->>p3: "progress.lastTool = 'bash'"
  p2->>p3: "progress.lastUpdate = Date()"
```

This provides real-time feedback on task activity without requiring API calls.

Sources: [src/features/background-agent/manager.ts L196-L216](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L196-L216)

**2. session.idle Events**

Primary completion detection mechanism:

```mermaid
flowchart TD

Idle["session.idle event<br>received"]
Find["findBySession(sessionID)"]
Check["Task exists?<br>Status == 'running'?"]
Todos["checkSessionTodos()"]
HasTodos["Has incomplete<br>todos?"]
Wait["Log: waiting for<br>todo-continuation"]
Complete["Set status = 'completed'<br>Set completedAt = Date()"]
Mark["markForNotification(task)"]
Notify["notifyParentSession(task)"]
End["End"]

Idle -.-> Find
Find -.->|"Yes"| Check
Check -.->|"No"| End
Check -.-> Todos
Todos -.-> HasTodos
HasTodos -.->|"Yes"| Wait
HasTodos -.->|"No"| Complete
Complete -.-> Mark
Mark -.-> Notify
Wait -.-> End
Notify -.-> End
```

The todo check is critical: a session may become idle while still having incomplete todos, indicating the task is waiting for todo-continuation enforcement (see [Todo Continuation Enforcer](/code-yeongyu/oh-my-opencode/7.3-todo-continuation-enforcer)).

Sources: [src/features/background-agent/manager.ts L218-L237](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L218-L237)

**3. session.deleted Events**

Handles session deletion gracefully:

* Marks running tasks as `cancelled` with error "Session deleted"
* Removes task from internal tracking
* Clears any pending notifications
* Removes session from `subagentSessions` set

Sources: [src/features/background-agent/manager.ts L239-L257](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L239-L257)

### Polling-Based Monitoring

The `pollRunningTasks()` method runs every 2 seconds when tasks are active, providing a reliable fallback for completion detection.

**Polling Lifecycle**

```mermaid
stateDiagram-v2
    [*] --> Idle : "No running tasks"
    Idle --> Active : "stopPolling()(no running tasks)"
    Active --> Idle : "stopPolling()(no running tasks)"
```

The interval uses `unref()` to prevent keeping the process alive unnecessarily.

Sources: [src/features/background-agent/manager.ts L284-L298](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L284-L298)

**Polling Procedure**

```mermaid
flowchart TD

Start["pollRunningTasks()"]
Status["client.session.status()<br>Get all session statuses"]
Loop["For each running task"]
GetStatus["sessionStatus = allStatuses[task.sessionID]"]
Found["Status found?"]
CheckIdle["sessionStatus.type<br>== 'idle'?"]
CheckTodos["checkSessionTodos()"]
HasTodos["Has incomplete<br>todos?"]
Complete["task.status = 'completed'<br>task.completedAt = Date()"]
Mark["markForNotification(task)"]
Notify["notifyParentSession(task)"]
FetchMsgs["client.session.messages()<br>Get full message history"]
UpdateProgress["Update progress:<br>- toolCalls count<br>- lastTool name<br>- lastMessage text"]
CheckRunning["Has running tasks?"]
Stop["stopPolling()"]
End["Wait 2s"]

Start -.->|"Done"| Status
Status -.-> Loop
Loop -.->|"No"| GetStatus
GetStatus -.->|"Yes"| Found
Found -.->|"No"| Loop
Found -.->|"Yes"| CheckIdle
CheckIdle -.-> CheckTodos
CheckIdle -.->|"Yes"| FetchMsgs
CheckTodos -.->|"No"| HasTodos
HasTodos -.->|"Yes"| Loop
HasTodos -.-> Complete
Complete -.-> Mark
Mark -.-> Notify
Notify -.->|"No"| Loop
FetchMsgs -.-> UpdateProgress
UpdateProgress -.-> Loop
Loop -.-> CheckRunning
CheckRunning -.-> Stop
CheckRunning -.-> End
```

The polling mechanism ensures completion is detected even if events are missed, and provides continuous progress updates by fetching the full message history.

Sources: [src/features/background-agent/manager.ts L380-L459](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L380-L459)

## Progress Tracking

The system maintains detailed progress information throughout task execution, enabling users to monitor long-running operations.

### Progress Structure

```sql
interface TaskProgress {
  toolCalls: number       // Total tool executions
  lastTool?: string      // Most recent tool name
  lastUpdate: Date       // Last progress update
  lastMessage?: string   // Last assistant message text
  lastMessageAt?: Date   // Timestamp of last message
}
```

### Progress Update Sources

| Source | Updates | Frequency | Method |
| --- | --- | --- | --- |
| `message.part.updated` events | `toolCalls`, `lastTool`, `lastUpdate` | Real-time | Event handler |
| `pollRunningTasks()` | All fields including `lastMessage` | Every 2s | Full message fetch |
| Initial creation | `toolCalls: 0`, `lastUpdate` | Once | Task launch |

**Progress Update Flow**

```mermaid
sequenceDiagram
  participant p1 as Background Session
  participant p2 as Event Stream
  participant p3 as BackgroundManager
  participant p4 as Polling Timer
  participant p5 as BackgroundTask

  note over p1: Tool execution
  p1->>p2: "message.part.updated"
  p2->>p3: "handleEvent()"<br/>"progress.toolCalls += 1
  p3->>p5: progress.lastTool = name<br/>progress.lastUpdate = Date()"
  note over p4: Every 2 seconds
  p4->>p3: "pollRunningTasks()"
  p3->>p1: "client.session.messages()"
  p1-->>p3: "Full message history"<br/>"progress.toolCalls = count<br/>progress.lastTool = latest
  p3->>p5: progress.lastMessage = text<br/>progress.lastMessageAt = Date()"
```

The event-based updates provide immediate feedback for tool calls, while polling enriches the progress with the actual message content.

Sources: [src/features/background-agent/manager.ts L196-L216](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L196-L216)

 [src/features/background-agent/manager.ts L410-L450](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L410-L450)

## Completion Detection

Completion detection combines multiple signals to determine when a task has finished executing. The system must distinguish between true completion and temporary idle states.

### Completion Criteria

A task is considered complete when all of the following conditions are met:

```mermaid
flowchart TD

SessionIdle["Session is idle<br>(no active execution)"]
NoTodos["No incomplete todos<br>(all completed/cancelled)"]
Running["Task status is 'running'<br>(not already completed)"]
Complete["✓ Task is complete"]

SessionIdle -.-> Complete
NoTodos -.-> Complete
Running -.-> Complete
```

### Todo Checking

The `checkSessionTodos()` method queries the session's todo list to determine if work remains:

```javascript
// Pseudo-code representation
async checkSessionTodos(sessionID: string): Promise<boolean> {
  const response = await client.session.todo({ path: { id: sessionID } })
  const todos = response.data ?? response
  
  if (!todos || todos.length === 0) return false
  
  const incomplete = todos.filter(
    t => t.status !== "completed" && t.status !== "cancelled"
  )
  
  return incomplete.length > 0
}
```

This check prevents premature completion when the session becomes idle while waiting for todo-continuation to inject a "continue working" prompt.

Sources: [src/features/background-agent/manager.ts L176-L191](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L176-L191)

### Completion Detection Points

The system has two points where completion can be detected:

**1. Event Handler (Immediate)**

When a `session.idle` event arrives, the event handler checks for completion:

```mermaid
sequenceDiagram
  participant p1 as OpenCode SDK
  participant p2 as Event Handler
  participant p3 as checkSessionTodos()
  participant p4 as BackgroundManager

  p1->>p2: "session.idle event"
  p2->>p3: "await checkSessionTodos(sessionID)"
  alt Has incomplete todos
    p3-->>p2: "true"
    note over p2: Log: waiting for todo-continuation
  else No incomplete todos
    p3-->>p2: "false"
    p2->>p4: "task.status = 'completed'"
    p2->>p4: "markForNotification(task)"
    p2->>p4: "notifyParentSession(task)"
    note over p2: Log: completed via session.idle
  end
```

Sources: [src/features/background-agent/manager.ts L218-L237](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L218-L237)

**2. Polling Loop (Reliable)**

The polling loop performs the same check every 2 seconds:

```mermaid
sequenceDiagram
  participant p1 as Polling Timer
  participant p2 as pollRunningTasks()
  participant p3 as client.session.status()
  participant p4 as checkSessionTodos()
  participant p5 as BackgroundManager

  p1->>p2: "Every 2 seconds"
  p2->>p3: "Get all session statuses"
  p3-->>p2: "{ [sessionID]: { type: 'idle' } }"
  note over p2: For task with idle session
  p2->>p4: "await checkSessionTodos(sessionID)"
  alt Has incomplete todos
    p4-->>p2: "true"
    note over p2: Log: has incomplete todos via polling
  else No incomplete todos
    p4-->>p2: "false"
    p2->>p5: "task.status = 'completed'"
    p2->>p5: "markForNotification(task)"
    p2->>p5: "notifyParentSession(task)"
    note over p2: Log: completed via polling
  end
```

Sources: [src/features/background-agent/manager.ts L380-L407](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L380-L407)

## Error Handling

Errors during task execution are captured and stored in the task structure, allowing the parent session to diagnose failures.

### Error Sources

**1. promptAsync Failures**

Errors during the initial prompt submission are caught immediately:

```mermaid
flowchart TD

Launch["manager.launch()"]
Create["Create background session"]
Prompt["client.session.promptAsync()"]
Error["Promise rejected"]
CheckAgent["Error mentions<br>'agent.name'?"]
AgentError["error = 'Agent not found.<br>Make sure agent is registered...'"]
GenericError["error = error.message"]
SetStatus["task.status = 'error'<br>task.completedAt = Date()"]
Mark["markForNotification(task)"]
Notify["notifyParentSession(task)"]

Launch -.-> Create
Create -.-> Prompt
Prompt -.-> Error
Error -.-> CheckAgent
CheckAgent -.->|"Yes"| AgentError
CheckAgent -.->|"No"| GenericError
AgentError -.-> SetStatus
GenericError -.-> SetStatus
SetStatus -.-> Mark
Mark -.-> Notify
```

This provides immediate feedback when an agent is not registered or misconfigured.

Sources: [src/features/background-agent/manager.ts L120-L135](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L120-L135)

**2. Session Deletion**

When a session is deleted, the task is marked as cancelled with a descriptive error:

```
if (task.status === "running") {
  task.status = "cancelled"
  task.completedAt = new Date()
  task.error = "Session deleted"
}
```

Sources: [src/features/background-agent/manager.ts L247-L251](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L247-L251)

### Error State Handling

Tasks in `error` or `cancelled` states are terminal and will not transition to other states:

| Current State | Can Transition To | Notification |
| --- | --- | --- |
| `error` | None | Yes (via notifyParentSession) |
| `cancelled` | None | No (notification cleared) |

The `background_output` tool returns status information for error/cancelled tasks without attempting to wait for completion.

Sources: [src/tools/background-task/tools.ts L258-L261](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/tools.ts#L258-L261)

## Polling Control

The polling mechanism is designed to minimize resource usage when no tasks are active.

### Automatic Start/Stop

```mermaid
stateDiagram-v2
    [*] --> Stopped : "Initial state"
    Stopped --> Running : "No running tasksstopPolling()"
    Running --> Stopped : "No running tasksstopPolling()"
```

**Implementation Details**

| Method | Purpose | Trigger |
| --- | --- | --- |
| `startPolling()` | Creates interval timer if not exists | First task launch |
| `stopPolling()` | Clears interval timer | End of `pollRunningTasks()` when no tasks remain |
| `hasRunningTasks()` | Checks if any task has status `running` | Called by `pollRunningTasks()` |

The `pollingInterval` is created with `setInterval(..., 2000).unref()`, allowing the Node.js process to exit even if the interval is active.

Sources: [src/features/background-agent/manager.ts L284-L298](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L284-L298)

 [src/features/background-agent/manager.ts L373-L378](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L373-L378)

 [src/features/background-agent/manager.ts L456-L459](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L456-L459)

### Polling Performance

Each poll cycle performs:

1. One `client.session.status()` call to get all session statuses
2. For each running task with non-idle status: one `client.session.messages()` call to fetch progress

This means the API call overhead is proportional to the number of active tasks. Idle tasks (waiting for completion confirmation) do not incur additional API calls beyond the initial status check.

**Polling Decision Tree**

```mermaid
flowchart TD

Start["Poll cycle starts"]
StatusCall["client.session.status()<br>(1 API call)"]
Loop["For each running task"]
CheckStatus["Session status<br>found?"]
CheckIdle["Status type<br>== 'idle'?"]
IdlePath["Check todos<br>(1 API call)"]
ActivePath["Fetch messages<br>(1 API call)"]
Complete["Complete task<br>(0 API calls)"]
Update["Update progress<br>(0 API calls)"]
NextTask["Next task"]

Start -.-> StatusCall
StatusCall -.-> Loop
Loop -.-> CheckStatus
CheckStatus -.->|"Yes"| NextTask
CheckStatus -.->|"Yes"| CheckIdle
CheckIdle -.->|"No"| IdlePath
CheckIdle -.->|"No"| ActivePath
IdlePath -.-> Complete
ActivePath -.-> Update
Complete -.-> NextTask
Update -.-> NextTask
NextTask -.-> Loop
```

Sources: [src/features/background-agent/manager.ts L380-L459](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L380-L459)

## Integration with Other Systems

Task execution and polling interact with several other system components:

### Subagent Session Tracking

Background task sessions are registered in the `subagentSessions` Set to prevent certain hooks from interfering with background work:

```sql
// During launch
subagentSessions.add(sessionID)

// During session deletion
subagentSessions.delete(sessionID)
```

This allows hooks like todo-continuation to skip background sessions.

Sources: [src/features/background-agent/manager.ts L86](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L86-L86)

 [src/features/background-agent/manager.ts L255](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L255-L255)

### Message Storage Integration

The manager reads message storage to preserve parent context when notifying:

```javascript
function getMessageDir(sessionID: string): string | null {
  if (!existsSync(MESSAGE_STORAGE)) return null
  
  // Check direct path
  const directPath = join(MESSAGE_STORAGE, sessionID)
  if (existsSync(directPath)) return directPath
  
  // Check nested paths
  for (const dir of readdirSync(MESSAGE_STORAGE)) {
    const sessionPath = join(MESSAGE_STORAGE, dir, sessionID)
    if (existsSync(sessionPath)) return sessionPath
  }
  
  return null
}
```

This enables finding the parent session's model and agent configuration for notification delivery.

Sources: [src/features/background-agent/manager.ts L41-L53](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L41-L53)

 [src/features/background-agent/manager.ts L331-L338](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L331-L338)

### Coordination with Notification System

When a task completes, the manager coordinates notification delivery:

```mermaid
sequenceDiagram
  participant p1 as Polling/Events
  participant p2 as BackgroundManager
  participant p3 as Notification Queue
  participant p4 as Parent Session

  p1->>p2: "Task completed"
  p2->>p2: "markForNotification(task)"
  p2->>p3: "Add to parent's queue"
  p2->>p2: "notifyParentSession(task)"
  note over p2: 200ms delay
  p2->>p4: "client.session.prompt()<br/>with completion message"
  p2->>p3: "clearNotificationsForTask()"
  p2->>p2: "tasks.delete(taskId)"
```

The 200ms delay allows the session to stabilize before injection. For more details on notification delivery, see [Notification System](/code-yeongyu/oh-my-opencode/6.3-notification-system).

Sources: [src/features/background-agent/manager.ts L259-L263](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L259-L263)

 [src/features/background-agent/manager.ts L306-L357](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/features/background-agent/manager.ts#L306-L357)