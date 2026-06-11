# @langgenius/dify-ui

공유 디자인 토큰, `cn()` 유틸리티, CSS 우선 Tailwind 스타일, `web/`에서 소비하는 헤드리스 프리미티브 컴포넌트.

## 컴포넌트 작성 규칙

- `@base-ui/react` 프리미티브 + `cva` + `cn`을 사용합니다.
- dify-ui 내부의 컴포넌트 간 임포트는 상대 경로(`../button`)를 사용합니다. 외부 소비자는 하위 경로 익스포트(`@langgenius/dify-ui/button`)를 사용합니다.
- `web/`에서의 임포트 금지. next / i18next / ky / jotai / zustand에 대한 의존성 금지.
- 폴더당 하나의 컴포넌트: `src/<name>/index.tsx`, 선택적 `index.stories.tsx` 및 `__tests__/index.spec.tsx`. 일치하는 `./<name>` 하위 경로를 `package.json#exports`에 추가합니다.
- Props 패턴: `Omit<BaseXxx.Root.Props, 'className' | ...> & VariantProps<typeof xxxVariants> & { /* custom */ }`.
- 공유 내부 모듈에서 타입이 지정된 prop을 컴포넌트가 받을 때만 일반 `Omit<...>`을 사용합니다. prop이 관련 prop의 유효한 형태를 변경할 때(예: `value` / `defaultValue`, `multiple` / `value`, 또는 `clearable` / `onChange`) 평탄화 대신 명시적 판별 유니온이나 분배 헬퍼로 그 관계를 모델링합니다.
- 컴포넌트가 공유 내부 모듈에서 타입이 지정된 prop을 받을 때, 소비자가 컴포넌트 하위 경로에서 임포트할 수 있도록 해당 컴포넌트에서 `export type`합니다.
- 시각적 상태에 대해 Base UI 데이터 속성과 CSS 변수를 선호합니다. 클래스 추가를 위해 React에 상태를 미러링하지 마세요.
- Base UI API나 셀렉터 계약이 불명확할 때 `README.md`에 링크된 문서와 로컬 `@base-ui/react` `.d.ts` 파일을 코딩 전에 읽습니다.

## 오버레이 프리미티브 선택: Tooltip vs PreviewCard vs Popover

**트리거의 목적**과 **접근성 도달 범위**로 선택합니다. 시각적 풍부함이 아닙니다.

| 프리미티브    | 열리는 시점           | 트리거의 목적         | 콘텐츠                 | 터치/스크린 리더에서 접근 가능? |
| ------------- | --------------------- | --------------------- | ---------------------- | ------------------------------- |
| `Tooltip`     | hover / focus         | 자체 액션을 가짐      | 짧은 일반 텍스트 라벨  | ❌ (라벨만)                     |
| `PreviewCard` | hover / focus         | 기본 클릭 대상을 가짐 | 보충 미리보기          | ❌ (클릭 대상 경유)             |
| `Popover`     | click / tap (+ hover) | **팝업을 열기 위함**  | 긴 텍스트 포함 모든 것 | ✅                              |

Base UI 결정 규칙([문서]):

> _"트리거의 목적이 팝업 자체를 여는 것이라면 팝오버입니다.
> 트리거의 목적이 팝업을 여는 것과 관련이 없다면 툴팁입니다."_

먼저 이것을 적용한 후 좁힙니다:

- `Tooltip` — 일시적인 시각적 라벨. 트리거는 이미 자체 `aria-label` / 보이는 텍스트를 가져야 합니다. 툴팁은 시각적 마우스/키보드 사용자를 위해 이를 미러링합니다. 대화형 UI, 여러 줄 산문 없음. 체류 불가.
- `PreviewCard` — 클릭이 다른 곳으로 이동하는 트리거(링크, 선택 가능한 행, 이동 가능한 칩)에 고정된 hover로 노출되는 풍부한 보충 미리보기. **엄격한 계약:** 팝업은 트리거의 클릭 대상에서 접근할 수 없는 정보나 액션을 절대 포함하지 않아야 합니다 — 터치와 스크린 리더 사용자는 열 수 없습니다. 정보가 팝업에 고유하다면 `Popover`(클릭 또는 `openOnHover`)로 전환하거나 클릭 대상으로 이동합니다. 이 구분을 회피하기 위해 `Popover` 위에 "hover to open"을 직접 구현하지 마세요.
- `Popover` — 자체 상호작용이 있는 모든 팝업, 또는 "infotip"(`?` / `(i)` 글리프의 유일한 목적이 도움말 텍스트를 노출하는 것). infotip 케이스에 대해 `PopoverTrigger`에 `openOnHover`를 전달합니다 — `Tooltip` / `PreviewCard`와 달리 터치와 스크린 리더 사용자에게 접근 가능합니다(팝오버가 탭과 포커스에서도 열리므로).

## Border Radius: Figma 토큰 → Tailwind 클래스 매핑

