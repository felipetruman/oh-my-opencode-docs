---
layout: default
title: CI/CD 파이프라인
parent: 개발
nav_order: 1
---

# CI/CD 파이프라인

> **관련 소스 파일**
> * [.github/workflows/ci.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml)
> * [.github/workflows/publish.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml)
> * [script/generate-changelog.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts)
> * [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts)

## 목적 및 범위

이 문서는 GitHub Actions를 통해 구현된 지속적 통합(CI) 및 배포(CD) 워크플로우에 대해 설명합니다. 자동화된 테스트, 타입 체크, 빌드 검증 및 릴리스 게시 프로세스를 다룹니다. 로컬 빌드 시스템 및 도구에 대한 정보는 [Build System](/code-yeongyu/oh-my-opencode/11.1-experimental-features)을 참조하십시오. 수동 릴리스 프로세스 및 버전 관리 전략은 [Release Process](/code-yeongyu/oh-my-opencode/11.3-model-configuration)를 참조하십시오.

CI/CD 시스템은 두 가지 주요 워크플로우로 구성됩니다:

* **CI 워크플로우** - 모든 푸시(Push) 및 풀 리퀘스트(Pull Request)에 대한 자동화된 품질 검사
* **Publish 워크플로우** - npm 게시를 위한 수동 릴리스 워크플로우

---

## 개요

### CI/CD 파이프라인 아키텍처

```

```

**소스:** [.github/workflows/ci.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml)

 [.github/workflows/publish.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml)

---

## CI 워크플로우

CI 워크플로우는 모든 코드 변경 사항에 대해 자동화된 품질 보증(QA)을 제공합니다.

### 트리거 조건

워크플로우는 다음 경우에 활성화됩니다:

* `master` 또는 `dev` 브랜치로의 푸시 이벤트 [.github/workflows/ci.yml L4-L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L4-L7)
* `master` 브랜치를 대상으로 하는 풀 리퀘스트 [.github/workflows/ci.yml L6-L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L6-L7)

동시성 제어(Concurrency control)는 브랜치당 여러 실행이 동시에 이루어지는 것을 방지하며, 새로운 커밋이 들어오면 진행 중인 실행을 자동으로 취소합니다. [.github/workflows/ci.yml L9-L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L9-L11)

### 작업 의존성 및 실행

```

```

**소스:** [.github/workflows/ci.yml L14-L74](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L14-L74)

### Test 작업

Bun을 사용하여 테스트 스위트를 실행합니다:

| 단계 | 명령 | 목적 |
| --- | --- | --- |
| Bun 설정 | `oven-sh/setup-bun@v2` | 최신 Bun 런타임 설치 |
| 의존성 설치 | `bun install` | `@ast-grep/napi`가 허용된 상태로 패키지 설치 |
| 테스트 실행 | `bun test` | 테스트 스위트 실행 |

`BUN_INSTALL_ALLOW_SCRIPTS` 환경 변수는 `@ast-grep/napi`에 대해서만 설치 스크립트를 허용합니다. [.github/workflows/ci.yml L25-L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L25-L26)

**소스:** [.github/workflows/ci.yml L14-L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L14-L29)

### Typecheck 작업

TypeScript 타입의 정확성을 검증합니다:

| 단계 | 명령 | 목적 |
| --- | --- | --- |
| Bun 설정 | `oven-sh/setup-bun@v2` | 최신 Bun 런타임 설치 |
| 의존성 설치 | `bun install` | 패키지 설치 |
| 타입 체크 | `bun run typecheck` | 체크 전용 모드로 TypeScript 컴파일러 실행 |

**소스:** [.github/workflows/ci.yml L31-L46](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L31-L46)

### Build 작업

전체 빌드를 수행하고 출력 아티팩트(Artifact)를 검증합니다. [.github/workflows/ci.yml L48-L74](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L48-L74)

:

1. **의존성:** `test` 및 `typecheck` 작업이 모두 성공적으로 완료되어야 합니다. [.github/workflows/ci.yml L50](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L50-L50)
2. **빌드 실행:** 빌드 스크립트를 호출하는 `bun run build`를 실행합니다. [.github/workflows/ci.yml L68](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L68-L68)
3. **출력 검증:** 필수 아티팩트를 확인합니다: * `dist/index.js` - 번들링된 JavaScript 출력물 * `dist/index.d.ts` - TypeScript 선언 파일

