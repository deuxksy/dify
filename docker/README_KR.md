## Docker 배포 README

Docker Compose를 사용하여 Dify를 배포하는 새로운 `docker` 디렉토리에 오신 것을 환영합니다. 이 README는 업데이트 사항, 배포 지침 및 기존 사용자를 위한 마이그레이션 정보를 안내합니다.

### 업데이트 사항

- **Certbot 컨테이너**: `docker-compose.yaml`에 SSL 인증서 관리를 위한 `certbot`이 포함되었습니다. 이 컨테이너는 인증서를 자동으로 갱신하고 안전한 HTTPS 연결을 보장합니다.
  자세한 내용은 `docker/certbot/README.md`를 참조하세요.

- **영구 환경 변수**: 필수 시작 기본값은 `.env.example`에 제공되며, 로컬 값은 `.env`에 저장되어 배포 간에 설정이 유지됩니다.

  > `.env`란?</br> </br>
  > `.env` 파일은 로컬 시작 파일입니다. 기본 배포를 위해 `.env.example`에서 복사합니다. 선택적 고급 설정은 `envs/*.env.example` 파일에 있습니다.

- **통합 벡터 데이터베이스 서비스**: 모든 벡터 데이터베이스 서비스가 단일 Docker Compose 파일 `docker-compose.yaml`에서 관리됩니다. `.env` 파일에서 `VECTOR_STORE` 환경 변수를 설정하여 다른 벡터 데이터베이스 간에 전환할 수 있습니다.

### `docker-compose.yaml`으로 Dify 배포 방법

