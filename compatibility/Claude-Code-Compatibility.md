---
layout: default
title: Claude Code Compatibility
parent: Compatibility
nav_order: 1
---

# Claude Code Compatibility

> **Relevant source files**
> * [README.ja.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ja.md)
> * [README.ko.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md)
> * [README.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md)
> * [README.zh-cn.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.zh-cn.md)
> * [src/shared/config-path.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts)

## Purpose and Scope

The Claude Code Compatibility layer enables seamless migration from Claude Code to oh-my-opencode by loading existing Claude Code configurations, commands, skills, agents, MCP servers, and hooks from the `~/.claude/` directory structure. This allows users to retain their custom configurations without modification when switching platforms.

For information about configuring oh-my-opencode-specific features, see [Configuration System](/code-yeongyu/oh-my-opencode/3.2-configuration-system). For details on the built-in agent system, see [Agent System](../agents/).

## Architecture Overview

```mermaid
flowchart TD

ClaudeHome["~/.claude/"]
Settings["settings.json<br>settings.local.json"]
Commands["commands/*.md"]
Skills["skills/*/SKILL.md"]
Agents["agents/*.md"]
MCPs[".mcp.json"]
Todos["todos/*.json"]
Transcripts["transcripts/*.jsonl"]
ProjectRoot["./.claude/"]
ProjectSettings["settings.json<br>settings.local.json"]
ProjectCommands["commands/*.md"]
ProjectSkills["skills/*/SKILL.md"]
ProjectAgents["agents/*.md"]
ProjectMCPs[".mcp.json"]
HooksLoader["Hooks Loader"]
CommandLoader["Command Loader"]
SkillLoader["Skill Loader"]
AgentLoader["Agent Loader"]
MCPLoader["MCP Loader"]
TodoManager["Todo Manager"]
TranscriptLogger["Transcript Logger"]
EventDispatch["Event Dispatch System"]
ToolRegistry["Tool Registry"]
AgentFactory["Agent Factory"]

Settings -.-> HooksLoader
ProjectSettings -.-> HooksLoader
Commands -.-> CommandLoader
ProjectCommands -.-> CommandLoader
Skills -.-> SkillLoader
ProjectSkills -.-> SkillLoader
Agents -.-> AgentLoader
ProjectAgents -.-> AgentLoader
MCPs -.-> MCPLoader
ProjectMCPs -.-> MCPLoader
HooksLoader -.-> EventDispatch
CommandLoader -.-> ToolRegistry
SkillLoader -.-> ToolRegistry
AgentLoader -.-> AgentFactory
MCPLoader -.-> ToolRegistry
TodoManager -.-> Todos
TranscriptLogger -.-> Transcripts
EventDispatch -.-> TodoManager
EventDispatch -.-> TranscriptLogger

subgraph subGraph3 ["Plugin Core"]
    EventDispatch
    ToolRegistry
    AgentFactory
end

subgraph subGraph2 ["Compatibility Layer"]
    HooksLoader
    CommandLoader
    SkillLoader
    AgentLoader
    MCPLoader
    TodoManager
    TranscriptLogger
end

subgraph subGraph1 ["Project Directory Structure"]
    ProjectRoot
    ProjectSettings
    ProjectCommands
    ProjectSkills
    ProjectAgents
    ProjectMCPs
    ProjectRoot -.-> ProjectSettings
    ProjectRoot -.-> ProjectCommands
    ProjectRoot -.-> ProjectSkills
    ProjectRoot -.-> ProjectAgents
    ProjectRoot -.-> ProjectMCPs
end

subgraph subGraph0 ["Claude Code Directory Structure"]
    ClaudeHome
    Settings
    Commands
    Skills
    Agents
    MCPs
    Todos
    Transcripts
    ClaudeHome -.-> Settings
    ClaudeHome -.-> Commands
    ClaudeHome -.-> Skills
    ClaudeHome -.-> Agents
    ClaudeHome -.-> MCPs
    ClaudeHome -.-> Todos
    ClaudeHome -.-> Transcripts
end
```

**Directory Loading Priority**

The compatibility layer loads assets from multiple locations with the following priority order (later sources override earlier ones):