**소스:** [.github/workflows/ci.yml L48-L74](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L48-L74)

### 스키마 자동 커밋 (Schema Auto-Commit)

`master` 브랜치로 푸시할 때, 워크플로우는 스키마 변경 사항을 자동으로 커밋합니다. [.github/workflows/ci.yml L75-L86](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L75-L86)

:

```

```

이를 통해 특히 Zod 스키마가 수정될 때 `assets/oh-my-opencode.schema.json`이 코드 변경 사항과 동기화된 상태를 유지하도록 합니다. 커밋은 `github-actions[bot]` ID를 사용합니다. [.github/workflows/ci.yml L81-L82](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L81-L82)

**소스:** [.github/workflows/ci.yml L75-L86](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L75-L86)

### 초안 릴리스 생성 (Draft Release Creation)

`dev` 브랜치로 푸시할 때, 워크플로우는 초안 릴리스(Draft release)를 생성하거나 업데이트합니다. [.github/workflows/ci.yml L88-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L88-L134)

:

1. **변경 로그 생성:** 릴리스 노트를 작성하기 위해 `script/generate-changelog.ts`를 실행합니다.
2. **초안 관리:** `next`로 태그된 새 초안 릴리스를 생성하거나 기존 초안을 업데이트합니다.
3. **릴리스 노트:** 커밋 히스토리와 기여자 정보가 포함됩니다.

초안 릴리스는 정식 릴리스 프로세스 전에 예정된 변경 사항의 미리보기를 제공합니다.

**소스:** [.github/workflows/ci.yml L88-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L88-L134)

 [script/generate-changelog.ts L16-L35](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L16-L35)

---

## Publish 워크플로우

Publish 워크플로우는 전체 버전 관리, 변경 로그 생성 및 GitHub 릴리스 생성을 포함하여 npm으로의 수동 릴리스를 처리합니다.

### 워크플로우 입력값

`workflow_dispatch`를 통한 수동 실행 시 두 가지 입력값을 사용합니다. [.github/workflows/publish.yml L5-L18](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L5-L18)

:

| 입력값 | 타입 | 필수 여부 | 옵션 | 설명 |
| --- | --- | --- | --- | --- |
| `bump` | choice | 예 | `major`, `minor`, `patch` | 증가시킬 시맨틱 버전(Semantic version) 구성 요소 |
| `version` | string | 아니요 | - | 버전 재정의 (버전 올림 계산을 건너뜀) |

### 작업 파이프라인

```

```

**소스:** [.github/workflows/publish.yml L26-L141](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L26-L141)

### 빌드 프로세스

Publish 워크플로우는 `bun run build`를 사용하는 대신 수동 빌드를 수행합니다. [.github/workflows/publish.yml L104-L112](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L104-L112)

:

```

```

이 명시적 빌드 명령은 다음 사항을 보장합니다:

* ESM 형식 출력
* `@ast-grep/napi` 바이너리 의존성의 외부 처리
* 선언 파일(Declaration file) 생성
* JSON 스키마 재생성

**소스:** [.github/workflows/publish.yml L104-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L104-L117)

### 게시 스크립트 통합 (Publish Script Integration)

워크플로우는 환경 변수와 함께 `script/publish.ts`를 호출합니다. [.github/workflows/publish.yml L119-L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L119-L126)

:

| 환경 변수 | 목적 |
| --- | --- |
| `BUMP` | 시맨틱 버전 구성 요소 (`major`, `minor`, `patch`) |
| `VERSION` | 선택적 버전 재정의 |
| `CI` | CI 전용 동작 활성화 (출처 증명, git 작업) |
| `GITHUB_TOKEN` | GitHub API 작업을 위한 인증 |
| `NPM_CONFIG_PROVENANCE` | 공급망 보안을 위한 npm 출처(Provenance) 활성화 |

**소스:** [.github/workflows/publish.yml L119-L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L119-L126)

### 게시 후 작업 (Post-Publish Operations)

npm 게시가 성공하면 워크플로우는 정리 및 브랜치 동기화를 수행합니다. [.github/workflows/publish.yml L128-L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L140)

:

