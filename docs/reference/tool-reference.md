---
layout: default
title: 도구 레퍼런스 (Tool Reference)
parent: "레퍼런스"
nav_order: 4
---

# 도구 레퍼런스 (Tool Reference)

> **관련 소스 파일**
> * [README.ja.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ja.md)
> * [README.ko.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.ko.md)
> * [README.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md)
> * [README.zh-cn.md](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.zh-cn.md)
> * [src/shared/config-path.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/config-path.ts)
> * [src/tools/background-task/constants.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/constants.ts)
> * [src/tools/background-task/index.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/index.ts)
> * [src/tools/background-task/types.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/types.ts)
> * [src/tools/call-omo-agent/constants.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/constants.ts)
> * [src/tools/interactive-bash/constants.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/interactive-bash/constants.ts)
> * [src/tools/look-at/constants.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/look-at/constants.ts)
> * [src/tools/look-at/tools.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/look-at/tools.ts)

이 페이지는 oh-my-opencode에서 사용할 수 있는 모든 도구에 대한 전체 레퍼런스를 제공합니다. 도구는 카테고리별로 정리되어 있으며, 정확한 매개변수(parameter), 설명 및 구현 세부 사항을 포함합니다. 아키텍처 맥락에 대해서는 5페이지(도구 시스템)를 참조하십시오.

모든 도구 설명과 매개변수는 정확성을 보장하기 위해 코드베이스에서 직접 가져왔습니다.

## 도구 구성 (Tool Organization)

Oh-my-opencode는 6개의 기능 카테고리에 걸쳐 20개 이상의 도구를 제공합니다. 도구는 OpenCode 플러그인 시스템을 통해 등록되며, 권한 설정에 따라 에이전트가 사용할 수 있게 됩니다.

**도구 등록 아키텍처 (Tool Registration Architecture)**

```

```

출처: [src/tools/index.ts L1-L69](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L1-L69)

## LSP 도구 (LSP Tools)

Language Server Protocol (LSP) 도구는 코드 탐색, 분석 및 리팩토링을 위한 IDE와 유사한 기능을 제공합니다. 모든 LSP 도구는 대상 파일의 언어에 대해 활성화된 LSP 서버가 필요합니다.

| 도구 이름 | 목적 | 주요 매개변수 | 출력 유형 |
| --- | --- | --- | --- |
| `lsp_hover` | 커서 위치의 타입 정보, 문서 및 시그니처(signature) 가져오기 | `path` (string)`line` (number)`character` (number) | 타입 시그니처 및 문서를 포함한 호버 정보 |
| `lsp_goto_definition` | 심볼 정의로 이동 | `path` (string)`line` (number)`character` (number) | 정의 위치(들) |
| `lsp_find_references` | 워크스페이스 전체에서 심볼의 모든 사용처 찾기 | `path` (string)`line` (number)`character` (number)`includeDeclaration` (boolean) | 참조 위치 배열 |
| `lsp_document_symbols` | 파일 심볼의 구조화된 개요 가져오기 | `path` (string) | 계층적 심볼 트리 |
| `lsp_workspace_symbols` | 전체 프로젝트에서 이름으로 심볼 검색 | `query` (string) | 위치를 포함한 일치하는 심볼 배열 |
| `lsp_diagnostics` | 파일의 오류, 경고 및 힌트 가져오기 | `path` (string) | 심각도가 포함된 진단 메시지 배열 |
| `lsp_servers` | 사용 가능한 LSP 서버 및 상태 목록 표시 | 없음 | 서버 정보 배열 |
| `lsp_prepare_rename` | 이름 변경 작업이 안전한지 확인 | `path` (string)`line` (number)`character` (number) | 범위 및 유효성 검사 결과 |
| `lsp_rename` | 전체 워크스페이스에서 심볼 이름 변경 | `path` (string)`line` (number)`character` (number)`newName` (string) | 적용할 워크스페이스 편집 사항 |
| `lsp_code_actions` | 사용 가능한 빠른 수정(quick fix) 및 리팩토링 가져오기 | `path` (string)`startLine` (number)`startChar` (number)`endLine` (number)`endChar` (number) | 사용 가능한 코드 액션 배열 |
| `lsp_code_action_resolve` | 선택한 코드 액션 적용 | `action` (object) | 액션 적용 결과 |