| Asset Type | User-Level | Project-Level | Local (Git-ignored) |
| --- | --- | --- | --- |
| Hooks | `~/.claude/settings.json` | `./.claude/settings.json` | `./.claude/settings.local.json` |
| Commands | `~/.claude/commands/*.md` | `./.claude/commands/*.md` | - |
| Skills | `~/.claude/skills/*/SKILL.md` | `./.claude/skills/*/SKILL.md` | - |
| Agents | `~/.claude/agents/*.md` | `./.claude/agents/*.md` | - |
| MCPs | `~/.claude/.mcp.json` | `./.mcp.json` | `./.claude/.mcp.json` |

Sources: [README.md L578-L662](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L578-L662)

## Configuration File Locations

```mermaid
flowchart TD

ProjectClaudeDir["./.claude/"]
ProjectSettings["settings.json"]
ProjectSettingsLocal["settings.local.json"]
ProjectCommands["commands/"]
ProjectSkills["skills/"]
ProjectAgents["agents/"]
ProjectMCP[".mcp.json"]
ProjectRoot["./.mcp.json"]
UserClaudeDir["~/.claude/"]
UserSettings["settings.json"]
UserCommands["commands/"]
UserSkills["skills/"]
UserAgents["agents/"]
UserMCP[".mcp.json"]
UserTodos["todos/"]
UserTranscripts["transcripts/"]

subgraph subGraph1 ["Project Directory"]
    ProjectClaudeDir
    ProjectSettings
    ProjectSettingsLocal
    ProjectCommands
    ProjectSkills
    ProjectAgents
    ProjectMCP
    ProjectRoot
    ProjectClaudeDir -.-> ProjectSettings
    ProjectClaudeDir -.-> ProjectSettingsLocal
    ProjectClaudeDir -.-> ProjectCommands
    ProjectClaudeDir -.-> ProjectSkills
    ProjectClaudeDir -.-> ProjectAgents
end

subgraph subGraph0 ["User Home Directory"]
    UserClaudeDir
    UserSettings
    UserCommands
    UserSkills
    UserAgents
    UserMCP
    UserTodos
    UserTranscripts
    UserClaudeDir -.-> UserSettings
    UserClaudeDir -.-> UserCommands
    UserClaudeDir -.-> UserSkills
    UserClaudeDir -.-> UserAgents
    UserClaudeDir -.-> UserMCP
    UserClaudeDir -.-> UserTodos
    UserClaudeDir -.-> UserTranscripts
end
```

The compatibility layer scans both user-level (`~/.claude/`) and project-level (`./.claude/` or `./.mcp.json` at project root) directories. Project-level configurations override user-level defaults, enabling team-wide standards while preserving personal preferences.

Sources: [README.md L586-L632](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L586-L632)

## Asset Loaders

### Command Loader

The Command Loader discovers markdown-based slash commands from four directories, merging them into the tool registry:

```mermaid
flowchart TD

ClaudeUserCommands["~/.claude/commands/*.md"]
ClaudeProjectCommands["./.claude/commands/*.md"]
OpenCodeUserCommands["~/.config/opencode/command/*.md"]
OpenCodeProjectCommands["./.opencode/command/*.md"]
MarkdownParser["Markdown Parser"]
FrontmatterExtractor["Frontmatter Extractor"]
CommandRegistry["Command Registry"]
ToolRegistry["Tool Registry"]
SlashCommands["Slash Commands"]

ClaudeUserCommands -.-> MarkdownParser
ClaudeProjectCommands -.-> MarkdownParser
OpenCodeUserCommands -.-> MarkdownParser
OpenCodeProjectCommands -.-> MarkdownParser
CommandRegistry -.-> ToolRegistry

subgraph subGraph2 ["Tool Integration"]
    ToolRegistry
    SlashCommands
    ToolRegistry -.-> SlashCommands
end

subgraph subGraph1 ["Command Parser"]
    MarkdownParser
    FrontmatterExtractor
    CommandRegistry
    MarkdownParser -.-> FrontmatterExtractor
    FrontmatterExtractor -.-> CommandRegistry
end

subgraph subGraph0 ["Command Discovery"]
    ClaudeUserCommands
    ClaudeProjectCommands
    OpenCodeUserCommands
    OpenCodeProjectCommands
end
```

Command files are markdown documents with optional frontmatter for metadata. Each command becomes available as a slash command (e.g., `/command-name`) that agents can invoke.

**Command File Format:**