1. **초안 릴리스 삭제:** CI 워크플로우에서 생성된 `next` 초안 릴리스를 제거합니다.
2. **Master 브랜치 동기화:** `master` 브랜치를 새로 태그된 버전으로 강제 리셋(Force-reset)합니다.

```

```

**소스:** [.github/workflows/publish.yml L128-L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L140)

---

## 지원 스크립트

### publish.ts

`script/publish.ts` 스크립트는 전체 게시 워크플로우를 조율합니다. [script/publish.ts L1-L183](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L1-L183)

#### 버전 관리

```

```

`bumpVersion()` 함수는 시맨틱 버전 관리 로직을 구현합니다. [script/publish.ts L24-L34](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L24-L34)

:

* `major`: 메이저(Major) 버전을 증가시키고, 마이너(Minor) 및 패치(Patch)를 0으로 리셋합니다.
* `minor`: 마이너 버전을 증가시키고, 패치를 0으로 리셋합니다.
* `patch`: 패치 버전을 증가시킵니다.

**소스:** [script/publish.ts L11-L42](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L11-L42)

 [script/publish.ts L153-L170](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L153-L170)

#### 변경 로그 생성

스크립트는 git 히스토리를 분석하여 릴리스 노트를 생성합니다. [script/publish.ts L44-L66](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L44-L66)

:

```

```

커밋 필터링 규칙 [script/publish.ts L51](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L51-L51)

:

* **제외된 접두사:** `ignore:`, `test:`, `chore:`, `ci:`, `release:`
* **포함됨:** 커밋 해시와 메시지가 있는 다른 모든 커밋

**소스:** [script/publish.ts L44-L66](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L44-L66)

#### 기여자 정보 (Contributor Attribution)

`getContributors()` 함수는 외부 기여자를 식별하기 위해 GitHub API를 쿼리합니다. [script/publish.ts L68-L107](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L68-L107)

:

```

```

**소스:** [script/publish.ts L68-L107](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L68-L107)

#### NPM 게시

`buildAndPublish()` 함수는 출처(Provenance) 지원과 함께 npm에 게시합니다. [script/publish.ts L109-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L109-L117)

:

```

```

`--ignore-scripts` 플래그는 워크플로우에서 이미 빌드가 완료되었으므로 `prepublishOnly`가 다시 실행되는 것을 방지합니다. `--provenance` 플래그는 OIDC를 사용하여 GitHub Actions에서 실행될 때 npm의 공급망 증명을 활성화합니다.

**소스:** [script/publish.ts L109-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L109-L117)

#### Git 작업

`gitTagAndRelease()` 함수는 버전 관리 작업을 처리합니다. [script/publish.ts L119-L151](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L119-L151)

:

| 작업 | 명령 | 목적 |
| --- | --- | --- |
| git 설정 | `git config user.email/name` | 봇 ID 설정 |
| 버전 커밋 | `git commit -m "release: v$VERSION"` | 버전 올림 기록 |
| 태그 생성 | `git tag v$VERSION` | 릴리스 커밋 태깅 |
| 변경 사항 푸시 | `git push origin HEAD --tags` | 원격 저장소와 동기화 |
| GitHub 릴리스 | `gh release create v$VERSION` | 공개 릴리스 생성 |

**소스:** [script/publish.ts L119-L151](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L119-L151)

### generate-changelog.ts

`script/generate-changelog.ts` 스크립트는 초안 릴리스를 위한 변경 로그 내용을 생성합니다. [script/generate-changelog.ts L1-L92](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L1-L92)

#### 최신 릴리스 감지

```

```

스크립트는 필터와 함께 `gh release list`를 사용합니다. [script/generate-changelog.ts L7-L13](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L7-L13)

:

* `--exclude-drafts` - 초안 릴리스 무시
* `--exclude-pre-releases` - 프리 릴리스(Pre-release) 무시
* `--limit 1` - 가장 최근의 릴리스만 가져옴

**소스:** [script/generate-changelog.ts L7-L14](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L7-L14)

#### 커밋 분석

동일한 제외 패턴을 사용하는 `publish.ts` 커밋 필터링과 동일합니다. [script/generate-changelog.ts L16-L35](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L16-L35)

:

```

```

**소스:** [script/generate-changelog.ts L16-L35](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L16-L35)

#### 기여자 감지

