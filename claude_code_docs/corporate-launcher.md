> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 기업용 런처 환경에서 Claude Code 실행하기

> `CLAUDE_CODE_PROCESS_WRAPPER` 또는 `processWrapper` 설정을 사용하여 백그라운드 서비스 및 모든 에이전트 뷰 세션을 포함하여 Claude Code 바이너리가 시작하는 프로세스를 지정된 필수의 런처를 통해 라우팅하세요.

일부 조직에서는 워크스테이션의 모든 프로세스를 필수 런처를 통해 시작하도록 요구합니다. 런처는 회사의 보안 태세에 필요한 샌드박스, 네트워크 제어 또는 자격 증명 주입을 적용하며, 런처 없이 시작되는 바이너리는 정책 위반에 해당합니다.

`CLAUDE_CODE_PROCESS_WRAPPER`는 Claude Code가 바이너리에서 실행하는 모든 프로세스(백그라운드 서비스, [에이전트 뷰](/docs/en/agent-view)에서 호스팅하는 모든 세션, 업데이트 후 Claude Code의 재실행)를 런처를 통해 시작합니다. 이 값을 런처의 절대 경로로 설정하면 Claude Code는 Claude Code 명령을 인수로 삼아 런처를 실행합니다.

`PATH`의 `claude` 명령을 감싸는 런처는 이러한 프로세스에 접근할 수 없습니다. 왜냐하면 이러한 프로세스는 `claude`를 검색하지 않고 바이너리의 직접 경로에서 바로 시작되기 때문입니다.

