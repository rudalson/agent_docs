> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 링크에서 세션 실행하기

> URL에서 Claude Code 터미널 세션을 엽니다. 런북, 알림 및 대시보드에 `claude-cli://` 링크를 포함하여 클릭 한 번으로 올바른 리포지토리와 적절한 프롬프트로 Claude Code를 열 수 있습니다.

딥링크(deep link)는 새 터미널 창에서 Claude Code를 열어주는 `claude-cli://` URL입니다. URL에는 작업 디렉터리와 사전에 채워둘 프롬프트를 포함할 수 있습니다.

이를 통해 작업에 대한 원클릭 시작 포인트를 공유할 수 있습니다: Claude Code가 설치된 사용자가 링크를 클릭하면 프롬프트가 이미 입력된 상태로 세션이 열립니다. 프롬프트는 미리 입력되지만 엔터를 누르기 전까지는 전송되지 않습니다.

딥링크는 URL이므로 링크가 들어갈 수 있는 모든 위치에 넣을 수 있습니다:

* 장애가 발생한 서비스의 리포지토리를 진단 프롬프트와 함께 여는 런북 단계
* 특정 메트릭에 대한 조사 프롬프트로 연결되는 모니터링 알림 또는 대시보드
* 온보딩 프롬프트로 프로젝트를 여는 README 또는 위키 페이지
* 실패한 작업의 이름을 미리 입력해주는 CI 실패 알림

