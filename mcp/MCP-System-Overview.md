---
layout: default
title: MCP System Overview
parent: MCP Integration
nav_order: 1
---

# MCP System Overview

> **Relevant source files**
> * [LICENSE.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/LICENSE.md)
> * [src/mcp/context7.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/context7.ts)
> * [src/mcp/grep-app.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/grep-app.ts)
> * [src/mcp/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts)
> * [src/mcp/types.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts)
> * [src/mcp/websearch-exa.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/websearch-exa.ts)

This document explains the Micro-Capability Provider (MCP) architecture in oh-my-opencode, including how remote services are integrated as tools, the configuration structure, and mechanisms for selective enabling/disabling.

For documentation of the specific built-in MCPs (context7, websearch_exa, grep_app), see [Built-in MCPs](/code-yeongyu/oh-my-opencode/8.2-built-in-mcps). For information about how agents access MCP tools, see [Tool System](../tools/) and [Agent Configuration](/code-yeongyu/oh-my-opencode/4.3-agent-configuration).

## Purpose and Scope

MCPs (Micro-Capability Providers) extend oh-my-opencode's capabilities by integrating external services as tools that agents can use. This system enables:

* Integration of remote HTTP services without custom code
* Standardized configuration format for all MCPs
* Selective enabling/disabling of MCPs per project or user
* Type-safe MCP names and configurations

## What are MCPs

MCPs are external services that provide additional capabilities to agents beyond the core tool set. Each MCP:

* Exposes its capabilities through a remote HTTP endpoint
* Follows the OpenCode SDK's remote tool integration protocol
* Is registered as a set of tools available to specific agents
* Can be independently enabled or disabled via configuration

The built-in MCPs provide three categories of external knowledge:

| MCP Name | Purpose | Primary Users |
| --- | --- | --- |
| `context7` | Official documentation for package ecosystems (NPM, PyPI, Cargo, etc.) | Librarian |
| `websearch_exa` | Real-time web search with 2025+ filtering | Librarian |
| `grep_app` | GitHub code search across millions of repositories | Librarian, Explore |

Sources: [src/mcp/index.ts L8-L12](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L8-L12)

 [src/mcp/types.ts L3](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L3-L3)

## MCP Architecture

The MCP system uses a factory pattern to instantiate MCPs based on configuration. The architecture separates MCP definitions from instantiation logic to support selective enabling/disabling.

### MCP Factory Pattern

```

```

The `createBuiltinMcps` function iterates through all defined MCPs and constructs a filtered map excluding any MCPs listed in the `disabledMcps` parameter. This filtered map is then passed to the OpenCode SDK during plugin initialization.

Sources: [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

## MCP Configuration Structure

Each MCP configuration object has a standardized structure:

```

```

The `McpName` type is a Zod enum that enforces type safety for MCP identifiers:

```

```

This type-safe approach ensures that:

* Only valid MCP names can be used in configuration
* TypeScript provides autocomplete for MCP names
* Invalid MCP names are caught at compile time

Sources: [src/mcp/types.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L1-L5)

 [src/mcp/context7.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/context7.ts#L1-L5)

 [src/mcp/grep-app.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/grep-app.ts#L1-L5)

 [src/mcp/websearch-exa.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/websearch-exa.ts#L1-L5)

## MCP Registration and Access

MCPs are registered during plugin initialization and made available to specific agents based on their tool access configuration.

### Registration Flow

```

```

Agent tool access is configured during agent creation. The following agents have MCP access:

| Agent | MCP Access | Rationale |
| --- | --- | --- |
| Sisyphus | All MCPs | Full orchestrator needs all capabilities |
| Librarian | All MCPs | Research agent requires documentation and web search |
| Explore | `grep_app` only | Code search agent needs GitHub search |
| Oracle, Frontend, DocWriter, Multimodal | None | Specialized roles don't require external search |

Sources: [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

## Selective Enabling/Disabling

MCPs can be disabled through the `disabled_mcps` configuration option. This supports several use cases:

### Use Cases for Disabling MCPs

1. **Rate Limit Management**: Disable expensive MCPs to reduce API usage
2. **Privacy Requirements**: Disable external lookups for sensitive projects
3. **Development Testing**: Disable MCPs to test agent behavior without external dependencies
4. **Network Constraints**: Disable MCPs in environments with restricted internet access

### Disabling Mechanism

The `createBuiltinMcps` function implements the filtering logic:

```

```

The implementation uses a simple includes check: `!disabledMcps.includes(name as McpName)`. Any MCP name in the `disabledMcps` array is excluded from the returned map.

Sources: [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

## Configuration Example

Configuration is specified in `oh-my-opencode.json`:

```

```

This configuration would disable the `websearch_exa` MCP, preventing agents from using web search capabilities. The Librarian agent would still have access to `context7` for documentation lookups and agents with `grep_app` access would retain GitHub search.

The configuration follows the hierarchical loading pattern (see [Configuration Files](../getting-started/Configuration-Files.md)), allowing project-level overrides to disable MCPs that are enabled at the user level.

Sources: [src/mcp/types.ts L3](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L3-L3)

## Integration with Plugin System

MCPs are integrated into the plugin system during initialization. The plugin passes the MCP configuration to the OpenCode SDK, which handles:

* Establishing connections to remote MCP services
* Exposing MCP tools to agents based on their tool access configuration
* Managing authentication and rate limiting for MCP services
* Handling errors and retries for remote service calls

The MCP system operates transparently to agents—from the agent's perspective, MCP tools appear identical to local tools. The OpenCode SDK handles all communication with remote services.

Sources: [src/mcp/index.ts L1-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L1-L24)