# @langgenius/dify-ui

Dify의 `web/` 앱에서 소비하는 공유 UI 프리미티브, 디자인 토큰, CSS 우선 Tailwind 스타일, `cn()` 유틸리티.

프리미티브는 [Base UI] 헤드리스 컴포넌트를 둘러싼 얇고 의견이 반영된 래퍼로, `cva` + `cn`과 Dify 디자인 토큰으로 스타일링됩니다.
업스트림 컴포넌트 문서는 [Base UI 문서 인덱스]에서 시작하세요.

> `private: true` — 이 패키지는 pnpm 워크스페이스를 통해 `web/`에서 소비되며 npm에 게시되지 않습니다. API를 Dify 내부용으로 간주하되 워크스페이스 내에서는 안정적으로 유지합니다.

## 설치

`web/package.json`에 워크스페이스 의존성으로 이미 연결되어 있습니다. 설치할 필요가 없습니다.

새로운 워크스페이스 소비자의 경우 추가:

```jsonc
{
  "dependencies": {
    "@langgenius/dify-ui": "workspace:*"
  }
}
```

## 임포트

항상 **하위 경로 익스포트**에서 임포트합니다 — 배럴이 없습니다:

```ts
import { Button } from '@langgenius/dify-ui/button'
import { cn } from '@langgenius/dify-ui/cn'
import { Dialog, DialogContent, DialogTrigger } from '@langgenius/dify-ui/dialog'
import { Drawer, DrawerPopup, DrawerTrigger } from '@langgenius/dify-ui/drawer'
import { FieldControl, FieldLabel, FieldRoot } from '@langgenius/dify-ui/field'
import { Form } from '@langgenius/dify-ui/form'
import { Kbd, KbdGroup } from '@langgenius/dify-ui/kbd'
import { Popover, PopoverContent, PopoverTrigger } from '@langgenius/dify-ui/popover'
import { SegmentedControl, SegmentedControlItem } from '@langgenius/dify-ui/segmented-control'
import { Textarea } from '@langgenius/dify-ui/textarea'
import '@langgenius/dify-ui/styles.css' // 앱 루트에서 한 번
```

`@langgenius/dify-ui`(하위 경로 없음)에서의 임포트는 의도적으로 지원되지 않습니다 — 트리 쉐이킹을 간단하게 유지하고 Storybook / 테스트 커버리지를 프리미티브별로 귀속시킵니다.

## 프리미티브

| 카테고리         | 하위 경로                                                                                                                                                                      | 참고                                                                                           |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------- |
| Actions          | `./button`                                                                                                                                                                     | `cva` 변형이 있는 디자인 시스템 CTA 프리미티브.                                                |
| Controls         | `./segmented-control`                                                                                                                                                          | 모드, 필터, 뷰 선택을 위한 SegmentedControl.                                                   |
| Display          | `./collapsible`, `./kbd`                                                                                                                                                       | Collapsible 공개 프리미티브; 키보드 입력 및 단축키 키캡 프리미티브.                            |
| Feedback         | `./meter`, `./toast`                                                                                                                                                           | Meter는 인라인 상태, Toast는 `z-60` 레이어를 소유.                                             |
| Form             | `./form`, `./field`, `./fieldset`, `./input`, `./textarea`, `./checkbox`, `./checkbox-group`, `./radio`, `./radio-group`, `./number-field`, `./select`, `./slider`, `./switch` | 네이티브 폼 경계, 필드 시맨틱, 컨트롤.                                                         |
| Layout           | `./scroll-area`                                                                                                                                                                | 호스트 뷰포트 위에 커스텀 스타일이 적용된 스크롤바.                                            |
| Media            | `./avatar`                                                                                                                                                                     | Avatar 루트, 이미지, 폴백 프리미티브.                                                          |
| Navigation       | `./file-tree`, `./pagination`, `./tabs`                                                                                                                                        | 미리보기 지향 파일 공개 목록을 위한 FileTree; 페이지 탐색을 위한 Pagination; 패널을 위한 Tabs. |
| Overlay / menu   | `./alert-dialog`, `./context-menu`, `./dialog`, `./drawer`, `./dropdown-menu`, `./popover`, `./preview-card`, `./tooltip`                                                      | 포탈 제공. 아래 [오버레이 및 포탈 계약] 참조.                                                  |
| Search / pickers | `./autocomplete`, `./combobox`, `./select`                                                                                                                                     | 검색 입력, 검색 가능한 피커, 닫힌 피커.                                                        |

유틸리티:

- `./cn` — `clsx` + `tailwind-merge` 래퍼. 조건부 클래스 조합에 사용합니다.
- `./styles.css` — 디자인 토큰, 테마 변수, 프로젝트 유틸리티/컴포넌트를 제공하는 단일 CSS 진입점. 앱 루트에서 한 번 임포트합니다.

## Segmented control 계약

`SegmentedControl`은 Dify 디자인 시스템의 모드, 필터, 뷰 선택용 프리미티브입니다. Base UI `ToggleGroup` + `Toggle`로 구축되어 있으므로 UI에 `tablist` / `tabpanel` 시맨틱이 필요하면 `Tabs`를 대신 사용합니다.

값 계약은 Base UI를 따릅니다: `value`, `defaultValue`, `onValueChange`는 배열을 사용하며, 단일 선택 모드에서 활성 항목이 토글 해제되면 빈 배열을 보고할 수 있습니다.

## Form 계약

Dify UI의 폼 프리미티브는 네이티브 폼 시맨틱, 필드 접근성, 디자인 시스템 스타일링을 위한 Base UI 컴포지션 레이어입니다. 폼 상태 관리 프레임워크가 아닙니다. 기본 컴포넌트 계약은 업스트림 [Base UI 폼 핸드북], [Base UI Form], [Base UI Field], [Base UI Fieldset] 문서를 참조하세요.

제출 경계에 `Form`을 사용합니다. 네이티브 `<form>`을 렌더링하고 Enter-to-submit 및 submit 버튼 동작을 유지하며, 구조화된 값과 통합 필드 검증을 위해 Base UI의 `onFormSubmit`, `errors`, `actionsRef`, `validationMode` API를 추가합니다. 폼이 Dify UI 필드로 구성될 때 베어 `<form>`보다 선호합니다.

각 독립적인 이름이 지정된 필드에 `FieldRoot`를 사용합니다. 필드는 안정적인 `name`, 라벨 관계, `FieldControl`이나 동일한 Base UI 필드 컨텍스트에 참여하는 다른 컨트롤이 있어야 합니다. 일반 폼 행에는 보이는 라벨을 선호합니다. 주변 UI가 이미 보이는 텍스트를 제공할 때는 매칭되는 라벨 프리미티브를 시각적으로 숨기거나 실제 대화형 컨트롤에 `aria-label`을 넣습니다. `FieldDescription`과 `FieldError`는 스크린 리더에 필요한 메시지 관계를 제공하며, Dify 래퍼는 디자인 시스템의 기본 Form Input Set 스타일링을 추가합니다.

컨트롤 시맨틱에 따라 라벨 프리미티브를 선택합니다. 텍스트형 입력, `Textarea`, 입력 기반 `Combobox` / `Autocomplete`, 단일 `Checkbox` / `Radio`, `Switch`, `NumberField`는 `FieldLabel`을 사용합니다. 트리거 기반 `Select` 필드는 `SelectLabel`을 사용합니다. `Slider` 필드는 `SliderLabel`을 사용하며, 엄지가 구별된 이름이 필요할 때만 엄지별 `aria-label`을 사용합니다. `SelectGroupLabel`과 `AutocompleteGroupLabel`은 팝업 콘텐츠 내의 그룹화된 옵션에만 라벨을 지정합니다. 필드 라벨이 아닙니다.

하나의 필드가 관련 컨트롤 그룹으로 표현될 때(체크박스 그룹, 라디오 그룹, 다중 엄지 슬라이더, 여러 입력을 결합하는 섹션) `FieldsetRoot`과 `FieldsetLegend`를 사용합니다. 체크박스와 라디오 그룹의 경우 각 옵션을 `FieldItem`으로 감싸고 각 옵션에 자체 라벨을 부여합니다:

```tsx
<FieldRoot name="allowedNetworkProtocols">
  <FieldsetRoot render={<CheckboxGroup />}>
    <FieldsetLegend>Allowed network protocols</FieldsetLegend>
    <FieldItem>
      <FieldLabel className="flex items-center gap-2">
        <Checkbox value="https" />
        HTTPS
      </FieldLabel>
    </FieldItem>
  </FieldsetRoot>
</FieldRoot>
```

`FieldsetRoot`는 그룹 시맨틱과 legend 관계를 제공합니다. 그룹화된 컨트롤의 대화형 상태를 소유하지 않습니다. `disabled`, `value`, `defaultValue`, 변경 핸들러를 필드셋 래퍼에 의존하지 않고 실제 그룹 프리미티브(`CheckboxGroup`, 라디오 그룹, 슬라이더 루트 등)에 전달합니다.

