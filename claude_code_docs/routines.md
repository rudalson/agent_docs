> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 루틴을 통한 작업 자동화

> Claude Code를 자동 조종 모드로 설정하세요. 일정한 일정에 따라 실행되거나, API 호출로 트리거되거나, Anthropic 관리 클라우드 인프라에서 GitHub 이벤트에 반응하는 루틴을 정의할 수 있습니다.

<Note>
  루틴 기능은 리서치 프리뷰 상태입니다. 동작, 제한 사항 및 API 사양이 변경될 수 있습니다.
</Note>

루틴은 저장된 Claude Code 구성입니다. 즉, 프롬프트, 하나 이상의 리포지토리, 하나 이상의 [커넥터](/docs/en/mcp) 세트를 한 번 패키지화하여 자동으로 실행합니다. 루틴은 Anthropic 관리 클라우드 인프라에서 실행되므로 노트북이 닫혀 있어도 계속 작동합니다.

각 루틴에는 하나 이상의 트리거를 연결할 수 있습니다:

* **Scheduled (일정)**: 매시간, 매일 밤, 매주 등 주기적으로 실행되거나 특정 미래 시점에 한 번 실행
* **API**: 전달자 토큰(bearer token)을 사용하여 루틴별 엔드포인트에 HTTP POST를 전송하여 필요에 따라 트리거
* **GitHub**: 풀 리퀘스트 또는 릴리스와 같은 리포지토리 이벤트에 반응하여 자동으로 실행

단일 루틴에 여러 트리거를 조합할 수 있습니다. 예를 들어, PR 리뷰 루틴은 매일 밤 실행되면서 배포 스크립트에서도 트리거되고, 새 PR이 올라올 때마다 반응하도록 설정할 수 있습니다.