```go
---
name: example-command
description: Example command description
---

Command instructions go here...
```

Sources: [README.md L614-L618](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L614-L618)

### Skill Loader

Skills are directory-based capabilities with a `SKILL.md` file that defines the skill's behavior:

```mermaid
flowchart TD

UserSkills["~/.claude/skills/*/SKILL.md"]
ProjectSkills["./.claude/skills/*/SKILL.md"]
SkillDir["skill-name/"]
SkillMD["SKILL.md"]
SkillAssets["Additional Files"]
SkillParser["Skill Parser"]
SkillRegistry["Skill Registry"]
SkillTool["skill tool"]

UserSkills -.-> SkillParser
ProjectSkills -.-> SkillParser
SkillMD -.-> SkillParser

subgraph subGraph2 ["Skill Registration"]
    SkillParser
    SkillRegistry
    SkillTool
    SkillParser -.-> SkillRegistry
    SkillRegistry -.-> SkillTool
end

subgraph subGraph1 ["Skill Structure"]
    SkillDir
    SkillMD
    SkillAssets
    SkillDir -.-> SkillMD
    SkillDir -.-> SkillAssets
end

subgraph subGraph0 ["Skill Discovery"]
    UserSkills
    ProjectSkills
end
```

Each skill directory contains a `SKILL.md` file with instructions and may include additional files referenced by the skill. Skills are invoked via the `skill` tool, which passes the skill name and arguments.

**Example Directory Structure:**

```
~/.claude/skills/
├── code-review/
│   └── SKILL.md
├── test-generator/
│   └── SKILL.md
└── documentation/
    └── SKILL.md
```

Sources: [README.md L621-L623](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L621-L623)

### Agent Loader

Custom agents are loaded from markdown files and integrated into the agent factory:

```mermaid
flowchart TD

UserAgents["~/.claude/agents/*.md"]
ProjectAgents["./.claude/agents/*.md"]
AgentMD["agent-name.md"]
AgentMetadata["Frontmatter Metadata"]
AgentPrompt["Agent Instructions"]
AgentParser["Agent Parser"]
AgentFactory["Agent Factory"]
CustomAgents["Custom Agents"]

UserAgents -.-> AgentParser
ProjectAgents -.-> AgentParser
AgentMetadata -.-> AgentParser
AgentPrompt -.-> AgentParser

subgraph subGraph2 ["Agent Registration"]
    AgentParser
    AgentFactory
    CustomAgents
    AgentParser -.-> AgentFactory
    AgentFactory -.-> CustomAgents
end

subgraph subGraph1 ["Agent Definition"]
    AgentMD
    AgentMetadata
    AgentPrompt
    AgentMD -.-> AgentMetadata
    AgentMD -.-> AgentPrompt
end

subgraph subGraph0 ["Agent Discovery"]
    UserAgents
    ProjectAgents
end
```

Custom agents extend the built-in agent system (oracle, librarian, explore, etc.) with project-specific or user-specific agents. Agent files define the agent's model, tools, permissions, and instructions.

**Agent File Format:**

```yaml
---
name: custom-agent
model: anthropic/claude-sonnet-4-5
temperature: 0.7
tools: [read, write, bash]
---

Custom agent instructions...
```

Sources: [README.md L625-L627](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L625-L627)

### MCP Loader

The MCP (Micro-Capability Provider) Loader discovers and registers external MCP servers:

```mermaid
flowchart TD

UserMCP["~/.claude/.mcp.json"]
ProjectRootMCP["./.mcp.json"]
ProjectClaudeMCP["./.claude/.mcp.json"]
MCPConfig["MCP Configuration"]
EnvVars["Environment Variables"]
ServerDefinitions["Server Definitions"]
MCPParser["MCP Parser"]
EnvExpander["Environment Variable Expander"]
MCPRegistry["MCP Registry"]
ExternalTools["External MCP Tools"]

UserMCP -.-> MCPParser
ProjectRootMCP -.-> MCPParser
ProjectClaudeMCP -.-> MCPParser
ServerDefinitions -.-> EnvExpander
EnvVars -.-> EnvExpander

subgraph subGraph2 ["MCP Integration"]
    MCPParser
    EnvExpander
    MCPRegistry
    ExternalTools
    MCPParser -.-> EnvExpander
    EnvExpander -.-> MCPRegistry
    MCPRegistry -.-> ExternalTools
end

subgraph subGraph1 ["MCP Configuration"]
    MCPConfig
    EnvVars
    ServerDefinitions
    MCPConfig -.-> EnvVars
    MCPConfig -.-> ServerDefinitions
end

subgraph subGraph0 ["MCP Discovery"]
    UserMCP
    ProjectRootMCP
    ProjectClaudeMCP
end
```

