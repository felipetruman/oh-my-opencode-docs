---
layout: default
title: 설정 스키마 레퍼런스
parent: Reference
nav_order: 1
---

# 설정 스키마 레퍼런스 (Configuration Schema Reference)

> **관련 소스 파일**
> * [README.ja.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ja.md)
> * [README.ko.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md)
> * [README.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md)
> * [README.zh-cn.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.zh-cn.md)
> * [assets/oh-my-opencode.schema.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json)
> * [src/config/schema.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts)
> * [src/hooks/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts)
> * [src/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts)
> * [src/shared/config-path.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts)

이 페이지는 `oh-my-opencode.json`에서 사용할 수 있는 모든 설정 옵션에 대한 전체 레퍼런스를 제공합니다. 설정 시스템은 런타임 검증(runtime validation)을 위해 Zod 스키마를 사용하며, IDE의 자동 완성(autocomplete) 지원을 위해 JSON Schema를 생성합니다.

설정이 로드되고 병합되는 방식에 대한 정보는 [설정 시스템(Configuration System)](/code-yeongyu/oh-my-opencode/3.2-configuration-system)을 참조하십시오. 에이전트별 설정 및 동작에 대해서는 [에이전트 레퍼런스(Agent Reference)](/code-yeongyu/oh-my-opencode/12.2-cicd-pipeline)를 참조하십시오.

## 설정 파일 위치 (Configuration File Locations)

oh-my-opencode는 정의된 우선순위에 따라 여러 위치에서 설정을 로드합니다. 프로젝트 수준의 설정이 사용자 수준의 설정보다 우선합니다. JSON 및 JSONC(주석이 포함된 JSON) 형식을 모두 지원합니다.

```

```

