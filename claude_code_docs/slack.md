> ## 문서 목차
> 문서 전체 목차 보기: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Slack에서 Claude Code 사용하기

> Slack 워크스페이스에서 직접 코딩 작업을 위임하세요.

<Note>
  Slack의 Claude Code는 Team 및 Enterprise 워크스페이스용 [Claude Tag](https://claude.com/product/tag)로 대체되고 있습니다. Claude Tag는 관리자가 구성을 마친 액세스 권한으로 조직의 공유 ID로서 동일한 Slack 앱에서 @Claude를 실행하므로 다시 설치할 필요가 없으며 기존 설정은 전환 중에도 계속 작동합니다. 워크스페이스를 전환하려면 [이전 Slack의 Claude에서 마이그레이션](https://claude.com/docs/claude-tag/admins/migrate-from-earlier)을 참조하세요.
</Note>

Slack의 Claude Code는 Claude Code의 강력한 기능을 Slack 워크스페이스로 직접 가져옵니다. 코딩 작업에 대해 `@Claude`를 멘션하면 Claude가 의도를 자동으로 감지하여 웹에 Claude Code 세션을 생성하므로 팀 대화를 떠나지 않고도 개발 작업을 위임할 수 있습니다.

이 통합은 기존 Claude for Slack 앱을 기반으로 구축되었지만 코딩 관련 요청의 경우 웹상의 Claude Code로 지능적으로 라우팅하는 기능이 추가되었습니다. 각 세션은 연결된 리포지토리 및 플랜 제한 사항을 사용하여 사용자 고유의 Claude 계정에서 실행됩니다.

## 사용 사례

* **버그 조사 및 수정**: Slack 채널에 보고되는 즉시 Claude에게 버그를 조사하고 수정하도록 요청합니다.
* **빠른 코드 검토 및 수정**: 팀 피드백에 따라 Claude가 소규모 기능을 구현하거나 코드를 리팩터링하도록 합니다.
* **협업 디버깅**: 팀 토론에서 중요한 컨텍스트(예: 오류 재현 또는 사용자 보고)를 제공할 때 Claude는 해당 정보를 사용하여 디버깅 접근 방식에 반영할 수 있습니다.
* **병렬 작업 실행**: 다른 작업을 계속 진행하는 동안 Slack에서 코딩 작업을 시작하고 완료되면 알림을 받습니다.

## 사전 요구 사항

Slack에서 Claude Code를 사용하기 전에 다음 사항을 확인하세요:

| 요구 사항 | 세부 정보 |
| :--- | :--- |
| Claude 플랜 | Claude Code 접근 권한이 있는 Pro, Max, Team 또는 Enterprise (프리미엄 시트 또는 Chat + Claude Code 시트) |
| 웹상의 Claude Code | [웹상의 Claude Code](/docs/en/claude-code-on-the-web)에 대한 접근 권한이 활성화되어 있어야 함 |
| GitHub 계정 | 인증된 리포지토리가 하나 이상 있는 상태로 웹상의 Claude Code에 연결되어 있어야 함 |
| Slack 인증 | Claude 앱을 통해 Claude 계정에 연결된 Slack 계정 |

## Slack에서 Claude Code 설정하기

<Steps>
  <Step title="Slack에 Claude 앱 설치">
    워크스페이스 관리자가 Slack 앱 마켓플레이스에서 Claude 앱을 설치해야 합니다. [Slack 앱 마켓플레이스](https://slack.com/marketplace/A08SF47R6P4)를 방문하여 "Add to Slack"을 클릭하고 설치 프로세스를 시작하세요.
  </Step>

  <Step title="Claude 계정 연결">
    앱이 설치된 후 개별 Claude 계정을 인증합니다:

    1. 앱 섹션에서 "Claude"를 클릭하여 Slack에서 Claude 앱을 엽니다.
    2. 앱 홈 탭으로 이동합니다.
    3. Slack 계정을 Claude 계정에 연결하려면 "Connect"를 클릭합니다.
    4. 브라우저에서 인증 흐름을 완료합니다.
  </Step>

  <Step title="웹상의 Claude Code 구성">
    웹상의 Claude Code가 올바르게 구성되었는지 확인하세요:

    * [claude.ai/code](https://claude.ai/code)를 방문하여 Slack에 연결한 계정과 동일한 계정으로 로그인합니다.
    * 아직 연결되지 않은 경우 GitHub 계정을 연결합니다.
    * Claude가 작업하도록 할 리포지토리를 하나 이상 인증합니다.
  </Step>

  <Step title="라우팅 모드 선택">
    계정을 연결한 후 Claude가 Slack에서 메시지를 처리하는 방식을 구성합니다. Slack의 Claude 앱 홈으로 이동하여 **Routing Mode** 설정을 찾으세요.

    | 모드 | 동작 |
    | :--- | :--- |
    | **Code only** | Claude가 모든 @멘션을 Claude Code 세션으로 라우팅합니다. Slack에서 전적으로 개발 작업용으로 Claude를 사용하는 팀에 가장 적합합니다. |
    | **Code + Chat** | Claude가 각 메시지를 분석하여 Claude Code(코딩 작업용)와 Claude Chat(글쓰기, 분석 및 일반 질문용) 간에 지능적으로 라우팅합니다. 모든 유형의 작업에 단일 @Claude 진입점을 원하는 팀에 가장 적합합니다. |

    <Note>
      Code + Chat 모드에서 Claude가 메시지를 Chat으로 라우팅했지만 코딩 세션을 원했던 경우, "Retry as Code"를 클릭하여 Claude Code 세션을 대신 생성할 수 있습니다. 마찬가지로 Code로 라우팅되었지만 Chat 세션을 원했던 경우 해당 스레드에서 해당 옵션을 선택할 수 있습니다.
    </Note>
  </Step>

  <Step title="채널에 Claude 추가">
    설치 후 Claude가 채널에 자동으로 추가되지 않습니다. 채널에서 Claude를 사용하려면 해당 채널에 `/invite @Claude`를 입력하여 초대하세요. Claude는 추가된 채널에서만 @멘션에 응답할 수 있습니다.
  </Step>
</Steps>

## 작동 방식

### 자동 감지

Slack 채널이나 스레드에서 @Claude를 멘션하면 Claude가 메시지를 자동으로 분석하여 코딩 작업인지 판단합니다. Claude가 코딩 의도를 감지하면 일반 대화 어시스턴트로 응답하는 대신 요청을 웹상의 Claude Code로 라우팅합니다.

자동으로 감지되지 않더라도 Claude에게 요청을 코딩 작업으로 처리하도록 명시적으로 지시할 수도 있습니다.

<Note>
  Slack의 Claude Code는 채널(공개 또는 비공개)에서만 작동합니다. 다이렉트 메시지(DM)에서는 작동하지 않습니다.
</Note>

### 컨텍스트 수집

**스레드에서**: 스레드에서 Claude를 @멘션하면 전체 대화를 이해하기 위해 해당 스레드의 모든 메시지에서 컨텍스트를 수집합니다.

**채널에서**: 채널에서 직접 멘션되면 Claude는 관련된 컨텍스트를 찾기 위해 최근 채널 메시지를 확인합니다.

이 컨텍스트는 Claude가 문제를 이해하고 적절한 리포지토리를 선택하며 작업 접근 방식에 대한 정보를 얻는 데 도움을 줍니다.

<Warning>
  Slack에서 @Claude가 호출되면 요청을 더 잘 이해할 수 있도록 대화 컨텍스트에 대한 접근 권한이 Claude에게 제공됩니다. Claude가 컨텍스트의 다른 메시지에 포함된 지침을 따를 수 있으므로 신뢰할 수 있는 Slack 대화에서만 Claude를 사용해야 합니다.
</Warning>

### 세션 흐름

1. **시작**: 코딩 요청과 함께 Claude를 @멘션함
2. **감지**: Claude가 메시지를 분석하고 코딩 의도를 감지함
3. **세션 생성**: claude.ai/code에 새 Claude Code 세션이 생성됨
4. **진행 상황 업데이트**: 작업이 진행됨에 따라 Claude가 Slack 스레드에 상태 업데이트를 게시함
5. **완료**: 작업이 끝나면 Claude가 요약 및 조치 버튼과 함께 사용자를 @멘션함
6. **검토**: 전체 트랜스크립트를 보려면 "View Session"을 클릭하고, 풀 리퀘스트를 열려면 "Create PR"을 클릭함

## 사용자 인터페이스 요소

### 앱 홈 (App Home)

앱 홈 탭은 연결 상태를 보여주고 Slack에서 Claude 계정을 연결하거나 연결 해제할 수 있도록 합니다.

### 메시지 조치 (Message actions)

* **View Session**: 브라우저에서 전체 Claude Code 세션을 열어 수행된 모든 작업을 확인하거나 세션을 계속하거나 추가 요청을 할 수 있습니다.
* **Create PR**: 세션의 변경 사항에서 직접 풀 리퀘스트를 생성합니다.
* **Retry as Code**: Claude가 처음에 대화 어시스턴트로 응답했지만 코딩 세션을 원했던 경우 이 버튼을 클릭하여 요청을 Claude Code 작업으로 다시 시도합니다.
* **Change Repo**: Claude가 잘못 선택한 경우 다른 리포지토리를 선택할 수 있도록 합니다.

### 리포지토리 선택

Claude는 Slack 대화의 컨텍스트를 기반으로 리포지토리를 자동으로 선택합니다. 여러 리포지토리가 적용될 수 있는 경우 Claude는 올바른 리포지토리를 선택할 수 있는 드롭다운을 표시할 수 있습니다.

## 접근 및 권한

### 사용자 수준 접근

| 접근 유형 | 요구 사항 |
| :--- | :--- |
| Claude Code 세션 | 각 사용자는 고유한 Claude 계정에서 세션을 실행함 |
| 사용량 및 속도 제한 | 세션은 개별 사용자의 플랜 제한에 합산됨 |
| 리포지토리 접근 | 사용자는 직접 연결한 리포지토리에만 접근할 수 있음 |
| 세션 기록 | 세션은 claude.ai/code의 Claude Code 기록에 표시됨 |

### 워크스페이스 수준 접근

Slack 워크스페이스 관리자는 워크스페이스에서 Claude 앱을 사용할 수 있는지 여부를 제어합니다:

| 제어 | 설명 |
| :--- | :--- |
| 앱 설치 | 워크스페이스 관리자가 Slack 앱 마켓플레이스에서 Claude 앱을 설치할지 결정함 |
| Enterprise Grid 배포 | Enterprise Grid 조직의 경우 조직 관리자가 Claude 앱에 접근할 수 있는 워크스페이스를 제어할 수 있음 |
| 앱 제거 | 워크스페이스에서 앱을 제거하면 해당 워크스페이스의 모든 사용자에 대한 접근 권한이 즉시 취소됨 |

### 채널 기반 접근 제어

설치 후 Claude가 채널에 자동으로 추가되지 않습니다. 사용자는 사용하고자 하는 채널에 Claude를 명시적으로 초대해야 합니다:

* **초대 필요**: 임의의 채널에 `/invite @Claude`를 입력하여 해당 채널에 Claude를 추가함
* **채널 멤버십으로 접근 제어**: Claude는 추가된 채널에서만 @멘션에 응답할 수 있음
* **채널을 통한 접근 게이팅**: 관리자는 Claude가 초대되는 채널과 해당 채널에 대한 접근 권한을 관리하여 Claude Code 사용자를 제어할 수 있음
* **비공개 채널 지원**: Claude는 공개 및 비공개 채널 모두에서 작동하므로 팀에 가시성 제어의 유연성을 제공함

이 채널 기반 모델을 통해 팀은 Claude Code 사용을 특정 채널로 제한할 수 있으며 워크스페이스 수준 권한을 넘어 추가적인 접근 제어 레이어를 제공합니다.

## 조회가 가능한 위치

**Slack에서**: 상태 업데이트, 완료 요약 및 조치 버튼이 표시됩니다. 전체 트랜스크립트는 보존되어 항상 열람할 수 있습니다.

**웹에서**: 전체 대화 기록, 모든 코드 변경 사항, 파일 작업 및 세션을 계속하거나 풀 리퀘스트를 생성하는 기능을 갖춘 완전한 Claude Code 세션입니다.

Enterprise 및 Team 계정의 경우 Slack의 Claude에서 생성된 세션이 조직에 자동으로 표시됩니다. 자세한 내용은 [웹상의 Claude Code 공유](/docs/en/claude-code-on-the-web#share-sessions)를 참조하세요.

## 모범 사례

### 효과적인 요청 작성하기

* **구체적으로 작성**: 관련 있는 경우 파일 이름, 함수 이름 또는 오류 메시지를 포함하세요.
* **컨텍스트 제공**: 대화에서 명확하지 않은 경우 리포지토리나 프로젝트를 언급하세요.
* **성공 기준 정의**: "완료"의 모습을 설명하세요(Claude가 테스트를 작성해야 하는지, 문서를 업데이트해야 하는지, PR을 생성해야 하는지 등).
* **스레드 사용**: 버그나 기능에 대해 논의할 때 스레드로 응답하여 Claude가 전체 컨텍스트를 수집할 수 있도록 하세요.

### Slack 대 웹 사용 시점

**Slack 사용 시점**: 컨텍스트가 Slack 토론에 이미 존재하는 경우, 비동기로 작업을 시작하려는 경우 또는 가시성이 필요한 팀원과 협업하는 경우.

**웹 직접 사용 시점**: 파일을 업로드해야 하는 경우, 개발 중 실시간 상호작용을 원하는 경우 또는 더 길고 복잡한 작업을 수행하는 경우.

## 문제 해결

### "Claude Code is not enabled for your account"

이 오류는 관리자가 무언가를 활성화해야 하는 것이 아니라 Claude 계정에 아직 클라우드 환경이 없음을 의미합니다. Slack에 연결한 것과 동일한 계정으로 [claude.ai/code](https://claude.ai/code)에 한 번 로그인하세요. 첫 방문 시 기본 클라우드 환경이 생성되고 다음 멘션 시 오류가 제거됩니다. 각 사용자가 이를 개별적으로 수행해야 합니다.

### 세션이 시작되지 않음

1. Claude 앱 홈에서 Claude 계정이 연결되어 있는지 확인합니다.
2. 웹상의 Claude Code 접근 권한이 활성화되어 있는지 확인합니다.
3. Claude Code에 최소 하나의 GitHub 리포지토리가 연결되어 있는지 확인합니다.

### 리포지토리가 표시되지 않음

1. [claude.ai/code](https://claude.ai/code)의 웹상의 Claude Code에서 리포지토리를 연결합니다.
2. 해당 리포지토리에 대한 GitHub 권한을 확인합니다.
3. GitHub 계정을 연결 해제했다가 다시 연결해 보세요.

### 잘못된 리포지토리가 선택됨

1. 다른 리포지토리를 선택하려면 "Change Repo" 버튼을 클릭합니다.
2. 더 정확한 선택을 위해 요청에 리포지토리 이름을 포함하세요.

### 인증 오류

1. 앱 홈에서 Claude 계정을 연결 해제했다가 다시 연결합니다.
2. 브라우저에서 올바른 Claude 계정으로 로그인되어 있는지 확인합니다.
3. Claude 플랜에 Claude Code 접근 권한이 포함되어 있는지 확인합니다.

### 세션 만료

1. 웹의 Claude Code 기록에서 세션에 계속 접근할 수 있습니다.
2. [claude.ai/code](https://claude.ai/code)에서 이전 세션을 계속하거나 참조할 수 있습니다.

## 현재 제한 사항

* **GitHub 전용**: 현재 GitHub의 리포지토리만 지원합니다.
* **한 번에 하나의 PR**: 각 세션은 하나의 풀 리퀘스트를 생성할 수 있습니다.
* **속도 제한 적용**: 세션은 개별 Claude 플랜의 속도 제한을 사용합니다.
* **웹 접근 필요**: 사용자가 웹상의 Claude Code 접근 권한을 가지고 있어야 합니다. 권한이 없는 사용자는 일반 Claude 대화 응답만 받게 됩니다.

## 관련 리소스

<CardGroup>
  <Card title="웹상의 Claude Code" icon="globe" href="/docs/en/claude-code-on-the-web">
    웹상의 Claude Code에 대해 자세히 알아보기
  </Card>

  <Card title="Claude for Slack" icon="slack" href="https://claude.com/claude-and-slack">
    일반 Claude for Slack 문서
  </Card>

  <Card title="Claude Tag" icon="users" href="https://claude.com/docs/claude-tag/overview">
    관리자가 구성을 마친 액세스 권한을 통해 Slack에서 조직 관리형 @Claude 사용
  </Card>

  <Card title="Slack 앱 마켓플레이스" icon="store" href="https://slack.com/marketplace/A08SF47R6P4">
    Slack 마켓플레이스에서 Claude 앱 설치
  </Card>

  <Card title="Claude 도움말 센터" icon="circle-question" href="https://support.claude.com">
    추가 지원 받기
  </Card>
</CardGroup>
