> ## 문서 색인
> 다음 위치에서 전체 문서 색인을 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 웹에서 Claude Code 사용하기

> Anthropic의 샌드박스에서 클라우드 환경, 설정 스크립트, 네트워크 접근 및 Docker를 구성하세요. `--cloud` 및 `--teleport`를 사용하여 웹과 터미널 간에 세션을 이동할 수 있습니다.

<Note>
  웹상의 Claude Code는 Pro, Max 및 Team 사용자, 그리고 프리미엄 시트 또는 Chat + Claude Code 시트를 보유한 Enterprise 사용자를 대상으로 연구 미리보기(research preview) 형태로 제공됩니다.
</Note>

웹상의 Claude Code는 [claude.ai/code](https://claude.ai/code)에서 Anthropic이 관리하는 클라우드 인프라 기반으로 작업을 실행합니다. 브라우저를 닫아도 세션이 유지되며 Claude 모바일 앱에서 세션을 모니터링할 수 있습니다.

<Tip>
  웹상의 Claude Code가 처음이신가요? [시작하기](/docs/en/web-quickstart)에서 GitHub 계정을 연결하고 첫 작업을 제출하는 단계부터 시작하세요.
</Tip>

이 페이지에서 다루는 내용:

* [GitHub 인증 옵션](#github-인증-옵션): GitHub을 연결하는 두 가지 방법
* [클라우드 환경](#클라우드-환경): 이전되는 설정, 설치된 도구 및 환경 구성 방법
* [설정 스크립트](#설정-스크립트) 및 의존성 관리
* [네트워크 접근](#네트워크-접근): 접근 수준, 프록시 및 기본 허용 목록
* `--cloud` 및 `--teleport`를 사용하여 [웹과 터미널 간에 작업 이동](#웹과-터미널-간에-작업-이동)
* [세션 작업](#세션-작업): 검토, 공유, 보관 및 삭제
* [풀 리퀘스트 자동 수정 (Auto-fix)](#풀-리퀘스트-자동-수정-auto-fix): CI 실패 및 리뷰 댓글에 자동으로 대응
* [보안 및 격리](#보안-및-격리): 세션 격리 방식
* [제한 사항](#제한-사항): 속도 제한 및 플랫폼 제약

## GitHub 인증 옵션

클라우드 세션에서 코드를 복제(clone)하고 브랜치를 푸시하려면 GitHub 리포지토리에 대한 접근 권한이 필요합니다. 두 가지 방법으로 권한을 부여할 수 있습니다:

| 방식             | 작동 방식                                                                                   | 적합한 대상                                                           |
| :--------------- | :------------------------------------------------------------------------------------------ | :-------------------------------------------------------------------- |
| **GitHub App**   | [웹 온보딩](/docs/en/web-quickstart) 중에 Claude GitHub App을 승인합니다.                    | 브라우저 온보딩, [Auto-fix](#풀-리퀘스트-자동-수정-auto-fix)를 원하는 팀 |
| **`/web-setup`** | 터미널에서 `/web-setup`을 실행하여 로컬 `gh` CLI 토큰을 Claude 계정과 동기화합니다.         | 이미 `gh`를 사용하고 있는 개별 개발자                                 |

<Note>
  어떤 방식을 사용하든 클라우드 세션은 Claude GitHub App이 설치된 리포지토리뿐만 아니라 연결된 GitHub 계정이 볼 수 있는 모든 리포지토리에 접근할 수 있습니다. App 설치는 [Auto-fix](#풀-리퀘스트-자동-수정-auto-fix)용 PR 웹훅을 활성화하기 위한 것이며 세션 수준의 접근 제어가 아닙니다. 팀이 클라우드 세션에서 접근할 수 있는 리포지토리를 제한하려면 연결된 GitHub 계정의 팀 또는 리포지토리 멤버십을 제한하는 등 GitHub 자체에서 접근을 제한하세요.
</Note>

두 방식 모두 작동합니다. [`/schedule`](/docs/en/routines)은 두 접근 방식 중 하나라도 설정되어 있는지 확인하고 둘 다 구성되어 있지 않은 경우 `/web-setup`을 실행하도록 안내합니다. `/web-setup` 가이드는 [터미널에서 연결](/docs/en/web-quickstart#connect-from-your-terminal)을 참조하세요.

PR 웹훅을 수신하는 [Auto-fix](#풀-리퀘스트-자동-수정-auto-fix)에는 GitHub App이 필요합니다. `/web-setup`으로 연결한 후 나중에 Auto-fix를 원하는 경우 해당 리포지토리에 App을 설치하세요.

Team 및 Enterprise 소유자(Owner)는 [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)의 Quick web setup 토글을 통해 `/web-setup`을 비활성화할 수 있습니다.

<Note>
  [데이터 보존 없음 (Zero Data Retention)](/docs/en/zero-data-retention)이 활성화된 조직은 `/web-setup` 또는 기타 클라우드 세션 기능을 사용할 수 없습니다.
</Note>

## 클라우드 환경

각 세션은 리포지토리가 복제된 깨끗한 Anthropic 관리 가상 머신(VM)에서 실행됩니다. 이 섹션에서는 세션 시작 시 사용 가능한 항목과 이를 맞춤 구성하는 방법을 설명합니다.

### 클라우드 세션에서 사용 가능한 항목

클라우드 세션은 리포지토리의 새로 복제된 상태에서 시작됩니다. 리포지토리에 커밋된 모든 내용을 사용할 수 있습니다. 본인의 머신에만 설치하거나 설정한 내용은 세션에서 사용할 수 없습니다. 조직의 정책은 [서버 관리형 설정](/docs/en/server-managed-settings)을 통해 별도로 전달됩니다.

|                                                                                                                                                                                           | 클라우드 세션 제공 여부     | 이유                                                                                                                                                                                                                                                                                                                  |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 리포지토리의 `CLAUDE.md`                                                                                                                                                                  | 예                          | 복제본의 일부임                                                                                                                                                                                                                                                                                                      |
| 리포지토리의 `.claude/settings.json` 훅                                                                                                                                                   | 예                          | 복제본의 일부임                                                                                                                                                                                                                                                                                                      |
| 리포지토리의 `.mcp.json` MCP 서버                                                                                                                                                         | 예                          | 복제본의 일부임                                                                                                                                                                                                                                                                                                      |
| 리포지토리의 `.claude/rules/`                                                                                                                                                             | 예                          | 복제본의 일부임                                                                                                                                                                                                                                                                                                      |
| 리포지토리의 `.claude/skills/`, `.claude/agents/`, `.claude/commands/`                                                                                                                     | 예                          | 복제본의 일부임                                                                                                                                                                                                                                                                                                      |
| `.claude/settings.json`에 선언된 플러그인                                                                                                                                                 | 예                          | 선언한 [마켓플레이스](/docs/en/plugin-marketplaces)에서 세션 시작 시 설치됩니다. 마켓플레이스 소스에 접근하기 위한 네트워크 연결이 필요합니다.                                                                                                                                                                       |
| 조직의 [서버 관리형 설정](/docs/en/server-managed-settings)                                                                                                                               | 예                          | 세션이 시작될 때 Anthropic 서버에서 가져옵니다. 클라우드 세션에서 `availableModels`가 적용되는 방식은 [표면 지원 범위](/docs/en/model-config#surface-coverage)를 참조하세요. MDM 또는 관리형 설정 파일을 통해 기기에 배포된 설정은 세션이 Anthropic 관리 VM에서 실행되므로 적용되지 않습니다.                        |
| 사용자 `~/.claude/CLAUDE.md`                                                                                                                                                              | 아니요                      | 리포지토리가 아닌 본인의 머신에 존재함                                                                                                                                                                                                                                                                               |
| 사용자 `~/.claude/skills/`, `~/.claude/agents/`, `~/.claude/commands/`                                                                                                                  | 아니요                      | 리포지토리가 아닌 본인의 머신에 존재함. 리포지토리의 `.claude/` 디렉터리에 커밋하세요. claude.ai에서 활성화한 스킬은 클라우드 세션으로 자동 로드됩니다.                                                                                                                                                               |
| 사용자 설정에서만 활성화된 플러그인                                                                                                                                                       | 아니요                      | 사용자 범위의 `enabledPlugins`는 `~/.claude/settings.json`에 존재함. 대신 리포지토리의 `.claude/settings.json`에 선언하세요.                                                                                                                                                                                         |
| `claude mcp add`로 추가한 MCP 서버                                                                                                                                                        | 아니요                      | 리포지토리가 아닌 로컬 사용자 설정에 기록됨. 대신 [`.mcp.json`](/docs/en/mcp#project-scope)에 서버를 선언하세요.                                                                                                                                                                                                    |
| 리포지토리의 `.claude/settings.json` `env` 블록에 포함된 전송 변수(`NODE_EXTRA_CA_CERTS` 및 [mTLS 클라이언트 인증서 변수](/docs/en/network-config#mtls-authentication) 등)               | 아니요                      | 호스팅 환경이 세션의 API 연결을 관리하므로 Claude Code는 이 키들을 무시하고 무시된 각 키를 세션 디버그 로그에 기록합니다.                                                                                                                                                                                            |
| 정적 API 토큰 및 자격 증명                                                                                                                                                                | 아니요                      | 전용 비밀 값 저장소가 아직 존재하지 않습니다. 아래 내용을 참조하세요.                                                                                                                                                                                                                                                |
| AWS SSO 같은 대화형 인증                                                                                                                                                                  | 아니요                      | 지원되지 않음. SSO는 클라우드 세션에서 실행할 수 없는 브라우저 기반 로그인이 필요합니다.                                                                                                                                                                                                                             |

자신의 설정을 클라우드 세션에서 사용할 수 있도록 하려면 리포지토리에 커밋하세요. 조직 정책은 [서버 관리형 설정](/docs/en/server-managed-settings)을 통해 별도로 적용됩니다.

전용 비밀 값 저장소는 아직 제공되지 않습니다. 환경 변수와 설정 스크립트 모두 해당 환경을 편집할 수 있는 사람에게 표시되는 환경 구성에 저장됩니다. 클라우드 세션에 비밀 값이 필요한 경우 이러한 노출 가능성을 염두에 두고 환경 변수로 추가하세요.

### 설치된 도구

클라우드 세션에는 공통 언어 런타임, 빌드 도구 및 데이터베이스가 사전 설치되어 제공됩니다. 아래 표는 카테고리별 포함 항목을 요약한 것입니다.

| 카테고리      | 포함 항목                                                                          |
| :------------ | :--------------------------------------------------------------------------------- |
| **Python**    | Python 3.x (pip, poetry, uv, black, mypy, pytest, ruff 포함)                      |
| **Node.js**   | nvm을 통한 20, 21, 22 (npm, yarn, pnpm, bun¹, eslint, prettier, chromedriver 포함) |
| **Ruby**      | 3.1, 3.2, 3.3 (gem, bundler, rbenv 포함)                                           |
| **PHP**       | 8.4 (Composer 포함)                                                                |
| **Java**      | OpenJDK 21 (Maven 및 Gradle 포함)                                                  |
| **Go**        | 모듈 지원이 포함된 최신 안정 버전                                                 |
| **Rust**      | rustc 및 cargo                                                                     |
| **C/C++**     | GCC, Clang, cmake, ninja, conan                                                    |
| **Docker**    | docker, dockerd, docker compose                                                    |
| **Databases** | PostgreSQL 16, Redis 7.0                                                           |
| **Utilities** | git, jq, yq, ripgrep, tmux, vim, nano                                              |

¹ Bun이 설치되어 있지만 패키지 수신과 관련해 알려진 [프록시 호환성 문제](#sessionstart-훅으로-의존성-설치)가 있습니다.

정확한 버전을 확인하려면 클라우드 세션에서 Claude에게 `check-tools`를 실행하도록 요청하세요. 이 명령어는 클라우드 세션에만 존재합니다.

### GitHub 이슈 및 풀 리퀘스트 작업

클라우드 세션에는 별도의 설정 없이도 Claude가 이슈를 읽고, 풀 리퀘스트를 나열하고, 디프(diff)를 가져오고, 댓글을 게시할 수 있는 내장 GitHub 도구가 포함되어 있습니다. 이러한 도구는 [GitHub 인증 옵션](#github-인증-옵션)에서 설정한 방식을 사용하여 [GitHub 프록시](#github-프록시)를 통해 인증하므로 토큰이 컨테이너 내부로 직접 들어가지 않습니다.

[환경 구성](#환경-구성)에서 `GH_TOKEN` 또는 `GITHUB_TOKEN`을 직접 설정하거나 둘 다 설정하지 않은 채 [GitHub 프록시](#github-프록시)가 인증을 대신하도록 둘 수 있습니다:

* 토큰을 설정하면 컨테이너로 그대로 전달되므로 `gh` 및 사용자 스크립트가 이를 직접 사용합니다.
* 둘 다 설정하지 않으면 컨테이너는 두 변수를 더미 문자열인 `proxy-injected`로 설정하고 프록시가 아웃바운드 GitHub 요청에 실제 자격 증명을 대신 채워 넣습니다. `gh`는 별도 토큰 없이도 작동하지만 `GITHUB_TOKEN`을 직접 읽는 스크립트는 실제 토큰이 아닌 플레이스홀더를 얻게 됩니다.

본인의 세션에 어떤 경우가 적용되는지 확인하려면 Claude에게 `echo $GH_TOKEN`을 실행하도록 요청하세요.

`gh` CLI는 사전 설치되어 있지 않습니다. 내장 도구가 지원하지 않는 `gh release` 또는 `gh workflow run` 같은 `gh` 명령어가 필요한 경우 직접 설치하고 인증하세요:

<Steps>
  <Step title="설정 스크립트에 gh 설치 추가">
    [설정 스크립트](#설정-스크립트)에 `apt update && apt install -y gh`를 추가하세요.
  </Step>

  <Step title="프록시가 인증을 처리하지 않는 경우 토큰 제공">
    `echo $GH_TOKEN` 실행 결과가 `proxy-injected`로 출력되면 [GitHub 프록시](#github-프록시)가 `gh` 인증을 대신 처리하므로 이 단계가 필요하지 않습니다. 그렇지 않은 경우, GitHub 개인 액세스 토큰(PAT)을 담아 [환경 구성](#환경-구성)에 `GH_TOKEN` 환경 변수를 추가하세요. `gh`가 `GH_TOKEN`을 자동으로 읽으므로 `gh auth login` 단계가 필요하지 않습니다.
  </Step>
</Steps>

### 출력을 세션으로 역링크

각 클라우드 세션은 claude.ai에 트랜스크립트 URL을 가지며, 세션은 `CLAUDE_CODE_REMOTE_SESSION_ID` 환경 변수에서 자체 ID를 읽을 수 있습니다. 리뷰어가 해당 결과를 생성한 실행 환경을 열어볼 수 있도록 PR 본문, 커밋 메시지, Slack 게시물 또는 생성된 보고서에 추적 가능한 링크를 넣는 데 이를 활용하세요.

v2.1.179부터 Claude가 웹 세션에서 생성하는 커밋에는 `Claude-Session: <url>` git 트레일러가 포함되고 PR 본문에는 별도 행에 세션 URL이 포함됩니다. {/* min-version: 2.1.182 */}v2.1.182부터는 [`attribution.sessionUrl`](/docs/en/settings#attribution-settings)을 `false`로 설정하여 트레일러 및 PR 본문 링크를 생략할 수 있습니다.

Slack 메시지나 보고서 파일처럼 커밋이나 PR 이외의 항목에 세션 링크를 포함하려면 Claude가 다음 명령어를 실행하고 그 출력을 사용하도록 하세요. 이 명령어는 환경 변수 값의 `cse_` 접두사를 트랜스크립트 URL이 필요로 하는 `session_` 접두사로 변환합니다:

```bash theme={null}
echo "https://claude.ai/code/${CLAUDE_CODE_REMOTE_SESSION_ID/#cse_/session_}"
```

### 테스트 실행, 서비스 시작 및 패키지 추가

Claude는 작업 과정의 일부로 테스트를 실행합니다. 프롬프트에서 "tests/의 실패하는 테스트 수정해줘" 또는 "변경할 때마다 pytest 실행해줘"와 같이 요청하세요. pytest, jest, cargo test 같은 테스트 러너는 사전 설치되어 있으며 추가 설정 없이 작동합니다.

PostgreSQL 및 Redis는 사전 설치되어 있지만 기본적으로 실행 중이지 않습니다. 세션 중에 Claude에게 각각을 시작하도록 요청하세요:

```bash theme={null}
service postgresql start
```

```bash theme={null}
service redis-server start
```

컨테이너화된 서비스를 실행하기 위해 Docker를 사용할 수 있습니다. Claude에게 `docker compose up`을 실행하여 프로젝트의 서비스를 시작하도록 요청하세요. 이미지를 가져오기 위한 네트워크 접근은 해당 환경의 [접근 수준](#접근-수준)을 따르며, [기본 허용 목록](#기본-허용-도메인)에는 Docker Hub 및 기타 공통 레지스트리가 포함되어 있습니다.

이미지 용량이 크거나 가져오는 데 시간이 걸리는 경우 [설정 스크립트](#설정-스크립트)에 `docker compose pull` 또는 `docker compose build`를 추가하세요. 가져온 이미지는 [캐시된 환경](#환경-캐싱)에 저장되므로 새 세션마다 디스크에 이미 준비되어 있습니다. 캐시는 실행 중인 프로세스가 아닌 파일만 저장하므로 Claude는 세션마다 컨테이너를 새로 시작합니다.

사전 설치되지 않은 패키지를 추가하려면 [설정 스크립트](#설정-스크립트)를 사용하세요. 스크립트의 출력은 [캐시](#환경-캐싱)되므로 설치된 패키지는 매번 재설치할 필요 없이 모든 세션 시작 시 즉시 사용할 수 있습니다. 세션 중간에 Claude에게 패키지 설치를 요청할 수도 있지만, 이러한 설치는 다른 세션으로 유지되지 않습니다.

### 리소스 제한

클라우드 세션은 시간에 따라 변경될 수 있는 대략적인 리소스 상한선 아래에서 실행됩니다:

* 4 vCPU
* 16 GB RAM
* 30 GB 디스크

대규모 빌드 작업이나 메모리 집약적인 테스트와 같이 상당히 많은 메모리가 필요한 작업은 실패하거나 종료될 수 있습니다. 이러한 제한을 초과하는 워크로드의 경우 [Remote Control](/docs/en/remote-control)을 사용하여 자체 하드웨어에서 Claude Code를 실행하세요.

### 환경 구성

환경은 [네트워크 접근](#네트워크-접근), 환경 변수 및 세션이 시작되기 전에 실행되는 [설정 스크립트](#설정-스크립트)를 제어합니다. 별도 설정 없이 사용 가능한 항목은 [설치된 도구](#설치된-도구)를 참조하세요. 웹 인터페이스 또는 터미널에서 환경을 관리할 수 있습니다:

| 작업                                               | 방법                                                                                                                                                                                                                     |
| :------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 환경 추가                                          | 현재 환경을 선택하여 선택기를 연 다음 **Add environment**를 선택합니다. 대화 상자에는 이름, 네트워크 접근 수준, 환경 변수 및 설정 스크립트가 포함됩니다.                                                                 |
| 환경 편집                                          | 현재 환경의 이름이 표시된 클라우드 아이콘을 선택하여 선택기를 열고, 환경 위에 마우스를 올린 다음 우측에 나타나는 설정 아이콘을 클릭합니다.                                                                               |
| 환경 보관                                          | 편집할 환경을 열고 **Archive**를 선택합니다. 보관된 환경은 선택기에서 숨겨지지만 기존 세션은 계속 실행됩니다.                                                                                                           |
| CLI 클라우드 세션용 기본 환경 설정                 | 터미널에서 `/remote-env`를 실행합니다. 환경이 하나만 있는 경우 이 명령어는 현재 설정을 보여줍니다. `/remote-env`는 기본값 선택만 수행하며, 환경의 추가, 편집 및 보관은 웹 인터페이스에서 수행하세요.                    |

환경 변수는 행당 하나의 `KEY=value` 쌍이 들어가는 `.env` 형식을 사용합니다. 따옴표는 값의 일부로 저장되므로 감싸지 마세요. 다음 예제는 3개의 변수를 정의합니다:

```text theme={null}
NODE_ENV=development
LOG_LEVEL=debug
DATABASE_URL=postgres://localhost:5432/myapp
```

### 조직 공유 환경

Team 및 Enterprise 플랜의 소유자(Owner) 및 관리자(Admin)는 조직의 모든 구성원과 공유되는 클라우드 환경을 생성할 수 있습니다. 공유 환경은 각 구성원의 개인 환경과 함께 환경 선택기에 나타나므로 팀원이 각각 재생성할 필요 없이 하나의 설정으로 표준화할 수 있습니다.

[관리자 설정](https://claude.ai/admin-settings)의 **Cloud environments** 페이지에서 공유 환경을 관리하세요. 다음 작업이 가능합니다:

* 공유 환경 생성, 편집 및 보관. 각 환경은 개인 환경과 동일한 필드(이름, [네트워크 접근 수준](#접근-수준), `.env` 형식의 [환경 변수](#환경-구성) 및 [설정 스크립트](#설정-스크립트))를 갖습니다.
* 조직의 기본 환경 설정.

공유 환경의 값은 해당 환경을 사용하는 모든 구성원의 세션에 전달됩니다. 개인 환경과 마찬가지로 공유 환경에도 전용 비밀 값 저장소가 없으므로 비밀 값을 포함하지 마세요.

자체 호스팅 러너 프로그램에 참여하는 조직도 동일한 페이지에서 러너 풀을 관리합니다.

## 설정 스크립트

설정 스크립트는 Claude Code가 실행되기 전, 새 클라우드 세션이 시작될 때 실행되는 Bash 스크립트입니다. 설정 스크립트를 사용하여 의존성을 설치하고, 도구를 구성하거나, 사전 설치되지 않은 세션 필요 항목을 가져오세요.

스크립트는 Ubuntu 24.04의 root 권한으로 실행되므로 `apt install` 및 대부분의 언어 패키지 관리자가 작동합니다.

설정 스크립트를 추가하려면 환경 설정 대화 상자를 열고 **Setup script** 필드에 스크립트를 입력하세요.

다음 예제는 사전 설치되어 있지 않은 `gh` CLI를 설치합니다:

```bash theme={null}
#!/bin/bash
apt update && apt install -y gh
```

스크립트가 0이 아닌 값을 반환하고 종료되면 세션 시작이 실패합니다. 일시적인 설치 실패로 세션이 차단되는 것을 방지하려면 중요하지 않은 명령 뒤에 `|| true`를 붙이세요.

[환경 캐시](#환경-캐싱)가 구축될 수 있도록 스크립트의 전체 실행 시간을 5분 미만으로 유지하세요. 독립적인 설치는 `&` 및 `wait`를 사용하여 병렬로 실행하세요. 단일 다운로드가 5분 제한에 맞지 않는 경우 배경에서 실행되는 [SessionStart 훅](#설정-스크립트-vs-sessionstart-훅)으로 이동하세요.

<Note>
  패키지를 설치하는 설정 스크립트는 레지스트리에 도달하기 위한 네트워크 접근이 필요합니다. 기본 **Trusted** 네트워크 접근은 npm, PyPI, RubyGems 및 crates.io를 포함한 [공통 패키지 레지스트리](#기본-허용-도메인)에 대한 연결을 허용합니다. 환경이 **None** 네트워크 접근을 사용하는 경우 스크립트의 패키지 설치가 실패합니다.
</Note>

### 환경 캐싱

설정 스크립트는 환경에서 세션을 처음 시작할 때 실행됩니다. 완료되면 Anthropic이 파일시스템 스냅샷을 찍고 이후 세션의 시작 지점으로 해당 스냅샷을 재사용합니다. 새 세션은 의존성, 도구 및 Docker 이미지가 디스크에 이미 준비된 상태로 시작되며 설정 스크립트 단계는 건너뜁니다. 이렇게 하면 스크립트가 대규모 툴체인을 설치하거나 컨테이너 이미지를 가져오더라도 빠른 시작 속도를 유지합니다.

캐시는 실행 중인 프로세스가 아닌 파일을 캡처합니다. 설정 스크립트가 디스크에 작성한 모든 내용은 이전됩니다. 스크립트가 시작한 서비스나 컨테이너는 이전되지 않으므로 Claude에게 요청하거나 [SessionStart 훅](#설정-스크립트-vs-sessionstart-훅)을 통해 세션마다 시작하세요.

환경의 설정 스크립트나 허용된 네트워크 호스트를 변경할 때, 그리고 약 7일 후 캐시 만료에 도달할 때 캐시를 다시 구축하기 위해 설정 스크립트가 재실행됩니다. 기존 세션을 다시 시작(resume)할 때는 설정 스크립트가 재실행되지 않습니다.

캐싱을 직접 활성화하거나 스냅샷을 직접 관리할 필요는 없습니다.

### 설정 스크립트 vs. SessionStart 훅

언어 런타임이나 CLI 도구처럼 랩톱에는 이미 있지만 클라우드 환경에 필요한 것을 설치할 때는 설정 스크립트를 사용하세요. `npm install`처럼 클라우드와 로컬 모두에서 실행되어야 하는 프로젝트 설정에는 [SessionStart 훅](/docs/en/hooks#sessionstart)을 사용하세요.

둘 다 세션 시작 시 실행되지만 소속이 다릅니다:

|               | 설정 스크립트                                                                               | SessionStart 훅                                                              |
| ------------- | -------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| 소속          | 클라우드 환경                                                                                | 리포지토리                                                                   |
| 설정 위치     | 클라우드 환경 UI                                                                             | 리포지토리의 `.claude/settings.json`                                         |
| 실행 시점     | Claude Code가 실행되기 전, [캐시된 환경](#환경-캐싱)을 이용할 수 없는 경우                   | Claude Code가 실행된 후, 다시 시작된 세션을 포함한 모든 세션에서             |
| 범위          | 클라우드 환경 전용                                                                           | 로컬 및 클라우드 모두                                                        |

SessionStart 훅은 로컬의 사용자 수준 `~/.claude/settings.json`에서도 정의할 수 있지만, 사용자 수준 설정은 클라우드 세션으로 이전되지 않습니다. 클라우드에서 훅은 리포지토리 및 조직의 [서버 관리형 설정](/docs/en/server-managed-settings)에서 가져옵니다.

### SessionStart 훅으로 의존성 설치

클라우드 세션에서만 의존성을 설치하려면 리포지토리의 `.claude/settings.json`에 SessionStart 훅을 추가하세요:

```json theme={null}
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/scripts/install_pkgs.sh"
          }
        ]
      }
    ]
  }
}
```

`scripts/install_pkgs.sh` 위치에 스크립트를 생성하세요. `CLAUDE_CODE_REMOTE` 환경 변수는 클라우드 세션에서 `true`로 설정되므로 이를 사용하여 로컬 실행을 건너뛸 수 있습니다:

```bash theme={null}
#!/bin/bash

if [ "$CLAUDE_CODE_REMOTE" != "true" ]; then
  exit 0
fi

npm install
pip install -r requirements.txt
exit 0
```

그런 다음 스크립트에 실행 권한을 부여하세요:

```bash theme={null}
chmod +x scripts/install_pkgs.sh
```

클라우드 세션에서 SessionStart 훅은 몇 가지 제한 사항이 있습니다:

* **클라우드 전용 범위 없음**: 훅은 로컬 및 클라우드 세션 모두에서 실행됩니다. 로컬 실행을 건너뛰려면 위에서 작성한 대로 `CLAUDE_CODE_REMOTE` 환경 변수를 확인하세요.
* **네트워크 접근 필요**: 설치 명령어는 패키지 레지스트리에 도달해야 합니다. 환경이 **None** 네트워크 접근을 사용하는 경우 이 훅은 실패합니다. **Trusted** 아래의 [기본 허용 목록](#기본-허용-도메인)에는 npm, PyPI, RubyGems 및 crates.io가 포함됩니다.
* **프록시 호환성**: 모든 아웃바운드 트래픽은 [보안 프록시](#보안-프록시)를 거칩니다. 일부 패키지 관리자는 이 프록시와 올바르게 작동하지 않으며, Bun이 대표적인 예입니다.
* **시작 지연 시간 추가**: [환경 캐싱](#환경-캐싱)의 이점을 얻는 설정 스크립트와 달리 훅은 세션이 시작되거나 다시 시작될 때마다 실행됩니다. 재설치 전에 의존성이 이미 존재하는지 확인하여 설치 스크립트 실행 속도를 빠르게 유지하세요.

이후의 Bash 명령에 대해 환경 변수를 유지하려면 `$CLAUDE_ENV_FILE` 위치의 파일에 작성하세요. 자세한 내용은 [SessionStart 훅](/docs/en/hooks#sessionstart)을 참조하세요.

베이스 이미지를 사용자 정의 Docker 이미지로 교체하는 것은 아직 지원되지 않습니다. 설정 스크립트를 사용하여 [제공되는 이미지](#설치된-도구) 위에 필요한 것을 설치하거나 `docker compose`를 사용하여 Claude와 함께 컨테이너로 이미지를 실행하세요.

## 네트워크 접근

네트워크 접근은 클라우드 환경에서 나가는 아웃바운드 연결을 제어합니다. 각 환경은 하나의 접근 수준을 지정하며 커스텀 허용 도메인으로 이를 확장할 수 있습니다. 기본값은 패키지 레지스트리 및 기타 [허용 목록 도메인](#기본-허용-도메인)을 허용하는 **Trusted**입니다.

환경의 네트워크 접근을 변경하려면 [편집을 위해 환경을 열고](#환경-구성) 대화 상자의 **Network access** 선택기를 사용하세요. 별도의 환경 페이지는 없습니다. 클라우드 아이콘은 클라우드 세션을 시작하거나 [루틴](/docs/en/routines#environments-and-network-access)을 구성하는 위치라면 어디서나 나타납니다.

<Note>
  MCP 커넥터 트래픽은 Anthropic 서버를 통해 라우팅되므로 세션이나 루틴에서 활성화한 커넥터는 호스트를 **Allowed domains**에 추가하지 않아도 작동합니다. 커넥터는 세션별 또는 루틴별로 구성됩니다. Claude가 접근할 수 있는 도구를 제한하려면 필요하지 않은 커넥터를 제거하세요. 이는 [보안 및 격리](#보안-및-격리)에 설명된 것과 동일한 Anthropic 전용 채널에 의존합니다.
</Note>

### 접근 수준

환경을 생성하거나 편집할 때 접근 수준을 선택하세요:

| 수준        | 아웃바운드 연결                                                                               |
| :---------- | :------------------------------------------------------------------------------------------- |
| **None**    | 아웃바운드 네트워크 접근 불가                                                                 |
| **Trusted** | [허용 목록 도메인](#기본-허용-도메인)만 허용 (패키지 레지스트리, GitHub, 클라우드 SDK 등)    |
| **Full**    | 모든 도메인 허용                                                                             |
| **Custom**  | 기본값을 옵션으로 포함하는 사용자 정의 허용 목록                                             |

GitHub 작업은 이 설정과 독립된 [별도의 프록시](#github-프록시)를 사용합니다.

### 특정 도메인 허용

Trusted 목록에 없는 도메인을 허용하려면 환경의 네트워크 접근 설정에서 **Custom**을 선택하세요. **Allowed domains** 필드가 나타납니다. 행당 하나의 도메인을 입력하세요:

```text theme={null}
api.example.com
*.internal.example.com
registry.example.com
```

와일드카드 서브도메인 매칭을 위해 `*.`를 사용하세요. 커스텀 항목과 함께 [Trusted 도메인](#기본-허용-도메인)을 계속 유지하려면 **Also include default list of common package managers**를 체크하고, 목록에 직접 작성한 항목만 허용하려면 체크를 해제하세요.

허용 도메인은 환경별로 구성됩니다. 소유자(Owner)가 모든 사용자 환경에 적용할 수 있는 조직 수준의 허용 목록은 없으며, [서버 관리형 설정](/docs/en/server-managed-settings)은 클라우드 세션을 제한할 수는 있지만 허용 도메인을 추가할 수는 없습니다.

### GitHub 프록시

보안을 위해 모든 GitHub 작업은 실제 GitHub 자격 증명을 샌드박스 외부에 유지하는 전용 프록시 서비스를 거칩니다. 프록시는 두 가지 종류의 트래픽을 인증합니다:

* Git 상호작용: 샌드박스 내부의 git 클라이언트는 커스텀 구축된 범위 지정 자격 증명을 사용하며, 프록시가 이를 검증하고 실제 GitHub 인증 토큰으로 변환합니다.
* GitHub API 요청: 프록시는 내장 GitHub 도구에서 보내는 요청 및 세션이 [GitHub 이슈 및 풀 리퀘스트 작업](#github-이슈-및-풀-리퀘스트-작업)에 설명된 `proxy-injected` 더미 값을 설정할 때 `gh`에서 보내는 요청에 대해 실제 자격 증명을 대신 채워 넣습니다.

또한 프록시는 안전을 위해 git push 작업을 현재 작업 브랜치로 제한하며, 보안 경계를 유지하면서 복제, 가져오기 및 PR 작업을 활성화합니다.

환경의 [네트워크 접근 수준](#접근-수준)과 관계없이 프록시는 GitHub API 및 릴리스 아셋 요청을 세션에 연결된 리포지토리로 제한합니다. 연결되지 않은 리포지토리에서 릴리스 아셋을 다운로드하는 설정 스크립트는 403을 반환합니다. 공개 리포지토리의 커밋된 파일은 [보안 프록시](#보안-프록시)가 대신 처리하는 `raw.githubusercontent.com`을 통해 가져옵니다. 해당 도메인은 기본 [Trusted 목록](#기본-허용-도메인)에 포함되어 있으므로 환경의 [접근 수준](#접근-수준)에서 이를 제외하지 않는 한 파일에 도달할 수 있습니다.

### 보안 프록시

환경은 보안 및 오남용 방지 목적으로 HTTP/HTTPS 네트워크 프록시 뒤에서 실행됩니다. 모든 아웃바운드 인터넷 트래픽은 이 프록시를 통과하며 다음 기능을 제공합니다:

* 악의적인 요청 방어
* 속도 제한 및 오남용 방지
* 보안 강화를 위한 콘텐츠 필터링
* 요청된 호스트 이름의 DNS 수준 감사 기록

### 기본 허용 도메인

**Trusted** 네트워크 접근을 사용할 때 다음 도메인이 기본적으로 허용됩니다. `*` 표시가 된 도메인은 와일드카드 서브도메인 매칭을 의미하므로 `*.gcr.io`는 `gcr.io` 하위의 모든 서브도메인을 허용합니다.

<AccordionGroup>
  <Accordion title="Anthropic 서비스">
    * api.anthropic.com
    * statsig.anthropic.com
    * docs.claude.com
    * platform.claude.com
    * code.claude.com
    * claude.ai
  </Accordion>

  <Accordion title="버전 제어">
    * github.com
    * [www.github.com](http://www.github.com)
    * api.github.com
    * npm.pkg.github.com
    * raw\.githubusercontent.com
    * pkg-npm.githubusercontent.com
    * objects.githubusercontent.com
    * release-assets.githubusercontent.com
    * codeload.github.com
    * avatars.githubusercontent.com
    * camo.githubusercontent.com
    * gist.github.com
    * gitlab.com
    * [www.gitlab.com](http://www.gitlab.com)
    * registry.gitlab.com
    * bitbucket.org
    * [www.bitbucket.org](http://www.bitbucket.org)
    * api.bitbucket.org
  </Accordion>

  <Accordion title="컨테이너 레지스트리">
    * registry-1.docker.io
    * auth.docker.io
    * index.docker.io
    * hub.docker.com
    * [www.docker.com](http://www.docker.com)
    * production.cloudflare.docker.com
    * download.docker.com
    * gcr.io
    * \*.gcr.io
    * ghcr.io
    * mcr.microsoft.com
    * \*.data.mcr.microsoft.com
    * public.ecr.aws
  </Accordion>

  <Accordion title="클라우드 플랫폼">
    * cloud.google.com
    * accounts.google.com
    * gcloud.google.com
    * \*.googleapis.com
    * storage.googleapis.com
    * compute.googleapis.com
    * container.googleapis.com
    * azure.com
    * portal.azure.com
    * microsoft.com
    * [www.microsoft.com](http://www.microsoft.com)
    * \*.microsoftonline.com
    * packages.microsoft.com
    * dotnet.microsoft.com
    * dot.net
    * visualstudio.com
    * dev.azure.com
    * \*.amazonaws.com
    * \*.api.aws
    * oracle.com
    * [www.oracle.com](http://www.oracle.com)
    * java.com
    * [www.java.com](http://www.java.com)
    * java.net
    * [www.java.net](http://www.java.net)
    * download.oracle.com
    * yum.oracle.com
  </Accordion>

  <Accordion title="JavaScript 및 Node 패키지 관리자">
    * registry.npmjs.org
    * [www.npmjs.com](http://www.npmjs.com)
    * [www.npmjs.org](http://www.npmjs.org)
    * npmjs.com
    * npmjs.org
    * yarnpkg.com
    * registry.yarnpkg.com
  </Accordion>

  <Accordion title="Python 패키지 관리자">
    * pypi.org
    * [www.pypi.org](http://www.pypi.org)
    * files.pythonhosted.org
    * pythonhosted.org
    * test.pypi.org
    * pypi.python.org
    * pypa.io
    * [www.pypa.io](http://www.pypa.io)
  </Accordion>

  <Accordion title="Ruby 패키지 관리자">
    * rubygems.org
    * [www.rubygems.org](http://www.rubygems.org)
    * api.rubygems.org
    * index.rubygems.org
    * ruby-lang.org
    * [www.ruby-lang.org](http://www.ruby-lang.org)
    * rubyforge.org
    * [www.rubyforge.org](http://www.rubyforge.org)
    * rubyonrails.org
    * [www.rubyonrails.org](http://www.rubyonrails.org)
    * rvm.io
    * get.rvm.io
  </Accordion>

  <Accordion title="Rust 패키지 관리자">
    * crates.io
    * [www.crates.io](http://www.crates.io)
    * index.crates.io
    * static.crates.io
    * rustup.rs
    * static.rust-lang.org
    * [www.rust-lang.org](http://www.rust-lang.org)
  </Accordion>

  <Accordion title="Go 패키지 관리자">
    * proxy.golang.org
    * sum.golang.org
    * index.golang.org
    * golang.org
    * [www.golang.org](http://www.golang.org)
    * goproxy.io
    * pkg.go.dev
  </Accordion>

  <Accordion title="JVM 패키지 관리자">
    * maven.org
    * repo.maven.org
    * central.maven.org
    * repo1.maven.org
    * repo.maven.apache.org
    * jcenter.bintray.com
    * gradle.org
    * [www.gradle.org](http://www.gradle.org)
    * services.gradle.org
    * plugins.gradle.org
    * kotlinlang.org
    * [www.kotlinlang.org](http://www.kotlinlang.org)
    * spring.io
    * repo.spring.io
  </Accordion>

  <Accordion title="기타 패키지 관리자">
    * packagist.org (PHP Composer)
    * [www.packagist.org](http://www.packagist.org)
    * repo.packagist.org
    * nuget.org (.NET NuGet)
    * [www.nuget.org](http://www.nuget.org)
    * api.nuget.org
    * pub.dev (Dart/Flutter)
    * api.pub.dev
    * hex.pm (Elixir/Erlang)
    * [www.hex.pm](http://www.hex.pm)
    * cpan.org (Perl CPAN)
    * [www.cpan.org](http://www.cpan.org)
    * metacpan.org
    * [www.metacpan.org](http://www.metacpan.org)
    * api.metacpan.org
    * cocoapods.org (iOS/macOS)
    * [www.cocoapods.org](http://www.cocoapods.org)
    * cdn.cocoapods.org
    * haskell.org
    * [www.haskell.org](http://www.haskell.org)
    * hackage.haskell.org
    * swift.org
    * [www.swift.org](http://www.swift.org)
  </Accordion>

  <Accordion title="Linux 배포판">
    * archive.ubuntu.com
    * security.ubuntu.com
    * ubuntu.com
    * [www.ubuntu.com](http://www.ubuntu.com)
    * \*.ubuntu.com
    * ppa.launchpad.net
    * launchpad.net
    * [www.launchpad.net](http://www.launchpad.net)
    * \*.nixos.org
  </Accordion>

  <Accordion title="개발 도구 및 플랫폼">
    * dl.k8s.io (Kubernetes)
    * pkgs.k8s.io
    * k8s.io
    * [www.k8s.io](http://www.k8s.io)
    * releases.hashicorp.com (HashiCorp)
    * apt.releases.hashicorp.com
    * rpm.releases.hashicorp.com
    * archive.releases.hashicorp.com
    * hashicorp.com
    * [www.hashicorp.com](http://www.hashicorp.com)
    * repo.anaconda.com (Anaconda/Conda)
    * conda.anaconda.org
    * anaconda.org
    * [www.anaconda.com](http://www.anaconda.com)
    * anaconda.com
    * continuum.io
    * apache.org (Apache)
    * [www.apache.org](http://www.apache.org)
    * archive.apache.org
    * downloads.apache.org
    * eclipse.org (Eclipse)
    * [www.eclipse.org](http://www.eclipse.org)
    * download.eclipse.org
    * nodejs.org (Node.js)
    * [www.nodejs.org](http://www.nodejs.org)
    * developer.apple.com
    * developer.android.com
    * pkg.stainless.com
    * binaries.prisma.sh
  </Accordion>

  <Accordion title="클라우드 서비스 및 모니터링">
    * statsig.com
    * [www.statsig.com](http://www.statsig.com)
    * api.statsig.com
    * sentry.io
    * \*.sentry.io
    * downloads.sentry-cdn.com
    * http-intake.logs.datadoghq.com
    * browser-intake-us5-datadoghq.com
    * \*.datadoghq.com
    * \*.datadoghq.eu
    * api.honeycomb.io
  </Accordion>

  <Accordion title="콘텐츠 전송 및 미러">
    * sourceforge.net
    * \*.sourceforge.net
    * packagecloud.io
    * \*.packagecloud.io
    * fonts.googleapis.com
    * fonts.gstatic.com
  </Accordion>

  <Accordion title="스키마 및 설정">
    * json-schema.org
    * [www.json-schema.org](http://www.json-schema.org)
    * json.schemastore.org
    * [www.schemastore.org](http://www.schemastore.org)
  </Accordion>

  <Accordion title="Model Context Protocol">
    * \*.modelcontextprotocol.io
  </Accordion>
</AccordionGroup>

## 웹과 터미널 간에 작업 이동

이 워크플로는 [Claude Code CLI](/docs/en/quickstart)가 동일한 claude.ai 계정으로 로그인되어 있어야 합니다. 터미널에서 새 클라우드 세션을 시작하거나 클라우드 세션을 터미널로 가죠와 로컬에서 계속할 수 있습니다. 클라우드 세션은 랩톱을 닫아도 유지되며 Claude 모바일 앱을 포함하여 어디서나 모니터링할 수 있습니다. `--cloud` 및 `--teleport` 플래그는 `claude --help` 출력에 표시되지 않지만 CLI는 아래 설명대로 수락합니다.

<Note>
  CLI에서 세션 인계(handoff)는 단방향입니다: `--teleport`로 클라우드 세션을 터미널로 가져올 수는 있지만 기존 터미널 세션을 웹으로 보낼 수는 없습니다. `--cloud` 플래그는 현재 리포지토리에 대한 새 클라우드 세션을 생성합니다. [데스크톱 앱](/docs/en/desktop#continue-in-another-surface)은 로컬 세션을 웹으로 보낼 수 있는 Continue in 메뉴를 제공합니다.
</Note>

### 터미널에서 웹으로

명령줄에서 `--cloud` 플래그를 사용하여 클라우드 세션을 시작합니다:

```bash theme={null}
claude --cloud "Fix the authentication bug in src/auth/login.ts"
```

이렇게 하면 claude.ai에 새 클라우드 세션이 생성됩니다. VM이 사용자의 머신 대신 GitHub에서 복제하므로 세션은 현재 브랜치에서 현재 디렉터리의 GitHub 원격(remote)을 복제합니다. 로컬 커밋이 있는 경우 먼저 푸시하세요. `--cloud`는 한 번에 단일 리포지토리에 대해서만 작동합니다. 로컬에서 작업을 계속하는 동안 작업이 클라우드에서 실행됩니다. 이전 문법인 `--remote`도 `--cloud`에 대한 구식 별칭(deprecated alias)으로 여전히 작동합니다.

{/* min-version: 2.1.195 */}v2.1.195부터 CLI는 클라우드 컨테이너가 시작되는 동안 리포지토리 복제 및 [설정 스크립트](#설정-스크립트) 실행과 같은 설정 단계의 라이브 체크리스트를 보여줍니다. 컨테이너가 프로비저닝되는 동안 입력하는 메시지는 큐에 저장되어 세션이 준비되면 전송됩니다.

<Note>
  `--cloud`는 클라우드 세션을 생성합니다. `--remote-control`은 관련이 없습니다: 웹에서 모니터링할 수 있도록 로컬 CLI 세션을 노출합니다. [Remote Control](/docs/en/remote-control)을 참조하세요.
</Note>

Claude Code CLI에서 `/tasks`를 사용하여 진행 상황을 확인하거나 claude.ai 또는 Claude 모바일 앱에서 세션을 열어 직접 상호작용하세요. 거기서 다른 대화와 마찬가지로 Claude를 안내하거나, 피드백을 제공하거나, 질문에 답할 수 있습니다.

Claude가 질문을 하고 세션이 유휴 상태가 되더라도 [환경 만료](#환경-만료) 전까지는 돌아왔을 때 답변할 수 있으며, 답변한 지점부터 세션이 계속 진행됩니다.

#### 클라우드 작업에 대한 팁

**로컬에서 계획하고 원격에서 실행**: 복잡한 작업의 경우, 방향성에 대해 협업하기 위해 플랜 모드(plan mode)에서 Claude를 시작한 다음 작업을 클라우드로 보내세요:

```bash theme={null}
claude --permission-mode plan
```

플랜 모드에서 Claude는 소스 코드를 편집하지 않고 파일을 읽고 명령어를 실행하여 탐색한 후 계획을 제안합니다. 충족되면 계획을 리포지토리에 저장하고 커밋 및 푸시하여 클라우드 VM이 이를 복제할 수 있도록 하세요. 그런 다음 자율 실행을 위해 클라우드 세션을 시작합니다:

```bash theme={null}
claude --cloud "Execute the migration plan in docs/migration-plan.md"
```

이 패턴을 통해 전략에 대한 제어권을 유지하는 동시에 Claude가 클라우드에서 자율적으로 실행하도록 할 수 있습니다.

**ultraplan으로 클라우드에서 계획 수립**: 웹 세션에서 계획 자체를 작성하고 검토하려면 [ultraplan](/docs/en/ultraplan)을 사용하세요. 작업을 계속하는 동안 Claude가 웹상의 Claude Code에서 계획을 생성하며, 브라우저에서 섹션별로 댓글을 달고 원격 실행을 선택하거나 계획을 터미널로 다시 보낼 수 있습니다.

**작업 병렬 실행**: 각 `--cloud` 명령은 독립적으로 실행되는 자체 클라우드 세션을 생성합니다. 여러 작업을 시작하면 별도의 세션에서 모든 작업이 동시에 실행됩니다:

```bash theme={null}
claude --cloud "Fix the flaky test in auth.spec.ts"
claude --cloud "Update the API documentation"
claude --cloud "Refactor the logger to use structured output"
```

Claude Code CLI에서 `/tasks`로 모든 세션을 모니터링하세요. 세션이 완료되면 웹 인터페이스에서 PR을 생성하거나 세션을 터미널로 [텔레포트](#웹에서-터미널로)하여 작업을 이어갈 수 있습니다.

#### GitHub 없이 로컬 리포지토리 전송

GitHub에 연결되지 않은 리포지토리에서 `claude --cloud`를 실행하면 Claude Code가 로컬 리포지토리를 번들로 묶어 클라우드 세션으로 직접 업로드합니다. 번들에는 모든 브랜치에 걸친 전체 리포지토리 이력과 추적 대상 파일의 커밋되지 않은 변경 사항이 포함됩니다.

이 폴백(fallback)은 GitHub 접근을 이용할 수 없을 때 자동으로 활성화됩니다. GitHub이 연결되어 있어도 이 방식을 강제하려면 `CCR_FORCE_BUNDLE=1`을 설정하세요:

```bash theme={null}
CCR_FORCE_BUNDLE=1 claude --cloud "Run the test suite and fix any failures"
```

번들로 묶인 리포지토리는 다음 제한 사항을 충족해야 합니다:

* 디렉터리는 최소 하나의 커밋이 있는 git 리포지토리여야 합니다.
* 번들로 묶인 리포지토리는 100MB 미만이어야 합니다. 용량이 더 큰 리포지토리는 현재 브랜치만 번들로 묶는 방식으로 전환되며, 그 후 작업 트리의 단일 압축 스냅샷으로 전환되고, 스냅샷이 여전히 너무 큰 경우에만 실패합니다.
* 추적되지 않는 파일(untracked files)은 포함되지 않습니다. 클라우드 세션에서 보기를 원하는 파일에 `git add`를 실행하세요.
* 번들에서 생성된 세션은 [GitHub 인증](#github-인증-옵션)도 구성되어 있지 않은 한 원격으로 다시 푸시할 수 없습니다.

### 웹에서 터미널로

다음 중 하나를 사용하여 클라우드 세션을 터미널로 가져옵니다:

* **`--teleport` 사용**: 명령줄에서 `claude --teleport`를 실행하여 대화형 세션 선택기를 열거나, `claude --teleport <session-id>`를 실행하여 특정 세션을 직접 다시 시작합니다. 커밋되지 않은 변경 사항이 있는 경우 먼저 스태시(stash)하도록 안내됩니다.
* **`/teleport` 사용**: 기존 CLI 세션 내부에서 `/teleport` 또는 `/tp`를 실행하여 Claude Code를 재시작하지 않고 동일한 세션 선택기를 엽니다.
* **`/tasks`에서 실행**: `/tasks`를 실행하여 배경 세션을 확인한 다음 `t`를 눌러 해당 세션으로 텔레포트합니다.
* **웹 인터페이스에서 실행**: **Open in CLI**를 선택하여 터미널에 붙여넣을 수 있는 명령어를 복사합니다.

세션을 텔레포트할 때 Claude는 올바른 리포지토리에 있는지 확인하고, 클라우드 세션에서 브랜치를 가져와(fetch) 체크아웃하고, 전체 대화 이력을 터미널로 로드합니다. 터미널은 자체 세션 사본을 얻게 됩니다. 거기서 수행하는 새 작업은 로컬에 유지되며 claude.ai 또는 Claude 모바일 앱의 클라우드 세션에는 나타나지 않습니다. 텔레포트 후 휴대폰에서 계속 제어하려면 로컬 세션에서 [`/remote-control`](/docs/en/remote-control)을 시작하세요.

`--teleport`는 `--resume`과 다릅니다. `--resume`은 이 머신의 로컬 이력에서 대화를 다시 열고 클라우드 세션을 나열하지 않습니다. `--teleport`는 클라우드 세션과 해당 브랜치를 가져옵니다.

#### 텔레포트 요구 사항

텔레포트는 세션을 다시 시작하기 전에 이러한 요구 사항을 확인합니다. 요구 사항이 충족되지 않으면 에러가 표시되거나 문제를 해결하라는 프롬프트가 표시됩니다.

| 요구 사항          | 상세 내용                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 깨끗한 git 상태    | 작업 디렉터리에 커밋되지 않은 변경 사항이 없어야 합니다. 필요한 경우 텔레포트가 변경 사항을 스태시하도록 안내합니다.                                                                                                                                                                                                                                                                                                                      |
| 올바른 리포지토리  | 포크가 아닌 동일한 리포지토리의 체크아웃 상태에서 `--teleport`를 실행해야 합니다. {/* min-version: 2.1.199 */}v2.1.199부터 Claude Code는 `git@work:owner/repo.git`과 같은 SSH 호스트 별칭이나 `insteadOf`로 다시 작성된 단축 형식처럼 원격을 호스트 이름으로 파싱할 수 없는 경우에도 체크아웃을 수용합니다. 원격의 소유자 및 리포지토리 이름이 세션의 리포지토리와 일치할 때만 확인 프롬프트를 먼저 표시합니다.                               |
| 브랜치 이용 가능   | 클라우드 세션의 브랜치가 원격(remote)에 푸시되어 있어야 합니다. 텔레포트가 자동으로 해당 브랜치를 가져와 체크아웃합니다.                                                                                                                                                                                                                                                                                                                 |
| 동일한 계정        | 클라우드 세션에서 사용된 것과 동일한 claude.ai 계정으로 인증되어 있어야 합니다.                                                                                                                                                                                                                                                                                                                                                           |

#### `--teleport`를 사용할 수 없는 경우

텔레포트에는 claude.ai 구독 인증이 필요합니다. API 키를 통해 인증된 경우 `/login`을 실행하여 claude.ai 계정으로 로그인하세요. Amazon Bedrock, Google Cloud Agent Platform 및 Microsoft Foundry에서는 클라우드 세션이 Anthropic 인프라에서 실행되고 해당 제공업체를 통해서는 이용할 수 없으므로 `Cloud sessions aren't available with <provider>` 메시지와 함께 `--teleport`가 중단됩니다. 이미 claude.ai를 통해 로그인되어 있는데도 `--teleport`를 이용할 수 없는 경우 조직에서 클라우드 세션을 비활성화했을 수 있습니다.

## 세션 작업

세션은 claude.ai/code의 사이드바에 표시됩니다. 거기서 변경 사항을 검토하고, 팀원과 공유하며, 완료된 작업을 보관하거나 세션을 영구적으로 삭제할 수 있습니다.

### 컨텍스트 관리

클라우드 세션은 텍스트 출력을 생성하는 [내장 명령어](/docs/en/commands)를 지원합니다. `/plugin` 또는 `/resume`과 같이 터미널 인터페이스에서만 실행되는 명령어는 사용할 수 없습니다. 터미널에서 선택기나 패널을 여는 명령어는 클라우드 세션에서 다르게 동작합니다:

* {/* min-version: 2.1.205 */}**`/model`, `/effort`, `/fast`, `/color` 및 `/rename`**: 터미널 선택기나 슬라이더를 여는 대신 `/model sonnet`과 같이 값을 인자로 전달하세요. 인자 형식은 세션 환경에 Claude Code v2.1.205 이상이 필요하며 각 명령어의 [이용 안내](/docs/en/commands#all-commands)를 따릅니다: 모델의 [출시 기본 노력이 유지](/docs/en/model-config#adjust-effort-level)되는 동안 `/effort`는 `Not applied`를 보고하고, `/fast`는 패스트 모드가 켜진 상태로 시작된 세션에서만 작동합니다.
* **`/config`**: 웹에서는 값을 설정하는 대신 설정의 Claude Code 섹션을 열며, `key=value`를 포함하여 명령어 뒤의 텍스트는 무시됩니다. 클라우드 세션의 설정을 변경하려면 [환경 변수](#환경-구성)를 사용하거나 [설정 파일](/docs/en/settings)을 리포지토리에 커밋하세요.

특히 컨텍스트 관리의 경우:

| 명령어     | 클라우드 세션 지원 여부 | 참고 사항                                                                                                                |
| :--------- | :---------------------- | :----------------------------------------------------------------------------------------------------------------------- |
| `/compact` | 예                      | 대화를 요약하여 컨텍스트를 확보합니다. `/compact keep the test output`과 같은 선택적 집중 지시 사항을 수락합니다.        |
| `/context` | 예                      | 현재 컨텍스트 창에 포함된 내용을 보여줍니다.                                                                             |
| `/clear`   | 아니요                  | 대신 사이드바에서 새 세션을 시작하세요.                                                                                  |

자동 축소(Auto-compaction)는 컨텍스트 창이 용량 한계에 도달하면 자동으로 실행됩니다. 더 일찍 트리거하려면 [환경 변수](#환경-구성)에서 [`CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`](/docs/en/env-vars)를 설정하세요. 예를 들어 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=70`은 창이 거의 찰 때까지 기다리지 않고 70% 용량에서 축소합니다. 축소 계산을 위한 유효 창 크기를 변경하려면 [`CLAUDE_CODE_AUTO_COMPACT_WINDOW`](/docs/en/env-vars)를 사용하세요.

[서브에이전트](/docs/en/sub-agents)는 로컬에서 작동하는 것과 동일하게 작동합니다. Claude는 Task 도구를 사용하여 서브에이전트를 생성하고 조사나 병렬 작업을 별도의 컨텍스트 창으로 오프로드하여 메인 대화를 가볍게 유지할 수 있습니다. 리포지토리의 `.claude/agents/`에 정의된 서브에이전트는 자동으로 감지됩니다.

[에이전트 팀](/docs/en/agent-teams)은 기본적으로 꺼져 있지만 [환경 변수](#환경-구성)에 `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`을 추가하여 활성화할 수 있습니다.

### 변경 사항 검토

각 세션은 `+42 -18`과 같이 추가되거나 삭제된 행이 있는 디프 지표를 보여줍니다. 이를 선택하여 디프 뷰를 열고 특정 행에 인라인 댓글을 남긴 다음 다음 메시지와 함께 Claude에게 전송할 수 있습니다. PR 생성을 포함한 전체 설명은 [검토 및 반복](/docs/en/web-quickstart#review-and-iterate)을 참조하세요. Claude가 CI 실패 및 리뷰 댓글에 대해 PR을 자동으로 모니터링하도록 하려면 [풀 리퀘스트 자동 수정 (Auto-fix)](#풀-리퀘스트-자동-수정-auto-fix)을 참조하세요.

### 세션 공유

세션을 공유하려면 아래 계정 유형에 따라 가시성(visibility)을 전환하세요. 그 후 세션 링크를 그대로 공유합니다. 수신자는 링크를 열 때 최신 상태를 볼 수 있지만 그들의 뷰가 실시간으로 업데이트되지는 않습니다.

#### Enterprise 또는 Team 계정에서 공유

Enterprise 및 Team 계정의 경우 두 가지 가시성 옵션은 **Private** 및 **Team**입니다. Team 가시성을 선택하면 claude.ai 조직의 다른 구성원에게 세션이 표시됩니다. [Slack의 Claude](/docs/en/slack) 세션은 Team 가시성으로 자동 공유됩니다.

수신자 계정에 연결된 GitHub 계정을 기반으로 리포지토리 접근 권한 검증이 기본적으로 활성화됩니다. 본인 계정의 표시 이름은 접근 권한이 있는 모든 수신자에게 표시됩니다.

#### Max 또는 Pro 계정에서 공유

Max 및 Pro 계정의 경우 두 가지 가시성 옵션은 **Private** 및 **Public**입니다. Public 가시성을 선택하면 claude.ai에 로그인한 모든 사용자에게 세션이 표시됩니다.

공유하기 전에 세션에 민감한 내용이 포함되어 있는지 확인하세요. 세션에는 사설 GitHub 리포지토리의 코드 및 자격 증명이 포함되어 있을 수 있습니다. 리포지토리 접근 권한 검증은 기본적으로 활성화되어 있지 않습니다.

수신자가 리포지토리 접근 권한을 갖도록 요구하거나 공유 세션에서 본인의 이름을 숨기려면 [**Settings > Claude Code > Sharing settings**](https://claude.ai/settings/claude-code)로 이동하세요.

### 세션 보관

세션 목록을 정리하기 위해 세션을 보관할 수 있습니다. 보관된 세션은 기본 세션 목록에서 숨겨지지만 보관된 세션을 필터링하여 볼 수 있습니다.

세션을 보관하려면 사이드바의 세션 위에 마우스를 올리고 보관 아이콘을 선택하세요.

### 세션 삭제

세션을 삭제하면 세션 및 해당 데이터가 영구적으로 제거됩니다. 이 작업은 취소할 수 없습니다. 두 가지 방법으로 세션을 삭제할 수 있습니다:

* **사이드바에서**: 보관된 세션을 필터링한 다음 삭제하려는 세션 위에 마우스를 올리고 삭제 아이콘을 선택
* **세션 메뉴에서**: 세션을 열고 세션 제목 옆의 드롭다운을 선택한 다음 **Delete** 선택

세션이 삭제되기 전에 확인 프롬프트가 표시됩니다.

## 풀 리퀘스트 자동 수정 (Auto-fix)

Claude는 풀 리퀘스트를 지켜보고 CI 실패 및 리뷰 댓글에 자동으로 대응할 수 있습니다. Claude는 PR의 GitHub 활동을 구독하고, 검사가 실패하거나 리뷰어가 댓글을 남기면 사유를 조사하고 해결책이 명확한 경우 수정 사항을 푸시합니다.

<Note>
  Auto-fix를 사용하려면 리포지토리에 Claude GitHub App이 설치되어 있어야 합니다. 아직 설치하지 않았다면 [GitHub App 페이지](https://github.com/apps/claude)에서 설치하거나 [설정](/docs/en/web-quickstart#connect-github-and-create-an-environment) 시 안내에 따라 설치하세요.
</Note>

PR이 어디서 시작되었는지와 사용 중인 기기에 따라 auto-fix를 켜는 몇 가지 방법이 있습니다:

* **웹상의 Claude Code에서 생성된 PR**: CI 상태 표시줄을 열고 **Auto-fix**를 선택
* **터미널에서**: PR 브랜치에 있는 동안 [`/autofix-pr`](/docs/en/commands)을 실행. Claude Code가 `gh`로 열린 PR을 감지하고 웹 세션을 생성하며 한 단계로 auto-fix를 활성화함
* **모바일 앱에서**: Claude에게 PR을 auto-fix하도록 지시 (예: "this PR을 지켜보고 CI 실패나 리뷰 댓글을 수정해줘")
* **기존의 모든 PR**: PR URL을 세션에 붙여넣고 Claude에게 auto-fix하도록 지시

Auto-fix는 PR별 토글입니다. 모니터링을 중지하려면 웹 세션에서 CI 상태 표시줄을 열고 **Auto-fix** 토글을 해제하거나 Claude에게 PR 감시를 중지하도록 지시하세요.

### PR 활동에 대한 Claude의 대응 방식

Auto-fix가 활성화되면 Claude는 새 리뷰 댓글 및 CI 검사 실패를 포함하여 PR에 대한 GitHub 이벤트를 수신합니다. 각 이벤트에 대해 Claude는 사유를 조사하고 진행 방식을 결정합니다:

* **명확한 수정 사항**: Claude가 수정 사항에 자신이 있고 이전 지시 사항과 충돌하지 않는 경우, 변경 조치를 취하고 이를 푸시한 후 세션에서 조치 내용을 설명합니다.
* **모호한 요청**: 리뷰어의 댓글이 여러 가지로 해석될 수 있거나 아키텍처상 중요한 내용인 경우, Claude는 조치를 취하기 전에 사용자에게 문의합니다.
* **중복 또는 조치 불필요 이벤트**: 이벤트가 중복이거나 변경이 필요하지 않은 경우, Claude는 세션에 기록하고 다음으로 넘어갑니다.

기본 브랜치가 진행되고 머지 충돌(merge conflict)이 발생할 때 GitHub은 웹훅을 내보내지 않으므로 auto-fix가 충돌에 자체 대응할 수 없습니다. 충돌을 해결하려면 세션을 열고 Claude에게 리베이스(rebase)하도록 요청하세요.

Claude는 해결 과정의 일부로 GitHub의 리뷰 댓글 스레드에 답글을 남길 수 있습니다. 이러한 답글은 본인의 GitHub 계정을 사용하여 게시되므로 본인의 사용자 이름 아래에 나타나지만, 리뷰어가 사용자가 직접 작성한 것이 아니라 에이전트가 작성한 것임을 알 수 있도록 각 답글에는 Claude Code에서 작성되었다는 레이블이 붙습니다.

<Warning>
  리포지토리가 `issue_comment` 이벤트에서 실행되는 Atlantis, Terraform Cloud 또는 커스텀 GitHub Actions와 같이 댓글로 트리거되는 자동화를 사용하는 경우, Claude가 대신 답글을 남겨 이러한 워크플로가 트리거될 수 있음을 유의하세요. Auto-fix를 활성화하기 전에 리포지토리의 자동화를 검토하고, PR 댓글이 인프라를 배포하거나 권한이 있는 작업을 실행할 수 있는 리포지토리에서는 auto-fix를 비활성화하는 것을 고려하세요.
</Warning>

## 보안 및 격리

각 클라우드 세션은 여러 계층을 통해 본인의 머신 및 다른 세션과 격리됩니다:

* **격리된 가상 머신**: 각 세션은 Anthropic이 관리하는 격리된 VM에서 실행됩니다.
* **네트워크 접근 제어**: 네트워크 접근은 기본적으로 제한되며 비활성화할 수 있습니다. 네트워크 접근이 비활성화된 상태에서 실행 중일 때도 Claude Code는 Anthropic API와 통신할 수 있어 데이터가 VM을 나갈 수 있습니다.
* **자격 증명 보호**: git 자격 증명이나 서명 키와 같은 민감한 자격 증명은 Claude Code가 있는 샌드박스 내부로 들어가지 않습니다. 인증은 범위 지정 자격 증명을 사용하여 보안 프록시를 통해 처리됩니다.
* **안전한 분석**: 코드는 PR을 생성하기 전에 격리된 VM 내에서 분석되고 수정됩니다.

## 문제 해결

대화에 표시되는 `API Error: 500`, `529 Overloaded`, `429` 또는 `Prompt is too long`과 같은 런타임 API 오류는 [오류 참조](/docs/en/errors)를 참조하세요. 해당 오류 및 해결 방법은 CLI 및 데스크톱 앱과 공유됩니다. 아래 섹션에서는 클라우드 세션에 특화된 문제를 다룹니다.

### 세션 생성 실패

새 세션이 `Session creation failed` 메시지와 함께 시작에 실패하거나 프로비저닝 단계에서 멈추는 경우 Claude Code가 클라우드 환경을 할당할 수 없는 상태입니다.

* 클라우드 세션 장애 여부는 [status.claude.com](https://status.claude.com)에서 확인하세요.
* 용량이 요청에 따라 프로비저닝되므로 1분 후 다시 시도하세요.
* 리포지토리에 도달할 수 있는지 확인하세요. 연결된 GitHub 계정은 Claude GitHub App 승인 또는 `/web-setup`을 통해 동기화된 `gh` 토큰을 통해 GitHub의 리포지토리에 접근할 수 있어야 합니다. 리포지토리에 App을 설치하는 것은 필수가 아닙니다. [GitHub 인증 옵션](#github-인증-옵션)을 참조하세요.

### 조직 UUID를 가져올 수 없음

`claude --cloud` 및 `claude --teleport`는 claude.ai 계정을 통한 로그인이 필요합니다. API 키로 인증하거나 저장된 계정 정보가 오래된 경우 이러한 명령어는 `Unable to get organization UUID` 메시지 또는 API 키 인증이 충분하지 않다는 메시지와 함께 실패합니다.

`/login`을 실행하여 claude.ai 계정으로 로그인한 후 명령을 다시 시도하세요.

Amazon Bedrock, Google Cloud Agent Platform 및 Microsoft Foundry에서는 클라우드 세션이 Anthropic 인프라에서 실행되고 해당 제공업체를 통해 이용할 수 없으므로 `Cloud sessions aren't available with <provider>` 메시지와 함께 명령어가 더 일찍 중단됩니다.

### Remote Control 세션 만료 또는 접근 거부

`--teleport`는 클라우드 세션이 사용하는 것과 동일한 Remote Control 세션 인프라를 통해 연결되므로 인증 및 세션 만료 에러가 Remote Control 문구로 표시됩니다. `Remote Control session expired` 또는 `Access denied` 메시지가 표시될 수 있습니다. 연결 토큰은 수명이 짧고 본인 계정으로 범위가 지정됩니다.

* 로컬에서 `/login`을 실행하여 자격 증명을 새로 고친 후 다시 연결하세요.
* 세션을 소유한 동일한 계정으로 로그인되어 있는지 확인하세요.
* `Remote Control may not be available for this organization` 메시지가 표시되면 소유자(Owner)가 조직의 클라우드 세션을 활성화하지 않은 상태입니다.

### 환경 만료

클라우드 세션은 일정 기간 비활동 상태가 지속되면 중지되고 원인 환경이 회수됩니다. 로컬 터미널에서는 `Could not resume session ... its environment has expired. Creating a fresh session instead.` 메시지로 표시됩니다. 웹에서는 세션 목록에 세션이 만료됨(expired)으로 표시됩니다.

[claude.ai/code](https://claude.ai/code)에서 세션을 다시 열어 대화 이력이 복원된 새 환경을 프로비저닝하세요.

## 제한 사항

워크로드에서 클라우드 세션에 의존하기 전에 다음 제약 사항을 고려하세요:

* **속도 제한**: 웹상의 Claude Code는 계정 내 모든 다른 Claude 및 Claude Code 사용량과 속도 제한을 공유합니다. 여러 작업을 병렬로 실행하면 속도 제한이 비례하여 더 소비됩니다. 클라우드 VM에 대한 별도의 컴퓨팅 비용은 없습니다.
* **리포지토리 인증**: 동일한 계정으로 인증된 경우에만 세션을 웹에서 로컬로 이동할 수 있습니다.
* **플랫폼 제약**: 리포지토리 복제 및 풀 리퀘스트 생성에는 GitHub이 필요합니다. 자체 호스팅 [GitHub Enterprise Server](/docs/en/github-enterprise-server) 인스턴스는 Team 및 Enterprise 플랜에서 지원됩니다. GitLab, Bitbucket 및 기타 비 GitHub 리포지토리는 [로컬 번들](#github-없이-로컬-리포지토리-전송)로 클라우드 세션에 보낼 수 있지만, 세션에서 결과를 원격으로 다시 푸시할 수는 없습니다.
* **조직 IP 허용 목록**: 클라우드 세션은 사용자 네트워크가 아닌 Anthropic이 관리하는 인프라에서 Anthropic API를 호출합니다. 조직에 [IP 허용 목록](https://support.claude.com/en/articles/13200993-restrict-access-to-claude-with-ip-allowlisting)이 활성화되어 있으면 모든 클라우드 세션이 인증 오류와 함께 실패합니다. [코드 리뷰](/docs/en/code-review) 및 [루틴](/docs/en/routines)에도 동일하게 적용됩니다. 조직의 IP 허용 목록에서 Anthropic 호스팅 서비스를 제외하려면 [Anthropic 지원팀](https://support.claude.com/)에 문의하세요.

## 관련 리소스

* [Ultraplan](/docs/en/ultraplan): 클라우드 세션에서 계획을 작성하고 브라우저에서 검토
* [Ultrareview](/docs/en/ultrareview): 클라우드 샌드박스에서 심층 다중 에이전트 코드 리뷰 실행
* [루틴](/docs/en/routines): 일정에 따라, API 호출을 통해 또는 GitHub 이벤트에 대응하여 작업 자동화
* [훅 설정](/docs/en/hooks): 세션 수명주기 이벤트에서 스크립트 실행
* [설정 참조](/docs/en/settings): 모든 설정 옵션
* [보안](/docs/en/security): 격리 보장 및 데이터 처리
* [데이터 사용량](/docs/en/data-usage): Anthropic이 클라우드 세션에서 보존하는 내용
* [Claude Tag](https://claude.com/docs/claude-tag/overview): 동일한 클라우드 환경에서 실행되는 조직 관리형 Slack의 @Claude
