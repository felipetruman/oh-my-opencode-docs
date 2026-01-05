---
layout: default
title: MCP 시스템 개요 (MCP System Overview)
parent: "MCP 통합"
nav_order: 1
---

# MCP 시스템 개요 (MCP System Overview)

> **관련 소스 파일**
> * [LICENSE.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/LICENSE.md)
> * [src/mcp/context7.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/context7.ts)
> * [src/mcp/grep-app.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/grep-app.ts)
> * [src/mcp/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts)
> * [src/mcp/types.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts)
> * [src/mcp/websearch-exa.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/websearch-exa.ts)

이 문서는 oh-my-opencode의 MCP(Micro-Capability Provider, 마이크로 기능 제공자) 아키텍처에 대해 설명합니다. 여기에는 원격 서비스가 도구(tools)로 통합되는 방식, 설정 구조, 그리고 선택적 활성화/비활성화 메커니즘이 포함됩니다.

특정 내장 MCP(`context7`, `websearch_exa`, `grep_app`)에 대한 문서는 [내장 MCP(Built-in MCPs)](/code-yeongyu/oh-my-opencode/8.2-built-in-mcps)를 참조하십시오. 에이전트가 MCP 도구에 액세스하는 방법에 대한 정보는 [도구 시스템(Tool System)](../tools/) 및 [에이전트 설정(Agent Configuration)](/code-yeongyu/oh-my-opencode/4.3-agent-configuration)을 참조하십시오.

## 목적 및 범위 (Purpose and Scope)

MCP(Micro-Capability Provider)는 외부 서비스를 에이전트가 사용할 수 있는 도구로 통합하여 oh-my-opencode의 기능을 확장합니다. 이 시스템은 다음과 같은 기능을 제공합니다:

* 커스텀 코드 없이 원격 HTTP 서비스 통합
* 모든 MCP에 대한 표준화된 설정 형식
* 프로젝트 또는 사용자별 MCP 선택적 활성화/비활성화
* 타입 안전(Type-safe)한 MCP 이름 및 설정

## MCP란 무엇인가

MCP는 핵심 도구 세트 이외에 에이전트에게 추가적인 기능을 제공하는 외부 서비스입니다. 각 MCP는 다음과 같은 특징을 가집니다:

* 원격 HTTP 엔드포인트를 통해 기능을 노출합니다.
* OpenCode SDK의 원격 도구 통합 프로토콜을 따릅니다.
* 특정 에이전트가 사용할 수 있는 도구 세트로 등록됩니다.
* 설정을 통해 독립적으로 활성화하거나 비활성화할 수 있습니다.

내장 MCP는 세 가지 카테고리의 외부 지식을 제공합니다:

| MCP 이름 | 목적 | 주요 사용자 |
| --- | --- | --- |
| `context7` | 패키지 생태계(NPM, PyPI, Cargo 등)의 공식 문서 | Librarian |
| `websearch_exa` | 2025년 이후 필터링 기능이 포함된 실시간 웹 검색 | Librarian |
| `grep_app` | 수백만 개의 저장소를 대상으로 하는 GitHub 코드 검색 | Librarian, Explore |

출처: [src/mcp/index.ts L8-L12](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L8-L12)

 [src/mcp/types.ts L3](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L3-L3)

## MCP 아키텍처 (MCP Architecture)

MCP 시스템은 설정에 따라 MCP를 인스턴스화하기 위해 팩토리 패턴(factory pattern)을 사용합니다. 이 아키텍처는 선택적 활성화/비활성화를 지원하기 위해 MCP 정의와 인스턴스화 로직을 분리합니다.

### MCP 팩토리 패턴 (MCP Factory Pattern)

```typescript
// src/mcp/index.ts 예시 (실제 코드는 소스 파일 참조)
export function createBuiltinMcps(disabledMcps: McpName[] = []): Record<string, McpConfig> {
  const allMcps: Record<McpName, McpConfig> = {
    context7,
    websearch_exa,
    grep_app,
  };

  return Object.entries(allMcps).reduce((acc, [name, config]) => {
    if (!disabledMcps.includes(name as McpName)) {
      acc[name] = config;
    }
    return acc;
  }, {} as Record<string, McpConfig>);
}
```

`createBuiltinMcps` 함수는 정의된 모든 MCP를 순회하며 `disabledMcps` 파라미터에 나열된 MCP를 제외한 필터링된 맵(map)을 구성합니다. 이 필터링된 맵은 플러그인 초기화 중에 OpenCode SDK로 전달됩니다.

