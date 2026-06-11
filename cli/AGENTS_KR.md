# AGENTS.md — difyctl (TypeScript CLI)

difyctl의 TypeScript 포트. 스택: 커스텀 CLI 프레임워크(`src/framework/`), Node 22+, ESM, ky for HTTP, vitest, eslint via @antfu/eslint-config.

> 아키텍처 패턴, 스캐폴딩 레시피, 프린터 체인, 전략 패턴, 테스트 컨벤션, 안티패턴: **[`ARD.md`]** 참조.

## 코드 규칙

- **공백, 탭 금지.**
- **최소 주석.** 코드가 스스로 말하게 합니다. 명확하지 않은 이유(숨겨진 제약, 미묘한 불변 조건, 버그 해결 메모)만 주석으로 작성합니다. 코드를 반복하지 마세요. 태스크, PR, 현재 호출자를 참조하지 마세요.
- **매직 문자열이나 숫자 금지.** 제한된 값 집합에는 enum이나 명명된 상수를 사용합니다.
- **긴 위치 인수 목록 금지.** 옵션 객체를 사용합니다.
- **판별자에 대한 긴 if/switch 사다리 금지.** 다형성, 디스패치 테이블, 또는 전략 패턴을 사용합니다. 개념에 이름을 부여하고 구현체가 플러그인되도록 합니다.
- **`any` 금지. 진정한 와이어 경계**(HTTP 본문 파싱, 환경 변수) 밖에서 `unknown` 금지. 다른 모든 곳에서 타입을 좁힙니다.
- **`!` non-null 단언 피하기.** 대신 좁히기를 사용합니다.
- **변경하지 않는 입력에 `readonly` 사용.**
- **변형 데이터에는 판별 유니온**을 사용합니다(SSE 이벤트, 실행 출력, 에러 형태). 선택적 필드 모음은 사용하지 않습니다.
- **하위 호환성 심 금지.** 이전 이름의 재수출 금지, `// removed:` 마커 금지, 폐기 노트 금지. 삭제하고 호출자를 업데이트합니다.
- **명시적 승인 없이 새 의존성 금지.**
- **리팩터 커밋에서 CLI 동작 변경 금지.** 동일한 플래그, 동일한 출력, 동일한 종료 코드.
- **모든 리프 명령은 `DifyCommand`를 상속.** 명령이 에이전트 워크플로우 문서의 이점이 있을 때 `static agentGuide` 문자열을 추가합니다. `src/commands/AGENTS.md` 참조.

## 레이어링

| 레이어    | 경로                             | 역할                                                                                                                         |
| --------- | -------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| commands  | `src/commands/`                  | 명령 클래스 셸(`DifyCommand` 상속). 프레임워크 임포트가 실행되는 유일한 장소.                                                |
| domain    | `src/run/`, `src/get/` 등        | 순수 TS 모듈. 옵션으로 타입이 지정된 의존성을 받습니다. 프레임워크 없이 테스트 가능.                                         |
| api       | `src/api/`                       | 리소스당 하나의 타입이 지정된 클라이언트. 각각 `KyInstance`를 받습니다.                                                      |
| http      | `src/http/`                      | `createClient` + 미들웨어(auth, retry, logging, error mapping). ky가 실행되는 유일한 장소.                                   |
| io        | `src/io/`                        | 스트림 + 스피너. 데이터 출력과 진행 UI 사이의 펜스.                                                                          |
| printers  | `src/printers/`                  | `CompositePrintFlags` + `-o {json,yaml,name,wide,text}` 매트릭스.                                                            |
| errors    | `src/errors/`                    | `BaseError`, `ErrorCode` enum, `ExitCode` enum, 디스패치 테이블, `formatErrorForCli`.                                        |
| guide     | `src/commands/**/<cmd>/guide.ts` | 명령별 에이전트 가이드 문자열. `agentGuide`를 내보내고 명령 클래스에 `static agentGuide = agentGuide` 할당. `--help`로 노출. |
| cache     | `src/cache/`                     | 온디스크 캐시(app-info 등).                                                                                                  |
| auth      | `src/auth/`                      | Hosts 파일, 토큰 스토어, 로그인 플로우.                                                                                      |
| config    | `src/config/`                    | XDG 디렉토리 해석, config.yml 로드/저장.                                                                                     |
| workspace | `src/workspace/`                 | 해석기: flag → env → bundle.                                                                                                 |
| types     | `src/types/`                     | 순수 데이터 + 서버 계약을 위한 zod 스키마. 외부로 런타임 임포트 없음.                                                        |

