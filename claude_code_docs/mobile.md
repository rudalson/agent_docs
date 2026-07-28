> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 모바일에서의 Claude Code

> iOS 및 Android용 Claude 앱을 사용하여 휴대폰에서 Claude Code 태스크를 시작, 모니터링 및 제어합니다.

[iOS](https://apps.apple.com/us/app/claude-by-anthropic/id6473753684) 및 [Android](https://play.google.com/store/apps/details?id=com.anthropic.claude)용 Claude 앱은 코드가 직접 실행되는 공간이 아니라 Claude Code 세션을 위한 클라이언트입니다. 휴대폰을 사용하여 Anthropic이 관리하는 인프라 위의 [클라우드 세션(cloud sessions)](#start-and-monitor-cloud-sessions)에 접속하거나, [Remote Control](#continue-a-local-session-with-remote-control)을 통해 사용자의 본인 머신에서 구동되는 세션에 접속하거나, [Dispatch](/docs/en/desktop#sessions-from-dispatch)를 통해 Desktop 앱에 접근할 수 있습니다.

<Note>
  Claude Code에는 별도의 모바일 앱이 없습니다: 클라우드 세션과 Remote Control 모두 Claude 앱의 **Code** 탭에 존재하며, Dispatch는 앱에서 메시지를 보내는 태스크 형태입니다.
</Note>

## 앱 다운로드

<Steps>
  <Step title="Claude 앱 다운로드">
    [iOS](https://apps.apple.com/us/app/claude-by-anthropic/id6473753684) 또는 [Android](https://play.google.com/store/apps/details?id=com.anthropic.claude)용 Claude 앱을 설치하세요. iPad의 경우 동일한 iOS 앱을 설치합니다.

    <Tip>
      Claude Code 세션에서 `/mobile`을 실행하면 스캔 가능한 다운로드 QR 코드가 표시됩니다. `/ios` 및 `/android`도 동일하게 동작합니다.
    </Tip>
  </Step>

  <Step title="로그인">
    Claude Code에서 사용하는 것과 동일한 claude.ai 계정 및 조직으로 로그인하세요. 클라우드 세션과 Remote Control은 claude.ai 계정이 필요하므로, Anthropic Console API 키나 Amazon Bedrock과 같은 서드파티 프로바이더를 통해서는 접근할 수 없습니다.
  </Step>

  <Step title="Code 탭 열기">
    앱 내 내비게이션에서 **Code**를 탭하여 세션에 접근하거나, 휴대폰에서 [claude.ai/code/new](https://claude.ai/code/new)를 열어 앱에서 새로운 Code 세션을 시작하세요. Code 탭이 보이지 않는 경우 사용 중인 요금제나 조직에 해당 기능이 포함되어 있지 않은 것일 수 있습니다; [구독 요금제별 제공 여부](/docs/en/feature-availability#availability-by-subscription-plan)를 참조하세요.
  </Step>
</Steps>

## 휴대폰에서 작업하기

앱을 사용하여 클라우드 세션을 시작하거나, 컴퓨터에서 실행 중인 Claude Code 세션을 조종하거나, Dispatch에 태스크 메시지를 전송할 수 있습니다. 세 가지 모두 앱 사용 환경은 동일하며 작업이 어디서 일어나는지만 다릅니다.

| 기능                                                 | 연결 대상                                           | 사용 시점                                                                                                                                            |
| :--------------------------------------------------- | :-------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Claude Code on the web](/docs/en/claude-code-on-the-web) | Anthropic 관리형 클라우드 인프라의 클라우드 세션    | 저장소가 GitHub에 있고 휴대폰을 치운 후에도 작업이 계속 진행되어야 할 때. 설정 방법은 [웹 빠른 시작](/docs/en/web-quickstart) 참조.                  |
| [Remote Control](/docs/en/remote-control)                 | 컴퓨터에서 구동 중인 Claude Code 세션               | 작업에 사용자의 로컬 파일 시스템, 도구, 또는 MCP 서버가 필요할 때.                                                                                   |
| [Dispatch](/docs/en/desktop#sessions-from-dispatch)       | 컴퓨터의 Desktop 앱                                 | 태스크 메시지를 보내고 Dispatch가 실행 방식을 스스로 결정하게 하고 싶을 때. Pro 또는 Max 요금제 필요.                                                |

컴퓨터가 꺼질 예정이라면 클라우드 세션을 사용하세요: Anthropic의 인프라에서 실행되므로 노트북을 닫아도 계속 진행됩니다. Remote Control과 Dispatch는 사용자의 머신을 조종하므로 Claude Code 또는 Desktop 앱이 실행된 상태로 머신이 켜져 있어야 합니다. Remote Control 세션 중 머신이 수면 상태로 진입하더라도 다시 온라인이 되었을 때 세션이 다시 연결됩니다.

채널, Slack, 예약된 태스크까지 다루는 보다 완전한 비교는 [터미널을 비운 동안 작업하기](/docs/en/platforms#work-when-you-are-away-from-your-terminal)를 참조하세요.

클라우드 세션과 Remote Control은 **Code** 탭에서 구동되며 아래에서 다룹니다. 앱에서 태스크 형태로 메시지를 보내는 Dispatch에 대해서는 [Dispatch를 통한 세션](/docs/en/desktop#sessions-from-dispatch)을 참조하세요.

### 클라우드 세션 시작 및 모니터링

Claude Code on the web은 Anthropic 관리형 클라우드 인프라에서 태스크를 구동하므로 휴대폰을 치운 후에도 세션이 유지됩니다. Code 탭에서 저장소와 브랜치를 선택하고 태스크를 설명한 후 제출하세요. 세션은 장치 전반에 걸쳐 지속됩니다: 노트북에서 시작한 태스크를 휴대폰에서 검토할 수 있고, 휴대폰에서 시작한 태스크가 데스크로 돌아왔을 때 기다리고 있습니다.

앱에서 세션을 열어 진행 상황을 확인하고, Claude의 질문에 답하거나, 새로운 방향으로 지시하세요. Claude에게 [풀 리퀘스트를 모니터링](/docs/en/claude-code-on-the-web#auto-fix-pull-requests)하고 CI 실패나 리뷰 댓글이 도착하면 즉시 수정하도록 지시할 수도 있습니다. GitHub을 연결하고 첫 환경을 생성하려면 [웹 빠른 시작](/docs/en/web-quickstart)을 따르고, 클라우드 세션이 할 수 있는 모든 것에 대해서는 [Claude Code on the web](/docs/en/claude-code-on-the-web)을 참조하세요.

### Remote Control로 로컬 세션 이어하기

Remote Control은 Claude 앱을 컴퓨터에서 실행 중인 Claude Code 세션에 연결하므로 휴대폰에서 세션을 조종하는 동안에도 코드 실행 및 파일 시스템 접근은 로컬에 머무릅니다. 컴퓨터에서 `claude remote-control`로 세션을 시작하거나 이미 열려 있는 세션에서 `/remote-control`을 실행하세요. 그런 다음 터미널에 표시된 세션 QR 코드를 스캔하거나, Claude 앱을 열고 **Code**를 탭한 뒤 목록에서 해당 세션을 선택하세요. 각 옵션은 [다른 장치에서 연결하기](/docs/en/remote-control#connect-from-another-device)를 참조하세요.

Claude 앱에서 추가한 첨부 파일도 로컬 세션에 전달됩니다: Claude Code가 사용자의 머신으로 이미지나 파일을 다운로드하고 `@` 파일 참조 형태로 Claude에게 전달합니다. 요구사항, 호출 모드, 문제 해결에 대해서는 [Remote Control 개요](/docs/en/remote-control)를 참조하세요.

### 푸시 알림 받기

Remote Control이 활성화되어 있을 때 장시간 구동되는 태스크가 완료되거나 사용자 의사결정이 필요할 때 Claude가 휴대폰으로 푸시 알림을 보낼 수 있습니다. `notify me when the tests finish`와 같이 프롬프트에서 알림을 직접 요청할 수도 있습니다. 두 가지 `/config` 토글 및 알림 전송 문제 해결은 [모바일 푸시 알림](/docs/en/remote-control#mobile-push-notifications)을 참조하세요.

Dispatch는 자신이 생성한 Code 세션이 완료되거나 승인이 필요할 때 자체 알림을 보내며, 이는 [Dispatch를 통한 세션](/docs/en/desktop#sessions-from-dispatch)에 설명되어 있습니다.

## 제한 사항

모바일 클라이언트는 몇 가지 제한 사항을 제외하고 세션에 필요한 대부분을 다룹니다:

* **로컬 전용 명령**: `/plugin` 및 `/resume`과 같이 터미널 인터페이스에서만 작동하는 명령은 앱에서 작동하지 않습니다. [Remote Control 제한 사항](/docs/en/remote-control#limitations)에 모바일에서 작동하는 명령과 해당 동작 방식 차이가 나열되어 있습니다.
* **권한 모드**: 클라우드 세션은 모드 드롭다운에서 Accept edits, Plan, Auto를 제공하고, Remote Control 세션은 Manual, Accept edits, Plan을 제공합니다. 두 경우 모두 앱에서 Bypass permissions를 선택할 수 없으며, Remote Control 세션에서는 Auto를 선택할 수 없습니다. [권한 모드 전환](/docs/en/permission-modes#switch-permission-modes)을 참조하세요.
* **Dispatch 요금제**: Dispatch는 Pro 또는 Max 요금제가 필요하며 Team이나 Enterprise에서는 제공되지 않습니다.

## 관련 리소스

* [플랫폼 및 통합](/docs/en/platforms): Claude Code가 구동되는 모든 서페이스 비교
* [Claude Code on the web](/docs/en/claude-code-on-the-web): 클라우드 세션 실행 방식, 네트워크 접근, 터미널과의 작업 이동
* [Remote Control](/docs/en/remote-control): 모든 장치에서 로컬 세션 이어하기
* [Dispatch를 통한 세션](/docs/en/desktop#sessions-from-dispatch): Dispatch 태스크가 Desktop 앱에서 Code 세션이 되는 방식
* [채널](/docs/en/channels): 작업이 사용자 머신에서 구동되는 동안 Telegram, Discord, iMessage를 통해 휴대폰으로 Claude에게 문의하기
* [Slack에서의 Claude Code](/docs/en/slack): `@Claude`를 멘션하여 Slack 워크스페이스에서 코딩 작업 위임하기
