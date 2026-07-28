> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 새로운 소식

> 주간 단위로 정리된 주목할 만한 Claude Code 기능 요약으로, 코드 스니펫, 데모 및 이것이 왜 중요한지에 대한 컨텍스트를 제공합니다.

주간 개발 요약(weekly dev digest)은 여러분의 작업 방식을 바꿀 가능성이 가장 높은 기능들을 조명합니다. 각 항목에는 실행 가능한 코드, 짧은 데모 및 전체 문서로 연결되는 링크가 포함되어 있습니다. 모든 버그 수정 및 소규모 개선 사항은 [변경 로그](/docs/en/changelog)를 참조하세요.

<Update label="Week 29" description="2026년 7월 13일–17일" tags={["v2.1.207–v2.1.212"]}>
  **아티팩트에서 MCP 커넥터 호출**: 게시된 아티팩트는 조회자가 페이지를 열 때 본인 계정의 MCP 커넥터를 통해 라이브 데이터를 가져오고 작업을 수행할 수 있으며, 이번 주에는 공개 공유 링크, Team 및 Enterprise에서의 편집자 역할, Claude Tag 세션에서 생성된 아티팩트도 추가되었습니다.

  이번 주 추가 사항: **스크린 리더 모드**는 VoiceOver 및 NVDA와 같은 스크린 리더를 위해 시각적 터미널 인터페이스를 플레인 리니어 텍스트로 대체합니다. **`/fork`**는 계속 작업하는 동안 대화를 새 백그라운드 세션으로 복사합니다. **자동 모드**는 Amazon Bedrock, Google Cloud's Agent Platform 및 Microsoft Foundry에서 더 이상 옵트인 변수가 필요하지 않습니다.

  [29주차 요약 읽기 →](/docs/en/whats-new/2026-w29)
</Update>

<Update label="Week 28" description="2026년 7월 6일–10일" tags={["v2.1.202–v2.1.206"]}>
  **Desktop의 인앱 브라우저**: 데스크톱용 Claude Code에 내장 브라우저가 제공되므로 Claude가 문서, 디자인 또는 기타 모든 사이트를 가동하고 로컬 개발 서버 미리 보기에서와 동일하게 페이지와 상호 작용할 수 있습니다.

  이번 주 추가 사항: **`/doctor`**는 문제를 진단하고 수정할 수 있는 전체 설정 점검으로, `/checkup`을 별칭으로 사용합니다. **자동 모드**는 트랜스크립트 변조를 차단하고 미확인 변수에 대한 `rm -rf` 실행 전에 묻습니다. **에이전트 뷰 행**은 색상이 지정된 상태 단어와 분류기가 작성한 헤드라인을 표시합니다.

  [28주차 요약 읽기 →](/docs/en/whats-new/2026-w28)
</Update>

<Update label="Week 27" description="2026년 6월 29일 – 7월 3일" tags={["v2.1.195–v2.1.201"]}>
  **Claude Sonnet 5**: Pro, Team Standard 및 Enterprise 구독 시트의 새로운 기본 모델로, Sonnet 가격에 최고 수준의 코딩 및 도구 사용, 기본 1M 토큰 컨텍스트 윈도우, 기본 활성화된 자율적 사고(adaptive thinking)를 제공합니다.

  이번 주 추가 사항: **Chrome의 Claude**가 모든 Anthropic 직접 플랜에서 정식 출시되었습니다. **서브에이전트 기본 백그라운드 실행**으로 실행 중에도 Claude가 작업을 계속합니다. **Linux용 Claude Desktop**이 Ubuntu 및 Debian에서 베타 버전으로 출시되었습니다. **`/radio`**는 Claude FM lo-fi 라디오 방송을 청취합니다.

  [27주차 요약 읽기 →](/docs/en/whats-new/2026-w27)
</Update>

