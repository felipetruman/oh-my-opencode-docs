---
layout: default
title: Explore Agent
parent: Agents
nav_order: 1
---

# Explore Agent

> **Relevant source files**
> * [src/agents/document-writer.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/document-writer.ts)
> * [src/agents/explore.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts)
> * [src/agents/frontend-ui-ux-engineer.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/frontend-ui-ux-engineer.ts)
> * [src/agents/librarian.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/librarian.ts)
> * [src/agents/multimodal-looker.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/multimodal-looker.ts)
> * [src/agents/oracle.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/oracle.ts)

## Purpose and Scope

The Explore Agent is a specialized read-only agent designed for codebase search and discovery operations. It answers questions like "Where is X implemented?", "Which files contain Y?", and "Find the code that does Z" by executing parallel tool searches and returning structured, actionable results. The agent operates as a subagent with no write capabilities, focusing exclusively on finding and analyzing code locations.

For the broader agent orchestration system, see [Agent System](../agents/). For other specialized agents like Librarian (external research) and Oracle (architecture decisions), see [Specialized Agents](/code-yeongyu/oh-my-opencode/4.2-specialized-agents).

**Sources:** [src/agents/explore.ts L1-L106](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L1-L106)

## Agent Configuration

The Explore Agent is configured through the `createExploreAgent` function with the following characteristics:

| Property | Value | Purpose |
| --- | --- | --- |
| **Model** | `opencode/grok-code` (default) | Grok optimized for code understanding |
| **Mode** | `subagent` | Operates as delegated task executor |
| **Temperature** | `0.1` | Deterministic, consistent results |
| **Tools Access** | Read-only | No write, edit, or background_task capabilities |
| **Description** | Contextual grep for codebases | Auto-invoked for location queries |

The agent can be invoked with thoroughness specifications: `"quick"` for basic searches, `"medium"` for moderate analysis, and `"very thorough"` for comprehensive exploration.

**Sources:** [src/agents/explore.ts L1-L12](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L1-L12)

## Intent Analysis Workflow

### Required Pre-Search Analysis

Before executing any search operations, the Explore Agent MUST wrap its analysis in `<analysis>` tags to clarify the actual need behind the literal request:

```mermaid
flowchart TD

UserRequest["User Request<br>'Where is auth?'"]
AnalysisBlock["<analysis> Block"]
LiteralRequest["Literal Request:<br>Location of auth code"]
ActualNeed["Actual Need:<br>Auth flow implementation"]
SuccessCriteria["Success Looks Like:<br>Auth entry points + flow"]
ToolSelection["Tool Selection:<br>LSP + ast_grep + grep"]

UserRequest -.-> AnalysisBlock
AnalysisBlock -.-> LiteralRequest
AnalysisBlock -.-> ActualNeed
AnalysisBlock -.-> SuccessCriteria
SuccessCriteria -.-> ToolSelection
```

**Analysis Structure**

The analysis block must include three mandatory elements:

```javascript
<analysis>
**Literal Request**: [What they literally asked]
**Actual Need**: [What they're really trying to accomplish]
**Success Looks Like**: [What result would let them proceed immediately]
</analysis>
```

This disambiguation step ensures the agent addresses the underlying need rather than just the surface-level query, preventing unnecessary follow-up questions.

**Sources:** [src/agents/explore.ts L26-L33](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L26-L33)

## Parallel Execution Requirements

### Mandatory Parallelism

The Explore Agent enforces **3+ simultaneous tool calls** in its first action. Sequential execution is only permitted when output explicitly depends on prior results.

```mermaid
flowchart TD

Tool1["lsp_workspace_symbols<br>'auth'"]
Tool2["ast_grep_search<br>'function.*auth'"]
Tool3["grep<br>'authenticate'"]
Files1["Files from LSP"]
Files2["Files from AST"]
Files3["Files from grep"]
Merge["Deduplicate &<br>Rank by Relevance"]

Tool1 -.-> Files1
Tool2 -.-> Files2
Tool3 -.-> Files3

subgraph subGraph1 ["Results Aggregation"]
    Files1
    Files2
    Files3
    Merge
    Files1 -.-> Merge
    Files2 -.-> Merge
    Files3 -.-> Merge
end

subgraph subGraph0 ["Single Message - First Action"]
    Tool1
    Tool2
    Tool3
end
```

