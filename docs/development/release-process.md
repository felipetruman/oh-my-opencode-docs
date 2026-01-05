---
layout: default
title: 릴리스 프로세스
parent: "개발 (Development)"
nav_order: 4
---

# 릴리스 프로세스 (Release Process)

> **관련 소스 파일**
> * [.github/workflows/ci.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml)
> * [.github/workflows/publish.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml)
> * [script/generate-changelog.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts)
> * [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts)

이 문서는 oh-my-opencode의 새 버전을 npm에 게시하는 데 사용되는 수동 릴리스 워크플로를 설명합니다. 릴리스 프로세스에는 버전 올림(version bumping), git 커밋을 통한 변경 로그(changelog) 생성, 출처 증명(provenance)을 포함한 npm 게시, 그리고 GitHub 릴리스 생성이 포함됩니다.

모든 푸시(push) 시 실행되는 자동 CI 파이프라인에 대한 정보는 [CI/CD 파이프라인](/code-yeongyu/oh-my-opencode/11.2-keyword-modes)을 참조하십시오. 플러그인을 컴파일하는 빌드 시스템에 대한 정보는 [빌드 시스템](/code-yeongyu/oh-my-opencode/11.1-experimental-features)을 참조하십시오.

## 개요 (Overview)

릴리스 프로세스는 완전히 수동으로 진행되며 GitHub Actions의 workflow dispatch를 통해 트리거됩니다. 프로세스는 다음과 같은 선형 파이프라인을 따릅니다.

1. **사전 점검(Pre-checks)**: 테스트 및 타입 체크 실행
2. **버전 결정(Version determination)**: 최신 npm 버전을 가져와 새 버전 계산
3. **빌드(Build)**: TypeScript 컴파일 및 스키마 생성
4. **게시(Publish)**: 출처 증명과 함께 npm 레지스트리에 푸시
5. **Git 작업(Git operations)**: 버전 올림 커밋, 태그 생성, 저장소에 푸시
6. **GitHub 릴리스(GitHub release)**: 자동 생성된 변경 로그와 함께 릴리스 생성
7. **브랜치 동기화(Branch sync)**: 릴리스 태그와 일치하도록 `master` 브랜치를 강제 업데이트(force-update)

**릴리스 파이프라인 흐름**

```

```

출처: [.github/workflows/publish.yml L1-L141](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L1-L141)

 [script/publish.ts L1-L184](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L1-L184)

## 수동 릴리스 워크플로 (Manual Release Workflow)

게시 워크플로는 `.github/workflows/publish.yml`에 정의되어 있으며 GitHub의 웹 인터페이스 또는 API를 통해 트리거됩니다. 다음 두 가지 입력을 받습니다.

| 입력(Input) | 유형 | 필수 여부 | 옵션 | 설명 |
| --- | --- | --- | --- | --- |
| `bump` | choice | 예 | `major`, `minor`, `patch` | 증가시킬 시맨틱 버전(Semantic version) 구성 요소 |
| `version` | string | 아니요 | 유효한 모든 semver | 계산된 버전을 무시하고 직접 지정 (올림 로직 건너뜀) |

이 워크플로는 포크(fork)된 저장소에서 승인되지 않은 버전이 게시되는 것을 방지하기 위해 정식 저장소(`code-yeongyu/oh-my-opencode`)에서만 실행됩니다.

**워크플로 입력 및 작업 종속성**

```

```

출처: [.github/workflows/publish.yml L4-L25](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L4-L25)

 [.github/workflows/publish.yml L61-L85](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L61-L85)

### 권한 (Permissions)

워크플로에는 다음과 같은 특정 GitHub 권한이 필요합니다.

* `contents: write` - 커밋, 태그 및 릴리스 생성용
* `id-token: write` - OIDC를 통한 npm 출처 증명(provenance attestation)용

출처: [.github/workflows/publish.yml L22-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L22-L24)

## 버전 올림 전략 (Version Bumping Strategy)

버전 결정은 [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts)에 의해 처리되며 다음 로직을 따릅니다.

1. **이전 버전 가져오기**: `https://registry.npmjs.org/oh-my-opencode/latest`에서 npm 레지스트리 쿼리
2. **새 버전 계산**: 시맨틱 버전 관리(semantic versioning) 규칙을 적용하거나 직접 입력된 버전 사용
3. **존재 여부 확인**: 새 버전이 이미 npm에 존재하는지 확인
4. **package.json 업데이트**: 문자열 대체를 통해 버전 필드 수정

