---
slug: cli-conversations
section: Antigravity CLI
title: 대화 관리
path:
  - Antigravity CLI
  - 대화
---




# 대화 관리

이전 개발 대화 스레드를 재개하고, 활성화된 기록의 범위를 로컬 작업 공간으로 제한하며, 대화를 분기(fork)하여 다른 아키텍처를 실험해 보세요.

## 작업 공간 범위 제한 

컨텍스트를 깔끔하게 유지하기 위해, Antigravity CLI는 대화 기록의 범위를 현재 작업 디렉터리로 직접 제한합니다. 특정 디렉터리에서 `agy`를 실행하면 에이전트는 해당 로컬 저장소 또는 하위 디렉터리와 관련된 세션만 표시하고 재개합니다.

이렇게 함으로써 컨텍스트 오염을 방지하고, 에이전트의 시맨틱 메모리와 토큰 제한이 오직 관련 코드베이스에만 집중되도록 보장합니다.

## 세션 재개

이전 대화로 돌아가 구현을 계속하거나 기존 솔루션을 보완할 수 있습니다.

### TUI 세션 선택기를 통한 재개

활성화된 터미널 화면 내에서 이전 대화를 검색하고 로드하려면 다음과 같이 하십시오.

1. 프롬프트 창에 `/resume`를 입력하고 `Enter`를 누릅니다.
2. 대화형 대화 선택기(Conversation Picker) 오버레이가 열립니다.
3. 키워드를 입력하여 설명이나 ID로 대화를 필터링합니다.
4. `↑`/`↓`를 사용하여 목록을 탐색하고, `←`/`→`를 사용하여 이전 레코드로 페이지를 넘깁니다.
5. `Enter`를 눌러 선택한 대화를 재개합니다. 취소하고 프롬프트로 돌아가려면 `Esc`를 누릅니다.

```text
   CLI    Antigravity   (tab 키로 탭 전환)

  대화 목록
  검색어를 입력하세요...
> implement-auth-pipeline                                      4단계     3시간 전
  refactor-db-connection-pool                                  7단계     5시간 전
  add-unit-tests-for-parser                                    2단계     1일 전
  fix-socket-connection-timeout-error                         14단계     2일 전
  update-project-dependencies                                  5단계     3일 전
  optimize-image-compression-algorithm                         9단계     4일 전
  draft-release-notes-v2.1.0                                   3단계     5월 23일
  clean-up-obsolete-cache-files                                6단계     5월 22일
  integrate-payment-gateway-sdk                               19단계     5월 20일
  fix-layout-alignment-in-navbar                              8단계     5월 19일
  [75개 항목 중 1-10]

키보드: ↑/↓ 이동  enter 선택  f2 이름 변경  tab 전환  esc 완료
```

### Antigravity 2.0에서 대화 가져오기

Antigravity CLI 공개 버전을 사용하는 경우, Antigravity 2.0 데스크톱 비주얼 에디터에서 시작된 활성 스레드를 가져와 재개할 수 있습니다.

1. 프롬프트 패널에 `/resume`를 입력하고 `Enter`를 눌러 선택기를 엽니다.
2. `Tab`을 눌러 **CLI** 탭(로컬 TUI 대화)과 **Antigravity** 탭(Antigravity 2.0 데스크톱 대화) 간에 탭을 전환합니다.
3. `↑`/`↓`를 사용하여 가져올 데스크톱 대화를 하이라이트하고 `Enter`를 누릅니다.
4. TUI에 확인 프롬프트가 표시됩니다. `Enter`(또는 `y`)를 눌러 가져오기를 확인합니다.
5. CLI가 데스크톱 대화 기록, 컨텍스트 및 궤적(trajectories)을 터미널 세션으로 복제하여 워크플로를 원활하게 이어갈 수 있도록 합니다.

```text
   CLI    Antigravity   (tab 키로 탭 전환)

  대화 목록
  검색어를 입력하세요...
  a1b2c3d4-e5f6-7890-abcd-ef1234567890                       로딩 중…   5월 23일
  f9e8d7c6-b5a4-3210-fedc-ba9876543210                       로딩 중…   5월 16일
> Design New Analytics Dashboard Layout [가져오시겠습니까? (y/n)] 12단계   4월 30일
  Implement Realtime Graph Plotter                           15단계   4월 30일
  fix-visual-flashes-on-load                                  2단계   4월 30일
  add-collapsible-menu-sidebar                                4단계   4월 29일
  refactor-utility-helper-methods                             6단계   4월 28일
  Verify Webpack Configuration Output                         5단계   4월 28일
  [13개 항목 중 1-10]

키보드: ↑/↓ 이동  enter 선택  tab 탭 전환  esc 뒤로 가기 / 검색어 비우기
```

### 명령줄을 통한 빠른 재개

선택기를 거치지 않고 활성 작업 공간에서 가장 최근 세션을 즉시 재개하려면 `--continue` 플래그를 사용하여 실행 파일을 실행합니다.

```bash
agy --continue
```

셸에서 특정 세션을 직접 로드하려면 대상 UUID를 전달합니다.

```bash
agy --conversation 9a8b7c6d-5e4f-3a2b-1c0d-ef1234567890
```

## `/fork`를 이용한 분기 

복잡한 기능을 구현할 때, 진행 상황을 잃지 않으면서 여러 가지 설계 대안을 탐색하고 싶을 수 있습니다. `/fork` 명령어는 안전하고 병렬적인 실험을 가능하게 합니다.

```text
/fork
```

*(별칭: `/branch`)*

`/fork` 명령어는 현재 턴까지의 전체 대화 기록을 새로운 독립 세션으로 복제합니다.

### 분기 워크플로 

1. 프롬프트 패널에 `/fork`를 입력하고 `Enter`를 누릅니다.
2. CLI가 새로운 고유 세션 ID를 할당하고 기존 작업 공간 상태와 에이전트 스레드를 복제합니다.
3. 활성화된 터미널이 새 분기(branch)로 즉시 전환됩니다.
4. 실험이 실패한 경우, `/resume`를 실행하여 안정적인 원래의 대화 분기로 되돌아갈 수 있습니다.

<Announcement>
icon: lightbulb
iconColor: var(--theme-primary)
color: var(--theme-surface-surface-container)
text: **파일 시스템 분기**: 포크(Forking)는 로컬 git 체크아웃이 아닌 *대화 스레드*를 복제합니다. 병렬 포크 과정에서 파일을 완전히 격리하려면, 상반된 접근 방식을 테스트하기 전에 git 브랜치를 사용하거나 로컬 변경 사항을 스태시(stash)하세요.
</Announcement>

## Next Steps

에이전트가 복잡한 비동기 작업 및 병렬 태스크를 처리하는 방법을 탐색해 보세요.

- **[백그라운드 태스크 및 서브에이전트](/docs/cli-subagents)**: 서브에이전트를 모니터링하고 빠른 승인을 처리합니다.
- **[설정, 렌더링 및 단축키](/docs/cli-settings)**: 렌더링 버퍼를 구성하고 JSON 설정을 재정의합니다.
- **[권한 및 샌드박스](/docs/cli-sandbox)**: 보안 프로필 및 시스템 명령어 목록을 관리합니다.
