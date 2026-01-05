---
layout: default
title: Build System
parent: "개발 (Development)"
nav_order: 2
---

# Build System (빌드 시스템)

> **관련 소스 파일**
> * [.github/workflows/ci.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml)
> * [.github/workflows/publish.yml](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml)
> * [bun.lock](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/bun.lock)
> * [package.json](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json)
> * [script/generate-changelog.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/generate-changelog.ts)
> * [script/publish.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts)
> * [src/cli/config-manager.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts)
> * [src/shared/jsonc-parser.test.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.test.ts)
> * [src/shared/jsonc-parser.ts](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/shared/jsonc-parser.ts)

빌드 시스템은 `src/`에 있는 TypeScript 소스 코드를 `dist/`에 배포 가능한 JavaScript 아티팩트(artifact)로 변환하고, TypeScript 선언 파일(declaration files)을 생성하며, 설정 검증을 위한 JSON 스키마를 생성합니다. 이 페이지에서는 빌드 스크립트, 컴파일 프로세스, 출력 아티팩트 및 스키마 생성에 대해 다룹니다.

CI/CD 파이프라인 통합에 대한 정보는 [CI/CD Pipeline](/code-yeongyu/oh-my-opencode/12.2-cicd-pipeline)을 참조하십시오. 릴리스 프로세스 및 npm 배포에 대해서는 [Release Process](/code-yeongyu/oh-my-opencode/12.3-release-process)를 참조하십시오.

---

## Build Overview (빌드 개요)

빌드 시스템은 패키지 관리자이자 JavaScript 런타임으로 Bun을 사용하며, 타입 체크 및 선언 파일 생성을 위해 TypeScript를 사용합니다. 빌드 프로세스는 세 개의 주요 엔트리 포인트(entry point)와 하나의 JSON 스키마를 생성합니다.

**Build System Architecture (빌드 시스템 아키텍처)**

```

```

Sources: [package.json L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L26)

---

## Build Scripts (빌드 스크립트)

`package.json`은 빌드 및 검증 작업을 위한 5개의 npm 스크립트를 정의합니다.

| 스크립트 | 명령어 | 목적 |
| --- | --- | --- |
| `build` | 복합 스크립트 | 모든 빌드 단계를 순차적으로 실행 |
| `build:schema` | `bun run script/build-schema.ts` | TypeScript 타입에서 JSON 스키마 생성 |
| `clean` | `rm -rf dist` | 빌드 아티팩트 제거 |
| `prepublishOnly` | `bun run clean && bun run build` | 배포 전 자동 정리 및 재빌드 |
| `typecheck` | `tsc --noEmit` | 파일 생성 없이 타입 체크 수행 |
| `test` | `bun test` | 테스트 스위트 실행 |

### Main Build Script (메인 빌드 스크립트)

`build` 스크립트는 네 가지 순차적 작업을 실행합니다.

```

```

**Build Script Breakdown (빌드 스크립트 상세 분석):**

1. **Main bundle compilation (메인 번들 컴파일)**: `bun build src/index.ts src/google-auth.ts --outdir dist --target bun --format esm --external @ast-grep/napi` * 플러그인 엔트리 및 OAuth 서버 컴파일 * 대상: Bun 런타임 * 형식: ES 모듈 * 외부(External): `@ast-grep/napi` (네이티브 애드온으로, 번들링할 수 없음)
2. **Declaration generation (선언 파일 생성)**: `tsc --emitDeclarationOnly` * TypeScript 사용자를 위한 `.d.ts` 파일 생성 * `tsconfig.json`에 의해 제어됨
3. **CLI compilation (CLI 컴파일)**: `bun build src/cli/index.ts --outdir dist/cli --target bun --format esm` * CLI 실행 파일을 위한 별도 번들 * 선언 파일 불필요 (라이브러리로 노출되지 않음)
4. **Schema generation (스키마 생성)**: `bun run build:schema` * `dist/oh-my-opencode.schema.json` 생성 * [Schema Generation](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/Schema Generation) 섹션을 참조하십시오.

Sources: [package.json L26-L27](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L27)

---

## Build Process Flow (빌드 프로세스 흐름)

**Complete Build Pipeline (전체 빌드 파이프라인)**