**Rationale:** Parallel execution maximizes search coverage and reduces total latency. The agent's read-only nature eliminates ordering dependencies for most operations.

**Sources:** [src/agents/explore.ts L35-L36](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L35-L36)

## Tool Selection Strategy

The agent selects tools based on the search pattern type, following a decision matrix:

| Search Pattern | Primary Tool | Secondary Tools | Use Case |
| --- | --- | --- | --- |
| **Semantic** (definitions, references) | LSP tools | `lsp_goto_definition`, `lsp_find_references` | Finding where symbols are defined/used |
| **Structural** (function shapes, class structures) | `ast_grep_search` | `grep` for validation | Pattern matching AST nodes |
| **Text** (strings, comments, logs) | `grep` | `lsp_workspace_symbols` | Literal text search |
| **File patterns** (name/extension) | `glob` | N/A | Finding files by pattern |
| **History** (when added, who changed) | git commands | `git log`, `git blame` | Evolution analysis |
| **External examples** | `grep_app` | Cross-validate with local tools | How others implement patterns |

### grep_app Special Handling

The `grep_app` tool searches millions of public GitHub repositories but requires special treatment:

```mermaid
flowchart TD

GrepApp["grep_app searches<br>(multiple variations)"]
BroadResults["Broad external<br>pattern discovery"]
Warning["⚠️ Results may be<br>outdated/different versions"]
LocalValidation["Cross-validate with<br>local tools"]
Decision["Patterns<br>match?"]
TrustResult["Trust & use result"]
DiscardResult["Discard as outdated"]

GrepApp -.-> BroadResults
BroadResults -.-> Warning
Warning -.-> LocalValidation
LocalValidation -.-> Decision
Decision -.->|"Yes"| TrustResult
Decision -.->|"No"| DiscardResult
```

**Critical Rule:** Always launch multiple `grep_app` calls with query variations in parallel, then cross-validate results with local tools (`grep`, `ast_grep_search`, LSP) before trusting them.

**Sources:** [src/agents/explore.ts L84-L101](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L84-L101)

## Structured Output Format

### Mandatory Results Block

Every response MUST end with a `<results>` block containing three structured sections:

```xml
<results>
<files>
- /absolute/path/to/file1.ts — [why this file is relevant]
- /absolute/path/to/file2.ts — [why this file is relevant]
</files>

<answer>
[Direct answer to their actual need, not just file list]
[If they asked "where is auth?", explain the auth flow you found]
</answer>

<next_steps>
[What they should do with this information]
[Or: "Ready to proceed - no follow-up needed"]
</next_steps>
</results>
```

### Path Requirements

**All file paths MUST be absolute** (starting with `/`). Relative paths are considered a failure condition.

```mermaid
flowchart TD

Invalid["src/auth/middleware.ts<br>./lib/security/jwt.ts"]
Valid["/src/auth/middleware.ts<br>/lib/security/jwt.ts"]

subgraph subGraph1 ["❌ Invalid"]
    Invalid
end

subgraph subGraph0 ["✅ Valid"]
    Valid
end
```

**Sources:** [src/agents/explore.ts L38-L56](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L38-L56)

## Success Criteria and Failure Conditions

### Success Criteria Table

| Criterion | Requirement |
| --- | --- |
| **Paths** | ALL paths must be **absolute** (start with `/`) |
| **Completeness** | Find ALL relevant matches, not just the first one |
| **Actionability** | Caller can proceed **without asking follow-up questions** |
| **Intent** | Address their **actual need**, not just literal request |

### Failure Detection

The response has **FAILED** if any of the following conditions are true:

```mermaid
flowchart TD

Response["Agent Response"]
CheckPath["All paths<br>absolute?"]
CheckComplete["All relevant<br>matches found?"]
CheckActionable["Caller needs<br>follow-up?"]
CheckIntent["Addressed<br>actual need?"]
CheckStructure["<results><br>block present?"]
Success["✅ Success"]
Failure["❌ Failure"]

Response -.-> CheckPath
CheckPath -.->|"No"| Failure
CheckPath -.->|"Yes"| CheckComplete
CheckComplete -.->|"No"| Failure
CheckComplete -.->|"Yes"| CheckActionable
CheckActionable -.->|"Yes"| Failure
CheckActionable -.->|"No"| CheckIntent
CheckIntent -.->|"No"| Failure
CheckIntent -.->|"Yes"| CheckStructure
CheckStructure -.->|"No"| Failure
CheckStructure -.->|"Yes"| Success
```

