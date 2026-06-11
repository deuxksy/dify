# Dify 백엔드 API

## 설정 및 실행

> [!IMPORTANT]
>
> v1.3.0 릴리즈에서 `poetry`가 [`uv`](https://docs.astral.sh/uv/)로 교체되어
> Dify API 백엔드 서비스의 패키지 매니저로 사용됩니다.

`uv`와 `pnpm`이 필요합니다.

### 스크립트 사용 (권장)

스크립트는 자체 위치를 기준으로 경로를 해석하므로 어디서든 실행할 수 있습니다.

1. 설정 실행 (환경 파일 복사 및 의존성 설치).

   ```bash
   ./dev/setup
   ```

1. `api/.env`, `web/.env.local`, `docker/middleware.env` 값을 검토 (`SECRET_KEY` 참고 사항은 아래 참조).

1. 미들웨어 시작 (PostgreSQL/Redis/Weaviate).

   ```bash
   ./dev/start-docker-compose
   ```

1. 백엔드 시작 (마이그레이션 먼저 실행).

   ```bash
   ./dev/start-api
   ```

1. Dify [웹](../web) 서비스 시작.

   ```bash
   ./dev/start-web
   ```

   `./dev/setup`과 `./dev/start-web`은 저장소 루트 워크스페이스를 통해 JavaScript 의존성을 설치하므로 별도의 `cd web && pnpm install` 단계가 필요 없습니다.

1. `http://localhost:3000`에서 애플리케이션 설정.

1. 워커 서비스 시작 (비동기 및 스케줄러 태스크, `api`에서 실행).

   ```bash
   ./dev/start-worker
   ```

1. 선택 사항: Celery Beat 시작 (예약된 태스크).

   ```bash
   ./dev/start-beat
   ```

### 환경 참고 사항

> [!IMPORTANT]
>
> 프론트엔드와 백엔드가 다른 서브도메인에서 실행될 때, `COOKIE_DOMAIN`을 사이트의 최상위 도메인(예: `example.com`)으로 설정하세요. 인증 쿠키를 공유하려면 프론트엔드와 백엔드가 같은 최상위 도메인 아래 있어야 합니다.

- `.env` 파일에 `SECRET_KEY`를 생성.

  Linux:

  ```bash
  sed -i "/^SECRET_KEY=/c\\SECRET_KEY=$(openssl rand -base64 42)" .env
  ```

  macOS:

  ```bash
  secret_key=$(openssl rand -base64 42)
  sed -i '' "/^SECRET_KEY=/c\\
  SECRET_KEY=${secret_key}" .env
  ```

## 테스트

1. 백엔드와 테스트 환경 모두에 의존성 설치

   ```bash
   cd api
   uv sync --group dev
   ```

1. `pyproject.toml`의 `tool.pytest_env` 섹션에 mocking된 시스템 환경 변수로 로컬 테스트 실행. 자세한 내용은 [CLAUDE.md](../CLAUDE.md) 참조

   ```bash
   cd api
   uv run pytest                           # 모든 테스트 실행
   uv run pytest tests/unit_tests/         # 단위 테스트만
   uv run pytest tests/integration_tests/  # 통합 테스트

   # 코드 품질
   ./dev/reformat               # 모든 포매터와 린터 실행
   uv run ruff check --fix ./   # 린팅 이슈 수정
   uv run ruff format ./        # 코드 포맷
   uv run pyrefly check         # 타입 체크
   ```

## TypeScript 스텁 생성

```bash
uv run dev/generate_swagger_specs.py --output-dir openapi
```

https://jsontotable.org/openapi-to-typescript를 사용하여 TypeScript로 변환