**출처:** [src/index.ts L125-L217](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L125-L217)

 [src/shared/config-path.ts L1-L48](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts#L1-L48)

 [src/shared/jsonc-parser.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.ts)

### JSONC 지원

설정 파일은 JSONC(JSON with Comments) 형식을 지원합니다:

* 한 줄 주석: `// comment`
* 블록 주석: `/* comment */`
* 후행 쉼표(Trailing commas): `{ "key": "value", }`

동일한 위치에 `.jsonc`와 `.json` 파일이 모두 존재하는 경우, `.jsonc` 파일이 우선권을 갖습니다.

```

```

**출처:** [README.md L715-L743](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L715-L743)

 [src/index.ts L129](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L129-L129)

 [src/shared/jsonc-parser.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.ts)

### 플랫폼별 경로

| 플랫폼 | 사용자 설정 경로 | 우선순위 |
| --- | --- | --- |
| **Linux/macOS** | `~/.config/opencode/oh-my-opencode.{jsonc,json}` | 표준 |
| **Windows** | `~/.config/opencode/oh-my-opencode.{jsonc,json}` | 권장 |
| **Windows** | `%APPDATA%\opencode\oh-my-opencode.{jsonc,json}` | 폴백(Fallback) |

Windows 구현에서는 기존 설치와의 하위 호환성을 위해 `%APPDATA%`로 폴백하기 전에 크로스 플랫폼 경로(`~/.config`)를 먼저 확인합니다. 각 위치 내에서는 `.jsonc` 파일을 `.json` 파일보다 먼저 확인합니다.

**출처:** [src/shared/config-path.ts L13-L33](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts#L13-L33)

 [src/index.ts L191-L198](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L191-L198)

### 하위 호환성 마이그레이션

설정 시스템은 로드 중에 레거시 에이전트 이름을 자동으로 마이그레이션합니다:

* `omo` / `OmO` → `Sisyphus`
* `OmO-Plan` / `omo-plan` → `Planner-Sisyphus`
* `omo_agent` → `sisyphus_agent`

마이그레이션된 설정은 파일을 업데이트하기 위해 디스크에 다시 기록됩니다.

```

```

**출처:** [src/index.ts L62-L123](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L62-L123)

## 스키마 구조 개요 (Schema Structure Overview)

설정 스키마는 Zod를 사용하여 정의되며 TypeScript 타입과 JSON Schema로 모두 내보내집니다. JSON Schema는 IDE 자동 완성을 위해 `https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json`에서 제공됩니다.

```

```

**출처:** [script/build-schema.ts L1-L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/build-schema.ts#L1-L29)

 [src/config/index.ts L1-L22](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/index.ts#L1-L22)

 [README.md L717-L722](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L717-L722)

## 최상위 속성 (Top-Level Properties)

### disabled_mcps

**타입:** `string[]`
**유효한 값:** `"websearch_exa"`, `"context7"`, `"grep_app"`
**기본값:** `[]` (모두 활성화)

특정 내장 MCP(Model Context Protocol) 서비스를 비활성화합니다. 내장 MCP는 기본적으로 로드되며 에이전트에게 외부 기능을 제공합니다.

```

```

| MCP | 설명 | 사용 에이전트 |
| --- | --- | --- |
| `context7` | 공식 라이브러리 문서 조회 | librarian |
| `websearch_exa` | Exa AI를 통한 실시간 웹 검색 | librarian |
| `grep_app` | 공개 리포지토리 대상 GitHub 코드 검색 | librarian, explore |

**출처:** [assets/oh-my-opencode.schema.json L11-L21](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L11-L21)

 [README.md L795-L808](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L795-L808)

### disabled_agents

**타입:** `string[]`
**유효한 값:** `"Sisyphus"`, `"oracle"`, `"librarian"`, `"explore"`, `"frontend-ui-ux-engineer"`, `"document-writer"`, `"multimodal-looker"`
**기본값:** `[]` (모두 활성화)

특정 에이전트를 완전히 비활성화하여 등록되거나 호출되지 않도록 합니다. 비활성화된 에이전트는 `call_omo_agent` 또는 작업 위임을 통해 호출할 수 없습니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L22-L36](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L22-L36)

 [README.md L797-L805](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L797-L805)

### disabled_hooks

**타입:** `string[]`
**유효한 값:** 아래 표 참조
**기본값:** `[]` (모두 활성화)

특정 생명주기 훅(lifecycle hooks)을 비활성화합니다. 훅은 이벤트를 가로채고 플러그인 생명주기의 다양한 단계에서 동작을 수정합니다.

```

```

**사용 가능한 훅:**

| 훅 이름 | 이벤트 | 목적 |
| --- | --- | --- |
| `todo-continuation-enforcer` | session.idle | TODO 항목의 완료를 강제함 |
| `context-window-monitor` | message.updated | 토큰 사용량을 모니터링함 |
| `session-recovery` | session.error | 세션 오류로부터 복구함 |
| `session-notification` | session.idle | 데스크톱 알림 제공 |
| `comment-checker` | tool.execute.after | 코드 주석의 유효성을 검사함 |
| `grep-output-truncator` | tool.execute.after | grep 출력을 잘라냄 |
| `tool-output-truncator` | tool.execute.after | 도구 출력을 잘라냄 |
| `directory-agents-injector` | tool.execute.before | AGENTS.md 컨텍스트를 주입함 |
| `directory-readme-injector` | tool.execute.before | README.md 컨텍스트를 주입함 |
| `empty-task-response-detector` | tool.execute.after | 빈 작업 결과를 감지함 |
| `think-mode` | chat.message | 확장 생각(extended thinking) 모드 활성화 |
| `anthropic-auto-compact` | session.error | 토큰 제한 시 자동 압축(compact) 수행 |
| `rules-injector` | tool.execute.before | 조건부 규칙을 주입함 |
| `background-notification` | session.idle | 백그라운드 작업 알림 제공 |
| `auto-update-checker` | session.created | 플러그인 업데이트 확인 |
| `startup-toast` | session.created | 시작 메시지 표시 |
| `keyword-detector` | chat.message | 특수 모드 활성화 |
| `agent-usage-reminder` | tool.execute.after | 에이전트 위임에 대한 알림 제공 |
| `non-interactive-env` | tool.execute.before | 비대화형 환경 처리 |
| `interactive-bash-session` | session.* | tmux 세션 관리 |
| `empty-message-sanitizer` | chat.message | 빈 메시지 정화(sanitize) |

**출처:** [assets/oh-my-opencode.schema.json L37-L65](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L37-L65)

 [README.md L784-L792](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L784-L792)

### google_auth

**타입:** `boolean`
**기본값:** `false`

Gemini 모델을 위한 내장 Google Antigravity OAuth 인증을 활성화합니다. `true`로 설정하면 플러그인이 단일 계정 인증을 사용하는 Google 프로바이더를 등록합니다.

```

```

**참고:** 내장 인증보다는 다중 계정 로드 밸런싱과 더 많은 모델을 제공하는 외부 `opencode-antigravity-auth` 플러그인을 사용하는 것이 권장됩니다. 외부 플러그인을 사용하는 경우 이 값을 `false`로 설정하십시오.

```

```

**출처:** [README.md L723-L747](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L723-L747)

 [assets/oh-my-opencode.schema.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json)

## 에이전트 설정 (Agent Configuration)

`agents` 객체는 내장 에이전트의 커스터마이징을 허용합니다. 각 에이전트 키는 에이전트 이름에 해당하며, 값은 `AgentOverrideConfig` 객체입니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L66-L1255](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L66-L1255)

 [README.md L749-L805](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L749-L805)

### 설정 가능한 에이전트

| 에이전트 이름 | 기본 모델 | 목적 |
| --- | --- | --- |
| `build` | (OpenCode 기본값) | 기본 빌드 에이전트 (서브에이전트로 강등됨) |
| `plan` | (OpenCode 기본값) | 계획 에이전트 (서브에이전트로 강등됨) |
| `Sisyphus` | `anthropic/claude-opus-4-5` | 기본 오케스트레이터(조정자) |
| `Planner-Sisyphus` | (plan에서 상속) | Sisyphus를 위한 계획 모드 |
| `oracle` | `openai/gpt-5.2` | 아키텍처 및 디버깅 |
| `librarian` | `anthropic/claude-sonnet-4-5` | 조사 및 문서화 |
| `explore` | `opencode/grok-code` | 코드베이스 탐색 |
| `frontend-ui-ux-engineer` | `google/gemini-3-pro-preview` | 프론트엔드 개발 |
| `document-writer` | `google/gemini-3-pro-preview` | 기술 문서 작성 |
| `multimodal-looker` | `google/gemini-3-flash` | 시각적 콘텐츠 분석 |

**출처:** [README.md L489-L496](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L489-L496)

 [README.md L749-L765](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L749-L765)

### 에이전트 속성

#### model

**타입:** `string`
**필수 여부:** 아니요

에이전트에 사용할 LLM 모델을 지정합니다. 모델 이름은 provider/model 형식을 따릅니다 (예: `anthropic/claude-opus-4-5`, `openai/gpt-5.2`).

```

```

**출처:** [assets/oh-my-opencode.schema.json L72-L74](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L72-L74)

 [README.md L751-L759](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L751-L759)

#### temperature

**타입:** `number`
**범위:** `0` ~ `2`
**필수 여부:** 아니요

모델 응답의 무작위성을 제어합니다. 낮은 값(0.0-0.3)은 더 결정론적인 출력을 생성하며, 높은 값(0.7-2.0)은 창의성과 변동성을 높입니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L75-L79](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L75-L79)

#### top_p

**타입:** `number`
**범위:** `0` ~ `1`
**필수 여부:** 아니요

핵심 샘플링(Nucleus sampling) 파라미터입니다. 고려되는 토큰의 누적 확률을 제한하여 다양성을 제어합니다. 낮은 값은 출력을 더 집중되게 만듭니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L80-L84](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L80-L84)

