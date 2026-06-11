# API 에이전트 가이드

## 에이전트를 위한 참고 사항 (필수 확인)

`api/` 아래의 백엔드 코드를 변경하기 전에 주변 docstring과 주석을 반드시 읽어야 합니다. 이 참고 사항은 필수 컨텍스트(불변 조건, 엣지 케이스, 트레이드오프)를 포함하며 스펙의 일부로 간주됩니다.

다음을 확인하세요:

- 소스 코드 파일 상단의 모듈(파일) docstring
- 클래스 및 함수/메서드의 docstring
- 명확하지 않은 로직에 대한 단락/블록 주석

### 무엇을 어디에 작성할 것인가

- 참고 사항의 범위를 유지하세요: 모듈 참고 사항은 모듈 전반의 컨텍스트, 클래스 참고 사항은 클래스 전반의 컨텍스트, 함수/메서드 참고 사항은 행동 계약, 단락/블록 주석은 로컬 "이유"를 다룹니다. 오용을 방지하지 않는 한 동일한 내용을 여러 범위에 중복하지 마세요.
- **모듈(파일) docstring**: 목적, 경계, 핵심 불변 조건, 새 독자가 편집 전에 알아야 할 "주의 사항".
  - 발견이 어려운 경우 핵심 협력자(모듈/서비스)에 대한 교차 링크를 포함합니다.
  - 일시적인 "오늘 우리는…" 참고 사항보다 안정적인 사실(불변 조건, 계약)을 선호합니다.
- **클래스 docstring**: 책임, 수명 주기, 불변 조건, 사용 방법(또는 사용하지 않아야 하는 방법).
  - 클래스가 의도적으로 상태를 가지는 경우, 어떤 상태가 존재하고 어떤 메서드가 상태를 변경하는지 명시합니다.
  - 동시성/비동기 가정이 중요한 경우 명시적으로 기술합니다.
- **함수/메서드 docstring**: 행동 계약.
  - 인수, 반환 형태, 부작용(DB 쓰기, 외부 I/O, 태스크 디스패치), 발생하는 도메인 예외를 문서화합니다.
  - 오용을 방지할 때만 예제를 추가합니다.
- **단락/블록 주석**: 코드가 이미 설명하는 *내용*이 아닌 *이유*(트레이드오프, 역사적 제약, 놀라운 엣지 케이스)를 설명합니다.
  - 주석은 정당화하는 로직 옆에 유지합니다. 더 이상 현실과 일치하지 않는 주석은 삭제하거나 재작성합니다.

### 규칙 (필수 준수)

이 섹션에서 "참고 사항"은 모듈/클래스/함수 docstring과 관련 단락/블록 주석을 의미합니다.

- **작업 전**
  - 수정할 영역의 참고 사항을 읽으세요. 스펙의 일부로 간주합니다.
  - docstring이나 주석이 현재 코드와 충돌하는 경우 **코드를 단일 진실 공급원**으로 간주하고 docstring이나 주석을 현실에 맞게 업데이트합니다.
  - 중요한 의도/불변 조건/엣지 케이스가 누락된 경우 가장 가까운 docstring이나 주석에 추가합니다(전반적 범위는 모듈, 행동은 함수).
- **작업 중**
  - 제약 조건을 발견하거나 결정을 내리거나 접근 방식을 변경할 때 참고 사항을 동기화하세요.
  - 모듈/클래스 간에 책임을 이동/이름 변경하는 경우 독자가 여전히 "이유"와 불변 조건을 찾을 수 있도록 영향을 받는 docstring과 주석을 업데이트합니다.
  - 명확하지 않은 엣지 케이스, 트레이드오프, 테스트/검증 계획을 올바르게 유지될 수 있는 가장 가까운 docstring이나 주석에 기록합니다.
  - 참고 사항을 **일관되게** 유지하세요: 새로운 발견을 관련 docstring과 주석에 통합합니다. 추가 전용 "최근 수정" / 변경 로그 스타일의 추가를 피합니다.
- **작업 완료 시**
  - 변경된 내용, 이유, 새로운 엣지 케이스/테스트를 반영하도록 참고 사항을 업데이트합니다.
  - 현재 지침으로 오해될 수 있지만 더 이상 적용되지 않는 주석은 제거하거나 재작성합니다.
  - Docstring과 주석을 간결하고 정확하게 유지합니다. 반복적인 재발견을 방지하기 위한 것입니다.

