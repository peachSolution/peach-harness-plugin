---
name: peach-agent-team-refactor
description: Use when refactoring existing PeachSolution backend or frontend modules into the test-data pattern while preserving behavior and coordinating independent QA.
---

# Peach Agent Team Refactor

## Overview

PeachSolution 레거시 모듈을 `test-data` 패턴으로 변환하는 리팩토링 전용 팀 스킬입니다.

기존 `team-refactor`를 대체하며, backend/frontend 역할 정의와 QA 절차를 이 문서 안에 포함합니다.

## Inputs

```bash
/peach-agent-team-refactor [모듈명] layer=backend|frontend|all [옵션]

# 옵션
# file=Y|N
# ui=crud|two-depth|select-list
# tdd=Y|N
```

## Preconditions

- 리팩토링 대상 모듈이 존재해야 합니다.
- Backend 리팩토링 시 DB 스키마가 존재해야 합니다.
- 기능 변경이 아니라 구조 정리만 수행합니다.

## Orchestration

### layer=backend
- refactor-backend → backend-qa

### layer=frontend
- refactor-frontend → frontend-qa

### layer=all
- refactor-backend → backend-qa
- refactor-backend → refactor-frontend → frontend-qa

## 역할별 지시

#### refactor-backend
- `peach-refactor-backend` 기준으로 type/dao/service/controller/test를 정리합니다.
- 기존 기능은 유지하고 구조만 개선합니다.
- 완료 기준: `bun test`, `bun run lint:fixed`, `bun run build`

#### backend-qa
- 리팩토링 후 구조, 패턴, 테스트, 빌드를 검증합니다.
- `test-data` 패턴 준수 여부를 확인합니다.

#### refactor-frontend
- `peach-refactor-frontend` 기준으로 type/store/pages/modals를 정리합니다.
- URL watch 패턴, Composition API, Pinia Option API를 강제합니다.
- 완료 기준: `npx vue-tsc --noEmit`, `bun run lint:fix`, `bun run build`

#### frontend-qa
- 파일 구조, watch 패턴, UI 패턴, 빌드 결과를 검증합니다.
- 금지된 UI 패턴과 타입 규칙 위반 여부를 확인합니다.

## Failure Handling

- Backend QA 실패 → refactor-backend 수정 → backend-qa 재검증
- Frontend QA 실패 → refactor-frontend 수정 → frontend-qa 재검증

## Completion

- 기능 100% 보존과 QA 통과가 모두 확인되어야 완료입니다.
- 최종 보고에는 변경 파일 목록, 보존한 기능, 테스트/빌드 결과를 포함합니다.

## Examples

```bash
# 전체 리팩토링
/peach-agent-team-refactor notice-board layer=all

# Backend만 리팩토링
/peach-agent-team-refactor product-manage layer=backend tdd=Y

# Frontend만 리팩토링
/peach-agent-team-refactor member-data layer=frontend ui=two-depth
```
