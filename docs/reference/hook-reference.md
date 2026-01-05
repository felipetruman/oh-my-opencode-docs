---
layout: default
title: 훅 참조(Hook Reference)
parent: 참조(Reference)
nav_order: 1
---

# 훅 참조(Hook Reference)

> **관련 소스 파일**
> * [assets/oh-my-opencode.schema.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/assets/oh-my-opencode.schema.json)
> * [src/config/schema.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts)
> * [src/hooks/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts)
> * [src/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts)

이 페이지는 oh-my-opencode에서 사용 가능한 모든 생명주기 훅(lifecycle hooks)에 대한 포괄적인 참조를 제공합니다. 훅은 OpenCode 실행 흐름의 특정 지점에서 동작을 가로채고 향상시키는 모듈형 구성 요소입니다. 전체적인 훅 시스템 아키텍처에 대한 정보는 [Hook System Overview](../reliability/)를 참조하십시오. 특정 훅 카테고리에 대한 자세한 구현 가이드는 [컨텍스트 관리 훅(Context Management Hooks)](/code-yeongyu/oh-my-opencode/7.1-session-recovery), [세션 관리 훅(Session Management Hooks)](/code-yeongyu/oh-my-opencode/7.2-message-validation), [도구 향상 훅(Tool Enhancement Hooks)](/code-yeongyu/oh-my-opencode/7.3-todo-continuation-enforcer), [컨텍스트 주입 훅(Context Injection Hooks)](/code-yeongyu/oh-my-opencode/7.4-context-management-hooks), 그리고 [사용자 경험 훅(User Experience Hooks)](/code-yeongyu/oh-my-opencode/7.5-context-injection-hooks)을 참조하십시오.

---

## 훅 시스템 아키텍처 (Hook System Architecture)

훅 시스템은 특정 OpenCode 플러그인 확장 지점에 핸들러를 등록함으로써 작동합니다. 각 훅은 에이전트 워크플로우의 다양한 단계에서 이벤트를 처리하거나, 도구 입력/출력을 수정하거나, 컨텍스트를 주입할 수 있습니다.

```

```

출처: [src/index.ts L211-L576](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L211-L576)

---

## 훅 생명주기 이벤트 (Hook Lifecycle Events)

훅은 특정 OpenCode 이벤트 유형에 연결됩니다. 다음 다이어그램은 각 이벤트가 발생하는 시점과 이를 처리하는 훅을 보여줍니다:

```

```

출처: [src/index.ts L480-L574](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L480-L574)

 [README.md L44-L146](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L44-L146)

---

## 전체 훅 참조 테이블 (Complete Hook Reference Table)

다음 표는 사용 가능한 모든 훅과 해당 식별자, 트리거 조건 및 주요 목적을 나열합니다:

