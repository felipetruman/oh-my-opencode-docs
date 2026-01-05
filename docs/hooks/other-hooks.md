---
layout: default
title: 기타 훅 (Other Hooks)
parent: Hooks
nav_order: 1
---

# 기타 훅 (Other Hooks)

> **관련 소스 파일**
> * [assets/oh-my-opencode.schema.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json)
> * [src/config/schema.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts)
> * [src/hooks/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts)
> * [src/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts)

## 목적 및 범위 (Purpose and Scope)

이 페이지는 다른 안정성 시스템 섹션에서 다루지 않은 훅(Hook)들에 대해 설명합니다. 이 훅들은 사용자 경험(UX), 품질 보증(QA) 및 시스템 통합을 향상시킵니다.

**사용자 대면 훅 (User-Facing Hooks)**: 자동 업데이트 확인(Auto-update checker), 시작 토스트(Startup toast), 키워드 탐지기(Keyword detector), 에이전트 사용 알림(Agent usage reminder)
**품질 보증 훅 (Quality Assurance Hooks)**: 주석 확인기(Comment checker), 빈 작업 응답 탐지기(Empty task response detector)
**통합 훅 (Integration Hooks)**: 세션 알림(Session notification), 백그라운드 알림(Background notification), 생각 모드(Think mode), Claude Code 훅

오류 복구 훅에 대해서는 7.1페이지(세션 복구)를 참조하십시오. 메시지 검증은 7.2페이지, 작업 연속성은 7.3페이지, 컨텍스트(Context) 관리는 7.4페이지, 컨텍스트 주입은 7.5페이지, 비대화형 환경은 7.6페이지를 참조하십시오.

---

## 자동 업데이트 확인 훅 (Auto-Update Checker Hook)

`auto-update-checker` 훅은 npm에서 새로운 플러그인 버전을 모니터링하고, 로컬 패키지 캐시를 무효화하여 OpenCode 재시작 시 자동 업데이트가 실행되도록 트리거합니다.

### 업데이트 탐지 흐름 (Update Detection Flow)

```

```

**다이어그램: 자동 업데이트 확인 결정 트리 (Auto-Update Checker Decision Tree)**