루틴은 [Claude Code on the web](/docs/en/claude-code-on-the-web)이 활성화된 Pro, Max, Team 및 Enterprise 플랜에서 사용할 수 있습니다. [claude.ai/code/routines](https://claude.ai/code/routines) 또는 CLI에서 `/schedule` 명령을 통해 루틴을 생성하고 관리할 수 있습니다.

Team 및 Enterprise 소유자(Owner)는 [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)의 루틴 토글을 통해 모든 멤버에 대해 루틴을 비활성화할 수 있습니다. 비활성화되면 기존 루틴 실행이 중지되며 멤버가 새 루틴을 생성할 수 없습니다.

이 페이지에서는 루틴 생성, 각 트리거 유형 구성, 실행 관리 및 사용 제한 적용 방식에 대해 설명합니다.

## 사용 예시

각 예시는 트리거 유형과 루틴에 적합한 작업 형태(자동화 가능하고 반복 가능하며 명확한 결과와 연결됨)를 보여줍니다.

**백로그 유지 관리.** 일정 트리거가 주중 매일 밤 커넥터를 통해 이슈 트래커를 대상으로 실행됩니다. 루틴은 마지막 실행 이후 열린 이슈를 읽고, 레이블을 적용하고, 참조된 코드 영역에 따라 담당자를 지정한 후 Slack에 요약을 게시하여 팀이 정리된 대기열로 하루를 시작할 수 있게 합니다.

**알림 트리아지.** 모니터링 도구가 오류 임계값을 초과할 때 알림 본문을 `text`로 전달하여 루틴의 API 엔드포인트를 호출합니다. 루틴 프롬프트는 전달된 페이로드의 알림을 조사하도록 Claude에 지시하므로, 스택 트레이스를 가져와 리포지토리의 최근 커밋과 연관 지어 분석하고 제안된 수정 사항과 알림 링크가 포함된 초안 풀 리퀘스트를 엽니다. 당직자는 빈 터미널에서 시작하는 대신 PR을 검토합니다.

**맞춤형 코드 리뷰.** GitHub 트리거가 `pull_request.opened` 시 실행됩니다. 루틴은 팀 자체의 리뷰 체크리스트를 적용하고 보안, 성능, 스타일 이슈에 대한 인라인 주석을 남기며 요약 주석을 추가하여 사람 리뷰어가 기계적인 점검 대신 설계에 집중할 수 있도록 합니다.

**배포 검증.** CD 파이프라인이 프로덕션 배포 후 루틴의 API 엔드포인트를 호출합니다. 루틴은 새 빌드에 대해 스모크 테스트를 실행하고 오류 로그에서 회귀를 스캔하여 배포 창이 닫히기 전에 릴리스 채널에 진행 또는 중단 메시지를 게시합니다.

**문서 이탈 점검.** 일정 트리거가 매주 실행됩니다. 루틴은 마지막 실행 이후 병합된 PR을 스캔하고, 변경된 API를 참조하는 문서를 플래그 지정하며, 편집자가 검토할 수 있도록 문서 리포지토리에 업데이트 PR을 엽니다.

**라이브러리 포팅.** GitHub 트리거가 하나의 SDK 리포지토리에서 병합된 PR에 대해 필터링되어 `pull_request.closed` 시 실행됩니다. 루틴은 변경 사항을 다른 언어로 된 병렬 SDK로 포팅하고 대응하는 PR을 열어, 사람이 각 변경 사항을 다시 구현하지 않고도 두 라이브러리의 동기화를 유지합니다.

아래 섹션에서는 루틴을 생성하고 이러한 각 트리거 유형을 구성하는 방법을 안내합니다.

## 루틴 생성하기

웹의 [claude.ai/code/routines](https://claude.ai/code/routines), Desktop 앱 또는 CLI에서 루틴을 생성할 수 있습니다. 세 가지 인터페이스 모두 동일한 클라우드 계정에 기록되므로 한 곳에서 생성한 루틴은 다른 곳에 즉시 표시됩니다. Desktop 앱에서는 사이드바의 **Routines**를 클릭한 다음 **New routine**을 누르고 **Cloud**를 선택합니다. **Local**을 선택하면 클라우드가 아닌 로컬 머신에서 실행되는 [Desktop scheduled task](/docs/en/desktop-scheduled-tasks)가 생성됩니다.

생성 양식에서는 루틴의 프롬프트, 리포지토리, 환경, 커넥터 및 트리거를 설정합니다.

루틴은 전체 Claude Code 클라우드 세션으로 자율 실행됩니다. 실행 중 권한 모드 선택기나 승인 프롬프트가 표시되지 않습니다. 세션은 쉘 명령을 실행하고, 복제된 리포지토리에 커밋된 [skills](/docs/en/skills)를 사용하며, 포함된 커넥터를 호출할 수 있습니다. 루틴이 접근할 수 있는 범위는 선택한 리포지토리 및 브랜치 푸시 설정, [환경의](/docs/en/claude-code-on-the-web#the-cloud-environment) 네트워크 접근 및 환경 변수, 포함된 커넥터에 의해 결정됩니다. 각 항목을 루틴에 실제 필요한 수준으로 범위 지정하세요.

루틴은 개별 claude.ai 계정에 속합니다. 팀원과 공유되지 않으며 계정의 일일 실행 허용량에 포함됩니다. 루틴이 연결된 GitHub 자격 증명이나 커넥터를 통해 수행하는 모든 작업은 본인 이름으로 표시됩니다. 커밋 및 풀 리퀘스트에는 본인의 GitHub 사용자가 표시되고 Slack 메시지, Linear 티켓 또는 기타 커넥터 작업에는 해당 서비스의 연결된 계정이 사용됩니다.

### 웹에서 생성하기

<Steps>
  <Step title="생성 양식 열기">
    [claude.ai/code/routines](https://claude.ai/code/routines)에 방문하여 **New routine**을 클릭합니다.
  </Step>

  <Step title="루틴 이름 지정 및 프롬프트 작성">
    루틴에 설명이 포함된 이름을 지정하고 Claude가 매번 실행할 프롬프트를 작성합니다. 프롬프트는 가장 중요한 부분입니다. 루틴이 자율적으로 실행되므로 프롬프트는 자체 완결적이어야 하며 수행할 작업과 성공의 기준을 명확하게 명시해야 합니다.

    트리거가 실행될 때 세션은 지정된 작업으로 루틴의 저장된 프롬프트를 수신하여 대화 중간에 도착한 신뢰할 수 없는 콘텐츠로 취급하지 않고 이를 수행합니다. 트리거는 프롬프트가 계정의 승인된 세션에 의해 사전에 저장되었음을 증명할 뿐이므로, 실행된 프롬프트는 라이브 사용자 입력이 아니며 실행 중인 작업에 대한 승인이나 동의로 작용할 수 없습니다. 세션이 실행 중에 가져오는 콘텐츠는 일반적인 처리 방식을 유지합니다. {/* min-version: 2.1.214 */}v2.1.214 이전에는 세션이 동일한 프롬프트를 신뢰할 수 없는 백그라운드 알림으로 수신하여 실행을 거부할 수 있었습니다.

    프롬프트 입력란에는 모델 선택기가 포함되어 있습니다. Claude는 매 실행마다 선택된 모델을 사용합니다.
  </Step>

  <Step title="리포지토리 선택">
    Claude가 작업할 하나 이상의 GitHub 리포지토리를 추가합니다. 각 리포지토리는 기본 브랜치에서 시작하여 실행을 시작할 때 복제됩니다. Claude는 변경 사항을 위해 `claude/` 접두사가 붙은 브랜치를 생성합니다.
  </Step>

  <Step title="환경 선택">
    루틴을 위한 [클라우드 환경](/docs/en/claude-code-on-the-web#the-cloud-environment)을 선택합니다. 환경은 클라우드 세션이 접근할 수 있는 대상을 제어합니다:

    * **Network access**: 각 실행 중에 사용할 수 있는 인터넷 접근 수준 설정
    * **Environment variables**: Claude가 사용할 수 있는 API 키, 토큰 또는 기타 보안 비밀 제공
    * **Setup script**: 루틴에 필요한 종속성 및 도구 설치. 결과는 [캐시되므로](/docs/en/claude-code-on-the-web#environment-caching) 스크립트가 세션마다 재실행되지 않습니다.

    **Trusted** 네트워크 접근이 포함된 **Default** 환경이 제공되어 패키지 레지스트리, 클라우드 제공업체 API, 컨테이너 레지스트리 및 일반적인 개발 도메인의 [기본 목록](/docs/en/claude-code-on-the-web#default-allowed-domains)을 허용하지만 그 외의 모든 것은 차단합니다. 루틴이 자체 서비스나 해당 목록 외의 도메인에 연결되어야 하는 경우 실행 전에 환경의 [네트워크 접근](/docs/en/claude-code-on-the-web#network-access)을 편집하세요. 별도의 환경을 사용하려면 먼저 [환경을 생성](/docs/en/claude-code-on-the-web#configure-your-environment)하세요.
  </Step>

  <Step title="트리거 선택">
    **Select a trigger** 아래에서 루틴 시작 방식을 선택합니다. 하나의 트리거 유형을 선택하거나 여러 개를 조합할 수 있습니다.

    <Tabs>
      <Tab title="Schedule">
        반복 실행을 위한 사전 설정 주기를 선택하거나 특정 타임스탬프에 실행되는 1회성 실행을 예약합니다. 시간대 처리, 시간 차 실행, 커스텀 cron 간격 및 1회성 실행에 대해서는 [일정 트리거 추가](#add-a-schedule-trigger)를 참조하세요.
      </Tab>

      <Tab title="GitHub event">
        리포지토리, 반응할 이벤트 및 선택적 필터를 선택합니다. 지원되는 이벤트 및 필터 필드의 전체 목록은 [GitHub 트리거 추가](#add-a-github-trigger)를 참조하세요.
      </Tab>

      <Tab title="API">
        여기서 **API**를 선택한 다음 루틴을 저장합니다. URL과 토큰은 루틴 ID에 의존하므로 루틴이 저장된 후에 생성됩니다. URL을 복사하고 토큰을 생성하려면 [API 트리거 추가](#add-an-api-trigger)를 참조하세요.
      </Tab>
    </Tabs>
  </Step>

  <Step title="커넥터 및 권한 검토">
    양식 하단의 **Connectors** 및 **Permissions** 탭은 루틴이 접근할 수 있는 대상을 제어합니다.

    Connectors 항목 아래에는 연결된 모든 [MCP 커넥터](/docs/en/mcp)가 기본적으로 포함됩니다. 루틴에 필요 없는 항목은 제거하세요. Claude는 실행 중에 권한을 요청하지 않고 포함된 커넥터의 모든 도구(쓰기 포함)를 사용할 수 있습니다.

    Permissions 항목 아래에서 Claude가 `claude/` 접두사가 붙은 브랜치뿐만 아니라 기존 브랜치에도 푸시할 수 있어야 하는 리포지토리에 대해 **Allow unrestricted branch pushes**를 활성화합니다.
  </Step>

  <Step title="루틴 생성">
    **Create**를 클릭합니다. 루틴이 목록에 표시되고 트리거 중 하나가 해당할 때 다음번에 실행됩니다. 즉시 실행을 시작하려면 루틴 세부 정보 페이지에서 **Run now**를 클릭합니다.

    각 실행은 다른 세션과 함께 새 세션을 생성하여 Claude가 수행한 작업을 확인하고, 변경 사항을 검토하고, 풀 리퀘스트를 생성할 수 있습니다.
  </Step>
</Steps>

### CLI에서 생성하기

임의의 세션에서 `/schedule`을 실행하여 대화 형식으로 예약된 루틴을 생성합니다. `/schedule daily PR review at 9am`과 같은 반복 루틴이나 `/schedule clean up feature flag in one week`와 같은 1회성 루틴에 대해 설명을 직접 전달할 수도 있습니다. Claude는 웹 양식이 수집하는 것과 동일한 정보를 확인한 다음 계정에 루틴을 저장합니다. 이 명령은 별칭인 `/routines`로도 이용 가능합니다.

성공적인 시작은 대화 형태로 진행됩니다: Claude가 저장하기 전에 일정, 리포지토리 및 프롬프트에 대해 추가 질문을 합니다. Claude가 인증이 필요하거나 원격 claude.ai 계정에 연결할 수 없다고 응답하면 루틴이 생성되지 않은 것입니다. [문제 해결](#troubleshooting)을 참조하세요.

CLI의 `/schedule`은 예약된 루틴만 생성합니다. API 또는 GitHub 트리거를 추가하려면 웹의 [claude.ai/code/routines](https://claude.ai/code/routines)에서 루틴을 편집하세요.

CLI는 기존 루틴 관리도 지원합니다. 모든 루틴을 보려면 `/schedule list`를, 루틴을 변경하려면 `/schedule update`를, 즉시 트리거하려면 `/schedule run`을 실행하세요.

API 호출이나 GitHub 이벤트로만 시작되는 등 일정 트리거가 없는 루틴은 다음 실행 시간이 없으며, Claude가 저장하거나 업데이트할 때 CLI에 아무것도 표시되지 않습니다. v2.1.211 이전에는 CLI가 이러한 루틴에 대해 1년의 다음 실행 시간을 보고했습니다.

## 트리거 구성하기

루틴은 트리거 중 하나가 일치할 때 시작됩니다. 동일한 루틴에 일정, API 및 GitHub 트리거의 모든 조합을 연결할 수 있으며, 루틴 편집 양식의 **Select a trigger** 섹션에서 언제든지 추가하거나 제거할 수 있습니다.

### 일정 트리거 추가 (Add a schedule trigger)

일정 트리거는 반복 주기에 따라 또는 미래의 특정 시간에 한 번 루틴을 실행합니다. **Select a trigger** 섹션에서 매시간, 매일, 평일 또는 매주 등 사전 설정된 주기를 선택합니다. 시간은 로컬 시간대로 입력되고 자동으로 변환되므로 클라우드 인프라의 위치와 상관없이 해당 시계 시간(wall-clock time)에 루틴이 실행됩니다.

스태거(시간 차 실행)로 인해 예약된 시간보다 몇 분 늦게 실행이 시작될 수 있습니다. 오프셋은 각 루틴마다 일정합니다.

2시간마다 또는 매월 1일과 같은 커스텀 간격의 경우 양식에서 가장 가까운 사전 설정을 선택한 다음 CLI에서 `/schedule update`를 실행하여 특정 cron 표현식을 설정하세요. 최소 간격은 1시간이며 더 자주 실행되는 표현식은 거부됩니다.

#### 1회성 실행 예약 (Schedule a one-off run)

1회성 일정은 특정 타임스탬프에 루틴을 단 한 번 실행합니다. 주 후반에 자신에게 알림을 주거나, 롤아웃이 완료된 후 정리 PR을 열거나, 상류 변경 사항이 반영될 때 후속 작업을 시작하는 데 사용하세요. 루틴이 실행된 후 자동으로 비활성화되며 웹 UI에 **Ran**으로 표시됩니다. 다시 실행하려면 루틴을 편집하고 새 1회성 시간을 설정하세요.

<Note>
  CLI에서의 1회성 예약 기능은 점진적으로 배포 중이며 아직 계정에서 사용하지 못할 수 있습니다. `/schedule`이 반복 일정만 제공하는 경우 웹([claude.ai/code/routines](https://claude.ai/code/routines))에서 1회성 실행을 생성하세요.
</Note>

자연어로 시간을 설명하여 CLI에서 1회성 실행을 생성할 수 있습니다. Claude는 현재 시간을 기준으로 문구를 확인하고 저장하기 전에 절대 타임스탬프를 확인합니다.

```text theme={null}
/schedule tomorrow at 9am, summarize yesterday's merged PRs
```

```text theme={null}
/schedule in 2 weeks, open a cleanup PR that removes the feature flag
```

반복 일정과 동일한 로컬-UTC 변환이 1회성 타임스탬프에도 적용됩니다.

1회성 실행은 일일 루틴 실행 한도에 포함되지 않습니다. 다른 세션과 마찬가지로 플랜의 일반 구독 사용량을 소비합니다. 자세한 내용은 [사용량 및 제한](#usage-and-limits)을 참조하세요.

### API 트리거 추가 (Add an API trigger)

API 트리거는 루틴에 전용 HTTP 엔드포인트를 제공합니다. 루틴의 전달자 토큰(bearer token)으로 엔드포인트에 POST를 전송하면 새 세션이 시작되고 세션 URL이 반환됩니다. 이를 사용하여 Claude Code를 알림 시스템, 배포 파이프라인, 내부 도구 또는 인증된 HTTP 요청을 보낼 수 있는 모든 곳에 연결할 수 있습니다.

API 트리거는 웹의 기존 루틴에 추가됩니다. CLI는 현재 토큰을 생성하거나 취소할 수 없습니다.

<Steps>
  <Step title="편집할 루틴 열기">
    [claude.ai/code/routines](https://claude.ai/code/routines)로 이동하여 API를 통해 트리거하려는 루틴을 클릭한 다음 연필 아이콘을 클릭하여 **Edit routine**을 엽니다.
  </Step>

  <Step title="API 트리거 추가">
    **Instructions** 상자 아래의 **Select a trigger** 섹션으로 스크롤하여 **Add another trigger**를 클릭하고 **API**를 선택합니다.
  </Step>

  <Step title="URL 복사 및 토큰 생성">
    모달에 샘플 curl 명령과 함께 이 루틴의 URL이 표시됩니다. URL을 복사한 다음 **Generate token**을 클릭하고 즉시 토큰을 복사합니다. 토큰은 한 번만 표시되며 나중에 조회할 수 없으므로 알림 도구의 보안 비밀 저장소와 같이 안전한 위치에 저장하세요.
  </Step>

  <Step title="엔드포인트 호출">
    URL에 POST 요청을 보낼 때 `Authorization: Bearer` 헤더에 토큰을 함께 전달합니다. 아래의 [루틴 트리거하기](#trigger-a-routine) 섹션에 전체 예시가 나와 있습니다.
  </Step>
</Steps>

각 루틴에는 해당 루틴 트리거 전용으로 범위가 지정된 자체 토큰이 있습니다. 토큰을 회전하거나 취소하려면 동일한 모달로 돌아가 **Regenerate** 또는 **Revoke**를 클릭합니다.

#### 루틴 트리거하기 (Trigger a routine)

`Authorization` 헤더에 전달자 토큰을 포함하여 `/fire` 엔드포인트에 POST 요청을 보냅니다. 요청 본문은 저장된 프롬프트와 함께 루틴에 전달되는 알림 본문이나 실패 로그와 같은 실행 관련 컨텍스트를 위해 옵션 필드인 `text`를 받아들입니다. 값은 자유 형식 텍스트이며 파싱되지 않습니다. JSON이나 다른 구조화된 페이로드를 보내면 루틴은 이를 리터럴 문자열로 받습니다.

`text` 값은 순수 메시지로 루틴에 전달되지 않습니다. 신뢰할 수 없는 데이터로 라벨을 지정하고 루틴의 자체 프롬프트가 지시하지 않는 한 내부 지침을 따르지 않도록 Claude에 알리는 `<routine-fire-payload>` 블록으로 감싸져 도착합니다. 웹 UI의 **Run now**에서 제공되는 텍스트에도 동일한 랩핑이 적용됩니다.

이는 루틴의 저장된 프롬프트가 실행 텍스트에 대한 처리를 명시적으로 허용해야 함을 의미합니다. 프롬프트를 작성할 때 예를 들어 "routine-fire-payload 블록에 설명된 알림을 조사하세요"와 같이 페이로드를 명시적으로 참조하도록 작성하세요. 그렇지 않으면 루틴은 텍스트를 비활성 컨텍스트로 취급합니다. 전달자 토큰을 가진 사람이라면 누구나 `text`를 보낼 수 있으므로, 래퍼는 유출된 토큰의 실행 텍스트가 루틴에 대한 직접 지시가 아니라 신뢰할 수 없는 데이터로 라벨 지정되어 도착하도록 보장합니다.

아래 예시는 쉘에서 루틴을 트리거하는 방법을 보여줍니다. 표시된 루틴 ID와 토큰은 자리 표시자입니다. [API 트리거 추가](#add-an-api-trigger) 시 복사한 URL과 토큰으로 교체하세요. 그렇지 않으면 요청이 `401` 인증 오류로 실패합니다:

```bash theme={null}
curl -X POST https://api.anthropic.com/v1/claude_code/routines/trig_01ABCDEFGHJKLMNOPQRSTUVW/fire \
  -H "Authorization: Bearer sk-ant-oat01-xxxxx" \
  -H "anthropic-beta: experimental-cc-routine-2026-04-01" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"text": "Sentry alert SEN-4521 fired in prod. Stack trace attached."}'
```

성공적인 요청은 새 세션 ID 및 URL이 포함된 JSON 본문을 반환합니다:

```json theme={null}
{
  "type": "routine_fire",
  "claude_code_session_id": "session_01HJKLMNOPQRSTUVWXYZ",
  "claude_code_session_url": "https://claude.ai/code/session_01HJKLMNOPQRSTUVWXYZ"
}
```

브라우저에서 세션 URL을 열어 실시간으로 실행을 확인하거나, 변경 사항을 검토하거나, 대화를 수동으로 이어갈 수 있습니다.

<Warning>
  `/fire` 엔드포인트는 `experimental-cc-routine-2026-04-01` 베타 헤더 아래 제공됩니다. 기능이 리서치 프리뷰에 있는 동안 요청 및 응답 형태, 속도 제한, 토큰 의미가 변경될 수 있습니다. 주요 변경 사항은 새로운 날짜가 지정된 베타 헤더 버전으로 제공되며, 호출자가 마이그레이션할 시간을 확보할 수 있도록 최근 2개의 이전 헤더 버전이 계속 작동합니다.
</Warning>

#### API 참조 문서 (API reference)

모든 오류 응답, 검증 규칙 및 필드 제한을 포함한 전체 API 참조 문서는 Claude Platform 문서의 [Trigger a routine via API](https://platform.claude.com/docs/en/api/claude-code/routines-fire)를 참조하세요.

`/fire` 엔드포인트는 claude.ai 사용자 전용이며 Claude Platform API 범위에 포함되지 않습니다.

### GitHub 트리거 추가 (Add a GitHub trigger)

GitHub 트리거는 연결된 리포지토리에서 일치하는 이벤트가 발생할 때 자동으로 새 세션을 시작합니다. 일치하는 각 이벤트는 자체 세션을 시작합니다.

<Note>
  리서치 프리뷰 동안 GitHub 웹훅 이벤트에는 루틴별 및 계정별 시간당 한도가 적용됩니다. 한도를 초과한 이벤트는 창이 재설정될 때까지 누락됩니다. [claude.ai/code/routines](https://claude.ai/code/routines)에서 현재 한도를 확인하세요.
</Note>

GitHub 트리거는 웹 UI에서만 구성할 수 있습니다.

<Steps>
  <Step title="편집할 루틴 열기">
    [claude.ai/code/routines](https://claude.ai/code/routines)로 이동하여 루틴을 클릭한 다음 연필 아이콘을 클릭하여 **Edit routine**을 엽니다.
  </Step>

  <Step title="GitHub 이벤트 트리거 추가">
    **Select a trigger** 섹션으로 스크롤하여 **Add another trigger**를 클릭하고 **GitHub event**를 선택합니다.
  </Step>

  <Step title="Claude GitHub App 설치">
    구독하려는 리포지토리에 Claude GitHub App이 설치되어 있어야 합니다. 앱이 설치되어 있지 않은 경우 트리거 설정에서 설치를 요청하는 프롬프트가 표시됩니다.

    <Note>
      CLI에서 `/web-setup`을 실행하면 클론 작업을 위한 리포지토리 접근 권한은 부여되지만 Claude GitHub App이 설치되지는 않으며 웹훅 전달도 활성화되지 않습니다. GitHub 트리거에는 Claude GitHub App 설치가 필요하며 트리거 설정 시 안내가 제공됩니다.
    </Note>
  </Step>

  <Step title="트리거 구성">
    리포지토리를 선택하고 [지원되는 이벤트](#supported-events) 목록에서 이벤트를 선택한 후 선택적으로 필터를 추가합니다. 트리거를 저장합니다.
  </Step>
</Steps>

#### 지원되는 이벤트 (Supported events)

GitHub 트리거는 다음 이벤트 카테고리 중 하나에 구독할 수 있습니다. 각 카테고리 내에서 `pull_request.opened`와 같은 특정 작업을 선택하거나 카테고리의 모든 작업에 반응할 수 있습니다.

| 이벤트 | 트리거 시점 |
| :----------- | :---------------------------------------------------------------------------- |
| Pull request | PR이 생성, 닫힘, 할당, 레이블 지정, 동기화 또는 업데이트될 때 |
| Release | 릴리스가 생성, 게시, 편집 또는 삭제될 때 |

#### 풀 리퀘스트 필터링 (Filter pull requests)

필터를 사용하여 어떤 풀 리퀘스트가 새 세션을 시작할지 범위를 좁힙니다. 루틴이 트리거되려면 모든 필터 조건이 일치해야 합니다. 사용 가능한 필터 필드는 다음과 같습니다:

| 필터 | 일치 항목 |
| :---------- | :------------------------------- |
| Author | PR 작성자의 GitHub 사용자 이름 |
| Title | PR 제목 텍스트 |
| Body | PR 설명 텍스트 |
| Base branch | PR이 타겟으로 하는 브랜치 |
| Head branch | PR이 시작된 브랜치 |
| Labels | PR에 적용된 레이블 |
| Is draft | PR이 초안(draft) 상태인지 여부 |
| Is merged | PR이 병합(merged)되었는지 여부 |

각 필터는 필드와 연산자(equals, contains, starts with, is one of, is not one of, 또는 matches regex)를 조합합니다.

`matches regex` 연산자는 내부의 부분 문자열이 아니라 전체 필드 값을 테스트합니다. `hotfix`를 포함하는 임의의 제목과 일치시키려면 `.*hotfix.*`를 작성하세요. 감싸는 `.*`가 없으면 앞뒤에 다른 문자가 없는 정확히 `hotfix`인 제목만 필터와 일치합니다. 정규식 구문 없이 리터럴 부분 문자열 일치를 수행하려면 `contains` 연산자를 대신 사용하세요.

몇 가지 예시 필터 조합:

* **인증 모듈 리뷰**: base branch `main`, head branch contains `auth-provider`. 인증을 수정하는 모든 PR을 전담 리뷰어에게 전달합니다.
* **리뷰 준비 완료 건만**: is draft is `false`. 초안을 건너뛰어 PR이 리뷰 준비가 되었을 때만 루틴이 실행됩니다.
* **레이블 조건부 백포트**: labels include `needs-backport`. 메인테이너가 PR에 태그를 지정했을 때만 타 브랜치 포팅 루틴을 트리거합니다.

#### 세션과 이벤트의 매핑 방식

일치하는 각 GitHub 이벤트는 새 세션을 시작합니다. GitHub 기반 트리거 루틴에서는 이벤트 간 세션 재사용이 지원되지 않으므로 두 번의 PR 업데이트는 두 개의 독립적인 세션을 생성합니다.

## 루틴 관리하기

목록에서 루틴을 클릭하여 세부 정보 페이지를 엽니다. 세부 정보 페이지에는 루틴의 리포지토리, 커넥터, 프롬프트, 일정, API 토큰, GitHub 트리거 및 과거 실행 목록이 표시됩니다.

### 실행 확인 및 상호작용

실행 항목을 클릭하여 전체 세션으로 엽니다. 거기서 Claude가 수행한 작업을 확인하고, 변경 사항을 검토하고, 풀 리퀘스트를 생성하거나, 대화를 이어갈 수 있습니다. 각 실행 세션은 다른 세션과 동일하게 작동합니다. 세션 제목 옆의 드롭다운 메뉴를 사용하여 이름 변경, 보관 또는 삭제할 수 있습니다.

<Note>
  실행 목록의 녹색 상태 표시는 세션이 인프라 오류 없이 시작되고 종료되었음을 의미합니다. 프롬프트의 작업이 성공했음을 의미하지는 않습니다. 실행을 열어 트랜스크립트를 읽고 Claude가 실제로 수행한 작업을 확인하세요. 차단된 네트워크 요청, 누락된 커넥터 도구 및 작업 수준의 실패는 상태 표시기가 아닌 트랜스크립트에 표출됩니다.
</Note>

### 루틴 편집 및 제어

루틴 세부 정보 페이지에서 다음을 수행할 수 있습니다:

* 다음 예약 시간까지 기다리지 않고 즉시 실행을 시작하려면 **Run now**를 클릭합니다. 선택적으로 실행 관련 텍스트를 제공할 수 있으며, 이는 API 트리거의 `text` 필드와 동일한 방식으로 루틴에 전달됩니다.
* **Repeats** 섹션의 토글을 사용하여 일정을 일시 중지하거나 다시 시작합니다. 일시 중지된 루틴은 구성을 유지하지만 다시 활성화할 때까지 실행되지 않습니다.
* 연필 아이콘을 클릭하여 **Edit routine**을 열고 이름, 프롬프트, 리포지토리, 환경, 커넥터 또는 루틴의 트리거를 변경합니다. **Select a trigger** 섹션에서 일정, API 토큰 및 GitHub 이벤트 트리거를 추가하거나 제거할 수 있습니다.
* 삭제 아이콘을 클릭하여 루틴을 제거합니다. 루틴에 의해 생성된 과거 세션은 세션 목록에 그대로 남습니다.

### 리포지토리 및 브랜치 권한

루틴이 리포지토리를 복제하려면 GitHub 접근 권한이 필요합니다. CLI에서 `/schedule`로 루틴을 생성할 때 Claude는 계정에 GitHub가 연결되어 있는지 확인하고 그렇지 않은 경우 `/web-setup`을 실행하라는 프롬프트를 표시합니다. 접근 권한을 부여하는 두 가지 방법에 대해서는 [GitHub 인증 옵션](/docs/en/claude-code-on-the-web#github-authentication-options)을 참조하세요.

추가된 각 리포지토리는 매 실행 시 복제됩니다. Claude는 프롬프트에 별도로 지정하지 않는 한 리포지토리의 기본 브랜치에서 시작합니다.

기본적으로 Claude는 `claude/` 접두사가 붙은 브랜치에만 푸시할 수 있습니다. 이는 루틴이 보호되거나 장기간 유지되는 브랜치를 실수로 수정하는 것을 방지합니다. 특정 리포지토리에 대해 이 제한을 해제하려면 루틴을 생성하거나 편집할 때 해당 리포지토리에 대해 **Allow unrestricted branch pushes**를 활성화하세요.

### 커넥터 (Connectors)

루틴은 연결된 MCP 커넥터를 사용하여 각 실행 중에 외부 서비스에서 읽고 쓸 수 있습니다. 예를 들어 지원 요청을 트리아지하는 루틴은 Slack 채널에서 읽고 Linear에서 이슈를 생성할 수 있습니다.

커넥터는 계정의 [claude.ai 통합](/docs/en/mcp#use-mcp-servers-from-claude-ai)입니다. CLI에서 `claude mcp add`로 로컬에 추가한 MCP 서버는 claude.ai 계정이 아닌 로컬 머신에 저장되므로 커넥터 목록에 표시되지 않습니다. 루틴에서 이러한 서버 중 하나를 사용하려면 [claude.ai/customize/connectors](https://claude.ai/customize/connectors)에서 커넥터로 추가하거나, 커밋된 [`.mcp.json`](/docs/en/mcp#project-scope)에 선언하여 복제된 리포지토리의 일부가 되도록 하세요.

루틴을 생성하면 현재 연결된 모든 커넥터가 기본적으로 포함됩니다. 실행 중 Claude가 접근할 수 있는 도구를 제한하려면 필요 없는 항목을 제거하세요. 루틴 양식에서 직접 커넥터를 추가할 수도 있습니다.

루틴 양식 외부에서 커넥터를 관리하거나 추가하려면 [claude.ai/customize/connectors](https://claude.ai/customize/connectors)를 방문하거나 CLI에서 `/schedule update`를 사용하세요.

### 환경 및 네트워크 접근

각 루틴은 네트워크 접근, 환경 변수 및 설정 스크립트를 제어하는 [클라우드 환경](/docs/en/claude-code-on-the-web#the-cloud-environment)에서 실행됩니다. 루틴은 모든 실행에서 환경의 네트워크 정책을 상속받습니다.

**Default** 환경은 **Trusted** 네트워크 접근을 사용합니다: 패키지 레지스트리, 클라우드 제공업체 API, 컨테이너 레지스트리 및 일반적인 개발 도메인의 [기본 허용 목록](/docs/en/claude-code-on-the-web#default-allowed-domains)에는 접근할 수 있지만 임의의 도메인은 접근할 수 없습니다. 다른 호스트에 대한 아웃바운드 요청은 `403` 및 `x-deny-reason: host_not_allowed`로 실패합니다. MCP 커넥터 트래픽은 Anthropic의 서버를 통해 라우팅되므로 루틴에 추가한 커넥터는 호스트를 **Allowed domains**에 추가하지 않고도 작동합니다. [Connectors](#connectors) 항목 아래에서 필요 없는 커넥터를 제거하세요.

추가 도메인을 허용하려면:

<Steps>
  <Step title="편집할 루틴 열기">
    루틴 세부 정보 페이지에서 연필 아이콘을 클릭하여 **Edit routine**을 엽니다.
  </Step>

  <Step title="환경 선택기 열기">
    **Instructions** 상자 아래에서 **Default**와 같이 환경 이름이 표시된 클라우드 아이콘을 선택합니다.
  </Step>

  <Step title="환경 설정 열기">
    목록의 환경 위에 마우스를 올리고 오른쪽에 나타나는 설정 아이콘을 클릭합니다.
  </Step>

  <Step title="네트워크 접근 수준 변경">
    **Update cloud environment** 대화 상자에서 **Network access**를 **Custom**으로 변경하고 **Allowed domains**에 도메인을 입력합니다. 사용자 지정 도메인과 함께 [기본 허용 목록](/docs/en/claude-code-on-the-web#default-allowed-domains)을 유지하려면 **Also include default list of common package managers**를 체크하세요. 제한 없는 접근을 원하면 **Full**을 선택합니다.
  </Step>

  <Step title="저장">
    **Save changes**를 클릭합니다. 새 정책은 다음 실행부터 적용됩니다.
  </Step>
</Steps>

접근 수준 및 기본 허용 목록에 대한 자세한 내용은 [네트워크 접근](/docs/en/claude-code-on-the-web#network-access)을 참조하세요.

## 사용량 및 제한 사항

루틴은 대화형 세션과 동일한 방식으로 구독 사용량을 차감합니다. 표준 구독 제한 외에도 루틴에는 계정당 시작할 수 있는 실행 횟수에 대한 일일 한도가 있습니다. [claude.ai/code/routines](https://claude.ai/code/routines) 또는 [claude.ai/settings/usage](https://claude.ai/settings/usage)에서 현재 소비량과 남은 일일 루틴 실행 횟수를 확인하세요.

루틴이 일일 한도나 구독 사용량 제한에 도달하면 사용 크레딧(usage credits)이 활성화된 조직은 종량제 초과 사용으로 루틴을 계속 실행할 수 있습니다. 사용 크레딧이 없으면 창이 재설정될 때까지 추가 실행이 거부됩니다. [claude.ai/settings/usage](https://claude.ai/settings/usage)에서 사용 크레딧을 켜세요. Team 및 Enterprise 플랜의 경우 관리자가 [claude.ai/admin-settings/usage](https://claude.ai/admin-settings/usage)에서 조직에 대한 크레딧을 활성화합니다.

1회성 실행은 일일 루틴 한도에 포함되지 않습니다. 다른 세션과 마찬가지로 일반 구독 사용량을 차감하지만 계정당 일일 루틴 실행 허용량에서는 제외됩니다.

## 문제 해결

### `/schedule`이 "Unknown command"를 반환함

CLI는 요구 사항 중 하나가 충족되지 않을 때 `/schedule`을 숨깁니다. 입력을 입력하는 동안 명령 메뉴에 `No commands match "/schedule"`이 표시되고 제출하면 `Unknown command: /schedule`이 반환됩니다. 원인은 보통 다음 중 하나입니다:

* Console API 키 또는 Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry와 같은 클라우드 제공업체로 인증했습니다. `/schedule`에는 claude.ai 구독 로그인이 필요합니다. 쉘에 `ANTHROPIC_API_KEY`나 `ANTHROPIC_AUTH_TOKEN`이 설정되어 있거나 `settings.json`에 `apiKeyHelper`가 설정되어 있는 경우, 이러한 설정이 claude.ai 로그인보다 우선하므로 먼저 제거하세요.
* 쉘 환경이나 [`settings.json` 파일](/docs/en/settings#available-settings)의 `env` 블록에 `DISABLE_TELEMETRY`, `DO_NOT_TRACK`, `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` 또는 `DISABLE_GROWTHBOOK`이 설정되어 있습니다. 이들은 `/schedule`이 의존하는 기능 플래그 조회를 비활성화합니다.
* 웹 세션의 Claude Code 내부입니다. 대신 [웹 UI](https://claude.ai/code/routines)에서 루틴을 관리하세요.

CLI 구성 방식과 관계없이 언제든지 [claude.ai/code/routines](https://claude.ai/code/routines)에서 루틴을 생성하고 관리할 수 있습니다.

### `/schedule`이 인증을 요청함

`/schedule`이 실행되지만 Claude가 먼저 claude.ai 계정으로 인증해야 한다고 응답하는 경우, CLI에 저장된 claude.ai 로그인이 없는 것입니다. API 계정은 루틴에 지원되지 않습니다. `/login`을 실행하고 claude.ai 계정으로 로그인한 다음 `/schedule`을 다시 실행하세요.

### "Routines are disabled by your organization's policy"

Team 또는 Enterprise 조직의 소유자(Owner)가 [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)에서 **Routines** 토글을 껐을 가능성이 높습니다. 이는 서버 측 조직 설정이므로 로컬 구성에서 재정의할 수 없습니다. 소유자에게 조직에 대한 루틴을 활성화하도록 요청하세요.

## 관련 리소스

* [`/loop` 및 세션 내 예약](/docs/en/scheduled-tasks): 열려 있는 CLI 세션 내에서 로컬 작업 예약
* [Desktop scheduled tasks](/docs/en/desktop-scheduled-tasks): 로컬 파일 접근 권한이 있는 머신에서 실행되는 로컬 예약 작업
* [클라우드 환경](/docs/en/claude-code-on-the-web#the-cloud-environment): 클라우드 세션용 런타임 환경 구성
* [MCP 커넥터](/docs/en/mcp): Slack, Linear, Google Drive 등의 외부 서비스 연결
* [GitHub Actions](/docs/en/github-actions): 리포지토리 이벤트 발생 시 CI 파이프라인에서 Claude 실행
