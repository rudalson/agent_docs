> ## 문서 색인
> 다음 위치에서 전체 문서 색인을 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 채널(channels)을 사용하여 실행 중인 세션에 이벤트 푸시하기

> 채널을 사용하여 MCP 서버에서 실행 중인 Claude Code 세션으로 메시지, 알림 및 웹훅을 푸시하세요. CI 결과, 채팅 메시지 및 모니터링 이벤트를 전달하여 자리를 비운 동안에도 Claude가 반응할 수 있도록 합니다.

<Note>
  채널은 [리서치 프리뷰](#리서치-프리뷰) 단계에 있습니다. claude.ai를 통한 Anthropic 인증 또는 Console API 키가 필요하며, Amazon Bedrock, Google Cloud Agent Platform 또는 Microsoft Foundry에서는 사용할 수 없습니다. Team 및 Enterprise 조직은 [명시적으로 활성화](#조직-제어)해야 합니다.
</Note>

채널은 실행 중인 Claude Code 세션에 이벤트를 푸시하는 MCP 서버로, 터미널 앞에 없어도 Claude가 발생하는 상황에 반응할 수 있게 해줍니다. 채널은 양방향일 수 있습니다. Claude는 이벤트를 읽고 동일한 채널을 통해 답변을 다시 보냅니다(채팅 브리지와 같은 역할). 이벤트는 세션이 열려 있는 동안에만 도착하므로 상시 가동(always-on) 환경을 원한다면 백그라운드 프로세스나 영구 터미널에서 Claude를 실행하세요.

새로운 클라우드 세션을 생성하거나 폴링을 기다리는 통합 방식과 달리, 이벤트는 이미 열려 있는 세션에 도착합니다. 자세한 내용은 [채널 비교](#채널-비교)를 참조하세요.

채널은 플러그인으로 설치하며 사용자 고유의 자격 증명으로 설정합니다. Telegram, Discord 및 iMessage가 리서치 프리뷰에 포함되어 있습니다.

Claude가 채널을 통해 답장할 때, 터미널에서는 수신 메시지가 표시되지만 답장 텍스트는 표시되지 않습니다. 터미널에는 도구 호출과 확인 메시지("sent" 등)만 표시되고, 실제 답장은 상대방 플랫폼에 나타납니다.

Team, Enterprise 또는 Console 조직을 관리하는 경우 [조직에 대한 채널 활성화](#조직-제어)를 참조하세요. 나만의 채널을 구축하려면 [채널 참조](/docs/en/channels-reference)를 참조하세요.

## 지원되는 채널

지원되는 각 채널은 [Bun](https://bun.sh)이 필요한 플러그인입니다. 실제 플랫폼을 연결하기 전에 플러그인 흐름을 직접 체험해 보려면 [fakechat 퀵스타트](#퀵스타트)를 시도해 보세요.

<Tabs>
  <Tab title="Telegram">
    전체 [Telegram 플러그인 소스](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram)를 확인하세요.

    <Steps>
      <Step title="Telegram 봇 생성">
        Telegram에서 [BotFather](https://t.me/BotFather)를 열고 `/newbot`을 전송합니다. 표시 이름과 `bot`으로 끝나는 고유한 사용자 이름을 지정합니다. BotFather가 반환한 토큰을 복사합니다.
      </Step>

      <Step title="플러그인 설치">
        Claude Code에서 다음을 실행합니다:

        ```
        /plugin install telegram@claude-plugins-official
        ```

        Claude Code에서 `Marketplace "claude-plugins-official" not found`라는 메시지가 나타나면 `/plugin marketplace add anthropics/claude-plugins-official` 명령으로 마켓플레이스를 추가하세요. 마켓플레이스에서 플러그인을 찾을 수 없다고 나오면 로컬 복사본이 오래된 것이므로 `/plugin marketplace update claude-plugins-official`로 새로 고친 후 설치를 다시 시도하세요.

        설치 시 설치 범위(installation scope)를 물어보면 user 범위 옵션을 선택하여 모든 프로젝트에서 플러그인을 사용할 수 있도록 합니다. 설치 후 `/reload-plugins`를 실행하여 플러그인의 configure 명령을 활성화하세요.
      </Step>

      <Step title="토큰 설정">
        BotFather에서 받은 토큰으로 configure 명령을 실행합니다:

        ```
        /telegram:configure <token>
        ```

        이렇게 하면 토큰이 `~/.claude/channels/telegram/.env`에 저장됩니다. Claude Code를 실행하기 전에 셸 환경에서 `TELEGRAM_BOT_TOKEN`을 설정할 수도 있습니다.
      </Step>

      <Step title="채널을 활성화하여 재시작">
        Claude Code를 종료하고 채널 플래그를 지정하여 재시작합니다. 이렇게 하면 Telegram 플러그인이 시작되어 봇의 메시지를 폴링하기 시작합니다:

        ```bash theme={null}
        claude --channels plugin:telegram@claude-plugins-official
        ```
      </Step>

      <Step title="계정 페어링">
        Telegram을 열고 봇에게 아무 메시지나 보냅니다. 봇이 페어링 코드로 답장합니다.

        <Note>봇이 응답하지 않으면 이전 단계에서 Claude Code가 `--channels`로 실행 중인지 확인하세요. 봇은 채널이 활성화되어 있는 동안에만 답장할 수 있습니다.</Note>

        Claude Code로 돌아와 다음을 실행합니다:

        ```
        /telegram:access pair <code>
        ```

        그런 다음 자신의 계정만 메시지를 보낼 수 있도록 액세스를 제한합니다:

        ```
        /telegram:access policy allowlist
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="Discord">
    전체 [Discord 플러그인 소스](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/discord)를 확인하세요.

    <Steps>
      <Step title="Discord 봇 생성">
        [Discord Developer Portal](https://discord.com/developers/applications)로 이동하여 **New Application**을 클릭하고 이름을 지정합니다. **Bot** 섹션에서 사용자 이름을 생성한 후 **Reset Token**을 클릭하고 토큰을 복사합니다.
      </Step>

      <Step title="Message Content Intent 활성화">
        봇 설정에서 **Privileged Gateway Intents**로 스크롤하여 **Message Content Intent**를 활성화합니다.
      </Step>

      <Step title="서버에 봇 초대">
        **OAuth2 > URL Generator**로 이동합니다. `bot` 스코프를 선택하고 다음 권한을 활성화합니다:

        * View Channels
        * Send Messages
        * Send Messages in Threads
        * Read Message History
        * Attach Files
        * Add Reactions

        생성된 URL을 열어 서버에 봇을 추가합니다.
      </Step>

      <Step title="플러그인 설치">
        Claude Code에서 다음을 실행합니다:

        ```
        /plugin install discord@claude-plugins-official
        ```

        Claude Code에서 `Marketplace "claude-plugins-official" not found` 메시지가 나타나면 `/plugin marketplace add anthropics/claude-plugins-official` 명령으로 마켓플레이스를 추가하세요. 마켓플레이스에서 플러그인을 찾을 수 없다고 나오면 로컬 복사본이 오래된 것이므로 `/plugin marketplace update claude-plugins-official`로 새로 고친 후 설치를 다시 시도하세요.

        설치 시 설치 범위(installation scope)를 물어보면 user 범위 옵션을 선택하여 모든 프로젝트에서 플러그인을 사용할 수 있도록 합니다. 설치 후 `/reload-plugins`를 실행하여 플러그인의 configure 명령을 활성화하세요.
      </Step>

      <Step title="토큰 설정">
        복사한 봇 토큰으로 configure 명령을 실행합니다:

        ```
        /discord:configure <token>
        ```

        이렇게 하면 토큰이 `~/.claude/channels/discord/.env`에 저장됩니다. Claude Code를 실행하기 전에 셸 환경에서 `DISCORD_BOT_TOKEN`을 설정할 수도 있습니다.
      </Step>

      <Step title="채널을 활성화하여 재시작">
        Claude Code를 종료하고 채널 플래그를 지정하여 재시작합니다. 이렇게 하면 Discord 플러그인이 연결되어 봇이 메시지를 수신하고 응답할 수 있게 됩니다:

        ```bash theme={null}
        claude --channels plugin:discord@claude-plugins-official
        ```
      </Step>

      <Step title="계정 페어링">
        Discord에서 봇에게 DM을 보냅니다. 봇이 페어링 코드로 답장합니다.

        <Note>봇이 응답하지 않으면 이전 단계에서 Claude Code가 `--channels`로 실행 중인지 확인하세요. 봇은 채널이 활성화되어 있는 동안에만 답장할 수 있습니다.</Note>

        Claude Code로 돌아와 다음을 실행합니다:

        ```
        /discord:access pair <code>
        ```

        그런 다음 자신의 계정만 메시지를 보낼 수 있도록 액세스를 제한합니다:

        ```
        /discord:access policy allowlist
        ```
      </Step>
    </Steps>
  </Tab>

  <Tab title="iMessage">
    전체 [iMessage 플러그인 소스](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/imessage)를 확인하세요.

    iMessage 채널은 메시지 데이터베이스를 직접 읽고 AppleScript를 통해 답장을 보냅니다. macOS가 필요하며 봇 토큰이나 외부 서비스가 필요하지 않습니다.

    <Steps>
      <Step title="전체 디스크 접근 권한 부여">
        `~/Library/Messages/chat.db`에 있는 메시지 데이터베이스는 macOS에 의해 보호됩니다. 서버가 이를 처음 읽을 때 macOS에서 접근 프롬프트가 표시됩니다. **허용(Allow)**을 클릭하세요. 프롬프트에는 Terminal, iTerm 또는 IDE 등 Bun을 실행한 앱의 이름이 표시됩니다.

        프롬프트가 나타나지 않거나 '허용 안 함'을 클릭한 경우, **시스템 설정 > 개인정보 보호 및 보안 > 전체 디스크 접근 권한(System Settings > Privacy & Security > Full Disk Access)**에서 수동으로 접근 권한을 부여하고 터미널을 추가하세요. 권한이 없으면 서버가 `authorization denied` 오류와 함께 즉시 종료됩니다.
      </Step>

      <Step title="플러그인 설치">
        Claude Code에서 다음을 실행합니다:

        ```
        /plugin install imessage@claude-plugins-official
        ```

        Claude Code에서 `Marketplace "claude-plugins-official" not found` 메시지가 나타나면 `/plugin marketplace add anthropics/claude-plugins-official` 명령으로 마켓플레이스를 추가하세요. 마켓플레이스에서 플러그인을 찾을 수 없다고 나오면 로컬 복사본이 오래된 것이므로 `/plugin marketplace update claude-plugins-official`로 새로 고친 후 설치를 다시 시도하세요.

        설치 시 설치 범위(installation scope)를 물어보면 user 범위 옵션을 선택하여 모든 프로젝트에서 플러그인을 사용할 수 있도록 합니다. Claude Code에서 `/reload-plugins` 실행을 제안하지만, 다음 단계에서 재시작할 때 플러그인을 불러오므로 여기서는 건너뛰어도 됩니다.
      </Step>

      <Step title="채널을 활성화하여 재시작">
        Claude Code를 종료하고 채널 플래그를 지정하여 재시작합니다:

        ```bash theme={null}
        claude --channels plugin:imessage@claude-plugins-official
        ```
      </Step>

      <Step title="자기 자신에게 문자 전송">
        Apple ID로 로그인된 모든 기기에서 메시지 앱을 열고 자기 자신에게 메시지를 보냅니다. Claude에게 즉시 전달됩니다. 자기 자신과의 채팅은 별도 설정 없이 접근 제어를 우회합니다.

        <Note>Claude가 보내는 첫 번째 답장은 터미널이 메시지 앱을 제어할 수 있는지 묻는 macOS 자동화 프롬프트를 트리거합니다. **확인(OK)**을 클릭하세요.</Note>
      </Step>

      <Step title="다른 발신자 허용">
        기본적으로 사용자 자신의 메시지만 전달됩니다. 다른 연락처가 Claude에게 도달할 수 있도록 하려면 해당 핸들을 추가하세요:

        ```
        /imessage:access allow +15551234567
        ```

        핸들은 `+국가번호` 형식의 전화번호이거나 `user@example.com`과 같은 Apple ID 이메일입니다.
      </Step>
    </Steps>
  </Tab>
</Tabs>

아직 플러그인이 없는 시스템을 위한 [나만의 채널 구축](/docs/en/channels-reference)도 가능합니다.

## 퀵스타트

Fakechat은 인증할 필요도 없고 외부 서비스를 설정할 필요도 없이 localhost에서 채팅 UI를 실행하는 공식 지원 데모 채널입니다.

fakechat을 설치하고 활성화하면 브라우저에 입력한 메시지가 Claude Code 세션에 전달됩니다. Claude가 답장하면 해당 답장이 브라우저에 다시 표시됩니다. fakechat 인터페이스를 테스트해 본 후 [Telegram](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram), [Discord](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/discord) 또는 [iMessage](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/imessage)를 시도해 보세요.

fakechat 데모를 사용하려면 다음이 필요합니다:

* claude.ai 계정 또는 Claude Console API 키로 [설치 및 인증](/docs/en/quickstart#step-1-install-claude-code)된 Claude Code
* [Bun](https://bun.sh) 설치됨. 사전 구축된 채널 플러그인은 Bun 스크립트입니다. `bun --version`으로 확인하세요. 실패하면 [Bun을 설치](https://bun.sh/docs/installation)하세요.
* **Team, Enterprise 또는 관리형 Console 조직**: 관리자가 관리형 설정에서 [채널을 활성화](#조직-제어)해야 합니다.

<Steps>
  <Step title="fakechat 채널 플러그인 설치">
    Claude Code 세션을 시작하고 설치 명령을 실행합니다:

    ```text theme={null}
    /plugin install fakechat@claude-plugins-official
    ```

    Claude Code에서 `Marketplace "claude-plugins-official" not found` 메시지가 나타나면 `/plugin marketplace add anthropics/claude-plugins-official` 명령으로 마켓플레이스를 추가하세요. 마켓플레이스에서 플러그인을 찾을 수 없다고 나오면 로컬 복사본이 오래된 것이므로 `/plugin marketplace update claude-plugins-official`로 새로 고친 후 설치를 다시 시도하세요.

    설치 시 설치 범위를 물어보면 user 범위 옵션을 선택하여 모든 프로젝트에서 플러그인을 사용할 수 있도록 합니다. Claude Code에서 `/reload-plugins` 실행을 제안하지만, 다음 단계에서 재시작할 때 플러그인을 불러오므로 여기서는 건너뛰어도 됩니다.
  </Step>

  <Step title="채널을 활성화하여 재시작">
    Claude Code를 종료한 다음, `--channels`와 설치한 fakechat 플러그인을 전달하여 재시작합니다:

    ```bash theme={null}
    claude --channels plugin:fakechat@claude-plugins-official
    ```

    fakechat 서버가 자동으로 시작됩니다. 시작 화면에는 `plugin:fakechat@claude-plugins-official`의 메시지가 이 세션에 직접 주입된다는 채널 알림이 표시됩니다. 플러그인이 설치되지 않았거나 승인된 허용 목록에 없으면 해당 알림 아래에 문제를 알리는 경고 줄이 나타납니다.

    <Tip>
      `--channels`에 공백으로 구분하여 여러 플러그인을 전달할 수 있습니다.
    </Tip>
  </Step>

  <Step title="메시지 푸시">
    [http://localhost:8787](http://localhost:8787)에서 fakechat UI를 열고 메시지를 입력합니다:

    ```text theme={null}
    what's in my working directory?
    ```

    메시지가 Claude Code 세션에 전달됩니다. 터미널에는 `← fakechat · web: what's in my working directory?`와 같은 수신 채널 줄로 표시되고, 모델은 플러그인의 범위 지정된 서버 이름을 사용하여 `<channel source="plugin:fakechat:fakechat">` 이벤트로 이를 수신합니다. Claude는 이를 읽고 작업을 수행한 뒤 fakechat의 `reply` 도구를 호출합니다. 첫 번째 답장은 터미널에서 권한 프롬프트를 트리거하며, 이를 승인하면 답변이 채팅 UI에 표시됩니다.
  </Step>
</Steps>

자리를 비운 동안 Claude가 권한 프롬프트를 만나면 응답할 때까지 세션이 일시 중지됩니다. [권한 릴레이 기능](/docs/en/channels-reference#relay-permission-prompts)을 선언한 채널 서버는 이러한 프롬프트를 원격으로 승인하거나 거부할 수 있도록 전달해 줄 수 있습니다. 무인 사용의 경우 [`--dangerously-skip-permissions`](/docs/en/permission-modes#skip-all-checks-with-bypasspermissions-mode)를 사용하면 대부분의 프롬프트를 우회할 수 있지만, 신뢰할 수 있는 환경에서만 사용하세요. 명시적 요청 규칙, [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구 및 [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구는 여전히 프롬프트를 표시합니다.

`-p` 옵션을 사용하여 비대화형 모드로 채널을 실행하면 객관식 질문이나 플랜 모드 승인과 같이 터미널 입력이 필요한 도구가 비활성화되어 세션이 입력을 기다리느라 멈추지 않습니다.

## 보안

승인된 모든 채널 플러그인은 발신자 허용 목록(allowlist)을 유지 관리합니다. 추가한 ID만 메시지를 푸시할 수 있으며, 그 외의 모든 메시지는 무시(drop)됩니다.

Telegram과 Discord는 페어링을 통해 목록을 부트스트랩합니다:

1. Telegram 또는 Discord에서 봇을 찾아 메시지를 전송합니다.
2. 봇이 페어링 코드로 답장합니다.
3. Claude Code 세션에서 프롬프트가 표시되면 코드를 승인합니다.
4. 발신자 ID가 허용 목록에 추가됩니다.

iMessage는 다르게 작동합니다. 자기 자신에게 문자 메시지를 보내면 게이트가 자동으로 우회되며, `/imessage:access allow` 명령으로 핸들을 통해 다른 연락처를 추가합니다.

게다가 각 세션마다 `--channels`로 활성화할 서버를 제어하며, 조직에서는 claude.ai Team 및 Enterprise 플랜과 관리형 설정을 배포하는 Console 조직의 [`channelsEnabled`](#조직-제어)를 통해 사용 가능 여부를 제어합니다.

`.mcp.json`에 포함되어 있는 것만으로는 메시지를 푸시하기에 충분하지 않으며, 서버가 `--channels`에도 지정되어야 합니다.

채널이 [권한 릴레이](/docs/en/channels-reference#relay-permission-prompts)를 선언한 경우 허용 목록은 권한 릴레이도 제어합니다. 채널을 통해 답장할 수 있는 사람은 누구나 세션에서 도구 사용을 승인하거나 거부할 수 있으므로, 해당 권한을 지닌 허용 목록 발신자만 신뢰하세요.

## 조직 제어 (Enterprise controls)

관리자는 사용자가 재정의할 수 없는 두 가지 [관리형 설정](/docs/en/settings)을 통해 사용 가능 여부를 제어합니다. 기본값은 인증 방식에 따라 다릅니다:

* **claude.ai Team 및 Enterprise**: 소유자가 활성화할 때까지 채널이 차단됩니다.
* **API 키 인증이 포함된 Anthropic Console**: 기본적으로 채널이 허용됩니다. 조직에서 관리형 설정을 배포하는 경우에만 이 설정이 필요합니다.

모든 경우에 사용자가 세션에 대해 `--channels`로 선택할 때까지는 어떠한 채널도 실행되지 않습니다.

| 설정                    | 목적                                                                                                                                                                                                                                                         | 설정되지 않은 경우                                                                                                                                                                     |
| :---------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `channelsEnabled`       | 마스터 스위치. 채널이 메시지를 전달하려면 `true`여야 합니다. [claude.ai 관리자 콘솔](https://claude.ai/admin-settings/claude-code) 토글 또는 관리형 설정에서 직접 설정합니다. 꺼져 있으면 개발 플래그를 포함한 모든 채널이 차단됩니다.                 | claude.ai Team 및 Enterprise: 채널 차단됨. Console: 조직에서 관리형 설정을 배포하지 않는 한 채널이 허용되며, 배포하는 경우 이 키가 설정될 때까지 채널이 차단됨                         |
| `allowedChannelPlugins` | 채널이 활성화된 후 등록할 수 있는 플러그인 목록입니다. 설정 시 Anthropic이 유지 관리하는 목록을 대체합니다. `channelsEnabled`가 `true`일 때만 적용됩니다.                                                                                                    | Anthropic 기본 목록이 적용됨                                                                                                                                                          |

조직이 없는 Pro 및 Max 사용자는 이러한 검사를 완전히 건너뜁니다. 채널을 사용할 수 있으며 사용자가 `--channels`로 세션별로 선택합니다.

### 조직에 대한 채널 활성화

소유자(Owner) 역할이 필요한 [**claude.ai → Admin settings → Claude Code → Channels**](https://claude.ai/admin-settings/claude-code)에서 조직의 채널을 활성화하거나, 관리형 설정에서 `channelsEnabled`를 `true`로 설정하여 활성화하세요.

활성화되면 조직의 사용자가 `--channels`를 사용하여 개별 세션에 채널 서버를 선택할 수 있습니다. 설정이 비활성화되어 있거나 설정되지 않은 경우에도 MCP 서버는 계속 연결되고 해당 도구는 작동하지만 채널 메시지는 도착하지 않습니다. 시작 시 표시되는 경고를 통해 관리자에게 설정을 활성화하도록 요청하라는 안내가 제공됩니다.

### 실행할 수 있는 채널 플러그인 제한

기본적으로 Anthropic이 유지 관리하는 허용 목록에 있는 모든 플러그인은 채널로 등록될 수 있습니다. Team 및 Enterprise 플랜의 관리자는 관리형 설정에서 `allowedChannelPlugins`를 설정하여 해당 허용 목록을 자체 목록으로 교체할 수 있습니다. 이를 사용하여 허용되는 공식 플러그인을 제한하거나 자체 내부 마켓플레이스의 채널을 승인하거나 둘 다 수행할 수 있습니다. 각 항목은 플러그인 이름과 해당 플러그인이 속한 마켓플레이스를 지정합니다:

```json theme={null}
{
  "channelsEnabled": true,
  "allowedChannelPlugins": [
    { "marketplace": "claude-plugins-official", "plugin": "telegram" },
    { "marketplace": "claude-plugins-official", "plugin": "discord" },
    { "marketplace": "acme-corp-plugins", "plugin": "internal-alerts" }
  ]
}
```

`allowedChannelPlugins`가 설정되면 Anthropic 허용 목록을 완전히 교체합니다. 즉, 목록에 포함된 플러그인만 등록할 수 있습니다. 기본 Anthropic 허용 목록으로 되돌리려면 설정하지 않은 상태로 두세요. 빈 배열을 설정하면 허용 목록의 모든 채널 플러그인이 차단되지만, `--dangerously-load-development-channels`를 사용하면 로컬 테스트를 위해 해당 차단을 우회할 수 있습니다. 개발 플래그를 포함하여 채널을 완전히 차단하려면 `channelsEnabled`를 설정하지 않은 상태로 두세요.

이 설정에는 `channelsEnabled: true`가 필요합니다. 사용자가 목록에 없는 플러그인을 `--channels`에 전달하는 경우 Claude Code는 정상적으로 시작되지만 채널은 등록되지 않으며, 시작 시 알림을 통해 해당 플러그인이 조직의 승인 목록에 없음을 설명합니다.

## 리서치 프리뷰

채널은 리서치 프리뷰 기능입니다. 사용 가능 여부가 순차적으로 배포되고 있으며 피드백에 따라 `--channels` 플래그 구문 및 프로토콜 계약이 변경될 수 있습니다.

기능이 프리뷰 상태인 동안에는 `claude --help`에 `--channels`나 `--dangerously-load-development-channels`가 표시되지 않습니다. 목록에 없더라도 플래그는 정상 동작합니다.

프리뷰 기간 동안 `--channels`는 Anthropic이 유지 관리하는 허용 목록의 플러그인 또는 관리자가 [`allowedChannelPlugins`](#실행할-수-있는-채널-플러그인-제한)를 설정한 경우 조직의 허용 목록에 있는 플러그인만 수락합니다. [claude-plugins-official](https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins)의 채널 플러그인이 기본 승인 세트입니다. 유효한 허용 목록에 없는 플러그인을 전달하면 Claude Code는 정상적으로 시작되지만 채널이 등록되지 않으며 시작 알림에서 그 이유를 알려줍니다.

작성 중인 채널을 테스트하려면 `--dangerously-load-development-channels`를 사용하세요. 직접 구축하는 커스텀 채널 테스트에 관한 자세한 내용은 [리서치 프리뷰 동안 테스트](/docs/en/channels-reference#test-during-the-research-preview)를 참조하세요.

[Claude Code GitHub 리포지토리](https://github.com/anthropics/claude-code/issues)에 문제나 피드백을 보고해 주세요.

## 채널 비교

여러 Claude Code 기능이 터미널 외부 시스템에 연결되며, 각 기능은 서로 다른 종류의 작업에 적합합니다:

| 기능                                                 | 수행 역할                                                             | 적합한 용도                                               |
| ---------------------------------------------------- | --------------------------------------------------------------------- | --------------------------------------------------------- |
| [웹 기반 Claude Code](/docs/en/claude-code-on-the-web) | GitHub에서 복제된 새로운 클라우드 샌드박스에서 작업 실행             | 나중에 확인할 자체 포함된 비동기 작업 위임                |
| [Slack에서의 Claude](/docs/en/slack)                 | 채널이나 스레드의 `@Claude` 멘션에서 웹 세션 생성                    | 팀 대화 컨텍스트에서 직접 작업 시작                       |
| 표준 [MCP 서버](/docs/en/mcp)                       | Claude가 작업 중 요청할 때 조회하며, 세션에 아무것도 푸시되지 않음     | Claude에게 시스템 읽기 또는 조회에 대한 요청 기반 접근 권한 부여 |
| [원격 제어 (Remote Control)](/docs/en/remote-control)  | claude.ai 또는 Claude 모바일 앱에서 로컬 세션을 직접 조작             | 자리를 비운 동안 진행 중인 세션 제어                      |

채널은 비-Claude 소스의 이벤트를 이미 실행 중인 로컬 세션으로 푸시하여 목록의 격차를 메웁니다.

* **채팅 브리지**: Telegram, Discord 또는 iMessage를 통해 휴대폰에서 Claude에게 질문하면, 작업이 실제 파일에 대해 시스템에서 실행되는 동안 동일한 채팅으로 답변이 돌아옵니다.
* **[웹훅 수신기](/docs/en/channels-reference#example-build-a-webhook-receiver)**: CI, 에러 트래커, 배포 파이프라인 또는 기타 외부 서비스의 웹훅이 Claude가 이미 파일을 열어두고 디버깅하던 내용을 기억하고 있는 곳으로 도착합니다.

## 다음 단계

채널이 실행되면 다음과 같은 관련 기능을 살펴보세요:

* 아직 플러그인이 없는 시스템을 위해 [나만의 채널 구축](/docs/en/channels-reference)
* 이벤트를 푸시하는 대신 휴대폰에서 로컬 세션을 직접 조작하기 위한 [원격 제어 (Remote Control)](/docs/en/remote-control)
* 푸시된 이벤트에 반응하는 대신 타이머에 따라 폴링하기 위한 [예약된 작업](/docs/en/scheduled-tasks)
