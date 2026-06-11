# AGENTS_KR.md

## 프로젝트 개요

Dify는 직관적인 인터페이스로 에이전트 AI 워크플로우, RAG 파이프라인, 에이전트 기능, 모델 관리를 결합한 오픈소스 LLM 애플리케이션 개발 플랫폼입니다.

코드베이스 구성:

- **Backend API** (`/api`): Domain-Driven Design 기반 Python Flask 애플리케이션
- **Frontend Web** (`/web`): TypeScript와 React 기반 Next.js 애플리케이션
- **CLI** (`/cli`): Dify 인스턴스 관리용 TypeScript CLI (`difyctl`)
- **Docker 배포** (`/docker`): 컨테이너화된 배포 설정
- **Dify Agent Backend** (`/dify-agent`): 에이전트 관리 및 실행 백엔드 서비스
- **E2E 테스트** (`/e2e`): Cucumber + Playwright 엔드투엔드 테스트 스위트
- **공유 패키지** (`/packages`): `dify-ui` (디자인 토큰 + 프리미티브), `iconify-collections`, `contracts`, `dev-proxy`, `tsconfig`
- **SDK** (`/sdks`): Node.js 클라이언트 SDK

## 백엔드 워크플로우

- 상세 내용은 `api/AGENTS.md` 참조
- 백엔드 CLI 명령은 `uv run --project api <command>`로 실행
- 통합 테스트는 CI 전용이며 로컬 환경에서 실행하지 않음

## 빠른 참조 명령어

| 작업 | 명령어 |
| :--- | :--- |
| 초기 개발 환경 설정 | `make dev-setup` |
| 백엔드 포맷 | `make format` |
| 백엔드 린트 | `make lint` |
| 백엔드 타입 체크 | `make type-check` |
| 백엔드 테스트 실행 | `make test` |
| 백엔드 전체 테스트 (Docker) | `make test-all` |
| 프론트엔드 개발 서버 | `pnpm --filter dify-web dev` |
| 프론트엔드 린트 | `pnpm --filter dify-web lint:fix` |
| 프론트엔드 타입 체크 | `pnpm --filter dify-web type-check` |
| E2E (인증) | `pnpm -C e2e e2e` |
| E2E (전체 초기화) | `pnpm -C e2e e2e:full` |
| CLI 개발 | `pnpm --filter difyctl dev <command>` |

## 프론트엔드 워크플로우

- 상세 내용은 `web/AGENTS.md` 참조
- CLI 개발은 `cli/AGENTS.md` 참조
- E2E 테스트는 `e2e/AGENTS.md` 참조

## 테스트 및 품질 실천

- TDD 원칙 준수: red → green → refactor
- 백엔드 테스트는 `pytest` 사용, Arrange-Act-Assert 구조 작성
- 강력한 타입 지향; `Any` 사용을 피하고 명시적 타입 어노테이션 선호
- 자기 설명적인 코드 작성; 의도를 설명하는 주석만 추가

## 언어 스타일

- **Python**: 함수와 속성에 타입 힌트를 유지하고, 관련 특수 메서드(예: `__repr__`, `__str__`)를 구현. 타입 안전성과 코드 문서화를 위해 `dict`나 `Mapping`보다 `TypedDict` 선호
- **TypeScript**: strict 설정 사용, ESLint(`pnpm lint:fix` 권장)과 `pnpm type-check`에 의존, `any` 타입 사용 지양

## 일반 실천

- 기존 파일 편집 우선; 새로운 문서는 요청 시에만 추가
- 생성자를 통한 의존성 주입 및 클린 아키텍처 경계 유지
- 올바른 레이어에서 도메인 특화 예외로 에러 처리

## 프로젝트 컨벤션

- 백엔드 아키텍처는 DDD와 Clean Architecture 원칙을 따름
- 비동기 작업은 Celery와 Redis 브로커를 통해 실행
- 프론트엔드 사용자 대면 문자열은 `web/i18n/en-US/` 사용; 하드코딩된 텍스트 금지
- pnpm 모노레포 구조. 워크스페이스 패키지 구성은 `pnpm-workspace.yaml` 참조
