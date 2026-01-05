---
layout: default
title: MCP 통합
parent: "MCP 통합"
nav_order: 2
---

# MCP 통합 (MCP Integration)

> **관련 소스 파일**
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

이 문서는 oh-my-opencode의 Micro-Capability Provider (MCP) 시스템에 대해 설명합니다. 이 시스템은 에이전트(Agent)가 원격 API 엔드포인트를 통해 외부 지식 소스 및 서비스에 액세스할 수 있도록 지원합니다. MCP는 에이전트 툴킷을 로컬 파일 시스템 및 코드 분석 도구 너머로 확장하여 웹 검색, 공식 문서 조회, GitHub 코드 검색 기능을 포함할 수 있게 합니다.

에이전트가 이러한 도구에 액세스하고 사용하는 방법에 대한 정보는 [도구 시스템(Tool System)](../tools/)을 참조하십시오. LSP 및 AST-grep을 포함한 특정 도구 구현에 대해서는 [LSP 도구](/code-yeongyu/oh-my-opencode/5.1-lsp-tools) 및 [AST-Grep 도구](/code-yeongyu/oh-my-opencode/5.2-ast-grep-tools)를 참조하십시오.

---

## MCP 아키텍처 (MCP Architecture)

MCP는 에이전트에게 특화된 검색 및 조사 도구를 노출하는 원격 기능 제공자입니다. 이 시스템은 에이전트 조사 워크플로우에서 각각 고유한 목적을 수행하는 세 가지 내장 MCP를 정의합니다.

```

```

**출처:** [src/mcp/index.ts L8-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L8-L24)

 [src/mcp/types.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L1-L6)

 [README.md L113-L114](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L113-L114)

MCP 시스템은 각 MCP가 원격 서비스 구성으로 정의되는 레지스트리 패턴(Registry pattern)을 따릅니다. [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)에 있는 `createBuiltinMcps()` 함수는 사용자 구성을 기반으로 레지스트리를 필터링하여 활성화된 MCP만 반환합니다.

---

## MCP 설정 구조 (MCP Configuration Structure)

각 MCP는 세 가지 속성으로 정의됩니다:

| 속성 | 타입 | 설명 |
| --- | --- | --- |
| `type` | `"remote"` | MCP가 HTTP 엔드포인트를 통해 액세스됨을 나타냄 |
| `url` | `string` | 원격 MCP 서버 엔드포인트 URL |
| `enabled` | `boolean` | MCP의 기본 활성화 여부 |

```

```

