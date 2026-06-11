# difyctl

[Dify] 플랫폼을 위한 CLI 클라이언트. 브라우저 Device Flow 로그인, 앱 목록/조회, 구조화된 입력으로 실행, JSON/YAML/텍스트 형식 출력 파싱.

## 설치

빌드는 독립 실행형 바이너리(Bun 컴파일)로 **GitHub Actions 워크플로우 아티팩트**로 게시됩니다 — npm이나 GitHub Release 에셋이 아닙니다. 설치 프로그램은 `main` 브랜치의 최신 성공적인 `cli-release.yml` 실행을 가져와 sha256을 검증하고 바이너리를 `$HOME/.local/bin/difyctl`에 복사합니다.

```sh
# 워크플로우 아티팩트 다운로드에는 퍼블릭 저장소라도 인증이 필요하므로
# GH_TOKEN에 actions:read 스코프가 필요합니다.
export GH_TOKEN=<your-pat>
curl -fsSL https://raw.githubusercontent.com/langgenius/dify/main/cli/scripts/install-cli.sh | sh
```

| 환경변수         | 기본값            | 용도                                                   |
| ---------------- | ----------------- | ------------------------------------------------------ |
| `GH_TOKEN`       | —                 | `actions:read` 권한의 GitHub PAT (또는 `GITHUB_TOKEN`) |
| `DIFYCTL_PREFIX` | `$HOME/.local`    | 설치 루트. 바이너리 위치: `<prefix>/bin/difyctl`       |
| `DIFYCTL_REPO`   | `langgenius/dify` | 소스 저장소                                            |
| `DIFYCTL_BRANCH` | `main`            | 최신 성공 실행을 선택할 브랜치                         |

지원 대상: `darwin-arm64`, `darwin-x64`, `linux-arm64`, `linux-x64`, `windows-x64.exe`. 셸 설치 프로그램은 Linux + macOS를 지원합니다. Windows 사용자는 같은 아티팩트에서 `.exe`를 직접 다운로드할 수 있습니다.

## 빠른 시작

```sh
difyctl auth login                                       # 브라우저 열기; 표시된 디바이스 코드 붙여넣기
difyctl get app                                          # 기본 워크스페이스의 앱 목록
difyctl describe app <app-id>                            # 파라미터 조회
difyctl run app <app-id> "hello"                         # 실행, 블로킹
difyctl run app <app-id> "hello" -o json | jq .answer    # JSON 출력
difyctl run app <app-id> --input name=world --input topic=cats   # 워크플로우 입력
```

추가 문서: `difyctl help account`, `difyctl help external`, `difyctl help environment`, `difyctl help agent`.

## 명령어

전체 명령어 목록은 `difyctl --help`를 실행하세요.
명령어별 참조는 `difyctl <cmd> --help`를 실행하세요.

에이전트(및 스크립팅)용으로는 `difyctl help agent`로 시작하세요 — 교차 명령어 운영 가이드(출력, 검색, 인증, 종료 코드, 에러, HITL, 재시도)입니다. 모든 도움말 표면은 머신 읽기도 가능합니다: `difyctl help -o json`은 전체 명령어 트리와 글로벌 계약(종료 코드, 출력 형식, 에러 봉투, HITL 프로토콜)을 덤프하고, `difyctl <cmd> --help -o json`은 단일 명령어의 디스크립터를 반환합니다.

## 에이전트 스킬

`difyctl skills install`은 로컬 에이전트에 자동 로드되는 단일 순수 위임 `SKILL.md`를 설치합니다. 이 스킬은 명령어 집합을 고정하지 않고 — 에이전트를 `difyctl help -o json`의 라이브 표면으로 안내하므로 바이너리와 절대 어긋나지 않습니다. 체크인이 아닌 바이너리에 내장(버전 스탬프)됩니다.