#### prompt

**타입:** `string`
**필수 여부:** 아니요

에이전트의 시스템 프롬프트를 덮어씁니다. 이는 내장 프롬프트를 사용자 정의 지침으로 완전히 대체합니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L85-L87](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L85-L87)

 [src/config/schema.ts L78](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L78-L78)

#### prompt_append

**타입:** `string`
**필수 여부:** 아니요

에이전트의 내장 시스템 프롬프트를 대체하는 대신 추가 지침을 덧붙입니다. 기본 동작을 유지하면서 프로젝트별 가이드라인을 추가할 때 유용합니다.

```

```

**출처:** [src/config/schema.ts L79](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L79-L79)

#### tools

**타입:** `Record<string, boolean>`
**필수 여부:** 아니요

에이전트의 특정 도구를 활성화하거나 비활성화합니다. 키는 도구 이름이고, 값은 사용 가능 여부를 나타내는 불리언(boolean)입니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L88-L96](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L88-L96)

#### disable

**타입:** `boolean`
**기본값:** `false`
**필수 여부:** 아니요

`true`인 경우 에이전트를 완전히 비활성화합니다. 이는 `disabled_agents`에 에이전트 이름을 추가하는 것과 동일하지만 개별 에이전트 설정으로 범위가 제한됩니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L97-L99](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L97-L99)

 [README.md L761-L763](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L761-L763)