```

```

Sources: [package.json L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L26)

 [.github/workflows/publish.yml L104-L113](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L104-L113)

---

## Output Artifacts (출력 아티팩트)

빌드 프로세스는 `dist/` 디렉토리에 다음과 같은 아티팩트를 생성합니다.

### Published Files (배포되는 파일)

`package.json`의 `files` 필드는 `dist/` 디렉토리만 npm 패키지에 포함되도록 지정합니다.

```

```

### Entry Points (엔트리 포인트)

`package.json`은 세 개의 모듈 내보내기(exports)를 정의합니다.

| 내보내기 | 타입 | 가져오기 | 설명 |
| --- | --- | --- | --- |
| `.` | `./dist/index.d.ts` | `./dist/index.js` | 메인 플러그인 엔트리 |
| `./google-auth` | `./dist/google-auth.d.ts` | `./dist/google-auth.js` | Gemini용 OAuth 서버 |
| `./schema.json` | N/A | `./dist/oh-my-opencode.schema.json` | 설정 스키마 |

**Entry Point Configuration (엔트리 포인트 설정):**

```

```

### CLI Binary (CLI 바이너리)

CLI는 `bin` 필드를 통해 실행 파일로 노출됩니다.

```

```

이를 통해 사용자는 설치 후 `bunx oh-my-opencode` 또는 `npx oh-my-opencode`를 실행할 수 있습니다.

**Distribution Artifact Structure (배포 아티팩트 구조):**

```

```

Sources: [package.json L11-L24](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L11-L24)

 [package.json L8-L10](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L8-L10)

---

## Schema Generation (스키마 생성)

`build:schema` 스크립트는 코드베이스에 정의된 Zod 검증 스키마로부터 JSON 스키마 문서를 생성합니다. 이 스키마는 IDE의 자동 완성 및 설정 파일 검증에 사용됩니다.

### Schema Build Process (스키마 빌드 프로세스)

**Schema Generation Flow (스키마 생성 흐름):**

```

```

스키마는 두 위치에 기록됩니다.

* **`dist/oh-my-opencode.schema.json`**: 런타임 접근을 위해 npm 패키지에 포함됨
* **`assets/oh-my-opencode.schema.json`**: GitHub 호스팅 스키마 URL을 위해 git 저장소에 커밋됨

### Schema URL in Configurations (설정 내 스키마 URL)

생성된 설정은 `$schema` 속성을 통해 스키마를 참조합니다.

```

```

이를 통해 `oh-my-opencode.json` 파일을 편집할 때 자동 완성 및 검증과 같은 IDE 기능을 사용할 수 있습니다.

