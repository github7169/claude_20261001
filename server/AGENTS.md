# server/AGENTS.md

## Module Context

Bun 런타임 API 프록시. 클라이언트(`src/`)의 `/api/generate`, `/api/config` 요청을 받아 Anthropic/Google API를 호출한다. 루트 `AGENTS.md`의 규칙은 반복하지 않는다.

## Tech Stack & Constraints

- 외부 HTTP 클라이언트 라이브러리 없이 전역 `fetch`만 사용한다 (`server/index.ts:69,101`).
- 서버 의존성 없음. `package.json`은 루트 하나를 공유한다.
- 실행: `bun --watch run server/index.ts` (`bun run server`).

## Implementation Patterns

- 응답은 `Response.json(..., { headers: CORS_HEADERS })` 형태. 모든 응답(에러 포함)에 `CORS_HEADERS`를 붙인다 (`server/index.ts:51-55`).
- 새 프로바이더는 `Provider` 유니온, `ENV_KEYS`, `callXxx` 함수, `/api/generate` 분기를 함께 추가한다 (`server/index.ts:57-62,183-186`).
- 순수 로직은 `generator.ts`/`fallback.ts`처럼 별도 파일로 분리하고 같은 이름의 `.test.ts`를 둔다.

## Testing Strategy

- `bun run test -- server/` 로 서버 테스트만 실행한다.
- 테스트 파일은 `vitest`에서 `describe/it/expect`를 import한다 (`server/generator.test.ts:1`). Bun 내장 러너 문법으로 작성하지 않는다.

## Local Golden Rules

- 모델명(`claude-haiku-4-5-20251001`, `GOOGLE_MODELS`)은 `server/index.ts`에 하드코딩되어 있다. 변경 시 이 파일에서만 수정하고 근거 없이 다른 모델로 바꾸지 않는다.
- `withModelFallback`은 모든 에러(429 포함)에서 다음 모델로 넘어가고, 전부 실패하면 마지막 에러만 던진다 (`server/fallback.ts:12-19`). 에러 종류별 재시도 로직을 이 함수에 섞지 않는다.
- `MAX_TOKENS` 절단 에러 메시지는 사용자에게 그대로 노출되는 한국어 문구다 (`server/index.ts:124`). 상태 코드 분기(503/429)에 걸리지 않도록 숫자를 메시지에 넣지 않는다.
- Google API 키는 URL 쿼리스트링에 포함된다 (`server/index.ts:99`). 이 URL이나 에러 객체를 로그/응답에 그대로 출력하지 않는다.
