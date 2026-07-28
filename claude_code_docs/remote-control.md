> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Remote Control을 통해 임의의 기기에서 로컬 세션 이어가기

> Remote Control을 사용하여 휴대폰, 태블릿, 또는 임의의 브라우저에서 로컬 Claude Code 세션을 이어가세요. claude.ai/code 및 Claude 모바일 앱과 연동됩니다.

<Note>
  Remote Control은 연구 프리뷰(research preview) 상태이며 모든 요금제에서 사용할 수 있습니다. Team 및 Enterprise 요금제에서는 소유자(Owner)가 [Claude Code 관리자 설정](https://claude.ai/admin-settings/claude-code)에서 Remote Control 토글을 활성화하기 전까지 기본적으로 비활성화되어 있습니다.
</Note>

Remote Control은 [claude.ai/code](https://claude.ai/code) 또는 [iOS](https://apps.apple.com/us/app/claude-by-anthropic/id6473753684) 및 [Android](https://play.google.com/store/apps/details?id=com.anthropic.claude)용 Claude 앱을 본인 머신에서 실행 중인 Claude Code 세션과 연결합니다. 책상 앞에서 태스크를 시작한 뒤, 소파에서 휴대폰으로 또는 다른 컴퓨터의 브라우저에서 태스크를 이어받으세요.

본인 머신에서 Remote Control 세션을 시작하면 세션 전체 기간 동안 Claude가 로컬에서 계속 실행되므로 코드 실행과 파일 시스템 접근이 본인 머신 내에 유지됩니다. Remote Control을 통해 다음 작업을 수행할 수 있습니다:

* **원격에서 전체 로컬 환경 사용**: 파일 시스템, [MCP 서버](/docs/en/mcp), 도구, 프로젝트 구성이 모두 사용 가능한 상태로 유지되며, `@`를 입력하면 로컬 프로젝트의 파일 경로가 자동 완성됨
* **두 서페이스에서 동시에 작업**: 대화 내용과 [subagents](/docs/en/sub-agents) 및 [동적 워크플로](/docs/en/workflows)의 진행 상황이 연결된 모든 기기 간에 동기화 상태를 유지하므로, 터미널, 브라우저, 휴대폰에서 교차하여 메시지를 전송 가능. v2.1.207 이전에는 [데스크톱 앱](/docs/en/desktop)이 호스팅하는 세션이 연결된 기기로 subagent나 워크플로 진행 상황을 보내지 않았었음.
* **휴대폰이나 브라우저에서 이미지 및 파일 전송**: Claude 앱이나 claude.ai/code에서 첨부 파일을 추가하면 Claude Code가 이를 본인 머신에 다운로드하여 캡션 포함/미포함 형태로 `@` 파일 참조로 Claude에게 전달함. v2.1.202 이전에는 캡션 없이 전송된 첨부 파일이 세션에 도달하기 전에 유실될 수 있었음.
* **중단 요소 극복**: 노트북이 절전 모드로 들어가거나 네트워크가 끊어지더라도 머신이 다시 온라인 상태가 되면 세션이 자동으로 재연결됨. Claude Code는 연결이 재구축되는 동안 subagent 및 워크플로의 상태 업데이트를 큐에 보관했다가 recovered 후 전달함. v2.1.207 이전에는 재연결이나 자격 증명 새로고침 중에 전송된 업데이트가 유실되어 연결된 기기가 이미 완료된 태스크를 계속 실행 중인 것으로 표시했었음.

클라우드 인프라 상에서 실행되는 [웹 상의 Claude Code](/docs/en/claude-code-on-the-web)와 달리, Remote Control 세션은 본인 머신에서 직접 구동되어 로컬 파일 시스템과 상호작용합니다. 웹 및 모바일 인터페이스는 해당 로컬 세션을 들여다보는 창 역할을 합니다.

이 페이지에서는 설정 방법, 세션 시작 및 연결 방법, Remote Control과 웹 상의 Claude Code 간 비교를 다룹니다.

## 요구사항

Remote Control을 사용하기 전에 사용 환경이 다음 조건을 충족하는지 확인하세요:

* **구독 요금제**: Pro, Max, Team, Enterprise 요금제에서 사용 가능. API 키는 지원되지 않음. Team 및 Enterprise 요금제에서는 소유자(Owner)가 [Claude Code 관리자 설정](https://claude.ai/admin-settings/claude-code)에서 Remote Control 토글을 먼저 활성화해야 함.
* **인증**: 아직 로그인하지 않았다면 `claude`를 실행하고 `/login`을 사용하여 claude.ai를 통해 로그인해야 함. 자격을 갖춘 로그인이 없으면 `claude remote-control`이 오류와 함께 종료되며, `claude --remote-control`은 실행 직후 Remote Control 실패 알림을 표시하며 대화형 세션을 시작함.
* **API 엔드포인트**: Amazon Bedrock, Google Cloud's Agent Platform, 또는 Microsoft Foundry에서는 사용할 수 없음. v2.1.196부터 [`ANTHROPIC_BASE_URL`](/docs/en/env-vars)이 [LLM 게이트웨이](/docs/en/llm-gateway)나 프록시 등 `api.anthropic.com` 외의 호스트를 가리킬 때도 Remote Control이 비활성화됨. Remote Control을 사용하려면 변수를 해제하세요.
* **워크스페이스 신뢰**: 프로젝트 디렉토리에서 `claude`를 최소 한 번 실행하여 워크스페이스 신뢰 대화 상자를 승인해야 함. 시작 시의 신뢰 대화 상자는 홈 디렉토리에 대한 신뢰를 절대 저장하지 않으므로 프로젝트 디렉토리에서 Remote Control을 시작하세요.

## Remote Control 세션 시작하기

CLI 또는 VS Code 확장 프로그램에서 Remote Control 세션을 시작할 수 있습니다. CLI는 세 가지 호출 모드를 제공하며, VS Code는 `/remote-control` 명령을 사용합니다.

<Tabs>
  <Tab title="서버 모드 (Server mode)">
    프로젝트 디렉토리로 이동하여 다음을 실행하세요:

    ```bash theme={null}
    claude remote-control
    ```

    프로세스는 원격 연결을 기다리며 서버 모드로 터미널에서 계속 구동됩니다. [다른 기기에서 연결](#connect-from-another-device)할 때 사용할 수 있는 세션 URL이 표시되며, 스페이스바를 눌러 휴대폰으로 빠르게 접근할 수 있는 QR 코드를 표시할 수 있습니다. 원격 세션이 활성 상태인 동안 터미널에 연결 상태와 도구 활동이 표시됩니다.

    사용 가능한 플래그:

    | 플래그                                          | 설명                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
    | ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
    | `--name "My Project"`                           | claude.ai/code의 세션 목록에 표시되는 커스텀 세션 제목 설정.                                                                                                                                                                                                                                                                                                                                                                                                                       |
    | `--remote-control-session-name-prefix <prefix>` | 명시적 이름이 설정되지 않았을 때 자동 생성되는 세션 이름의 접두사. 기본값은 머신의 호스트 이름이며 `myhost-graceful-unicorn`과 같은 이름을 생성함. 동일한 효과를 내려면 `CLAUDE_REMOTE_CONTROL_SESSION_NAME_PREFIX`를 설정.                                                                                                                                                                                                                                                    |
    | `-c`, `--continue`                              | 새 세션을 생성하는 대신 이 디렉토리에서 시작된 가장 최근의 Remote Control 세션을 재개함. `--session-id`, `--spawn`, `--capacity`, `--create-session-in-dir`과 조합할 수 없음. Claude Code v2.1.200 이상이 필요하며 이전 버전은 해당 플래그를 알 수 없는 인수로 거부함.                                                                                                                                                                                                           |
    | `--session-id <id>`                             | ID를 지정하여 특정 Remote Control 세션을 재개함. `--continue`, `--spawn`, `--capacity`, `--create-session-in-dir`과 조합할 수 없음. Claude Code v2.1.200 이상이 필요하며 이전 버전은 해당 플래그를 알 수 없는 인수로 거부함.                                                                                                                                                                                                                                                     |
    | `--spawn <mode>`                                | 서버가 세션을 생성하는 방식.<br />• `same-dir` (기본값): 모든 세션이 현재 작업 디렉토리를 공유하므로 동일한 파일을 편집할 때 충돌이 발생할 수 있음.<br />• `worktree`: 각 온디맨드 세션이 자체 [git worktree](/docs/en/worktrees)를 얻음. git 저장소가 필요함.<br />• `session`: 단일 세션 모드. 정확히 하나의 세션만 서빙하고 추가 연결을 거부함. 시작 시에만 설정 가능.<br />런타임 시 `w`를 눌러 `same-dir`과 `worktree` 간을 토글. |
    | `--capacity <N>`                                | 동시 세션의 최대 개수. 기본값은 32. `--spawn=session`과 함께 사용할 수 없음.                                                                                                                                                                                                                                                                                                                                                                                                       |
    | `--[no-]create-session-in-dir`                  | 서버가 시작될 때 즉시 입력할 수 있는 공간이 확보되도록 현재 디렉토리에 세션 하나를 사전 생성함. `worktree` 모드에서 이 세션은 현재 디렉토리에 유지되고 온디맨드 세션은 격리된 worktree를 얻음. 기본적으로 켜져 있음; 세션 없이 시작하려면 `--no-create-session-in-dir`을 전달.                                                                                                                                                                                                    |
    | `--verbose`                                     | 상세 연결 및 세션 로그 표시.                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
    | `--sandbox` / `--no-sandbox`                    | 파일 시스템 및 네트워크 격리를 위한 [샌드박싱](/docs/en/sandboxing) 활성화 또는 비활성화. 기본적으로 꺼져 있음.                                                                                                                                                                                                                                                                                                                                                                     |
  </Tab>

  <Tab title="대화형 세션 (Interactive session)">
    Remote Control이 활성화된 일반 대화형 Claude Code 세션을 시작하려면 `--remote-control` 플래그(또는 `--rc`)를 사용하세요:

    ```bash theme={null}
    claude --remote-control
    ```

    선택적으로 세션 이름을 전달할 수 있습니다:

    ```bash theme={null}
    claude --remote-control "My Project"
    ```

    이렇게 하면 터미널에서 전체 대화형 세션이 제공되며, claude.ai나 Claude 앱에서도 원격 제어할 수 있습니다. `claude remote-control`(서버 모드)과 달리, 세션이 원격에서 사용 가능한 동안 로컬에서도 메시지를 입력할 수 있습니다.
  </Tab>

  <Tab title="기존 세션에서 시작">
    이미 Claude Code 세션 내에 있으며 이를 원격으로 이어가려면 `/remote-control`(또는 `/rc`) 명령을 사용하세요:

    ```text theme={null}
    /remote-control
    ```

    커스텀 세션 제목을 설정하려면 인수로 이름을 전달하세요:

    ```text theme={null}
    /remote-control My Project
    ```

    이렇게 하면 현재 대화 히스토리를 전달받는 Remote Control 세션이 시작됩니다.

    `--verbose`, `--sandbox`, `--no-sandbox` 플래그는 이 명령에서 이용할 수 없습니다.
  </Tab>

  <Tab title="VS Code">
    [Claude Code VS Code 확장 프로그램](/docs/en/vs-code)의 프롬프트 상자에 `/remote-control` 또는 `/rc`를 입력하거나, `/`로 명령 메뉴를 열고 선택하세요.

    ```text theme={null}
    /remote-control
    ```

    프롬프트 상자 위에 연결 상태를 보여주는 배너가 나타납니다. 연결되면 배너의 **claude.ai/code**를 클릭하여 세션으로 바로 이동하거나 [claude.ai/code](https://claude.ai/code)의 세션 목록에서 찾으세요. Claude Code가 대화창에도 세션 URL을 게시합니다.

    연결을 해제하려면 배너의 닫기 아이콘을 클릭하거나 `/remote-control`을 다시 실행하세요.

    CLI와 달리 VS Code 명령은 이름 인수를 수용하지 않고 QR 코드를 표시하지도 않습니다. 세션 제목은 대화 히스토리나 첫 프롬프트로부터 파생됩니다.
  </Tab>
</Tabs>

### 연결 상태 확인

대화형 터미널 세션에서 연결이 유지되는 동안 입력 상자 아래 푸터에 `/rc active` 표시기가 위치하며, 터미널 폭이 너무 좁으면 숨겨집니다. 표시기 텍스트는 claude.ai 상의 세션 링크에 해당합니다. 아래쪽 화살표 키로 선택하고 Enter를 누르거나 `/remote-control`을 다시 실행하여 세션 URL과 [다른 기기에서 연결](#connect-from-another-device)할 때 사용할 수 있는 QR 코드가 포함된 상태 패널을 여세요.

연결이 실패하면 실패 원인이 담긴 알림이 표시되고 표시기가 푸터에서 사라집니다. 다시 시도하려면 `/remote-control`을 다시 실행하세요.

### 세션 URL 알림

Remote Control이 연결되어 있는 동안, 휴대폰이나 브라우저로 전환하는 것이 가장 도움이 되는 시점에 Claude Code가 세션 URL을 상기시켜 주므로 `/remote-control`에서 링크를 찾아 헤맬 필요가 없습니다. Claude Code v2.1.208 이상이 필요합니다. 다음 중 어느 시점에든 프롬프트 상자 위에 알림이 표시됩니다:

* **긴 턴 (Long turn)**: 턴이 서버에서 조정된 임계값보다 길게 구동될 때, **Check in from your phone** 링크가 포함된 **Still working** 알림이 표시되어 터미널에서 기다리는 대신 휴대폰이나 브라우저에서 턴을 팔로우할 수 있게 합니다. 턴이 끝나면 Claude Code가 알림을 제거합니다.
* **반복된 권한 프롬프트**: 세션에서 여러 [권한 프롬프트](/docs/en/permissions)에 응답한 후 **Approve tool calls from your phone** 알림이 세션 URL을 보여줍니다. 다음 턴이 시작되면 Claude Code가 이를 제거합니다.

이 알림들은 Remote Control이 [자동으로 연결되는 세션](#enable-remote-control-for-all-sessions)을 포함하여 연결된 모든 세션에 표시될 수 있습니다. 해당 조건이 발생할 때마다 매번 표시되는 것은 아니며 각 알림은 세션 전체에 걸쳐 총 몇 번만 나타납니다. 이를 설정하거나 끄는 것은 불가능하며 각각 알아서 지워집니다.

### 다른 기기에서 연결하기

Remote Control 세션이 활성화되면 다음 방법으로 다른 기기에서 연결할 수 있습니다:

* 브라우저에서 **세션 URL 열기**: [claude.ai/code](https://claude.ai/code) 상의 해당 세션으로 바로 이동합니다.
* 세션 URL 옆에 표시된 **QR 코드 스캔**: Claude 앱에서 바로 엽니다. `claude remote-control` 실행 시 스페이스바를 눌러 QR 코드 표시를 토글할 수 있습니다.
* **[claude.ai/code](https://claude.ai/code) 또는 Claude 앱 열기**: 세션 목록에서 이름으로 세션을 찾습니다. Claude 모바일 앱에서는 네비게이션에서 **Code**를 탭하여 세션 목록으로 이동합니다. Remote Control 세션은 온라인일 때 녹색 상태 점이 붙은 컴퓨터 아이콘으로 나타납니다.

연결 시 해당 기기는 세션이 백그라운드에서 이미 구동하고 있는 임의의 subagent 및 워크플로를 표시합니다. v2.1.208 이전에는 대화형 터미널에서 호스팅되는 세션에 연결된 기기가 이미 구동 중이던 subagent 및 워크플로를 이들 중 하나가 시작하거나 정지할 때까지 표시하지 않았었습니다. v2.1.212 이전에는 실행 도중에 참여한 기기가 다음 에이전트 업데이트 전까지 구동 중인 워크플로의 에이전트를 표시하지 않았었습니다.

원격 세션 제목은 다음 순서로 선택됩니다:

1. `--name`, `--remote-control`, 또는 `/remote-control`에 전달한 이름
2. `/rename`으로 설정한 제목
3. 기존 대화 히스토리의 마지막으로 의미 있는 메시지
4. `myhost-graceful-unicorn`과 같이 자동 생성된 이름 (`myhost`는 머신의 호스트 이름 또는 `--remote-control-session-name-prefix`로 설정한 접두사)

명시적 이름을 설정하지 않은 경우 프롬프트를 전송하면 해당 프롬프트를 반영하도록 제목이 업데이트됩니다. Claude Code v2.1.176부터 자동 생성된 제목이 대화 언어 또는 설정된 [`language`](/docs/en/settings#available-settings) 설정과 일치하게 됩니다. claude.ai나 Claude 앱에서 세션 이름을 변경하면 `claude --resume`에 표시되는 로컬 제목도 업데이트됩니다.

환경에 이미 활성 세션이 있는 경우 이를 재개할지 아니면 새 세션을 시작할지 묻는 프롬프트가 표시됩니다.

Claude 앱이 아직 없다면 Claude Code 내부에서 `/mobile` 명령을 사용하여 [iOS](https://apps.apple.com/us/app/claude-by-anthropic/id6473753684) 또는 [Android](https://play.google.com/store/apps/details?id=com.anthropic.claude)용 다운로드 QR 코드를 표시하세요.

### 모든 세션에 대해 Remote Control 활성화하기

자동 연결이 켜져 있지 않은 한, Remote Control은 `claude remote-control`, `claude --remote-control`, 또는 `/remote-control`을 명시적으로 실행할 때만 활성화됩니다. 모든 대화형 세션에 대해 이를 자동으로 활성화하려면 Claude Code 내부에서 `/config`를 실행하고 **Enable Remote Control for all sessions**를 `true`로 설정하세요. 절대 자동 연결하지 않으려면 `false`로 설정하고, 조직의 기본값을 따르려면 unset 상태로 두세요. 데스크톱 앱에서는 **Settings → Claude Code → Enable remote control by default**에서 이를 토글할 수 있습니다. [VS Code 확장 프로그램](/docs/en/vs-code#use-the-prompt-box)에서는 명령 메뉴의 Settings 섹션에 동일한 토글이 **Enable Remote Control for all sessions**로 표시됩니다 (Claude Code v2.1.203 이상 필요).

이 설정이 켜져 있으면 대화형 Claude Code 프로세스마다 하나의 원격 세션이 등록됩니다. 여러 인스턴스를 실행하면 각 인스턴스가 자체 환경과 세션을 얻게 됩니다. 단일 프로세스에서 여러 동시 세션을 구동하려면 [서버 모드](#start-a-remote-control-session)를 대신 사용하세요.

## 연결 및 보안

로컬 Claude Code 세션은 아웃바운드 HTTPS 요청만 수행하며 본인 머신에 인바운드 포트를 절대 열지 않습니다. Remote Control을 시작하면 Anthropic API에 등록되고 작업을 폴링합니다. 다른 기기에서 연결하면 서버가 스트리밍 연결을 통해 웹 또는 모바일 클라이언트와 로컬 세션 간에 메시지를 라우팅합니다.

모든 트래픽은 임의의 Claude Code 세션과 동일한 전송 보안 방식인 TLS를 거쳐 Anthropic API를 통해 이동합니다. 연결은 각각 단일 목적으로 범위가 지정되고 독립적으로 만료되는 수명이 짧은(short-lived) 여러 자격 증명을 사용합니다.

Remote Control이 연결되어 있는 동안 메시지, Claude의 응답, 도구 활동을 포함한 세션 트랜스크립트가 Anthropic 서버에 저장됩니다. 저장된 트랜스크립트는 기기 간 대화를 동기화 상태로 유지하고 네트워크가 끊어진 후 세션이 다시 연결될 수 있게 해줍니다. 실행 및 파일 시스템 접근은 본인 머신 내에 유지되며, 저장된 트랜스크립트는 [데이터 사용량](/docs/en/data-usage) 정책에 따라 보관됩니다.

Remote Control을 완전히 끄려면 [`disableRemoteControl`](/docs/en/settings#available-settings) 설정을 사용하세요. ZDR(Zero Data Retention)과 같은 컴플라이언스 요구사항이 있는 조직은 Remote Control을 활성화할 수 없습니다.

## 신뢰할 수 있는 기기 (Trusted Devices)

<Note>
  Trusted Devices는 현재 베타 상태입니다. 경험이 다듬어짐에 따라 기능이 진화할 수 있습니다.

  Trusted Devices는 Team 및 Enterprise 요금제에서 이용할 수 있습니다. 관리자가 활성화하기 전까지 기본적으로 꺼져 있습니다.
</Note>

Trusted Devices는 멤버가 claude.ai, Claude 모바일 앱, 또는 Claude Desktop에서 Remote Control 세션을 보거나 조종하기 전에 해당 기기를 검증하도록 요구하는 조직 차원의 설정입니다. 이는 단순 로그인된 계정이 아닌 알려진 기기 및 최근의 인증에 Remote Control 접근 권한을 바인딩합니다.

이 설정이 켜져 있을 때 Remote Control 세션과 상호작용하려면 다음 두 가지가 모두 필요합니다:

* **등록된 기기**: 멤버가 Remote Control에 사용하는 각 브라우저, 휴대폰, 데스크톱 앱이 자체 자격 증명을 등록함. 등록은 전체 로그인 직후에만 제공되므로 백그라운드에서 조용히 이루어지는 것이 아니라 실제 인증의 일부로 신뢰할 수 있는 목록에 참여함.
* **최근 로그인**: 멤버의 로그인이 18시간을 넘지 않아야 함. 매일 다시 로그인하는 대신 멤버가 Face ID, Touch ID, Windows Hello, 또는 패스키로 존재를 확인하면 세션이 즉시 새로고침됨.

생체 인식 검사는 패스키 로그인과 동일한 메커니즘으로 운영체제나 브라우저를 통해 기기 상에서 실행됩니다. Anthropic은 지문, 얼굴 데이터, 기타 임의의 생체 정보를 절대 수신하거나 저장하지 않습니다. 기기의 공개 키와 표시 이름, 플랫폼, 등록 시간과 같은 기본 메타데이터만 저장됩니다.

이 설정은 Remote Control에만 적용됩니다. 일반 Claude 채팅, 터미널에서의 Claude Code, API 사용은 영향받지 않습니다.

### 조직에 대해 Trusted Devices 활성화하기

관리자는 Claude Code 관리자 콘솔에서 이 설정을 활성화합니다.

<Steps>
  <Step title="Claude Code 관리자 설정 열기">
    [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)로 이동합니다. Remote Control 설정 아래에 **Require trusted devices** 토글이 표시됩니다.
  </Step>

  <Step title="Require trusted devices 켜기">
    이 설정은 조직의 모든 멤버와 이를 활성화한 후 시작되는 Remote Control 세션에 적용됩니다. 토글이 켜지기 전에 이미 구동 중이던 세션은 소급 적용되어 보호되지 않으며 끝날 때까지 기기 요구사항 없이 계속됩니다. 팀별 또는 프로젝트별 범위 지정은 제공되지 않습니다.
  </Step>

  <Step title="멤버들에게 예상 사항 전달하기">
    설정이 활성화된 후 멤버가 브라우저, 휴대폰, 데스크톱 앱에서 새 Remote Control 세션을 처음 보거나 조종하려 할 때 해당 기기를 등록하라는 프롬프트가 표시됩니다. 사전에 알려주면 혼란을 방지할 수 있습니다.
  </Step>
</Steps>

### 멤버들이 보게 되는 내용

등록은 기기당 한 번 수행하는 단계입니다. 그 후 보이는 유일한 변경 사항은 간혹 나타나는 생체 인식 프롬프트뿐입니다.

* **각 기기에서의 첫 사용**: 멤버에게 등록하라는 요청이 표시됨. 로그인이 최근 것이 아니면 구성된 SSO를 포함한 일반적인 흐름을 통해 먼저 로그인한 후 등록을 확인함.
* **일상적 사용**: 등록된 기기와 최근 로그인을 보유한 멤버에게는 아무런 프롬프트가 표시되지 않음. 로그인이 18시간을 경과하면 다음 Remote Control 상호작용 시 한 번의 Face ID, Touch ID, Windows Hello, 또는 패스키 프롬프트가 표시됨.
* **미등록 기기**: 기기가 등록될 때까지 Remote Control 세션을 보거나 조종할 수 없음. 해당 기기에서의 일반 Claude 채팅은 영향받지 않음.
* **플랫폼 인증기 없음**: Face ID, Touch ID, Windows Hello가 없는 머신의 멤버는 하드웨어 보안 키를 사용하거나 스텝업 대신 다시 로그인할 수 있음.
* **터미널 내부**: Claude Code를 실행하는 머신은 개발자가 CLI에 로그인할 때 자신의 자격 증명을 자동으로 받음. 터미널에는 별도의 등록 단계가 없음.
</Steps>

### 등록된 기기 관리하기

멤버는 계정 설정에서 자신의 기기를 검토하고 취소할 수 있습니다.

[claude.ai/settings/account](https://claude.ai/settings/account#trusted-devices)를 열고 **Trusted devices** 섹션에서 이름, 플랫폼, 등록 날짜와 함께 등록된 모든 기기를 확인하세요. 기기를 제거하면 자격 증명이 즉시 취소되며 해당 기기는 나중에 새 로그인 후 다시 등록할 수 있습니다. 자격 증명은 갱신되지 않으면 알아서 만료되므로 사용되지 않는 기기는 신뢰할 수 있는 목록에서 자동으로 떨어져 나갑니다.

분실하거나 도난당한 기기의 경우 멤버가 이 페이지에서 이를 제거합니다. 멤버가 로그인할 수 없는 경우 관리자가 관리자 콘솔의 **Sign out everywhere**를 사용하여 해당 멤버의 모든 세션과 등록된 기기를 취소할 수 있으며, 그 후 멤버가 아직 보유하고 있는 기기들을 다시 등록합니다.

## Remote Control vs 웹 상의 Claude Code

Remote Control과 [웹 상의 Claude Code](/docs/en/claude-code-on-the-web)는 둘 다 claude.ai/code 인터페이스를 사용합니다. 핵심 차이점은 세션이 실행되는 위치입니다: Remote Control은 본인 머신에서 실행되므로 로컬 MCP 서버, 도구, 프로젝트 구성을 계속 사용할 수 있습니다. 웹 상의 Claude Code는 Anthropic이 관리하는 클라우드 인프라에서 실행됩니다.

로컬 작업 도중에 다른 기기에서 계속 작업을 이어가고 싶을 때 Remote Control을 사용하세요. 아무런 로컬 설정 없이 태스크를 시작하고 싶거나, 클론하지 않은 저장소에서 작업하고 싶거나, 여러 태스크를 병렬로 실행하고 싶을 때 웹 상의 Claude Code를 사용하세요.

## 모바일 푸시 알림

Remote Control이 활성화되어 있을 때 Claude가 휴대폰으로 푸시 알림을 보낼 수 있습니다.

푸시 시점은 Claude가 결정합니다. 일반적으로 장시간 구동되는 태스크가 끝나거나 계속 진행하기 위해 사용자의 결정이 필요할 때 푸시를 보냅니다. `notify me when the tests finish`와 같이 프롬프트로 푸시를 요청할 수도 있습니다. 아래 두 토글 외에 이벤트별 개별 설정은 존재하지 않습니다.

모바일 푸시 알림을 설정하려면:

<Steps>
  <Step title="Claude 모바일 앱 설치">
    [iOS](https://apps.apple.com/us/app/claude-by-anthropic/id6473753684) 또는 [Android](https://play.google.com/store/apps/details?id=com.anthropic.claude)용 Claude 앱을 다운로드하세요.
  </Step>

  <Step title="Claude Code 계정으로 로그인">
    터미널의 Claude Code에 사용하는 것과 동일한 계정 및 조직을 사용하세요.
  </Step>

  <Step title="알림 허용">
    운영체제의 알림 권한 요청을 승인하세요.
  </Step>

  <Step title="Claude Code에서 푸시 활성화">
    터미널에서 `/config`를 실행하고 선제적 알림을 위해 **Push when Claude decides**를, 권한 프롬프트 및 질문을 위해 **Push when actions required**를, 또는 둘 다 활성화하세요.
  </Step>
</Steps>

알림이 도착하지 않는 경우:

* `/config`에 **No mobile registered**가 표시되면 휴대폰에서 Claude 앱을 열어 푸시 토큰을 새로고침하도록 하세요. 경고는 다음 번 Remote Control이 연결될 때 지워집니다.
* iOS에서는 집중 모드(Focus mode) 및 알림 요약이 푸시를 억제하거나 지연시킬 수 있습니다. 설정 → 알림 → Claude를 확인하세요.
* Android에서는 공격적인 배터리 최적화가 전달을 지연시킬 수 있습니다. 시스템 설정에서 Claude 앱을 배터리 최적화 대상에서 제외하세요.

연결된 터미널에 입력을 하고 있거나 터미널에 포커스가 맞춰져 있는 동안에는 Claude Code가 모바일 푸시 알림을 건너뜁니다. v2.1.181부터 [`CLAUDE_CLIENT_PRESENCE_FILE`](/docs/en/env-vars)을 마커 파일 경로로 설정하여 다른 창에 있더라도 머신에 자리하고 있는 임의의 시간으로 이를 확장할 수 있습니다: 해당 파일이 존재하는 동안에는 알림이 건너뛰어집니다. 화면이 잠금 해제될 때 파일을 생성하고 화면이 잠길 때 삭제하도록 화면 잠금 리스너나 유사 도구를 구성하세요.

## 제약 사항

* **대화형 프로세스당 하나의 원격 세션**: 서버 모드 외에서는 각 Claude Code 인스턴스가 한 번에 하나의 원격 세션만 지원합니다. 단일 프로세스에서 여러 동시 세션을 구동하려면 [서버 모드](#start-a-remote-control-session)를 사용하세요.
* **로컬 프로세스가 계속 구동되어야 함**: Remote Control은 로컬 프로세스로 구동됩니다. 터미널을 닫거나, VS Code를 종료하거나, `claude` 프로세스를 정지하면 세션이 종료됩니다.
* **장기 네트워크 장애**: 머신이 깨어있지만 약 10분 이상 네트워크에 도달할 수 없는 경우 세션이 타임아웃되고 프로세스가 종료됩니다. 새 세션을 시작하려면 `claude remote-control`을 다시 실행하세요.
* **Ultraplan 사용 시 Remote Control 연결 해제**: [ultraplan](/docs/en/ultraplan) 세션을 시작하면 두 기능 모두 claude.ai/code 인터페이스를 점유하고 한 번에 하나만 연결될 수 있으므로 활성화된 Remote Control 세션이 연결 해제됩니다.
* **일부 명령은 로컬 전용임**: `/plugin`이나 `/resume`처럼 터미널 인터페이스에서만 실행되는 명령은 인수를 전달하든 안 하든 로컬 CLI에서만 작동합니다. 모바일 및 웹에서 작동하는 명령은 다음과 같습니다:
  * 텍스트 출력 명령: `/compact`, `/clear`, `/context`, `/usage`, `/exit`, `/usage-credits` (브라우저를 여는 대신 청구 URL을 출력함), `/recap`, `/reload-plugins`
  * `/model`, `/effort`, `/fast`, `/color`, `/rename`: `/model sonnet`이나 `/effort high`와 같이 인수로 값을 전달함. 모바일 및 웹에서 `/model`과 `/effort`는 터미널 피커나 슬라이더 대신 인수를 받아 처리함.
  * `/mcp` (v2.1.166 이상): 모바일 앱에서 실행 시 피커를 여는 대신 서버 상태의 텍스트 요약을 반환함. 웹에서 단독으로 `/mcp`를 실행하면 요약을 반환하는 대신 [claude.ai 커넥터](/docs/en/mcp#use-mcp-servers-from-claude-ai) 디렉토리를 얾. `reconnect`, `enable`, `disable` [하위 명령](/docs/en/commands#all-commands)은 모바일/웹 모두에서 작함. 로컬 CLI와 달리 서버 이름 없이 `/mcp reconnect`를 실행하면 실패했거나 인증이 필요한 모든 서버를 다시 연결함.
  * `/config` (v2.1.181 이상): 모바일 앱에서 `key=value`를 전달하여 설정을 변경하거나 인수 없이 실행하여 설정 가능한 키 목록을 출력함. 웹에서 `/config`는 설정의 Claude Code 섹션을 열며 명령 뒤의 텍스트를 무시함.
  * Team 및 Enterprise에서 모바일이나 웹의 `/usage-credits`는 [관리자에게 usage-credits 요청을 보내지 않음](/docs/en/costs#add-usage-credits-to-your-subscription). 전송에는 대화형 CLI에만 표시되는 승인이 필요하므로 명령이 거기서 실행하라고 알려줌. v2.1.211 이전에는 텍스트 형태가 승인 없이 요청을 보냈었음.

## 문제 해결

### "Remote Control requires a claude.ai subscription"

claude.ai 계정으로 인증되지 않았습니다. `claude auth login`을 실행하고 claude.ai 옵션을 선택하세요. 환경 변수에 `ANTHROPIC_API_KEY`가 설정되어 있다면 먼저 해제하세요.

v2.1.206 이전에는 로그아웃 상태에서 `/remote-control`을 실행하면 이 메시지 대신 `Unknown command: /remote-control`을 보고했었습니다.

### "Remote Control requires a full-scope login token"

`claude setup-token`이나 `CLAUDE_CODE_OAUTH_TOKEN` 환경 변수의 장기 유지 토큰(long-lived token)으로 인증되어 있습니다. 이러한 토큰은 모델 요청만 수행할 수 있으므로 Remote Control 세션을 확립할 수 없습니다. `claude auth login`을 실행하여 풀 스코프 세션 토큰으로 인증하세요.

### "Unable to determine your organization for Remote Control eligibility"

캐시된 계정 정보가 오래되었거나 불완전합니다. `claude auth login`을 실행하여 새로고침하세요.

### "Remote Control is not yet enabled for your account"

Remote Control 배포가 계정에 도달하지 않았거나 캐시된 권한이 오래되었습니다. 최근 요금제를 변경했다면 `claude auth logout` 후 `claude auth login`을 실행하여 새로고침하세요. 개별 자격 검사 중 어느 것이 실패했는지 보려면 `claude doctor`를 실행하세요. 환경 변수 충돌, 도달 불가능한 검사, 조직 정책은 각자의 메시지를 생성하므로 이 오류는 배포 게이트 자체를 의미합니다.

### "Couldn't verify Remote Control eligibility"

Claude Code가 계정에 대해 Remote Control이 활성화되어 있는지 확인하기 위해 피처 플래그 서비스에 도달하지 못했습니다. 일반적으로 오프라인 상태이거나 프록시가 요청을 차단하고 있기 때문입니다. 네트워크에 연결된 후 다시 시도하거나 세부 정보를 보려면 `claude doctor`를 구동하세요. 관련 메시지인 "Couldn't verify your organization's Remote Control policy"도 동일한 원인과 해결책을 가집니다. 두 메시지 모두 v2.1.178에 추가되었습니다.

### "Remote Control is only available when using Claude via api.anthropic.com"

세션이 Anthropic API와 직접 통신하지 않고 있으므로 페어링할 claude.ai 백엔드가 존재하지 않습니다. Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry에서 이것이 발생합니다. v2.1.196부터 claude.ai로 로그인했더라도 [`ANTHROPIC_BASE_URL`](/docs/en/env-vars)이 [LLM 게이트웨이](/docs/en/llm-gateway)나 프록시 등 `api.anthropic.com` 외의 호스트를 가리킬 때도 이것이 발생합니다. `ANTHROPIC_BASE_URL`을 해제하고 세션을 재시작하여 Remote Control을 사용하세요.

### "Remote Control is disabled by your organization's policy"

이 오류에는 네 가지 명확한 원인이 있습니다. 사용 중인 로그인 방식과 구독을 확인하려면 먼저 `/status`를 실행하세요.

* **API 키 또는 Console 계정으로 인증함**: Remote Control은 claude.ai OAuth를 요구합니다. `/login`을 실행하고 claude.ai 옵션을 선택하세요. 환경 변수에 `ANTHROPIC_API_KEY`가 설정되어 있다면 해제하세요.
* **소유자(Owner)가 조직에 대해 활성화하지 않음**: Remote Control은 Team 및 Enterprise 요금제에서 기본적으로 꺼져 있습니다. 소유자는 [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)에서 **Remote Control** 토글을 켜서 이를 활성화할 수 있습니다. 이 토글은 서버 측 조직 설정입니다.
* **관리자 토글이 회색으로 비활성화되어 있음**: 조직에 Remote Control과 호환되지 않는 데이터 보관 또는 컴플라이언스 구성이 적용되어 있습니다. 이는 관리자 패널에서 변경할 수 없습니다. 옵션을 논의하려면 Anthropic 지원팀에 문의하세요.
* **오류 메시지에 `disableRemoteControl`이 언급됨**: IT 관리자가 조직 전체 토글과 별개로 [관리형 설정](/docs/en/settings#settings-files)을 통해 이 기기에서 Remote Control을 비활성화했습니다.

### "Remote credentials fetch failed"

Claude Code가 연결을 확립하기 위한 단기 자격 증명을 Anthropic API로부터 가져오지 못했습니다. 전체 오류를 보려면 `--verbose`와 함께 다시 실행하세요:

```bash theme={null}
claude remote-control --verbose
```

흔한 원인들:

* 로그인되어 있지 않음: `claude`를 실행하고 `/login`을 사용하여 claude.ai 계정으로 인증하세요. API 키 인증은 Remote Control에서 지원되지 않습니다.
* 네트워크 또는 프록시 문제: 방화벽이나 프록시가 아웃바운드 HTTPS 요청을 차단하고 있을 수 있습니다. Remote Control은 443 포트를 통한 Anthropic API 접근이 필요합니다.
* 세션 생성 실패: `SessionCreation failed — see debug log`도 함께 보이는 경우 설정 초기에 실패가 일어난 것입니다. 구독이 활성 상태인지 확인하세요.

### "Couldn't reconnect to your Remote Control session"

`claude --resume`이나 `claude --continue`로 대화를 재개할 때 Claude Code는 해당 대화에 기록된 Remote Control 세션으로 다시 연결합니다. 이 메시지는 네트워크 중단이나 서버 오류 등 일시적일 수 있는 원인으로 재연결이 실패하여 로컬 세션이 원격 세션의 존재 여부를 확인할 수 없음을 의미합니다. 이전 세션이 더 이상 존재하지 않음을 서버가 확인하면 Claude Code가 이 메시지를 표시하지 않고 새 Remote Control 세션을 생성합니다.

로컬 세션은 Remote Control 없이 계속 구동됩니다. 연결을 다시 시도하려면 `/remote-control`을 실행하고, 새 Remote Control 세션을 생성하려면 `--resume` 없이 Claude Code를 시작하세요.

v2.1.200 이전에는 재연결 실패 시 이 메시지를 표시하는 대신 새 Remote Control 세션을 생성하여 claude.ai/code의 세션 목록에 여분의 세션이 남았었습니다.

### "Your organization requires Trusted Devices for Remote Control, but this device is not enrolled"

조직에 [Trusted Devices](#trusted-devices)가 활성화되어 있으나 이 머신이 아직 등록되지 않았습니다. Claude Code 내부에서 `/login`을 실행하세요. 등록은 로그인 과정의 일부로 진행되며 별도의 등록 명령은 존재하지 않습니다.

### "session expired for trusted-device check"

로그인이 18시간을 경과했습니다. Claude Code에서 `/login`을 실행하거나 claude.ai 또는 모바일 앱이 프롬프트를 띄울 때 Face ID, Touch ID, Windows Hello, 또는 패스키로 확인하세요. [Trusted Devices](#trusted-devices)를 참조하세요.

## 적절한 접근 방식 선택하기

Claude Code는 터미널 앞에 있지 않을 때 작업할 수 있는 여러 방식을 제공합니다. 이들은 작업이 트리거되는 방식, Claude가 구동되는 위치, 설정이 얼마나 필요한가에서 차이가 납니다.

|                                                | 트리거 방식                                                                                    | Claude 구동 위치                                                                             | 설정                                                                                                                                 | 적합한 대상                                                   |
| :--------------------------------------------- | :--------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------ |
| [Dispatch](/docs/en/desktop#sessions-from-dispatch) | Claude 모바일 앱에서 태스크 메시지 전송                                                        | 본인 머신 (Desktop)                                                                          | [모바일 앱과 Desktop 페어링](https://support.claude.com/en/articles/13947068)                                                        | 자리를 비운 동안 최소한의 설정으로 작업 위임                  |
| [Remote Control](/docs/en/remote-control)           | [claude.ai/code](https://claude.ai/code) 또는 Claude 모바일 앱에서 구동 중인 세션 조종        | 본인 머신 (CLI 또는 VS Code)                                                                 | `claude remote-control` 실행                                                                                                         | 다른 기기에서 진행 중인 작업 조종                             |
| [Channels](/docs/en/channels)                       | Telegram, Discord 등 채팅 앱이나 자체 서버에서 이벤트 푸시                                     | 본인 머신 (CLI)                                                                              | [채널 플러그인 설치](/docs/en/channels#quickstart) 또는 [자체 채널 구축](/docs/en/channels-reference)                                | CI 실패나 채팅 메시지 등 외부 이벤트에 반응                   |
| [Slack](/docs/en/slack)                             | 팀 채널에서 `@Claude` 멘션                                                                     | Anthropic 클라우드                                                                           | [웹 상의 Claude Code](/docs/en/claude-code-on-the-web)가 활성화된 상태에서 [Slack 앱 설치](/docs/en/slack#setting-up-claude-code-in-slack) | 팀 채팅에서의 PR 및 리뷰                                      |
| [Scheduled tasks](/docs/en/scheduled-tasks)         | 일정(Schedule) 설정                                                                            | [CLI](/docs/en/scheduled-tasks), [Desktop](/docs/en/desktop-scheduled-tasks), [클라우드](/docs/en/routines) | 주기 선택                                                                                                                            | 일일 리뷰 등 반복적인 자동화                                  |

## 관련 리소스

* [웹 상의 Claude Code](/docs/en/claude-code-on-the-web): 본인 머신 대신 Anthropic이 관리하는 클라우드 환경에서 세션 구동
* [Ultraplan](/docs/en/ultraplan): 터미널에서 클라우드 플래닝 세션을 시작하고 브라우저에서 계획 검토
* [Channels](/docs/en/channels): 자리를 비운 동안 Claude가 메시지에 반응하도록 Telegram, Discord, iMessage를 세션으로 전달
* [Dispatch](/docs/en/desktop#sessions-from-dispatch): 휴대폰에서 태스크 메시지를 보내 이를 처리할 데스크톱 세션 스패닝
* [인증](/docs/en/authentication): claude.ai를 위한 `/login` 설정 및 자격 증명 관리
* [CLI 참조 문서](/docs/en/cli-reference): `claude remote-control`을 포함한 전체 플래그 및 명령 목록
* [보안](/docs/en/security): Remote Control 세션이 Claude Code 보안 모델에 들어맞는 방식
* [데이터 사용량](/docs/en/data-usage): 로컬 및 원격 세션 동안 Anthropic API를 통해 흐르는 데이터
