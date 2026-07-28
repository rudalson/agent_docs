> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# 스크린 리더로 Claude Code 사용하기

> VoiceOver 및 NVDA와 같은 스크린 리더를 위한 Claude Code 설정과 화면 확대기, 동작 줄이기, 색맹 지원 테마 설정을 안내합니다.

Claude Code에는 시각적 터미널 인터페이스를 일반 선형 텍스트로 대체하는 스크린 리더 모드가 있습니다. 상자, 진행 상태 애니메이션, 인플레이스 재그리기(in-place redraws) 대신, VoiceOver나 NVDA 같은 스크린 리더가 순서대로 읽을 수 있도록 레이블이 지정된 줄을 출력하므로, 전체 대화를 진행하고 도구 권한을 승인하며 출력을 처음부터 끝까지 검토할 수 있습니다.

스크린 리더 모드는 옵트인(opt-in) 방식입니다. 스크린 리더 대신 화면 확대기, 동작 줄이기 또는 색맹 지원 테마를 사용하는 경우 [스크린 리더 모드 이외의 접근성 설정](#스크린-리더-모드-이외의-접근성-설정)을 참조하세요.

<Note>
  스크린 리더 모드를 사용하려면 Claude Code v2.1.181 이상이 필요합니다. 이전 버전에서는 `--ax-screen-reader` 플래그를 제공하면 `error: unknown option '--ax-screen-reader'` 오류가 발생합니다.
</Note>

## 스크린 리더 모드 켜기

스크린 리더를 얼마나 자주 사용하는지에 따라 적절한 방법을 선택하세요:

* 단일 세션 동안 사용: `claude --ax-screen-reader`를 실행합니다.
* 특정 쉘에서 시작된 세션 동안 사용: `CLAUDE_AX_SCREEN_READER` 환경 변수를 `1`로 설정합니다. Bash 또는 Zsh에서는 `export CLAUDE_AX_SCREEN_READER=1`을 실행하고, PowerShell에서는 `$env:CLAUDE_AX_SCREEN_READER = "1"`을 실행합니다. 모든 쉘에 적용하려면 쉘 프로필 파일에 해당 줄을 추가하세요.
* 머신의 모든 세션 동안 사용: 사용자 [설정 파일](/docs/en/settings)에 `"axScreenReader": true`를 추가합니다. 이렇게 하면 VS Code 통합 터미널을 포함한 모든 터미널에 적용됩니다.

<Note>
  위의 방법들은 우선순위 순으로 나열되어 있습니다: [`--ax-screen-reader`](/docs/en/cli-reference#cli-flags) 플래그는 [`CLAUDE_AX_SCREEN_READER`](/docs/en/env-vars) 환경 변수를 재정의하고, 환경 변수는 [`axScreenReader`](/docs/en/settings#available-settings) 설정을 재정의합니다.
</Note>

SSH를 통해 Claude Code를 사용하는 경우, Claude Code가 실행되는 원격 머신에서 환경 변수 또는 설정을 지정하세요.

모드가 켜지면 Claude Code가 가장 먼저 출력하는 것은 모드를 켠 방법을 명시하는 확인 줄입니다: `[Screen Reader Mode: on via flag]`, `[Screen Reader Mode: on via env]`, 또는 `[Screen Reader Mode: on via settings]`. 이러한 방식 명시 형식은 Claude Code v2.1.206 이상이 필요합니다. 업데이트 설치 완료 등을 위해 Claude Code가 스스로 재실행될 때, 새 프로세스는 `CLAUDE_AX_SCREEN_READER` 환경 변수를 통해 모드를 상속받으므로 어떤 방식을 사용했든 확인 줄에 `[Screen Reader Mode: on via env]`라고 표시됩니다.
{/* max-version: 2.1.205 */}이전 버전에서는 `[Accessible screen reader mode: on]`으로 출력됩니다.

## 스크린 리더 모드 끄기

모드를 켰던 방법을 역으로 적용하세요: 플래그 없이 시작하거나, 환경 변수를 해제하거나, `axScreenReader`를 `false`로 설정합니다. `CLAUDE_AX_SCREEN_READER=0`으로 설정하면 설정 파일의 값이 `true`이더라도 모드가 꺼진 상태로 유지됩니다.

## 스크린 리더로 들리는 내용

스크린 리더 모드에서 Claude Code는 평면(flat) 텍스트를 작성합니다:

* 인터페이스 테두리를 위한 상자 그리기 문자가 없음
* 색상으로만 전달되는 신호가 없음
* 변경되지 않은 콘텐츠의 재그리기가 없음. 진행 스피너는 정적 텍스트로 렌더링됨
* Claude 응답의 테이블은 상자 문자 그리드 대신 `Header: value` 문장으로 읽힘. {/* min-version: 2.1.198 */}Claude Code v2.1.198 이상이 필요하며, 이전 버전에서는 스크린 리더 모드에서도 테이블을 그리드로 그립니다.

출력은 터미널의 스크롤백에 누적되므로 스크린 리더의 검토 명령어나 터미널 검색 기능을 사용하여 이전 대화 내용을 다시 읽을 수 있습니다.

[`tui` 설정](/docs/en/settings#available-settings)으로 [전체 화면 렌더링](/docs/en/fullscreen)을 켰더라도 스크린 리더 모드는 일반 스크롤 텍스트로 렌더링되며, 모드가 활성화되어 있는 동안 해당 설정은 적용되지 않습니다. 연결된 백그라운드 세션은 여전히 전체 화면으로 렌더링됩니다. [알려진 제한 사항](#알려진-제한-사항)을 참조하세요.

트랜스크립트의 각 메시지는 스크린 리더가 읽어주는 레이블로 시작되어 사용자의 메시지, Claude의 응답, 도구 활동, 오류, 프롬프트 등 그것이 무엇인지 알려줍니다. 이 레이블은 검색도 가능하므로 터미널의 스크롤백에서 검색하여 트랜스크립트의 섹션 간을 이동할 수 있습니다:

| 레이블 | 의미 |
| :--------------------- | :---------------------------------------------------------------------------------------- |
| `you:` | 사용자의 메시지 |
| `claude:` | Claude의 응답 |
| `tool:` | 파일 편집이나 명령 실행과 같은 도구 활동 |
| `tool error:` | 실패한 도구 |
| `error:` | 실패한 API 요청 등 대화 중 발생한 오류 |
| `Permission Required:` | 답변을 기다리는 권한 요청 프롬프트 |
| `Cost:` | Claude Code 종료 시 세션 비용 요약(계정에 [비용 표시](/docs/en/costs)가 설정된 경우) |

터미널 커서가 입력 캐럿을 따르므로 스크린 리더의 현재 줄 읽기 명령을 사용하면 편집 중인 프롬프트로 "현재 위치"를 확인할 수 있습니다.

{/* min-version: 2.1.218 */}입력창에서 단어나 줄을 삭제할 때 Claude Code가 삭제된 텍스트를 알립니다. Claude Code v2.1.218 이상이 필요합니다. 안내 대상:

* macOS에서 `Ctrl+W`, `Option+Delete`, Windows에서 `Ctrl+Backspace`로 단어 삭제
* `Ctrl+U` 또는 `Cmd+Backspace`로 줄 시작까지 삭제
* `Ctrl+K`로 줄 끝까지 삭제

각 키의 기능은 [텍스트 편집 단축키](/docs/en/interactive-mode#text-editing)를 참조하세요.

{/* min-version: 2.1.210 */}`Shift+Tab`으로 [권한 모드](/docs/en/permission-modes)를 전환하면 `[plan mode on]` 또는 `[accept edits on]`과 같이 전환된 모드가 안내됩니다. Claude Code는 안내를 한 번만 출력하고 이후 재그리기 시에는 반복하지 않습니다. Claude Code v2.1.210 이상이 필요합니다.

### 대화 턴(turn) 간 이동

Claude Code는 턴 경계에 OSC 133 쉘 통합 마커를 생성하므로 터미널의 이전 프롬프트로 이동하는 키를 사용하면 전체 트랜스크립트를 읽지 않고도 턴 간을 이동할 수 있습니다:

* iTerm2: Cmd+Shift+Up
* VS Code 터미널: Windows에서 Ctrl+Up, macOS에서 Cmd+Up
* Windows Terminal: 기본 키 없음. 설정에서 `scrollToMark` 동작 바인딩
* Kitty 및 Ghostty: 터미널의 프롬프트 이동 키에 대한 설명서 확인

macOS Terminal은 이 마커에 반응하지 않으며, WezTerm에서는 Claude Code가 마커를 생성하지 않습니다. 이러한 터미널에서는 스크롤백에서 `you:` 레이블을 검색하세요.

## 메뉴 및 프롬프트 응답하기

스크린 리더 모드에서는 권한 프롬프트를 포함하여 normalmente 화살표 키로 탐색하던 메뉴가 번호가 매겨진 목록이 됩니다. 각 옵션은 번호가 붙은 줄로 안내되며, 이어서 유효한 범위가 명시된 `Enter selection` 프롬프트가 나타납니다. 원하는 옵션의 번호를 입력하고 Enter를 누르세요.

* 취소 가능한 메뉴를 취소하려면: Escape를 누르세요. 해당 프롬프트는 `or Escape to cancel`로 끝납니다.
* 목록에 없는 번호를 입력한 경우: Claude Code가 유효한 범위를 안내하고 다시 시도할 수 있게 합니다.

Yes/No 프롬프트는 두 개의 옵션 메뉴 대신 직접 입력하는 답변을 요청합니다. `y` 또는 `n`을 입력하고 Enter를 누르세요. `yes`와 `no`도 작동합니다.

## Claude Code의 확인 요청 알림 듣기

스크린 리더 모드에서 Claude Code는 사용자의 주의가 필요할 때 터미널 벨(bell)을 울리므로 트랜스크립트를 계속 확인할 필요가 없습니다. 벨은 다음 상황에서 울립니다:

* Claude가 응답을 완료했을 때
* 권한 프롬프트가 나타났을 때
* 5초 이상 실행된 도구가 완료되었을 때

벨은 터미널의 표준 알림음입니다. 알림음을 끄려면 터미널 애플리케이션의 벨 설정을 변경하세요. 벨을 사용하기 위해 스크린 리더 모드가 필수인 것은 아닙니다. 모드 외부에서도 Claude가 대기 중일 때 유사한 알림을 받으려면 [`preferredNotifChannel`](/docs/en/settings#available-settings)을 `"terminal_bell"`로 설정하세요. [터미널 벨 또는 알림 설정하기](/docs/en/terminal-config#get-a-terminal-bell-or-notification)를 참조하세요.

## 스크린 리더 모드 이외의 접근성 설정

이 옵션들은 스크린 리더 모드 외의 접근성 니즈를 다룹니다. 모든 옵션은 스크린 리더 모드와 함께 작동합니다.

* `CLAUDE_CODE_ACCESSIBILITY` [환경 변수](/docs/en/env-vars)는 화면 확대기를 위한 설정입니다. macOS Zoom과 같은 화면 확대기가 커서 위치를 추적할 수 있도록 네이티브 터미널 커서를 표시 상태로 유지하려면 `CLAUDE_CODE_ACCESSIBILITY=1`로 설정하세요. 커서는 키보드 포커스를 따릅니다: 텍스트를 입력할 때는 입력 캐럿을 따르고, `/config` 및 `/plugin`과 같은 메뉴 및 패널에서 화살표 키로 이동할 때는 강조 표시된 행을 따릅니다. {/* min-version: 2.1.218 */}메뉴 및 패널의 행 추적에는 Claude Code v2.1.218 이상이 필요합니다.
* `prefersReducedMotion` [설정](/docs/en/settings#available-settings)은 인터페이스의 다른 부분을 변경하지 않고 스피너, 쉬머(shimmer) 및 기타 애니메이션을 줄이거나 비활성화합니다.
* `theme` [설정](/docs/en/settings#available-settings)은 색맹 지원 테마인 `dark-daltonized` 및 `light-daltonized`를 포함하여 인터페이스 색상을 선택합니다.

## 알려진 제한 사항

일부 동작은 스크린 리더 모드에 맞춰 조정되지 않습니다:

* 스크린 리더가 실행 중일 때 스크린 리더 모드가 자동으로 켜지지 않습니다.
* 명령에서 [플랜 모드](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)에 진입하는 등 `Shift+Tab` 전환 이외의 방법으로 변경된 권한 모드 변경 사항은 Claude Code가 안내하지 않습니다.
* `claude attach` 명령이나 에이전트 뷰에서 [백그라운드 세션](/docs/en/agent-view)에 연결하면 기본 스크롤백이 없는 터미널의 대체 화면(alternate screen)으로 진입합니다. 이는 [다른 연결된 세션과 동일한 동작](/docs/en/fullscreen)입니다. 다시 돌아가려면 빈 프롬프트에서 왼쪽 화살표 키를 누르거나 대화 상자에 포커스가 있는 경우 Ctrl+Z를 누르세요.
* Claude Code는 비용을 대화 턴당 알리지 않고 종료 시 출력하는 요약 정보에서 알립니다.
* 스크린 리더 모드는 `-p` 플래그를 사용하는 [비대화형 모드](/docs/en/headless)를 변경하지 않습니다. 비대화형 모드는 이미 일반 텍스트를 작성하며 스크립팅을 위한 대안으로 유지됩니다.

## 문제 보고하기

스크린 리더, 화면 확대기 또는 터미널에서 작동하지 않는 부분이 있으면 [Claude Code 이슈 트래커](https://github.com/anthropics/claude-code/issues)에 이슈를 생성하고 제목에 보조 공학 기술(assistive technology) 이름을 기재해 주세요. 보고서에 운영체제, 터미널 애플리케이션, 보조 공학 기술 이름 및 버전을 포함해 주세요.

## 관련 리소스

다음 페이지에서 이 페이지에서 다룬 내용의 전체 레퍼런스 항목 및 관련 설정을 확인할 수 있습니다:

* [설정](/docs/en/settings#available-settings): `axScreenReader`, `prefersReducedMotion`, `theme`, `preferredNotifChannel` 항목
* [환경 변수](/docs/en/env-vars): `CLAUDE_AX_SCREEN_READER`, `CLAUDE_CODE_ACCESSIBILITY` 항목
* [CLI 레퍼런스](/docs/en/cli-reference#cli-flags): `--ax-screen-reader` 플래그
* [터미널 구성](/docs/en/terminal-config): 스크린 리더 모드 외부의 벨, 알림 및 테마
* [비대화형 모드](/docs/en/headless): 스크린 리더 모드 없이 일반 텍스트를 작성하는 스크립트 기반 `claude -p` 실행