**액세스 제한**: 특별한 제한 없음. 도구 권한을 통해 명시적으로 거부되지 않는 한 모든 에이전트가 사용할 수 있습니다.

출처: [src/tools/index.ts L1-L13](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L1-L13)

 [README.md L310-L321](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L310-L321)

## AST 도구 (AST Tools)

Abstract Syntax Tree (AST, 추상 구문 트리) 도구는 25개 이상의 프로그래밍 언어에 대해 구문 인식 코드 검색 및 교체 기능을 제공합니다.

| 도구 이름 | 목적 | 주요 매개변수 | 출력 유형 | 비고 |
| --- | --- | --- | --- | --- |
| `ast_grep_search` | AST 규칙을 사용하여 코드 패턴 검색 | `pattern` (string)`language` (string)`paths` (array, optional)`rule` (object, optional) | 컨텍스트를 포함한 일치 항목 배열 | 복잡한 AST 쿼리 및 패턴 매칭 지원 |
| `ast_grep_replace` | AST 구조를 보존하며 코드 패턴 교체 | `pattern` (string)`replacement` (string)`language` (string)`paths` (array) | 수정된 파일 배열 | 정규식(regex) 교체보다 안전함 |

**지원 언어**: JavaScript, TypeScript, Python, Rust, Go, Java, C, C++, C#, Ruby, PHP, Swift, Kotlin, Dart, Lua, Bash 및 기타 10개 이상.

**액세스 제한**: 특별한 제한 없음. 모든 에이전트가 사용할 수 있습니다.

출처: [src/tools/index.ts L15-L18](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L15-L18)

 [README.md L322-L323](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L322-L323)

## 검색 및 파일 탐색 도구 (Search and File Discovery Tools)

성능 최적화 및 타임아웃 보호 기능이 강화된 파일 시스템 도구입니다.

| 도구 이름 | 목적 | 주요 매개변수 | 출력 유형 | 구현 방식 |
| --- | --- | --- | --- | --- |
| `grep` | 패턴 매칭을 통한 파일 내용 검색 | `pattern` (string)`paths` (array, optional)`file_pattern` (string, optional)`case_sensitive` (boolean) | 라인 번호를 포함한 일치 항목 배열 | 성능을 위해 ripgrep 바이너리 사용 |
| `glob` | 경로 패턴과 일치하는 파일 찾기 | `patterns` (array)`exclude` (array, optional) | 일치하는 파일 경로 배열 | 표준 glob 패턴 지원 |

**grep 구현 세부 사항**:

* 처음 사용 시 플랫폼별 ripgrep 바이너리를 다운로드합니다.
* 바이너리는 `~/.cache/oh-my-opencode/bin/`에 캐시됩니다.
* 타임아웃 보호 기능을 제공합니다 (기본값: 30초).
* 타임아웃 기능이 없는 OpenCode의 기본 grep을 대체합니다.

**액세스 제한**: 특별한 제한 없음. 출력 결과는 tool-output-truncator 훅에 의해 잘릴 수 있습니다.

출처: [src/tools/index.ts L20-L21](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L20-L21)

 [README.md L362](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L362-L362)

## 백그라운드 도구 (Background Tools)

병렬 백그라운드 작업 실행을 관리하기 위한 도구입니다. 이를 통해 진정한 멀티 에이전트 병렬 처리가 가능해집니다.

| 도구 이름 | 목적 | 주요 매개변수 | 출력 유형 | 설명 출처 |
| --- | --- | --- | --- | --- |
| `background_task` | 에이전트 실행을 백그라운드에서 시작 | `description` (string)`prompt` (string)`agent` (string) | 작업 ID (즉시 반환됨) | `BACKGROUND_TASK_DESCRIPTION` 상수 |
| `background_output` | 상태 확인 및 작업 결과 검색 | `task_id` (string)`block` (boolean, optional)`timeout` (number, optional) | 작업 상태, 출력 및 완료 상태 | `BACKGROUND_OUTPUT_DESCRIPTION` 상수 |
| `background_cancel` | 실행 중인 백그라운드 작업 취소 | `taskId` (string, optional)`all` (boolean, optional) | 취소 확인 | `BACKGROUND_CANCEL_DESCRIPTION` 상수 |