MCP configuration files define external services that provide additional capabilities. The loader supports environment variable expansion using `${VAR}` syntax.

**MCP Configuration Format:**

```json
{
  "mcpServers": {
    "custom-service": {
      "command": "node",
      "args": ["${HOME}/mcp-servers/custom/index.js"],
      "env": {
        "API_KEY": "${CUSTOM_SERVICE_API_KEY}"
      }
    }
  }
}
```

Sources: [README.md L629-L632](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L629-L632)

## Hooks System

### Hook Types and Events

```mermaid
flowchart TD

PreToolUse["PreToolUse"]
PostToolUse["PostToolUse"]
UserPromptSubmit["UserPromptSubmit"]
Stop["Stop"]
Settings["settings.json"]
Matcher["Hook Matcher<br>(regex pattern)"]
HookActions["Hook Actions"]
EventDispatch["Event Dispatch"]
HookRunner["Hook Runner"]
CommandExec["Command Execution"]
BlockAction["Block Action"]
InjectContext["Context Injection"]

PreToolUse -.-> EventDispatch
PostToolUse -.-> EventDispatch
UserPromptSubmit -.-> EventDispatch
Stop -.-> EventDispatch
Matcher -.-> HookRunner
HookActions -.-> HookRunner

subgraph subGraph2 ["Hook Execution"]
    EventDispatch
    HookRunner
    CommandExec
    BlockAction
    InjectContext
    EventDispatch -.-> HookRunner
    HookRunner -.-> CommandExec
    HookRunner -.-> BlockAction
    HookRunner -.-> InjectContext
end

subgraph subGraph1 ["Hook Configuration"]
    Settings
    Matcher
    HookActions
    Settings -.-> Matcher
    Settings -.-> HookActions
end

subgraph subGraph0 ["Hook Events"]
    PreToolUse
    PostToolUse
    UserPromptSubmit
    Stop
end
```

The hooks system intercepts four key events:

| Event | Timing | Capabilities |
| --- | --- | --- |
| `PreToolUse` | Before tool execution | Block execution, modify tool input |
| `PostToolUse` | After tool execution | Add warnings, inject context |
| `UserPromptSubmit` | When user submits prompt | Block submission, inject messages |
| `Stop` | When session goes idle | Inject follow-up prompts |

Sources: [README.md L593-L609](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L593-L609)

### Hook Execution Flow

```mermaid
sequenceDiagram
  participant p1 as Agent
  participant p2 as EventSystem
  participant p3 as HookLoader
  participant p4 as HookRunner
  participant p5 as FileSystem

  p1->>p2: Trigger Event (e.g., tool.execute)
  p2->>p3: Load Hooks for Event Type
  p3->>p5: Read ~/.claude/settings.json
  p3->>p5: Read ./.claude/settings.json
  p3->>p5: Read ./.claude/settings.local.json
  p5-->>p3: Hook Configurations
  p3-->>p2: Registered Hooks
  p2->>p4: Execute Matching Hooks
  loop For Each Hook
    p4->>p4: Check Matcher Pattern
  alt Matcher Matches
    p4->>p4: Execute Hook Action
  alt Command Hook
    p4->>p5: Execute Shell Command
    p5-->>p4: Command Output
  end
  alt Block Hook
    p4-->>p2: Block Event
  end
  alt Inject Hook
    p4-->>p2: Inject Context
  end
  end
  end
  p4-->>p2: Hook Results
  p2-->>p1: Continue or Block
```

Hooks are executed in the order they are defined in configuration files. Local settings (`./.claude/settings.local.json`) override project settings, which override user settings.