Figma 디자인 시스템의 `--radius/*` 토큰 스케일은 Tailwind CSS v4 기본값에서 **한 단계 오프셋**되어 있습니다. Figma 스펙을 코드로 변환할 때 항상 이 매핑을 사용합니다 — `radius-*`를 CSS 클래스로 사용하지 않고, 프리셋에서 `borderRadius`를 확장하지 않습니다.

| Figma 토큰      | 값    | Tailwind 클래스  |
| --------------- | ----- | ---------------- |
| `--radius/2xs`  | 2px   | `rounded-xs`     |
| `--radius/xs`   | 4px   | `rounded-sm`     |
| `--radius/sm`   | 6px   | `rounded-md`     |
| `--radius/md`   | 8px   | `rounded-lg`     |
| `--radius/lg`   | 10px  | `rounded-[10px]` |
| `--radius/xl`   | 12px  | `rounded-xl`     |
| `--radius/2xl`  | 16px  | `rounded-2xl`    |
| `--radius/3xl`  | 20px  | `rounded-[20px]` |
| `--radius/6xl`  | 28px  | `rounded-[28px]` |
| `--radius/full` | 999px | `rounded-full`   |

### 규칙

- 커스텀 `borderRadius` 테마 값을 **추가하지 마세요**. Tailwind v4 기본값과 표준 없는 크기에 대한 임의의 값(`rounded-[Npx]`)을 사용합니다.
- CSS 클래스 이름으로 `radius-*`를 **사용하지 마세요**. 이전 `@utility radius-*` 정의는 제거되었습니다.
- Figma MCP가 `rounded-[var(--radius/sm, 6px)]`를 반환하면 위 테이블의 표준 Tailwind 클래스로 변환합니다(예: `rounded-md`).
- 표준 Tailwind에 해당하지 않는 값(10px, 20px, 28px)에는 `rounded-[10px]` 같은 임의의 값을 사용합니다.

## 검색 / 피커 프리미티브 선택: Autocomplete vs Combobox vs Select

사용자가 자유 형식 텍스트를 입력하는지, 기억된 값을 선택하는지, 닫힌 목록에서 선택하는지에 따라 결정합니다.

Base UI 결정 규칙:

- [Autocomplete 문서]: 선택이 기억되어야 하고 입력 값이 커스텀이 될 수 없으면 `Autocomplete` 대신 `Combobox`를 사용합니다.
- [Combobox 문서]: 제한 없는 텍스트 입력이 필요한 단순 검색 위젯에는 `Combobox`를 사용하지 마세요. 대신 `Autocomplete`를 사용합니다.

Dify UI에서 이 구분을 적용합니다:

- `Autocomplete` — 선택적 제안이나 완성이 있는 자유 형식 텍스트 입력. 입력 값이 커스텀일 수 있으며 반드시 선택된 옵션이 될 필요는 없습니다. 검색 상자, 명령형 제안, 태그 제안, 비동기 텍스트 완성에 사용합니다.
- `Combobox` — 값이 컬렉션에서 선택된 하나 이상의 항목인 검색 가능한 피커. 선택된 값은 루트에 의해 기억되고 자유 형식 텍스트는 최종 값이 아닙니다. 모델 피커, 사용자 피커, 데이터셋/문서 피커, 다중 선택 칩에 사용합니다.
- `Select` — 텍스트 입력 없는 닫힌 목록 피커. 옵션 집합이 작거나 이미 스캔 가능하고 필터링이 불필요할 때 사용합니다.

구성 규칙:

- 퍼블릭 API에서 Base UI 프리미티브 시맨틱을 유지합니다. `ComboboxInputGroup`, `ComboboxInput`, `ComboboxContent`, `ComboboxList`, `ComboboxItem`, `ComboboxItemIndicator` 같은 복합 파트를 하나의 비즈니스 컴포넌트로 래핑하지 않고 익스포트합니다.
- `Combobox` 다중 선택은 공식 칩 패턴을 따릅니다: `ComboboxInputGroup`에 `ComboboxChips`가 포함되고, `ComboboxValue`가 `ComboboxChip` 항목을 렌더링하며, `ComboboxInput`은 칩 행 안에 유지됩니다. 칩은 줄바꿈되고 입력 그룹이 수직으로 확장되도록 허용합니다(수평 오버플로우 강제 금지).
- 콘텐츠 프리미티브는 자체 Base UI `Portal`을 소유하고 `README.md`의 오버레이 계약에 맞게 `Positioner`에 `z-50`을 사용합니다. Toast는 `z-60`을 소유합니다.
- `Autocomplete`와 `Combobox` 팝업에 `w-(--anchor-width)`를 사용하되 뷰포트 인식 최대 너비를 적용합니다. 가용 너비 클램핑을 무효화할 때 `min-w-(--anchor-width)`를 추가하지 마세요.

[Autocomplete 문서]: https://base-ui.com/react/components/autocomplete.md#usage-guidelines
[Combobox 문서]: https://base-ui.com/react/components/combobox.md#usage-guidelines
[문서]: https://base-ui.com/react/components/tooltip#infotips