#### description

**타입:** `string`
**필수 여부:** 아니요

에이전트 목록에 표시되고 에이전트 선택 컨텍스트에서 사용되는 사용자 정의 설명입니다. 내장 설명을 덮어씁니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L100-L102](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L100-L102)

#### mode

**타입:** `"subagent" | "primary" | "all"`
**필수 여부:** 아니요

에이전트 선택 시 에이전트가 나타나는 시점을 제어합니다:

* `"subagent"`: 위임(`call_omo_agent`)을 통해서만 사용 가능
* `"primary"`: 메인 에이전트 선택 시에만 사용 가능
* `"all"`: 두 컨텍스트 모두에서 사용 가능

```

```

**출처:** [assets/oh-my-opencode.schema.json L103-L110](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L103-L110)

#### color

**타입:** `string` (hex color)
**패턴:** `^#[0-9A-Fa-f]{6}$`
**필수 여부:** 아니요

UI에서 에이전트에 사용할 사용자 정의 색상입니다. 6자리 16진수 색상 코드여야 합니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L111-L114](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L111-L114)

#### permission

**타입:** `PermissionConfig` 객체
**필수 여부:** 아니요

에이전트에 대한 세밀한 권한 제어입니다. 아래의 [권한 시스템(Permission System)](#권한-시스템)을 참조하십시오.

**출처:** [assets/oh-my-opencode.schema.json L115-L177](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L115-L177)

## 권한 시스템 (Permission System)

권한 시스템은 에이전트가 수행할 수 있는 작업에 대해 세밀한 제어를 제공합니다. 각 권한은 `"ask"` (사용자에게 확인), `"allow"` (자동 승인), 또는 `"deny"` (차단)로 설정할 수 있습니다.

```

```

**출처:** [README.md L773-L795](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L773-L795)

 [assets/oh-my-opencode.schema.json L115-L177](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L115-L177)

### 권한 속성

#### edit

**타입:** `"ask" | "allow" | "deny"`
**기본값:** `"ask"`

파일 편집 권한을 제어합니다. `write_file`, `edit_file` 및 LSP 리팩토링 작업과 같은 도구에 적용됩니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L118-L125](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L118-L125)

#### bash

**타입:** `"ask" | "allow" | "deny"` 또는 `Record<string, "ask" | "allow" | "deny">`
**기본값:** `"ask"`

bash 명령 실행을 제어합니다. 전역 권한 또는 명령별 세부 제어가 가능합니다.

**전역 권한:**

```

```

**명령별 권한:**

```

```

**출처:** [assets/oh-my-opencode.schema.json L126-L151](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L126-L151)

 [README.md L788-L790](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L788-L790)

#### webfetch

**타입:** `"ask" | "allow" | "deny"`
**기본값:** `"ask"`

웹 요청 권한을 제어합니다. MCP 또는 내장 도구를 통해 수행되는 HTTP 요청에 적용됩니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L152-L159](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L152-L159)

#### doom_loop