출처: [src/hooks/auto-update-checker/index.ts L8-L69](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/index.ts#L8-L69)

 [src/hooks/auto-update-checker/checker.ts L173-L205](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/checker.ts#L173-L205)

### 버전 탐지 전략 (Version Detection Strategies)

이 훅은 폴백(Fallback, 대체 작동) 로직을 포함한 네 가지 버전 탐지 전략을 구현합니다.

| 전략 (Strategy) | 위치 | 우선순위 | 사용 사례 |
| --- | --- | --- | --- |
| **로컬 개발 (Local Dev)** | `opencode.json` 내의 `file://` | 가장 높음 | `bun link`를 사용한 개발 환경 |
| **고정 버전 (Pinned Version)** | 설정 내의 `oh-my-opencode@1.2.3` | 높음 | 특정 버전으로 고정 |
| **캐시된 패키지 (Cached Package)** | `~/.cache/opencode/node_modules/oh-my-opencode/package.json` | 중간 | 일반적인 설치 환경 |
| **현재 모듈 (Current Module)** | `import.meta.url`에서 상위로 탐색 | 가장 낮음 | 폴백 해결 |

출처: [src/hooks/auto-update-checker/checker.ts L15-L150](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/checker.ts#L15-L150)

### 설정 검색 경로 (Configuration Search Paths)

이 훅은 다음 우선순위에 따라 OpenCode 설정을 검색합니다.

```

```

**다이어그램: 설정 탐지 흐름 (Configuration Detection Flow)**

출처: [src/hooks/auto-update-checker/checker.ts L19-L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/checker.ts#L19-L126)

 [src/hooks/auto-update-checker/constants.ts L29-L42](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/constants.ts#L29-L42)

### 캐시 무효화 메커니즘 (Cache Invalidation Mechanism)

업데이트가 탐지되면, 훅은 캐시된 플러그인을 무효화하여 OpenCode가 다시 다운로드하도록 강제합니다.

```

```

`invalidatePackage()` 함수는 두 가지 작업을 수행합니다.

1. **플러그인 디렉토리 제거**: `~/.cache/opencode/node_modules/oh-my-opencode/` 삭제
2. **의존성 항목 제거**: `~/.cache/opencode/package.json` 의존성에서 `oh-my-opencode` 제거

출처: [src/hooks/auto-update-checker/cache.ts L6-L41](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/cache.ts#L6-L41)

### 토스트 알림 변형 (Toast Notification Variants)

이 훅은 탐지 결과에 따라 다른 토스트(Toast) 메시지를 표시합니다.

| 조건 | 제목 | 메시지 | 지속 시간 | 변형 (Variant) |
| --- | --- | --- | --- | --- |
| **업데이트 가능** | `OhMyOpenCode {latestVersion}` | `OpenCode is now on Steroids. oMoMoMoMo...\nv{latestVersion} available. Restart OpenCode to apply.` | 8000ms | info |
| **최신 상태** | `OhMyOpenCode {currentVersion}` | `OpenCode is now on Steroids. oMoMoMoMo...` | 5000ms | info |
| **로컬 개발** | `OhMyOpenCode {devVersion}` | `OpenCode is now on Steroids. oMoMoMoMo...` | 5000ms | info |
| **버전 고정** | `OhMyOpenCode {pinnedVersion}` | `OpenCode is now on Steroids. oMoMoMoMo...` | 5000ms | info |

출처: [src/hooks/auto-update-checker/index.ts L52-L83](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/index.ts#L52-L83)

### 훅 초기화 및 설정 (Hook Initialization and Configuration)

```

```

이 훅은 `showStartupToast` 플래그를 포함한 `AutoUpdateCheckerOptions` 객체를 허용합니다. 설정에서 `startup-toast`가 비활성화되면, `auto-update-checker`가 활성화되어 있더라도 버전 토스트가 표시되지 않습니다.

출처: [src/index.ts L232-L236](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L232-L236)

 [src/hooks/auto-update-checker/types.ts L25-L27](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/types.ts#L25-L27)

---

## 시작 토스트 훅 (Startup Toast Hook)

`startup-toast` 훅은 별도의 훅이 아니라 `auto-update-checker`를 위한 설정 플래그입니다. 활성화되면 첫 번째 메인 세션이 생성될 때 현재 플러그인 버전을 보여주는 환영 토스트를 표시합니다.

### 토스트 제어 로직 (Toast Control Logic)

```

```

**다이어그램: 시작 토스트 제어 흐름 (Startup Toast Control Flow)**

시작 토스트는 명시적으로 비활성화되지 않는 한 모든 탐지 결과(업데이트 가능, 최신 상태, 로컬 개발, 버전 고정)에 대해 나타납니다.

출처: [src/index.ts L232-L236](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L232-L236)

 [src/hooks/auto-update-checker/index.ts L9-L83](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/auto-update-checker/index.ts#L9-L83)

---

## 키워드 탐지 훅 (Keyword Detector Hook)

`keyword-detector` 훅은 사용자 메시지에서 특수 키워드를 분석하고 대화에 최적화 제안을 주입합니다. 이 훅은 `chat.message` 및 `event` 생명주기(Lifecycle) 훅에서 작동합니다.

### 탐지된 키워드 및 동작 (Detected Keywords and Actions)

```

```

**다이어그램: 키워드 탐지 및 응답 매핑 (Keyword Detection and Response Mapping)**

출처: [src/index.ts L237-L395](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L237-L395)

### 훅 구현 구조 (Hook Implementation Structure)

키워드 탐지기는 두 가지 생명주기 메서드를 구현합니다.

| 메서드 | 트리거 | 목적 |
| --- | --- | --- |
| `chat.message` | 사용자가 메시지 제출 | 에이전트가 메시지를 보기 전에 키워드 전처리 |
| `event` | 세션 이벤트 | 세션 상태 변경에 반응 |

이 훅은 탐지된 키워드를 기반으로 시스템 프롬프트(System prompt)를 주입하거나 에이전트 지침을 수정할 가능성이 높으나, 정확한 구현은 로드되지 않은 파일에 있습니다.

출처: [src/index.ts L237-L282](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L237-L282)

---

## 에이전트 사용 알림 훅 (Agent Usage Reminder Hook)

`agent-usage-reminder` 훅은 사용자 쿼리가 특정 패턴과 일치할 때 전문화된 에이전트 사용을 제안합니다. 이 훅은 `event` 및 `tool.execute.after` 생명주기 훅에서 작동합니다.

### 훅 생명주기 통합 (Hook Lifecycle Integration)

```

```

**다이어그램: 에이전트 사용 알림 트리거 포인트 (Agent Usage Reminder Trigger Points)**

출처: [src/index.ts L240-L530](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L240-L530)

### 컨텍스트 인식 제안 패턴 (Context-Aware Suggestion Patterns)

이 훅은 대화 컨텍스트와 도구 사용 패턴을 모니터링하여 적절한 시점에 전문 에이전트를 제안합니다.

| 탐지된 패턴 | 제안된 에이전트 | 근거 |
| --- | --- | --- |
| 결과가 적은 다수의 `grep` 호출 | `explore` | 병렬 코드베이스 탐색이 더 효율적임 |
| 외부 라이브러리에 대한 질문 | `librarian` | GitHub 및 문서 조사가 필요함 |
| 아키텍처 또는 설계 질문 | `oracle` | 전문가 수준의 기술 가이드가 필요함 |
| UI/UX 구현 요청 | `frontend` | 전문 프론트엔드 엔지니어가 필요함 |

알림 훅은 대화에서 이미 언급된 에이전트를 추적하여 중복 제안을 방지합니다.

출처: [src/index.ts L240-L242](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L240-L242)

---

## 훅 설정 및 비활성화 (Hook Configuration and Disabling)

모든 사용자 경험 훅은 `disabled_hooks` 설정 배열을 통해 개별적으로 비활성화할 수 있습니다.

```

```

### 훅 활성화 로직 (Hook Enablement Logic)

```

```

**다이어그램: 훅 조건부 초기화 흐름 (Hook Conditional Initialization Flow)**

`isHookEnabled()` 헬퍼 함수는 훅 이름이 비활성화 세트에 존재하는지 확인합니다: `!disabledHooks.has(hookName)`. 비활성화된 경우 훅은 `null`을 반환하며 해당 생명주기 메서드는 호출되지 않습니다.

출처: [src/index.ts L182-L248](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L182-L248)

 [src/config/schema.ts L44-L65](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L44-L65)

### 훅 이름 스키마 검증 (Hook Name Schema Validation)

훅 이름은 설정의 `HookNameSchema` 열거형(Enum)에 대해 검증됩니다.

```

```

`disabled_hooks`에 유효하지 않은 훅 이름이 있으면 플러그인 로드 시 설정 검증 오류가 발생합니다.

출처: [src/config/schema.ts L44-L65](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L44-L65)

---

## 이벤트 시스템과의 통합 (Integration with Event System)

사용자 경험 훅은 여러 생명주기 지점을 통해 플러그인 이벤트 시스템과 통합됩니다.

```

```

**다이어그램: 사용자 경험 훅 이벤트 통합 (User Experience Hooks Event Integration)**

각 훅은 [src/index.ts L383-L491](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L383-L491)에 있는 중앙 디스패처(Dispatcher)를 통해 이벤트를 수신하며, 이를 통해 대화 흐름을 관찰하고 영향을 줄 수 있습니다.

출처: [src/index.ts L279-L491](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L279-L491)