**Hook Configuration Example:**

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
    ],
    "PreToolUse": [
      {
        "matcher": "bash",
        "hooks": [
          {
            "type": "block",
            "message": "Bash execution requires approval"
          }
        ]
      }
    ]
  }
}
```

Sources: [README.md L599-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L599-L610)

### Hook Variables

Hooks support variable substitution for dynamic command execution:

| Variable | Description | Available In |
| --- | --- | --- |
| `$FILE` | File path affected by tool | PostToolUse (Write, Edit) |
| `$TOOL` | Tool name being executed | PreToolUse, PostToolUse |
| `$ARGS` | Tool arguments (JSON) | PreToolUse, PostToolUse |
| `$CWD` | Current working directory | All hooks |
| `$SESSION_ID` | Current session identifier | All hooks |

Sources: [README.md L599-L610](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L599-L610)

## Data Storage

### Todo Management

```mermaid
flowchart TD

TodoCreate["Todo Creation"]
TodoStore["~/.claude/todos/"]
TodoFile["session-id.json"]
TodoUpdate["Todo Updates"]
TodoComplete["Todo Completion"]
TodoJSON["Todo JSON"]
TodoItems["Todo Items"]
TodoMetadata["Metadata"]
SisyphusAgent["Sisyphus Agent"]
TodoEnforcer["Todo Continuation Enforcer"]
SessionState["Session State"]

SisyphusAgent -.-> TodoCreate
TodoUpdate -.-> TodoEnforcer
TodoEnforcer -.-> TodoComplete
TodoItems -.-> SessionState
TodoMetadata -.-> SessionState

subgraph Integration ["Integration"]
    SisyphusAgent
    TodoEnforcer
    SessionState
end

subgraph subGraph1 ["Todo Structure"]
    TodoJSON
    TodoItems
    TodoMetadata
    TodoJSON -.-> TodoItems
    TodoJSON -.-> TodoMetadata
end

subgraph subGraph0 ["Todo Lifecycle"]
    TodoCreate
    TodoStore
    TodoFile
    TodoUpdate
    TodoComplete
    TodoCreate -.-> TodoStore
    TodoStore -.-> TodoFile
    TodoFile -.-> TodoUpdate
end
```

Session todos are stored in `~/.claude/todos/` with one JSON file per session. The format is compatible with Claude Code, enabling seamless transition between platforms.

**Todo File Format:**

```json
{
  "sessionId": "abc123",
  "createdAt": "2025-01-01T00:00:00Z",
  "updatedAt": "2025-01-01T00:30:00Z",
  "items": [
    {
      "id": "1",
      "description": "Implement feature X",
      "status": "in_progress",
      "createdAt": "2025-01-01T00:00:00Z"
    }
  ]
}
```

Sources: [README.md L636-L637](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L636-L637)

### Transcript Logging

```mermaid
flowchart TD

UserMessage["User Messages"]
AgentMessage["Agent Messages"]
ToolExecution["Tool Executions"]
ToolResults["Tool Results"]
TranscriptLogger["Transcript Logger"]
TranscriptFile["~/.claude/transcripts/<br>session-id.jsonl"]
JSONLFormat["JSONL Format"]
TranscriptReader["Transcript Reader"]
SessionReconstruction["Session Reconstruction"]
AnalysisTools["Analysis Tools"]

UserMessage -.-> TranscriptLogger
AgentMessage -.-> TranscriptLogger
ToolExecution -.-> TranscriptLogger
ToolResults -.-> TranscriptLogger
JSONLFormat -.-> TranscriptReader

subgraph subGraph2 ["Replay & Analysis"]
    TranscriptReader
    SessionReconstruction
    AnalysisTools
    TranscriptReader -.-> SessionReconstruction
    TranscriptReader -.-> AnalysisTools
end

subgraph subGraph1 ["Transcript System"]
    TranscriptLogger
    TranscriptFile
    JSONLFormat
    TranscriptLogger -.-> TranscriptFile
    TranscriptFile -.-> JSONLFormat
end

subgraph subGraph0 ["Session Activity"]
    UserMessage
    AgentMessage
    ToolExecution
    ToolResults
end
```

Session activity is logged to `~/.claude/transcripts/` in JSONL (JSON Lines) format, where each line represents a single event. This enables session replay, debugging, and analysis.

**Transcript Entry Format:**

```sql
{"type":"user_message","timestamp":"2025-01-01T00:00:00Z","content":"Implement feature X"}
{"type":"agent_message","timestamp":"2025-01-01T00:00:10Z","content":"I'll create the implementation..."}
{"type":"tool_execution","timestamp":"2025-01-01T00:00:15Z","tool":"write","args":{"path":"src/feature.ts"}}
{"type":"tool_result","timestamp":"2025-01-01T00:00:16Z","tool":"write","result":"File written"}
```

Sources: [README.md L639](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L639-L639)

## Configuration Toggles

```mermaid
flowchart TD