1. **사전 요구 사항**: 시스템에 Docker와 Docker Compose가 설치되어 있어야 합니다.
2. **환경 설정**:
   - `docker` 디렉토리로 이동합니다.
   - `.env.example`을 `.env`로 복사합니다.
   - 필수 시작 기본값을 변경하려면 `.env`를 커스터마이즈합니다. 고급 설정이 필요한 경우 `envs/`에서 `.example` 접미사 없이 파일을 복사합니다.
   - **선택 사항 (고급 배포)**:
     `.env.example`에서 복사한 전체 `.env` 파일을 유지하는 경우, 환경 변수 동기화 도구를 사용하여 커스텀 설정을 유지하면서 최신 `.env.example` 업데이트에 맞출 수 있습니다.
     아래 [환경 변수 동기화](#environment-variables-synchronization) 섹션을 참조하세요.
3. **서비스 실행**:
   - `docker` 디렉토리에서 `docker compose up -d`를 실행하여 서비스를 시작합니다.
   - 벡터 데이터베이스를 지정하려면 `.env` 파일의 `VECTOR_STORE` 변수를 원하는 벡터 데이터베이스 서비스(예: `milvus`, `weaviate`, `opensearch`)로 설정합니다. 지원되는 전체 목록은 `envs/vectorstores/`를 참조하세요.
   ```bash
   cp .env.example .env
   docker compose up -d
   ```

4. **SSL 인증서 설정**:
   - `docker/certbot/README.md`를 참조하여 Certbot으로 SSL 인증서를 설정합니다.
5. **OpenTelemetry Collector 설정**:
   - `envs/core-services/shared.env.example`을 `envs/core-services/shared.env`로 복사합니다.
   - `ENABLE_OTEL=true`를 설정하고 `OTLP_BASE_ENDPOINT`를 구성합니다. 필요에 따라 같은 파일의 다른 `OTEL_*` 설정을 조정합니다.

### Dify 개발을 위한 미들웨어 배포

1. **미들웨어 설정**:
   - `docker-compose.middleware.yaml`을 사용하여 데이터베이스 및 캐시와 같은 필수 미들웨어 서비스를 설정합니다.
   - `docker` 디렉토리로 이동합니다.
   - `cp envs/middleware.env.example middleware.env`를 실행하여 `middleware.env` 파일을 생성합니다 (`envs/middleware.env.example` 파일 참조).
2. **미들웨어 서비스 실행**:
   - `docker` 디렉토리로 이동합니다.
   - `docker compose --env-file middleware.env -f docker-compose.middleware.yaml -p dify up -d`를 실행하여 PostgreSQL/MySQL(`DB_TYPE`에 따라)과 번들로 제공되는 Weaviate 인스턴스를 시작합니다.

> Compose는 `middleware.env`에서 `COMPOSE_PROFILES=${DB_TYPE:-postgresql},weaviate`를 자동으로 로드하므로 추가 `--profile` 플래그가 필요하지 않습니다. 다른 서비스 조합을 원하면 `middleware.env`의 변수를 조정하세요.

### 기존 사용자 마이그레이션

`docker-legacy` 설정에서 마이그레이션하는 사용자:

1. **변경 사항 검토**: 새로운 `.env` 설정과 Docker Compose 설정에 익숙해지세요.
2. **커스터마이징 이전**:
   - `docker-compose.yaml`, `ssrf_proxy/squid.conf`, `nginx/conf.d/default.conf` 등 커스터마이즈된 설정이 있는 경우, 생성한 `.env` 파일에 이러한 변경 사항을 반영해야 합니다.
3. **데이터 마이그레이션**:
   - 필요한 경우 데이터베이스, 캐시 등 서비스의 데이터를 적절히 백업하고 새 구조로 마이그레이션합니다.

### `.env`, `.env.example`, `envs/` 개요

- `.env.example`은 Docker Compose 배포를 위한 필수 기본 설정을 포함합니다.
- `.env`은 `.env.example`에서 복사한 로컬 시작 값과 로컬 변경 사항을 포함합니다.
- `envs/*.env.example` 파일은 테마별로 그룹화된 선택적 고급 설정을 포함합니다.

루트 `.env.example`은 기본 Docker Compose 배포를 시작하는 데 필요한 변수로만 제한하세요.
선택적, 고급, 프로바이더별 또는 서비스별 변수는 추가하지 말고 해당 `envs/*.env.example` 파일에 배치하세요.

Docker Compose는 `envs/*.env` 파일이 있으면 먼저 읽고, 마지막으로 `.env`를 읽어 `.env`의 값이 우선 적용됩니다.

#### 주요 모듈 및 커스터마이징

- **벡터 데이터베이스 서비스**: 사용하는 벡터 데이터베이스 유형(`VECTOR_STORE`)에 따라 특정 엔드포인트, 포트, 인증 정보를 설정할 수 있습니다.
- **스토리지 서비스**: 스토리지 유형(`STORAGE_TYPE`)에 따라 S3, Azure Blob, Google Storage 등에 대한 특정 설정을 구성할 수 있습니다.
- **API 및 웹 서비스**: API와 웹 프론트엔드 작동 방식에 영향을 미치는 URL 및 기타 설정을 정의할 수 있습니다.

#### 기타 주요 변수

루트 `.env.example` 파일은 필수 시작 설정을 포함합니다. 선택적 및 프로바이더별 설정은 `envs/*.env.example` 파일에 그룹화되어 있습니다. 주요 섹션과 변수는 다음과 같습니다:

1. **공통 변수**:

   - `CONSOLE_API_URL`, `CONSOLE_WEB_URL`, `SERVICE_API_URL`, `APP_API_URL`, `APP_WEB_URL`: API 및 프론트엔드 서비스의 공개 URL입니다.
   - `SERVER_CONSOLE_API_URL`: 웹 서비스가 서버 측 콘솔 API 요청에 사용하는 내부 URL입니다. 표준 Docker Compose 배포에서는 기본값 `http://api:5001`을 유지하고, 웹 서비스가 다른 내부 주소를 통해 API에 접근해야 하는 경우에만 변경하세요.
   - `FILES_URL`, `INTERNAL_FILES_URL`: 파일 다운로드 및 미리보기를 위한 공개 및 내부 기본 URL입니다.
   - `ENDPOINT_URL_TEMPLATE`, `NEXT_PUBLIC_SOCKET_URL`, `TRIGGER_URL`: 추가 서비스 URL입니다.

   전체 목록은 `.env.example`을 참조하세요.

2. **서버 설정**:

   - `LOG_LEVEL`, `DEBUG`, `FLASK_DEBUG`: 로깅 및 디버그 설정입니다.
   - `SECRET_KEY`: 세션, JWT, 파일 URL 서명에 사용되는 키입니다. 비워두면 Dify가 스토리지 디렉토리에 지속적인 키를 생성합니다. 직접 고유한 값을 설정할 수도 있습니다.

3. **데이터베이스 설정**:

   - `DB_USERNAME`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`: PostgreSQL 데이터베이스 자격 증명 및 연결 정보입니다.

4. **Redis 설정**:

   - `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`: Redis 서버 연결 설정입니다.
   - `REDIS_KEY_PREFIX`: Redis 키, 토픽, 스트림, Celery Redis 전송 아티팩트에 대한 선택적 글로벌 네임스페이스 접두사입니다.

5. **Celery 설정**:

   - `CELERY_BROKER_URL`: Celery 메시지 브로커 설정입니다.

6. **스토리지 설정**:

   - `STORAGE_TYPE`, `OPENDAL_SCHEME`, `OPENDAL_FS_ROOT`: 기본 로컬 파일 스토리지 설정입니다. 선택적 스토리지 백엔드는 `envs/` 아래의 파일에서 구성합니다.

7. **벡터 데이터베이스 설정**:

   - `VECTOR_STORE`: 벡터 데이터베이스 유형(예: `weaviate`, `milvus`). 지원되는 전체 목록은 `envs/vectorstores/`를 참조하세요.
   - 각 벡터 스토어의 특정 설정(예: `WEAVIATE_ENDPOINT`, `MILVUS_URI`).

8. **CORS 설정**:

   - `WEB_API_CORS_ALLOW_ORIGINS`, `CONSOLE_CORS_ALLOW_ORIGINS`: 교차 출처 리소스 공유 설정입니다.

9. **OpenTelemetry 설정**:

   - `ENABLE_OTEL`: API에서 OpenTelemetry Collector 활성화
   - `OTLP_BASE_ENDPOINT`: OTLP 익스포터 엔드포인트

10. **기타 서비스별 환경 변수**:

    - `nginx`, `redis`, `db`, 벡터 데이터베이스 등 각 서비스는 `docker-compose.yaml`에서 직접 참조되는 특정 환경 변수를 가집니다.

### 환경 변수 동기화

Dify를 업그레이드하거나 최신 변경 사항을 가져올 때, 시작에 필요한 경우에만 `.env.example`에 새로운 환경 변수가 추가되거나, 고급/프로바이더별/서비스별 설정을 위해 `envs/` 아래의 선택적 파일에 추가될 수 있습니다.

기본 워크플로우를 사용하는 경우 `.env.example`을 검토하고 `.env`를 필수 시작 값에 맞게 유지하세요.

`.env.example`에서 복사한 커스터마이즈된 `.env` 파일을 유지하는 경우, 선택적 환경 변수 동기화 도구가 제공됩니다.

> 이 도구는 `.env.example`에서 `.env`로 **단방향 동기화**를 수행합니다.
> `.env`의 기존 값은 자동으로 덮어쓰지 않습니다.

#### `dify-env-sync.sh` (선택 사항)

이 스크립트는 현재 `.env` 파일과 최신 `.env.example` 템플릿을 비교하고 새로 추가되거나 업데이트된 환경 변수를 안전하게 적용합니다.

**수행하는 작업**

- 변경 전 현재 `.env` 파일의 백업을 생성합니다.
- `.env.example`에서 새로 추가된 환경 변수를 동기화합니다.
- `.env`의 모든 기존 커스텀 값을 보존합니다.
- 검토를 위해 차이점과 `.env.example`에서 제거된 변수를 표시합니다.

**백업 동작**

동기화 전 현재 `.env` 파일이 타임스탬프 파일명(예: `env-backup/.env.backup_20231218_143022`)으로 `env-backup/` 디렉토리에 저장됩니다.

**사용 시기**

- Dify를 새 버전으로 업그레이드한 후 전체 `.env` 파일을 사용하는 경우
- `.env.example`에 새로운 환경 변수가 업데이트된 경우
- `.env.example`에서 복사한 크거나 많이 커스터마이즈된 `.env` 파일을 관리하는 경우

**사용법**

```bash
# 실행 권한 부여 (최초 1회)
chmod +x dify-env-sync.sh

# 동기화 실행
./dify-env-sync.sh
```

### 추가 정보

- **지속적 개선 단계**: 커뮤니티의 피드백을 적극적으로 수렴하여 배포 프로세스를 개선하고 있습니다. 더 많은 사용자가 이 새로운 방식을 채택함에 따라 여러분의 경험과 제안을 바탕으로 지속적으로 개선해 나가겠습니다.
- **지원**: 상세한 설정 옵션과 환경 변수 설정은 `docker` 디렉토리의 `.env.example` 파일과 Docker Compose 설정 파일을 참조하세요.

이 README는 새로운 Docker Compose 설정을 사용한 배포 과정을 안내합니다. 문제가 있거나 추가 도움이 필요한 경우 공식 문서를 참조하거나 지원팀에 문의하세요.