<Update label="Week 26" description="2026년 6월 22일–26일" tags={["v2.1.185–v2.1.193"]}>
  **`claude mcp login`**: 대화형 `/mcp` 메뉴 대신 쉘에서 구성된 MCP 서버를 인증하고, 나중에 `claude mcp logout`으로 저장된 자격 증명을 삭제합니다.

  이번 주 추가 사항: **명령 출력에 응답하는 쉘 모드**(`! npm test` 실행 시 두 번째 프롬프트 없이 설명 제공). **`/rewind`**는 `/clear` 실행 이전 대화를 재개할 수 있습니다. **백그라운드 서브에이전트**는 이제 자동 거부 대신 메인 세션에 권한 프롬프트를 표시합니다.

  [26주차 요약 읽기 →](/docs/en/whats-new/2026-w26)
</Update>

<Update label="Week 25" description="2026년 6월 15일–19일" tags={["v2.1.178–v2.1.183"]}>
  **아티팩트**: 세션 출력을 claude.ai의 라이브 공유 가능 페이지로 전환하여 세션이 작업함에 따라 그 자리에서 업데이트되며, 이제 Team 및 Enterprise 플랜에서 베타 버전으로 제공됩니다.

  이번 주 추가 사항: **거부 및 요청 규칙에서 도구 매개변수 일치**(`Tool(param:value)` 구문 사용, 예: `Agent(model:opus)`). **`/config key=value`**는 프롬프트, `-p` 모드 및 Remote Control에서 설정을 조정합니다. **자동 모드**는 로컬 작업을 삭제하라고 요청하지 않은 경우 파괴적인 git 명령을 차단합니다.

  [25주차 요약 읽기 →](/docs/en/whats-new/2026-w25)
</Update>

<Update label="Week 24" description="2026년 6월 8일–12일" tags={["v2.1.166–v2.1.176"]}>
  **`/cd`**: 대화 도중에 프롬프트 캐시를 재구축하지 않고 현재 세션을 새 작업 디렉토리로 이동합니다.

  이번 주 추가 사항: **서브에이전트의 자체 서브에이전트 생성**(백그라운드 체인은 최대 5단계 깊이로 제한됨). **`--safe-mode`**는 문제 해결을 위해 모든 커스터마이징을 비활성화한 상태로 Claude Code를 시작합니다. **`fallbackModel`**은 순서대로 시도할 수 있는 대체 모델을 최대 3개까지 구성합니다.

  [24주차 요약 읽기 →](/docs/en/whats-new/2026-w24)
</Update>

<Update label="Week 23" description="2026년 6월 1일–5일" tags={["v2.1.158–v2.1.165"]}>
  **Amazon Bedrock, Google Cloud's Agent Platform 및 Microsoft Foundry에서의 자동 모드**: 써드파티 제공업체에서 Opus 4.7 및 Opus 4.8에 자동 모드가 제공되어 권한 프롬프트를 백그라운드 안전성 검사로 대체합니다.

  이번 주 추가 사항: **더 안전한 자동 편집**으로 `acceptEdits` 모드에서 코드를 실행할 수 있는 파일을 쓰기 전에 프롬프트를 표시합니다. **`/plugin list`**는 설치된 플러그인을 인라인으로 출력합니다. **버전 요구 사항**을 통해 관리형 배포에서 승인된 Claude Code 버전 범위를 요구할 수 있습니다.

  [23주차 요약 읽기 →](/docs/en/whats-new/2026-w23)
</Update>

<Update label="Week 22" description="2026년 5월 25일–29일" tags={["v2.1.150–v2.1.157"]}>
  **Claude Opus 4.8**: Max, Team Premium, Enterprise 종량제 및 Anthropic API 계정의 새로운 기본 모델로, 기본 high 노력 수준과 가장 어려운 작업을 위한 `/effort xhigh`를 지원합니다.

  이번 주 추가 사항: **동적 워크플로우**는 Claude가 작성한 스크립트에서 수십~수백 개의 서브에이전트를 조율합니다. **security-guidance 플러그인**은 작업 중 Claude의 변경 사항에 대해 취약점을 검토합니다. **패스트 모드**는 Opus 4.8에서 MTok당 \$10/\$50으로 실행됩니다.

  [22주차 요약 읽기 →](/docs/en/whats-new/2026-w22)