**출처:** [src/mcp/context7.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/context7.ts#L1-L6)

 [src/mcp/websearch-exa.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/websearch-exa.ts#L1-L6)

 [src/mcp/grep-app.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/grep-app.ts#L1-L6)

---

## 내장 MCP 개요 (Built-in MCPs Overview)

oh-my-opencode에는 상호 보완적인 조사 기능을 제공하는 세 가지 엄선된 MCP가 포함되어 있습니다:

| MCP 이름 | URL | 주요 목적 | 주요 특징 |
| --- | --- | --- | --- |
| `context7` | `https://mcp.context7.com/mcp` | 공식 문서 조회 | NPM, PyPI, Cargo, 패키지 레지스트리 문서 |
| `websearch_exa` | `https://mcp.exa.ai/mcp?tools=web_search_exa` | 실시간 웹 검색 | 2025년 이후 필터링, AI 기반 검색 |
| `grep_app` | `https://mcp.grep.app` | GitHub 코드 검색 | 수백만 개의 공개 저장소 검색 |

**출처:** [src/mcp/context7.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/context7.ts#L1-L6)

 [src/mcp/websearch-exa.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/websearch-exa.ts#L1-L6)

 [src/mcp/grep-app.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/grep-app.ts#L1-L6)

 [README.md L173-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L173-L176)

### context7

패키지 레지스트리의 공식 문서에 대한 액세스를 제공합니다. 에이전트는 이를 사용하여 API 레퍼런스, 라이브러리 문서 및 공식 사용 예시를 조회합니다.

### websearch_exa

최신 콘텐츠(2025년 이후)에 대한 명시적 필터링을 통해 실시간 웹 검색을 가능하게 합니다. 이는 에이전트가 오래된 정보를 수신하여 더 이상 사용되지 않는(deprecated) API를 사용하는 것을 방지합니다.

### grep_app

수백만 개의 공개 GitHub 저장소에서 초고속 코드 검색을 제공합니다. 특히 오픈 소스 프로젝트에서 구현 예시와 패턴을 찾는 데 유용합니다.

---

## MCP 등록 및 필터링 (MCP Registration and Filtering)

`createBuiltinMcps()` 함수는 설정을 기반으로 선택적인 MCP 등록을 구현합니다:

```

```

**출처:** [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

[src/mcp/index.ts L17-L20](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L17-L20)의 필터링 로직은 `disabledMcps` 배열에 없는 MCP만 반환된 구성에 포함되도록 보장합니다. 이를 통해 사용자는 기본 레지스트리를 그대로 유지하면서 특정 MCP를 선택적으로 비활성화할 수 있습니다.

---

## 설정 옵션 (Configuration Options)

사용자는 `oh-my-opencode.json` 설정 파일을 통해 MCP를 비활성화할 수 있습니다:

```

```

이 설정은 웹 검색과 GitHub 검색을 비활성화하고 `context7`만 활성화된 상태로 둡니다.

**출처:** [README.md L1009-L1016](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L1009-L1016)

[src/mcp/types.ts L3-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L3-L5)의 `McpName` 타입은 Zod 스키마 검증을 사용하여 설정에서 유효한 MCP 이름만 수락되도록 보장합니다.

---

## 에이전트 MCP 액세스 패턴 (Agent MCP Access Patterns)

각 에이전트는 역할에 따라 서로 다른 수준의 MCP 액세스 권한을 가집니다:

```

```

**출처:** [README.md L464-L476](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L464-L476)

 "Tool and Integration Ecosystem" 상위 레벨 다이어그램

### 액세스 근거

* **Sisyphus**: 오케스트레이션(orchestration)의 유연성을 극대화하기 위해 모든 MCP에 대한 무제한 액세스 권한을 가집니다.
* **Oracle**: MCP 액세스 권한이 없으며, LSP 도구와 기존 컨텍스트를 사용한 분석에 집중합니다.
* **Librarian**: 주요 MCP 사용자입니다. 외부 조사 및 문서 조회를 위해 설계되었습니다.
* **Explore**: 집중적인 코드 검색을 위해 `grep_app`으로 제한됩니다. 웹 액세스를 차단하여 범위 확장(scope creep)을 방지합니다.
* **Frontend Engineer**: MCP 액세스 권한이 없으며, 조사보다는 구현에 집중합니다.

---

## Claude Code MCP 호환성 (Claude Code MCP Compatibility)

oh-my-opencode는 하위 호환성을 위해 Claude Code 설정 파일에서 추가 MCP를 로드합니다:

```

```

**출처:** [README.md L629-L633](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L629-L633)

 [README.md L642-L663](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L642-L663)

### MCP 로드 위치

Claude Code MCP 설정은 우선순위에 따라 다음 세 위치에서 로드됩니다:

1. `~/.claude/.mcp.json` (사용자 수준)
2. `./.mcp.json` (프로젝트 루트)
3. `./.claude/.mcp.json` (로컬, git-ignored)

각 파일은 `${VAR}` 구문을 사용한 환경 변수 확장을 지원하여 안전한 자격 증명 관리를 가능하게 합니다.

### Claude Code MCP 로딩 비활성화

내장 MCP는 유지하면서 Claude Code MCP 로딩만 비활성화하려면 다음과 같이 설정합니다:

```

```

[README.md L642-L663](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L642-L663)에 있는 이 토글은 Claude Code의 `.mcp.json` 파일에만 영향을 미치며, 내장 MCP(`context7`, `websearch_exa`, `grep_app`)에는 영향을 주지 않습니다.

---

## MCP 타입 시스템 (MCP Type System)

MCP 타입 시스템은 런타임 검증을 위해 Zod를 사용합니다:

```

```

이 스키마는 설정에서 MCP를 참조할 때 타입 안전성을 보장하며, 오타나 유효하지 않은 MCP 이름으로 인해 런타임 오류가 발생하는 것을 방지합니다.

**출처:** [src/mcp/types.ts L1-L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L1-L6)

---

## 플러그인 시스템과의 통합 (Integration with Plugin System)

MCP는 에이전트 팩토리 구성의 일부로 플러그인 초기화 중에 등록됩니다:

```

```

**출처:** [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

 [README.md L108-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L108-L117)

MCP 구성은 다른 플러그인 구성 요소(에이전트, 후크, 도구)와 병합되어 완전한 에이전트 환경을 생성합니다. 각 에이전트는 역할과 기능에 따라 지정된 MCP 액세스 권한을 할당받습니다.

---

## 요약 (Summary)

MCP 시스템은 원격 기능 제공자를 oh-my-opencode 에이전트 생태계에 통합하기 위한 깔끔한 추상화를 제공합니다. 주요 특징은 다음과 같습니다:

* **세 가지 내장 MCP**: 문서 조회, 웹 검색 및 코드 검색 제공
* **역할 기반 액세스**: 에이전트는 기능에 적합한 MCP 액세스 권한을 부여받음
* **구성 가능**: 사용자는 설정을 통해 모든 MCP를 비활성화할 수 있음
* **Claude Code 호환**: `.mcp.json` 파일에서 추가 MCP 로드 가능
* **타입 안전성**: Zod 스키마 검증을 통해 설정 오류 방지
* **원격 전용**: 모든 MCP는 HTTP 엔드포인트를 통해 액세스되며 로컬 실행은 없음

개별 MCP 및 해당 기능에 대한 자세한 정보는 [내장 MCP](/code-yeongyu/oh-my-opencode/8.2-built-in-mcps)를 참조하십시오. MCP 시스템 아키텍처 및 구성 패턴에 대해서는 [MCP 시스템 개요](/code-yeongyu/oh-my-opencode/8.1-mcp-system-overview)를 참조하십시오.

**출처:** [src/mcp/index.ts L1-L25](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L1-L25)

 [README.md L113-L114](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L113-L114)

 [README.md L173-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L173-L176)

 [README.md L563-L568](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L563-L568)