## 코딩 스타일

이 저장소의 백엔드 코드에 대한 기본 표준입니다. 새 코드에 적용하고 변경 사항을 리뷰할 때 체크리스트로 사용합니다.

### 린트 및 포맷팅

- Ruff를 사용하여 포맷팅과 린트 수행(`.ruff.toml` 따름).
- 각 줄은 120자 이하로 유지(공백 포함).

### 네이밍 컨벤션

- 변수와 함수에는 `snake_case` 사용.
- 클래스에는 `PascalCase` 사용.
- 상수에는 `UPPER_CASE` 사용.

### 타이핑 및 클래스 레이아웃

- 코드는 일반적으로 저장소의 현재 Python 버전과 일치하는 타입 어노테이션을 포함해야 합니다(타입이 없는 퍼블릭 API와 "미스터리" 값을 피하세요).
- 최신 타이핑 형식(예: `list[str]`, `dict[str, int]`)을 선호하고 강력한 이유가 없는 한 `Any`를 피합니다.
- 키와 값 타입이 알려진 사전 형 데이터에는 `dict[...]`나 `Mapping[...]`보다 `TypedDict`를 선호합니다.
- 타입이 지정된 페이로드의 선택적 키에는 `NotRequired[...]`를 사용합니다(또는 대부분의 필드가 선택적일 때 `total=False`).
- 키 집합이 알려지지 않은 진정한 동적 키 공간에는 `dict[...]` / `Mapping[...]`을 유지합니다.

```python
from datetime import datetime
from typing import NotRequired, TypedDict


class UserProfile(TypedDict):
    user_id: str
    email: str
    created_at: datetime
    nickname: NotRequired[str]
```

- 클래스의 경우, 클래스가 dataclass나 Pydantic 모델이 아니더라도 클래스 본문 상단(`__init__` 전)에 모든 멤버 변수를 타입과 함께 명시적으로 선언하여 클래스 형태를 한눈에 파악할 수 있게 합니다:

```python
from datetime import datetime


class Example:
    user_id: str
    created_at: datetime

    def __init__(self, user_id: str, created_at: datetime) -> None:
        self.user_id = user_id
        self.created_at = created_at
```

### 일반 규칙

- Pydantic v2 컨벤션을 사용합니다.
- 이 저장소의 Python 패키지 관리에 `uv`를 사용합니다(보통 `--project api`와 함께).
- 가벼운 헬퍼에는 작은 "유틸리티 클래스"보다 단순한 함수를 선호합니다.
- 명확히 필요하고 기존 패턴과 일치하는 경우가 아니면 dunder 메서드를 구현하지 않습니다.
- 에이전트 작업의 일환으로 장기 실행 서비스를 시작하지 마세요(`uv run app.py`, `flask run` 등). 테스트 실행은 허용됩니다.
- 파일은 ~800줄 미만으로 유지하고 필요 시 분할합니다.
- 코드를 읽기 쉽고 명시적으로 유지합니다. 영리한 핵은 피합니다.

### 아키텍처 및 경계

- 계층화된 아키텍처를 반영합니다: controller → service → core/domain.
- 새 추상화를 만들기 전에 `core/`, `services/`, `libs/`의 기존 헬퍼를 재사용합니다.
- 관측성에 최적화합니다: 결정론적 제어 흐름, 명확한 로깅, 실행 가능한 에러.

### 로깅 및 에러

- `print`를 절대 사용하지 마세요. 모듈 수준 로거를 사용합니다:
  - `logger = logging.getLogger(__name__)`
- 관련성이 있을 때 로그 컨텍스트에 tenant/app/workflow 식별자를 포함합니다.
- 도메인별 예외(`services/errors`, `core/errors`)를 발생시키고 컨트롤러에서 HTTP 응답으로 변환합니다.
- 재시도 가능한 이벤트는 `warning`, 치명적 실패는 `error`로 로깅합니다.

### SQLAlchemy 패턴

- 모델은 `models.base.TypeBase`를 상속합니다. 임의의 메타데이터나 엔진을 생성하지 마세요.
- 컨텍스트 매니저로 세션을 엽니다:

```python
from sqlalchemy.orm import Session

with Session(db.engine, expire_on_commit=False) as session:
    stmt = select(Workflow).where(
        Workflow.id == workflow_id,
        Workflow.tenant_id == tenant_id,
    )
    workflow = session.execute(stmt).scalar_one_or_none()
```

- SQLAlchemy 표현식을 선호합니다. 필요한 경우가 아니면 raw SQL을 피합니다.
- 항상 `tenant_id`로 쿼리 범위를 지정하고 안전장치(`FOR UPDATE`, 행 수 등)로 쓰기 경로를 보호합니다.
- 매우 큰 테이블(예: 워크플로우 실행)이나 대체 스토리지 전략이 필요한 경우에만 리포지토리 추상화를 도입합니다.

### 스토리지 및 외부 I/O

- `extensions.ext_storage.storage`를 통해 스토리지에 접근합니다.
- 아웃바운드 HTTP 페치에는 `core.helper.ssrf_proxy`를 사용합니다.
- 스토리지를 다루는 백그라운드 태스크는 멱등성이어야 하며 관련 객체 식별자를 로깅해야 합니다.

### Pydantic 사용

- Pydantic v2 모델로 DTO를 정의하고 기본적으로 extras를 금지합니다.
- 도메인 규칙에 `@field_validator` / `@model_validator`를 사용합니다.

예시:

```python
from pydantic import BaseModel, ConfigDict, HttpUrl, field_validator


class TriggerConfig(BaseModel):
    endpoint: HttpUrl
    secret: str

    model_config = ConfigDict(extra="forbid")

    @field_validator("secret")
    def ensure_secret_prefix(cls, value: str) -> str:
        if not value.startswith("dify_"):
            raise ValueError("secret must start with dify_")
        return value
```

### 제네릭 및 프로토콜

- `typing.Protocol`을 사용하여 행동 계약(예: 캐시 인터페이스)을 정의합니다.
- 캐시나 프로바이더 같은 재사용 유틸리티에 제네릭(`TypeVar`, `Generic`)을 적용합니다.
- 제네릭만으로 안전성을 보장할 수 없는 경우 런타임에 동적 입력을 검증합니다.

### 도구 및 검사

반복 중 빠른 검사:

- 포맷: `make format`
- 린트(자동 수정 포함): `make lint`
- 타입 체크: `make type-check`
- 단위 테스트: `make test`
- Docker 기반 스위트를 포함한 전체 백엔드 테스트: `make test-all`
- 대상 테스트: `make test TARGET_TESTS=./api/tests/<target_tests>`

PR 열기 / 제출 전:

- `make lint`
- `make type-check`
- `make test`

### 컨트롤러 및 서비스

- 컨트롤러: Pydantic으로 입력 파싱, 서비스 호출, 직렬화된 응답 반환. 비즈니스 로직 없음.
- 서비스: 리포지토리, 프로바이더, 백그라운드 태스크를 조정합니다. 부작용을 명시적으로 유지합니다.
- 명확하지 않은 동동작은 간결한 docstring과 주석으로 문서화합니다.
- `204 No Content` 응답의 경우 빈 본문만 반환합니다. dict, 모델 또는 다른 페이로드를 반환하지 마세요.
- Flask-RESTX 컨트롤러의 요청, 쿼리, 응답 스키마는 `controllers/API_SCHEMA_GUIDE.md`를 따릅니다.
  요약하면: Pydantic 모델을 사용하고, GET 쿼리 파라미터를 `query_params_from_model(...)`로 문서화하고, 응답 DTO를 `register_response_schema_models(...)`로 등록하고, 응답 DTO를 `dump_response(...)`로 직렬화하며,
  새로운 레거시 `ns.model(...)`, `@marshal_with(...)`, GET `@ns.expect(...)` 패턴을 추가하지 마세요.

### 기타

- 설정에 `configs.dify_config`를 사용합니다. 환경 변수를 직접 읽지 마세요.
- 엔드투엔드로 테넌트 인지를 유지합니다. `tenant_id`는 공유 리소스를 다루는 모든 계층을 통과해야 합니다.
- 비동기 작업은 `services/async_workflow_service`를 통해 큐에 넣습니다. 태스크는 `tasks/` 아래에 명시적 큐 선택과 함께 구현합니다.
- 실험적 스크립트는 `dev/` 아래에 유지합니다. 프로덕션 빌드에 포함하지 마세요.