ClaudeCodeConfig["claude_code"]
MCPToggle["mcp: true/false"]
CommandsToggle["commands: true/false"]
SkillsToggle["skills: true/false"]
AgentsToggle["agents: true/false"]
HooksToggle["hooks: true/false"]
MCPLoader["MCP Loader"]
CommandLoader["Command Loader"]
SkillLoader["Skill Loader"]
AgentLoader["Agent Loader"]
HooksLoader["Hooks Loader"]

MCPToggle -.->|"disable"| MCPLoader
CommandsToggle -.-> CommandLoader
SkillsToggle -.-> SkillLoader
AgentsToggle -.-> AgentLoader
HooksToggle -.-> HooksLoader

subgraph subGraph1 ["Loader Behavior"]
    MCPLoader
    CommandLoader
    SkillLoader
    AgentLoader
    HooksLoader
end

subgraph subGraph0 ["Configuration Object"]
    ClaudeCodeConfig
    MCPToggle
    CommandsToggle
    SkillsToggle
    AgentsToggle
    HooksToggle
    ClaudeCodeConfig -.->|"disable"| MCPToggle
    ClaudeCodeConfig -.->|"disable"| CommandsToggle
    ClaudeCodeConfig -.->|"disable"| SkillsToggle
    ClaudeCodeConfig -.-> AgentsToggle
    ClaudeCodeConfig -.->|"disable"| HooksToggle
end
```

The `claude_code` configuration object provides granular control over which compatibility features to enable:

| Toggle | When `false` | Unaffected |
| --- | --- | --- |
| `mcp` | Stops loading `~/.claude/.mcp.json`, `./.mcp.json`, `./.claude/.mcp.json` | Built-in MCPs (context7, websearch_exa, grep_app) |
| `commands` | Stops loading `~/.claude/commands/*.md`, `./.claude/commands/*.md` | `~/.config/opencode/command/`, `./.opencode/command/` |
| `skills` | Stops loading `~/.claude/skills/*/SKILL.md`, `./.claude/skills/*/SKILL.md` | - |
| `agents` | Stops loading `~/.claude/agents/*.md`, `./.claude/agents/*.md` | Built-in agents (oracle, librarian, explore, etc.) |
| `hooks` | Stops loading `~/.claude/settings.json`, `./.claude/settings.json`, `./.claude/settings.local.json` | - |

**Configuration Example:**

```json
{
  "claude_code": {
    "mcp": false,
    "commands": false,
    "skills": false,
    "agents": false,
    "hooks": false
  }
}
```

All toggles default to `true` (enabled). Omitting the `claude_code` object enables full compatibility. This allows users to selectively disable Claude Code features while retaining oh-my-opencode-specific functionality.

Sources: [README.md L641-L664](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L641-L664)

## File Path Resolution

The compatibility layer resolves configuration files using platform-specific paths:

| Platform | User Config Directory | Example |
| --- | --- | --- |
| Linux/macOS | `~/.claude/` | `/home/user/.claude/` |
| Windows | `~/.claude/` or `%USERPROFILE%\.claude\` | `C:\Users\user\.claude\` |

Project-level configurations are always resolved relative to the workspace root:

* `./.claude/` for project-specific overrides
* `./.mcp.json` for project-root MCP configuration

Sources: [README.md L586-L632](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L586-L632)

 [src/shared/config-path.ts L1-L48](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts#L1-L48)

## Migration Strategy

For users migrating from Claude Code:

1. **No Action Required**: Existing `~/.claude/` configurations are automatically discovered and loaded
2. **Incremental Migration**: Use compatibility toggles to gradually transition to oh-my-opencode-specific features
3. **Parallel Operation**: Both Claude Code and oh-my-opencode can share the same `~/.claude/` directory structure
4. **Project Isolation**: Use `./.claude/settings.local.json` for project-specific overrides without affecting the team's shared configuration

The compatibility layer ensures zero-downtime migration while providing an on-ramp to oh-my-opencode's enhanced features.

Sources: [README.md L578-L665](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L578-L665)