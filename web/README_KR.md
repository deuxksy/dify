# Dify 프론트엔드

이 프로젝트는 [Next.js] 기반이지만 [vinext]로 개발할 수도 있습니다. [Vite+]와 해당 `vp` 명령어를 사용할 수도 있습니다.

## 시작하기

### 소스 코드로 실행

웹 프론트엔드 서비스를 시작하기 전에 다음 환경이 준비되어 있는지 확인하세요.

- [Node.js]
- [pnpm]

[vinext]와 해당 `vp` 명령어를 사용할 수도 있습니다.
예를 들어 `pnpm install` 대신 `vp install`, `pnpm run test` 대신 `vp test`를 사용합니다.

> [!TIP]
> 패키지 매니저 버전을 자동으로 관리하기 위해 Corepack을 설치하고 활성화하는 것이 권장됩니다:
>
> ```bash
> npm install -g corepack
> corepack enable
> ```
>
> 자세한 내용: [Corepack]

저장소 루트에서 다음 명령어를 실행합니다.

먼저 의존성을 설치합니다:

```bash
pnpm install
```

> [!NOTE]
> JavaScript 의존성은 저장소 루트의 워크스페이스 파일로 관리됩니다: `package.json`, `pnpm-lock.yaml`, `pnpm-workspace.yaml`, `.nvmrc`.
> 저장소 루트에서 의존성을 설치한 후 `web/`에서 프론트엔드 스크립트를 실행하세요.

그런 다음 환경 변수를 설정합니다.
`web/.env.local`을 생성하고 `web/.env.example`의 내용을 복사합니다.
필요에 따라 환경 변수 값을 수정합니다:

```bash
cp web/.env.example web/.env.local
```

> [!IMPORTANT]
>
> 1. 프론트엔드와 백엔드가 다른 서브도메인에서 실행될 때 `NEXT_PUBLIC_COOKIE_DOMAIN=1`을 설정하세요. 인증 쿠키를 공유하려면 프론트엔드와 백엔드가 같은 최상위 도메인 아래 있어야 합니다.
> 1. `NEXT_PUBLIC_API_PREFIX`와 `NEXT_PUBLIC_PUBLIC_API_PREFIX`를 올바른 백엔드 API URL로 설정해야 합니다.

마지막으로 개발 서버를 실행합니다:

```bash
pnpm -C web run dev
# 또는 더 나은 개발 경험을 제공하는 vinext 사용
pnpm -C web run dev:vinext
# (선택 사항) 개발 중 온라인 API를 사용할 수 있도록 dev proxy 서버 시작
# web/dev-proxy.config.ts를 편집하여 프록시 경로 선택
# web/.env.local을 편집하여 DEV_PROXY_TARGET, DEV_PROXY_ENTERPRISE_TARGET, DEV_PROXY_HOST, DEV_PROXY_PORT 오버라이드
pnpm -C web run dev:proxy
```

브라우저에서 <http://localhost:3000>을 열어 결과를 확인합니다.

`web/app` 아래의 파일을 편집할 수 있습니다.
파일을 편집하면 페이지가 자동으로 업데이트됩니다.

## 배포

### 서버 배포

먼저 프로덕션용으로 빌드합니다:

```bash
pnpm -C web run build
```

그런 다음 서버를 시작합니다:

```bash
pnpm -C web run start
```

Docker 이미지를 수동으로 빌드하는 경우 저장소 루트를 빌드 컨텍스트로 사용합니다:

```bash
docker build -f web/Dockerfile -t dify-web .
```

호스트와 포트를 커스터마이즈하려면:

```bash
pnpm -C web run start --port=3001 --host=0.0.0.0
```

## Storybook

이 프로젝트는 UI 컴포넌트 개발을 위해 [Storybook]을 사용합니다.

Storybook 서버를 시작하려면:

```bash
pnpm -C web storybook
```

브라우저에서 <http://localhost:6006>을 열어 결과를 확인합니다.

## 린트

VSCode를 사용하는 경우 `.vscode/settings.example.json`을 `.vscode/settings.json`으로 이름 변경하여 린트 설정을 적용합니다.

그런 다음 [Lint 문서]를 따라 코드를 린트합니다.

## 테스트

유닛 테스트에 [Vitest]와 [React Testing Library]를 사용합니다.

**📖 전체 테스트 가이드**: 상세한 테스트 사양, 모범 사례, 예제는 [web/docs/test.md]를 참조하세요.

> [!IMPORTANT]
> Vite+를 사용하므로 `vitest` 명령어를 사용할 수 없습니다.
> 반드시 `vp` 명령어로 테스트를 실행하세요.
> 예를 들어 `npx vitest` 대신 `npx vp test`를 사용합니다.

테스트 실행:

```bash
pnpm -C web test
```

> [!NOTE]
> 테스트가 아직 완전히 안정적이지 않으며 적극적으로 개선하고 있습니다.
> CI에서만 실패하고 로컬에서는 성공하는 경우 무시하고 이슈를 보고해 주세요.
> CI에서 테스트를 다시 실행하면 성공할 수 있습니다.

### 예제 코드

테스트 작성에 익숙하지 않다면 다음을 참조하세요:

- [index.spec.tsx] - 컴포넌트 테스트 예제

### 컴포넌트 복잡도 분석

테스트를 작성하기 전에 스크립트로 컴포넌트 복잡도를 분석하세요:

```bash
pnpm analyze-component app/components/your-component/index.tsx
```

이를 통해 테스트 전략을 결정할 수 있습니다. 자세한 내용은 [web/testing/testing.md]를 참조하세요.

## 문서

전체 문서는 <https://docs.dify.ai>에서 확인할 수 있습니다.

## 커뮤니티

Dify 커뮤니티는 [Discord 커뮤니티]에서 질문, 아이디어 공유, 프로젝트 자랑을 할 수 있습니다.

[Corepack]: https://github.com/nodejs/corepack#readme
[Discord 커뮤니티]: https://discord.gg/5AEfbxcd9k
[Lint 문서]: ./docs/lint.md
[Next.js]: https://nextjs.org
[Node.js]: https://nodejs.org
[React Testing Library]: https://testing-library.com/docs/react-testing-library/intro
[Storybook]: https://storybook.js.org
[Vite+]: https://viteplus.dev
[Vitest]: https://vitest.dev
[index.spec.tsx]: ./app/components/base/action-button/__tests__/index.spec.tsx
[pnpm]: https://pnpm.io
[vinext]: https://github.com/cloudflare/vinext
[web/docs/test.md]: ./docs/test.md