| 훅 이름 | 카테고리 | 트리거 이벤트 | 기본 활성화 | 주요 목적 |
| --- | --- | --- | --- | --- |
| `todo-continuation-enforcer` | 세션 관리 | `session.idle` | ✓ | 에이전트가 중단되기 전에 TODO를 완료하도록 강제함 |
| `context-window-monitor` | 컨텍스트 관리 | `message.updated`, `tool.execute.after` | ✓ | 토큰 사용량을 추적하고 에이전트에게 여유 공간을 알림 |
| `session-recovery` | 세션 관리 | `session.error` | ✓ | 누락된 도구 결과, 생각 오류(thinking errors)로부터 자동 복구 |
| `session-notification` | UX | `session.idle` | ✓ | 에이전트가 유휴 상태가 되면 OS 알림을 보냄 |
| `comment-checker` | 도구 향상 | `tool.execute.before`, `tool.execute.after` | ✓ | 과도한 코드 주석 생성을 방지함 |
| `grep-output-truncator` | 도구 향상 | `tool.execute.after` | ✓ | 컨텍스트 창에 따라 grep 출력을 자름 |
| `tool-output-truncator` | 도구 향상 | `tool.execute.after` | ✓ | 컨텍스트 보존을 위해 도구 출력을 동적으로 자름 |
| `directory-agents-injector` | 컨텍스트 주입 | `tool.execute.before`, `tool.execute.after`, `event` | ✓ | 디렉토리 계층 구조에서 AGENTS.md를 주입함 |
| `directory-readme-injector` | 컨텍스트 주입 | `tool.execute.before`, `tool.execute.after`, `event` | ✓ | 프로젝트 루트에서 README.md를 주입함 |
| `empty-task-response-detector` | 도구 향상 | `tool.execute.after` | ✓ | Task 도구가 빈 응답을 반환할 때 경고함 |
| `think-mode` | UX | `event` | ✓ | 복잡한 쿼리에 대해 확장 생각(extended thinking)을 자동 활성화함 |
| `anthropic-auto-compact` | 컨텍스트 관리 | `session.error`, `event` | ✓ | 토큰 제한 시 Claude 세션을 자동으로 압축함 |
| `preemptive-compaction` | 컨텍스트 관리 | `event` | ✓ | 80% 임계값에서 선제적으로 압축을 수행함 |
| `compaction-context-injector` | 컨텍스트 관리 | (내부) | ✓ | 요약을 위한 컨텍스트를 주입함 |
| `rules-injector` | 컨텍스트 주입 | `tool.execute.before`, `tool.execute.after`, `event` | ✓ | .claude/rules/에서 조건부 규칙을 주입함 |
| `background-notification` | UX | `event` | ✓ | 백그라운드 작업 완료 시 알림을 보냄 |
| `auto-update-checker` | UX | `event` | ✓ | oh-my-opencode 업데이트를 확인함 |
| `startup-toast` | UX | `session.created` | ✓ | 로드 시 환영 메시지를 표시함 |
| `keyword-detector` | UX | `chat.message` | ✓ | 특화된 모드 활성화를 위한 키워드를 감지함 |
| `agent-usage-reminder` | UX | `tool.execute.after`, `event` | ✓ | 사용자에게 특화된 에이전트 활용을 권장함 |
| `non-interactive-env` | 도구 향상 | `tool.execute.before` | ✓ | 비대화형 환경에서 권한을 검증함 |
| `interactive-bash-session` | 도구 향상 | `tool.execute.after`, `event` | ✓ | interactive_bash 도구를 위한 tmux 세션을 관리함 |
| `empty-message-sanitizer` | 세션 관리 | `experimental.chat.messages.transform` | ✓ | 빈 메시지로 인한 API 오류를 방지함 |
| `claude-code-hooks` | 호환성 | 다수 | ✓ | Claude Code 호환 훅을 실행함 |

출처: [src/config/schema.ts L44-L66](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L44-L66)

 [src/index.ts L230-L304](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L230-L304)

 [README.md L792-L793](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L792-L793)

---

## 훅 설정 (Hook Configuration)

### 훅 비활성화 (Disabling Hooks)

개별 훅은 설정 파일의 `disabled_hooks` 배열을 통해 비활성화할 수 있습니다:

```

```

**설정 위치:**

* 사용자: `~/.config/opencode/oh-my-opencode.json`
* 프로젝트: `.opencode/oh-my-opencode.json`

**훅 활성화 로직:**

```

```

출처: [src/index.ts L213-L214](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L213-L214)

 [README.md L846-L854](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L846-L854)

---

## 컨텍스트 관리 훅 (Context Management Hooks)

이 훅들은 토큰 사용량과 컨텍스트 창(context window) 제한을 관리합니다.

### context-window-monitor

**트리거 이벤트:** `message.updated`, `tool.execute.after`

