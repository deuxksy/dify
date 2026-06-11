## 프론트엔드 워크플로우

- 상세 프론트엔드 워크플로우 지침은 `./docs/test.md`와 `./docs/lint.md`를 참조
- 프론트엔드 코딩 작업 시 React 컴포넌트, 상태 소유권, 라우팅, 스타일링 또는 Tailwind 클래스를 다루는 경우 저장소 내 `how-to-write-component` 스킬도 적용
- 프론트엔드 리뷰 시 저장소 내 `frontend-code-review` 스킬을 공식 체크리스트로 사용

## i18n

- 사용자 대면 문자열은 하드코딩된 텍스트 대신 `web/i18n/en-US/` 키를 사용
- i18n 키를 추가하거나 이름을 변경할 때, 지원되는 모든 로케일 파일을 올바른 현지화 값으로 업데이트. 해당 키에 대해 저장소가 의도적으로 영어 폴백을 사용하지 않는 한 비영어 로케일에 폴백 영어를 남기지 않음

## 오버레이 컴포넌트 (필수)

- `../packages/dify-ui/README.md`는 오버레이 프리미티브, 포털, 루트 `isolation: isolate`, `z-50` / `z-60` 레이어링에 대한 영구 계약
- `./docs/overlay.md`는 현재 웹 오버레이 모범 사례를 기록
- 새 코드나 수정된 코드에서는 `@langgenius/dify-ui/*`의 오버레이 프리미티브만 사용
- `@/app/components/base/*`에서 오버레이 가져오기를 도입하지 않음; 기존 호출자를 다룰 때는 마이그레이션

## SVG 아이콘 (필수)

- 새 커스텀 SVG 아이콘은 `../packages/iconify-collections/assets/...`에 추가
- `pnpm --filter @dify/iconify-collections generate`를 실행하고 Tailwind `i-custom-*` 클래스로 생성된 아이콘 사용
- Tailwind가 시작 시 커스텀 아이콘 컬렉션을 로드하므로 아이콘 재생성 후 웹 개발 서버 재시작 필수
- `app/components/base/icons/src/...`에 새로 생성된 React 아이콘 컴포넌트나 JSON 파일을 추가하지 않음
- 전체 워크플로우는 `../packages/iconify-collections/README.md` 참조

## 디자인 토큰 매핑

- Figma 디자인을 코드로 변환할 때, Figma `--radius/*` 토큰에서 Tailwind `rounded-*` 클래스로의 매핑은 `../packages/dify-ui/AGENTS.md`를 참조. 두 스케일은 한 단계 오프셋되어 있음

## 클라이언트 상태 관리

- 하나의 컴포넌트가 소유한 상태에는 로컬 컴포넌트 상태 사용
- 동일한 기능 내 여러 컴포넌트에서 공유되는 간단한 클라이언트 상태에는 기능 수준 Jotai atom 사용. 특히 컴포넌트가 공유 진실 공급원, 파생 값 또는 공유 액션을 필요로 할 때
- 워크플로우 캔버스, 드래그, 리사이즈, 패널 런타임 상태와 같은 복잡하거나 고빈도 상호작용 상태에는 기존 기능 스토어 사용
- `foxact/use-local-storage`는 사용자 기본 설정, 해제된 알림, UI 기본값과 같은 저빈도 클라이언트 전용 지속성에만 사용. localStorage를 앱 상태의 라이브 진실 공급원으로 사용하지 않음
- 고빈도 상호작용의 경우, 상호작용 중에 기능 상태를 업데이트하고 커밋 또는 안정된 업데이트 시에만 스토리지를 지속
- 앱 코드에서 `localStorage`, `window.localStorage`, `globalThis.localStorage`에 직접 접근하지 않음; 스토리지 훅 경계를 사용하고 기존 raw/커스텀 스토리지 형식 보존
- 공유 상태를 위해 임시 글로벌 이벤트 리스너를 추가하지 않음. atom, 기존 스토어 또는 공유 구독 훅을 선호하여 리스너를 중앙 집중화하고 중복 제거

## 자동화된 테스트 생성

- 프론트엔드 자동화된 테스트 생성의 공식 지침서로 `./docs/test.md` 사용
- 테스트를 제안하거나 저장할 때, 해당 문서를 다시 읽고 모든 요구 사항을 따름
- 모든 프론트엔드 테스트는 `frontend-testing` 스킬도 준수해야 함. 스킬을 선택적 지침이 아닌 필수 제약으로 취급