## 명령 구조

스캐폴딩 레시피 + 체크리스트: `ARD.md §New command scaffold` 참조. 전체 폴더 컨벤션(서브명령, guide.ts): `src/commands/AGENTS.md` 참조.

레이어 규칙:

- 명령은 얇은 셸입니다. 베어러 컨텍스트에 `this.authedCtx(opts)`를 사용하고 도메인 함수에 위임합니다.
- 도메인은 옵션을 통해 의존성을 받습니다. `src/framework/`를 임포트하지 않습니다.
- 런타임에 ky를 임포트하는 것은 `src/http/client.ts`와 `src/api/*`뿐입니다. 다른 곳에서는 `import type { KyInstance }`를 사용합니다.
- `process.*`는 `src/io/`, `src/store/dir.ts`, `src/util/browser.ts`에 있습니다. 다른 곳에는 없습니다.
- 순환 임포트 금지. `types/`는 순수 리프.

## 개발 명령어

```sh
pnpm install                                   # 최초 1회
pnpm dev <command> [args...]                   # 소스에서 CLI 실행 (--separator 불필요)
pnpm test                                      # vitest
pnpm test:coverage                             # 커버리지 포함
pnpm type-check                                # tsc, no emit
pnpm lint                                      # eslint
pnpm lint:fix                                  # eslint --fix
pnpm build                                     # 프로덕션 번들 (vp pack)
pnpm tree:gen                                  # src/commands/tree.ts 재생성 (레지스트리)
pnpm tree:check                                # tree.ts가 파일 시스템과 동기화되었는지 확인
```

릴리스 바이너리(5개 플랫폼 대상, Bun 컴파일)는 `pnpm build:bin`에서 생성됩니다(`.github/workflows/cli-release.yml`에서 호출).

## 테스트

- 동작 테스트는 `test/fixtures/dify-mock/`의 실제 Hono 목에 대해 실행됩니다. `nock`, `msw`, `fetchMock` 없음 — 모든 테스트가 실제 HTTP를 실행합니다.
- 테스트 파일은 함께 위치합니다: `foo.ts` 옆에 `foo.test.ts`.
- 커밋 전에 타입 체크, 린트, 전체 테스트 스위트가 녹색이어야 합니다.

## 스펙 문서 (`docs/specs/`)

동작 계약. 살아있는 트리 — 제자리에서 수정, 버전 하위 폴더 없음.

**유지:** HTTP 와이어 형태(요청/응답 JSON, 헤더, 상태 코드), SQL DDL, Redis 키 + TTL, 상태 전환, 감사 이벤트 이름 + 페이로드, 에러/종료 코드, 속도 제한 값, JWS/쿠키 봉투 클레임.

**제거:** 언어 타입 선언, 내부 헬퍼 서명, 데코레이터 스니펫, 파일 경로 테이블, 코드를 반영하는 의사코드, "Open items"/"Handler walk"/"CI guard"/"Migration" 섹션, 근거(`Rejected:`/`Why X not Y`/`Historical note:`/제품 비교), 릴리스 파이프라인 라인, 버전 고정(`in v1.0`, `post-v1.0`, 마일스톤 코드), frontmatter `date`/`status`/`author`.

**테스트:** "내일 Rust로 재작성해도 스펙이 유지되는가?" HTTP/SQL/Redis는 유지, 타입 정의는 사라집니다.

**규칙:** 근거가 아닌 동작. 파일당 하나의 주제, 교차 참조는 `auth.md §Storage`. 산문보다 테이블. 드리프트 시 코드가 우선 — 스펙 업데이트.

## 관련 없는 작업의 범위 외

지나가면서 수정하지 마세요:

- `test/fixtures/dify-mock/`의 퍼블릭 표면(엔드포인트, JSON 형태, 상태 코드, 시나리오 이름) — dify-api 계약입니다.
- `bin/`, `scripts/`, `Makefile`, `eslint.config.js`, `tsconfig*.json`, `package.json`(태스크에 필요한 변경이 아닌 경우).

## 커밋

- 커밋당 하나의 관심사. 스타일: `<type>(<scope>): <imperative subject>` 소문자. 본문은 명확하지 않은 경우 이유를 설명.
- 명시적 사용자 승인 없이 push, amend, force-push, hooks 건너뛰기(`--no-verify`) 금지.

[`ARD.md`]: ARD.md