**버전 계산 흐름**

```

```

출처: [script/publish.ts L11-L42](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L11-L42)

 [script/publish.ts L153-L180](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L153-L180)

### 버전 올림 구현 (Version Bump Implementation)

`bumpVersion()` 함수는 시맨틱 버전 관리 규칙을 구현합니다.

```

```

| 올림 유형(Bump Type) | 예시 | 효과 |
| --- | --- | --- |
| `major` | `1.5.3` → `2.0.0` | 하위 호환성이 깨지는 변경(Breaking changes) |
| `minor` | `1.5.3` → `1.6.0` | 새로운 기능 추가 |
| `patch` | `1.5.3` → `1.5.4` | 버그 수정 |

출처: [script/publish.ts L24-L34](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L24-L34)

## 변경 로그 생성 (Changelog Generation)

변경 로그 생성은 git 히스토리에서 커밋 메시지와 기여자 정보를 추출합니다. 이 프로세스는 [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts) (릴리스용)와 [script/generate-changelog.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts) (초안 릴리스용) 모두에 구현되어 있습니다.

**변경 로그 생성 파이프라인**

```

```

출처: [script/publish.ts L44-L107](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L44-L107)

 [script/generate-changelog.ts L16-L71](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L16-L71)

### 커밋 필터링 규칙 (Commit Filtering Rules)

사용자에게 직접적인 관련이 없는 변경 사항을 제외하기 위해 접두사(prefix)를 기준으로 커밋을 필터링합니다.

| 접두사(Prefix) | 제외 여부 | 이유 |
| --- | --- | --- |
| `ignore:` | ✓ | 내부 메모 |
| `test:` | ✓ | 테스트 코드 변경 |
| `chore:` | ✓ | 유지보수 작업 |
| `ci:` | ✓ | CI 설정 |
| `release:` | ✓ | 자동 릴리스 커밋 |

출처: [script/publish.ts L51](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L51-L51)

 [script/generate-changelog.ts L23](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L23-L23)

### 기여자 기여 표시 (Contributor Attribution)

기여자는 GitHub API를 통해 식별되며 팀 목록과 대조하여 필터링됩니다.

```

```

최종 변경 로그에는 외부 기여자의 사용자 이름과 커밋 제목이 포함된 감사 섹션이 포함됩니다.

출처: [script/publish.ts L68-L107](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L68-L107)

 [script/generate-changelog.ts L5-L71](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L5-L71)

## npm 게시 (npm Publishing)

플러그인은 출처 증명(provenance attestation)과 함께 npm 레지스트리에 게시됩니다. 게시 단계에는 중복 버전 방지를 위한 유효성 검사가 포함되며 빌드 결과물이 존재하는지 확인합니다.

**npm 게시 워크플로**

```

```

출처: [.github/workflows/publish.yml L104-L127](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L104-L127)

 [script/publish.ts L109-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L109-L117)

### 출처 증명 (Provenance Attestation)

CI에서 실행될 때, 게시 단계에는 패키지를 소스 코드 및 빌드 프로세스와 연결하는 OIDC 서명 증명을 생성하기 위해 `--provenance` 플래그가 포함됩니다.

```

```

이를 위해 다음 사항이 필요합니다.

* npm 버전 9.5.0 이상 (워크플로에서 업그레이드됨: [.github/workflows/publish.yml L80-L81](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L80-L81) )
* `id-token: write` 권한 ([.github/workflows/publish.yml L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L24-L24) )
* GitHub OIDC 제공자 설정

`--ignore-scripts` 플래그는 워크플로에서 이미 빌드가 실행되었으므로 `prepublishOnly`가 빌드를 다시 실행하는 것을 방지합니다.

출처: [.github/workflows/publish.yml L119-L127](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L119-L127)

 [script/publish.ts L109-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L109-L117)

## Git 작업 및 GitHub 릴리스 (Git Operations and GitHub Releases)

npm 게시가 성공적으로 완료되면, 릴리스 스크립트는 git 작업을 수행하고 GitHub 릴리스를 생성합니다.

**Git 및 릴리스 작업**

```

```

출처: [script/publish.ts L119-L151](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L119-L151)

 [.github/workflows/publish.yml L128-L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L140)

### 태그 및 릴리스 생성 (Tag and Release Creation)

