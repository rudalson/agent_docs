> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 데스크톱 앱 시작하기

> 데스크톱에 Claude Code를 설치하고 첫 코딩 세션을 시작하세요

데스크톱 앱은 여러 세션을 나란히 실행할 수 있도록 구축된 그래픽 인터페이스와 함께 Claude Code를 제공합니다: 병렬 작업을 관리하는 사이드바, 통합 터미널 및 파일 편집기가 포함된 드래그 앤 드롭 레이아웃, 시각적 diff 검토, 라이브 앱 미리보기, 자동 병합 기능이 있는 GitHub PR 모니터링, 예약된 작업. 터미널이 필요하지 않습니다.

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

Windows ARM64의 경우 [ARM64 설치 프로그램](https://claude.ai/api/desktop/win32/arm64/setup/latest/redirect?utm_source=claude_code\&utm_medium=docs)을 다운로드하세요. Linux에서는 apt로 설치하세요; [Linux에서의 Claude Desktop](/docs/en/desktop-linux)을 참조하세요.

<Note>
  Claude Code에는 [Pro, Max, Team 또는 Enterprise 구독](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=desktop_quickstart_pricing)이 필요합니다.
</Note>

이 페이지에서는 앱 설치와 첫 세션 시작 과정을 단계별로 설명합니다. 이미 설정이 완료되어 있다면 전체 참조 문서인 [Claude Code Desktop 사용법](/docs/en/desktop)을 참조하세요.

데스크톱 앱에는 3개의 탭이 있습니다:

* **Chat**: claude.ai와 유사한 파일 접근 권한이 없는 일반 대화입니다.
* **Cowork**: 사용자가 다른 작업을 수행하는 동안 독립적으로 실행되며, 자체 환경을 가진 샌드박스화된 가상 머신에서 작업에 대해 작동하는 자율 백그라운드 에이전트입니다. 온디바이스 Cowork 세션은 사용자의 컴퓨터에서 VM을 실행하고, 원격 Cowork 세션은 Anthropic 관리형 VM에서 대신 실행됩니다.
* **Code**: 로컬 파일에 직접 접근할 수 있는 대화형 코딩 도우미입니다. 각 변경 사항을 실시간으로 검토하고 승인합니다.

Chat 및 Cowork는 [Claude 도움말 센터](https://support.claude.com/)에서 다루며, 데스크톱 앱의 설치 및 배포는 [Claude Desktop 지원 문서](https://support.claude.com/en/collections/16163169-claude-desktop)에서 다룹니다. 이 페이지는 **Code** 탭에 집중합니다.

## 설치

<Steps>
  <Step title="설치 및 로그인">
    macOS 및 Windows에서는 위의 링크에서 설치 프로그램을 다운로드하여 실행하세요. Linux에서는 [Linux에서의 Claude Desktop](/docs/en/desktop-linux)의 설치 단계를 따르세요. macOS의 Applications 폴더, Windows의 시작 메뉴 또는 Linux의 애플리케이션 실행기에서 Claude를 실행한 다음 Anthropic 계정으로 로그인하세요.
  </Step>

  <Step title="Code 탭 열기">
    상단 중앙의 **Code** 탭을 클릭하세요. Code를 클릭했을 때 업그레이드하라는 메시지가 표시되면 먼저 [유료 플랜에 구독](https://claude.com/pricing?utm_source=claude_code\&utm_medium=docs\&utm_content=desktop_quickstart_upgrade)해야 합니다. 온라인으로 로그인하라는 메시지가 표시되면 로그인을 완료하고 앱을 다시 시작하세요. 403 오류가 발생하면 [인증 문제 해결](/docs/en/desktop#403-or-authentication-errors-in-the-code-tab)을 참조하세요.
  </Step>
</Steps>

데스크톱 앱에는 Claude Code가 포함되어 있습니다. Node.js나 CLI를 별도로 설치할 필요가 없습니다. 터미널에서 `claude`를 사용하려면 CLI를 별도로 설치하세요. [CLI 시작하기](/docs/en/quickstart)를 참조하세요.

## 첫 세션 시작하기

Code 탭이 열린 상태에서 프로젝트를 선택하고 Claude에게 수행할 작업을 전달하세요.

<Steps>
  <Step title="환경 및 폴더 선택">
    파일을 직접 사용하여 머신에서 Claude를 실행하려면 **Local**을 선택하세요. **Select folder**를 클릭하고 프로젝트 디렉터리를 선택하세요.

    <Tip>
      잘 알고 있는 소규모 프로젝트로 시작하세요. Claude Code가 수행할 수 있는 작업을 확인하는 가장 빠른 방법입니다. Windows에서 로컬 세션이 작동하려면 [Git](https://git-scm.com/downloads/win)이 설치되어 있어야 합니다. 대부분의 Mac에는 기본적으로 Git이 포함되어 있습니다.
    </Tip>

    다음 항목도 선택할 수 있습니다:

    * **Cloud**: 앱을 닫아도 계속되는 Anthropic의 클라우드 인프라에서 세션을 실행합니다. 클라우드 세션은 [웹에서의 Claude Code](/docs/en/claude-code-on-the-web)와 동일한 인프라를 사용합니다.
    * **SSH**: 자체 서버, 클라우드 VM 또는 개발 컨테이너와 같이 SSH를 통해 원격 머신에 연결합니다. Desktop은 처음 연결할 때 원격 머신에 Claude Code를 자동으로 설치합니다.
    * **WSL** (Windows): [WSL 2 배포판](/docs/en/desktop-wsl) 내부에서 세션을 실행합니다; Claude Code, 도구 및 git이 네이티브 경로와 함께 Linux 측에서 실행됩니다.
  </Step>

  <Step title="모델 선택">
    전송 버튼 옆의 드롭다운에서 모델을 선택하세요. 사용 가능한 모델 비교는 [모델](/docs/en/model-config#available-models)을 참조하세요. 동일한 드롭다운에서 나중에 모델을 변경할 수 있습니다.
  </Step>

  <Step title="Claude에게 수행할 작업 전달">
    Claude가 수행하길 원하는 작업을 입력하세요:

    * `Find a TODO comment and fix it`
    * `Add tests for the main function`
    * `Create a CLAUDE.md with instructions for this codebase`

    [세션](/docs/en/desktop#work-in-parallel-with-sessions)은 코드에 대한 Claude와의 대화입니다. 각 세션은 자체 컨텍스트와 변경 사항을 추적하므로 서로 방해받지 않고 여러 작업을 처리할 수 있습니다.
  </Step>

  <Step title="변경 사항 검토 및 승인">
    기본적으로 Code 탭은 [Manual 모드](/docs/en/desktop#choose-a-permission-mode)에서 시작하며, Claude가 변경 사항을 제안하고 적용하기 전에 사용자의 승인을 기다립니다. 다음 항목이 표시됩니다:

    1. 각 파일에서 변경될 내용을 정확히 보여주는 [diff 뷰](/docs/en/desktop#review-changes-with-diff-view)
    2. 각 변경 사항을 승인하거나 거부하는 Accept/Reject 버튼
    3. Claude가 요청을 처리함에 따른 실시간 업데이트

    변경 사항을 거부하면 Claude가 다르게 진행하는 방식을 물어봅니다. 승인할 때까지 파일이 수정되지 않습니다.
  </Step>
</Steps>

## 이제 무엇을 할까요?

첫 편집을 완료했습니다. Desktop이 수행할 수 있는 모든 것에 대한 전체 참조 문서는 [Claude Code Desktop 사용법](/docs/en/desktop)을 참조하세요. 다음으로 시도해 볼 만한 몇 가지 항목입니다.

**중단 및 방향 전환.** 언제든지 Claude의 방향을 전환할 수 있습니다. 중지 버튼을 클릭하여 즉시 중단하거나, 수정 사항을 입력하고 **Enter**를 눌러 실행 중인 동작을 멈추지 않고 전송할 수 있습니다. 어느 쪽이든 완료될 때까지 기다리거나 처음부터 다시 시작할 필요가 없습니다.

**Claude에게 더 많은 컨텍스트 제공.** 프롬프트 상자에 `@filename`을 입력하여 특정 파일을 대화로 가져오거나, 첨부 버튼을 사용하여 이미지 및 PDF를 첨부하거나, 프롬프트에 직접 파일을 드래그 앤 드롭하세요. Claude가 더 많은 컨텍스트를 가질수록 결과가 좋아집니다. [파일 및 컨텍스트 추가](/docs/en/desktop#add-files-and-context-to-prompts)를 참조하세요.

**반복 가능한 작업에 스킬 사용.** `/`를 입력하거나 **+** → **Slash commands**를 클릭하여 [내장 명령](/docs/en/commands), [커스텀 스킬](/docs/en/skills) 및 플러그인 스킬을 탐색하세요. 스킬은 코드 검토 체크리스트나 배포 단계와 같이 필요할 때 언제든지 호출할 수 있는 재사용 가능한 프롬프트입니다.

**커밋 전 변경 사항 검토.** Claude가 파일을 편집한 후 `+12 -1` 표시기가 나타납니다. 이를 클릭하여 [diff 뷰](/docs/en/desktop#review-changes-with-diff-view)를 열고, 파일별로 수정 사항을 검토하며, 특정 줄에 주석을 남기세요. Claude가 주석을 읽고 수정합니다. **Review code**를 클릭하면 Claude가 직접 diff를 평가하고 인라인 제안을 남기도록 할 수 있습니다.

**제어 수준 조정.** 사용자의 [권한 모드](/docs/en/desktop#choose-a-permission-mode)는 Claude가 승인 요청 없이 수행할 수 있는 작업량을 설정합니다:

* **Manual**: 기본값. Claude가 파일을 편집하거나 명령을 실행하기 전에 물어봅니다.
* **Accept edits**: 빠른 반복 작업을 위해 Claude가 파일 편집을 자동 승인합니다.
* **Plan**: 파일 편집 없이 접근 방식을 제안하며, 대규모 리팩토링 전에 유용합니다.

**더 많은 기능을 위해 플러그인 추가.** 프롬프트 상자 옆의 **+** 버튼을 클릭하고 **Plugins**를 선택하여 스킬, 에이전트, MCP 서버 등을 추가하는 [플러그인](/docs/en/desktop#install-plugins)을 탐색하고 설치하세요.

**작업 공간 배치.** 채팅, diff, 터미널, 파일 및 브라우저 창을 원하는 레이아웃으로 드래그하세요. **Ctrl+\`**로 터미널을 열어 세션과 함께 명령을 실행하거나 파일 경로를 클릭하여 파일 창에서 여세요. [작업 공간 배치](/docs/en/desktop#arrange-your-workspace)를 참조하세요.

**앱 미리보기.** 데스크톱에서 개발 서버를 실행하면 앱이 Browser 창에서 열리며, [외부 사이도 열 수 있습니다](/docs/en/desktop#browse-external-sites). Claude는 실행 중인 앱을 확인하고, 엔드포인트를 테스트하고, 로그를 검사하고, 보이는 내용을 바탕으로 반복 조치할 수 있습니다. [앱 미리보기](/docs/en/desktop#preview-your-app)를 참조하세요.

**풀 리퀘스트 추적.** PR을 연 후 Claude Code는 CI 체크 결과를 모니터링하며 실패를 자동으로 수정하거나 모든 체크가 통과하면 PR을 병합할 수 있습니다. [풀 리퀘스트 상태 모니터링](/docs/en/desktop#monitor-pull-request-status)을 참조하세요.

**Claude를 일정에 따라 실행.** 매일 아침 일일 코드 검토, 주간 종속성 감사 또는 연결된 도구에서 정보를 가져오는 브리핑 등 정기적으로 Claude를 자동 실행하도록 [예약된 작업](/docs/en/desktop-scheduled-tasks)을 설정하세요.

**준비되면 규모 확장.** 사이드바에서 [병렬 세션](/docs/en/desktop#work-in-parallel-with-sessions)을 열어 각자의 Git 워크트리에서 한 번에 여러 작업을 처리하고, [tasks 창](/docs/en/desktop#watch-background-tasks)을 열어 세션이 실행 중인 서브에이전트 및 백그라운드 명령을 관찰하세요. 메인 스레드를 이탈하지 않고 질문하려면 [사이드 챗](/docs/en/desktop#ask-a-side-question-without-derailing-the-session)을 여세요. 앱을 닫아도 계속되도록 [장시간 실행 작업을 클라우드로 전달](/docs/en/desktop#run-long-running-tasks-remotely)하거나, 작업이 예상보다 오래 걸리는 경우 [웹 또는 IDE에서 세션을 계속하기](/docs/en/desktop#continue-in-another-surface)를 이용하세요. GitHub, Slack, Linear와 같은 [외부 도구를 연결](/docs/en/desktop#extend-claude-code)하여 워크플로를 하나로 모으세요.

## CLI에서 오셨나요?

Desktop은 그래픽 인터페이스와 함께 CLI와 동일한 엔진을 실행합니다. 동일한 프로젝트에서 두 환경을 동시에 실행할 수 있으며 구성(CLAUDE.md 파일, MCP 서버, 훅, 스킬 및 설정)을 공유합니다. 전체 기능 비교, 플래그 대응표 및 Desktop에서 사용할 수 없는 사항은 [CLI 비교](/docs/en/desktop#coming-from-the-cli)를 참조하세요.

## 다음 단계

* [Claude Code Desktop 사용법](/docs/en/desktop): 권한 모드, 병렬 세션, diff 뷰, 커넥터 및 엔터프라이즈 구성
* [문제 해결](/docs/en/desktop#troubleshooting): 일반적인 오류 및 설정 문제에 대한 해결 방법
* [모범 사례](/docs/en/best-practices): 효과적인 프롬프트 작성 및 Claude Code 활용 팁
* [일반적인 워크플로](/docs/en/common-workflows): 디버깅, 리팩토링, 테스트 등에 대한 튜토리얼
