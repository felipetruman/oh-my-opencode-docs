---
layout: default
title: Agent Reference
parent: Reference
nav_order: 1
---

# Agent Reference Table

> **Relevant source files**
> * [.github/assets/sisyphus.png](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/assets/sisyphus.png)
> * [README.ja.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ja.md)
> * [README.ko.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md)
> * [README.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md)
> * [README.zh-cn.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.zh-cn.md)
> * [src/agents/document-writer.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/document-writer.ts)
> * [src/agents/explore.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts)
> * [src/agents/frontend-ui-ux-engineer.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/frontend-ui-ux-engineer.ts)
> * [src/agents/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/index.ts)
> * [src/agents/librarian.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/librarian.ts)
> * [src/agents/multimodal-looker.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/multimodal-looker.ts)
> * [src/agents/oracle.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/oracle.ts)
> * [src/agents/sisyphus.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/sisyphus.ts)
> * [src/agents/types.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/types.ts)
> * [src/agents/utils.test.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/utils.test.ts)
> * [src/agents/utils.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/utils.ts)
> * [src/shared/config-path.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts)

This page provides a quick reference for all built-in agents in the oh-my-opencode plugin. Each agent is specialized for specific tasks and uses optimized models for their domain. For detailed information about individual agents, see sections [4.1](/code-yeongyu/oh-my-opencode/4.1-sisyphus-orchestrator) through [4.5](#4.5). For agent configuration and customization, see [Agent Configuration](#4.6).

## Agent Overview

The plugin provides seven specialized agents organized into three tiers: orchestration, research, and implementation. All agents except OmO operate as subagents invoked through the `call_omo_agent` tool (see [Agent Invocation Tools](/code-yeongyu/oh-my-opencode/5.3-background-task-tools)) or `background_task` tool (see [Background Tools](/code-yeongyu/oh-my-opencode/6.2-task-execution-and-polling)).

### Primary Agents

| Agent | Mode | Model | Temperature | Thinking Budget | Max Tokens | Primary Role |
| --- | --- | --- | --- | --- | --- | --- |
| **omo** | `primary` | `anthropic/claude-opus-4-5` | default | 32000 | 64000 | Task orchestration, delegation, planning |

### Research & Discovery Agents

| Agent | Mode | Model | Temperature | Special Config | Primary Role |
| --- | --- | --- | --- | --- | --- |
| **oracle** | `subagent` | `openai/gpt-5.2` | 0.1 | `reasoningEffort: medium``textVerbosity: high` | Architecture review, technical decisions |
| **explore** | `subagent` | `opencode/grok-code` | 0.1 | - | Fast internal codebase exploration (read-only) |
| **librarian** | `subagent` | `anthropic/claude-sonnet-4-5` | 0.1 | - | External research, documentation, GitHub (read-only) |

### Implementation Agents

| Agent | Mode | Model | Temperature | Primary Role |
| --- | --- | --- | --- | --- |
| **frontend-ui-ux-engineer** | `subagent` | `google/gemini-3-pro-preview` | default | UI/UX implementation, visual work |
| **document-writer** | `subagent` | `google/gemini-3-pro-preview` | default | Technical documentation generation |
| **multimodal-looker** | `subagent` | `google/gemini-2.5-flash` | 0.1 | Media file analysis (PDFs, images) |

Sources: [src/agents/omo.ts L765-L777](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L765-L777)

 [src/agents/oracle.ts L3-L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/oracle.ts#L3-L11)

 [src/agents/explore.ts L3-L9](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L3-L9)

 [src/agents/librarian.ts L3-L9](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/librarian.ts#L3-L9)

 [src/agents/frontend-ui-ux-engineer.ts L3-L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/frontend-ui-ux-engineer.ts#L3-L7)

 [src/agents/document-writer.ts L3-L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/document-writer.ts#L3-L8)

 [src/agents/multimodal-looker.ts L3-L9](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/multimodal-looker.ts#L3-L9)

## Agent Capabilities Matrix

### Tool Permissions

Each agent has carefully restricted tool access based on their role:

| Agent | read | write | edit | bash | task | call_omo_agent | background_task | Special Tools |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **omo** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | All LSP, AST, file system tools |
| **oracle** | ✓ | ✗ | ✗ | ✗ | ✗ | ✓ | ✗ | Analysis only, can invoke explore/librarian |
| **explore** | ✓ | ✗ | ✗ | ✓ | ✗ | ✗ | ✗ | Read-only bash (git commands) |
| **librarian** | ✓ | ✗ | ✗ | ✓ | ✗ | ✗ | ✗ | Read-only bash, context7, websearch_exa |
| **frontend-ui-ux-engineer** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | Full write access for UI files |
| **document-writer** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | Full write access for docs |
| **multimodal-looker** | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | Read only for media files |

**Key Constraints:**

* **Read-only agents** (explore, librarian): Cannot modify files or create background tasks to prevent accidental changes
* **Oracle**: Can invoke explore/librarian via `call_omo_agent` for research but cannot edit files
* **Multimodal looker**: Isolated context window, single-purpose analysis only

Sources: [src/agents/omo.ts L765-L777](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L765-L777)

 [src/agents/oracle.ts L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/oracle.ts#L11-L11)

 [src/agents/explore.ts L9](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L9-L9)

 [src/agents/librarian.ts L9](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/librarian.ts#L9-L9)

 [src/agents/frontend-ui-ux-engineer.ts L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/frontend-ui-ux-engineer.ts#L8-L8)

 [src/agents/document-writer.ts L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/document-writer.ts#L8-L8)

 [src/agents/multimodal-looker.ts L9](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/multimodal-looker.ts#L9-L9)

## Model Selection Rationale

### Model-to-Purpose Mapping

```

```

**Design Philosophy:**

* **Opus for orchestration**: Highest capability model for complex planning and delegation
* **GPT-5.2 for reasoning**: Advanced reasoning capabilities for architecture decisions
* **Sonnet for research**: High quality external research without opus-level cost
* **Grok for exploration**: Fast, code-specialized model for internal search
* **Gemini Pro for implementation**: Cost-effective code generation for UI and docs
* **Gemini Flash for media**: Ultra-fast, cheap multimodal analysis

Sources: [src/agents/omo.ts L769](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L769-L769)

 [src/agents/oracle.ts L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/oracle.ts#L7-L7)

 [src/agents/explore.ts L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L7-L7)

 [src/agents/librarian.ts L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/librarian.ts#L7-L7)

 [src/agents/frontend-ui-ux-engineer.ts L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/frontend-ui-ux-engineer.ts#L7-L7)

 [src/agents/document-writer.ts L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/document-writer.ts#L7-L7)

 [src/agents/multimodal-looker.ts L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/multimodal-looker.ts#L7-L7)

## Agent Invocation Patterns

### Delegation Hierarchy

```

```

**Invocation Methods:**

| Pattern | Agents | Method | Use Case |
| --- | --- | --- | --- |
| **Parallel Background** | explore, librarian | `background_task(agent="explore", prompt=...)` | Fire 2-3+ agents, continue working, collect later with `background_output` |
| **Synchronous** | frontend-ui-ux-engineer, document-writer | `task(subagent_type="frontend-ui-ux-engineer", prompt=...)` | Block until completion, return full result |
| **Advisory** | oracle | `task(subagent_type="oracle", prompt=...)` OR `call_omo_agent(...)` | Consult for decisions, blocking |
| **Isolated Context** | multimodal-looker | `look_at(file_path=..., goal=...)` | Separate context window, returns extracted data only |

Sources: [src/agents/omo.ts L145-L169](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L145-L169)

 [src/agents/omo.ts L322-L345](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L322-L345)

 [src/tools/background-task/constants.ts L1-L17](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/constants.ts#L1-L17)

 [src/tools/call-omo-agent/constants.ts L3-L25](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/constants.ts#L3-L25)

 [src/tools/look-at/constants.ts L3-L10](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/look-at/constants.ts#L3-L10)

## When to Use Each Agent

### Decision Table

| Question | Agent | Why | Invocation |
| --- | --- | --- | --- |
| Need to design system architecture? | **oracle** | Deep reasoning, architectural expertise | `call_omo_agent` or `task` |
| Need to review my code? | **oracle** | Expert code analysis, security review | `call_omo_agent` or `task` |
| Need to debug complex issue? | **oracle** | Advanced debugging strategy | `call_omo_agent` or `task` |
| Need to find code in THIS codebase? | **explore** (2-3 parallel) | Fast internal search, parallel-friendly | `background_task(agent="explore")` |
| Need to understand module structure? | **explore** (2-3 parallel) | Contextual code understanding | `background_task(agent="explore")` |
| Need official library documentation? | **librarian** | Context7 + web search | `background_task(agent="librarian")` |
| Need GitHub code examples? | **librarian** | GitHub CLI, grep.app | `background_task(agent="librarian")` |
| Need OSS reference implementation? | **librarian** | Remote repo cloning, analysis | `background_task(agent="librarian")` |
| ANY UI/frontend work? | **frontend-ui-ux-engineer** | MANDATORY delegation for .tsx/.jsx/.vue/.css | `task(subagent_type="frontend-ui-ux-engineer")` |
| Need to write documentation? | **document-writer** | Technical writing, README, API docs | `task(subagent_type="document-writer")` |
| Need to analyze PDF/image? | **multimodal-looker** | Gemini multimodal, separate context | `look_at(file_path=..., goal=...)` |

### Parallel Execution Guidelines

**Research Agents (explore, librarian):**

* Always fire as background tasks: `background_task(agent="explore", prompt=...)`
* Launch 2-3 in parallel with different focuses
* Continue working immediately, don't wait
* Collect results later with `background_output(task_id=...)`

**Implementation Agents (frontend, document-writer):**

* Use synchronous `task(subagent_type=...)` calls
* Block until completion
* Cannot run in background

**Advisory Agents (oracle):**

* Use for complex decisions, not simple queries
* Typically synchronous consultation
* Can invoke explore/librarian for additional research

Sources: [src/agents/omo.ts L219-L278](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L219-L278)

 [src/agents/omo.ts L281-L318](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L281-L318)

 [src/agents/omo.ts L504-L513](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L504-L513)

 [src/tools/background-task/constants.ts L1-L17](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/constants.ts#L1-L17)

## Agent Prompt Structures

### 7-Section Prompt Template (Required for task invocations)

When invoking agents via `task(subagent_type=...)`, use this structure:

```yaml
TASK: [Exactly what to do - obsessively specific]
EXPECTED OUTCOME: [Concrete deliverables]
REQUIRED SKILLS: [Which skills to invoke]
REQUIRED TOOLS: [Which tools to use]
MUST DO: [Exhaustive requirements - leave NOTHING implicit]
MUST NOT DO: [Forbidden actions - anticipate rogue behavior]
CONTEXT: [File paths, constraints, related info]
```

**Example - Frontend Delegation:**

```python
task(subagent_type="frontend-ui-ux-engineer", prompt="""
TASK: Implement responsive navigation bar with hamburger menu
EXPECTED OUTCOME: Working navigation component with mobile breakpoint at 768px
REQUIRED SKILLS: frontend-ui-ux-engineer
REQUIRED TOOLS: read, edit, grep (for existing patterns)
MUST DO: Follow existing design system, match current styling patterns
MUST NOT DO: Add new dependencies, break existing styles
CONTEXT: src/components/Header.tsx, use Tailwind classes from tailwind.config.js
""")
```

### Background Task Prompts (For explore, librarian)

For `background_task(agent=...)` calls, provide detailed, focused prompts:

**Example - Explore Agent:**

```

```

**Example - Librarian Agent:**

```

```

**Critical Rules:**

* Always write prompts in **English** regardless of user's language (LLMs perform better)
* Be specific about what to find, not just keywords
* For explore: Specify thoroughness level ("quick", "medium", "very thorough")
* For librarian: Specify which type (docs, GitHub, OSS implementation)

Sources: [src/agents/omo.ts L346-L360](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L346-L360)

 [src/tools/background-task/constants.ts L10-L16](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/constants.ts#L10-L16)

 [src/agents/explore.ts L40-L68](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L40-L68)

 [src/agents/librarian.ts L26-L129](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/librarian.ts#L26-L129)

## Agent Configuration Override

All agents can be customized through configuration. See [Agent Configuration](#4.6) for full details.

### Configuration Schema Location

Agent overrides are defined in `.opencode/oh-my-opencode.json` or `~/.config/opencode/oh-my-opencode.json`:

```

```

**Common Overrides:**

* `model`: Change the underlying model
* `temperature`: Adjust creativity (0.0-1.0)
* `maxTokens`: Increase output capacity
* `disabled_agents`: Array of agent names to disable

Sources: Agent configuration is handled by the config system, but agent definitions are in [src/agents/omo.ts L765-L777](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/omo.ts#L765-L777)

 [src/agents/oracle.ts L3-L77](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/oracle.ts#L3-L77)

 [src/agents/explore.ts L3-L257](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L3-L257)

 [src/agents/librarian.ts L3-L240](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/librarian.ts#L3-L240)

 [src/agents/frontend-ui-ux-engineer.ts L3-L92](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/frontend-ui-ux-engineer.ts#L3-L92)

 [src/agents/document-writer.ts L3-L203](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/document-writer.ts#L3-L203)

 [src/agents/multimodal-looker.ts L3-L42](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/multimodal-looker.ts#L3-L42)

## Access Control Summary

### Read-Only Agents (Exploration & Research)

**explore** and **librarian** are strictly read-only to prevent accidental file modifications:

* Cannot write, edit, or delete files
* Cannot create background tasks (prevents recursion)
* Cannot use `call_omo_agent` (prevents delegation loops)
* Can use bash for read-only commands (git log, git blame)
* Purpose: Safe, parallel exploration without side effects

Sources: [src/agents/explore.ts L9-L22](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L9-L22)

 [src/agents/librarian.ts L9](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/librarian.ts#L9-L9)

### Advisory Agent (Oracle)

**oracle** has restricted permissions optimized for analysis:

* Can read files for review
* Can invoke explore/librarian via `call_omo_agent` for research
* Cannot write, edit, or execute bash
* Cannot create background tasks
* Purpose: Pure advisory, no implementation

Sources: [src/agents/oracle.ts L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/oracle.ts#L11-L11)

### Implementation Agents

**frontend-ui-ux-engineer** and **document-writer** have write access:

* Full file read/write/edit permissions
* Can use bash for build/test commands
* Can invoke other agents via task tool
* Cannot create background tasks
* Purpose: Direct implementation work

Sources: [src/agents/frontend-ui-ux-engineer.ts L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/frontend-ui-ux-engineer.ts#L8-L8)

 [src/agents/document-writer.ts L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/document-writer.ts#L8-L8)

### Isolated Context Agent

**multimodal-looker** operates in separate context:

* Only Read tool access
* No write, edit, bash, or delegation
* Separate context window to save tokens
* Returns extracted data only
* Purpose: Media analysis without polluting main context

Sources: [src/agents/multimodal-looker.ts L9](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/multimodal-looker.ts#L9-L9)

 [src/tools/look-at/constants.ts L3-L10](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/look-at/constants.ts#L3-L10)