- `difyctl skills install` — dry-run: 감지된 에이전트(Claude Code, Codex, opencode, Cursor, pi)를 탐지하고 스킬이 위치할 경로를 출력합니다. 아무것도 쓰지 않습니다.
- `difyctl skills install --yes` — 감지된 모든 에이전트에 기록하며 각 경로를 출력합니다. `--agent claude-code[,cursor]`로 하위 집합 제한; `<dir>`로 명시적 디렉토리 강제 지정(에이전트가 감지되지 않을 때 유용).
- `difyctl skills install --stdout` — `SKILL.md`를 stdout에 출력(파이핑 또는 자가 설치용); 아무것도 쓰지 않습니다.

감지는 설정 디렉토리 존재 여부(`~/.claude`, `~/.codex`, `~/.config/opencode`, `~/.cursor`, `~/.pi`)로 수행됩니다. 사본이 오래된 것 같으면 `difyctl version`을 실행하고 `difyctl skills install`을 다시 실행하세요.

## 출력 형식

| 플래그    | 동작                                               |
| --------- | -------------------------------------------------- |
| (없음)    | 사람용 테이블, 컬럼은 터미널에 맞게 자동 조정      |
| `-o wide` | 테이블과 동일, 컬럼 자르기 없음                    |
| `-o json` | pretty-printed JSON, 머신 파싱 가능, 안정적 구조   |
| `-o yaml` | `-o json`의 YAML 미러                              |
| `-o name` | ID만, 줄바꿈 구분 — `xargs`로 파이프 가능          |
| `-o text` | kubectl-describe 스타일 텍스트 (`describe`, `run`) |

에러는 `-o json` 모드에서 stderr로 JSON 봉투를 출력합니다. 그 외에는 사람용 메시지. 종료 코드는 결정론적입니다.

## 설정

| OS      | 설정 경로                                    |
| ------- | -------------------------------------------- |
| Linux   | `${XDG_CONFIG_HOME:-$HOME/.config}/difyctl/` |
| macOS   | `$HOME/.config/difyctl/`                     |
| Windows | `%APPDATA%\difyctl\`                         |

`DIFY_CONFIG_DIR=/some/path`로 오버라이드. 파일은 `0600`, 디렉토리는 `0700`으로 생성. 토큰은 기본적으로 OS 키체인을 사용하며, 키체인이 없는 호스트에서는 sealed 파일로 폴백합니다.

`difyctl`이 읽는 모든 환경 변수는 `difyctl env list`(머신 읽기) 또는 `difyctl help environment`(서술형)로 확인합니다.

## 스트리밍

`run app`은 기본적으로 블로킹 전송을 사용합니다. 오래 실행되는 앱(약 30초 초과 예상)의 경우 `--stream`을 전달하세요:

```sh
difyctl run app app-1 "tell me about cats" --stream
```

에이전트 앱(`mode === 'agent-chat'` 또는 `is_agent` 플래그 설정)은 항상 스트리밍합니다 — Dify 백엔드가 에이전트 모드의 블로킹 요청을 거부합니다. `--stream`과 `-o json` 또는 `-o yaml`을 결합하면 SSE 이벤트를 블로킹 응답과 동일한 봉투 구조로 집계하므로 전송 방식과 관계없이 구조화된 출력이 동일합니다.

## HTTP 재시도

멱등 요청(`GET`, `PUT`, `DELETE`)은 지수 백오프로 일시적 네트워크/DNS 장애 시 재시도합니다. 기본 횟수: **3**. `POST`와 `PATCH`는 부작용 가능성으로 재시도하지 않습니다.

| 설정                     | 효과                                       |
| ------------------------ | ------------------------------------------ |
| `--http-retry <n>`       | 호출별 오버라이드. `0`이면 재시도 비활성화 |
| `DIFYCTL_HTTP_RETRY=<n>` | 프로세스 수준 기본값                       |

해결 순서: 플래그 → 환경변수 → 3.

## 기여

아키텍처 패턴, 스캐폴딩 레시피, 개발 워크플로우는 [`ARD.md`]를 참조하세요.

## 라이선스

Apache-2.0.

[Dify]: https://dify.ai
[`ARD.md`]: ARD.md