<Note>
  `CLAUDE_CODE_PROCESS_WRAPPER`에는 Claude Code v2.1.208 이상이 필요합니다. 이전 버전은 이 변수를 무시하고 모든 프로세스를 래퍼 없이 시작합니다. {/* min-version: 2.1.210 */}동일한 역할을 하는 [`processWrapper` 설정](/docs/en/settings#available-settings)에는 v2.1.210 이상이 필요합니다. 이전 버전은 이를 알 수 없는 키로 무시하여 런처를 적용하지 않으며 오류도 보고하지 않습니다.

  어느 형식이든 배포한 후에는 [검증 단계](#set-up-the-launcher)를 수행하여 실행 중인 버전이 이를 제대로 적용하는지 확인하세요.
</Note>

## 런처가 적용되는 범위

`CLAUDE_CODE_PROCESS_WRAPPER`가 설정되면 Claude Code는 런처를 통해 다음 각 프로세스를 시작합니다.

* `claude agents` 및 백그라운드 세션이 필요에 따라 시작하는 백그라운드 서비스.
* 서비스가 준비 상태로 유지하는 웜 스탠바이(warm standby) 세션을 포함하여 매 에이전트 뷰 행 내부의 터미널 호스트 및 Claude Code 세션.
* 업데이트나 충돌 발생 후 서비스가 다시 생성(respawn)하는 세션.
* 에이전트 뷰의 업데이트 재시작 동작을 포함하여 업데이트 설치를 완료하기 위해 Claude Code가 스스로 수행하는 재실행.
* {/* min-version: 2.1.210 */}백그라운드 서비스가 관리하는 [Remote Control](/docs/en/remote-control) 워커 프로세스. Claude Code v2.1.210 이상이 필요합니다.
* {/* min-version: 2.1.210 */}tmux 또는 iTerm2에서 [에이전트 팀](/docs/en/agent-teams)이 시작하는 분할 패널 팀원 세션. 팀원 패널은 백그라운드 프로세스가 아닌 대화형 프로세스이지만 Claude Code가 자체 바이너리에서 시작하므로 런처가 적용됩니다. Claude Code v2.1.210 이상이 필요합니다.

Windows에서는 이 변수가 무시됩니다. 런처 계약은 Windows에서 지원하지 않는 `exec`에 의존합니다. 변수가 설정된 Windows 머신은 모든 프로세스를 래퍼 없이 실행하며 계속 동작하고, 유일한 신호는 [디버그 로그](/docs/en/troubleshooting)의 경고뿐입니다. 런처 정책이 Windows에 적용되는 경우 이 변수는 Windows 환경을 만족시키지 못하므로 배포를 계획할 때 Windows 머신을 래퍼 미적용 대상으로 간주하세요.

### 런처 외부에서 시작되는 프로세스

다음 프로세스들은 런처를 통해 시작되지 않습니다.

* 런처가 구성되기 전에 유닛이 작성된 [설치된 백그라운드 서비스](/docs/en/agent-view#the-supervisor-process): `launchd` 또는 `systemd`가 해당 유닛 파일에서 프로세스를 시작합니다. 실행 중인 서비스와 구성된 런처가 일치하지 않는 동안 `/status` 및 `claude daemon status`가 경고를 표시하며, 변수가 설정에 포함되어 서비스가 재시작되면 서비스가 생성하는 세션은 런처를 통해 시작됩니다.
* 터미널에서 직접 시작한 세션: 사용자가 호출한 방식대로 실행됩니다. 이러한 세션에 적용하려면 `PATH`의 앞쪽 디렉토리에 런처와 실제 바이너리를 실행하는 `claude`라는 스크립트를 배치하세요. 관리 대상 심볼릭 링크를 대체하지는 마세요. 자체 생성(self-spawn) 시에는 `PATH`를 참조하지 않으므로 두 런처가 중첩 적용되지 않습니다.
* 운영체제의 프로토콜 핸들러가 직접 시작하는 `claude-cli://` 딥 링크의 첫 번째 프로세스. 이후 해당 세션이 백그라운드에서 시작하는 모든 항목은 런처를 통해 실행됩니다. 이 경로를 완전히 닫으려면 `disableDeepLinkRegistration` 설정으로 [핸들러 등록을 방지](/docs/en/deep-links#registration-and-supported-platforms)하세요.
* `--worktree`와 `--tmux`가 결합되어 실행하는 재실행: Claude Code의 바이너리가 아닌 터미널 멀티플렉서가 해당 패널을 시작합니다.
* [Claude in Chrome](/docs/en/chrome)이 등록하는 네이티브 메시징 호스트: Claude Code의 바이너리가 아닌 브라우저가 시작합니다.

### 프로세스 모니터상의 헬퍼 프로세스 이름

런처가 구성되면 `ps` 및 활동 모니터(Activity Monitor)는 Claude Code의 `claude bg-pty-host` 및 `claude bg-spare` 라벨 대신 백그라운드 헬퍼 프로세스의 버전에 맞는 바이너리 이름을 표시합니다. 이는 런처의 `exec`가 인수 목록을 재구성하기 때문에 발생하는 부작용이며 은폐가 아닙니다. 프로세스는 그 외에는 동일하며 Claude Code는 표시 이름이 아닌 바이너리 경로로 자체 프로세스를 식별합니다.

## 런처 설정하기

<Steps>
  <Step title="런처 스크립트 작성">
    `/opt/corp/launcher`와 같은 절대 경로에 실행 가능한 스크립트를 생성합니다. Claude Code는 전체 Claude Code 명령을 인수로 전달하여 런처를 실행하며, 스크립트는 스스로를 Claude Code로 교체하도록 `exec "$@"`를 호출하며 종료되어야 합니다.

    ```bash theme={null}
    #!/bin/sh
    # 조직의 설정: 샌드박스 진입, 네트워크 제어 적용,
    # 또는 자격 증명 주입.
    exec "$@"
    ```

    `chmod +x`로 실행 권한을 부여하세요. 설정 부분은 Claude Code가 실행되기 전 런처가 수행해야 하는 작업입니다. 아래의 [런처 계약](#the-launcher-contract)에 스크립트가 따라야 하는 규칙이 나와 있습니다.

    <Note>
      이전에 `~/.local/bin/claude` 심볼릭 링크를 런처로 대체한 적이 있다면 동일한 변경 시 원본 심볼릭 링크로 복원하세요. 대체된 심볼릭 링크는 첫 래핑된 세션이 두 런처를 통해 동시에 백그라운드 서비스를 시작하게 만들며 외부 관리 상태로 만듭니다: `/doctor`가 이를 보고하고 자동 업데이트가 해당 파일을 그대로 유지하며 이 경로를 다시 관리하기 전까지 이전 버전 정리가 비활성화됩니다.
    </Note>
  </Step>

  <Step title="설정 파일에 CLAUDE_CODE_PROCESS_WRAPPER 설정">
    분리된 백그라운드 서비스가 상속받을 수 있도록 설정 파일의 `env` 블록에 변수를 설정하세요. 셸의 `export`로는 부족합니다. 백그라운드 서비스는 요청에 따라 시작되고 셸보다 오래 유지되며 셸 프로필을 다시 읽지 않기 때문입니다.

    단일 머신의 경우 `~/.claude/settings.json`에 추가하세요. 조직의 모든 머신에 배포하려면 [관리 대상 설정(managed settings)](/docs/en/permissions#managed-settings)에 동일한 블록을 추가하세요.

    ```json theme={null}
    {
      "env": {
        "CLAUDE_CODE_PROCESS_WRAPPER": "/opt/corp/launcher"
      }
    }
    ```

    둘 이상의 출처에서 변수를 설정하는 경우 관리 대상 설정 값이 `~/.claude/settings.json`과 셸에 내보내진 값 모두를 재정의하므로 사용자가 직접 생성한 세션을 다른 런처로 지정할 수 없습니다.

    [`processWrapper` 설정](/docs/en/settings#available-settings)은 지정된 최상위 설정 키로 동일한 값을 전달합니다. 조직에서 `env` 블록 대신 개별 키로 설정을 푸시할 때 설정하세요. `processWrapper` 설정에는 Claude Code v2.1.210 이상이 필요합니다. 다음 설정 파일은 키를 통해 동일한 런처를 설정합니다.

    ```json theme={null}
    {
      "processWrapper": "/opt/corp/launcher"
    }
    ```

    둘 다 설정되면 `CLAUDE_CODE_PROCESS_WRAPPER`가 우선권을 가집니다.

    `processWrapper`는 이름이 지정된 설정이므로 [원격 관리 대상 설정](/docs/en/settings#settings-files)을 통해 제공하는 조직에서는 관리자가 제공한 실행 파일을 실행하는 다른 설정들과 함께 [보안 승인 다이얼로그](/docs/en/server-managed-settings#security-approval-dialogs)에 해당 목록이 나타납니다.

    프로젝트 및 로컬 설정은 런처를 구성할 수 없습니다. 리포지토리에 커밋된 파일이 머신의 모든 Claude Code 프로세스 앞에 바이너리를 두어서는 안 되므로 Claude Code는 `.claude/settings.json` 또는 `.claude/settings.local.json`에 지정된 `CLAUDE_CODE_PROCESS_WRAPPER`를 무시하고 [디버그 로그](/docs/en/troubleshooting)에 경고를 남기며 해당 파일에서 `processWrapper` 키를 읽지 않습니다.
  </Step>

  <Step title="백그라운드 서비스 및 세션 재시작">
    실행 중인 백그라운드 서비스 및 열려 있는 `claude` 세션은 시작 시 변수를 한 번 읽으므로 재시작할 때까지 래핑되지 않은 프로세스를 계속 실행합니다. 요청에 따라 실행되는 서비스를 중지하려면 `claude daemon stop --any`를 실행하세요. `claude agents`와 같이 이를 필요로 하는 다음 명령이 래핑된 서비스를 시작합니다. [설치된 서비스](/docs/en/agent-view#the-supervisor-process)는 `--any` 없이 `claude daemon stop`을 실행합니다. 그런 다음 열려 있는 `claude` 세션을 재시작하세요.

    수동으로 재시작할 수 없는 머신의 경우 설정 푸시 후 시작되는 첫 번째 세션이 남아 있는 래핑되지 않은 온디맨드 서비스를 자동으로 정리합니다. 새 세션이 시작되지 않는 머신은 시작될 때까지 래핑되지 않은 서비스를 유지하며, 설치된 서비스는 이 단계에서 항상 재시작이 필요합니다.
  </Step>

  <Step title="검증">
    세션에서 `/status`를 실행하세요: Self-exec 항목이 확인된 실행 명령을 보여주며 실행 중인 백그라운드 서비스와 일치하지 않을 때 경고합니다. `claude daemon status`는 변수를 해제한 후(`/status`에 항목이 나타나지 않음)를 포함하여 셸에서 동일한 정보를 출력합니다.
  </Step>
</Steps>

## 런처 계약 (Launcher contract)

런처를 실행할 수 없을 때 Claude Code는 프로세스를 래핑되지 않은 상태로 시작하는 대신 시작을 거부합니다. Windows에서는 [변수가 무시되며](#what-the-launcher-covers) 프로세스가 래핑되지 않은 상태로 시작됩니다. Claude Code는 스크립트가 다음 규칙을 따르도록 요구합니다.

* **`exec "$@"`로 종료할 것.** 자식을 포크하고 종료하는 런처는 백그라운드 서비스가 추적할 수 없는 고아 Claude Code 프로세스를 남깁니다. 에이전트 뷰는 이러한 세션을 런처 이름을 지정하는 메시지와 함께 실패로 표시하고 서비스는 런처가 남긴 프로세스를 회수합니다.
* **인수의 순서를 변경하거나 흡수하거나 앞에 추가하지 말 것.** 첫 번째 인수는 Claude Code 바이너리이며 그 이후의 모든 항목은 해당 argv입니다.
* **상속된 모든 환경 변수를 `exec`로 전달할 것.** 주입된 자격 증명과 같이 변수를 추가하는 것은 괜찮지만 상속된 변수를 삭제하는 것은 안 됩니다.
  * 세션별 인증 토큰, 모델 및 공급자 선택, `CLAUDE_CODE_PROCESS_WRAPPER` 자체가 모두 상속된 환경에서 전달되므로 허용 목록에서 이를 재구성하는 런처는 시작하는 세션을 깨뜨리고 `/status`가 런처 불일치를 보고합니다.
  * 런처가 환경을 재설정하는 네임스페이스나 샌드박스에 진입해야 하는 경우 내부에서 상속된 환경을 문자 그대로 다시 내보내세요.
* **런처가 실행될 때마다 약 3초 이내에 `exec`에 도달할 것.** 코콜드 백그라운드 디스패치는 첫 번째 출력 바이트 전 직렬로 런처를 두 번 실행하므로 단일 로그온(SSO) 교환과 같은 느린 작업은 지연(lazy) 처리하거나 캐시에서 처리하세요.
  * 예산을 훨씬 초과하여 실행되는 런처는 정지된 시작으로 취급되어 재시작됩니다.
* **자신 내부에서 호출되는 경우를 허용할 것.** Claude Code는 모든 중첩된 자체 생성(self-spawn)에 런처를 적용하므로 전유 리소스를 획득하는 런처는 이미 해당 리소스를 보유하고 있는지 감지해야 합니다.
* **Claude Code가 시작되기 전에 터미널에 쓰지 말 것.** `exec` 전에 출력된 모든 내용은 초기화 전 세션이 종료되는 경우 충돌 원인으로 보고됩니다.

### 런처 값의 형식

`CLAUDE_CODE_PROCESS_WRAPPER`와 `processWrapper` 설정은 동일한 형식을 갖습니다. 대부분의 런처의 경우 값은 `/opt/corp/launcher`와 같은 스크립트의 절대 경로입니다.

런처 자체의 인수를 전달하려면 경로 뒤에 작성하세요. Claude Code는 이 값을 셸 명령이 아닌 인수 목록으로 파싱합니다.

* 공백은 토큰을 구분하며 큰따옴표는 공백이 포함된 토큰을 묶습니다.
* `[`로 시작하는 값은 `["/opt/corp/launcher", "--profile", "cc"]`와 같은 JSON 문자열 배열로 읽힙니다.
* 셸 구문은 작동하지 않습니다: 변수 확장이나 글로빙(globbing)이 없으며, 따옴표로 묶이지 않은 `;`, `|`, `&`, `$(`와 같은 연산자는 재해석되지 않고 구성 오류로 거부됩니다.

값을 사용할 수 없을 때 Claude Code는 영향을 받는 프로세스의 시작을 거부하고 [이유를 보고합니다](/docs/en/errors#claude_code_process_wrapper-launcher-errors).

## `CLAUDE_CODE_SHELL_PREFIX`와의 관계

`CLAUDE_CODE_PROCESS_WRAPPER`는 Claude Code 자체 프로세스를 감싸고 명령을 런처가 `exec`할 별도의 argv 토큰으로 전달합니다. [`CLAUDE_CODE_SHELL_PREFIX`](/docs/en/env-vars)는 사용자를 대신해 Claude가 실행하는 셸 명령(Bash 도구 호출, 훅, stdio MCP 서버를 시작하는 명령)을 감싸고 각각을 래퍼가 재평가할 수 있도록 `$1`의 단일 셸 따옴표 문자열로 전달합니다. 하나를 위해 작성된 런처는 다른 용도로 작동하지 않습니다.

## 관련 리소스

* [에이전트 뷰 (Agent view)](/docs/en/agent-view): 런처가 적용되는 백그라운드 세션 및 수퍼바이저 프로세스
* [환경 변수 (Environment variables)](/docs/en/env-vars): `CLAUDE_CODE_PROCESS_WRAPPER` 레퍼런스 항목
* [관리 대상 설정 (Managed settings)](/docs/en/permissions#managed-settings): 전체 환경에 `env` 블록 배포
* [런처 오류 레퍼런스](/docs/en/errors#claude_code_process_wrapper-launcher-errors): 거부 메시지 및 복구 방법
