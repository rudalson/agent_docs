> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 웹에서 Claude Code 시작하기

> 브라우저나 휴대전화에서 클라우드의 Claude Code를 실행하세요. 로컬 설정 없이 GitHub 리포지토리를 연결하고, 작업을 제출하며, PR을 리뷰하세요.

<Note>
  Claude Code on the web은 Pro, Max 및 Team 사용자, 그리고 프리미엄 시트 또는 Chat + Claude Code 시트가 있는 Enterprise 사용자를 위한 리서치 프리뷰(research preview) 상태입니다.
</Note>

Claude Code on the web은 사용자의 머신 대신 Anthropic이 관리하는 클라우드 인프라에서 실행됩니다. 브라우저의 [claude.ai/code](https://claude.ai/code)나 Claude 모바일 앱에서 작업을 제출하세요.

[시작하려면](#connect-github-and-create-an-environment) GitHub 리포지토리가 필요합니다. Claude는 이를 격리된 가상 머신으로 클론하고, 변경 사항을 적용하며, 사용자가 리뷰할 수 있도록 브랜치를 푸시합니다. 세션은 기기 간에 유지되므로 노트북에서 시작한 작업을 나중에 휴대전화에서 리뷰할 준비를 마칠 수 있습니다.

Claude Code on the web은 다음과 같은 경우에 잘 맞습니다:

* **병렬 작업**: 여러 워크트리를 관리하지 않고 각각 자체 세션과 브랜치에서 여러 독립적인 작업을 동시에 실행
* **로컬에 없는 리포지토리**: Claude가 매 세션마다 리포지토리를 새로 클론하므로 로컬에 체크아웃할 필요가 없음
* **자주 조율할 필요가 없는 작업**: 잘 정의된 작업을 제출하고, 다른 일을 하다가, Claude가 완료하면 결과를 리뷰
* **코드 질문 및 탐색**: 로컬 체크아웃 없이 코드베이스를 이해하거나 기능이 구현된 방식을 추적

사용자의 로컬 구성, 도구 또는 환경이 필요한 작업에는 로컬에서 Claude Code를 실행하거나 [Remote Control](/docs/en/remote-control)을 사용하는 것이 더 적합합니다.

## 세션 실행 방식

작업을 제출하면:

1. **클론 및 준비**: 리포지토리가 Anthropic 관리 VM으로 클론되고, 구성된 경우 [setup script](/docs/en/claude-code-on-the-web#setup-scripts)가 실행됩니다.
2. **네트워크 구성**: 환경의 [access level](/docs/en/claude-code-on-the-web#access-levels)에 따라 인터넷 접근이 설정됩니다.
3. **작업**: Claude가 코드를 분석하고, 변경을 수행하며, 테스트를 실행하고, 작업을 검사합니다. 전체 과정을 지켜보며 조율하거나, 자리를 비웠다가 완료되었을 때 돌아올 수 있습니다.
4. **브랜치 푸시**: Claude가 정지 지점에 도달하면 해당 브랜치를 GitHub에 푸시합니다. 사용자는 diff를 리뷰하고, 인라인 주석을 남기며, PR을 생성하거나, 계속 진행하도록 다른 메시지를 보냅니다.

브랜치가 푸시된다고 해서 세션이 닫히지는 않습니다. PR 생성과 추가 편집은 모두 동일한 대화 내에서 발생합니다.

## Claude Code 실행 방식 비교

Claude Code는 어디서나 동일하게 동작합니다. 달라지는 것은 코드가 실행되는 위치와 로컬 구성을 사용할 수 있는지 여부입니다. Desktop 앱은 로컬 세션과 클라우드 세션을 모두 제공하므로 아래의 답변은 선택한 항목에 따라 달라집니다:

| | 웹에서 (On the web) | Remote Control | 터미널 CLI | Desktop 앱 |
| :------------------------------------------- | :------------------------------------------------------------------------------------------------------------- | :------------------------- | :--------------------- | :-------------------------- |
| **코드 실행 위치** | Anthropic 클라우드 VM | 사용자의 머신 | 사용자의 머신 | 사용자의 머신 또는 클라우드 VM |
| **채팅 위치** | claude.ai 또는 모바일 앱 | claude.ai 또는 모바일 앱 | 사용자의 터미널 | Desktop UI |
| **로컬 구성 사용** | 아니오, 리포지토리 전용 | 예 | 예 | 로컬은 예, 클라우드는 아니오 |
| **GitHub 필요 여부** | 예, 또는 `--cloud`로 [로컬 리포지토리 번들링](/docs/en/claude-code-on-the-web#send-local-repositories-without-github) | 아니오 | 아니오 | 클라우드 세션에만 필요 |
| **연결 해제 시 지속 실행** | 예 | 터미널이 열려 있는 동안 | 아니오 | 세션 유형에 따라 다름 |
| **[권한 모드](/docs/en/permission-modes)** | Accept edits, Plan, Auto | Manual, Accept edits, Plan | 모든 모드 | 세션 유형에 따라 다름 |
| **네트워크 접근** | 환경별 구성 가능 | 사용자의 머신 네트워크 | 사용자의 머신 네트워크 | 세션 유형에 따라 다름 |

이들을 설정하려면 [terminal quickstart](/docs/en/quickstart), [Desktop app](/docs/en/desktop), 또는 [Remote Control](/docs/en/remote-control) 문서를 참조하세요.

## GitHub 연결 및 환경 생성

설정은 1회성 프로세스입니다. 이미 GitHub CLI를 사용 중인 경우 브라우저 대신 [터미널에서 이 작업을 수행](#connect-from-your-terminal)할 수 있습니다.

<Steps>
  <Step title="claude.ai/code 방문">
    [claude.ai/code](https://claude.ai/code)로 이동하여 Anthropic 계정으로 로그인합니다.
  </Step>

  <Step title="Claude GitHub App 설치">
    로그인 후 claude.ai/code가 GitHub 연결을 요청합니다. 안내를 따라 Claude GitHub App을 설치하고 리포지토리에 대한 접근 권한을 부여하세요. 클라우드 세션은 기존 GitHub 리포지토리에서 작동하므로 새 프로젝트를 시작하려면 먼저 [GitHub에서 빈 리포지토리를 생성](https://github.com/new)하세요.
  </Step>

  <Step title="환경 생성">
    GitHub을 연결하면 클라우드 환경을 생성하라는 프롬프트가 표시됩니다. 환경은 세션 동안 Claude가 갖는 네트워크 접근 권한과 새 세션이 생성될 때 실행되는 내용을 제어합니다. 별도의 구성 없이 사용할 수 있는 항목은 [Installed tools](/docs/en/claude-code-on-the-web#installed-tools)를 참조하세요.

    양식에는 다음 필드가 있습니다:

    * **Name**: 표시 레이블. 서로 다른 프로젝트나 접근 수준에 대해 여러 환경을 가지고 있을 때 유용합니다.
    * **Network access**: 세션이 인터넷에서 도달할 수 있는 대상을 제어합니다. 기본값인 `Trusted`는 일반 인터넷 접근을 차단하면서 npm, PyPI, RubyGems와 같은 [공통 패키지 레지스트리](/docs/en/claude-code-on-the-web#default-allowed-domains)로의 연결을 허용합니다.
    * **Environment variables**: `.env` 형식의 모든 세션에서 사용할 수 있는 선택적 변수입니다. 따옴표가 값의 일부로 저장되므로 값을 따옴표로 감싸지 마세요. 이들은 해당 환경을 편집할 수 있는 모든 사람에게 표시됩니다.
    * **Setup script**: Claude Code가 시작되기 전에 실행되는 선택적 Bash 스크립트입니다. `apt install -y gh`와 같이 클라우드 VM에 포함되어 있지 않은 시스템 도구를 설치하는 데 사용합니다. 결과가 [캐시](/docs/en/claude-code-on-the-web#environment-caching)되므로 스크립트가 매 세션마다 다시 실행되지는 않습니다. 예시 및 디버깅 팁은 [Setup scripts](/docs/en/claude-code-on-the-web#setup-scripts)를 참조하세요.

    첫 번째 프로젝트의 경우 기본값을 그대로 두고 **Create environment**를 클릭하세요. 나중에 [이를 편집하거나 다른 프로젝트를 위해 추가 환경을 생성](/docs/en/claude-code-on-the-web#configure-your-environment)할 수 있습니다.
  </Step>
</Steps>

### 터미널에서 연결

이미 GitHub CLI(`gh`)를 사용 중인 경우 브라우저를 열지 않고도 웹에서 Claude Code를 설정할 수 있습니다. 이를 위해서는 [Claude Code CLI](/docs/en/quickstart)가 필요합니다. `/web-setup`은 로컬 `gh` 토큰을 읽고, 이를 Claude 계정에 연결하며, 아직 클라우드 환경이 없는 경우 기본 클라우드 환경을 생성합니다.

<Note>
  [Zero Data Retention](/docs/en/zero-data-retention)이 활성화된 조직은 `/web-setup` 또는 기타 클라우드 세션 기능을 사용할 수 없습니다. GitHub CLI가 설치되어 있지 않거나 인증되지 않은 경우 `/web-setup`은 브라우저 온보딩 흐름을 대신 엽니다.
</Note>

<Steps>
  <Step title="GitHub CLI로 인증">
    쉘에서 아직 인증하지 않았다면 GitHub CLI를 인증합니다:

    ```bash theme={null}
    gh auth login
    ```
  </Step>

  <Step title="Claude에 로그인">
    Claude Code CLI에서 `/login`을 실행하여 claude.ai 계정으로 로그인합니다. 이미 로그인한 경우 이 단계를 건너뜁니다.
  </Step>

  <Step title="/web-setup 실행">
    Claude Code CLI에서 다음을 실행합니다:

    ```text theme={null}
    /web-setup
    ```

    이렇게 하면 `gh` 토큰이 Claude 계정에 동기화됩니다. 아직 클라우드 환경이 없는 경우 `/web-setup`은 Trusted 네트워크 접근 권한과 설정 스크립트가 없는 클라우드 환경을 생성합니다. 나중에 [환경을 편집하거나 변수를 추가](/docs/en/claude-code-on-the-web#configure-your-environment)할 수 있습니다. `/web-setup`이 완료되면 [`--cloud`](/docs/en/claude-code-on-the-web#from-terminal-to-web)를 사용하여 터미널에서 클라우드 세션을 시작하거나 [`/schedule`](/docs/en/routines)로 반복 작업을 설정할 수 있습니다.
  </Step>
</Steps>

## 작업 시작하기

GitHub이 연결되고 환경이 생성되면 작업을 제출할 준비가 된 것입니다.

<Steps>
  <Step title="리포지토리 및 브랜치 선택">
    [claude.ai/code](https://claude.ai/code)나 Claude 모바일 앱의 Code 탭에서 입력 상자 아래의 리포지토리 선택기를 클릭하고 Claude가 작업할 리포지토리를 선택하세요. 각 리포지토리에는 브랜치 선택기가 표시됩니다. 기본 브랜치 대신 피처 브랜치에서 Claude를 시작하려면 변경하세요. 단일 세션에서 여러 리포지토리를 가로질러 작업하도록 여러 리포지토리를 추가할 수 있습니다.
  </Step>

  <Step title="권한 모드 선택">
    입력 옆의 모드 드롭다운은 기본적으로 **Accept edits**로 설정되어 있으며, 이 모드에서 Claude는 승인을 받기 위해 멈추지 않고 변경을 수행하고 브랜치를 푸시합니다. 파일을 편집하기 전에 Claude가 접근 방식을 제안하고 승인을 기다리도록 하려면 **Plan**으로 전환하세요. 클라우드 세션은 Manual 또는 Bypass 권한을 제공하지 않습니다. 각 권한 모드가 허용하는 바는 [전체 권한 모드 목록](/docs/en/permission-modes#available-modes)을 참조하세요.
  </Step>

  <Step title="작업 설명 작성 및 제출">
    원하는 작업 설명을 입력하고 Enter를 누르세요. 구체적으로 작성하세요:

    * 파일이나 함수 이름 명시: "fix tests"보다 "Add a README with setup instructions" 또는 "Fix the failing auth test in `tests/test_auth.py`"가 더 낫습니다
    * 오류 출력이 있는 경우 붙여넣기
    * 단순 증상뿐만 아니라 예상되는 동작 설명

    Claude가 리포지토리를 클론하고, 구성된 경우 설정 스크립트를 실행한 다음 작업을 시작합니다. 각 작업은 자체 세션과 브랜치를 받으므로 다른 작업을 시작하기 전에 하나가 완료될 때까지 기다릴 필요가 없습니다.
  </Step>
</Steps>

## 세션 사전 채우기

[claude.ai/code](https://claude.ai/code) URL에 쿼리 매개변수를 추가하여 새 세션의 프롬프트, 리포지토리 및 환경을 사전에 채울 수 있습니다. 이슈 트래커의 버튼을 통해 이슈 설명을 프롬프트로 사용하여 Claude Code를 여는 것과 같은 통합을 구축할 때 이를 사용하세요.

| 매개변수 | 설명 |
| :------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| `prompt` | 입력 상자에 미리 채울 프롬프트 텍스트. 별칭 `q`도 허용됨. |
| `prompt_url` | 쿼리 문자열에 포함하기에는 너무 긴 프롬프트에 대해 프롬프트 텍스트를 가져올 URL. URL은 교차 출처(cross-origin) 요청을 허용해야 함. `prompt`도 설정된 경우 무시됨. |
| `repositories` | 사전 선택할 `owner/repo` 슬러그의 쉼표로 구분된 목록. 별칭 `repo`도 허용됨. |
| `environment` | 사전 선택할 [환경](#connect-github-and-create-an-environment)의 이름 또는 ID. |

각 값을 URL 인코딩하세요. 아래 예시는 프롬프트와 리포지토리가 이미 선택된 상태로 양식을 엽니다:

```text theme={null}
https://claude.ai/code?prompt=Fix%20the%20login%20bug&repositories=acme/webapp
```

## 리뷰 및 반복

Claude가 완료되면 변경 사항을 리뷰하고, 특정 줄에 피드백을 남기며, diff가 만족스러울 때까지 계속 진행하세요.

<Steps>
  <Step title="diff 뷰 열기">
    diff 표시기는 세션 전체에서 추가되거나 제거된 줄 수를 보여줍니다(예: `+42 -18`). 선택하면 좌측에 파일 목록이, 우측에 변경 사항이 있는 diff 뷰가 열립니다.
  </Step>

  <Step title="인라인 주석 남기기">
    diff에서 임의의 줄을 선택하고 피드백을 입력한 후 Enter를 누르세요. 주석은 다음 메시지를 보낼 때까지 대기열에 들어간 다음 함께 번들로 전달됩니다. 문제의 위치를 설명할 필요 없이 Claude는 기본 지침과 함께 "at `src/auth.ts:47`, don't catch the error here"를 보게 됩니다.
  </Step>

  <Step title="풀 리퀘스트 생성">
    diff가 제대로 보이면 diff 뷰 상단에서 **Create PR**을 선택하세요. 전체 PR, 드래프트로 열거나, 생성된 제목과 설명이 있는 GitHub의 작성 페이지로 건너뛸 수 있습니다.
  </Step>

  <Step title="PR 이후에도 계속 반복">
    PR이 생성된 후에도 세션은 계속 유지됩니다. CI 실패 출력이나 리뷰어 주석을 채팅에 붙여넣고 Claude에게 처리하도록 요청하세요. Claude가 PR을 자동으로 모니터링하도록 하려면 [Auto-fix pull requests](/docs/en/claude-code-on-the-web#auto-fix-pull-requests)를 참조하세요.
  </Step>
</Steps>

## 설정 문제 해결

### GitHub 연결 후 리포지토리가 보이지 않음

클라우드 세션은 Claude GitHub App이 설치된 리포지토리에 관계없이 연결된 GitHub 계정이 볼 수 있는 모든 리포지토리를 사용할 수 있습니다. 리포지토리가 누락된 경우 연결된 GitHub 계정이 GitHub에서 해당 리포지토리에 접근할 수 있는지 확인하세요. 리포지토리에 대해 [Auto-fix](/docs/en/claude-code-on-the-web#auto-fix-pull-requests)도 원하는 경우 해당 리포지토리에 App을 설치하세요: github.com에서 **Settings → Applications → Claude → Configure**를 열고 **Repository access** 아래에 리포지토리가 나열되어 있는지 확인합니다. 사설 리포지토리는 공개 리포지토리와 동일한 승인이 필요합니다.

### 페이지에 GitHub 로그인 버튼만 표시됨

클라우드 세션에는 연결된 GitHub 계정이 필요합니다. 위의 브라우저 흐름을 통해 연결하거나 GitHub CLI를 사용하는 경우 터미널에서 `/web-setup`을 실행하세요. GitHub을 전혀 연결하고 싶지 않은 경우 [Remote Control](/docs/en/remote-control)을 참조하여 자체 머신에서 Claude Code를 실행하고 웹에서 모니터링하세요.

### "Not available for the selected organization"

Enterprise 조직의 경우 소유자가 Claude Code on the web을 활성화해야 할 수 있습니다. Anthropic 계정 팀에 문의하세요.

### `/web-setup` 실행 시 "No commands match" 또는 "Unknown command" 표시

`/web-setup`은 쉘이 아니라 Claude Code CLI 내부에서 실행됩니다. 먼저 `claude`를 실행한 후 프롬프트에 `/web-setup`을 입력하세요.

Claude Code 내부에서 입력했으나 명령어 메뉴에 `No commands match "/web-setup"`이 표시되거나 제출할 때 `Unknown command: /web-setup`이 반환되는 경우 요구 사항이 충족되지 않아 명령어가 숨겨진 것입니다. 가장 일반적인 원인은 claude.ai 구독 대신 API 키나 서드파티 제공업체로 인증했기 때문입니다. claude.ai 계정으로 로그인하려면 `/login`을 실행하세요.

Team 및 Enterprise 플랜의 경우 다음 항목 중 하나라도 적용되면 명령어가 숨겨집니다:

* 관리자가 조직에 대해 Claude Code on the web을 비활성화한 경우
* 관리자가 [Quick web setup toggle](/docs/en/claude-code-on-the-web#github-authentication-options)을 비활성화한 경우
* Enterprise 조직에 [Zero Data Retention](/docs/en/zero-data-retention)이 활성화되어 있어 Claude Code on the web을 사용할 수 없는 경우

### `--cloud` 또는 ultraplan 사용 시 "Could not create a cloud environment" 또는 "No cloud environment available"

원격 세션 기능은 클라우드 환경이 없는 경우 기본 클라우드 환경을 자동으로 생성합니다. "Could not create a cloud environment"가 보인다면 자동 생성에 실패한 것입니다. {/* max-version: 2.1.100 */} "No cloud environment available"이 보인다면 CLI가 자동 생성 기능 이전의 버전인 것입니다. 두 경우 모두 Claude Code CLI에서 `/web-setup`을 실행하여 수동으로 환경을 생성하거나 [claude.ai/code](https://claude.ai/code)를 방문하여 위의 **Create your environment** 단계를 따르세요.

### 설정 스크립트 실패

설정 스크립트가 0이 아닌 상태로 종료되어 세션 시작이 차단되었습니다. 일반적인 원인:

* 패키지 레지스트리가 [network access level](/docs/en/claude-code-on-the-web#access-levels)에 포함되어 있지 않아 패키지 설치에 실패함. `Trusted`는 대부분의 패키지 관리자를 커버하며 `None`은 모두 차단함.
* 스크립트가 새로 클론된 환경에 존재하지 않는 파일이나 경로를 참조함.
* 로컬에서 작동하는 명령이 Ubuntu에서는 다른 호출 방식을 필요로 함.

디버깅하려면 스크립트 상단에 `set -x`를 추가하여 실패한 명령을 확인하세요. 중요하지 않은 명령의 경우 세션 시작을 차단하지 않도록 끝에 `|| true`를 덧붙이세요.

### 새 세션이 설정 중에 멈추거나 시간 초과됨

새 세션이 설정 스크립트 단계에서 멈추거나 스크립트가 끝나기 전에 일반적인 컨테이너 오류로 실패하는 경우 스크립트가 [environment cache](/docs/en/claude-code-on-the-web#environment-caching) 빌드를 위한 약 5분의 시간 예산을 초과했을 가능성이 높습니다. 대형 Docker 이미지를 불러오거나, 전체 종속성 트리를 동기화하거나, 모델 가중치를 다운로드하는 것과 같은 무거운 단계는 특히 연달아 실행될 때 전체 시간을 한도 이상으로 늘리곤 합니다.

이를 수정하려면 5분 이내에 안정적으로 완료되도록 스크립트를 다듬으세요:

* 직렬로 실행하는 대신 `&`와 최종 `wait`를 사용하여 독립적인 설치를 병렬로 실행하세요.
* 가장 큰 다운로드를 설정 스크립트에서 백그라운드에서 실행하는 [SessionStart hook](/docs/en/claude-code-on-the-web#setup-scripts-vs-sessionstart-hooks)으로 이동하여 다운로드가 끝나는 동안 세션을 즉시 사용할 수 있도록 하세요.
* 멈춘 재시도 루프가 예산에 합산되므로 설정 스크립트에서 긴 재시도 지연(sleep)을 제거하세요.

### 탭을 닫은 후에도 세션이 계속 실행됨

이는 의도된 설계입니다. 탭을 닫거나 다른 곳으로 이동해도 세션이 중지되지 않습니다. Claude가 현재 작업을 완료할 때까지 백그라운드에서 계속 실행된 후 유휴 상태가 됩니다. 사이드바에서 [archive a session](/docs/en/claude-code-on-the-web#archive-sessions)을 선택하여 목록에서 숨기거나 [delete it](/docs/en/claude-code-on-the-web#delete-sessions)을 선택하여 영구 삭제할 수 있습니다.

## 다음 단계

이제 작업을 제출하고 리뷰할 수 있으므로 다음 페이지에서는 이후 과정에 대해 다룹니다: 터미널에서 클라우드 세션 시작하기, 반복 작업 예약하기, Claude에게 상시 지침 제공하기.

* [Use Claude Code on the web](/docs/en/claude-code-on-the-web): 세션을 터미널로 텔레포트하기, 설정 스크립트, 환경 변수, 네트워크 구성을 포함한 전체 참조
* [Routines](/docs/en/routines): 일정에 따라, API 호출을 통해, 또는 GitHub 이벤트에 대응하여 작업 자동화
* [CLAUDE.md](/docs/en/memory): 매 세션 시작 시 로드되는 영구 지침 및 컨텍스트를 Claude에게 제공
* 휴대전화에서 세션을 모니터링하려면 [iOS](https://apps.apple.com/us/app/claude-by-anthropic/id6473753684) 또는 [Android](https://play.google.com/store/apps/details?id=com.anthropic.claude)용 Claude 모바일 앱을 설치하세요. Claude Code CLI에서 `/mobile`을 실행하면 QR 코드가 표시됩니다.