**Example Failure Indicators:**

* Any path is relative (not absolute)
* Missed obvious matches in the codebase
* Caller needs to ask "but where exactly?" or "what about X?"
* Only answered the literal question, not the underlying need
* No `<results>` block with structured output

**Sources:** [src/agents/explore.ts L58-L74](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L58-L74)

## Constraints and Anti-Patterns

### Hard Constraints

The Explore Agent operates under strict read-only constraints:

| Constraint | Reason |
| --- | --- |
| **No write operations** | `tools: { write: false }` - Search agent cannot modify code |
| **No edit operations** | `tools: { edit: false }` - Prevents unintended modifications |
| **No background tasks** | `tools: { background_task: false }` - Avoid recursive spawning |
| **No emojis in output** | Keep output clean and parseable |
| **No file creation** | Report findings as message text only |

### Anti-Pattern: Sequential Execution

```mermaid
flowchart TD

S1["Tool 1: grep 'auth'"]
S2["Tool 2: ast_grep_search"]
S3["Tool 3: lsp_workspace_symbols"]
P1["Tool 1: grep"]
P2["Tool 2: ast_grep_search"]
P3["Tool 3: lsp_workspace_symbols"]
Result["Single turn,<br>3x faster"]

P1 -.-> Result
P2 -.-> Result
P3 -.-> Result

subgraph subGraph1 ["✅ Good: Parallel"]
    P1
    P2
    P3
end

subgraph subGraph0 ["❌ Bad: Sequential"]
    S1
    S2
    S3
    S1 -.-> S2
    S2 -.-> S3
end
```

**Sources:** [src/agents/explore.ts L76-L81](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L76-L81)

## Integration with Agent System

### Invocation by Sisyphus

Sisyphus invokes the Explore Agent during Phase 2A (Exploration & Research) with parallel background execution:

```mermaid
flowchart TD

Sisyphus["Sisyphus Agent<br>Phase 2A: Exploration"]
Decision["Need code<br>locations?"]
CallExplore["call_omo_agent<br>subagent_type: 'Explore'<br>run_in_background: true"]
BackgroundManager["BackgroundManager<br>Creates child session"]
ExploreAgent["Explore Agent<br>Executes parallel searches"]
Results["Structured <results><br>with file paths"]
Notification["Notify parent session<br>Toast + prompt injection"]
Phase2B["Phase 2B: Implementation<br>with discovered paths"]

Sisyphus -.-> Decision
Decision -.->|"Yes"| CallExplore
CallExplore -.-> BackgroundManager
BackgroundManager -.-> ExploreAgent
ExploreAgent -.-> Results
Results -.-> Notification
Notification -.-> Phase2B
```

### Access Pattern Comparison

| Agent | LSP | AST-Grep | grep_app | Write | Edit | Background Task |
| --- | --- | --- | --- | --- | --- | --- |
| **Explore** | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **Librarian** | ❌ No | ❌ No | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **Oracle** | ✅ Yes | ✅ Yes | ❌ No | ❌ No | ❌ No | ❌ No |
| **Sisyphus** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |

**Key Distinctions:**

* **Explore vs Librarian:** Explore searches *local* codebase with LSP/AST tools; Librarian searches *external* documentation and GitHub repos
* **Explore vs Oracle:** Explore finds locations; Oracle provides architectural reasoning (no external search access)
* **Explore vs Sisyphus:** Explore is read-only subagent; Sisyphus has full write/edit capabilities

**Sources:** [src/agents/explore.ts L1-L106](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L1-L106)

 High-Level Diagram 3 (Agent Orchestration), High-Level Diagram 5 (Tool Ecosystem)

## Code Location Reference

The Explore Agent implementation is located in a single file:

```
src/agents/explore.ts
├── createExploreAgent(model)  [5-103]
│   ├── Agent configuration
│   ├── System prompt with workflow rules
│   └── Tool access restrictions
└── exploreAgent (default export) [105]
```

The agent factory system loads it via:

```
src/agents/builtin-agents.ts
└── Creates explore agent with model override
```

**Sources:** [src/agents/explore.ts L1-L106](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/agents/explore.ts#L1-L106)