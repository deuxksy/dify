# @langgenius/dev-proxy

프론트엔드 프로젝트를 위한 범용 Hono 기반 개발 프록시. 이 패키지는 제품별 라우트, 쿠키 이름, 환경 변수 컨벤션을 포함하지 않습니다. 모든 프록시 경로와 업스트림 대상은 로컬 설정 파일에 선언합니다.

## 설치

```bash
pnpm add -D @langgenius/dev-proxy
```

프론트엔드 프로젝트에 스크립트를 추가합니다:

```json
{
  "scripts": {
    "dev:proxy": "dev-proxy --config ./dev-proxy.config.ts --env-file ./.env.local"
  }
}
```

실행:

```bash
pnpm dev:proxy
```

## CLI

```bash
dev-proxy --config ./dev-proxy.config.ts
```

지원 옵션:

- `--config`, `-c`: 설정 파일 경로. 기본값 `dev-proxy.config.ts`.
- `--env-file`: 설정 파일 평가 전 환경 변수 로드.
- `--host`: 설정의 `server.host` 오버라이드.
- `--port`: 설정의 `server.port` 오버라이드.
- `--watch`: 설정 파일 및 환경 변수 변경 시 리로드. 기본 활성화.
- `--no-watch`: 설정 및 환경 변수 리로드 비활성화.
- `--help`, `-h`: 도움말 출력.

`--target`은 지원하지 않습니다. 라우트와 업스트림이 명시적으로 유지되도록 설정 파일에 대상을 넣으세요.

CLI는 기본적으로 설정 파일과 명시적 `--env-file`을 감시합니다. 라우트, CORS, 대상, 쿠키 재작성 변경은 실행 중인 프로세스에 즉시 적용됩니다. 해결된 호스트나 포트가 변경되면 프록시가 기존 서버를 닫고 새로 시작합니다.

## 설정 구조

```ts
import { defineDevProxyConfig } from '@langgenius/dev-proxy'

export default defineDevProxyConfig({
  server: {
    host: '127.0.0.1',
    port: 5001,
  },
  routes: [
    {
      paths: '/api',
      target: 'https://example.com',
    },
  ],
  cors: {
    allowedOrigins: 'local',
  },
})
```

설정 파일은 `.ts`, `.mts`, `.js`, `.mjs` 형식을 지원합니다.

`routes`는 선언 순서대로 매칭됩니다. 첫 번째로 매칭된 라우트가 우선합니다. 각 경로는 정확한 경로와 모든 하위 경로를 매칭하므로 `paths: '/api'`는 `/api`, `/api/apps`, `/api/apps/123`을 모두 매칭합니다.

기본적으로 `localhost`, `127.0.0.1`, `::1` 등 로컬 개발 출처에 대해 credentialed CORS가 허용됩니다. 특정 출처로 제한하려면:

```
cors: {
  allowedOrigins: ['http://localhost:3000'],
}
```

## 시나리오 1: 단일 로컬 라우트 그룹을 온라인 백엔드로 프록시

로컬 프론트엔드가 하나의 프록시 서버를 통해 온라인 백엔드를 호출해야 할 때 사용합니다. 예를 들어 프론트엔드가 `http://127.0.0.1:5001/api/apps`를 호출하면 프록시가 `https://cloud.example.com/api/apps`로 전달합니다.

```ts
import { defineDevProxyConfig } from '@langgenius/dev-proxy'

const target = process.env.DEV_PROXY_TARGET || 'https://cloud.example.com'

export default defineDevProxyConfig({
  server: {
    host: process.env.DEV_PROXY_HOST || '127.0.0.1',
    port: Number(process.env.DEV_PROXY_PORT || 5001),
  },
  routes: [
    {
      paths: '/api',
      target,
    },
  ],
})
```

선택적 `.env`:

```env
DEV_PROXY_TARGET=https://cloud.example.com
DEV_PROXY_HOST=127.0.0.1
DEV_PROXY_PORT=5001
```

명령어:

```bash
dev-proxy --config ./dev-proxy.config.ts --env-file ./.env.local
```