스크립트는 주석이 달린(annotated) git 태그와 그에 해당하는 GitHub 릴리스를 생성합니다.

```

```

태그나 릴리스가 이미 존재하는 경우(예: 재시도 시), 멱등성(idempotency)을 유지하기 위해 해당 작업은 건너뜁니다.

출처: [script/publish.ts L133-L150](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L133-L150)

### Master 브랜치 동기화 (Master Branch Synchronization)

릴리스 생성 후, `master` 브랜치는 릴리스 태그와 일치하도록 강제 업데이트됩니다.

```

```

이를 통해 `master` 브랜치는 항상 최신 안정 릴리스를 가리키게 되며, 활발한 개발은 `dev` 브랜치에서 계속됩니다.

출처: [.github/workflows/publish.yml L133-L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L133-L140)

## 초안 릴리스 (Draft Releases)

CI 워크플로는 향후 변경 사항을 미리 볼 수 있도록 `dev` 브랜치에서 초안(draft) 릴리스를 자동으로 생성하거나 업데이트합니다. 이 초안은 실제 릴리스가 게시될 때 삭제됩니다.

**초안 릴리스 생명주기**

```

```

출처: [.github/workflows/ci.yml L88-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L88-L134)

 [script/generate-changelog.ts L1-L93](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts#L1-L93)

### 초안 릴리스 태그 (Draft Release Tag)

초안 릴리스는 버전에 기반하지 않은 특수 태그 이름인 `next`를 사용합니다. 이를 통해 여러 개의 초안 릴리스를 만들지 않고도 지속적인 업데이트가 가능합니다.

| 속성 | 값 |
| --- | --- |
| 태그 이름 | `next` |
| 제목 | "Upcoming Changes 🍿" |
| 상태 | 초안 (대중에게 보이지 않음) |
| 대상 | 현재 `dev` 브랜치의 SHA |

실제 릴리스가 게시되면 초안은 다음을 통해 삭제됩니다: [.github/workflows/publish.yml L128-L131](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L131)

출처: [.github/workflows/ci.yml L115-L132](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L115-L132)

 [.github/workflows/publish.yml L128-L131](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L128-L131)

## 환경 변수 (Environment Variables)

게시 스크립트와 워크플로는 여러 환경 변수에 의존합니다.

| 변수 | 소스 | 목적 | 사용처 |
| --- | --- | --- | --- |
| `BUMP` | 워크플로 입력 | 버전 올림 유형 (`major`/`minor`/`patch`) | [script/publish.ts L6](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L6-L6) |
| `VERSION` | 워크플로 입력 | 버전 직접 지정 | [script/publish.ts L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L7-L7) |
| `CI` | GitHub Actions | 출처 증명 및 git 작업 활성화 | [script/publish.ts L112-L120](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L112-L120) |
| `GITHUB_TOKEN` | GitHub Actions | gh CLI 인증 | [.github/workflows/publish.yml L125](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L125-L125) |
| `NPM_CONFIG_PROVENANCE` | 워크플로 환경 변수 | npm 출처 증명 활성화 | [.github/workflows/publish.yml L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L126-L126) |
| `GH_TOKEN` | GitHub Actions | GitHub API 호출 인증 | [.github/workflows/ci.yml L113-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L113-L134) |

출처: [script/publish.ts L6-L7](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L6-L7)

 [.github/workflows/publish.yml L121-L126](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L121-L126)

 [.github/workflows/ci.yml L113-L134](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L113-L134)

## 빌드 검증 (Build Verification)

게시하기 전에 워크플로는 빌드 결과물(artifacts)이 존재하는지 확인합니다.

```

```

이를 통해 불완전한 패키지를 게시하기 전에 빌드 실패를 조기에 발견할 수 있습니다. 검증은 빌드 단계 직후와 게시 스크립트 실행 전에 수행됩니다.

출처: [.github/workflows/publish.yml L114-L117](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L114-L117)

## 동시성 제어 (Concurrency Control)

릴리스 워크플로는 동시 릴리스를 방지하기 위해 동시성 그룹(concurrency groups)을 사용합니다.

```

```

이를 통해 여러 릴리스가 동시에 트리거되는 경우 하나만 실행되고 나머지는 대기하게 됩니다. 이는 npm 레지스트리에서의 버전 충돌 및 경합 상태(race conditions)를 방지합니다.

출처: [.github/workflows/publish.yml L20](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L20-L20)