**타입:** `"ask" | "allow" | "deny"`
**기본값:** `"ask"`

에이전트가 무한 루프 감지를 무시할 수 있는지 여부를 제어합니다. 거부(`deny`)로 설정된 경우, 에이전트가 끝없는 사이클에 빠진 것으로 보이면 차단됩니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L160-L167](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L160-L167)

#### external_directory

**타입:** `"ask" | "allow" | "deny"`
**기본값:** `"ask"`

프로젝트 루트 디렉토리 외부의 파일에 대한 액세스를 제어합니다. 에이전트를 현재 작업 공간으로 제한하는 데 유용합니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L168-L175](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L168-L175)

## Sisyphus 에이전트 설정 (Sisyphus Agent Configuration)

Sisyphus 에이전트 시스템은 특화된 Sisyphus 오케스트레이터가 OpenCode의 기본 `build` 및 `plan` 에이전트를 대체할지 여부를 제어합니다. 설정은 `sisyphus_agent` 객체(또는 레거시 `omo_agent`)를 통해 이루어집니다.

### sisyphus_agent

**타입:** `object`
**속성:**

* `disabled`: `boolean` (기본값: `false`)
* `default_builder_enabled`: `boolean` (기본값: `false`)
* `planner_enabled`: `boolean` (기본값: `true`)
* `replace_plan`: `boolean` (기본값: `true`)

#### disabled

`false`(기본값)인 경우 Sisyphus 오케스트레이션이 활성화됩니다:

* **Sisyphus**가 기본 에이전트가 됩니다 (primary 모드)
* **OpenCode-Builder**(활성화된 경우)가 서브에이전트 모드로 실행됩니다
* **Planner-Sisyphus**(활성화된 경우)가 서브에이전트 모드로 실행됩니다
* 원래의 `build` 및 `plan`은 서브에이전트 모드로 강등됩니다

`true`인 경우 표준 OpenCode 에이전트가 기본으로 유지됩니다.

```

```

#### default_builder_enabled

`true`인 경우 `OpenCode-Builder` 에이전트(이름이 변경된 OpenCode의 기본 빌드 에이전트)를 등록합니다. 에이전트 목록이 복잡해지는 것을 방지하기 위해 기본값은 `false`입니다.

```

```

#### planner_enabled

`true`(기본값)인 경우 OpenCode의 plan 에이전트를 기반으로 OhMyOpenCode의 계획 프롬프트를 사용하는 `Planner-Sisyphus` 에이전트를 등록합니다.

```

```

#### replace_plan

`true`(기본값)인 경우 Sisyphus가 활성화될 때 원래의 `plan` 에이전트가 서브에이전트 모드로 강등됩니다. `false`인 경우 `plan`은 기본 에이전트로 계속 사용할 수 있습니다.

```

```

**출처:** [src/config/schema.ts L115-L120](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L115-L120)

 [src/index.ts L415-L477](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L415-L477)

 [README.md L807-L844](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L807-L844)

### 에이전트 등록 흐름

```

```

**출처:** [src/index.ts L415-L486](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L415-L486)

### Sisyphus 에이전트 커스터마이징

Sisyphus 관련 에이전트는 `agents` 객체에서 커스터마이징할 수 있습니다:

```

```

**출처:** [README.md L826-L844](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L826-L844)

 [src/index.ts L420-L477](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L420-L477)

## Claude Code 호환성 (Claude Code Compatibility)

`claude_code` 객체는 특정 Claude Code 호환성 기능을 비활성화하는 토글을 제공합니다. 모든 토글의 기본값은 `true`(활성화)입니다.

```

```

**출처:** [README.md L654-L676](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L654-L676)

 [assets/oh-my-opencode.schema.json L1297-L1334](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1297-L1334)

### Claude Code 토글

