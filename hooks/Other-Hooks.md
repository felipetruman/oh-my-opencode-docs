---
layout: default
title: Other Hooks
parent: Hooks
nav_order: 1
---

# Other Hooks

> **Relevant source files**
> * [assets/oh-my-opencode.schema.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json)
> * [src/config/schema.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts)
> * [src/hooks/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts)
> * [src/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts)

## Purpose and Scope

This page documents hooks not covered in other reliability system sections. These hooks enhance user experience, quality assurance, and system integration:

**User-Facing Hooks**: Auto-update checker, startup toast, keyword detector, agent usage reminder
**Quality Assurance Hooks**: Comment checker, empty task response detector
**Integration Hooks**: Session notification, background notification, think mode, Claude Code hooks

For error recovery hooks, see page 7.1 (Session Recovery). For message validation, see page 7.2. For task continuation, see page 7.3. For context management, see page 7.4. For context injection, see page 7.5. For non-interactive environment, see page 7.6.

---

## Auto-Update Checker Hook

The `auto-update-checker` hook monitors npm for new plugin versions and invalidates the local package cache to trigger automatic updates on OpenCode restart.

### Update Detection Flow

```

```

**Diagram: Auto-Update Checker Decision Tree**

Sources: [src/hooks/auto-update-checker/index.ts L8-L69](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/index.ts#L8-L69)

 [src/hooks/auto-update-checker/checker.ts L173-L205](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/checker.ts#L173-L205)

### Version Detection Strategies

The hook implements four version detection strategies with fallback logic:

| Strategy | Location | Priority | Use Case |
| --- | --- | --- | --- |
| **Local Dev** | `file://` in `opencode.json` | Highest | Development with `bun link` |
| **Pinned Version** | `oh-my-opencode@1.2.3` in config | High | Lock to specific version |
| **Cached Package** | `~/.cache/opencode/node_modules/oh-my-opencode/package.json` | Medium | Normal installation |
| **Current Module** | Walk up from `import.meta.url` | Lowest | Fallback resolution |

Sources: [src/hooks/auto-update-checker/checker.ts L15-L150](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/checker.ts#L15-L150)

### Configuration Search Paths

The hook searches for OpenCode configuration in priority order:

```

```

**Diagram: Configuration Detection Flow**

Sources: [src/hooks/auto-update-checker/checker.ts L19-L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/checker.ts#L19-L126)

 [src/hooks/auto-update-checker/constants.ts L29-L42](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/constants.ts#L29-L42)

### Cache Invalidation Mechanism

When an update is detected, the hook invalidates the cached plugin to force OpenCode to re-download:

```

```

The `invalidatePackage()` function performs two operations:

1. **Remove Plugin Directory**: Deletes `~/.cache/opencode/node_modules/oh-my-opencode/`
2. **Remove Dependency Entry**: Removes `oh-my-opencode` from `~/.cache/opencode/package.json` dependencies

Sources: [src/hooks/auto-update-checker/cache.ts L6-L41](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/cache.ts#L6-L41)

### Toast Notification Variants

The hook displays different toast messages based on detection results:

| Condition | Title | Message | Duration | Variant |
| --- | --- | --- | --- | --- |
| **Update Available** | `OhMyOpenCode {latestVersion}` | `OpenCode is now on Steroids. oMoMoMoMo...\nv{latestVersion} available. Restart OpenCode to apply.` | 8000ms | info |
| **Up to Date** | `OhMyOpenCode {currentVersion}` | `OpenCode is now on Steroids. oMoMoMoMo...` | 5000ms | info |
| **Local Dev** | `OhMyOpenCode {devVersion}` | `OpenCode is now on Steroids. oMoMoMoMo...` | 5000ms | info |
| **Pinned** | `OhMyOpenCode {pinnedVersion}` | `OpenCode is now on Steroids. oMoMoMoMo...` | 5000ms | info |

Sources: [src/hooks/auto-update-checker/index.ts L52-L83](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/index.ts#L52-L83)

### Hook Initialization and Configuration

```

```

The hook accepts an `AutoUpdateCheckerOptions` object with `showStartupToast` flag. When `startup-toast` is disabled in configuration, version toasts are suppressed even if `auto-update-checker` is enabled.

Sources: [src/index.ts L232-L236](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L232-L236)

 [src/hooks/auto-update-checker/types.ts L25-L27](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/types.ts#L25-L27)

---

## Startup Toast Hook

The `startup-toast` hook is not a separate hook but a configuration flag for `auto-update-checker`. When enabled, it displays a welcome toast on the first main session creation showing the current plugin version.

### Toast Control Logic

```

```

**Diagram: Startup Toast Control Flow**

The startup toast appears for all detection results (update available, up to date, local dev, pinned) unless explicitly disabled.

Sources: [src/index.ts L232-L236](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L232-L236)

 [src/hooks/auto-update-checker/index.ts L9-L83](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/index.ts#L9-L83)

---

## Keyword Detector Hook

The `keyword-detector` hook analyzes user messages for special keywords and injects optimization suggestions into the conversation. It operates on `chat.message` and `event` lifecycle hooks.

### Detected Keywords and Actions

```

```

**Diagram: Keyword Detection and Response Mapping**

Sources: [src/index.ts L237-L395](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L237-L395)

### Hook Implementation Structure

The keyword detector implements two lifecycle methods:

| Method | Trigger | Purpose |
| --- | --- | --- |
| `chat.message` | User submits message | Pre-process keywords before agent sees them |
| `event` | Session events | React to session state changes |

The hook likely injects system prompts or modifies agent instructions based on detected keywords, though the exact implementation is in an unloaded file.

Sources: [src/index.ts L237-L282](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L237-L282)

---

## Agent Usage Reminder Hook

The `agent-usage-reminder` hook suggests specialized agents when user queries match specific patterns. It operates on `event` and `tool.execute.after` lifecycle hooks.

### Hook Lifecycle Integration

```

```

**Diagram: Agent Usage Reminder Trigger Points**

Sources: [src/index.ts L240-L530](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L240-L530)

### Context-Aware Suggestion Patterns

The hook monitors conversation context and tool usage patterns to suggest specialized agents at opportune moments:

| Pattern Detected | Suggested Agent | Rationale |
| --- | --- | --- |
| Multiple `grep` calls with low results | `explore` | Parallel codebase exploration more efficient |
| Questions about external libraries | `librarian` | GitHub and documentation research needed |
| Architecture or design questions | `oracle` | Expert technical guidance required |
| UI/UX implementation requests | `frontend` | Specialized frontend engineer required |

The reminder hook prevents redundant suggestions by tracking which agents have already been mentioned in the conversation.

Sources: [src/index.ts L240-L242](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L240-L242)

---

## Hook Configuration and Disabling

All user experience hooks can be individually disabled via the `disabled_hooks` configuration array:

```

```

### Hook Enablement Logic

```

```

**Diagram: Hook Conditional Initialization Flow**

The `isHookEnabled()` helper function checks if a hook name exists in the disabled set: `!disabledHooks.has(hookName)`. When disabled, hooks return `null` and their lifecycle methods are never called.

Sources: [src/index.ts L182-L248](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L182-L248)

 [src/config/schema.ts L44-L65](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L44-L65)

### Hook Name Schema Validation

Hook names are validated against the `HookNameSchema` enum in configuration:

```

```

Invalid hook names in `disabled_hooks` will cause configuration validation errors at plugin load time.

Sources: [src/config/schema.ts L44-L65](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L44-L65)

---

## Integration with Event System

User experience hooks integrate with the plugin event system through multiple lifecycle points:

```

```

**Diagram: User Experience Hooks Event Integration**

Each hook receives events through the central dispatcher in [src/index.ts L383-L491](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L383-L491)

 allowing them to observe and influence the conversation flow.

Sources: [src/index.ts L279-L491](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L279-L491)