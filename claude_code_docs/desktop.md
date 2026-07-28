> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 데스크톱 애플리케이션

> Claude Code Desktop 활용도 높이기: Git 격리를 지원하는 병렬 세션, 드래그 앤 드롭 창 레이아웃, 통합 터미널 및 파일 편집기, 사이드 챗, 컴퓨터 사용(computer use), 휴대폰에서 Dispatch 세션 실행, 시각적 diff 검토, 앱 미리보기, PR 모니터링, 커넥터 및 엔터프라이즈 구성.

Claude Desktop 앱에는 3개의 탭이 있습니다: 대화를 위한 **Chat**, [Dispatch 및 더 긴 에이전트 작업](https://claude.com/product/cowork)을 위한 **Cowork**, 소프트웨어 개발을 위한 **Code**. 이 페이지는 Code 탭에 대한 참조 문서입니다.

<CardGroup cols={3}>
  <Card title="macOS용 다운로드" icon="apple" href="https://claude.ai/api/desktop/darwin/universal/dmg/latest/redirect?utm_source=claude_code&utm_medium=docs">
    Intel 및 Apple Silicon용 유니버설 빌드
  </Card>

  <Card title="Windows용 다운로드" icon="windows" href="https://claude.ai/api/desktop/win32/x64/setup/latest/redirect?utm_source=claude_code&utm_medium=docs">
    x64 프로세서용
  </Card>

  <Card title="Linux용 Claude 다운로드 (beta)" icon="linux" href="/docs/en/desktop-linux">
    Ubuntu 및 Debian용 apt 또는 .deb
  </Card>
</CardGroup>

Windows ARM64의 경우 [ARM64 설치 프로그램](https://claude.ai/api/desktop/win32/arm64/setup/latest/redirect?utm_source=claude_code\&utm_medium=docs)을 다운로드하세요. Linux에서는 apt로 설치하세요. 자세한 내용은 [Linux에서의 Claude Desktop](/docs/en/desktop-linux)을 참조하세요.

설치 후 Claude를 실행하고 로그인한 다음 **Code** 탭을 클릭하세요. Windows에서 처음 열 때 [Git for Windows](https://git-scm.com/downloads/win)가 설치되어 있어야 합니다. 설치 후 앱을 다시 시작하세요. 첫 세션에 대한 안내는 [시작하기 가이드](/docs/en/desktop-quickstart)를 참조하세요.

Code 탭에서 각 대화는 **세션**입니다: 다른 세션과 독립적으로 자체 채팅 기록, 프로젝트 폴더 및 코드 변경 사항을 가집니다. 사이드바에는 세션이 나열되며 여러 세션을 병렬로 실행할 수 있습니다. 세션 내에서 다음을 수행할 수 있습니다:

* [diff 검토 및 주석 달기](#review-changes-with-diff-view) 후 [CI를 통해 결과 PR 모니터링](#monitor-pull-request-status)
* Claude가 스스로 변경 사항을 검증하는 동안 Browser 창에서 [실행 중인 앱 미리보기](#preview-your-app) 및 그 옆에서 [외부 사이트 열기](#browse-external-sites)
* iOS 시뮬레이터 창에서 Claude가 [iOS 앱을 실행하고 테스트](/docs/en/desktop-ios-simulator)하는 모습 관찰
* 채팅, diff, 브라우저, 터미널 및 파일 편집기 [창을 나란히 배치](#arrange-your-workspace)
* 세션의 진행을 방해하지 않고 컨텍스트를 활용하는 [사이드 질문하기](#ask-a-side-question-without-derailing-the-session)
* GitHub, Slack 및 Linear와 같은 [외부 도구 연결](#connect-external-tools)
* Claude가 [앱을 열고 화면을 제어](#let-claude-use-your-computer)하도록 허용
* 사용자의 머신, [클라우드](#run-long-running-tasks-remotely) 또는 [SSH](#ssh-sessions)에서 실행

[정기적인 예약 작업](/docs/en/desktop-scheduled-tasks), [키보드 단축키](#keyboard-shortcuts), 또는 [휴대폰에서 작업 전송](#sessions-from-dispatch)에 대해서는 링크된 페이지 및 섹션을 참조하세요. 이미 터미널 기반 CLI를 사용 중이라면 인계되는 내용에 대해 [CLI 비교](#coming-from-the-cli)를 참조하세요.

## 세션 시작하기

첫 메시지를 보내기 전에 프롬프트 영역에서 4가지를 구성하세요:

* **Environment (환경)**: Claude가 실행되는 위치를 선택합니다. 사용자의 머신은 **Local**, Anthropic 호스팅 클라우드 세션은 **Cloud**, 직접 관리하는 원격 머신은 [**SSH 연결**](#ssh-sessions), Windows의 경우 [**WSL 배포판**](/docs/en/desktop-wsl)을 선택하세요. [환경 구성](#environment-configuration)을 참조하세요.
* **Project folder (프로젝트 폴더)**: Claude가 작업할 폴더 또는 리포지토리를 선택합니다. 클라우드 세션의 경우 [여러 리포지토리](#run-long-running-tasks-remotely)를 추가할 수 있습니다.
* **Model (모델)**: 전송 버튼 옆의 드롭다운에서 [모델](/docs/en/model-config#available-models)을 선택합니다. 세션 중에 이를 변경할 수 있습니다.
* **Permission mode (권한 모드)**: [모드 선택기](#choose-a-permission-mode)에서 Claude의 자율성 수준을 선택합니다. 세션 중에 이를 변경할 수 있습니다.

작업을 입력하고 **Enter**를 눌러 시작하세요. 각 세션은 컨텍스트와 변경 사항을 독립적으로 추적합니다.

## 코드로 작업하기

Claude에게 올바른 컨텍스트를 제공하고, 스스로 수행할 작업량을 제어하며, 변경된 내용을 검토하세요.

### 프롬프트 상자 사용법

Claude가 수행하기를 원하는 작업을 입력하고 **Enter**를 눌러 전송하세요. Claude는 지정된 [권한 모드](#choose-a-permission-mode)에 따라 프로젝트 파일을 읽고, 변경 사항을 만들고, 명령을 실행합니다. 언제든지 Claude의 방향을 전환할 수 있습니다: 중지 버튼을 클릭하여 즉시 중단하거나, 수정 사항을 입력하고 **Enter**를 눌러 현재 실행 중인 동작을 멈추지 않고 전송할 수 있습니다. Claude는 현재 동작이 완료되는 즉시 수정 사항을 읽고 다음 단계 전에 조정합니다.

프롬프트 상자 옆의 **+** 버튼을 통해 파일 첨부, [스킬](#use-skills), [커넥터](#connect-external-tools) 및 [플러그인](#install-plugins)에 접근할 수 있습니다.

### 프롬프트에 파일 및 컨텍스트 추가하기

프롬프트 상자는 외부 컨텍스트를 가져오는 두 가지 방법을 지원합니다:

* **@mention 파일**: `@` 뒤에 파일명을 입력하여 대화 컨텍스트에 파일을 추가합니다. 그러면 Claude가 해당 파일을 읽고 참조할 수 있습니다. @mention은 클라우드 또는 WSL 세션에서는 사용할 수 없습니다.
* **파일 첨부**: 첨부 버튼을 사용하거나 프롬프트에 직접 파일을 드래그 앤 드롭하여 이미지, PDF 및 기타 파일을 프롬프트에 첨부합니다.이는 버그 스크린샷, 디자인 목업 또는 참조 문서를 공유할 때 유용합니다.

### 권한 모드 선택하기

권한 모드는 세션 동안 Claude가 가지는 자율성 수준을 제어합니다. 파일을 편집하거나 명령을 실행하거나 둘 다를 수행하기 전에 물어볼지 여부를 결정합니다. 전송 버튼 옆의 모드 선택기를 사용하여 언제든지 모드를 전환할 수 있습니다. Claude가 수행하는 작업을 정확히 확인하려면 Manual(수동)로 시작한 다음, 익숙해지면 Accept edits(편집 승인) 또는 Plan(계획)으로 이동하세요.

새 로컬 세션의 기본 모드를 설정하려면 [설정 파일](/docs/en/settings#settings-files)에 `permissions.defaultMode`를 추가하세요. 데스크톱 앱은 CLI와 동일한 설정 파일을 읽습니다. 선택기에서 선택한 모드는 폴더별로 기억되어 현재 세션에만 적용되는 Plan을 제외하고 해당 폴더의 `defaultMode`보다 우선합니다.

| 모드 | 설정 키 | 동작 |
| ---------------------- | ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Manual** | `default` | Claude가 파일을 편집하거나 명령을 실행하기 전에 물어봅니다. diff를 확인하고 각 변경 사항을 승인하거나 거부할 수 있습니다. 신규 사용자에게 권장됩니다. |
| **Accept edits** | `acceptEdits` | Claude가 파일 편집 및 `mkdir`, `touch`, `mv`와 같은 일반적인 파일 시스템 명령을 자동으로 승인하지만, 다른 터미널 명령을 실행하기 전에는 여전히 물어봅니다. 파일 변경 사항을 신뢰하고 빠른 반복 작업을 원할 때 사용하세요. |
| **Plan** | `plan` | Claude가 파일을 읽고 명령을 실행하여 탐색한 후 소스 코드를 편집하지 않고 계획을 제안합니다. 먼저 접근 방식을 검토하려는 복잡한 작업에 적합합니다. |
| **Auto** | `auto` | Claude가 요청과의 정합성을 검증하는 백그라운드 안전 검사와 함께 모든 작업을 실행합니다. 감독 상태를 유지하면서 권한 프롬프트를 줄입니다. 계정이 아래의 [사용 가능 요건](#auto-mode-availability)을 충족할 때 나타나며 별도의 Settings 토글은 없습니다. |
| **Bypass permissions** | `bypassPermissions` | Claude가 명시적인 [ask 규칙](/docs/en/permissions#manage-permissions), [조직에서 `ask`로 설정한 커넥터 도구](/docs/en/mcp#organization-controls-on-connector-tools), [`requiresUserInteraction`으로 표시된 MCP 도구](/docs/en/mcp#require-approval-for-a-specific-tool), 또는 Claude가 [외부 사이트에서 작동할 때](#browse-external-sites)의 안전 분류기에 의해 강제되는 경우를 제외하고 권한 프롬프트 없이 실행됩니다; CLI의 `--dangerously-skip-permissions`와 동일합니다. Pro 및 Max 플랜의 경우 Settings → Claude Code의 "Allow bypass permissions mode"에서 활성화하고, Team 및 Enterprise 플랜의 경우 별도의 Settings 토글이 없으며 조직 정책에 의해 제어됩니다. 샌드박스화된 컨테이너나 VM에서만 사용하세요. |

Code 탭의 이전 버전에서는 이러한 모드를 Ask permissions, Auto accept edits, Plan mode로 표기했습니다.

`dontAsk` 권한 모드는 [CLI](/docs/en/permission-modes#allow-only-pre-approved-tools-with-dontask-mode)에서만 사용할 수 있습니다.

<span id="auto-mode-availability" />

Auto 모드는 Anthropic API의 모든 사용자에게 제공되며 Claude Opus 4.6 이상, Sonnet 4.6 이상 또는 Fable 5가 필요합니다. 조직 관리자는 [관리형 설정](#managed-settings)의 `disableAutoMode` 키로 Auto 모드를 끌 수 있습니다.

Desktop을 Google Cloud's Agent Platform으로 라우팅하는 엔터프라이즈 배포에서는 Auto 모드가 [기본적으로 사용 가능](/docs/en/permission-modes#enable-auto-mode-on-bedrock-agent-platform-or-foundry)하며, 거기서는 Claude Sonnet 5, Opus 4.7, Opus 4.8 및 Fable 5만 지원됩니다. {/* min-version: 2.1.207 */}Claude Code v2.1.207 이전에는 Google Cloud's Agent Platform의 엔터프라이즈 배포에서 Auto 모드를 활성화하려면 `CLAUDE_CODE_ENABLE_AUTO_MODE`를 설정해야 했습니다.

<Tip title="모범 사례">
  복잡한 작업은 Plan에서 시작하여 Claude가 변경을 시작하기 전에 접근 방식을 수립하도록 하세요. 계획을 승인한 후 Accept edits 또는 Manual로 전환하여 실행하세요. 이 워크플로에 대한 자세한 내용은 [먼저 탐색하고, 계획을 세운 다음, 코딩하기](/docs/en/best-practices#explore-first-then-plan-then-code)를 참조하세요.
</Tip>

클라우드 세션은 Accept edits, Plan 및 Auto를 지원합니다. Accept edits는 `default` 모드에 해당합니다: 클라우드 세션은 파일 편집을 사전에 승인하므로 선택기에는 Manual 대신 Accept edits가 표시됩니다. 클라우드 환경은 이미 샌드박스화되어 있으므로 Bypass permissions는 사용할 수 없습니다.

엔터프라이즈 관리자는 어떤 권한 모드를 사용할 수 있는지 제한할 수 있습니다. 자세한 내용은 [엔터프라이즈 구성](#enterprise-configuration)을 참조하세요.

### 앱 미리보기

Claude는 개발 서버를 시작하고 Browser 창에서 열어 변경 사항을 검증할 수 있습니다. 이는 프론트엔드 웹앱뿐만 아니라 백엔드 서버에서도 작동합니다: Claude는 API 엔드포인트를 테스트하고, 서버 로그를 확인하며, 찾아낸 문제를 반복적으로 수정할 수 있습니다. 대부분의 경우 Claude는 프로젝트 파일을 편집한 후 서버를 자동으로 시작합니다. 언제든지 Claude에게 미리보기를 요청할 수도 있습니다. 기본적으로 Claude는 모든 편집 후에 변경 사항을 [자동 검증](#auto-verify-changes)합니다.

Browser 창은 프로젝트의 정적 HTML 파일, PDF, 이미지 및 비디오도 열 수 있습니다. 채팅에서 HTML, PDF, 이미지 또는 비디오 경로를 클릭하여 거기서 여세요.

Browser 창에서 다음을 수행할 수 있습니다:

* Browser 창에서 실행 중인 앱과 직접 상호작용
* Claude가 자체 변경 사항을 자동으로 검증하는 과정 관찰: 스크린샷을 찍고, DOM을 검사하고, 요소를 클릭하고, 폼을 채우고, 발견한 문제를 수정합니다.
* 세션 툴바의 서버 드롭다운에서 서버 시작 또는 중지
* 드롭다운에서 **Persist sessions**를 선택하여 서버 재시작 시 쿠키 및 로컬 스토리지를 유지함으로써 개발 중에 다시 로그인할 필요가 없도록 설정
* 서버 구성 편집 또는 모든 서버 한 번에 중지

Claude는 프로젝트를 기반으로 초기 서버 구성을 생성합니다. 앱에서 커스텀 개발 명령을 사용하는 경우 `.claude/launch.json`을 설정에 맞게 편집하세요. 전체 참조는 [미리보기 서버 구성](#configure-preview-servers)을 참조하세요.

저장된 세션 데이터를 지우거나 Browser를 완전히 끄려면 Settings → Claude Code의 토글을 사용하세요.

### 외부 사이트 둘러보기

Browser 창은 탭 브라우저이므로 실행 중인 앱 옆에 문서, 이슈 트래커 또는 다른 사이트를 열 수 있습니다. Browser를 열려면 macOS에서 **Cmd+Shift+B**, Windows에서 **Ctrl+Shift+B**를 누르거나 **Views** 메뉴에서 선택하세요. 채팅에서 외부 링크를 클릭하면 브라우저 창을 사용하는 **Open in app**과 본인의 브라우저를 사용하는 **Default browser** 중 선택할 수 있는 선택기가 표시됩니다; macOS에서 **Cmd**-클릭, Windows에서 **Ctrl**-클릭하면 시스템 브라우저에서 링크가 직접 열립니다. Google OAuth와 같은 팝업 로그인 흐름을 포함하여 창 내 사이트에 로그인할 수 있습니다.

Claude는 [앱 미리보기](#preview-your-app)에 사용하는 것과 동일한 도구를 사용하여 외부 페이지를 읽고 상호작용할 수 있으며, 두 가지 추가 안전 검사가 적용됩니다:

* 안전 분류기는 모든 권한 모드에서 클릭 및 타이핑과 같은 외부 페이지에서의 Claude 쓰기 작업을 검토합니다. 이는 [Auto 모드](#choose-a-permission-mode)에서 사용하는 분류기와 동일하며, 작업에 플래그가 지정되면 모드에 관계없이 권한 프롬프트가 표시됩니다.
* Auto 및 Bypass permissions 이외의 권한 모드에서는 Claude가 새 사이트로 이동하기 전에 도메인 허용 목록 검사도 적용됩니다.

#### 사이트에서 Claude의 작업 승인하기

Claude가 외부 사이트에서 처음 작동할 때 권한 카드가 나타나고 Claude는 **Allow once**, **Always allow**, 또는 **Deny** 중 사용자의 선택을 기다립니다. **Allow once**는 아무것도 저장하지 않고 작업을 승인합니다. **Always allow**는 기기에 해당 사이트에 대한 승인을 저장하며 Settings에서 취소할 수 있습니다. 서브도메인을 포함하여 각 사이트마다 자체 승인이 필요합니다. 로컬 개발 서버와 프로젝트 파일은 승인이 필요하지 않으므로 [자동 검증](#auto-verify-changes)은 프롬프트 없이 계속 작동합니다.

승인된 사이트에서도 Claude는 사용자의 입력 없이 항목을 구매하거나, 계정을 생성하거나, CAPTCHA를 우회하지 않습니다. Browser 창에서의 탐색은 [Claude in Chrome 확장 프로그램](/docs/en/chrome)과 동일한 안전 모델을 사용합니다. 민감한 사이트 및 위험한 작업을 Claude가 처리하는 방법은 [Claude in Chrome 안전하게 사용하기](https://support.claude.com/en/articles/12902428-using-claude-in-chrome-safely)를 참조하세요.

#### Browser와 Chrome 확장 프로그램 중에서 선택하기

Browser 창은 저장된 로그인이나 기록이 없는 개인 브라우저와 분리된 깨끗한 브라우저 프로필을 사용합니다. 앱 빌드 및 테스트, 정체성이 필요 없는 사이트에 이를 사용하세요. 로그인된 세션에서 Claude가 사용자로 작동하기를 원할 때는 로그인 상태를 공유하는 [Claude in Chrome 확장 프로그램](/docs/en/chrome)을 대신 사용하세요.

#### 조직에 대한 외부 탐색 제한하기

Browser는 Claude in Chrome 확장 프로그램과 동일한 [사이트 허용 목록 및 차단 목록 제어](https://support.claude.com/en/articles/13065128-claude-in-chrome-admin-controls)를 따릅니다. 조직에서 확장 프로그램에 대한 목록을 이미 구성한 경우 Browser가 이를 자동으로 준수합니다. 관리자는 [`browserExternalPageTools` 관리형 설정](#managed-settings)을 통해 외부 페이지에서 Claude의 도구를 끌 수도 있습니다. 도구가 비활성화되어도 사용자는 여전히 외부 사이트로 이동할 수 있습니다; Claude의 도구는 이를 읽거나 작업할 수 없습니다.

외부 탐색을 완전히 끄려면 [`disableBrowserExternalNavigation` 관리형 설정](#managed-settings)을 `true`로 설정하세요. 이렇게 하면 조직의 허용 목록에 있는 사이트를 포함하여 Browser에서의 모든 외부 탐색이 차단됩니다; localhost 개발 서버 및 파일 미리보기는 계속 작동합니다. Claude의 도구 없이 사용자가 외부 사이트를 계속 둘러볼 수 있도록 하려면 `browserExternalPageTools`를 사용하고, 사용자 및 Claude 모두에 대해 외부 사이트를 차단하려면 `disableBrowserExternalNavigation`을 사용하세요.

### diff 뷰로 변경 사항 검토하기

Claude가 코드를 변경한 후 diff 뷰를 사용하면 풀 리퀘스트를 생성하기 전에 파일별로 수정 사항을 검토할 수 있습니다.

Claude가 파일을 변경하면 추가되고 제거된 줄 수(예: `+12 -1`)를 보여주는 diff 통계 표시기가 나타납니다. 이 표시기를 클릭하면 왼쪽에 파일 목록이, 오른쪽에 각 파일의 변경 사항이 표시되는 diff 뷰어가 열립니다.

특정 줄에 주석을 달려면 diff의 임의 줄을 클릭하여 주석 상자를 엽니다. 피드백을 입력하고 **Enter**를 눌러 주석을 추가하세요. 여러 줄에 주석을 추가한 후 모든 주석을 한 번에 제출하세요:

* **macOS**: **Cmd+Enter** 누르기
* **Windows**: **Ctrl+Enter** 누르기

Claude가 주석을 읽고 요청된 변경을 수행하며, 이는 검토할 수 있는 새 diff로 나타납니다.

### 코드 검토하기

diff 뷰의 오른쪽 상단 툴바에서 **Review code**를 클릭하여 커밋하기 전에 Claude가 변경 사항을 평가하도록 요청하세요. Claude는 현재 diff를 검사하고 diff 뷰에 직접 주석을 남깁니다. 모든 주석에 답글을 달거나 Claude에게 수정하도록 요청할 수 있습니다.

검토는 신호가 높은 문제에 집중합니다: 컴파일 오류, 명확한 논리 오류, 보안 취약점 및 명백한 버그. 스타일, 서식, 기존 문제 또는 린터가 잡아낼 수 있는 항목은 플래그를 지정하지 않습니다.

### 풀 리퀘스트 상태 모니터링하기

풀 리퀘스트를 연 후 세션에 CI 상태 표시줄이 나타납니다. Claude Code는 GitHub CLI를 사용하여 체크 결과를 폴링하고 실패를 표면화합니다.

* **Auto-fix**: 활성화되면 Claude가 실패 출력을 읽고 반복 조치하여 실패한 CI 체크를 자동으로 수정하려고 시도합니다.
* **Auto-merge**: 활성화되면 모든 체크가 통과했을 때 Claude가 PR을 병합합니다. 병합 방식은 스쿼시(squash)입니다. 이 기능이 작동하려면 [GitHub 리포지토리 설정에서 자동 병합이 활성화](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-auto-merge-for-pull-requests-in-your-repository)되어 있어야 합니다.

CI 상태 표시줄의 **Auto-fix** 및 **Auto-merge** 토글을 사용하여 옵션을 활성화하세요. Claude Code는 CI가 완료되면 데스크톱 알림도 보냅니다. PR이 병합되거나 닫힐 때 세션을 자동으로 아카이브하려면 Settings → Claude Code에서 [자동 아카이브](#work-in-parallel-with-sessions)를 켜세요.

<Note>
  PR 모니터링에는 머신에 [GitHub CLI (`gh`)](https://cli.github.com/)가 설치되어 있고 인증되어 있어야 합니다. `gh`가 설치되어 있지 않은 경우 Desktop에서 PR을 처음 생성하려고 할 때 설치를 요청합니다.
</Note>

## 작업 공간 배치하기

Code 탭은 원하는 레이아웃으로 배치할 수 있는 창을 중심으로 구축되어 있습니다: chat, diff, browser, terminal, file, plan, tasks 및 subagent, 그리고 macOS에서의 [iOS Simulator](/docs/en/desktop-ios-simulator). 창의 헤더를 드래그하여 위치를 변경하거나, 창 가장자리를 드래그하여 크기를 조정하세요. 포커스된 창을 닫으려면 macOS에서 **Cmd+\\**, Windows에서 **Ctrl+\\**를 누르세요. 세션 툴바의 **Views** 메뉴에서 추가 창을 여세요.

<Note>
  이 섹션의 창 레이아웃, 터미널, 파일 편집기 및 뷰 모드에는 Claude Desktop v1.2581.0 이상이 필요합니다. 업데이트하려면 macOS에서 **Claude → Check for Updates**, Windows에서 **Help → Check for Updates**를 연 후 업데이트하세요.
</Note>

### 터미널에서 명령 실행하기

통합 터미널을 사용하면 다른 앱으로 전환하지 않고도 세션과 함께 명령을 실행할 수 있습니다. **Views** 메뉴에서 열거나 macOS 또는 Windows에서 **Ctrl+\`**를 누르세요. 터미널은 세션의 작업 디렉터리에서 열리고 Claude와 동일한 환경을 공유하므로 `npm test`나 `git status`와 같은 명령이 Claude가 편집 중인 것과 동일한 파일을 봅니다. 두 번째 터미널 탭을 열려면 터미널 창 헤더에서 **+**를 클릭하거나 채팅의 폴더를 마우스 우클릭하여 **Open in terminal**을 선택하세요. 터미널은 로컬 세션에서만 사용할 수 있습니다.

### 파일 열기 및 편집하기

채팅이나 diff 뷰어에서 파일 경로를 클릭하여 파일 창에서 여세요. HTML, PDF, 이미지 및 비디오 경로는 대신 [Browser 창](#preview-your-app)에서 열립니다. 즉석 편집을 수행하고 **Save**를 클릭하여 다시 기록하세요. 연 후 디스크에서 파일이 변경된 경우 창에서 경고가 표시되며 덮어쓰거나 취소할 수 있습니다. 편집 내용을 되돌리려면 **Discard**를 클릭하고, 절대 경로를 복사하려면 창 헤더의 경로를 클릭하세요.

파일 창은 로컬 및 SSH 세션에서 사용할 수 있습니다. 클라우드 세션의 경우 Claude에게 변경을 요청하세요.

### 다른 앱에서 파일 열기

채팅, diff 뷰어 또는 파일 창에서 임의의 파일 경로를 우클릭하여 컨텍스트 메뉴를 엽니다:

* **Attach as context**: 다음 프롬프트에 파일 추가
* **Open in**: VS Code, Cursor 또는 Zed와 같이 설치된 편집기에서 파일 열기
* **Show in Finder** (macOS), **Show in Explorer** (Windows): 포함된 폴더 열기
* **Copy path**: 클립보드에 절대 경로 복사

### 뷰 모드 전환하기

뷰 모드는 채팅 트랜스크립트에 표시되는 상세 수준을 제어합니다. 전송 버튼 옆의 **Transcript view** 드롭다운에서 모드를 전환하거나 macOS 또는 Windows에서 **Ctrl+O**를 눌러 순환하세요.

| 모드 | 표시 내용 |
| ----------- | -------------------------------------------------------------- |
| **Normal** | 도구 호출이 요약으로 축소되고 전체 텍스트 응답이 표시됨 |
| **Verbose** | Claude가 수행하는 모든 도구 호출, 파일 읽기 및 중간 단계 표시 |
| **Summary** | Claude의 최종 응답과 변경 사항만 표시 |

Claude가 특정 조치를 취한 이유를 디버깅할 때는 Verbose를 사용하세요. 여러 세션을 실행하고 결과를 빠르게 스캔하려는 경우 Summary를 사용하세요.

### 키보드 단축키

Code 탭에서 사용할 수 있는 모든 단축키를 보려면 macOS에서 **Cmd+/**, Windows에서 **Ctrl+/**를 누르세요. Windows에서는 아래 단축키에서 **Cmd** 대신 **Ctrl**을 사용하세요. 세션 순환, 터미널 토글 및 뷰 모드 토글은 모든 플랫폼에서 **Ctrl**을 사용합니다.

| 단축키 | 동작 |
| ------------------------------------- | -------------------------------- |
| `Cmd` `/` | 키보드 단축키 표시 |
| `Cmd` `N` | 새 세션 |
| `Cmd` `W` | 세션 닫기 |
| `Ctrl` `Tab` / `Ctrl` `Shift` `Tab` | 다음 또는 이전 세션 |
| `Cmd` `Shift` `]` / `Cmd` `Shift` `[` | 다음 또는 이전 세션 |
| `Esc` | Claude의 응답 중지 |
| `Cmd` `Shift` `D` | diff 창 토글 |
| `Cmd` `Shift` `B` | Browser 창 토글 |
| `Cmd` `Shift` `S` | Browser에서 요소 선택 |
| `Ctrl` `` ` `` | 터미널 창 토글 |
| `Cmd` `\` | 포커스된 창 닫기 |
| `Cmd` `;` | 사이드 챗 열기 |
| `Ctrl` `O` | 뷰 모드 순환 |
| `Cmd` `Shift` `M` | 권한 모드 메뉴 열기 |
| `Cmd` `Shift` `I` | 모델 메뉴 열기 |
| `Cmd` `Shift` `E` | 노력(effort) 메뉴 열기 |
| `1`–`9` | 열린 메뉴에서 항목 선택 |

이 단축키는 Code 탭에만 적용됩니다. 모드를 순환하는 `Shift+Tab`과 같은 터미널 기반 [대화형 모드 단축키](/docs/en/interactive-mode#keyboard-shortcuts)는 Desktop에 적용되지 않습니다.

### 사용량 확인하기

모델 피커 옆의 사용량 링을 클릭하여 현재 컨텍스트 창 사용량과 해당 기간 동안의 플랜 사용량을 확인하세요. 컨텍스트 사용량은 세션별로 집계되며, 플랜 사용량은 모든 Claude Code 영역에 걸쳐 공유됩니다.

## Claude가 컴퓨터를 사용하도록 허용하기

컴퓨터 사용(computer use)을 통해 Claude는 앱을 열고, 화면을 제어하고, 사용자가 하는 방식 그대로 머신에서 직접 작업할 수 있습니다. CLI가 없는 데스크톱 도구와 상호작용하도록 Claude에게 요청하거나, GUI를 통해서만 작동하는 작업을 자동화하세요. iOS 앱을 실행하고 테스트하기 위해 Desktop은 화면을 제어하는 대신 전용 [iOS Simulator 창](/docs/en/desktop-ios-simulator)을 엽니다; 이 창은 컴퓨터 사용을 활성화하지 않고도 작동합니다.

<Note>
  컴퓨터 사용은 Pro 또는 Max 플랜이 필요한 macOS 및 Windows에서의 리서치 프리뷰 기능입니다. Team 또는 Enterprise 플랜에서는 사용할 수 없습니다. Claude Desktop 앱이 실행 중이어야 합니다.
</Note>

컴퓨터 사용은 기본적으로 꺼져 있습니다. Claude가 화면을 제어하기 전에 [Settings에서 활성화](#enable-computer-use)하세요. macOS에서는 Accessibility(손쉬운 사용) 및 Screen Recording(화면 기록) 권한도 부여해야 합니다.

<Warning>
  [샌드박스화된 Bash 도구](/docs/en/sandboxing)와 달리 컴퓨터 사용은 사용자가 승인한 모든 것에 접근할 수 있는 실제 데스크톱에서 실행됩니다. Claude는 각 작업을 확인하고 화면 내용의 잠재적 프롬프트 주입에 플래그를 지정하지만 신뢰 경계가 다릅니다. 모범 사례는 [컴퓨터 사용 안전 가이드](https://support.claude.com/en/articles/14128542)를 참조하세요.
</Warning>

### 컴퓨터 사용이 적용되는 시점

Claude는 앱 또는 서비스와 상호작용하는 여러 방법을 가지고 있으며, 컴퓨터 사용은 가장 광범위하고 가장 완만한 방법입니다. Claude는 가장 정밀한 도구를 먼저 시도합니다:

* 서비스용 [커넥터](#connect-external-tools)가 있는 경우 Claude는 커넥터를 사용합니다.
* 작업이 셸 명령인 경우 Claude는 Bash를 사용합니다.
* 작업이 브라우저 작업이고 [Claude in Chrome](/docs/en/chrome)이 설정되어 있는 경우 Claude는 이를 사용합니다.
* 작업이 iOS 앱 실행 또는 테스트인 경우 Claude는 화면 제어를 사용하지 않는 [iOS Simulator 창](/docs/en/desktop-ios-simulator)을 사용합니다.
* 위의 어느 것도 해당하지 않는 경우 Claude는 컴퓨터 사용을 적용합니다.

[앱별 접근 계층](#app-permissions)은 이를 강화합니다: 브라우저는 보기 전용(view-only)으로 제한되고, 터미널 및 IDE는 클릭 전용(click-only)으로 제한되어 컴퓨터 사용이 활성화되어 있더라도 전용 도구를 사용하도록 유도합니다. 화면 제어는 기본 앱, 하드웨어 제어판 또는 API가 없는 독점 도구와 같이 다른 방법으로 접근할 수 없는 용도로 예약되어 있습니다.

### 컴퓨터 사용 활성화하기

컴퓨터 사용은 기본적으로 꺼져 있습니다. 꺼져 있는 동안 이를 필요로 하는 작업을 Claude에게 요청하면 Claude는 Settings에서 컴퓨터 사용을 활성화하면 해당 작업을 수행할 수 있다고 알려줍니다.

<Steps>
  <Step title="데스크톱 앱 업데이트">
    최신 버전의 Claude Desktop을 가지고 있는지 확인하세요. macOS 및 Windows에서는 [claude.com/download](https://claude.com/download)에서 다운로드하거나 업데이트하고, Linux에서는 패키지 관리자를 통해 업데이트하세요([지침](/docs/en/desktop-linux)). 그런 다음 앱을 다시 시작하세요.
  </Step>

  <Step title="토글 켜기">
    데스크톱 앱에서 **Settings > General** (**Desktop app** 아래)로 이동하세요. **Computer use** 토글을 찾아 켜세요. Windows에서는 토글이 즉시 적용되어 설정이 완료됩니다. macOS에서는 다음 단계로 이동하세요.

    토글이 보이지 않는 경우 Pro 또는 Max 플랜이 적용된 macOS 또는 Windows인지 확인한 후 앱을 업데이트하고 다시 시작하세요.
  </Step>

  <Step title="macOS 권한 부여">
    macOS에서는 토글이 적용되기 전에 두 가지 시스템 권한을 부여하세요:

    * **Accessibility (손쉬운 사용)**: Claude가 클릭하고, 타이핑하고, 스크롤할 수 있도록 허용
    * **Screen Recording (화면 기록)**: Claude가 화면의 내용을 볼 수 있도록 허용

    Settings 페이지에 각 권한의 현재 상태가 표시됩니다. 둘 중 하나라도 거부된 경우 배지를 클릭하여 관련 System Settings 창을 여세요.
  </Step>
</Steps>

### 앱 권한

Claude가 앱을 처음 사용해야 할 때 세션에 프롬프트가 나타납니다. **Allow for this session** 또는 **Deny**를 클릭하세요. 승인은 현재 세션 동안, 또는 [Dispatch로 생성된 세션](#sessions-from-dispatch)에서는 30분 동안 유지됩니다.

프롬프트에는 Claude가 해당 앱에 대해 얻는 제어 수준도 표시됩니다. 이러한 계층은 앱 카테고리별로 고정되어 있으며 변경할 수 없습니다:

| 계층 | Claude가 할 수 있는 작업 | 적용 대상 |
| :----------- | :------------------------------------------------------- | :-------------------------- |
| View only (보기 전용) | 스크린샷으로 앱 확인 | 브라우저, 트레이딩 플랫폼 |
| Click only (클릭 전용) | 클릭 및 스크롤은 가능하지만 타이핑이나 키보드 단축키 사용은 불가 | 터미널, IDE |
| Full control (전체 제어) | 클릭, 타이핑, 드래그 및 키보드 단축키 사용 | 기타 모든 앱 |

터미널, Finder 또는 File Explorer, System Settings 또는 Settings와 같이 범주가 넓은 앱은 승인 시 허용되는 사항을 알 수 있도록 프롬프트에 추가 경고를 표시합니다.

**Settings > General** (**Desktop app** 아래)에서 두 가지 설정을 구성할 수 있습니다:

* **Denied apps**: 묻지 않고 거부하려면 여기에 앱을 추가하세요. Claude는 승인된 앱에서의 작업을 통해 거부된 앱에 간접적으로 영향을 미칠 수 있지만 거부된 앱과 직접 상호작용할 수는 없습니다.
* **Unhide apps when Claude finishes**: Claude가 작업하는 동안 승인된 앱하고만 상호작용하도록 다른 창이 숨겨집니다. Claude가 끝나면 이 설정을 끄지 않는 한 숨겨진 창이 복원됩니다.

## 세션 관리하기

각 세션은 자체 컨텍스트와 변경 사항을 갖는 독립적인 대화입니다. 여러 세션을 병렬로 실행하거나, 사이드 챗으로 분기하거나, 작업을 클라우드로 보내거나, Dispatch가 휴대폰에서 세션을 시작하도록 허용할 수 있습니다.

### 세션으로 병렬 작업하기

사이드바에서 **+ New session**을 클릭하거나 macOS에서 **Cmd+N**, Windows에서 **Ctrl+N**을 눌러 여러 작업을 병렬로 처리하세요. 사이드바의 세션을 순환하려면 **Ctrl+Tab** 및 **Ctrl+Shift+Tab**을 누르세요. Git 리포지토리의 경우 각 세션은 [Git 워크트리](/docs/en/worktrees)를 사용하여 프로젝트의 격리된 사본을 얻으므로, 한 세션의 변경 사항은 커밋할 때까지 다른 세션에 영향을 미치지 않습니다.

두 세션을 한 번에 보려면 macOS에서 **Cmd**, Windows에서 **Ctrl**을 누른 상태에서 사이드바의 세션을 클릭하세요. 연 세션 옆의 두 번째 창에서 세션이 열립니다. 분할이 활성화되어 있는 동안 다른 사이드바 세션을 클릭하면 포커스가 있는 창이 교체됩니다. 포커스된 창을 닫고 단일 세션으로 돌아가려면 macOS에서 **Cmd+\\**, Windows에서 **Ctrl+\\**를 누르세요.

워크트리는 기본적으로 `<project-root>/.claude/worktrees/`에 저장됩니다. Settings → Claude Code의 "Worktree location"에서 이를 커스텀 디렉터리로 변경할 수 있습니다. 또한 모든 워크트리 브랜치 이름 앞에 접두사를 붙이도록 브랜치 접두사를 설정할 수 있으며, 이는 Claude가 생성한 브랜치를 정리하는 데 유용합니다. 작업이 완료되었을 때 워크트리를 제거하려면 사이드바의 세션 위에 마우스를 올리고 아카이브 아이콘을 클릭하세요. 풀 리퀘스트가 병합되거나 닫힐 때 세션이 스스로 아카이브되도록 하려면 Settings → Claude Code에서 **Auto-archive after PR merge or close**를 켜세요. 자동 아카이브는 실행이 완료된 로컬 세션에만 적용됩니다.

새 워크트리에 `.env`와 같이 gitignore 처리된 파일을 포함하려면 프로젝트 루트에 [`.worktreeinclude` 파일](/docs/en/worktrees#copy-gitignored-files-into-worktrees)을 생성하세요.

<Note>
  세션 격리에는 [Git](https://git-scm.com/downloads)이 필요합니다. 대부분의 Mac에는 기본적으로 Git이 포함되어 있습니다. 확인하려면 Terminal에서 `git --version`을 실행하세요. Windows에서는 Code 탭이 작동하려면 Git이 필요합니다: [Git for Windows 다운로드](https://git-scm.com/downloads/win)하여 설치한 후 앱을 다시 시작하세요. Git 오류가 발생하는 경우 [Cowork 탭](https://claude.com/product/cowork)에서 Claude에게 문제 해결 도움을 요청하세요.
</Note>

사이드바 상단의 컨트롤을 사용하여 상태, 프로젝트 또는 환경별로 세션을 필터링하고 프로젝트별로 세션을 그룹화하세요. 세션 이름을 변경하려면 활성 세션 상단 툴바의 세션 제목을 클릭하세요. 컨텍스트 사용량을 확인하려면 [사용량 확인하기](#check-usage)를 참조하세요. 컨텍스트가 가득 차면 Claude가 대화를 자동으로 요약하고 작업을 계속합니다. `/compact`를 입력하여 요약을 더 일찍 트리거하고 컨텍스트 공간을 확보할 수도 있습니다. 압축 작동 방식에 대한 자세한 내용은 [컨텍스트 창](/docs/en/how-claude-code-works#the-context-window)을 참조하세요.

데스크톱 앱은 Code 세션이 작업을 완료하고 현재 해당 세션을 보고 있지 않을 때 OS 알림을 보냅니다.

### 세션을 이탈하지 않고 사이드 질문하기

사이드 챗을 사용하면 메인 대화에 아무것도 추가하지 않고 세션의 컨텍스트를 사용하여 Claude에게 질문할 수 있습니다. 세션을 다른 길로 새게 하지 않고 코드 조각을 이해하거나, 가정을 확인하거나, 아이디어를 탐색하고 싶을 때 사용하세요.

macOS에서 **Cmd+;**, Windows에서 **Ctrl+;**를 누르거나 프롬프트 상자에 `/btw`를 입력하여 사이드 챗을 여세요. 사이드 챗은 해당 시점까지의 메인 스레드에 있는 모든 내용을 읽을 수 있습니다. 완료되면 사이드 챗을 닫고 중단했던 지점에서 메인 세션을 계속하세요. 사이드 챗은 로컬, SSH 및 WSL 세션에서 사용할 수 있습니다.

### 백그라운드 작업 모니터링하기

tasks 창에는 현재 세션 내부에서 실행 중인 백그라운드 작업이 표시됩니다: 서브에이전트, 백그라운드 셸 명령 및 [동적 워크플로](/docs/en/workflows). **Views** 메뉴에서 열거나 레이아웃으로 드래그하세요.

임의의 항목을 클릭하여 subagent 창에서 출력을 확인하거나 중지하세요. 다른 세션이 수행 중인 작업을 보려면 [사이드바](#work-in-parallel-with-sessions)를 사용하세요.

### 장시간 실행되는 작업을 원격으로 실행하기

대규모 리팩토링, 테스트 스위트, 마이그레이션 또는 기타 장시간 실행되는 작업의 경우 세션을 시작할 때 **Local** 대신 **Cloud**를 선택하세요. 클라우드 세션은 Anthropic의 클라우드 인프라에서 실행되며 앱을 닫거나 컴퓨터를 꺼도 계속됩니다. 언제든지 다시 돌아와 진행 상황을 확인하거나 Claude의 방향을 조정할 수 있습니다. [claude.ai/code](https://claude.ai/code) 또는 [Claude 모바일 앱](/docs/en/mobile)에서 클라우드 세션을 모니터링할 수도 있습니다.

클라우드 세션은 여러 리포지토리도 지원합니다. 클라우드 환경을 선택한 후 리포지토리 알약 옆의 **+** 버튼을 클릭하여 세션에 리포지토리를 추가하세요. 각 리포지토리는 자체 브랜치 선택기를 가집니다. 이는 공유 라이브러리와 그 소비자를 업데이트하는 등 여러 코드베이스에 걸쳐 있는 작업에 유용합니다.

클라우드 세션 작동 방식에 대한 자세한 내용은 [웹에서의 Claude Code](/docs/en/claude-code-on-the-web)를 참조하세요.

### 다른 영역에서 계속하기

세션 툴바 오른쪽 하단의 VS Code 아이콘에서 접근할 수 있는 **Continue in** 메뉴를 통해 세션을 다른 영역으로 이동할 수 있습니다:

* **Claude Code on the Web**: 로컬 세션을 원격으로 계속 실행하도록 전송합니다. Desktop이 브랜치를 푸시하고, 대화 요약을 생성하며, 전체 컨텍스트를 보유한 새 클라우드 세션을 생성합니다. 그 후 로컬 세션을 아카이브하거나 유지하도록 선택할 수 있습니다. 이 기능은 깨끗한 작업 트리(clean working tree)가 필요하며, SSH 세션에서는 사용할 수 없습니다.
* **Your IDE**: 현재 작업 디렉터리에서 지원되는 IDE로 프로젝트를 엽니다.

### Dispatch의 세션

[Dispatch](https://support.claude.com/en/articles/13947068)는 [Cowork](https://claude.com/product/cowork) 탭에 존재하는 Claude와의 지속적인 대화입니다. Dispatch에 작업을 메시지로 보내면 이를 처리하는 방법을 결정합니다.

작업은 두 가지 방법으로 Code 세션이 될 수 있습니다: "Claude Code 세션을 열고 로그인 버그를 수정해 줘"와 같이 직접 요청하거나, Dispatch가 해당 작업이 개발 작업이라고 판단하여 스스로 세션을 생성하는 경우입니다. 일반적으로 Code로 라우팅되는 작업에는 버그 수정, 종속성 업데이트, 테스트 실행 또는 풀 리퀘스트 오픈이 포함됩니다. 리서치, 문서 편집 및 스프레드시트 작업은 Cowork에 남아 있습니다.

어느 쪽이든 Code 세션은 **Dispatch** 배지와 함께 Code 탭의 사이드바에 나타납니다. 완료되거나 승인이 필요할 때 휴대폰으로 푸시 알림을 받게 됩니다.

[컴퓨터 사용](#let-claude-use-your-computer)이 활성화되어 있으면 Dispatch로 생성된 Code 세션도 이를 사용할 수 있습니다. 해당 세션의 앱 승인은 일반 Code 세션처럼 세션 전체 동안 유지되는 대신 30분 후 만료되고 다시 프롬프트를 표시합니다.

설정, 페어링 및 Dispatch 설정은 [Dispatch 도움말 문서](https://support.claude.com/en/articles/13947068)를 참조하세요. Dispatch에는 Pro 또는 Max 플랜이 필요하며 Team 또는 Enterprise 플랜에서는 사용할 수 없습니다.

Dispatch는 터미널을 떠나 있을 때 Claude와 함께 작업할 수 있는 여러 방법 중 하나입니다. Remote Control, Channels, Slack 및 예약된 작업과의 비교는 [플랫폼 및 연동](/docs/en/platforms#work-when-you-are-away-from-your-terminal)을 참조하세요.

## Claude Code 확장하기

외부 서비스를 연결하고, 재사용 가능한 워크플로를 추가하고, Claude의 동작을 맞춤화하고, 미리보기 서버를 구성하세요. 커넥터, 스킬 및 플러그인을 한 곳에서 관리하려면 사이드바에서 **Customize**를 클릭하세요. Desktop 앱의 [Cowork](https://claude.com/product/cowork) 탭은 CLI의 `~/.claude` 디렉터리가 아닌 claude.ai 계정을 통해 동기화되는 이 Customize 구성에서 스킬, 플러그인 및 커넥터를 가져옵니다.

### 외부 도구 연결하기

로컬 및 [SSH](#ssh-sessions) 세션의 경우 프롬프트 상자 옆의 **+** 버튼을 클릭하고 **Connectors**를 선택하여 Google Calendar, Slack, GitHub, Linear, Notion 등과 같은 연동 기능을 추가하세요. 세션 전이나 세션 중에 커넥터를 추가할 수 있습니다. **+** 버튼은 클라우드 또는 WSL 세션에서는 사용할 수 없지만, [루틴](/docs/en/routines)은 루틴 생성 시점에 커넥터를 구성합니다.

커넥터를 관리하거나 연결 해제하려면 데스크톱 앱의 Settings → Connectors로 이동하거나 프롬프트 상자의 Connectors 메뉴에서 **Manage connectors**를 선택하세요.

연결되면 Claude가 달력을 읽고, 메시지를 보내고, 이슈를 생성하고, 도구와 직접 상호작용할 수 있습니다. 세션에 어떤 커넥터가 구성되어 있는지 Claude에게 물어볼 수 있습니다.

커넥터는 그래픽 설정 흐름을 가진 [MCP 서버](/docs/en/mcp)입니다. 지원되는 서비스와의 빠른 연동을 위해 사용하세요. Connectors에 나열되지 않은 연동의 경우 [설정 파일](/docs/en/mcp#installing-mcp-servers)을 통해 MCP 서버를 수동으로 추가하세요. [커스텀 커넥터 생성](https://support.claude.com/en/articles/11175166-getting-started-with-custom-connectors-using-remote-mcp)도 가능합니다.

### 스킬 사용하기

[스킬](/docs/en/skills)은 Claude가 할 수 있는 일을 확장합니다. Claude는 관련이 있을 때 이를 자동으로 로드하거나 직접 호출할 수 있습니다: 프롬프트 상자에 `/`를 입력하거나 **+** 버튼을 클릭하고 **Slash commands**를 선택하여 사용 가능한 명령을 탐색하세요. 여기에는 [내장 명령](/docs/en/commands), [커스텀 스킬](/docs/en/skills#create-your-first-skill), 코드베이스의 프로젝트 스킬, [설치된 플러그인](/docs/en/plugins)의 스킬이 포함됩니다. 하나를 선택하면 입력 필드에 강조되어 나타납니다. 그 뒤에 작업을 입력하고 평소처럼 전송하세요.

Claude가 작업하는 동안 다른 메시지와 동일하게 명령을 보낼 수 있으며, 턴이 끝나면 세션은 대기 상태로 돌아갑니다. v2.1.206 이전에는 턴 도중에 전송된 명령으로 인해 세션이 실행 중인 것으로 계속 표시되고 이후 전송한 메시지가 전달되지 않을 수 있었습니다.

`~/.claude/skills/`에 있는 개인 스킬은 로컬 세션에 적용됩니다; [SSH](#ssh-sessions) 세션은 사용자의 머신이 아니라 원격 호스트의 홈 디렉터리에서 `~/.claude/skills/`를 읽습니다. 클라우드 세션은 대신 claude.ai 계정에 활성화된 스킬을 로드합니다. [Cowork 및 클라우드 세션에서의 스킬](/docs/en/skills#skills-in-cowork-and-cloud-sessions)을 참조하세요.

### 플러그인 설치하기

[플러그인](/docs/en/plugins)은 스킬, 에이전트, 훅, MCP 서버 및 LSP 구성을 Claude Code에 추가하는 재사용 가능한 패키지입니다. 터미널을 사용하지 않고도 데스크톱 앱에서 플러그인을 설치할 수 있습니다.

로컬 및 [SSH](#ssh-sessions) 세션의 경우 프롬프트 상자 옆의 **+** 버튼을 클릭하고 **Plugins**를 선택하여 설치된 플러그인 및 스킬을 확인하세요. 플러그인을 추가하려면 서브메뉴에서 **Add plugin**을 선택하여 공식 Anthropic 마켓플레이스를 포함하여 구성된 [마켓플레이스](/docs/en/plugin-marketplaces)에서 사용 가능한 플러그인을 보여주는 플러그인 브라우저를 여세요. 플러그인을 활성화, 비활성화 또는 제거하려면 **Manage plugins**를 선택하세요.

플러그인은 사용자 계정, 특정 프로젝트 또는 로컬 전용으로 범위를 지정할 수 있습니다. 조직에서 플러그인을 중앙에서 관리하는 경우 CLI에서와 마찬가지로 데스크톱 세션에서도 해당 플러그인을 사용할 수 있습니다. 플러그인 브라우저는 클라우드 세션에서는 사용할 수 없으며 데스크톱 앱에서 설치한 플러그인은 클라우드 세션에서 사용할 수 없습니다; 클라우드 세션에서 플러그인을 사용하려면 리포지토리의 `.claude/settings.json` 내 [`enabledPlugins`](/docs/en/settings#enabledplugins) 아래에 선언하여 [세션 시작 시 설치](/docs/en/claude-code-on-the-web#what’s-available-in-cloud-sessions)되도록 하세요. 플러그인은 WSL 세션에서는 사용할 수 없습니다. 직접 플러그인을 만드는 방법을 포함한 전체 플러그인 참조는 [플러그인](/docs/en/plugins)을 참조하세요.

### 미리보기 서버 구성하기

Claude는 개발 서버 설정을 자동으로 감지하고 세션을 시작할 때 선택한 폴더의 루트에 있는 `.claude/launch.json`에 구성을 저장합니다. 미리보기는 이 폴더를 작업 디렉터리로 사용하므로 상위 폴더를 선택한 경우 자체 개발 서버가 있는 하위 폴더는 자동으로 감지되지 않습니다. 하위 폴더의 서버로 작업하려면 해당 폴더에서 세션을 직접 시작하거나 구성을 수동으로 추가하세요.

서버가 시작되는 방식을 변경하려면(예: `npm run dev` 대신 `yarn dev`를 사용하거나 포트를 변경하려는 경우) 파일을 수동으로 편집하거나 서버 드롭다운에서 **Edit configuration**을 클릭하여 코드 편집기에서 여세요. 이 파일은 주석이 포함된 JSON을 지원합니다.

```json theme={null}
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "my-app",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "port": 3000
    }
  ]
}
```

프론트엔드와 API 등 동일한 프로젝트에서 서로 다른 서버를 실행하도록 여러 구성을 정의할 수 있습니다. 아래 [예제](#examples)를 참조하세요.

#### 변경 사항 자동 검증

`autoVerify`가 활성화되면 Claude가 파일을 편집한 후 코드 변경 사항을 자동으로 검증합니다. 스크린샷을 찍고, 오류를 확인하며, 응답을 완료하기 전에 변경 사항이 제대로 작동하는지 확인합니다.

자동 검증은 기본적으로 켜져 있습니다. 프로젝트별로 비활성화하려면 `.claude/launch.json`에 `"autoVerify": false`를 추가하거나 서버 드롭다운 메뉴에서 토글하세요.

```json theme={null}
{
  "version": "0.0.1",
  "autoVerify": false,
  "configurations": [...]
}
```

비활성화되어도 미리보기 도구는 계속 사용할 수 있으며 언제든지 Claude에게 검증을 요청할 수 있습니다. 자동 검증은 매 편집 후 이를 자동으로 수행합니다.

#### 구성 필드

`configurations` 배열의 각 항목은 다음 필드를 허용합니다:

| 필드 | 유형 | 설명 |
| ------------------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `name` | string | 이 서버에 대한 고유 식별자 |
| `runtimeExecutable` | string | 실행할 명령(예: `npm`, `yarn`, `node`) |
| `runtimeArgs` | string\[] | `runtimeExecutable`에 전달되는 인수(예: `["run", "dev"]`) |
| `port` | number | 서버가 수신 대기하는 포트입니다. 기본값은 3000입니다. |
| `cwd` | string | 프로젝트 루트에 대한 상대 작업 디렉터리입니다. 기본값은 프로젝트 루트입니다. 프로젝트 루트를 명시적으로 참조하려면 `${workspaceFolder}`를 사용하세요. |
| `env` | object | `{ "NODE_ENV": "development" }`와 같이 키-값 쌍 형태의 추가 환경 변수입니다. 이 파일은 리포지토리에 커밋되므로 여기에 보안 비밀을 넣지 마세요. 개발 서버에 보안 비밀을 전달하려면 [로컬 환경 편집기](#local-sessions)에서 설정하세요. |
| `autoPort` | boolean | 포트 충돌 처리 방식입니다. 아래 참조 |
| `program` | string | `node`로 실행할 스크립트입니다. [`program` 대 `runtimeExecutable` 사용 시기](#when-to-use-program-vs-runtimeexecutable)를 참조하세요. |
| `args` | string\[] | `program`에 전달되는 인수입니다. `program`이 설정된 경우에만 사용됩니다. |
| `url` | string | 미리보기가 `http://localhost:<port>` 대신 여는 주소입니다. [특정 URL에서 미리보기 열기](#open-the-preview-at-a-specific-url)를 참조하세요. |

<a id="when-to-use-program-vs-runtimeexecutable" />

##### `program` 대 `runtimeExecutable` 사용 시기

패키지 관리자를 통해 개발 서버를 시작하려면 `runtimeArgs`와 함께 `runtimeExecutable`을 사용하세요. 예를 들어 `"runtimeExecutable": "npm"`에 `"runtimeArgs": ["run", "dev"]`를 지정하면 `npm run dev`가 실행됩니다.

`node`로 직접 실행할 독립 실행형 스크립트가 있는 경우 `program`을 사용하세요. 예를 들어 `"program": "server.js"`는 `node server.js`를 실행합니다. 추가 플래그는 `args`로 전달하세요.

<a id="open-the-preview-at-a-specific-url" />

##### 특정 URL에서 미리보기 열기

기본적으로 미리보기는 `http://localhost:<port>`를 엽니다. 서버에 다른 주소가 필요한 경우 `url`을 설정하세요. 일반적인 경우는 로컬 HTTPS가 필요한 서버, `*.localhost` 서브도메인을 사용하는 앱, 리다이렉션을 통해 로그인하는 앱입니다.

```json theme={null}
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "my-app",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "dev"],
      "port": 8443,
      "url": "https://localhost:8443"
    }
  ]
}
```

Localhost 주소는 기본 포트 주소와 똑같이 직접 열립니다. 여기에는 `localhost`, 임의의 `*.localhost` 서브도메인, `127.0.0.1` 및 `::1`이 포함됩니다. 보안을 위해 localhost `url`은 서버의 출처(origin)여야 합니다(경로나 쿼리가 없고 포트가 항목의 포트와 일치해야 함). 특정 페이지를 표시하려면 미리보기가 열린 후 Claude에게 그리로 이동하도록 요청하세요. 경로, 쿼리 또는 불일치 포트가 있는 localhost `url`은 url 이름을 밝히고 수정 사항을 보여주는 구성 오류로 보고됩니다.

다른 주소의 경우 미리보기에서 새 사이트로 이동할 때와 동일하게 처음 열릴 때 사용자의 권한을 요청합니다. 외부 주소에는 경로가 포함될 수 있습니다. 향후 해당 사이트에 대한 프롬프트를 건너뛰려면 **Always allow**를 선택하세요. 미리보기에서 외부 사이트를 제한하는 조직 정책은 여전히 적용됩니다.

직접 이미 실행 중인 서버를 미리 보려면 명령 없이 `url`을 설정하세요. Claude는 새 서버를 시작하는 대신 실행 중인 서버에 미리보기를 연결합니다:

```json theme={null}
{
  "version": "0.0.1",
  "configurations": [
    {
      "name": "my-app",
      "url": "https://app.localhost:3000"
    }
  ]
}
```

`url`은 `http` 또는 `https`여야 하며 사용자 이름이나 비밀번호를 포함해서는 안 됩니다.

#### 포트 충돌

`autoPort` 필드는 선호하는 포트가 이미 사용 중일 때 발생하는 상황을 제어합니다:

* **`true`**: Claude가 사용 가능한 포트를 자동으로 찾아 사용합니다. 대부분의 개발 서버에 적합합니다.
* **`false`**: Claude가 오류와 함께 실패합니다. OAuth 콜백이나 CORS 허용 목록 등 서버가 특정 포트를 반드시 사용해야 할 때 이를 사용하세요.
* **미설정 (기본값)**: Claude가 서버에 해당 지정 포트가 필요한지 물어본 후 답변을 저장합니다.

Claude가 다른 포트를 선택하면 지정된 포트를 `PORT` 환경 변수를 통해 서버에 전달합니다.

#### 예제

이 구성들은 서로 다른 프로젝트 유형에 대한 일반적인 설정을 보여줍니다:

<Tabs>
  <Tab title="Next.js">
    이 구성은 포트 3000에서 Yarn을 사용하여 Next.js 앱을 실행합니다:

    ```json theme={null}
    {
      "version": "0.0.1",
      "configurations": [
        {
          "name": "web",
          "runtimeExecutable": "yarn",
          "runtimeArgs": ["dev"],
          "port": 3000
        }
      ]
    }
    ```
  </Tab>

  <Tab title="Multiple servers">
    프론트엔드와 API 서버가 있는 모노리포의 경우 여러 구성을 정의하세요. 프론트엔드는 `autoPort: true`를 사용하여 3000 포트가 사용 중이면 사용 가능한 포트를 선택하고, API 서버는 정확히 8080 포트를 요구합니다:

    ```json theme={null}
    {
      "version": "0.0.1",
      "configurations": [
        {
          "name": "frontend",
          "runtimeExecutable": "npm",
          "runtimeArgs": ["run", "dev"],
          "cwd": "apps/web",
          "port": 3000,
          "autoPort": true
        },
        {
          "name": "api",
          "runtimeExecutable": "npm",
          "runtimeArgs": ["run", "start"],
          "cwd": "server",
          "port": 8080,
          "env": { "NODE_ENV": "development" },
          "autoPort": false
        }
      ]
    }
    ```
  </Tab>

  <Tab title="Node.js script">
    패키지 관리자 명령을 사용하는 대신 Node.js 스크립트를 직접 실행하려면 `program` 필드를 사용하세요:

    ```json theme={null}
    {
      "version": "0.0.1",
      "configurations": [
        {
          "name": "server",
          "program": "server.js",
          "args": ["--verbose"],
          "port": 4000
        }
      ]
    }
    ```
  </Tab>
</Tabs>

## 환경 구성

[세션을 시작할 때](#start-a-session) 선택하는 환경에 따라 Claude가 실행되는 위치와 연결 방식이 결정됩니다:

* **Local**: 파일에 직접 접근하여 머신에서 실행됩니다.
* **Cloud**: Anthropic의 클라우드 인프라에서 실행됩니다. 앱을 닫아도 세션이 유지됩니다.
* **SSH**: 자체 서버, 클라우드 VM 또는 개발 컨테이너와 같이 SSH로 연결하는 원격 머신에서 실행됩니다.
* **WSL** (Windows): Linux 툴체인과 네이티브 경로를 사용하여 머신의 [WSL 2 배포판](/docs/en/desktop-wsl) 내부에서 실행됩니다.

### 로컬 세션

데스크톱 앱이 항상 전체 셸 환경을 상속받는 것은 아닙니다. macOS에서 Dock 또는 Finder로부터 앱을 실행할 때 `~/.zshrc` 또는 `~/.bashrc`와 같은 셸 프로필을 읽어 `PATH` 및 고정된 Claude Code 변수 세트를 추출하지만, 거기서 내보낸(export) 다른 변수는 선택되지 않습니다. Windows에서 앱은 사용자 및 시스템 환경 변수를 상속받지만 PowerShell 프로필을 읽지는 않습니다.

모든 플랫폼에서 로컬 세션 및 개발 서버의 환경 변수를 설정하려면 프롬프트 상자의 환경 드롭다운을 열고 **Local** 위에 마우스를 올린 후 톱니바퀴 아이콘을 클릭하여 로컬 환경 편집기를 여세요. 여기에 저장한 변수는 머신에 암호화되어 저장되며 시작하는 모든 로컬 세션 및 미리보기 서버에 적용됩니다. `~/.claude/settings.json` 파일의 `env` 키에 변수를 추가할 수도 있지만, 이는 Claude 세션에만 적용되고 개발 서버에는 도달하지 않습니다. 지원되는 전체 변수 목록은 [환경 변수](/docs/en/env-vars)를 참조하세요.

[Extended thinking(확장 사고)](/docs/en/model-config#extended-thinking)은 기본적으로 활성화되어 있어 복잡한 추론 작업의 성능을 향상시키지만 추가 토큰을 사용합니다. 사고 기능을 비활성화하려면 로컬 환경 편집기에서 `MAX_THINKING_TOKENS`를 `0`으로 설정하세요; 항상 확장 사고를 사용하는 Fable 5에는 영향이 없습니다. [제3자 제공업체](/docs/en/third-party-integrations)에서는 `0` 지정 시 `thinking` 매개변수가 생략되며 적응형 추론 모델은 여전히 사고를 수행할 수 있습니다. [적응형 추론](/docs/en/model-config#adjust-effort-level)이 적용된 모델에서는 적응형 추론이 사고 깊이를 제어하므로 다른 `MAX_THINKING_TOKENS` 값이 무시됩니다. Opus 4.6 및 Sonnet 4.6에서는 `CLAUDE_CODE_DISABLE_ADAPTIVE_THINKING`을 `1`로 설정하여 고정된 사고 예산을 사용하세요; Fable 5, Sonnet 5, 그리고 Opus 4.7 이상은 항상 적응형 추론을 사용하며 고정 예산 모드가 없습니다.

### 클라우드 세션

클라우드 세션은 앱을 닫아도 백그라운드에서 계속됩니다. 사용량은 별도의 컴퓨팅 요금 없이 [구독 플랜 한도](/docs/en/costs)에 반영됩니다.

다양한 네트워크 접근 수준과 환경 변수를 가진 커스텀 클라우드 환경을 생성할 수 있습니다. 클라우드 세션을 시작할 때 환경 드롭다운을 선택하고 **Add environment**를 선택하세요. 네트워크 접근 및 환경 변수 구성에 대한 자세한 내용은 [클라우드 환경](/docs/en/claude-code-on-the-web#the-cloud-environment)을 참조하세요.

### SSH 세션

SSH 세션을 사용하면 데스크톱 앱을 인터페이스로 사용하면서 원격 머신에서 Claude Code를 실행할 수 있습니다. 이는 클라우드 VM, 개발 컨테이너 또는 특정 하드웨어나 종속성이 있는 서버에 상주하는 코드베이스로 작업할 때 유용합니다.

SSH 연결을 추가하려면 세션을 시작하기 전에 환경 드롭다운을 클릭하고 **+ Add SSH connection**을 선택하세요. 대화 상자에서 다음을 요청합니다:

* **Name**: 이 연결에 대한 이름
* **SSH Host**: `user@hostname` 또는 `~/.ssh/config`에 정의된 호스트
* **SSH Port**: 비워 두면 기본값 22가 사용되거나 SSH 구성의 포트가 사용됩니다.
* **Identity File**: `~/.ssh/id_rsa`와 같은 개인 키 경로입니다. 기본 키나 SSH 구성을 사용하려면 비워 두세요.

추가되면 연결이 환경 드롭다운에 나타납니다. 선택하여 해당 머신에서 세션을 시작하세요. Claude는 파일 및 도구에 접근할 수 있는 원격 머신에서 실행됩니다.

원격 머신은 Linux 또는 macOS를 실행해야 합니다. Desktop은 처음 연결할 때 원격 머신에 Claude Code를 자동으로 설치합니다. 연결되면 SSH 세션은 권한 모드, 커넥터, 플러그인 및 MCP 서버를 지원합니다.

#### 팀을 위한 SSH 연결 사전 구성

관리자는 [관리형 설정](/docs/en/settings#settings-precedence) 파일에 `sshConfigs`를 추가하여 팀원에게 SSH 연결을 배포할 수 있습니다. 이 방식으로 정의된 연결은 각 사용자의 환경 드롭다운에 자동으로 나타나고 관리형으로 표시되므로 사용자가 선택할 수는 있지만 앱에서 편집하거나 삭제할 수는 없습니다.

다음 예제는 원격 호스트의 `~/projects`에서 열리는 단일 연결을 사전 구성합니다:

```json theme={null}
{
  "sshConfigs": [
    {
      "id": "shared-dev-vm",
      "name": "Shared Dev VM",
      "sshHost": "user@dev.example.com",
      "sshPort": 22,
      "sshIdentityFile": "~/.ssh/id_ed25519",
      "startDirectory": "~/projects"
    }
  ]
}
```

각 항목에는 `id`, `name`, `sshHost`가 필요합니다. `sshPort`, `sshIdentityFile`, `startDirectory` 필드는 선택 사항입니다. 사용자는 본인의 `~/.claude/settings.json`에 `sshConfigs`를 추가할 수도 있으며, 대화 상자를 통해 추가된 연결이 여기에 저장됩니다.

#### 사용자가 연결할 수 있는 SSH 호스트 제한

관리자는 [관리형 설정](/docs/en/settings#settings-precedence) 파일에 `sshHostAllowlist`를 추가하여 Desktop의 SSH 세션을 승인된 호스트 세트로 제한할 수 있습니다. 설정하면 사용자는 확인된 호스트 이름이 패턴 중 하나와 일치하는 호스트에만 연결할 수 있습니다. SSH 세션을 완전히 비활성화하려면 빈 배열로 설정하세요.

다음 예제는 `devboxes.example.com` 아래의 모든 호스트 및 지정된 단일 배스천(bastion) 호스트에 대한 연결을 허용합니다:

```json theme={null}
{
  "sshHostAllowlist": ["*.devboxes.example.com", "bastion.example.com"]
}
```

패턴은 대소문자를 구분하지 않습니다. `*`는 모든 호스트와 일치하고 `*.example.com`은 `example.com` 및 모든 서브도메인과 일치합니다. 다른 모든 것은 정확히 일치해야 합니다. 검사는 `ssh -G`를 통한 `~/.ssh/config` 확인 후의 호스트 이름에 대해 실행되므로, 확인된 `HostName`이 일치하는 한 `Host` 별칭 및 `ProxyCommand`/`ProxyJump` 항목이 허용됩니다.

`sshHostAllowlist`는 관리형 설정에서만 읽습니다; 사용자 또는 프로젝트 설정의 값은 무시됩니다. Claude Desktop 앱만 이 설정을 준수합니다; Claude Code CLI 및 IDE 확장 프로그램은 이를 읽지 않으며, Bash 도구를 통해 실행되는 `ssh` 명령을 제한하지 않습니다. 강력한 경계가 필요한 경우 조직의 네트워크 또는 제로 트러스트 제어와 함께 사용하세요.

## 엔터프라이즈 구성

Team 또는 Enterprise 플랜의 조직은 관리 콘솔 제어, 관리형 설정 파일 및 기기 관리 정책을 통해 데스크톱 앱 동작을 관리할 수 있습니다.

### 관리 콘솔 제어

이러한 설정은 [관리 설정 콘솔](https://claude.ai/admin-settings/claude-code)을 통해 구성됩니다:

* **Code in the desktop**: 조직 사용자가 데스크톱 앱에서 Claude Code에 접근할 수 있는지 여부를 제어
* **Code in the web**: 조직의 [웹 세션](/docs/en/claude-code-on-the-web) 활성화 또는 비활성화
* **Remote Control**: 조직의 [Remote Control](/docs/en/remote-control) 활성화 또는 비활성화
* **Disable Bypass permissions mode**: 조직 사용자가 바이패스 권한 모드를 활성화하지 못하도록 방지

### 관리형 설정

관리형 설정은 프로젝트 및 사용자 설정을 재정의하며 Desktop의 Claude Code 세션에 적용됩니다. 조직의 [관리형 설정](/docs/en/settings#settings-precedence) 파일에 이러한 키를 설정하거나 관리 콘솔을 통해 원격으로 푸시할 수 있습니다.

| 키 | 설명 |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `permissions.disableBypassPermissionsMode` | 사용자가 Bypass permissions 모드를 활성화하지 못하도록 하려면 `"disable"`로 설정하세요. |
| `disableAutoMode` | 사용자가 [Auto](/docs/en/permission-modes#eliminate-prompts-with-auto-mode) 모드를 활성화하지 못하도록 하려면 `"disable"`로 설정하세요. 모드 선택기에서 Auto를 제거합니다. `permissions` 아래에서도 허용됩니다. |
| `autoMode` | 조직 전체에서 자동 모드 분류기가 신뢰하고 차단하는 대상을 지정하세요. [Auto 모드 구성](/docs/en/auto-mode-config)을 참조하세요. |
| `browserExternalPageTools` | Claude가 [Browser 창](#browse-external-sites)의 외부 페이지를 읽거나 조치를 취하는 도구를 사용하지 못하도록 하려면 `"disabled"`로 설정하세요. 사용자는 여전히 외부 사이트를 직접 둘러볼 수 있으며 로컬 개발 서버 미리보기에는 영향을 주지 않습니다. |
| `disableMobileSimulatorTools` | [iOS Simulator 창](/docs/en/desktop-ios-simulator#turn-off-simulator-access)에서 장치를 제어하고 캡처하기 위한 Claude의 도구를 차단하려면 `true`로 설정하세요. 사용자의 탭 작업에는 창을 계속 사용할 수 있으며 Claude의 접근만 제거됩니다. |
| `disableBrowserExternalNavigation` | [Browser 창](#browse-external-sites)에서 외부 탐색을 완전히 끄려면 `true`로 설정하세요. 사용자나 Claude 모두 외부 사이트로 이동할 수 없으며 localhost 개발 서버 미리보기에는 영향을 주지 않습니다. 값은 JSON 불리언 `true`여야 하며 문자열 `"true"`는 무시됩니다. |
| `sshConfigs` | 환경 드롭다운에 나타나는 [SSH 연결](#pre-configure-ssh-connections-for-your-team)을 사전 구성하세요. 사용자는 관리형 연결을 편집하거나 삭제할 수 없습니다. |
| `sshHostAllowlist` | 확인된 호스트 이름이 이러한 패턴 중 하나와 일치하는 호스트로 [SSH 세션](#restrict-which-ssh-hosts-users-can-connect-to)을 제한합니다. 빈 배열은 SSH 세션을 비활성화합니다. 관리형 설정에서만 읽습니다. |
| `managedMcpServers` | 제3자 배포의 모든 사용자에게 MCP 서버 구성을 푸시하세요. 각 항목은 `"http"`, `"sse"`, 또는 `"stdio"` 트랜스포트, 연결 세부 정보, 그리고 옵션으로 사용자가 해당 서버에서 호출할 수 있는 도구를 제한하는 `toolPolicy` 맵을 지정합니다. 제3자(3P) Desktop 배포에서만 사용할 수 있습니다. 제3자 배포는 관리 콘솔 설정을 수신하지 않으므로 관리형 설정 파일이나 MDM을 통해 이 키를 전달하세요. |

Desktop 세션에 적용되는 관리형 설정은 해당 세션이 실행되는 위치에 따라 다릅니다. [`availableModels`](/docs/en/model-config#restrict-model-selection)과 같은 모델 제한은 터미널 CLI에서와 동일하게 Desktop의 Claude Code 세션에 적용됩니다; [영역 범위](/docs/en/model-config#surface-coverage)를 참조하세요.

* **이 머신의 로컬 세션**: 디스크에 배포된 관리형 설정 파일이 적용됩니다. 관리 콘솔을 통해 원격으로 푸시된 관리형 설정은 세션이 조직 로그인 또는 직접 구성된 API 키로 인증될 때 Anthropic API의 해당 세션에도 도달하며 터미널 CLI와 동일한 [설정 우선순위](/docs/en/settings#settings-precedence)를 따릅니다.
* **[클라우드 세션](#cloud-sessions)**: Anthropic 관리형 VM에서 실행되며 [서버 관리형 설정](/docs/en/server-managed-settings)만 수신합니다.
* **[SSH 세션](#ssh-sessions)**: 세션이 원격 호스트에서 관리형 설정 파일을 읽습니다. Desktop 자체는 연결을 생성할 때 로컬 머신의 관리형 설정에서 `sshConfigs` 및 `sshHostAllowlist`를 읽습니다.

`permissions.disableBypassPermissionsMode` 및 `disableAutoMode`는 사용자 및 프로젝트 설정에서도 작동하지만 관리형 설정에 배치하면 사용자가 재정의하지 못하게 됩니다.

{/* min-version: 2.1.207 */}Claude Code는 사용자 설정, `--settings` 플래그 및 관리형 설정에서 `autoMode`를 읽지만, `.claude/settings.json` 또는 `.claude/settings.local.json`에서는 읽지 않습니다: 두 파일 모두 리포지토리 디렉터리에 상주하므로 복제된 리포지토리나 빌드 단계에서 자체 분류기 규칙을 주입할 수 없습니다. v2.1.207 이전에는 Claude Code가 `.claude/settings.local.json`도 읽었습니다.

`allowManagedPermissionRulesOnly` 및 `allowManagedHooksOnly`를 포함한 전체 관리 전용 설정 목록은 [관리 전용 설정](/docs/en/permissions#managed-only-settings)을 참조하세요.

### 기기 관리 정책

IT 팀은 macOS의 MDM 또는 Windows의 그룹 정책을 통해 데스크톱 앱을 관리할 수 있습니다. 사용할 수 있는 정책에는 Claude Code 기능 활성화 또는 비활성화, 자동 업데이트 제어, 커스텀 배포 URL 설정이 포함됩니다.

* **macOS**: Jamf나 Kandji와 같은 도구를 사용하여 `com.anthropic.claudefordesktop` 환경설정 도메인을 통해 구성
* **Windows**: `SOFTWARE\Policies\Claude` 레지스트리를 통해 구성

### 네트워크 접근 요건

Desktop은 Anthropic CDN 호스트에서 애플리케이션 코드 및 사용자 콘텐츠를 로드합니다.

```text theme={null}
anthropic.com
*.anthropic.com
claude.ai
*.claude.ai
claude.com
*.claude.com
claude.app
*.claude.app
*.claudeusercontent.com
*.claudemcpcontent.com
```

[OTLP](/docs/en/monitoring-usage), LLM 게이트웨이 또는 MCP 서버에 커스텀 포트를 구성하지 않는 한 트래픽은 포트 443의 HTTPS입니다.

프록시 서버, 커스텀 인증기관(CA), mTLS 및 독립 실행형 CLI가 필요로 하는 도메인은 [네트워크 구성](/docs/en/network-config)을 참조하세요.

방화벽 와일드카드 수를 줄이려면 다음 Anthropic 호스트를 대신 허용하세요. 특정 서브도메인은 동적으로 생성되므로 와일드카드로 유지되어야 합니다.

```text theme={null}
anthropic.com
api.anthropic.com
a-api.anthropic.com
a-cdn.anthropic.com
s-cdn.anthropic.com
assets-proxy.anthropic.com
claude.ai
a.claude.ai
a-cdn.claude.ai
assets.claude.ai
downloads.claude.ai
*.livepreview.claude.ai
claude.com
platform.claude.com
*.livepreview.claude.app
*.claudeusercontent.com
*.claudemcpcontent.com
```

### 인증 및 SSO

엔터프라이즈 조직은 모든 사용자에게 SSO를 요구할 수 있습니다. 플랜 수준 세부 정보는 [인증](/docs/en/authentication)을, SAML 구성은 [SSO 설정](https://support.claude.com/en/articles/13132885-setting-up-single-sign-on-sso)을 참조하세요. OIDC 설정은 [Claude Enterprise Administrator Guide](https://claude.com/resources/tutorials/claude-enterprise-administrator-guide)에서 다룹니다.

### 데이터 처리

Claude Code는 로컬 세션의 경우 로컬에서, 클라우드 세션의 경우 Anthropic의 클라우드 인프라에서 코드를 처리합니다. 대화 및 코드 컨텍스트는 처리를 위해 Anthropic의 API로 전송됩니다. 데이터 보관, 개인정보 보호 및 준수에 대한 자세한 내용은 [데이터 처리](/docs/en/data-usage)를 참조하세요.

### 배포

Desktop은 엔터프라이즈 배포 도구를 통해 배포할 수 있습니다:

* **macOS**: `.dmg` 설치 프로그램을 사용하여 Jamf 또는 Kandji와 같은 MDM을 통해 배포
* **Windows**: MSIX 패키지를 통해 배포. 무소음 설치를 포함한 엔터프라이즈 배포 옵션은 [Deploy Claude Desktop for Windows](https://support.claude.com/en/articles/12622703-deploy-claude-desktop-for-windows)를 참조하세요.

방화벽에 허용 목록으로 추가할 도메인은 위의 [네트워크 접근 요건](#network-access-requirements)을 참조하세요. 프록시 설정, 커스텀 인증기관 및 LLM 게이트웨이는 [네트워크 구성](/docs/en/network-config)을 참조하세요.

전체 엔터프라이즈 구성 참조는 [엔터프라이즈 구성 가이드](https://support.claude.com/en/articles/12622667-enterprise-configuration)를 참조하세요.

## CLI에서 오셨나요?

이미 Claude Code CLI를 사용 중이라면 Desktop은 그래픽 인터페이스와 함께 동일한 기본 엔진을 실행합니다. 동일한 프로젝트에 대해서도 동일한 머신에서 두 환경을 동시에 실행할 수 있습니다. 각 환경은 별도의 세션 기록을 유지하지만 CLAUDE.md 파일을 통해 구성 및 프로젝트 메모리를 공유합니다.

CLI 세션을 Desktop으로 이동하려면 터미널에서 `/desktop`을 실행하세요. Claude가 세션을 저장하고 데스크톱 앱에서 연 다음 CLI를 종료합니다. 이 명령은 Claude 구독으로 로그인된 macOS 및 Windows에서 사용할 수 있습니다. API 키 인증 시나 Amazon Bedrock, Google Cloud's Agent Platform, 또는 Microsoft Foundry에서는 사용할 수 없습니다.

<Tip>
  Desktop과 CLI 사용 시기: 한 창에서 병렬 세션을 관리하거나, 창을 나란히 배치하거나, 변경 사항을 시각적으로 검토하려는 경우 Desktop을 사용하세요. 스크립팅, 자동화가 필요하거나 터미널 워크플로를 선호하는 경우 CLI를 사용하세요.
</Tip>

### CLI 플래그 대응표

이 표는 일반적인 CLI 플래그에 대한 데스크톱 앱 대응 항목을 보여줍니다. 나열되지 않은 플래그는 스크립팅 또는 자동화용으로 설계되어 데스크톱 대응 항목이 없습니다.

| CLI | Desktop 대응 항목 |
| ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--model sonnet` | 전송 버튼 옆의 모델 드롭다운 |
| `--resume`, `--continue` | 사이드바에서 세션 클릭 |
| `--permission-mode` | 전송 버튼 옆의 모드 선택기 |
| `--dangerously-skip-permissions` | Bypass permissions 모드. Pro 및 Max 플랜에서는 Settings → Claude Code → "Allow bypass permissions mode"에서 활성화하고, Team 및 Enterprise 플랜에서는 조직 정책으로 제어합니다. |
| `--add-dir` | 클라우드 세션에서 **+** 버튼으로 여러 리포지토리 추가 |
| `--allowedTools`, `--disallowedTools` | 세션별 대응 항목이 없습니다. [설정 파일](/docs/en/settings)의 권한 규칙은 여전히 적용됩니다. |
| `--verbose` | Transcript view 드롭다운의 [Verbose 뷰 모드](#switch-view-modes) |
| `--print`, `--output-format` | 지원 안 됨. Desktop은 대화형 전용입니다. |
| `ANTHROPIC_MODEL` 환경 변수 | 전송 버튼 옆의 모델 드롭다운 |
| `MAX_THINKING_TOKENS` 환경 변수 | 로컬 환경 편집기에서 설정. [환경 구성](#environment-configuration)을 참조하세요. |

### 공유 구성

Desktop과 CLI는 동일한 구성 파일을 읽으므로 설정이 그대로 인계됩니다:

* 프로젝트의 **[CLAUDE.md](/docs/en/memory)** 및 `CLAUDE.local.md` 파일은 둘 다 사용됩니다.
* `~/.claude.json` 또는 `.mcp.json`에 구성된 **[MCP 서버](/docs/en/mcp)**는 둘 다 작동합니다.
* 설정에 정의된 **[Hooks](/docs/en/hooks)** 및 **[skills](/docs/en/skills)**는 둘 다 적용됩니다.
* `~/.claude.json` 및 `~/.claude/settings.json`에 정의된 **[Settings](/docs/en/settings)**가 공유됩니다. `settings.json`에 있는 권한 규칙, 허용된 도구 및 기타 설정이 Desktop 세션에 적용됩니다.
* **Models**: 동일한 [모델](/docs/en/model-config#available-models)을 둘 다 사용할 수 있습니다. Desktop에서는 전송 버튼 옆의 드롭다운에서 모델을 선택하세요. 동일한 드롭다운에서 세션 도중에 모델을 변경할 수 있습니다.

<Note>
  **Claude Desktop 채팅 앱의 MCP 서버**: Desktop 앱은 `~/.claude.json` 및 `.mcp.json` 서버와 함께 `claude_desktop_config.json`에서 MCP 서버를 Code 탭 세션으로 로드합니다. `claude_desktop_config.json`에 정의된 서버는 Desktop 채팅 및 Code 탭 둘 다에서 사용할 수 있습니다.

  독립 실행형 CLI는 `claude_desktop_config.json`을 읽지 않습니다. macOS 및 WSL에서 `claude mcp add-from-claude-desktop`을 실행하여 해당 서버를 `~/.claude.json`으로 복사하세요. 가져오기 흐름 및 범위 옵션은 [Claude Desktop에서 MCP 서버 가져오기](/docs/en/mcp#import-mcp-servers-from-claude-desktop)를 참조하세요.
</Note>

### 기능 비교

이 표는 CLI와 Desktop 간의 핵심 기능을 비교합니다. CLI 플래그 전체 목록은 [CLI 참조](/docs/en/cli-reference)를 참조하세요.

| 기능 | CLI | Desktop |
| ------------------------------------- | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 권한 모드 | `dontAsk`를 포함한 모든 모드 | Manual, Accept edits, Plan 및 Auto. Bypass permissions는 Pro 및 Max 플랜의 Settings 토글, 또는 Team 및 Enterprise 플랜의 조직 정책을 통해 활성화된 후 모드 선택기에 나타납니다. |
| `--dangerously-skip-permissions` | CLI 플래그 | Bypass permissions 모드. Pro 및 Max 플랜의 경우 Settings → Claude Code → "Allow bypass permissions mode"에서 활성화하고, Team 및 Enterprise 플랜의 경우 조직 정책으로 제어합니다. |
| [제3자 제공업체](/docs/en/third-party-integrations) | Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry | 기본적으로 Anthropic의 API. 게이트웨이 라우팅은 [데스크톱 앱을 게이트웨이에 연결](/docs/en/llm-gateway-connect#desktop-app)을 참조하세요. Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry 또는 자체 호스팅 LLM 게이트웨이에서 Code 탭을 실행하려면 [3P에서의 Claude Desktop](https://claude.com/docs/third-party/claude-desktop/overview)을 참조하세요. |
| [MCP 서버](/docs/en/mcp) | 설정 파일에서 구성 | 로컬 및 SSH 세션의 경우 커넥터 UI 사용, 또는 설정 파일 사용 |
| [플러그인](/docs/en/plugins) | `/plugin` 명령 | 플러그인 관리자 UI |
| @mention 파일 | 텍스트 기반 | 자동 완성 지원; 로컬 및 SSH 세션 전용 |
| 파일 첨부 | 지원 안 됨 | 이미지, PDF |
| 세션 격리 | [`--worktree`](/docs/en/cli-reference) 플래그 | 자동 워크트리 |
| 다중 세션 | 별도의 터미널 | 사이드바 탭 |
| 정기 작업 | Cron 작업, CI 파이프라인 | [예약된 작업](/docs/en/desktop-scheduled-tasks) |
| 컴퓨터 사용 | macOS에서 [`/mcp`를 통해 활성화](/docs/en/computer-use) | macOS 및 Windows에서 [앱 및 화면 제어](#let-claude-use-your-computer) |
| iOS 시뮬레이터 | [컴퓨터 사용](/docs/en/computer-use#test-a-simulator-flow)을 통해 시뮬레이터 구동 | [iOS Simulator 창](/docs/en/desktop-ios-simulator)이 자동으로 열림 |
| Dispatch 연동 | 지원 안 됨 | 사이드바의 [Dispatch 세션](#sessions-from-dispatch) |
| 스크립팅 및 자동화 | [`--print`](/docs/en/cli-reference), [Agent SDK](/docs/en/headless) | 지원 안 됨 |

### Desktop에서 사용할 수 없는 기능

달리 언급된 경우를 제외하고 다음 기능은 CLI 또는 VS Code 확장 프로그램에서만 사용할 수 있습니다:

* **제3자 제공업체**: Desktop은 기본적으로 Anthropic API에 연결됩니다. 게이트웨이를 통해 Desktop을 라우팅하려면 [데스크톱 앱을 게이트웨이에 연결](/docs/en/llm-gateway-connect#desktop-app)을 참조하세요. 엔터프라이즈 배포는 [관리형 설정](https://claude.com/docs/third-party/claude-desktop/configuration)을 통해 Google Cloud's Agent Platform 및 게이트웨이 제공업체를 구성할 수 있습니다. CLI에서의 Amazon Bedrock 또는 Microsoft Foundry에 대해서는 [시작하기](/docs/en/quickstart)를 참조하세요. 위 섹션의 예외로서 [3P에서의 Claude Desktop](https://claude.com/docs/third-party/claude-desktop/overview)은 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry 또는 자체 호스팅 LLM 게이트웨이에서 Code 탭을 실행합니다.
* **Linux (beta)**: 컴퓨터 사용은 Linux 데스크톱 앱에서 아직 사용할 수 없습니다. [Linux에서의 Claude Desktop](/docs/en/desktop-linux)을 참조하세요.
* **인라인 코드 제안**: Desktop은 자동 완성 스타일 제안을 제공하지 않습니다. 대화형 프롬프트 및 명시적 코드 변경을 통해 작동합니다.
* **에이전트 팀**: 서로 메시지를 주고받는 병렬 Claude Code 세션은 Desktop이 아닌 [CLI](/docs/en/agent-teams)에서 사용할 수 있습니다. 단일 세션 내부의 멀티 에이전트 작업에는 Desktop에서 실행되는 [동적 워크플로](/docs/en/workflows)를 사용하세요.
* **터미널 대화 상자 명령**: 터미널에서 대화형 패널을 여는 내장 명령은 Code 탭에서 다르게 동작합니다. 권한 규칙 및 구성을 관리하려면 [설정 파일](/docs/en/settings)을 직접 편집하거나 독립 실행형 CLI에서 명령을 실행하세요.
  * `/permissions`와 같이 인수가 없는 형식의 명령은 `isn't available in this environment`라고 답글을 남깁니다.
  * `/config`는 Settings → Claude Code를 엽니다. 명령 뒤의 텍스트는 무시되므로 `/config theme=dark`는 테마를 설정하지 않습니다.

## 문제 해결

아래 섹션에서는 데스크톱 앱에 특화된 문제를 다룹니다. `API Error: 500`, `529 Overloaded`, `429`, 또는 `Prompt is too long`과 같이 채팅에 나타나는 런타임 API 오류는 [오류 참조](/docs/en/errors)를 참조하세요. 해당 오류와 수정 방법은 CLI, 데스크톱, 웹 전체에서 동일합니다.

### 버전 확인하기

실행 중인 데스크톱 앱 버전을 확인하려면:

* **macOS**: 메뉴 바에서 **Claude**를 클릭한 후 **About Claude** 선택
* **Windows**: **Help**를 클릭한 후 **About** 선택

버전 번호를 클릭하면 클립보드에 복사됩니다.

### Code 탭의 403 또는 인증 오류

Code 탭을 사용할 때 `Error 403: Forbidden` 또는 기타 인증 실패가 나타나는 경우:

1. 앱 메뉴에서 로그아웃한 후 다시 로그인하세요. 가장 일반적인 해결 방법입니다.
2. 유효한 유료 구독(Pro, Max, Team 또는 Enterprise)이 있는지 확인하세요.
3. CLI는 작동하지만 Desktop이 작동하지 않는 경우 창만 닫는 것이 아니라 데스크톱 앱을 완전히 종료한 다음 다시 열고 다시 로그인하세요.
4. 인터넷 연결 및 프록시 설정을 확인하세요.

### 실행 시 빈 화면 또는 멈춤 화면

앱은 열리지만 응답하지 않거나 빈 화면이 표시되는 경우:

1. 앱을 다시 시작하세요.
2. 보류 중인 업데이트를 확인하세요. macOS 및 Windows에서 앱은 실행 시 자동 업데이트됩니다; Linux에서는 [Linux에서의 Claude Desktop](/docs/en/desktop-linux)에 설명된 대로 apt를 통해 업데이트하세요.
3. 관리형 네트워크에서 방화벽이 [네트워크 접근 요건](#network-access-requirements)의 CDN 호스트를 허용하는지 확인하세요.
4. Windows의 경우 **Windows Logs → Application** 아래의 이벤트 뷰어에서 충돌 로그를 확인하세요.

### "Failed to load session"

`Failed to load session`이 표시되면 선택한 폴더가 더 이상 존재하지 않거나, Git 리포지토리에 설치되지 않은 Git LFS가 필요하거나, 파일 권한으로 인해 접근이 차단된 것일 수 있습니다. 다른 폴더를 선택하거나 앱을 다시 시작해 보세요.

### 세션에서 설치된 도구를 찾지 못함

Claude가 `npm`, `node` 또는 기타 CLI 명령과 같은 도구를 찾을 수 없는 경우 일반 터미널에서 해당 도구가 작동하는지 확인하고, 셸 프로필이 PATH를 올바르게 설정하는지 확인한 후 데스크톱 앱을 다시 시작하여 환경 변수를 다시 로드하세요.

### Git 및 Git LFS 오류

Windows에서 Code 탭이 로컬 세션을 시작하려면 Git이 필요합니다. "Git is required"가 표시되면 [Git for Windows](https://git-scm.com/downloads/win)를 설치하고 앱을 다시 시작하세요.

"Git LFS is required by this repository but is not installed"가 표시되면 [git-lfs.com](https://git-lfs.com/)에서 Git LFS를 설치하고 `git lfs install`을 실행한 다음 앱을 다시 시작하세요.

### Windows에서 MCP 서버가 작동하지 않음

Windows에서 MCP 서버 토글이 응답하지 않거나 서버 연결에 실패하는 경우 설정에서 서버가 올바르게 구성되어 있는지 확인하고, 앱을 다시 시작하고, 작업 관리자에서 서버 프로세스가 실행 중인지 확인하며, 연결 오류에 대해 서버 로그를 검토하세요.

### 앱이 종료되지 않음

* **macOS**: Cmd+Q를 누르세요. 앱이 응답하지 않는 경우 Cmd+Option+Esc로 강제 종료 창을 열고 Claude를 선택한 다음 Force Quit를 클릭하세요.
* **Windows**: Ctrl+Shift+Esc로 작업 관리자를 열고 Claude 프로세스를 종료하세요.

### Windows 특정 문제

* **설치 후 PATH가 업데이트되지 않음**: 새 터미널 창을 여세요. PATH 업데이트는 새 터미널 세션에만 적용됩니다.
* **동시 설치 오류**: 다른 설치가 진행 중이라는 오류가 표시되지만 진행 중인 설치가 없는 경우 Administrator(관리자 권한)로 설치 프로그램을 실행해 보세요.

### CLI에서 열 때 "Branch doesn't exist yet"

클라우드 세션은 로컬 머신에 존재하지 않는 브랜치를 생성할 수 있습니다. 세션 툴바의 브랜치 이름을 클릭하여 복사한 다음 로컬에서 가져오세요:

```bash theme={null}
git fetch origin <branch-name>
git checkout <branch-name>
```

### 여전히 해결되지 않나요?

* 데스크톱 앱에서 Help → Get Support를 열거나 [Claude 지원 센터](https://support.claude.com/)를 직접 방문하세요.
* 독립 실행형 `claude` CLI에서도 재현되는 문제의 경우 [GitHub Issues](https://github.com/anthropics/claude-code/issues)에서 검색하거나 버그를 제출하세요.

문제를 보고할 때는 데스크톱 앱 버전, 운영체제, 정확한 오류 메시지 및 관련 로그를 포함하세요. macOS에서는 Console.app을 확인하세요. Windows에서는 Event Viewer → Windows Logs → Application을 확인하세요. 공개 이슈에 게시하기 전에 로그 발췌 내용을 검토하세요; 로그에는 환경의 파일 경로 및 기타 세부 정보가 포함될 수 있습니다.