복잡한 비즈니스 폼의 경우 이 프리미티브 외부에 상태 소유권을 유지합니다. TanStack Form, zod, 서버 검증, 다이얼로그 리셋 동동작, 스키마 기반 렌더링은 `web/`의 기능 레이어에 속합니다. 이 프리미티브에 `name`, `invalid`, `dirty`, `touched`, `value`, `onValueChange`, 에러를 전달합니다. 이 저장소에서 `web/app/components/base/form`은 TanStack/스키마 런타임 어댑터입니다. `packages/dify-ui`는 프리미티브 레이어로 유지됩니다.

`web/`의 마이그레이션 규칙: UI에 저장/제출 액션이 있으면 관련 없는 `Input`과 `Button` 조각으로 두지 마세요. `Form`이나 네이티브 `<form>`으로 실제 제출 경계를 부여하고, 적절한 라벨 프리미티브(`FieldLabel`, `SelectLabel`, `SliderLabel`, 또는 `FieldsetLegend`)를 통해 보이는 필드 이름을 부여하고, `FieldDescription` / `FieldError`로 헬퍼/에러 텍스트를 노출하고, 제출이 아닌 버튼은 `type="button"`으로 유지합니다.

## Tailwind CSS v4 통합

이 패키지는 Tailwind CSS v4의 CSS 우선 설정 모델을 사용합니다. 소비자는 자체 루트 스타일시트에서 Tailwind를 임포트한 후 이 패키지의 CSS 진입점을 임포트합니다:

```css
@import 'tailwindcss';
@import '@langgenius/dify-ui/styles.css';
```

소비자가 워크스페이스를 통해 Dify UI 소스 파일을 사용하는 경우, Tailwind가 유틸리티 클래스를 감지할 수 있도록 명시적 소스를 추가합니다:

```css
@source '../packages/dify-ui/src';
```

## 오버레이 및 포탈 계약

오버레이 프리미티브는 `document.body`에 연결된 [Base UI Portal] 안에 떠 있는 표면을 렌더링합니다. 이것이 Base UI 기본값입니다 — 기본 동작은 업스트림 [포탈][Base UI Portal] 문서를 참조하세요. `DialogContent`, `PopoverContent`, `SelectContent` 같은 편의 콘텐츠 컴포넌트는 내부적으로 포탈을 소유합니다. 명시적 포탈 구조가 있는 프리미티브(예: `Drawer`)는 소비자가 전체 Base UI 구조를 조합할 수 있도록 매칭되는 `DrawerPortal` 파트를 노출합니다.

### 루트 격리 요구 사항

호스트 앱은 루트에 격리된 스태킹 컨텍스트를 설정하여 포탈된 오버레이 레이어가 조상의 `transform` / `filter` / `contain` 스타일에 의해 잘리거나 재정렬되지 않도록 **해야 합니다**. Dify 웹 앱에서는 `web/app/layout.tsx`에서 수행합니다:

```tsx
<body>
  <div className="isolate h-full">{children}</div>
</body>
```

동등한 방법: CSS에서 `isolation: isolate`가 있는 모든 루트 요소. 없으면 하위 항목이 새 스태킹 컨텍스트를 생성할 때 오버레이가 Safari에서 시각적으로 잘릴 수 있습니다.

### z-index 레이어링

모든 오버레이 프리미티브는 단일 공유 z-index를 사용합니다. 호출 사이트에서 **절대** 오버라이드하지 마세요.

| 레이어                                                                                                              | z-index | 위치                                                              |
| ------------------------------------------------------------------------------------------------------------------- | ------- | ----------------------------------------------------------------- |
| Overlays (Dialog, AlertDialog, Autocomplete, Combobox, Drawer, Popover, DropdownMenu, ContextMenu, Select, Tooltip) | `z-50`  | Positioner / Backdrop                                             |
| Toast viewport                                                                                                      | `z-60`  | 알림이 다이얼로그 아래에 숨겨지지 않도록 오버레이 위에 한 레이어. |

근거: Dify UI는 일반 애플리케이션 오버레이 레이어를 소유합니다. 오버레이 프리미티브는 `z-50`을 공유하며 **DOM 순서**에 의존하여 스태킹합니다 — 나중에 마운트된 포탈이 우선합니다. Toast는 `z-60`을 소유하여 알림이 `z-9999`로 폴백하지 않고도 다이얼로그, 팝오버, 기타 포탈된 표면 위에 유지됩니다.

`web/docs/overlay.md`에서 웹 앱 오버레이 모범 사례를 확인하세요.

### 규칙