| 토글 | `false`일 때 로드를 중단하는 위치... | 영향받지 않음 |
| --- | --- | --- |
| `mcp` | `~/.claude/.mcp.json``./.mcp.json``./.claude/.mcp.json` | 내장 MCP (context7, websearch_exa, grep_app) |
| `commands` | `~/.claude/commands/*.md``./.claude/commands/*.md` | `~/.config/opencode/command/``./.opencode/command/` |
| `skills` | `~/.claude/skills/*/SKILL.md``./.claude/skills/*/SKILL.md` | - |
| `agents` | `~/.claude/agents/*.md``./.claude/agents/*.md` | 내장 에이전트 (oracle, librarian 등) |
| `hooks` | `~/.claude/settings.json``./.claude/settings.json``./.claude/settings.local.json` | - |

**출처:** [README.md L668-L675](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L668-L675)

### 토글 속성

#### mcp

**타입:** `boolean`
**기본값:** `true`

Claude Code 스타일의 `.mcp.json` 파일로부터 MCP 설정 로드를 제어합니다. 내장 MCP(context7, websearch_exa, grep_app)에는 영향을 주지 않습니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1300-L1303](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1300-L1303)

#### commands

**타입:** `boolean`
**기본값:** `true`

Claude Code 디렉토리(`~/.claude/commands/`, `./.claude/commands/`)로부터 슬래시 명령 로드를 제어합니다. OpenCode 명령 디렉토리에는 영향을 주지 않습니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1304-L1307](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1304-L1307)

#### skills

**타입:** `boolean`
**기본값:** `true`

Claude Code 디렉토리(`~/.claude/skills/`, `./.claude/skills/`)로부터 스킬 로드를 제어합니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1308-L1311](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1308-L1311)

#### agents

**타입:** `boolean`
**기본값:** `true`

Claude Code 디렉토리(`~/.claude/agents/*.md`)로부터 사용자 정의 에이전트 정의 로드를 제어합니다. 내장 에이전트에는 영향을 주지 않습니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1312-L1315](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1312-L1315)

#### hooks

**타입:** `boolean`
**기본값:** `true`

Claude Code 설정 파일(`~/.claude/settings.json`, `./.claude/settings.json`, `./.claude/settings.local.json`)로부터 훅 로드를 제어합니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1316-L1319](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1316-L1319)

## LSP 설정 (LSP Configuration)

`lsp` 객체는 OpenCode의 메인 설정에 구성된 것 외에 추가적인 LSP(Language Server Protocol) 서버 구성을 허용합니다. 각 키는 LSP 서버 이름이고, 값은 LSP 서버 설정 객체입니다.

```

```

**출처:** [README.md L810-L833](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L810-L833)

 [assets/oh-my-opencode.schema.json L1335-L1418](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1335-L1418)

### LSP 서버 속성

#### command

**타입:** `string[]`
**필수 여부:** 예 (`disabled`가 `true`가 아닌 경우)

LSP 서버를 시작하기 위한 명령 배열입니다. 첫 번째 요소는 실행 파일이고, 그 뒤에 인자들이 옵니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1341-L1346](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1341-L1346)

#### extensions

**타입:** `string[]`
**필수 여부:** 예 (`disabled`가 `true`가 아닌 경우)

이 LSP 서버가 처리하는 파일 확장자입니다. 확장자에는 앞에 점(`.`)이 포함되어야 합니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1347-L1352](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1347-L1352)

#### priority

**타입:** `number`
**필수 여부:** 아니요
**기본값:** `0`

여러 서버가 동일한 확장자와 일치할 때 서버 선택을 위한 우선순위입니다. 높은 값이 우선권을 갖습니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1353-L1355](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1353-L1355)

#### env

**타입:** `Record<string, string>`
**필수 여부:** 아니요

LSP 서버를 시작할 때 설정할 환경 변수입니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1356-L1362](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1356-L1362)

#### initialization

**타입:** `object`
**필수 여부:** 아니요

초기화 요청 중에 서버에 전달되는 LSP 초기화 옵션입니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1363-L1365](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1363-L1365)

#### disabled

**타입:** `boolean`
**기본값:** `false`
**필수 여부:** 아니요

`true`인 경우 이 LSP 서버를 비활성화합니다. 설정을 삭제하지 않고 일시적으로 서버를 비활성화할 때 유용합니다.

```

```