</Update>

<Update label="Week 21" description="2026년 5월 18일–22일" tags={["v2.1.143–v2.1.149"]}>
  **Pro 플랜의 자동 모드**: 자동 모드가 이제 Pro 계정에서 실행되고 Opus와 함께 Sonnet 4.6을 지원하여 권한 프롬프트를 백그라운드 안전성 검사로 대체합니다.

  이번 주 추가 사항: **`/usage`**는 스킬, 서브에이전트, 플러그인 및 MCP 서버별로 플랜 한도를 이끄는 요소를 보여줍니다. 새로운 **`/code-review`** 명령은 정성적 버그를 보고합니다. **백그라운드 세션**은 `/resume`에 나타나며 고정 시 유지됩니다.

  [21주차 요약 읽기 →](/docs/en/whats-new/2026-w21)
</Update>

<Update label="Week 20" description="2026년 5월 11일–15일" tags={["v2.1.139–v2.1.142"]}>
  **에이전트 뷰**: `claude agents`는 모든 Claude Code 세션에 대한 단일 화면을 열어 실행 중인 항목, 차단된 항목, 완료된 항목을 보여줍니다.

  이번 주 추가 사항: **`/goal`**은 완료 조건이 충족될 때까지 턴에 걸쳐 Claude가 작동하도록 합니다. **패스트 모드**는 이제 Opus 4.7에서 기본 실행됩니다. **Rewind 메뉴**는 "Summarize up to here"로 이전 컨텍스트를 압축할 수 있습니다.

  [20주차 요약 읽기 →](/docs/en/whats-new/2026-w20)
</Update>

<Update label="Week 19" description="2026년 5월 4일–8일" tags={["v2.1.128–v2.1.136"]}>
  **`.zip` 아카이브 및 URL에서 플러그인 로드**: `--plugin-dir`은 이제 `.zip` 파일을 지원하며 `--plugin-url`은 현재 세션을 위한 플러그인 아카이브를 가져옵니다.

  이번 주 추가 사항: **`worktree.baseRef`**는 새 워크트리가 원격 기본 브랜치 또는 로컬 `HEAD`에서 분기할지 선택합니다. **자동 모드 하드 거부 규칙**은 허용 예외와 상관없이 무조건 작업을 차단합니다. **훅은 활성 노력 수준을 참조**합니다(`effort.level` 및 `$CLAUDE_EFFORT` 통해).

  [19주차 요약 읽기 →](/docs/en/whats-new/2026-w19)
</Update>

<Update label="Week 18" description="2026년 4월 27일 – 5월 1일" tags={["v2.1.120–v2.1.126"]}>
  **Git Bash 없는 Windows**: Windows용 Git이 더 이상 필요하지 않으며, Bash가 없을 때 Claude Code가 PowerShell을 쉘 도구로 사용합니다.

  이번 주 추가 사항: **`claude ultrareview`**는 CI 및 스크립트에 클라우드 코드 리뷰를 제공합니다. **`claude project purge`**는 프로젝트의 로컬 상태를 정리합니다. **`/resume`에 PR URL 붙여넣기**로 해당 PR을 작성한 세션을 찾습니다.

  [18주차 요약 읽기 →](/docs/en/whats-new/2026-w18)
</Update>

<Update label="Week 17" description="2026년 4월 20일–24일" tags={["v2.1.114–v2.1.119"]}>
  **`/ultrareview`**가 공개 리서치 프리뷰로 공개됩니다: 클라우드에서 버그 탐색 에이전트 군집이 실행되고 탐색 결과가 CLI 또는 Desktop으로 자동 전달됩니다.

  이번 주 추가 사항: **세션 요약**은 터미널이 비포커스 상태였던 동안 발생한 일을 보여줍니다. **커스텀 테마**를 사용하여 `/theme` 또는 플러그인에서 색상 팔레트를 빌드하고 제공할 수 있습니다. **웹상 Claude Code**가 새 세션 사이드바 및 드래그 앤 드롭 레이아웃으로 새로워졌습니다.

  [17주차 요약 읽기 →](/docs/en/whats-new/2026-w17)