Sources: [package.json L27](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L27-L27)

 [src/cli/config-manager.ts L140](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/src/cli/config-manager.ts#L140-L140)

---

## TypeScript Configuration (TypeScript 설정)

빌드 프로세스는 타입 체크 및 선언 파일 생성을 위해 `tsconfig.json`에 의존합니다. 주요 컴파일러 옵션은 다음과 같습니다.

* **`declaration: true`**: `.d.ts` 파일 생성
* **`emitDeclarationOnly: true`** (`tsc --emitDeclarationOnly` 사용 시): JavaScript가 아닌 선언 파일만 출력
* **`outDir: "dist"`**: 컴파일된 파일의 출력 디렉토리
* **`target: "ES2022"`**: 현대적인 JavaScript 기능 사용
* **`module: "ES2022"`**: ES 모듈 시스템 사용
* **`moduleResolution: "bundler"`**: 번들러 호환 모듈 해석

`typecheck` 스크립트의 `tsc --noEmit` 명령어는 출력 파일 생성 없이 타입의 유효성을 검사하며, CI에서 검증 용도로 사용됩니다.

Sources: [package.json L30-L31](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L30-L31)

---

## External Dependencies (외부 종속성)

### @ast-grep/napi

`@ast-grep/napi` 패키지는 AST 파싱 기능을 제공하는 네이티브 Node.js 애드온입니다. 빌드 프로세스에서 특별한 처리가 필요합니다.

1. **Build exclusion (빌드 제외)**: 번들링을 방지하기 위해 Bun 빌드 명령어에서 `--external`로 표시됨
2. **Trusted dependency (신뢰할 수 있는 종속성)**: 설치 스크립트를 허용하기 위해 `trustedDependencies`에 나열됨
3. **Install-time compilation (설치 시 컴파일)**: `bun install` 중에 패키지가 네이티브 코드를 컴파일함

**Install Command with Trusted Dependencies (신뢰할 수 있는 종속성을 포함한 설치 명령어):**

```

```

CI 환경 변수:

```

```

이는 `@ast-grep/napi`만 설치 스크립트를 실행하도록 명시적으로 허용하여, 다른 종속성으로부터의 임의 코드 실행을 방지합니다.

Sources: [package.json L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L26-L26)

 [package.json L73-L77](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L73-L77)

 [.github/workflows/ci.yml L24-L26](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L24-L26)

---

## Build Verification (빌드 검증)

CI 워크플로우에는 빌드 출력이 올바르게 생성되었는지 확인하는 검증 단계가 포함되어 있습니다.

### Verification Steps in CI (CI 검증 단계)

```

```

이 체크 항목들은 다음을 보장합니다.

* 메인 플러그인 JavaScript 번들 존재 여부
* TypeScript 선언 파일 생성 여부
* CLI 실행 파일 생성 여부

**Build Verification Flow (빌드 검증 흐름):**

```

```

배포 워크플로우에는 배포 전 빌드 환경을 디버깅하기 위한 추가 검증 단계가 포함되어 있습니다.

Sources: [.github/workflows/ci.yml L70-L73](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/ci.yml#L70-L73)

 [.github/workflows/publish.yml L115-L122](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L115-L122)

---

## Pre-Publish Hook (배포 전 후크)

`prepublishOnly` 스크립트는 npm에 배포하기 전에 깨끗한 빌드를 보장합니다.

```

```

이 라이프사이클 후크는 다음을 수행합니다.

1. `bun run clean`으로 이전의 모든 빌드 아티팩트 제거
2. `bun run build`로 새로 빌드 수행
3. `npm publish` (또는 `bun publish`) 전에 자동으로 실행

하지만 CI 배포 워크플로우에서는 이미 검증된 빌드 후 다시 빌드하는 것을 피하기 위해 `--ignore-scripts`를 사용하여 이 단계를 건너뜁니다.

```

```

Sources: [package.json L29](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/package.json#L29-L29)

 [script/publish.ts L113](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/script/publish.ts#L113-L113)

---

## Debug Build Information (빌드 정보 디버깅)

배포 워크플로우에는 빌드 문제를 진단하는 데 도움이 되는 디버그 출력이 포함되어 있습니다.

```

```

빌드가 완료된 후 아티팩트 내용이 확인됩니다.

```

```

이 진단 정보는 빌드 실패 시 GitHub Actions 로그에 표시되어 환경 또는 설정 문제를 식별하는 데 도움을 줍니다.

Sources: [.github/workflows/publish.yml L91-L102](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L91-L102)

 [.github/workflows/publish.yml L115-L122](https://github.com/code-yeongyu/oh-my-opencode/blob/b92cd6ab/.github/workflows/publish.yml#L115-L122)

---

## Summary (요약)

빌드 시스템은 4단계 프로세스를 통해 TypeScript 소스를 배포 가능한 아티팩트로 변환합니다.

1. **Bundle compilation (번들 컴파일)**: Bun을 이용한 컴파일 (Bun 런타임 대상, ESM 형식)
2. **Declaration generation (선언 파일 생성)**: TypeScript 컴파일러를 이용한 생성
3. **CLI bundling (CLI 번들링)**: 별도 실행 파일로 번들링
4. **Schema generation (스키마 생성)**: Zod 스키마로부터 생성

시스템은 명시적 제외 및 신뢰할 수 있는 종속성 설정을 통해 네이티브 종속성(@ast-grep/napi)을 처리하고, 검증 체크를 통해 빌드 정확성을 보장하며, 배포 전 후크를 통해 깨끗한 빌드 상태를 유지합니다.

모든 빌드 출력물은 `dist/`에 기록되어 npm에 배포되며, 스키마는 GitHub 호스팅 URL 접근을 위해 `assets/`에 추가로 복사됩니다.