**출처:** [assets/oh-my-opencode.schema.json L1366-L1368](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1366-L1368)

## 주석 검사기 설정 (Comment Checker Configuration)

`comment_checker` 객체는 에이전트에게 과도한 코드 주석에 대해 경고하는 주석 검사기 훅을 설정합니다.

### comment_checker

**타입:** `object`
**속성:**

* `custom_prompt`: `string` (선택 사항)

#### custom_prompt

**타입:** `string`
**필수 여부:** 아니요

과도한 주석이 감지되었을 때 표시할 사용자 정의 경고 메시지입니다. 감지된 주석 XML의 플레이스홀더로 `{{comments}}`를 사용하십시오.

```

```

**출처:** [src/config/schema.ts L122-L125](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L122-L125)

## 자동 업데이트 설정 (Auto Update Configuration)

### auto_update

**타입:** `boolean`
**기본값:** `true`

플러그인이 업데이트를 자동으로 확인하고 설치할지 여부를 제어합니다. `true`인 경우, `auto-update-checker` 훅이 세션 시작 시 npm 레지스트리와 설치된 버전을 대조하여 확인합니다.

```

```

업데이트 확인을 완전히 비활성화하려면 `disabled_hooks`에 `"auto-update-checker"`를 추가하십시오:

```

```

**출처:** [src/config/schema.ts L190](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L190-L190)

 [src/index.ts L281-L286](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L281-L286)

## 실험적 기능 (Experimental Features)

`experimental` 객체는 개발 중인 기능을 포함하며, 향후 버전에서 변경되거나 제거될 수 있습니다. 프로덕션 환경에서는 주의해서 사용하십시오.

```

```

**출처:** [src/config/schema.ts L163-L176](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L163-L176)

 [README.md L835-L851](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L835-L851)

### 실험적 속성

#### aggressive_truncation

**타입:** `boolean`
**기본값:** `false`

`true`인 경우, 토큰 제한에 도달할 때 도구 출력을 공격적으로 잘라내는 기능을 활성화합니다. 기본 트런케이션(truncation) 전략보다 더 공격적으로 잘라내며, 여전히 제한을 초과하는 경우 요약(summarization)으로 폴백합니다.

```

```

이 기능은 내장된 `tool-output-truncator` 훅보다 더 공격적입니다. 상당한 데이터 손실을 의미하더라도 남은 컨텍스트 윈도우에 맞게 도구 출력을 잘라냅니다.

**출처:** [src/config/schema.ts L164](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L164-L164)

#### auto_resume

**타입:** `boolean`
**기본값:** `false`

`true`인 경우, 생각 차단(thinking block) 오류 또는 생각 비활성화 위반으로부터 성공적으로 복구한 후 실행을 자동으로 재개합니다. 이 기능이 없으면 오류 수정 후 복구가 중단됩니다.

```

```

**경고:** 에이전트가 특정 이유로 오류 상태에 있었던 경우 예상치 못한 동작이 발생할 수 있습니다. 복구 메커니즘을 신뢰할 수 있는 경우에만 사용하십시오.

**출처:** [src/config/schema.ts L165](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L165-L165)

#### preemptive_compaction

**타입:** `boolean`
**기본값:** `true`

하드 토큰 제한에 도달하기 전에 선제적 세션 압축(compaction)을 활성화합니다. 활성화되면 토큰 사용량이 `preemptive_compaction_threshold`를 초과할 때 세션이 자동으로 요약됩니다.

```

```

**출처:** [src/config/schema.ts L167](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L167-L167)

#### preemptive_compaction_threshold

**타입:** `number`
**범위:** `0.5` ~ `0.95`
**기본값:** `0.80`

선제적 압축을 트리거하는 컨텍스트 윈도우 사용 비율입니다. 예를 들어, `0.80`은 컨텍스트 윈도우의 80%가 사용되었을 때 압축을 트리거합니다.

```

```

**출처:** [src/config/schema.ts L169](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L169-L169)

#### truncate_all_tool_outputs

**타입:** `boolean`
**기본값:** `true`