</Update>

<Update label="Week 16" description="2026년 4월 13일–17일" tags={["v2.1.105–v2.1.113"]}>
  **Claude Opus 4.7**이 Max 및 Team Premium의 새로운 기본 모델로 제공되며, 대부분의 코딩 작업에 권장되는 새로운 `xhigh` 노력 수준과 조정을 위한 대화형 `/effort` 슬라이더를 포함합니다.

  이번 주 추가 사항: 웹상 Claude Code의 **루틴(Routines)**이 스케줄, GitHub 이벤트 또는 API 호출에서 템플릿 클라우드 에이전트를 실행합니다. **모바일 푸시 알림**은 긴 작업이 끝나거나 Claude가 필요한 경우 전화로 핑을 보냅니다. `/usage`는 사용 한도를 이끄는 요소를 보여주며 CLI는 네이티브 바이너리로 이동합니다.

  [16주차 요약 읽기 →](/docs/en/whats-new/2026-w16)
</Update>

<Update label="Week 15" description="2026년 4월 6일–10일" tags={["v2.1.92–v2.1.101"]}>
  **Ultraplan**이 얼리 프리뷰로 진입합니다: CLI에서 클라우드의 플랜 초안을 작성하고, 웹 에디터에서 리뷰 및 댓글을 작성한 다음, 원격으로 실행하거나 로컬로 다시 가져옵니다. 첫 번째 실행은 이제 자동으로 클라우드 환경을 생성합니다.

  이번 주 추가 사항: **Monitor** 도구는 백그라운드 이벤트를 대화로 스트리밍하여 Claude가 로그를 추적하고 실시간으로 반응할 수 있습니다. `/loop`는 간격을 생략할 때 자체적으로 속도를 조절합니다. `/team-onboarding`은 설정을 재생 가능한 가이드로 패키징합니다. `/autofix-pr`은 터미널에서 PR 자동 수정을 켭니다.

  [15주차 요약 읽기 →](/docs/en/whats-new/2026-w15)
</Update>

<Update label="Week 14" description="2026년 3월 30일 – 4월 3일" tags={["v2.1.86–v2.1.91"]}>
  **Computer use**가 리서치 프리뷰로 CLI에 제공됩니다: Claude가 터미널에서 기본 앱을 열고 UI를 클릭하며 변경 사항을 확인할 수 있습니다. GUI로만 확인 가능한 작업을 완결 짓는 데 가장 적합합니다.

  이번 주 추가 사항: `/powerup` 대화형 레슨, 깜빡임 없는 대체 화면 렌더링, 최대 500K까지의 도구별 MCP 결과 크기 재정의, Bash 도구 `PATH`상의 플러그인 실행 파일.

  [14주차 요약 읽기 →](/docs/en/whats-new/2026-w14)
</Update>

<Update label="Week 13" description="2026년 3월 23일–27일" tags={["v2.1.83–v2.1.85"]}>
  **자동 모드**가 리서치 프리뷰에 도달했습니다: 분류기가 권한 프롬프트를 처리하여 안전한 작업은 방해 없이 실행되고 위험한 작업은 차단됩니다. 모두 승인하는 것과 `--dangerously-skip-permissions` 사이의 절충안입니다.

  이번 주 추가 사항: Desktop 앱에서의 computer use, 웹에서의 PR 자동 수정, `/`를 통한 트랜스크립트 검색, Windows용 네이티브 PowerShell 도구, 조건부 `if` 훅.

  [13주차 요약 읽기 →](/docs/en/whats-new/2026-w13)
</Update>