**백그라운드 작업 생명주기 (Background Task Lifecycle)**:

```

```

**도구 설명** (코드에 정의된 내용):

* `background_task`: "에이전트 작업을 백그라운드에서 실행합니다. 즉시 task_id를 반환하며 완료 시 알림을 보냅니다. 결과를 얻으려면 `background_output`을 사용하십시오. 프롬프트는 반드시 영어여야 합니다."
* `background_output`: "백그라운드 작업에서 출력을 가져옵니다. 시스템이 완료 시 알림을 보내므로 block=true가 필요한 경우는 드뭅니다."
* `background_cancel`: "실행 중인 백그라운드 작업을 취소합니다. 최종 답변 전에 모든 작업을 취소하려면 all=true를 사용하십시오."

**액세스 제한**:

* multimodal-looker를 제외한 모든 에이전트가 사용할 수 있습니다.
* 작업 생성에는 유효한 에이전트 이름이 필요합니다.
* 부모 세션만 자신의 작업을 조회하거나 취소할 수 있습니다.
* 완료 시 background-notification 훅을 통해 부모 세션에 알림이 전송됩니다.

출처: [src/tools/background-task/constants.ts L1-L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/constants.ts#L1-L8)

 [src/tools/background-task/types.ts L1-L17](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/background-task/types.ts#L1-L17)

## 에이전트 위임 도구 (Agent Delegation Tool)

특화된 에이전트를 동기 또는 비동기 방식으로 호출하기 위한 도구입니다.

| 도구 이름 | 목적 | 주요 매개변수 | 출력 유형 | 설명 출처 |
| --- | --- | --- | --- | --- |
| `call_omo_agent` | 프롬프트와 함께 특화 에이전트 호출 | `agent` (string)`prompt` (string)`run_in_background` (boolean, required) | 에이전트 응답 (동기) 또는 작업 ID (비동기) | `CALL_OMO_AGENT_DESCRIPTION` 상수 |

**도구 설명** (코드에 정의된 내용):
"explore/librarian 에이전트를 생성합니다. run_in_background는 필수 항목입니다 (true=task_id를 포함한 비동기, false=동기). 사용 가능 에이전트: {agents}. 프롬프트는 반드시 영어여야 합니다. 비동기 결과에는 `background_output`을 사용하십시오."

**허용된 에이전트** (`ALLOWED_AGENTS` 상수 기준):

* `explore` - 코드베이스 탐색
* `librarian` - 문서 조사

**참고**: `call_omo_agent`를 통해서는 `explore`와 `librarian`만 허용됩니다. README에 나열된 다른 에이전트들은 다른 메커니즘을 통해 호출됩니다.

**에이전트 호출 흐름 (Agent Invocation Flow)**:

```

```

**액세스 제한**:

* 이 도구를 통해서는 `explore` 및 `librarian` 에이전트만 호출할 수 있습니다.
* 이 에이전트들은 `call_omo_agent`를 재귀적으로 호출할 수 없습니다 (무한 재귀 방지).
* `run_in_background` 매개변수는 필수입니다 (선택 사항 아님).
* 프롬프트는 반드시 영어여야 합니다.

출처: [src/tools/call-omo-agent/constants.ts L1-L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/constants.ts#L1-L8)

## 특수 도구 (Special Tools)

멀티모달(Multimodal) 분석, 대화형 세션 및 사용자 정의 확장을 위한 특수 도구입니다.

| 도구 이름 | 목적 | 주요 매개변수 | 출력 유형 | 설명 출처 |
| --- | --- | --- | --- | --- |
| `look_at` | 시각적 콘텐츠(PDF, 이미지, 다이어그램) 분석 | `path` (string)`prompt` (string, optional) | 추출된 텍스트 및 시각적 분석 결과 | `LOOK_AT_DESCRIPTION` 상수 |
| `interactive_bash` | 지속적인 세션을 위해 tmux 명령 실행 | `tmux_command` (string) | 명령 출력 또는 오류 | `INTERACTIVE_BASH_DESCRIPTION` 상수 |
| `slashcommand` | 사용자 정의 슬래시 명령 실행 | `command` (string)추가 매개변수는 가변적임 | 명령별 출력 | Claude Code 호환 경로에서 로드됨 |
| `skill` | 사용자 정의 스킬(skill) 호출 | `skill_name` (string)추가 매개변수는 가변적임 | 스킬별 출력 | Claude Code 호환 경로에서 로드됨 |

**도구 설명** (코드에 정의된 내용):

* `look_at`: "별도의 컨텍스트에서 Gemini 2.5 Flash를 통해 미디어 파일(PDF, 이미지, 다이어그램)을 분석합니다. 메인 컨텍스트의 토큰을 절약합니다."
* `interactive_bash`: "tmux 명령을 실행합니다. 'omo-{name}' 세션 패턴을 사용하십시오. 차단됨 (대신 bash 사용): capture-pane, save-buffer, show-buffer, pipe-pane."

**look_at 도구 세부 사항**:

* 내부적으로 `MULTIMODAL_LOOKER_AGENT` 상수에 지정된 에이전트("multimodal-looker")에게 위임합니다.
* PDF, PNG, JPG, GIF, WebP 형식을 지원합니다.
* 텍스트 추출, 다이어그램 분석, 시각적 콘텐츠 해석을 수행합니다.
* 부모 컨텍스트에 전체 파일 내용을 포함하지 않음으로써 토큰을 절약합니다.

**interactive_bash 도구 세부 사항**:

```

```

**interactive_bash 요구 사항**:

* tmux가 설치되어 있어야 하며 PATH에 포함되어야 합니다.
* 플러그인 로드 시 백그라운드 체크를 통해 경로를 탐색합니다.
* 기본 타임아웃: `DEFAULT_TIMEOUT_MS`에 정의된 60,000ms (60초).
* interactive-bash-session 훅을 통해 OpenCode 세션별로 세션이 추적됩니다.
* 세션 삭제 시 자동 정리됩니다.
* 차단된 tmux 하위 명령은 `BLOCKED_TMUX_SUBCOMMANDS` 배열에 정의되어 있습니다.

출처: [src/tools/interactive-bash/constants.ts L1-L17](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/interactive-bash/constants.ts#L1-L17)

**slashcommand 및 skill 로딩 경로**:

* 사용자 명령: `~/.claude/commands/*.md`
* 프로젝트 명령: `./.claude/commands/*.md`, `./.opencode/command/*.md`
* 사용자 스킬: `~/.claude/skills/*/SKILL.md`
* 프로젝트 스킬: `./.claude/skills/*/SKILL.md`

출처: [src/tools/interactive-bash/constants.ts L1-L17](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/interactive-bash/constants.ts#L1-L17)

 [src/tools/look-at/constants.ts L1-L4](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/look-at/constants.ts#L1-L4)

 [src/tools/call-omo-agent/constants.ts L1-L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/constants.ts#L1-L8)

## 도구 액세스 제어 매트릭스 (Tool Access Control Matrix)

각 에이전트는 역할 및 권한 구성에 따라 서로 다른 도구 액세스 권한을 가집니다.

| 에이전트 | LSP 도구 | AST 도구 | 검색 도구 | 백그라운드 도구 | call_omo_agent | 특수 도구 |
| --- | --- | --- | --- | --- | --- | --- |
| **Sisyphus** | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ explore, librarian 전용 | ✓ 전체 |
| **explore** | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✗ 호출 불가 | ✓ 제한적 |
| **librarian** | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✗ 호출 불가 | ✓ 제한적 |
| **oracle** | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ explore, librarian 전용 | ✓ 전체 |
| **frontend-ui-ux-engineer** | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ explore, librarian 전용 | ✓ 전체 |
| **document-writer** | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✓ explore, librarian 전용 | ✓ 전체 |
| **multimodal-looker** | ✓ 전체 | ✓ 전체 | ✓ 전체 | ✗ 거부됨 | ✗ 호출 불가 | ✗ look_at 거부됨 |

**액세스 제어 규칙**:

* `call_omo_agent`는 `explore` 및 `librarian`만 유효한 에이전트 이름으로 수락합니다 (`ALLOWED_AGENTS` 상수 기준).
* `explore` 및 `librarian`은 `call_omo_agent`를 재귀적으로 호출할 수 없습니다 (무한 재귀 방지).
* `multimodal-looker`는 `background_task`, `call_omo_agent`, 또는 `look_at`을 사용할 수 없습니다 (자기 재귀 방지).
* 다른 모든 에이전트는 기본적으로 모든 도구에 액세스할 수 있습니다.
* 에이전트 구성의 `permission` 설정을 통해 추가 제한을 구성할 수 있습니다.

**도구 권한 구성**:

에이전트별 도구 제한은 에이전트 팩토리 함수에 정의되어 있으며 구성에서 재정의할 수 있습니다. 권한 시스템은 OpenCode 플러그인 레벨에서 작동합니다.

출처: [src/tools/call-omo-agent/constants.ts L1-L8](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/call-omo-agent/constants.ts#L1-L8)

 [README.md L489-L506](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L489-L506)

## 도구 구현 패턴 (Tool Implementation Patterns)

**내장 도구 등록 (Built-in Tool Registration)**:

```

```

**동적 도구 생성 (Dynamic Tool Creation)**:

```

```

**도구 실행 파이프라인 (Tool Execution Pipeline)** (모든 도구에 적용):

1. **실행 전 훅 (Pre-execution hooks)** - `tool.execute.before` 이벤트 * 도구 사용 기록 * 입력 캐싱 * 인자 유효성 검사 * 외부 PreToolUse 스크립트
2. **도구 실행 (Tool execution)** - 실제 도구 로직 실행 * 프로세스 생성 가능 (grep, bash) * API 호출 가능 (LSP) * 에이전트 위임 가능 (look_at, call_omo_agent)
3. **실행 후 훅 (Post-execution hooks)** - `tool.execute.after` 이벤트 * 도구 결과 기록 * 출력 자르기 (tool-output-truncator) * 컨텍스트 주입 (디렉토리 인젝터) * 외부 PostToolUse 스크립트
4. **에이전트 반환 (Return to agent)** - 호출한 에이전트에게 결과 전달

출처: [src/tools/index.ts L42-L68](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/tools/index.ts#L42-L68)

 상위 레벨 아키텍처의 다이어그램 5

## 도구 출력 자르기 (Tool Output Truncation)

컨텍스트 창(context window) 소진을 방지하기 위해 여러 도구에 동적 출력 자르기 기능이 적용되어 있습니다.

| 도구 | 자르기 전략 | 대상 크기 | 담당 훅 |
| --- | --- | --- | --- |
| `grep` | 일치 항목 보존을 포함한 라인 기반 자르기 | 컨텍스트 여유 공간의 50%, 최대 50k 토큰 | grep-output-truncator |
| `glob` | 파일 목록 자르기 | 컨텍스트 여유 공간의 50% | tool-output-truncator |
| `lsp_*` | 대규모 결과에 대한 응답 자르기 | 컨텍스트 여유 공간의 50% | tool-output-truncator |
| `ast_grep_search` | 일치 항목 자르기 | 컨텍스트 여유 공간의 50% | tool-output-truncator |

**자르기 알고리즘 (Truncation Algorithm)**:

1. 남은 컨텍스트 창 공간을 계산합니다.
2. 남은 공간의 50%를 목표로 설정합니다 (여유 공간 유지).
3. 최대 50k 토큰으로 제한합니다.
4. "..." 표시 및 요약과 함께 자릅니다.

자르기 기능은 구성을 통해 비활성화할 수 있습니다.

```

```

출처: [README.md L473-L475](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/README.md#L473-L475)

 상위 레벨 아키텍처의 다이어그램 3