`true`인 경우, 모든 도구의 출력을 잘라냅니다 (grep/glob과 같이 화이트리스트에 등록된 도구뿐만 아니라). `false`인 경우, 특정 고출력 도구만 잘라냅니다.

```

```

**출처:** [src/config/schema.ts L171](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L171-L171)

#### dcp_for_compaction

**타입:** `boolean`
**기본값:** `false`

`true`인 경우, 토큰 제한을 초과했을 때 전체 요약으로 폴백하기 전에 동적 컨텍스트 프루닝(Dynamic Context Pruning, DCP)을 첫 번째 전략으로 실행합니다. DCP는 중복된 메시지를 지능적으로 제거하려고 시도합니다.

```

```

**출처:** [src/config/schema.ts L175](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L175-L175)

#### dynamic_context_pruning

**타입:** `object`
**기본값:** `{ enabled: false }`

토큰 제한 오류를 방지하기 위해 컨텍스트 윈도우에서 중복되거나 대체된 메시지를 제거하는 실험적 시스템인 동적 컨텍스트 프루닝(DCP)을 설정합니다.

```

```

##### enabled

**타입:** `boolean`
**기본값:** `false`

DCP의 마스터 토글입니다. `false`인 경우 모든 DCP 전략이 비활성화됩니다.

##### notification

**타입:** `"off" | "minimal" | "detailed"`
**기본값:** `"detailed"`

DCP가 메시지를 프루닝(가지치기)할 때 알림의 상세도를 제어합니다:

* `"off"`: 알림 없음
* `"minimal"`: 프루닝된 메시지 수만 표시
* `"detailed"`: 프루닝된 각 메시지에 대한 상세 정보 표시

##### turn_protection

**타입:** `object`
**속성:**

* `enabled`: `boolean` (기본값: `true`)
* `turns`: `number` (기본값: `3`, 범위: `1` ~ `10`)

최근 도구 출력의 프루닝을 방지합니다. 활성화되면 마지막 N 턴(turn)의 메시지는 모든 프루닝 전략으로부터 보호됩니다.

##### protected_tools

**타입:** `string[]`
**기본값:** `["task", "todowrite", "todoread", "lsp_rename", "lsp_code_action_resolve", "session_read", "session_write", "session_search"]`

중복 여부나 생성 시기에 관계없이 출력이 절대 프루닝되지 않아야 하는 도구 이름 목록입니다.

##### strategies

**타입:** `object`

개별 프루닝 전략에 대한 설정입니다.

###### deduplication

**타입:** `object`
**속성:**

* `enabled`: `boolean` (기본값: `true`)

중복된 도구 호출(동일한 인자를 가진 동일한 도구 이름)을 제거합니다. 가장 최근의 항목만 유지합니다.

###### supersede_writes

**타입:** `object`
**속성:**

* `enabled`: `boolean` (기본값: `true`)
* `aggressive`: `boolean` (기본값: `false`)

파일이 나중에 읽혔을 때 이전의 파일 쓰기 작업을 프루닝합니다:

* `aggressive: false`인 경우, 나중에 읽힌 정확히 동일한 파일에 대한 쓰기만 프루닝합니다.
* `aggressive: true`인 경우, 이후에 읽기 작업이 하나라도 있으면 모든 쓰기 작업을 프루닝합니다.

###### purge_errors

**타입:** `object`
**속성:**

* `enabled`: `boolean` (기본값: `true`)
* `turns`: `number` (기본값: `5`, 범위: `1` ~ `20`)

N 턴이 지난 후 오류를 발생시킨 도구 호출 입력을 제거합니다. 토큰을 절약하기 위해 오류 메시지는 유지하되 입력값은 제거합니다.

**출처:** [src/config/schema.ts L127-L161](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L127-L161)

## 전체 설정 예시 (Complete Configuration Example)

다음은 모든 주요 설정 섹션을 보여주는 종합적인 예시입니다:

```

```

**출처:** [README.md L717-L851](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L717-L851)

 [assets/oh-my-opencode.schema.json L1-L1442](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json#L1-L1442)