`./.env.local` 편집 시 프록시가 자동으로 리로드됩니다.

## 시나리오 2: 두 라우트 그룹을 두 로컬 백엔드로 프록시

하나의 프론트엔드가 두 개의 다른 로컬 서비스와 통신해야 할 때 사용합니다. 예:

- `/console/api/*`는 로컬 콘솔 백엔드 `http://127.0.0.1:5001`로 전달
- `/api/*`는 로컬 퍼블릭 API 백엔드 `http://127.0.0.1:5002`로 전달

```ts
import { defineDevProxyConfig } from '@langgenius/dev-proxy'

const consoleApiTarget = process.env.DEV_PROXY_CONSOLE_API_TARGET || 'http://127.0.0.1:5001'
const publicApiTarget = process.env.DEV_PROXY_PUBLIC_API_TARGET || 'http://127.0.0.1:5002'

export default defineDevProxyConfig({
  server: {
    host: process.env.DEV_PROXY_HOST || '127.0.0.1',
    port: Number(process.env.DEV_PROXY_PORT || 8082),
  },
  routes: [
    {
      paths: '/console/api',
      target: consoleApiTarget,
    },
    {
      paths: '/api',
      target: publicApiTarget,
    },
  ],
})
```

선택적 `.env`:

```env
DEV_PROXY_CONSOLE_API_TARGET=http://127.0.0.1:5001
DEV_PROXY_PUBLIC_API_TARGET=http://127.0.0.1:5002
DEV_PROXY_HOST=127.0.0.1
DEV_PROXY_PORT=8082
```

두 라우트 그룹이 겹칠 때 더 구체적인 것을 먼저 배치하세요:

```ts
routes: [
  { paths: '/api/enterprise', target: 'http://127.0.0.1:5003' },
  { paths: '/api', target: 'http://127.0.0.1:5002' },
]
```

## 쿠키 재작성

쿠키 재작성은 옵트인 방식이며 설정으로 구동됩니다. 이 패키지는 애플리케이션 쿠키 이름을 알지 못합니다.

업스트림이 `__Host-`나 `__Secure-` 같은 보안 쿠키 접두사를 사용하지만 로컬 개발에서는 `http://localhost`로 쿠키가 작동해야 할 때 `cookieRewrite`를 사용합니다.

```ts
import type { CookieRewriteOptions } from '@langgenius/dev-proxy'
import { defineDevProxyConfig } from '@langgenius/dev-proxy'

const cookieRewrite: CookieRewriteOptions = {
  hostPrefixCookies: ['access_token', 'refresh_token', /^passport-/],
}

export default defineDevProxyConfig({
  routes: [
    {
      paths: '/api',
      target: 'https://cloud.example.com',
      cookieRewrite,
    },
  ],
})
```

라우트에 대해 쿠키 재작성을 비활성화하려면 `cookieRewrite: false`를 설정하세요.

하나의 로컬 프록시가 여러 온라인 대상을 가리킬 수 있는 경우, 인증 쿠키에 `localCookieScope: 'target-origin'`을 사용합니다. 프록시는 설정된 쿠키를 대상별 로컬 이름으로 저장하고, 활성 대상의 쿠키만 업스트림으로 전달하며, 활성 범위 쿠키에서 오래된 프론트엔드 CSRF 헤더를 오버라이드할 수 있습니다:

```ts
const cookieRewrite: CookieRewriteOptions = {
  hostPrefixCookies: ['access_token', 'csrf_token', 'refresh_token'],
  localCookieScope: 'target-origin',
  csrfHeader: {
    cookieName: 'csrf_token',
    headerName: 'X-CSRF-Token',
  },
}
```

## 동작

- 프록시는 요청 전달 시 매칭된 경로 접두사를 유지합니다.
- 요청 본문은 스트림으로 전달됩니다.
- 홉바이홉 헤더는 전달 전 제거됩니다.
- 로컬 credentialed CORS 및 프리플라이트 요청은 프록시에서 처리됩니다.
- 라우트 매칭은 명시적이며 선언 순서에 따릅니다.
