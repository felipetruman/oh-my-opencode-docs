---
layout: default
title: MCP Integration
parent: MCP Integration
nav_order: 1
---

# MCP Integration

> **Relevant source files**
> * [LICENSE.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/LICENSE.md)
> * [README.ja.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ja.md)
> * [README.ko.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md)
> * [README.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md)
> * [README.zh-cn.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.zh-cn.md)
> * [src/mcp/context7.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/context7.ts)
> * [src/mcp/grep-app.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/grep-app.ts)
> * [src/mcp/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts)
> * [src/mcp/types.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts)
> * [src/mcp/websearch-exa.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/websearch-exa.ts)
> * [src/shared/config-path.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts)

This document explains the Micro-Capability Provider (MCP) system in oh-my-opencode, which enables agents to access external knowledge sources and services through remote API endpoints. MCPs extend the agent toolkit beyond local filesystem and code analysis tools to include web search, official documentation lookup, and GitHub code search capabilities.

For information about how agents access and use these tools, see [Tool System](../tools/). For specific tool implementations including LSP and AST-grep, see [LSP Tools](/code-yeongyu/oh-my-opencode/5.1-lsp-tools) and [AST-Grep Tools](/code-yeongyu/oh-my-opencode/5.2-ast-grep-tools).

---

## MCP Architecture

MCPs are remote capability providers that expose specialized search and research tools to agents. The system defines three built-in MCPs, each serving a distinct purpose in the agent research workflow.

```

```

**Sources:** [src/mcp/index.ts L8-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L8-L24)

 [src/mcp/types.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L1-L6)

 [README.md L113-L114](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L113-L114)

The MCP system follows a registry pattern where each MCP is defined as a remote service configuration. The `createBuiltinMcps()` function at [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

 filters the registry based on user configuration, returning only enabled MCPs.

---

## MCP Configuration Structure

Each MCP is defined with three properties:

| Property | Type | Description |
| --- | --- | --- |
| `type` | `"remote"` | Indicates the MCP is accessed via HTTP endpoint |
| `url` | `string` | The remote MCP server endpoint URL |
| `enabled` | `boolean` | Whether the MCP is enabled by default |

```

```

**Sources:** [src/mcp/context7.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/context7.ts#L1-L6)

 [src/mcp/websearch-exa.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/websearch-exa.ts#L1-L6)

 [src/mcp/grep-app.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/grep-app.ts#L1-L6)

---

## Built-in MCPs Overview

oh-my-opencode includes three curated MCPs that provide complementary research capabilities:

| MCP Name | URL | Primary Purpose | Key Features |
| --- | --- | --- | --- |
| `context7` | `https://mcp.context7.com/mcp` | Official documentation lookup | NPM, PyPI, Cargo, package registry docs |
| `websearch_exa` | `https://mcp.exa.ai/mcp?tools=web_search_exa` | Real-time web search | 2025+ filtering, AI-powered search |
| `grep_app` | `https://mcp.grep.app` | GitHub code search | Millions of public repositories |

**Sources:** [src/mcp/context7.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/context7.ts#L1-L6)

 [src/mcp/websearch-exa.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/websearch-exa.ts#L1-L6)

 [src/mcp/grep-app.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/grep-app.ts#L1-L6)

 [README.md L173-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L173-L176)

### context7

Provides access to official documentation for package registries. Agents use this to look up API references, library documentation, and official usage examples.

### websearch_exa

Enables real-time web search with explicit filtering for recent content (2025+). This prevents agents from receiving outdated information that might lead to deprecated API usage.

### grep_app

Offers ultra-fast code search across millions of public GitHub repositories. Particularly useful for finding implementation examples and patterns from open-source projects.

---

## MCP Registration and Filtering

The `createBuiltinMcps()` function implements selective MCP registration based on configuration:

```

```

**Sources:** [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

The filtering logic at [src/mcp/index.ts L17-L20](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L17-L20)

 ensures that only MCPs not present in the `disabledMcps` array are included in the returned configuration. This allows users to selectively disable MCPs while maintaining the default registry intact.

---

## Configuration Options

Users can disable MCPs through the `oh-my-opencode.json` configuration file:

```

```

This configuration would disable web search and GitHub search, leaving only `context7` enabled.

**Sources:** [README.md L1009-L1016](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L1009-L1016)

The `McpName` type at [src/mcp/types.ts L3-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L3-L5)

 uses Zod schema validation to ensure only valid MCP names are accepted in configuration.

---

## Agent MCP Access Patterns

Different agents have different levels of MCP access based on their roles:

```

```

**Sources:** [README.md L464-L476](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L464-L476)

 High-level diagram "Tool and Integration Ecosystem"

### Access Rationale

* **Sisyphus**: Unrestricted access to all MCPs for maximum flexibility in orchestration
* **Oracle**: No MCP access; focuses on analysis using LSP tools and existing context
* **Librarian**: Primary MCP user; designed for external research and documentation lookup
* **Explore**: Limited to `grep_app` for focused code search; no web access prevents scope creep
* **Frontend Engineer**: No MCP access; focuses on implementation rather than research

---

## Claude Code MCP Compatibility

oh-my-opencode loads additional MCPs from Claude Code configuration files for backward compatibility:

```

```

**Sources:** [README.md L629-L633](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L629-L633)

 [README.md L642-L663](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L642-L663)

### MCP Loading Locations

Claude Code MCP configurations are loaded from three locations in priority order:

1. `~/.claude/.mcp.json` (user-level)
2. `./.mcp.json` (project root)
3. `./.claude/.mcp.json` (local, git-ignored)

Each file supports environment variable expansion using `${VAR}` syntax, enabling secure credential management.

### Disabling Claude Code MCP Loading

To disable Claude Code MCP loading while keeping built-in MCPs:

```

```

This toggle at [README.md L642-L663](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L642-L663)

 affects only Claude Code `.mcp.json` files, leaving built-in MCPs (`context7`, `websearch_exa`, `grep_app`) unaffected.

---

## MCP Type System

The MCP type system uses Zod for runtime validation:

```

```

This schema ensures type safety when referencing MCPs in configuration and prevents typos or invalid MCP names from causing runtime errors.

**Sources:** [src/mcp/types.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L1-L6)

---

## Integration with Plugin System

MCPs are registered during plugin initialization as part of the agent factory configuration:

```

```

**Sources:** [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

 [README.md L108-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L108-L117)

The MCP configuration is merged with other plugin components (agents, hooks, tools) to create a complete agent environment. Each agent receives its designated MCP access based on its role and capabilities.

---

## Summary

The MCP system provides a clean abstraction for integrating remote capability providers into the oh-my-opencode agent ecosystem. Key characteristics:

* **Three built-in MCPs**: Documentation lookup, web search, and code search
* **Role-based access**: Agents receive MCP access appropriate to their function
* **Configurable**: Users can disable any MCP via configuration
* **Claude Code compatible**: Loads additional MCPs from `.mcp.json` files
* **Type-safe**: Zod schema validation prevents configuration errors
* **Remote-only**: All MCPs are accessed via HTTP endpoints, no local execution

For detailed information on individual MCPs and their capabilities, see [Built-in MCPs](/code-yeongyu/oh-my-opencode/8.2-built-in-mcps). For MCP system architecture and configuration patterns, see [MCP System Overview](/code-yeongyu/oh-my-opencode/8.1-mcp-system-overview).

**Sources:** [src/mcp/index.ts L1-L25](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L1-L25)

 [README.md L113-L114](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L113-L114)

 [README.md L173-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L173-L176)

 [README.md L563-L568](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L563-L568)