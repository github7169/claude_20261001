# AGENTS.md

## Operational Commands

- 패키지 관리자는 bun 고정. npm/yarn/pnpm 사용 금지 (`bun.lock` 사용).
- 개발 서버(API + Vite 동시): `bun run dev`
- 테스트: `bun run test` (vitest). `bun test`(Bun 내장 러너) 금지.
- 단일 테스트: `bun run test -- server/generator.test.ts`
- 타입 검사 + 빌드: `bun run build`
- 린트: `bun run lint`
- 작업 완료 전 `bun run lint`, `bun run test`, `bun run build` 통과 확인.

## Golden Rules

### Immutable

- API 키는 `.env`(gitignore 처리됨)에만 둔다. 커밋 금지. 클라이언트로는 키 값이 아닌 보유 여부만 내려준다 (`server/index.ts:147-157` `/api/config`는 `!!ENV_KEYS.x`만 반환).
- 클라이언트가 보낸 `apiKey`는 서버에서 `.env` 키보다 우선한다 (`server/index.ts:64-66`). 이 우선순위와 로그에 키를 남기지 않는 동작을 유지한다.
- API 서버 포트 3002는 `server/index.ts:139`와 `vite.config.ts:11`에 중복 정의되어 있다. 하나를 바꾸면 반드시 둘 다 바꾼다.

### Do's & Don'ts

- AI 응답 후처리(`stripCodeFences`, `ensureRenderCall`)는 `server/generator.ts`의 순수 함수로 유지하고 변경 시 `server/generator.test.ts`를 함께 갱신한다. `server/index.ts`는 `Bun.serve`를 실행하는 부수효과가 있어 테스트에서 import할 수 없다.
- 모델 폴백 로직은 `server/fallback.ts`(`withModelFallback`)에만 둔다. 새 프로바이더/모델 추가 시 `GOOGLE_MODELS`(`server/index.ts:5`) 우선순위 배열을 수정한다.
- 서버 에러 메시지는 `message.includes('503'|'429')`로 분기한다 (`server/index.ts:194-206`). 프로바이더 에러를 던질 때 `API error: <status>` 형태를 유지한다.

### 프로젝트 고유 규칙 (코드 근거)

- Asymmetry: Google 경로만 `finishReason === 'MAX_TOKENS'` 절단 검사와 모델 폴백이 있다 (`server/index.ts:123`, `134-136`). Anthropic 경로(`callAnthropic`)에는 없다. Anthropic 경로를 수정할 때 동일한 보호가 필요한지 판단한다.
- Hard Constraint: 미리보기는 react-live `noInline` 모드이며 `render(<X />)` 호출이 없으면 아무것도 그려지지 않는다 (`src/components/LivePreview.tsx:14`, `server/generator.ts:12-15`). 시스템 프롬프트는 import 문과 TypeScript 문법을 금지한다 (`server/index.ts:11,20`). 이 세 지점은 서로 의존하므로 하나만 바꾸지 않는다.
- Double Defense: `render()` 호출 누락은 시스템 프롬프트(`server/index.ts:12`)와 `ensureRenderCall`(`server/generator.ts:16`) 양쪽에서 막는다. 한쪽을 제거하지 않는다.
- Test Boundary: 테스트는 `server/generator.ts`, `server/fallback.ts`, `PromptInput`에만 있다. `server/index.ts`(HTTP 핸들러, 외부 API 호출)와 `useComponentGenerator`, `ComponentCard`, `LivePreview`는 테스트가 없다. 이 영역을 수정할 때는 순수 함수로 분리해 테스트를 추가하는 방향을 우선한다.
- Security Boundary: 생성된 코드는 react-live가 브라우저에서 실행한다 (`src/components/LivePreview.tsx`). 서버/키/쿠키 등 민감 값을 이 코드 경로(미리보기 스코프)에 주입하지 않는다.

## Project Context

- 목표: 프롬프트로 React 컴포넌트를 생성하고 즉시 미리보기/코드 확인.
- Stack: React 19, TypeScript, Vite, Bun(API 서버), react-live, vitest, Anthropic/Google Gemini.

## Standards & References

- 설치/실행/기능은 `README.md` 참조.
- 커밋: `.claude/skills/commit/SKILL.md` 규칙. `타입: 한국어 요약` (feat/fix/refactor/chore). 사용자 승인 없이 커밋하지 않는다. 기본 브랜치는 `master`.
- 코드 주석과 UI 문구는 한국어.
- Maintenance Policy: 이 문서의 규칙과 코드가 어긋나면(예: 포트, 모델 목록, 테스트 범위) 작업 중 업데이트를 제안한다.

## Context Map

- **[API 서버 / AI 프로바이더 수정](./server/AGENTS.md)** — Bun 서버, 프롬프트, 모델 폴백 작업 시.
- **[React 클라이언트 UI 수정](./src/AGENTS.md)** — 컴포넌트, 훅, 미리보기 작업 시.
