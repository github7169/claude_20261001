# src/AGENTS.md

## Module Context

Vite + React 19 클라이언트. 프롬프트 입력, 프로바이더/키 선택, 생성된 컴포넌트 카드(미리보기/코드)를 렌더링한다. 서버와는 `/api/*` 프록시(`vite.config.ts`)로만 통신한다. 루트 `AGENTS.md`의 규칙은 반복하지 않는다.

## Tech Stack & Constraints

- 미리보기는 `react-live`의 `LiveProvider noInline` 사용. 생성 코드는 `render(...)` 호출 필수.
- 서버 호출은 `useComponentGenerator`의 `fetch('/api/generate')`와 `App.tsx`의 `fetch('/api/config')` 두 곳뿐이다. 외부 API를 클라이언트에서 직접 호출하지 않는다.
- 공유 타입은 `src/types/index.ts`(`Provider`, `GeneratedComponent`)에서만 정의한다. 서버의 `Provider`(`server/index.ts:57`)와 값이 같아야 한다.

## Implementation Patterns

- 컴포넌트 파일은 `src/components/<Name>.tsx` 이름 있는 export, 테스트는 같은 폴더의 `<Name>.test.tsx`.
- 상태는 훅(`src/hooks/useComponentGenerator.ts`)에 모은다. 컴포넌트 내부에서 직접 fetch하지 않는다.
- 스타일은 `App.css`/`index.css`의 클래스 기반. 카드 상태 클래스는 `tab--active`처럼 BEM식 `--modifier`를 쓴다.

## Testing Strategy

- `bun run test -- src/`
- 테스트 환경은 jsdom, 설정은 `src/test/setup.ts`. 사용자 상호작용은 `@testing-library/user-event`를 쓴다 (`PromptInput.test.tsx:3`).

## Local Golden Rules

- Do: 미리보기 새로고침은 `LivePreview`에 `key={previewKey}`를 바꿔 리마운트한다 (`ComponentCard.tsx:17,33,70`). 애니메이션 재생이 이 방식에 의존하므로 `setState`로 코드를 재주입하는 방식으로 바꾸지 않는다.
- Don't: 사용자가 입력한 API 키를 `localStorage` 등에 저장하지 않는다. 키는 `App.tsx`의 메모리 상태로만 존재하고 요청 시에만 전송된다 (`App.tsx:14`, `useComponentGenerator.ts:26`).
- Test Boundary: 훅과 `ComponentCard`/`LivePreview`는 테스트가 없다. 로직을 추가하면 순수 함수로 분리하거나 테스트를 같이 추가한다.
- Hard Constraint: `generate`의 `apiKey`가 빈 문자열이면 body에서 제외된다 (`useComponentGenerator.ts:26`, `...(apiKey && { apiKey })`). 서버가 `.env` 키로 폴백하는 동작이 이에 의존한다.