- 이 패키지의 프리미티브에 임시 `z-*` 오버라이드를 절대 추가하지 마세요. 무언가 잘린다면 하위 프리미티브 대신 부모 오버레이 구조를 수정하세요.
- 프리미티브 위에 수동 포탈을 추가로 생성하지 마세요 — `DialogContent`, `PopoverContent`, `DrawerPortal` 같은 익스포트된 콘텐츠 / 포탈 파트를 사용합니다. Base UI가 포커스 관리, 스크롤 락, 해제를 처리합니다.
- 프리미티브에 추가 프레젠테이션 크롬(예: 커스텀 백드롭)이 필요할 때 호출 사이트가 아닌 익스포트된 컴포넌트 **내부**에 추가합니다.

### Tooltip, infotip, 팝오버 시맨틱

- `Tooltip`은 짧고 비대화형 시각적 라벨에만 사용합니다. 트리거에는 이미 보이는 텍스트나 `aria-label`이 있어야 합니다. 툴팁은 접근 가능한 이름이 아니며 링크, 버튼, 폼, 구조화된 산문을 포함해서는 안 됩니다.
- `Popover`는 설명 콘텐츠, 긴 텍스트, 풍부한 레이아웃, 터치나 보조 기술로 접근해야 하는 모든 것에 사용합니다. `web/`에서 `Infotip` 래퍼는 `Popover`로 백업된 `?` 도움말 글리프에 권장되는 패턴입니다.
- `placement`를 선택하고 프리미티브가 간격을 소유하게 합니다. 컴포넌트 API가 명시적으로 측정된 레이아웃 예외를 필요로 하지 않는 한 호출 사이트별 오프셋은 피합니다.
- Base UI 트리거 `render` prop을 전달할 때 버튼형 트리거에는 실제 `<button type="button">`을 렌더링합니다. Popover 트리거가 `div`, `span` 또는 다른 비버튼 요소를 렌더링해야 하는 경우 `nativeButton={false}`를 전달합니다.

## 개발

- `pnpm -C packages/dify-ui test` — 프리미티브의 Vitest 단위 테스트.
- `pnpm -C packages/dify-ui storybook` — 기본 포트에서 Storybook. 각 프리미티브에 `index.stories.tsx`가 있습니다.
- `pnpm -C packages/dify-ui type-check` — 이 패키지만 `tsgo --noEmit`.

### 테스트에서 애니메이션 비활성화

Base UI는 오버레이, 패널, 전환 기반 컴포넌트를 언마운트하기 전에 `element.getAnimations()`가 완료될 때까지 대기할 수 있습니다. 브라우저 기반 테스트 러너는 이 타이밍을 불안정하게 만들 수 있습니다. 특히 테스트가 애니메이션 동작이 아닌 최종 DOM 상태를 단언할 때.

Vitest 설정 파일에서 Base UI 테스트 플래그를 설정하여 대기를 건너뜁니다:

```ts
(
  globalThis as typeof globalThis & {
    BASE_UI_ANIMATIONS_DISABLED: boolean
  }
).BASE_UI_ANIMATIONS_DISABLED = true
```

`packages/dify-ui/vitest.setup.ts`가 프리미티브 테스트에 이 설정을 이미 적용하고 있습니다.

`[AGENTS.md](./AGENTS.md)`에서 다음을 확인하세요:

- 컴포넌트 작성 규칙(폴더당 하나의 컴포넌트, `cva` + `cn`, 패키지 내 상대 경로 임포트, 소비자의 하위 경로 임포트).
- Figma `--radius/`* 토큰 → Tailwind `rounded-*` 클래스 매핑.

## 이 패키지에 포함되지 않는 것

- 애플리케이션 상태(`jotai`, `zustand`), 데이터 페칭(`ky`, `@tanstack/react-query`, `@orpc/*`), i18n(`next-i18next` / `react-i18next`), 라우팅(`next`)은 모두 `web/`에 있습니다. 이 패키지는 그들에 대한 의존성이 없으며, 다른 앱에서 소비되거나 추출될 수 있도록 그렇게 유지해야 합니다.
- 비즈니스 컴포넌트(채팅, 워크플로우, 데이터셋 뷰 등). 이들은 `web/app/components/...`에 속합니다.

[Base UI Field]: https://base-ui.com/react/components/field
[Base UI Fieldset]: https://base-ui.com/react/components/fieldset
[Base UI Form]: https://base-ui.com/react/components/form
[Base UI Portal]: https://base-ui.com/react/overview/quick-start#portals
[Base UI 문서 인덱스]: https://base-ui.com/llms.txt
[Base UI 폼 핸드북]: https://base-ui.com/react/handbook/forms
[Base UI]: https://base-ui.com/react
[오버레이 및 포탈 계약]: #오버레이-및-포탈-계약