이 페이지에서는 [링크 작성법](#build-a-link), [런북에 포함하거나 셸에서 실행하는 법](#examples), 그리고 각 플랫폼에서 [핸들러 등록 관리 또는 비활성화 방법](#registration-and-supported-platforms)을 다룹니다.

## 작동 방식

`claude-cli://` 접두사는 `mailto:` 링크가 이메일 클라이언트를 여는 것과 유사하게 Claude Code가 운영체제에 등록하는 사용자 지정 URL 스키마입니다. 링크는 웹 페이지, 위키, Slack 메시지 또는 링크를 렌더링하는 모든 앱에 배치할 수 있습니다. 링크를 클릭하면:

1. 브라우저나 앱이 URL을 운영체제에 전달합니다.
2. 운영체제가 `claude-cli://` 접두사를 인식하고 머신에서 Claude Code를 시작합니다.
3. 링크에 지정된 디렉터리에서 Claude Code가 실행되고 링크의 프롬프트 텍스트가 입력 상자에 이미 입력된 상태로 새 터미널 창이 열립니다.
4. 사용자가 프롬프트를 확인하고 필요에 따라 편집한 후 엔터를 눌러 전송합니다.

링크 자체는 어디에나 호스팅될 수 있지만, 세션은 항상 클릭한 컴퓨터의 로컬에서 열립니다. 각 운영체제에서 열리는 터미널 에뮬레이터는 [등록 및 지원되는 플랫폼](#registration-and-supported-platforms)을 참조하세요.

<Note>
  링크를 표시하는 플랫폼이 사용자 지정 URL 스키마를 허용해야 합니다. GitHub에서 렌더링되는 Markdown은 `http` 및 `https`는 허용하지만 README, 이슈, 풀 리퀘스트 및 위키에서 `claude-cli://`와 같은 스키마는 제거합니다. 링크 없이 텍스트만 표시되고 URL은 숨겨집니다. 해결 방법은 [문제 해결](#the-link-renders-as-plain-text-instead-of-being-clickable)을 참조하세요.
</Note>

### 실행된 세션에 표시되는 내용

딥링크는 스스로 아무것도 실행하지 않습니다. 링크는 디렉터리를 선택하고 프롬프트 상자를 채울 뿐입니다. 신뢰하지 않는 페이지의 링크를 클릭하더라도 프롬프트는 여전히 비활성 상태입니다: 채워진 내용을 읽고 엔터를 누르기 전까지는 모델에 아무것도 전달되지 않습니다.

세션이 열리면 입력 상자 아래에 `Prompt from an external link`라는 경고 줄이 표시되며 프롬프트를 전송하거나 지울 때까지 유지됩니다. 1,000자가 넘는 프롬프트의 경우, 긴 프롬프트로 인해 지침이 화면 밖으로 밀려날 수 있으므로 경고에 글자 수가 포함되고 엔터를 누르기 전에 전체 텍스트를 스크롤하여 검토하라는 메시지가 표시됩니다. 선택한 디렉터리에 대한 권한 규칙, `CLAUDE.md` 및 신뢰 프롬프트는 다른 일반 세션과 동일하게 적용됩니다.

## 링크 작성법

모든 딥링크는 핸들러가 수용하는 유일한 경로인 `claude-cli://open`으로 시작하며 뒤이어 선택적 쿼리 매개변수가 옵니다. 가장 간단한 형태는 홈 디렉터리에서 빈 프롬프트로 Claude Code를 엽니다:

```text theme={null}
claude-cli://open
```

페이지에 넣지 않고 링크를 테스트하려면 브라우저 주소창에 붙여넣거나 [셸에서 열기](#open-a-link-from-the-shell)를 사용하세요.

매개변수를 추가하여 세션 시작 위치와 프롬프트 상자 내용을 제어하세요:

| 매개변수 | 설명 |
| --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `q` | 프롬프트 상자에 미리 채울 텍스트입니다. 값을 [URL 인코딩](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent)하세요. 다중 행 프롬프트의 줄 바꿈에는 `%0A`를 사용하세요. 최대 5,000자. |
| `cwd` | 작업 디렉터리로 사용할 절대 경로입니다. 네트워크 및 UNC 경로는 거부되며, 숨겨지거나 양방향 제어 문자가 포함된 경로도 거부됩니다. |
| `repo` | GitHub `owner/name` 슬러그입니다. Claude Code는 이전에 만난 로컬 복제본으로 이를 확인하고 거기서 시작합니다. 일치하는 복제본이 없으면 세션이 홈 디렉터리에서 열립니다. |

`cwd`와 `repo`는 [작업 디렉터리를 설정하는 두 가지 방법](#choose-between-cwd-and-repo)입니다. 둘 다 전달하면 `cwd` 경로가 존재하지 않더라도 `cwd`가 우선하고 `repo`는 무시됩니다.

다음 링크는 2줄의 진단 프롬프트와 함께 `acme/payments`라는 리포지토리를 가리킵니다. 직접 작성할 때는 `acme/payments`를 본인의 리포지토리 `owner/name` 슬러그로 교체하세요:

```text theme={null}
claude-cli://open?repo=acme/payments&q=Investigate%20the%20failed%20deploy%20of%20payments-api.%0ACheck%20recent%20commits%20to%20main%20and%20the%20last%20successful%20build.
```

클릭하면 새 터미널 창이 열리고 로컬 `acme/payments` 복제본에서 Claude Code가 시작되며 디코딩된 텍스트가 프롬프트 상자에 채워집니다:

```text theme={null}
Investigate the failed deploy of payments-api.
Check recent commits to main and the last successful build.
```

엔터를 눌러 전송하기 전에 프롬프트를 편집할 수 있습니다. 리포지토리의 로컬 복제본이 없는 경우 세션이 홈 디렉터리에서 대신 열립니다. 복제본이나 워크트리가 여러 개 있을 때 로컬 경로가 선택되는 방식은 [`cwd`와 `repo` 중에서 선택하기](#choose-between-cwd-and-repo)를 참조하세요.

### `cwd`와 `repo` 중에서 선택하기

표준화된 devcontainer 또는 VM 이미지와 같이 링크를 클릭하는 모든 사람이 동일한 절대 경로에 프로젝트를 가지고 있는 경우 `cwd`를 사용하세요.

링크가 공유되고 각 사람이 서로 다른 위치에 복제하는 경우 `repo`를 사용하세요. Claude Code는 슬러그를 다음과 같이 로컬 경로로 확인합니다:

* Git 리포지토리에서 `claude`를 실행할 때마다 해당 디렉터리의 파일 시스템 경로가 리포지토리의 GitHub `owner/name` 슬러그에 대해 기록됩니다.
* 딥링크가 도착하면 `repo`는 일치하는 경로 중 가장 최근에 사용한 경로를 엽니다. 여러 복제본과 워크트리가 개별적으로 추적되므로 마지막으로 작업한 경로가 선택됩니다.
* 조회 시 Claude Code를 한 번 이상 실행했던 경로만 찾습니다.
* 링크는 체크아웃된 브랜치를 변경하지 않습니다. 세션은 해당 디렉터리가 현재 존재하는 상태 그대로 열립니다.

환영 헤더에 선택된 경로가 표시되므로 올바른 복제본이 열렸는지 확인할 수 있습니다.

## 예제

아래 섹션에서는 딥링크를 사용하는 두 가지 일반적인 방법인 문서의 Markdown 링크 및 스크립트나 셸 별칭의 명령을 보여줍니다.

### 런북에 링크 포함하기

런북의 딥링크는 문제 해결 담당자에게 준비된 프롬프트와 함께 올바른 리포지토리에서 조사를 시작할 수 있는 원클릭 방법을 제공합니다. 런북을 렌더링하는 플랫폼은 사용자 지정 URL 스키마를 허용해야 합니다. GitHub에서 렌더링되는 Markdown은 `claude-cli://`를 허용하지 않으므로 GitHub README, 이슈 또는 위키의 딥링크는 클릭 가능한 링크 없이 레이블만 표시됩니다. 해결 방법은 [문제 해결 노트](#the-link-renders-as-plain-text-instead-of-being-clickable)를 참조하세요.

프롬프트는 URL의 일부이므로 URL 인코딩되어야 합니다. 인코딩된 값을 생성하려면 브라우저 콘솔 또는 임의의 URL 인코더에서 프롬프트 텍스트를 `encodeURIComponent`에 전달하세요.

아래 예제는 `web-gateway` 서비스에 대한 장애 대응 런북에 조사 진입점을 추가합니다:

```markdown theme={null}
## High 5xx rate on web-gateway

1. PagerDuty에서 알림을 확인합니다.
2. [gateway 리포지토리에서 Claude Code 열기](claude-cli://open?repo=acme/web-gateway&q=5xx%20rate%20is%20elevated%20on%20web-gateway.%20Check%20recent%20deploys%2C%20error%20logs%20from%20the%20last%2030%20minutes%2C%20and%20open%20incidents%20in%20Linear.)
3. #incident에 초기 조치 결과를 게시합니다.
```

본인의 런북에서 이를 사용하려면 `acme/web-gateway`를 서비스의 리포지토리 슬러그로 바꾸세요. 이를 통해 Claude Code가 설치되어 있고 해당 리포지토리의 로컬 복제본을 가진 엔지니어가 2 단계를 클릭하여 전송할 준비가 된 프롬프트로 조사를 시작할 수 있습니다.

### 셸에서 링크 열기

클릭하는 대신 셸 스크립트, 별칭 또는 자동화에서 딥링크를 열 수도 있습니다. 링크를 인수로 하여 운영체제의 URL 열기 명령을 호출하세요. 이러한 명령은 머신에서 [대화형 세션의 첫 프롬프트를 보낼 때](#registration-and-supported-platforms) Claude Code가 등록하는 핸들러에 의존합니다.

<Tabs>
  <Tab title="macOS">
    내장된 `open` 명령은 등록된 `claude-cli://` 핸들러에 URL을 전달합니다:

    ```bash theme={null}
    open "claude-cli://open?repo=acme/payments&q=review%20open%20PRs"
    ```
  </Tab>

  <Tab title="Linux">
    대부분의 데스크톱 환경은 등록된 핸들러로 URL을 전달하는 `xdg-open`을 제공합니다:

    ```bash theme={null}
    xdg-open "claude-cli://open?repo=acme/payments&q=review%20open%20PRs"
    ```

    성공하면 Claude Code가 실행되고 프롬프트가 사전 입력된 새 터미널 창이 열립니다. 셸에서 `xdg-open`을 찾을 수 없다고 보고하는 경우 [문제 해결](#xdg-open-is-not-found-on-linux)을 참조하세요.
  </Tab>

  <Tab title="Windows">
    PowerShell에서 `Start-Process`는 등록된 핸들러에 URL을 전달합니다:

    ```powershell theme={null}
    Start-Process "claude-cli://open?repo=acme/payments&q=review%20open%20PRs"
    ```

    `cmd.exe`에서 `start`는 큰따옴표로 둘러싸인 첫 번째 인수를 창 제목으로 취급하므로 URL 앞에 빈 제목을 전달하세요:

    ```cmd theme={null}
    start "" "claude-cli://open?repo=acme/payments&q=review%20open%20PRs"
    ```
  </Tab>
</Tabs>

## 등록 및 지원되는 플랫폼

Claude Code는 macOS, Linux 및 Windows에서 대화형 세션의 첫 프롬프트를 보낼 때 운영체제에 `claude-cli://` 핸들러를 등록합니다. `claude`를 시작하고 프롬프트를 보내지 않고 종료하면 핸들러가 등록되지 않습니다. 별도의 설치 명령을 실행할 필요가 없습니다. 등록은 사용자 수준 위치에만 기록됩니다:

| 플랫폼 | 핸들러 위치 |
| -------- | ---------------------------------------------------------------------------------- |
| macOS | `~/Applications/Claude Code URL Handler.app` |
| Linux | `$XDG_DATA_HOME/applications` 아래 `claude-code-url-handler.desktop` (기본값: `~/.local/share/applications`) |
| Windows | `HKEY_CURRENT_USER\Software\Classes\claude-cli` |

핸들러는 감지된 터미널 에뮬레이터에서 Claude Code를 실행합니다. macOS에서 Claude Code는 가장 최근의 대화형 세션에서 사용한 터미널을 기억하고 이를 재사용하며 iTerm2, Ghostty, kitty, Alacritty, WezTerm 및 Terminal.app을 지원합니다. Linux에서는 `$TERMINAL` 환경 변수, `x-terminal-emulator`, 그 다음 일반적인 에뮬레이터 목록 순으로 적용합니다. Windows에서는 Windows Terminal, PowerShell, `cmd.exe` 순으로 우대합니다.

등록을 완전히 방지하려면 `settings.json`에서 [`disableDeepLinkRegistration`](/docs/en/settings)을 `"disable"`로 설정하세요. 사용자가 이를 다시 활성화할 수 없도록 조직 전체에서 강제하려면 [관리형 설정](/docs/en/server-managed-settings)에 설정하세요.

## 터미널 대신 VS Code 탭 열기

VS Code 확장 프로그램은 터미널 창 대신 Claude Code 편집기 탭을 열어주는 `vscode://anthropic.claude-code/open`에 자체 핸들러를 등록합니다. 해당 URL의 매개변수는 [다른 도구에서 VS Code 탭 실행](/docs/en/vs-code#launch-a-vs-code-tab-from-other-tools)을 참조하세요.

## 문제 해결

### 링크를 클릭해도 아무 반응이 없음

핸들러가 아직 등록되지 않았을 가능성이 큽니다. 등록은 세션이 시작될 때가 아니라 대화형 세션의 첫 프롬프트를 보낼 때 수행됩니다. 해당 머신에서 대화형 `claude` 세션을 시작하고 아무 프롬프트나 보낸 후 종료하고 링크를 다시 시도하세요. 데스크톱 환경이 없는 Linux를 사용 중인 경우 `xdg-open`이 디스패치할 대상이 없을 수 있습니다.

### Linux에서 xdg-open을 찾을 수 없음

`xdg-open` 명령은 최소 서버 이미지, 컨테이너 및 WSL 배포판에서 자주 제외되는 `xdg-utils` 패키지의 일부입니다. 배포판의 패키지 관리자로 `xdg-utils`를 설치한 후(예: `sudo apt install xdg-utils`) 명령을 다시 실행하세요. 해당 명령이 실행되었으나 아무것도 열리지 않는 경우 `xdg-open`이 디스패치할 데스크톱 환경이 없는 것일 수 있습니다. [링크를 클릭해도 아무 반응이 없음](#clicking-the-link-does-nothing)을 참조하세요.

### 링크가 클릭 가능하지 않고 일반 텍스트로 렌더링됨

일부 Markdown 렌더러는 `http` 및 `https` 링크만 허용하고 다른 URL 스키마는 제거합니다. GitHub는 README, 이슈, 풀 리퀘스트 및 위키에서 다음과 같이 처리합니다: `[label](claude-cli://...)`은 링크 없이 URL이 제거된 `label`로만 렌더링됩니다. 이러한 플랫폼에서는 딥링크를 코드 블록에 배치하여 독자가 URL을 확인하고 브라우저 주소창에 붙여넣을 수 있도록 하세요.

### 세션이 리포지토리가 아닌 홈 디렉터리에서 열림

`repo` 매개변수는 Claude Code가 이미 확인한 복제본만 해결합니다. 경로가 기록되도록 복제본 내부에서 `claude`를 한 번 실행하거나 절대 경로가 포함된 `cwd`를 사용하도록 링크를 전환하세요.

### 링크가 잘못된 터미널에서 열림

macOS에서는 원하는 터미널에서 `claude`를 한 번 시작하면 다음 딥링크에서 해당 터미널을 사용합니다. Linux에서는 `$TERMINAL` 환경 변수를 원하는 에뮬레이터의 명령 이름으로 설정하세요. Windows에서는 순서가 고정되어 있습니다: PowerShell이나 `cmd.exe` 창 대신 거기서 열리도록 하려면 Windows Terminal을 설치하세요.

## 더 알아보기

다음 페이지에서 Claude Code 세션을 실행하거나 확장하는 관련 방법을 다룹니다:

* [스킬](/docs/en/skills): 긴 런북 프롬프트를 리포지토리에 `/skill`로 저장하여 딥링크의 `q` 매개변수에는 해당 이름만 지정할 수 있도록 설정
* [비대화형 모드](/docs/en/headless): 터미널을 열지 않고 스크립트에서 Claude를 실행하고 출력을 캡처