출처: [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

## MCP 설정 구조 (MCP Configuration Structure)

각 MCP 설정 객체는 표준화된 구조를 가집니다:

```typescript
// src/mcp/types.ts 및 개별 MCP 파일 참조
export interface McpConfig {
  name: McpName;
  baseUrl: string;
  description: string;
}
```

`McpName` 타입은 MCP 식별자에 대한 타입 안전성을 강제하는 Zod 열거형(enum)입니다:

```typescript
export const McpNameSchema = z.enum(["context7", "websearch_exa", "grep_app"]);
export type McpName = z.infer<typeof McpNameSchema>;
```

이러한 타입 안전 접근 방식은 다음을 보장합니다:

* 설정에는 유효한 MCP 이름만 사용할 수 있습니다.
* TypeScript가 MCP 이름에 대한 자동 완성을 제공합니다.
* 유효하지 않은 MCP 이름은 컴파일 타임에 감지됩니다.

출처: [src/mcp/types.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L1-L5)

 [src/mcp/context7.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/context7.ts#L1-L5)

 [src/mcp/grep-app.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/grep-app.ts#L1-L5)

 [src/mcp/websearch-exa.ts L1-L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/websearch-exa.ts#L1-L5)

## MCP 등록 및 액세스 (MCP Registration and Access)

MCP는 플러그인 초기화 중에 등록되며, 도구 액세스 설정에 따라 특정 에이전트가 사용할 수 있게 됩니다.

### 등록 흐름 (Registration Flow)

```typescript
// 초기화 과정의 개념적 흐름
const disabledMcps = config.disabled_mcps || [];
const mcps = createBuiltinMcps(disabledMcps);
await sdk.registerMcps(mcps);
```

에이전트 도구 액세스는 에이전트 생성 시 설정됩니다. 다음 에이전트들이 MCP 액세스 권한을 가집니다:

| 에이전트 | MCP 액세스 | 근거 |
| --- | --- | --- |
| Sisyphus | 모든 MCP | 전체 오케스트레이터(orchestrator)는 모든 기능이 필요함 |
| Librarian | 모든 MCP | 리서치 에이전트는 문서 및 웹 검색이 필요함 |
| Explore | `grep_app`만 허용 | 코드 검색 에이전트는 GitHub 검색이 필요함 |
| Oracle, Frontend, DocWriter, Multimodal | 없음 | 특화된 역할은 외부 검색이 필요하지 않음 |

출처: [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

## 선택적 활성화/비활성화 (Selective Enabling/Disabling)

MCP는 `disabled_mcps` 설정 옵션을 통해 비활성화할 수 있습니다. 이는 다음과 같은 여러 유스케이스를 지원합니다:

### MCP 비활성화 유스케이스

1. **속도 제한 관리(Rate Limit Management)**: 비용이 많이 드는 MCP를 비활성화하여 API 사용량 절감
2. **개인정보 보호 요구사항(Privacy Requirements)**: 민감한 프로젝트에 대해 외부 조회를 비활성화
3. **개발 테스트(Development Testing)**: 외부 의존성 없이 에이전트 동작을 테스트하기 위해 MCP 비활성화
4. **네트워크 제약(Network Constraints)**: 인터넷 액세스가 제한된 환경에서 MCP 비활성화

### 비활성화 메커니즘 (Disabling Mechanism)

`createBuiltinMcps` 함수는 필터링 로직을 구현합니다:

```typescript
// src/mcp/index.ts
if (!disabledMcps.includes(name as McpName)) {
  acc[name] = config;
}
```

구현에는 간단한 포함 여부 확인(`!disabledMcps.includes(name as McpName)`)을 사용합니다. `disabledMcps` 배열에 있는 모든 MCP 이름은 반환되는 맵에서 제외됩니다.

출처: [src/mcp/index.ts L14-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L14-L24)

## 설정 예시 (Configuration Example)

설정은 `oh-my-opencode.json`에서 지정합니다:

```json
{
  "disabled_mcps": ["websearch_exa"]
}
```

이 설정은 `websearch_exa` MCP를 비활성화하여 에이전트가 웹 검색 기능을 사용하지 못하게 합니다. Librarian 에이전트는 여전히 문서 조회를 위해 `context7`에 액세스할 수 있으며, `grep_app` 액세스 권한이 있는 에이전트는 GitHub 검색 기능을 유지합니다.

설정은 계층적 로딩 패턴(참조: [설정 파일(Configuration Files)](../getting-started/Configuration-Files.md))을 따르므로, 사용자 수준에서 활성화된 MCP를 프로젝트 수준에서 덮어써서 비활성화할 수 있습니다.

출처: [src/mcp/types.ts L3](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/types.ts#L3-L3)

## 플러그인 시스템과의 통합 (Integration with Plugin System)

MCP는 초기화 중에 플러그인 시스템에 통합됩니다. 플러그인은 MCP 설정을 OpenCode SDK에 전달하며, SDK는 다음을 처리합니다:

* 원격 MCP 서비스와의 연결 설정
* 도구 액세스 설정에 따라 에이전트에게 MCP 도구 노출
* MCP 서비스에 대한 인증 및 속도 제한 관리
* 원격 서비스 호출에 대한 오류 처리 및 재시도

MCP 시스템은 에이전트에게 투명하게 작동합니다. 에이전트의 관점에서는 MCP 도구가 로컬 도구와 동일하게 보입니다. OpenCode SDK가 원격 서비스와의 모든 통신을 처리합니다.

출처: [src/mcp/index.ts L1-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/mcp/index.ts#L1-L24)