**목적:** [컨텍스트 창 불안 관리(Context Window Anxiety Management)](https://agentic-patterns.com/patterns/context-window-anxiety-management/) 패턴을 구현합니다. 세션당 누적 토큰 사용량을 추적하고 용량의 70% 이상에 도달하면 에이전트에게 알립니다.

**주요 기능:**

* 메시지 메타데이터에서 토큰 수를 파싱함
* 세션별 토큰 누적량을 유지함
* 임계값 도달 시 알림 메시지를 주입함

**팩토리(Factory):** `createContextWindowMonitorHook(ctx)`

출처: [src/index.ts L233-L235](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L233-L235)

 [README.md L689-L691](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L689-L691)

---

### preemptive-compaction

**트리거 이벤트:** `message.updated`

**목적:** 하드 토큰 제한에 도달하기 전에 선제적으로 세션 요약을 트리거합니다. 기본 임계값은 모델 용량의 80%입니다.

**설정 옵션:**

```

```

**기능:**

* `getModelLimit()`를 통한 모델별 제한 감지
* `compaction-context-injector`와 협력
* Anthropic 1M 토큰 컨텍스트 지원

**팩토리:** `createPreemptiveCompactionHook(ctx, options)`

출처: [src/index.ts L275-L279](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L275-L279)

 [src/config/schema.ts L112-L115](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L112-L115)

---

### anthropic-auto-compact

**트리거 이벤트:** `session.error`, `event`

**목적:** 세션을 요약하고 압축하여 Anthropic 토큰 제한 오류로부터 자동으로 복구합니다. "maximum context length" 및 "prompt is too long"과 같은 특정 오류 패턴을 감지합니다.

**복구 흐름:**

1. 복구 가능한 토큰 제한 오류 감지
2. 저장소에서 유효하지 않은 메시지 삭제
3. 세션 요약 트리거
4. 압축된 컨텍스트로 재시도

**실험적 옵션:**

```

```

**팩토리:** `createAnthropicAutoCompactHook(ctx, options)`

출처: [src/index.ts L271-L273](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L271-L273)

 [README.md L692](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L692-L692)

---

### compaction-context-injector

**트리거 이벤트:** 내부 (압축 훅에 의해 호출됨)

**목적:** 세션 요약을 트리거할 때 컨텍스트 인식 지침을 주입합니다. 요약이 진행 중인 작업에 대한 중요한 정보를 보존하도록 보장합니다.

**팩토리:** `createCompactionContextInjector()`

출처: [src/index.ts L274](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L274-L274)

---

## 세션 관리 훅 (Session Management Hooks)

이 훅들은 세션 생명주기, 오류 복구 및 상태 유지를 처리합니다.

### session-recovery

**트리거 이벤트:** `session.error`

**목적:** 다음을 포함한 세션 오류로부터 자동으로 복구합니다:

* 누락된 도구 호출 결과
* 생각 블록(thinking block) 위반
* 빈 메시지 오류
* 고립된(orphaned) 함수 호출

**복구 전략:**

1. 복구 가능한 오류 패턴 감지
2. 불일치를 수정하기 위해 메시지 저장소 조작
3. 선택적으로 추출된 사용자 프롬프트로 자동 재개

**상태 조정:** 활성 복구 중에 프롬프트 주입 충돌을 방지하기 위해 `todo-continuation-enforcer`와 통합됩니다.

```

```

**실험적 옵션:**

```

```

**팩토리:** `createSessionRecoveryHook(ctx, options)`

출처: [src/index.ts L236-L248](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L236-L248)

 [src/index.ts L515-L539](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L515-L539)

 [README.md L693](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L693-L693)

---

### todo-continuation-enforcer

**트리거 이벤트:** `session.idle`

**목적:** 에이전트가 작업 중간에 멈추는 것을 방지합니다. 완료되지 않은 TODO가 있는 상태에서 세션이 유휴 상태가 되면 자동으로 연속 프롬프트를 주입합니다.

**조정:** 오류 처리 중 프롬프트 충돌을 피하기 위해 `session-recovery`의 복구 상태를 존중합니다.

**저장소 통합:** Claude Code 호환 형식으로 `~/.claude/todos/`에서 TODO 목록을 읽습니다.

**팩토리:** `createTodoContinuationEnforcer(ctx)`

출처: [src/index.ts L230-L232](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L230-L232)

 [src/index.ts L245-L247](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L245-L247)

 [README.md L687](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L687-L687)

---

### session-notification

**트리거 이벤트:** `session.idle`

**목적:** 에이전트 세션이 유휴 상태가 되면 OS 수준의 데스크톱 알림을 보냅니다. 크로스 플랫폼(macOS, Linux, Windows)을 지원합니다.

**알림 내용:**

* 세션 제목
* 마지막 에이전트 작업
* 행동 유도(Call to action)

**팩토리:** `createSessionNotification(ctx)`

출처: [src/index.ts L239-L241](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L239-L241)

 [README.md L697](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L697-L697)

---

### empty-message-sanitizer

**트리거 이벤트:** `experimental.chat.messages.transform`

**목적:** 전송 전에 메시지 내용을 정리하여 빈 채팅 메시지로 인한 API 오류를 방지합니다. 텍스트나 이미지 내용이 없는 메시지를 제거합니다.

**팩토리:** `createEmptyMessageSanitizerHook()`

출처: [src/index.ts L302-L304](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L302-L304)

 [src/index.ts L338-L344](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L338-L344)

 [README.md L699](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L699-L699)

---

## 도구 향상 훅 (Tool Enhancement Hooks)

이 훅들은 도구 실행 동작을 가로채고 향상시킵니다.

### tool-output-truncator

**트리거 이벤트:** `tool.execute.after`

**목적:** 컨텍스트 창을 보존하기 위해 장황한 도구 출력을 동적으로 자릅니다. 다음에 적용됩니다:

* `grep` - 50k 토큰으로 제한
* `glob` - 50k 토큰으로 제한
* LSP 도구 (`lsp_*`)
* `ast_grep_search`

**자르기 전략:**

* 50%의 컨텍스트 창 여유 공간 유지
* 모델별 제한 사항 준수
* 실험적 설정을 통해 공격적 모드 사용 가능

**설정:**

```

```

**팩토리:** `createToolOutputTruncatorHook(ctx, options)`

출처: [src/index.ts L253-L255](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L253-L255)

 [README.md L700-L702](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L700-L702)

---

### comment-checker

**트리거 이벤트:** `tool.execute.before`, `tool.execute.after`

**목적:** 에이전트가 과도한 코드 주석을 추가하는 것을 방지합니다. 코드 변경 사항을 분석하고 불필요한 주석에 대해 경고하는 동시에 다음과 같은 유효한 패턴은 보존합니다:

* BDD 스타일 주석 (Given/When/Then)
* 지시어 주석 (eslint-disable, prettier-ignore)
* 문서화 주석 (JSDoc, TSDoc)
* 섹션 마커

**감지 로직:**

* 변경 전/후 코드 비교
* 새로 추가된 주석 식별
* 주석 유형 분류
* 가치가 낮은 주석에 대해 경고 발행

**팩토리:** `createCommentCheckerHooks()`

출처: [src/index.ts L250-L252](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L250-L252)

 [README.md L687-L688](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L687-L688)

---

### empty-task-response-detector

**트리거 이벤트:** `tool.execute.after`

**목적:** `task` 도구가 빈 응답을 반환하는 경우를 감지하여 잠재적인 에이전트 실패를 나타냅니다. 사용자에게 무한 대기를 피하도록 경고합니다.

**감지 기준:**

* 도구 이름: `task`
* 출력에 의미 있는 내용이 없음
* 출력이 임계값보다 짧음

**팩토리:** `createEmptyTaskResponseDetectorHook(ctx)`

출처: [src/index.ts L262-L264](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L262-L264)

 [README.md L698](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L698-L698)

---

### non-interactive-env

**트리거 이벤트:** `tool.execute.before`

**목적:** 비대화형 환경(CI/CD, 헤드리스)에서 도구 실행을 검증합니다. 사용자가 대화식으로 승인할 수 없는 경우 파괴적인 작업에 대한 권한 확인을 강제합니다.

**영향을 받는 도구:**

* `bash` - 명령 실행
* `edit` - 파일 수정
* `webfetch` - 네트워크 요청

**팩토리:** `createNonInteractiveEnvHook(ctx)`

출처: [src/index.ts L296-L298](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L296-L298)

---

### interactive-bash-session

**트리거 이벤트:** `tool.execute.after`, `session.deleted`

**목적:** `interactive_bash` 도구를 위한 tmux 세션을 관리합니다. 세션 상태를 추적하고 정보를 유지하며 정리를 처리합니다.

**상태 관리:**

* tmux 세션 생성/연결
* 세션과 tmux 간의 매핑 추적
* 상태를 JSON 파일로 유지
* 고립된 세션 정리

**저장 위치:** 저장소 디렉토리 내 세션별 JSON 파일

**팩토리:** `createInteractiveBashSessionHook(ctx)`

출처: [src/index.ts L299-L301](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L299-L301)

 [README.md L147-L148](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L147-L148)

---

## 컨텍스트 주입 훅 (Context Injection Hooks)

이 훅들은 전략적 지점에 컨텍스트 정보를 주입합니다.

### directory-agents-injector

**트리거 이벤트:** `tool.execute.before`, `tool.execute.after`, `event`

**목적:** 파일을 읽을 때 디렉토리 계층 구조에서 `AGENTS.md` 파일을 자동으로 주입합니다. 파일 디렉토리에서 프로젝트 루트까지 거슬러 올라가며 모든 `AGENTS.md` 파일을 수집합니다.

**주입 패턴:**

```markdown
project/AGENTS.md          # 프로젝트 전반의 컨텍스트
├── src/AGENTS.md          # src 전용 컨텍스트
    └── components/AGENTS.md   # 컴포넌트 전용 컨텍스트
```

**기능:**

* 경로 기반 계층 구조 탐색
* 세션당 1회 주입
* 누적 컨텍스트 축적

**팩토리:** `createDirectoryAgentsInjectorHook(ctx)`

출처: [src/index.ts L256-L258](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L256-L258)

 [README.md L551-L561](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L551-L561)

---

### directory-readme-injector

**트리거 이벤트:** `tool.execute.before`, `tool.execute.after`, `event`

**목적:** 상위 수준의 프로젝트 컨텍스트를 제공하기 위해 프로젝트 루트의 `README.md`를 주입합니다. AGENTS.md 주입과 유사하지만 특히 프로젝트 문서에 특화되어 있습니다.

**주입 시점:** 세션 내 첫 번째 파일 읽기 작업 시

**팩토리:** `createDirectoryReadmeInjectorHook(ctx)`

출처: [src/index.ts L259-L261](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L259-L261)

---

### rules-injector

**트리거 이벤트:** `tool.execute.before`, `tool.execute.after`, `event`

**목적:** 글로브(glob) 패턴 매칭을 기반으로 `.claude/rules/`에서 조건부 코딩 규칙을 주입합니다. 항상 적용되는 규칙과 컨텍스트별 규칙을 모두 지원합니다.

**규칙 파일 형식:**

```

```

**검색 경로:**

1. 사용자 규칙: `~/.claude/rules/`
2. 프로젝트 디렉토리 → 루트 (상향 탐색)

**지원 형식:** `.md`, `.mdc`

**팩토리:** `createRulesInjectorHook(ctx)`

출처: [src/index.ts L280-L282](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L280-L282)

 [README.md L562-L577](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L562-L577)

---

## 사용자 경험 훅 (User Experience Hooks)

이 훅들은 사용자 상호작용과 워크플로우를 향상시킵니다.

### keyword-detector

**트리거 이벤트:** `chat.message`

**목적:** 사용자 프롬프트에서 키워드를 감지하여 특화된 모드를 자동으로 활성화합니다:

| 키워드 | 활성화 모드 | 효과 |
| --- | --- | --- |
| `ultrawork`, `ulw` | 최대 성능 | 병렬 에이전트 오케스트레이션 |
| `search`, `find`, `찾아`, `検索` | 최대 검색 | 병렬 탐색 + 사서(librarian) |
| `analyze`, `investigate`, `분석`, `調査` | 심층 분석 | 다단계 전문가 컨설팅 |

**구현:** 수신 메시지 내용에서 키워드 패턴을 스캔하고 에이전트 설정을 동적으로 수정합니다.

**팩토리:** `createKeywordDetectorHook()`

출처: [src/index.ts L290-L292](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L290-L292)

 [README.md L682-L685](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L682-L685)

---

### think-mode

**트리거 이벤트:** `event`

**목적:** 확장 생각(extended thinking)이 필요한 시점을 자동으로 감지하고 모델 설정을 동적으로 조정합니다. "think deeply", "ultrathink"와 같은 문구나 복잡한 추론 요청 시 트리거됩니다.

**모델 조정:**

* 확장 생각 모드 활성화
* 생각 토큰 할당량 증가
* 추론을 위한 온도(temperature) 조정

**팩토리:** `createThinkModeHook()`

출처: [src/index.ts L265-L267](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L265-L267)

 [README.md L688-L689](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L688-L689)

---

### auto-update-checker

**트리거 이벤트:** `config`, `event`

**목적:** npm에서 새로운 oh-my-opencode 버전을 확인하고 사용자에게 알립니다. 선택적으로 플러그인을 자동 업데이트합니다.

**설정:**

```

```

**기능:**

* npm 레지스트리와 버전 비교
* 데스크톱 토스트 알림
* 자동 플러그인 업데이트 (활성화 시)
* 업데이트 후 캐시 무효화

**팩토리:** `createAutoUpdateCheckerHook(ctx, options)`

출처: [src/index.ts L283-L289](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L283-L289)

 [README.md L694](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L694-L694)

---

### startup-toast

**트리거 이벤트:** `session.created` (메인 세션 전용)

**목적:** OhMyOpenCode가 로드될 때 환영 메시지를 표시합니다. 버전 정보와 주요 기능을 보여줍니다.

**표시 조건:**

* 메인 세션에만 해당 (하위 에이전트 제외)
* 첫 번째 세션 생성 시
* `auto-update-checker` 옵션에 의해 제어됨

**메시지 내용:** 버전 번호가 포함된 "oMoMoMo..."

출처: [src/index.ts L285-L286](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L285-L286)

 [README.md L695](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L695-L695)

---

### background-notification

**트리거 이벤트:** `session.idle`

**목적:** 백그라운드 에이전트 작업이 완료되면 부모 세션에 알립니다. 데스크톱 알림 시스템과 통합되어 있습니다.

**알림 흐름:**

1. 백그라운드 작업 완료
2. `BackgroundManager`가 완료 표시
3. 훅이 유휴 상태의 부모 세션 감지
4. 작업 결과와 함께 데스크톱 알림 전송

**팩토리:** `createBackgroundNotificationHook(backgroundManager)`

출처: [src/index.ts L308-L310](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L308-L310)

 [README.md L696](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L696-L696)

---

### agent-usage-reminder

**트리거 이벤트:** `tool.execute.after`, `event`

**목적:** 검색 도구를 직접 호출할 때 사용자에게 특화된 에이전트 활용을 권장합니다. 더 나은 결과를 위해 백그라운드 에이전트에게 위임할 것을 제안합니다.

**트리거 도구:**

* `grep`
* `glob`
* `ast_grep_search`

**권장 메시지:** "더 철저한 분석을 위해 background_task를 통해 @librarian 또는 @explore 에이전트를 사용하는 것을 고려해 보세요."

**팩토리:** `createAgentUsageReminderHook(ctx)`

출처: [src/index.ts L293-L295](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L293-L295)

 [README.md L691-L692](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L691-L692)

---

## Claude Code 호환성 훅 (Claude Code Compatibility Hook)

### claude-code-hooks

**트리거 이벤트:** 다수 (모든 훅 유형)

**목적:** 이전 버전과의 호환성을 위해 Claude Code `settings.json` 파일의 훅을 실행합니다. PreToolUse, PostToolUse, UserPromptSubmit 및 Stop 훅을 지원합니다.

**검색 경로:**

1. `~/.claude/settings.json` (사용자)
2. `./.claude/settings.json` (프로젝트)
3. `./.claude/settings.local.json` (로컬, git 무시됨)

**지원되는 훅 유형:**

* **PreToolUse:** 도구 실행 전에 실행되며, 입력을 차단하거나 수정할 수 있음
* **PostToolUse:** 도구 실행 후에 실행되며, 경고나 컨텍스트를 추가할 수 있음
* **UserPromptSubmit:** 프롬프트 제출 시 실행되며, 메시지를 차단하거나 주입할 수 있음
* **Stop:** 세션 유휴 시 실행되며, 후속 프롬프트를 주입할 수 있음

**설정 토글:**

```

```

**팩토리:** `createClaudeCodeHooksHook(ctx, options)`

출처: [src/index.ts L268-L270](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L268-L270)

 [README.md L595-L623](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L595-L623)

---

## 훅 실행 순서 (Hook Execution Order)

각 이벤트 유형 내에서 훅은 플러그인에 등록된 순서대로 실행됩니다. 다음 다이어그램은 일반적인 실행 시퀀스를 보여줍니다:

```

```

출처: [src/index.ts L480-L574](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L480-L574)

---

## 고급 훅 패턴 (Advanced Hook Patterns)

### 훅 간의 상태 조정 (State Coordination Between Hooks)

일부 훅은 충돌을 방지하기 위해 상태를 조정합니다:

```

```

이는 활성 오류 복구 중에 continuation enforcer가 프롬프트를 주입하는 것을 방지합니다.

출처: [src/index.ts L244-L248](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L244-L248)

---

### 훅 전용 실험적 플래그 (Hook-Specific Experimental Flags)

많은 훅이 실험적 설정 플래그를 준수합니다:

```

```

이 플래그들은 특정 훅에 대해 미리보기 기능이나 더 공격적인 동작을 활성화합니다.

출처: [src/config/schema.ts L109-L118](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/config/schema.ts#L109-L118)

 [src/index.ts L237-L254](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L237-L254)

---

## 훅 팩토리 패턴 (Hook Factory Pattern)

모든 훅은 생성을 위해 팩토리 패턴을 따릅니다:

```

```

훅은 처리하는 생명주기 이벤트에 따라 `null` 또는 부분 핸들러를 반환합니다.

출처: [src/hooks/index.ts L1-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/hooks/index.ts#L1-L24)

 [src/index.ts L230-L304](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/index.ts#L230-L304)