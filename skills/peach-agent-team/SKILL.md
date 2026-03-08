---
name: peach-agent-team
description: Use when orchestrating PeachSolution module delivery across backend, store, UI, and QA, especially when replacing separate backend-only, UI-only, or fullstack team workflows.
---

# Peach Agent Team

## Overview

PeachSolution 아키텍처에서 신규 기능 개발을 조율하는 통합 팀 스킬입니다.

기존 `team-backend`, `team-ui`, `team-fullstack`를 하나로 통합하며, 역할별 페르소나와 체크리스트를 이 문서 안에 포함합니다.

## Modes

| mode | 용도 | 포함 역할 |
| --- | --- | --- |
| `backend` | 기존 UI에 API + Store 연결 | backend-dev, backend-qa, store-dev, frontend-qa |
| `ui` | Store 기반 UI만 구현 | ui-dev, frontend-qa |
| `fullstack` | DB 스키마 기반 전체 생성 | backend-dev, backend-qa, store-dev, ui-dev, frontend-qa |

## Preconditions

- DB 스키마가 필요한 모드(`backend`, `fullstack`)에서는 `api/db/schema/[도메인]/[테이블].sql`이 존재해야 합니다.
- `ui` 모드에서는 `front/src/modules/[모듈명]/store/[모듈명].store.ts`가 존재해야 합니다.
- Store가 없으면 먼저 `peach-gen-store`, UI가 없으면 `peach-gen-ui`를 기준으로 생성합니다.

## Inputs

```bash
/peach-agent-team [모듈명] mode=backend|ui|fullstack [옵션]

# 공통 옵션
# figma=[URL]
# ui=crud|page|two-depth|infinite-scroll|select-list
# file=Y
# excel=Y
# storeTdd=Y
```

## Orchestration

### 1. 환경 확인

```bash
# 스키마 / 타입 / 가이드 코드 확인
ls api/db/schema/
head -5 api/src/modules/test-data/dao/test-data.dao.ts
head -3 api/src/modules/test-data/controller/test-data.controller.ts
ls front/src/modules/test-data/
```

### 2. 역할 구성

`mode=backend`
- backend-dev → backend-qa
- backend-dev → store-dev → frontend-qa

`mode=ui`
- ui-dev → frontend-qa

`mode=fullstack`
- backend-dev → backend-qa
- backend-dev → store-dev → ui-dev → frontend-qa

### 3. 역할별 지시

#### backend-dev
- `peach-gen-backend` 기준으로 API 코드를 생성합니다.
- Koa/Elysia 모드를 감지합니다.
- 완료 기준: `bun test`, `bun run lint:fixed`, `bun run build` 통과
- 산출물: API 파일 목록, 엔드포인트 스펙, 테스트 결과

#### backend-qa
- Backend 구조, 패턴, TDD, 빌드, 엔드포인트를 독립 검증합니다.
- 확인 항목:
  - `type/`, `dao/`, `service/`, `controller/`, `test/` 존재
  - static service 규칙
  - FK 금지
  - 테스트 통과
  - 빌드 성공

#### store-dev
- `peach-gen-store` 기준으로 Pinia Store를 생성합니다.
- Backend 타입과 인터페이스를 맞춥니다.
- 완료 기준: `npx vue-tsc --noEmit`

#### ui-dev
- `peach-gen-ui`, 필요 시 `peach-gen-design`을 사용합니다.
- `figma=[URL]`가 있으면 디자인을 해석합니다.
- UI 패턴(`ui=`)이 없으면 사용자에게 확인합니다.
- 완료 기준: `npx vue-tsc --noEmit`, `bun run lint:fix`, `bun run build`

#### frontend-qa
- Store와 UI 구조를 독립 검증합니다.
- 확인 항목:
  - 파일 구조
  - Composition API / Pinia Option API 패턴
  - `listAction`, `resetAction`, `listMovePage`, `watch` 패턴
  - 타입 체크, 린트, 빌드
  - 금지된 AI Slop UI 패턴 없음

## Failure Handling

- Backend QA 실패 → backend-dev 수정 → backend-qa 재검증
- Store 문제 → store-dev 수정 → frontend-qa 재검증
- UI 문제 → ui-dev 수정 → frontend-qa 재검증

## Completion

- 모든 QA가 통과해야 완료입니다.
- 최종 보고에는 다음을 포함합니다:
  - 모듈명
  - mode
  - 생성 파일 목록
  - 테스트/타입체크/린트/빌드 결과
  - 남은 수동 확인 항목

## Examples

```bash
# 기존 UI에 API + Store 연결
/peach-agent-team notice-board mode=backend

# UI만 구현
/peach-agent-team member-list mode=ui ui=two-depth figma=https://figma.com/file/xxx

# 전체 풀스택 생성
/peach-agent-team product-manage mode=fullstack ui=page file=Y
```