GitHub API를 사용하여 커밋 범위를 비교하고 기여자 정보를 추출합니다. [script/generate-changelog.ts L37-L71](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L37-L71)

:

```

```

기여자는 사용자 이름별로 그룹화되며 그 아래에 커밋 제목이 나열됩니다. 팀 구성원(`actions-user`, `github-actions[bot]`, `code-yeongyu`)은 제외됩니다. [script/generate-changelog.ts L5](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L5-L5)

**소스:** [script/generate-changelog.ts L37-L71](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L37-L71)

---

## 동시성 및 권한

### 동시성 제어 (Concurrency Control)

두 워크플로우 모두 동시성 관리를 구현합니다. [.github/workflows/ci.yml L9-L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L9-L11)

 [.github/workflows/publish.yml L20](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L20-L20)

:

| 워크플로우 | 그룹 | 동작 |
| --- | --- | --- |
| CI | `${{ github.workflow }}-${{ github.ref }}` | 새 푸시 발생 시 진행 중인 실행 취소 |
| Publish | `${{ github.workflow }}-${{ github.ref }}` | 순차적 실행 대기열 생성 |

이는 여러 커밋이 빠르게 푸시될 때 경합 상태(Race condition)와 리소스 낭비를 방지합니다.

### 필수 권한

#### CI 워크플로우 권한

[.github/workflows/ci.yml L51-L52](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L51-L52)

 [.github/workflows/ci.yml L92-L93](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L92-L93)

| 권한 | 범위 | 목적 |
| --- | --- | --- |
| `contents: write` | `build` 작업 | 스키마 변경 사항 자동 커밋 |
| `contents: write` | `draft-release` 작업 | 초안 릴리스 생성/업데이트 |

#### Publish 워크플로우 권한

[.github/workflows/publish.yml L22-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L22-L24)

| 권한 | 범위 | 목적 |
| --- | --- | --- |
| `contents: write` | 모든 작업 | Git 태그 생성, 릴리스 게시 |
| `id-token: write` | `publish` 작업 | npm 출처(Provenance)를 위한 OIDC 인증 |

`id-token: write` 권한은 GitHub의 OIDC 제공자가 npm의 출처 검증을 위한 단기 토큰을 생성할 수 있게 하여, 패키지 빌드 원본에 대한 암호화된 증명을 제공합니다.

**소스:** [.github/workflows/ci.yml L9-L11](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L9-L11)

 [.github/workflows/ci.yml L51-L52](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L51-L52)

 [.github/workflows/ci.yml L92-L93](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L92-L93)

 [.github/workflows/publish.yml L20-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L20-L24)

---

## 환경 구성

### Node.js 및 Bun 설정

모든 워크플로우는 일관된 런타임 버전을 사용합니다:

| 런타임 | 버전 | 목적 |
| --- | --- | --- |
| Bun | `latest` | 빌드 및 테스트 실행 |
| Node.js | `24` | npm 게시 (Publish 워크플로우 전용) |

Publish 워크플로우는 OIDC 신뢰 게시(Trusted publishing) 지원을 위해 npm을 최신 버전으로 업그레이드합니다. [.github/workflows/publish.yml L80-L81](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L80-L81)

**소스:** [.github/workflows/ci.yml L19-L21](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L19-L21)

 [.github/workflows/publish.yml L32-L34](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L32-L34)

 [.github/workflows/publish.yml L72-L81](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L72-L81)

### 의존성 설치

설치 시 post-install 스크립트에 대해 제한된 허용 목록(Allow-list)을 사용합니다:

```

```

이 보안 조치는 설치 중 임의의 코드 실행을 방지하며, 네이티브 컴파일이 필요한 `@ast-grep/napi` 패키지만 허용합니다.

**소스:** [.github/workflows/ci.yml L24-L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L24-L26)

 [.github/workflows/publish.yml L36-L39](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L36-L39)

 [.github/workflows/publish.yml L86-L89](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L86-L89)

### 저장소 조건

Publish 워크플로우에는 저장소 확인 단계가 포함되어 있습니다. [.github/workflows/publish.yml L64](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L64-L64)

:

```

```

이는 포크(Fork)된 저장소에서 실수로 게시되는 것을 방지하여, 정식 저장소만 npm에 게시할 수 있도록 보장합니다.

**소스:** [.github/workflows/publish.yml L61-L64](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L61-L64)