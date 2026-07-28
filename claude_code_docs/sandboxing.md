> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 샌드박스 Bash 도구 구성하기

> Claude Code의 샌드박스 Bash 도구가 보다 안전하고 자율적인 에이전트 실행을 위해 파일 시스템 및 네트워크 격리를 제공하는 방법을 알아보세요.

Bash 샌드박스를 사용하면 Claude가 매번 권한을 묻지 않고도 대부분의 쉘 명령을 실행할 수 있습니다. 각 명령을 일일이 승인하는 대신, 명령이 접근할 수 있는 파일과 네트워크 도메인을 정의하면 운영체제가 모든 Bash 명령과 그 자식 프로세스에 대해 해당 경계를 강제 적용합니다.

<Note>
  개발 컨테이너, 커스텀 컨테이너, 가상 머신과 같은 다른 격리 접근 방식을 비교하려면 [Sandbox environments](/docs/en/sandbox-environments)를 참조하세요. Bash 이외의 도구에 대해 권한 프롬프트를 줄이려면 [permission modes](/docs/en/permission-modes)를 참조하세요.
</Note>

## 시작하기

샌드박스는 Claude Code에 내장되어 있으며 macOS, Linux, WSL2에서 작동합니다. 네이티브 Windows는 지원되지 않습니다. Windows에서는 WSL2 배포판 내부에서 Claude Code를 실행하세요.

macOS에서는 별도로 설치할 것이 없습니다. 샌드박스는 내장된 Seatbelt 프레임워크를 사용합니다. Linux 및 WSL2에서 샌드박스는 두 개의 패키지에 의존하며, 이는 [Linux 및 WSL2 설정](#set-up-linux-and-wsl2)에서 다룹니다. 해당 패키지를 아직 설치하지 않았더라도 패널에서 누락된 항목이 있는지 보여주므로 `/sandbox`로 바로 시작할 수 있습니다.

<Steps>
  <Step title="/sandbox 실행">
    Claude Code 세션을 시작하고 `/sandbox` 명령을 실행합니다:

    ```text theme={null}
    /sandbox
    ```

    이 명령은 3개의 탭으로 구성된 샌드박스 패널을 엽니다(Linux에서 옵션 seccomp 필터가 없는 경우 Dependencies 탭이 추가됨):

    * **Mode**: 샌드박스 처리된 명령이 승인되는 방식을 선택합니다(다음 단계에서 다룸)
    * **Overrides**: 샌드박스 아래에서 실패한 명령이 샌드박스 해제(unsandboxed) 상태로 대체 실행될 수 있는지 여부를 선택합니다. 이는 [`allowUnsandboxedCommands`](/docs/en/settings#sandbox-settings) 설정입니다
    * **Config**: 확인된 샌드박스 설정을 봅니다

    패널에 Dependencies 탭만 표시되는 경우 필수 패키지가 누락된 것입니다. [Linux 및 WSL2 설정](#set-up-linux-and-wsl2)에 설명된 대로 설치하고 Claude Code를 재시작한 후 `/sandbox`를 다시 실행하세요.
  </Step>

  <Step title="모드 선택">
    Mode 탭에서 자동 허용(auto-allow) 또는 일반 권한(regular permissions)을 선택합니다. 자동 허용은 프롬프트 없이 샌드박스 명령을 실행하며, 일반 권한은 명령이 샌드박스 처리된 경우에도 일반 권한 프롬프트를 유지합니다. 자동 허용 모드에서도 프롬프트를 표시하는 명령은 [샌드박스 모드](#sandbox-modes)를 참조하세요.
  </Step>

  <Step title="Bash 명령 실행">
    Claude에게 빌드나 테스트 수트 실행과 같은 명령을 요청합니다. 기본적으로 샌드박스 내부의 명령은 작업 디렉터리와 세션 임시 디렉터리에만 쓸 수 있습니다. 명령이 새로운 네트워크 도메인을 처음으로 필요로 할 때 Claude Code가 승인을 요청합니다.

    샌드박스 처리될 수 없는 명령은 일반 권한 흐름으로 전환됩니다. 이러한 경계를 넓히거나 좁히려면 [샌드박스 구성하기](#configure-sandboxing)를 참조하세요.
  </Step>
</Steps>

패널에서 모드를 선택하면 프로젝트의 로컬 설정 파일인 `.claude/settings.local.json`에 기록되어 현재 프로젝트에 적용되고 git에 커밋되지 않습니다. 모든 프로젝트에서 샌드박스를 활성화하려면 `~/.claude/settings.json`의 사용자 설정에서 [`sandbox.enabled`](/docs/en/settings#sandbox-settings)를 `true`로 설정하세요. 조직의 모든 개발자에게 샌드박스를 강제 적용하려면 [관리 설정](#enforce-sandboxing-with-managed-settings)을 사용하세요.

<Warning>
  기본적으로 종속성이 누락되었거나 플랫폼이 지원되지 않아 샌드박스를 시작할 수 없는 경우, Claude Code는 경고를 표시하고 샌드박스 없이 명령을 실행합니다. 이를 하드 실패로 변경하려면 [`sandbox.failIfUnavailable`](/docs/en/settings#sandbox-settings)을 `true`로 설정하세요. 이는 보안 게이트로 샌드박스를 필수로 하는 관리형 배포를 위한 것입니다.
</Warning>

### Linux 및 WSL2 설정

Linux 및 WSL2에서 샌드박스는 다음 두 가지 패키지에 의존합니다:

* [`bubblewrap`](https://github.com/containers/bubblewrap): 파일 시스템 격리를 강제하는 비권한(unprivileged) 샌드박스 도구
* [`socat`](http://www.dest-unreach.org/socat/): 샌드박스 프록시를 통해 네트워크 트래픽을 라우팅하는 데 사용되는 릴레이

배포판의 패키지 관리자로 설치하세요:

<Tabs>
  <Tab title="Ubuntu/Debian">
    ```bash theme={null}
    sudo apt-get install bubblewrap socat
    ```
  </Tab>

  <Tab title="Fedora">
    ```bash theme={null}
    sudo dnf install bubblewrap socat
    ```
  </Tab>
</Tabs>

종속성이 누락되면 `/sandbox` 의 Dependencies 탭에 해당 플랫폼에서 `ripgrep`, `bubblewrap`, `socat` 및 seccomp 필터 중 무엇이 부족한지 나열됩니다. 설치 및 Claude Code 재시작 후 해당 탭이 보이지 않는다면 모든 종속성이 준비된 것입니다.

Ripgrep은 네이티브 Claude Code 바이너리에 번들로 제공됩니다. seccomp 필터는 옵션이며 Unix 도메인 소켓 차단을 추가합니다. 누락된 경우 `npm install -g @anthropic-ai/sandbox-runtime`으로 설치하세요.

필수 종속성이 누락된 경우 설치할 때까지 Dependencies 탭만 표시됩니다. 옵션인 seccomp 필터만 누락된 경우 Dependencies 탭이 다른 탭과 함께 표시됩니다. 종속성 검사는 시작 시 실행되므로 패키지를 설치한 후 Claude Code를 재시작해야 `/sandbox`에서 이를 감지합니다.

<AccordionGroup>
  <Accordion title="Ubuntu 24.04 이상: bubblewrap의 사용자 네임스페이스 생성 허용">
    Ubuntu 24.04 이상에서는 기본 AppArmor 정책으로 인해 bubblewrap이 격리에 필요한 사용자 네임스페이스를 생성하지 못합니다.

    WSL2 내부를 포함하여 사용자의 환경이 이 제한을 강제하는지 확인하려면 `sysctl kernel.apparmor_restrict_unprivileged_userns`를 실행하세요. 명령이 `0`을 반환하면 이 단계를 건너뜁니다. `No such file or directory` 오류를 출력하면 해당 키가 존재하지 않는 것이므로 이 단계를 건너뜁니다. `1`을 반환하면 `bwrap`에 이 기능을 부여하는 AppArmor 프로필을 추가하세요:

    ```bash theme={null}
    sudo tee /etc/apparmor.d/bwrap > /dev/null <<'EOF'
    abi <abi/4.0>,
    include <tunables/global>

    profile bwrap /usr/bin/bwrap flags=(unconfined) {
      userns,
      include if exists <local/bwrap>
    }
    EOF
    ```

    해당 프로필은 샌드박스 내부에서 실행하는 명령이 아니라 `bwrap` 자체에만 적용됩니다. AppArmor를 다시 로드하여 적용하세요:

    ```bash theme={null}
    sudo systemctl reload apparmor
    ```
  </Accordion>

  <Accordion title="WSL2 참고 사항">
    PowerShell에서 `wsl -l -v`로 WSL 버전을 확인하세요. `Sandboxing requires WSL2`가 표시되면 배포판이 WSL1으로 실행 중인 것입니다. WSL2로 업그레이드하거나 샌드박스 없이 Claude Code를 실행하세요.

    WSL2에서 샌드박스 처리된 명령은 `cmd.exe`, `powershell.exe` 또는 `/mnt/c/` 아래의 항목과 같은 Windows 바이너리를 실행할 수 없습니다. WSL은 샌드박스가 차단하는 Unix 소켓을 통해 이를 Windows 호스트로 전달합니다. 명령이 Windows 바이너리를 호출해야 하는 경우 샌드박스 외부에서 실행되도록 [`excludedCommands`](/docs/en/settings#sandbox-settings)에 추가하세요.
  </Accordion>
</AccordionGroup>

### 샌드박스 모드

Claude Code는 두 가지 샌드박스 모드를 제공합니다:

**Auto-allow 모드 (자동 허용 모드)**: Bash 명령이 샌드박스 내부에서 실행되도록 시도하며 권한을 요구하지 않고 자동으로 허용됩니다. 허용되지 않은 호스트에 대한 네트워크 접근이 필요한 명령과 같이 샌드박스 처리될 수 없는 명령은 일반 권한 흐름으로 전환됩니다. 이 흐름에서는 Claude Code가 사용자의 [권한 규칙](/docs/en/permissions)을 확인하고 기본 모드의 프롬프트 또는 [오토 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)의 분류기를 통해 해당 규칙이 아직 허용하지 않은 모든 명령을 차단합니다.

자동 허용 모드에서도 다음 사항은 여전히 적용됩니다:

* 명시적인 [deny 규칙](/docs/en/permissions)은 항상 준수됩니다.
* `/`, 홈 디렉터리 또는 기타 중요한 시스템 경로를 대상으로 하는 `rm` 또는 `rmdir` 명령은 여전히 권한 프롬프트를 트리거합니다.
* `Bash(git push *)`와 같이 내용 범위가 지정된 [ask 규칙](/docs/en/permissions)은 샌드박스 처리된 명령에 대해서도 프롬프트를 적용합니다.
* 단독 `Bash` ask 규칙 또는 이에 해당하는 `Bash(*)` 형식은 샌드박스 처리된 명령에 대해서는 건너뜁니다. 일반 권한 흐름으로 전환되는 명령에는 여전히 적용됩니다. {/* min-version: 2.1.212 */} [plan 모드](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)에서는 해당 규칙을 건너뛰지 않으며 읽기 전용 명령을 포함하여 샌드박스 처리된 명령에 대해서도 프롬프트합니다. v2.1.212 이전에는 plan 모드에서도 건너뛰기가 적용되었습니다.

**Regular permissions 모드 (일반 권한 모드)**: 모든 Bash 명령은 샌드박스 처리되더라도 일반 권한 흐름을 거칩니다. 이는 더 많은 제어권을 제공하지만 더 많은 승인이 필요합니다.

두 모드 모두에서 샌드박스는 동일한 파일 시스템 및 네트워크 제한을 적용합니다. 차이점은 샌드박스 처리된 명령이 자동 승인되는지 아니면 명시적인 권한이 필요한지에만 있습니다.

세션 임시 디렉터리는 작업 디렉터리와 함께 기본적으로 샌드박스 내부에서 쓰기가 가능합니다. Claude Code는 샌드박스 처리된 명령에 대해 `$TMPDIR`을 이 디렉터리로 설정하므로 임시 파일을 쓰는 도구가 추가 구성 없이 작동합니다. 샌드박스 처리되지 않은 명령은 쉘의 `$TMPDIR`을 변경 없이 상속받으므로, 사용자가 [파일 시스템 격리를 비활성화](#disable-filesystem-isolation)하지 않는 한 샌드박스 처리된 명령과 비샌드박스 명령은 `$TMPDIR`을 서로 다른 디렉터리로 확인합니다. 두 사이에서 임시 파일을 전달하려면 작업 디렉터리 아래에 작성하세요.

일부 명령은 샌드박스와 호환되지 않는 도구이거나 허용하지 않은 호스트가 필요한 경우 등 샌드박스 내부에서 전혀 실행할 수 없습니다. Claude Code는 작업을 실패시키거나 샌드박스를 끄도록 요구하는 대신 탈출구(escape hatch)를 포함합니다. 샌드박스 제한으로 인해 명령이 실패하면 Claude는 실패를 분석하고 `dangerouslyDisableSandbox` 매개변수를 사용하여 명령을 재시도할 수 있습니다. 재시도된 명령은 샌드박스 외부에서 실행되므로 일반 권한 흐름을 거칩니다. 기본 모드에서는 확인 프롬프트가 표시되며, [오토 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)에서는 분류기가 프롬프트 대신 밑바탕의 명령을 평가합니다. 오토 모드에서도 샌드박스 해제 재시도 시 매번 프롬프트를 받으려면 `Bash(dangerouslyDisableSandbox:true)`에 대한 [ask 규칙](/docs/en/permissions#match-by-input-parameter)을 추가하세요.

[샌드박스 설정](/docs/en/settings#sandbox-settings)에서 `"allowUnsandboxedCommands": false`를 설정하여 이 탈출구를 비활성화할 수 있습니다. 비활성화되면(`/sandbox` Overrides 탭에 **Strict sandbox mode**로 표시됨) `dangerouslyDisableSandbox` 매개변수가 완전히 무시되며 모든 명령은 샌드박스 처리되어 실행되거나 `excludedCommands`에 명시적으로 나열되어야 합니다.

<Info>
  자동 허용 모드는 한 가지 예외인 [plan 모드](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)를 제외하고 권한 모드 설정과 독립적으로 작동합니다. "accept edits" 모드가 아니더라도 자동 허용이 활성화되어 있으면 샌드박스 처리된 Bash 명령이 자동으로 실행됩니다. 이는 파일 편집 도구가 일반적으로 승인을 요구하는 경우에도 샌드박스 경계 내에서 파일을 수정하는 Bash 명령이 프롬프트 없이 실행됨을 의미합니다.

  {/* min-version: 2.1.212 */}plan 모드에서는 [읽기 전용 명령](/docs/en/permissions#read-only-commands)만 프롬프트 없이 실행되며, 그 외의 다른 Bash 명령은 자동 허용이 활성화되어 있더라도 승인 프롬프트를 표시합니다. v2.1.212 이전에는 plan 모드에서도 자동 허용이 프롬프트 없이 샌드박스 명령을 실행했습니다.
</Info>

## 샌드박스 구성하기

`settings.json` 파일을 통해 샌드박스 동작을 커스텀하세요. 전체 구성 참조는 [Settings](/docs/en/settings#sandbox-settings)를 보세요.

기본적으로 샌드박스 처리된 명령은 현재 작업 디렉터리와 세션 임시 디렉터리에만 쓸 수 있습니다. `kubectl`, `terraform` 또는 `npm`과 같은 하위 프로세스 명령이 해당 디렉터리 외부에 써야 하는 경우 `sandbox.filesystem.allowWrite`를 사용하여 특정 경로에 대한 접근을 허용하세요:

```json theme={null}
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "allowWrite": ["~/.kube", "/tmp/build"]
    }
  }
}
```

이러한 경로는 OS 수준에서 강제 적용되므로 샌드박스 내부에서 실행되는 모든 명령(자식 프로세스 포함)이 이를 준수합니다. 이는 `excludedCommands`로 도구를 샌드박스에서 완전히 제외하는 것보다, 도구가 특정 위치에 쓰기 접근이 필요할 때 권장하는 접근 방식입니다.

동일한 filesystem 배열이 여러 [설정 범위](/docs/en/settings#settings-precedence)에 정의된 경우 배열이 병합됩니다. 즉, 교체되지 않고 모든 범위의 경로가 조합됩니다.

경로 접두사는 경로가 해석되는 방식을 제어합니다:

| 접두사 | 의미 | 예시 |
| :---------------- | :------------------------------------------------------------------------------------- | :------------------------------------------------------------------------ |
| `/` | 파일 시스템 루트 기준 절대 경로 | `/tmp/build`는 `/tmp/build` 유지 |
| `~/` | 홈 디렉터리 기준 상대 경로 | `~/.kube`는 `$HOME/.kube`로 변환 |
| `./` 또는 접두사 없음 | 프로젝트 설정의 경우 프로젝트 루트 기준, 사용자 설정의 경우 `~/.claude` 기준 | `.claude/settings.json`의 `./output`은 `<project-root>/output`으로 해석 |

이 구문은 absolute에 `//path`를, project-relative에 `/path`를 사용하는 [Read 및 Edit 권한 규칙](/docs/en/permissions#read-and-edit)과 다릅니다. 샌드박스 파일 시스템 경로는 표준 관례를 따릅니다: `/tmp/build`는 절대 경로입니다.

`sandbox.filesystem.denyWrite` 및 `sandbox.filesystem.denyRead`를 사용하여 쓰기 또는 읽기 접근을 거부할 수도 있으며, `sandbox.filesystem.allowRead`를 사용하여 거부된 영역 내의 특정 경로를 다시 허용할 수 있습니다. 읽기 규칙이 겹칠 때 더 구체적인 경로가 우선합니다:

| 예시 규칙 | 결과 |
| :------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `"denyRead": ["~/"]` 및 `"allowRead": ["~/projects"]` | `~/projects`는 읽을 수 있고 홈 디렉터리의 나머지는 차단 상태를 유지합니다. 더 좁은 allow가 거부된 영역의 일부를 다시 엽니다. |
| `"allowRead": ["~/"]` 및 `"denyRead": ["~/.env"]` | `~/.env`는 차단 상태를 유지하고 홈 디렉터리의 나머지는 읽을 수 있습니다. 정확한 deny는 넓은 allow 내부에서 유지되므로 broad allow가 보안 비밀을 자동으로 다시 노출할 수 없습니다. |

아래 예시는 현재 프로젝트에서의 읽기를 허용하면서 전체 홈 디렉터리에서의 읽기를 차단합니다. 상대 경로 `.`는 구성이 프로젝트 설정에 있을 때만 프로젝트 루트로 해석되므로 이를 프로젝트의 `.claude/settings.json`에 배치하세요:

```json theme={null}
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "denyRead": ["~/"],
      "allowRead": ["."]
    }
  }
}
```

이 구성이 프로젝트 설정에 있으므로 `allowRead`의 `.`는 프로젝트 루트로 해석됩니다. 동일한 구성을 `~/.claude/settings.json`에 두면 `.`는 `~/.claude`로 해석되며 프로젝트 파일은 `denyRead` 규칙에 의해 차단된 상태로 남습니다.

### 파일 시스템 격리 비활성화

네트워크 격리를 유지하면서 파일 시스템 격리를 건너뛰려면 `sandbox.filesystem.disabled`를 `true`로 설정하세요. 아래 예시는 허용된 네트워크 도메인 목록을 유지하면서 파일 시스템 격리를 끕니다:

```json theme={null}
{
  "sandbox": {
    "enabled": true,
    "filesystem": {
      "disabled": true
    },
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org"]
    }
  }
}
```

샌드박스에는 두 개의 독립적인 레이어가 있습니다. [파일 시스템 격리](#filesystem-isolation)는 샌드박스 처리된 명령이 읽고 쓸 수 있는 경로를 제어하고, [네트워크 격리](#network-isolation)는 도달할 수 있는 도메인을 제어합니다. 파일 시스템 레이어가 꺼지면 샌드박스 처리된 명령은 호스트 파일 시스템에 대한 제한 없는 읽기 및 쓰기 접근 권한을 얻지만, 네트워크 아웃바운드는 허용된 도메인으로 국한됩니다. 쓰는 대상보다 연결하는 위치를 제어하기 위해 샌드박스를 적용할 때 이 레이어를 끄세요.

이 설정은 기본적으로 꺼져 있으며 샌드박스가 실행되는 플랫폼(macOS, Linux, WSL2)에 적용됩니다. Claude Code v2.1.216 이상이 필요합니다.

<Warning>
  파일 시스템 격리가 꺼지고 명령이 자동 허용되면, 샌드박스 처리된 명령이 쉘 시작 파일, `$PATH`의 실행 파일, 또는 `~/.claude/settings.json`과 같이 이후 명령이 실행하거나 읽는 파일을 작성하여 다음 실행 시 자체 접근 권한을 넓히는 데 사용할 수 있습니다. 파일 시스템을 수정을 통해 스스로 접근 권한을 승격시키지 않는다고 신뢰할 수 있는 워크로드에 대해서만 `filesystem.disabled`를 `true`로 설정하세요. [`allowManagedDomainsOnly`](#keep-developers-from-widening-the-policy)로 네트워크 도메인을 잠그면 위험이 좁아지지만, 해당 잠금은 샌드박스 내부에서 실행되는 명령에만 적용되므로 위험이 완전히 제거되지는 않습니다.
</Warning>

#### 비활성화할 수 있는 설정 출처

파일 시스템 격리를 끄면 샌드박스 처리된 명령이 수행할 수 있는 작업이 넓어지므로, Claude Code는 다음 설정 출처의 `filesystem.disabled`만 준수합니다:

* 사용자 설정, 관리 설정 및 `--settings` CLI 플래그에서 설정할 수 있습니다. `.claude/settings.json` 및 `.claude/settings.local.json`의 프로젝트 설정에서는 설정할 수 없으므로 체크아웃된 프로젝트가 파일 시스템 격리를 끌 수는 없습니다.
* 관리 설정이 `sandbox.filesystem`을 구성하거나 `sandbox.credentials.files` 항목을 나열하는 경우, 관리 설정만 해당 키를 설정할 수 있습니다. 이는 관리자가 배포한 파일 시스템 제한을 유효하게 유지합니다. 이러한 배포를 완화하려면 관리 설정에서 `"disabled": true`를 설정하세요.
* [`CLAUDE_CODE_SUBPROCESS_ENV_SCRUB`](/docs/en/env-vars)가 설정되어 있으면 Claude Code는 관리 설정을 포함한 모든 출처의 `filesystem.disabled`를 무시하고 파일 시스템 격리를 켜 둡니다.

#### 격리가 꺼졌을 때 변경되는 사항

파일 시스템 레이어를 끄면 읽기 보호가 해제되고 `$TMPDIR` 재정의가 중지되는 반면, 샌드박스 처리된 명령은 계속 자동 허용됩니다:

* 파일 시스템 레이어가 둘 다를 강제하므로 `filesystem.denyRead` 및 [`credentials.files`](#protect-credentials)의 읽기 보호가 샌드박스 처리된 명령에 적용되지 않습니다. 환경 변수 제거는 파일 시스템 레이어와 독립적이므로 `credentials.envVars` deny 및 mask 항목은 여전히 적용됩니다.
* 모든 임시 디렉터리가 쓰기 가능하므로 샌드박스 처리된 명령은 세션 임시 디렉터리 대신 쉘의 `$TMPDIR`을 상속받습니다.
* [`autoAllowBashIfSandboxed`](/docs/en/settings#sandbox-settings)는 여전히 기본적으로 `true`이므로 샌드박스 처리된 명령은 프롬프트 없이 계속 실행됩니다. 샌드박스 처리된 명령에 대해 프롬프트를 계속 받으려면 이를 `false`로 설정하세요.

### 자격 증명 보호

`sandbox.credentials` 설정은 샌드박스 처리된 명령으로부터 보호할 자격 증명 파일 및 환경 변수를 선언합니다. 각 항목은 파일 경로 또는 환경 변수 이름과 `mode`를 지정합니다. 전용 `credentials` 블록은 자격 증명 규칙을 일반 파일 시스템 규칙과 분리하여 그룹화합니다. Claude Code v2.1.187 이상이 필요합니다.

`"mode": "deny"`인 항목의 경우 파일 경로는 샌드박스 내부에서 읽기가 거부되며( `filesystem.denyRead`가 적용하는 제한과 동일), 환경 변수는 각 샌드박스 명령이 실행되기 전에 해제됩니다. 파일 보호는 파일 시스템 레이어의 일부이므로 [파일 시스템 격리를 비활성화](#disable-filesystem-isolation)하면 적용되지 않지만, 환경 변수 보호는 여전히 적용됩니다.

아래 예시는 AWS 자격 증명 파일 및 SSH 디렉터리의 읽기를 차단하고 샌드박스 처리된 명령의 환경에서 `GITHUB_TOKEN` 및 `NPM_TOKEN`을 제거합니다:

```json theme={null}
{
  "sandbox": {
    "enabled": true,
    "credentials": {
      "files": [
        { "path": "~/.aws/credentials", "mode": "deny" },
        { "path": "~/.ssh", "mode": "deny" }
      ],
      "envVars": [
        { "name": "GITHUB_TOKEN", "mode": "deny" },
        { "name": "NPM_TOKEN", "mode": "deny" }
      ]
    }
  }
}
```

파일 항목은 `"mode": "deny"`만 지원합니다. 환경 변수 항목은 아래에 설명된 `"mode": "mask"`도 허용합니다.

파일 경로는 `sandbox.filesystem.*` 설정과 동일한 [접두사 규칙](/docs/en/settings#sandbox-path-prefixes)을 따르며, 모든 [설정 범위](/docs/en/settings#settings-precedence)의 `deny` 항목이 병합됩니다. `deny` 항목은 접근 권한을 좁히기만 하므로 모든 범위가 이를 추가할 수 있지만, 어떤 범위도 다른 범위가 추가한 항목을 제거할 수는 없습니다.

내장된 자격 증명 거부 목록은 없으므로 사용자가 나열한 파일과 변수만 제한됩니다. 이 설정은 샌드박스 처리된 Bash 명령에만 영향을 줍니다. 샌드박스 여부와 관계없이 모든 하위 프로세스에서 Anthropic 및 클라우드 제공업체 자격 증명을 제거하려면 [`CLAUDE_CODE_SUBPROCESS_ENV_SCRUB`](/docs/en/env-vars)를 설정하세요.

#### 환경 변수 마스킹

`"mode": "mask"`는 자격 증명으로 인증하는 도구를 작동 가능하게 유지하면서 자격 증명을 보호합니다. `deny`는 변수를 완전히 제거하여 `gh`나 `npm`과 같이 해당 변수가 필요한 도구를 작동 불능 상태로 만듭니다. Claude Code v2.1.199 이상이 필요합니다.

`mask`를 사용하면 샌드박스 처리된 명령은 실제 값 대신 세션별 감시용 센티널(sentinel) 값을 봅니다. 요청이 자격 증명의 `injectHosts` 중 하나를 위해 샌드박스를 이탈할 때 [샌드박스 프록시](#network-isolation)가 센티널을 실제 값으로 대체합니다. 명령과 기록되는 모든 로그에는 실제 자격 증명이 담기지 않지만 요청은 인증을 거칩니다.

프록시는 요청 내용 내부의 자격 증명을 대체하므로 해당 내용을 볼 수 있어야 합니다. 프록시가 TLS를 직접 종료하도록 [`network.tlsTerminate`](/docs/en/settings#sandbox-settings)를 설정하세요. 설정하지 않으면 마스킹이 실패 상태로 닫힙니다. 즉 명령은 여전히 센티널만 보고, 센티널이 변경 없이 서버에 전달되어 인증이 실패합니다. Claude Code는 시작할 때 이 오설정을 보고합니다.

아래 예시는 두 개의 토큰을 마스킹합니다. `GH_TOKEN`은 `api.github.com`으로의 요청에만 대체되는 반면, `NPM_TOKEN`은 `injectHosts`가 없으며 `network.allowedDomains`의 모든 호스트로의 요청에 대해 대체됩니다. 각 `injectHosts` 항목 자체도 `network.allowedDomains`에 포함되어 있어야 합니다.

```json theme={null}
{
  "sandbox": {
    "enabled": true,
    "network": {
      "tlsTerminate": {},
      "allowedDomains": ["*.github.com", "registry.npmjs.org"]
    },
    "credentials": {
      "envVars": [
        { "name": "GH_TOKEN", "mode": "mask", "injectHosts": ["api.github.com"] },
        { "name": "NPM_TOKEN", "mode": "mask" }
      ]
    }
  }
}
```

`deny`와 달리 마스킹은 프록시가 나열된 호스트로 실제 자격 증명을 전송하도록 권한을 부여하므로 사용자 또는 관리자가 제어하는 설정(사용자 설정, 관리 설정, `--settings` CLI 플래그)에서만 수용됩니다. 리포지토리의 `.claude/settings.json` 또는 `.claude/settings.local.json`에 있는 `mask` 항목, `network.tlsTerminate` 및 [`credentials.allowPlaintextInject`](/docs/en/settings#sandbox-settings)는 무시됩니다.

동일한 변수가 임의의 범위에서 `deny`로 나열된 경우 `deny`가 우선합니다.

## 샌드박스 작동 방식

### 파일 시스템 격리

샌드박스 Bash 도구는 파일 시스템 접근을 특정 디렉터리로 제한합니다:

* **기본 쓰기 동작**: 현재 작업 디렉터리 및 그 하위 디렉터리, 그리고 `$TMPDIR`이 가리키는 세션 임시 디렉터리에 대한 읽기 및 쓰기 접근 권한
* **기본 읽기 동작**: 특정 거부된 디렉터리를 제외한 전체 컴퓨터에 대한 읽기 접근 권한. 이 기본값이 `~/.aws/credentials` 및 `~/.ssh/`와 같은 자격 증명 파일 읽기를 여전히 허용함에 유의하세요. 이러한 파일의 읽기를 차단하고 보안 환경 변수를 해제하려면 [`sandbox.credentials`](#protect-credentials)를 사용하거나 `denyRead`에 경로를 추가하세요.
* **차단된 접근**: 명시적인 권한 없이는 `~/.bashrc`와 같은 쉘 구성 파일 및 `/bin/`의 시스템 바이너리를 포함하여 현재 작업 디렉터리 및 세션 임시 디렉터리 외부의 파일을 수정할 수 없음
* **Git 워크트리**: 작업 디렉터리가 [연결된 git 워크트리](/docs/en/worktrees)인 경우, 샌드박스는 메인 리포지토리의 공유 `.git` 디렉터리에 대한 쓰기도 허용하여 `git commit`과 같은 명령이 ref 및 인덱스를 업데이트할 수 있게 합니다. 해당 디렉터리 내부의 `hooks/` 및 `config`에 대한 쓰기는 계속 거부됩니다.
* **구성 가능**: 설정을 통해 커스텀 허용 및 거부 경로 정의

설정에서 `sandbox.filesystem.allowWrite`를 사용하여 추가 경로에 대한 쓰기 접근 권한을 부여할 수 있습니다. 이러한 제한은 OS 수준에서 적용되므로 Claude의 파일 도구뿐만 아니라 `kubectl`, `terraform`, `npm`과 같은 도구를 포함한 모든 하위 프로세스 명령에 적용됩니다. 네트워크 격리를 유지하면서 파일 시스템 격리를 완전히 건너뛰려면 [`sandbox.filesystem.disabled`](#disable-filesystem-isolation)를 설정하세요.

### 네트워크 격리

네트워크 접근은 샌드박스 외부에서 실행되는 프록시 서버를 통해 제어됩니다:

* **도메인 제한**: 기본적으로 사전 허용된 도메인이 없습니다. 명령이 새로운 도메인을 처음으로 필요로 할 때 Claude Code가 승인을 요청합니다. {/* min-version: 2.1.191 */}v2.1.191부터 'Yes'를 선택하면 현재 세션의 나머지 기간 동안 해당 호스트가 허용되므로 동일한 호스트에 대한 이후 연결은 프롬프트를 다시 표시하지 않습니다. 프롬프트를 완전히 피하려면 [`allowedDomains`](/docs/en/settings#sandbox-settings)로 도메인을 사전 허용하세요. [권한 규칙](#permission-rules)에 설명된 대로 `WebFetch` 허용 규칙도 도메인을 사전 허용합니다.
* **관리형 잠금**: 관리 설정에서 [`allowManagedDomainsOnly`](/docs/en/settings#sandbox-settings)가 설정된 경우, 허용되지 않은 도메인은 프롬프트 대신 자동으로 차단되며 관리 설정의 `allowedDomains` 및 `WebFetch(domain:...)` 허용 규칙만 준수됩니다.
* **커스텀 프록시 지원**: 고급 사용자는 아웃바운드 트래픽에 커스텀 규칙을 구현할 수 있습니다.
* **광범위한 적용**: 제한 사항은 명령에 의해 생성된 모든 스크립트, 프로그램 및 하위 프로세스에 적용됩니다.

<Note>
  내장 프록시는 요청된 호스트 이름을 기반으로 허용 목록을 강제 적용하며 기본적으로 TLS 트래픽을 종료하거나 검사하지 않습니다. {/* min-version: 2.1.199 */}Claude Code v2.1.199 이상에서 사용할 수 있는 실험적인 [`network.tlsTerminate`](/docs/en/settings#sandbox-settings) 설정을 사용하면 내장 프록시가 TLS 자체를 종료할 수 있으며, 이는 [`mask` 자격 증명 항목](#protect-credentials)에 필요합니다. 기본값의 영향은 [보안 제한 사항](#security-limitations)을, 위협 모델에 TLS 검사가 필요한 경우 [커스텀 프록시 구성](#custom-proxy-configuration)을 참조하세요.
</Note>

### OS 수준의 강제 적용

샌드박스 Bash 도구는 운영체제 보안 프리미티브를 활용합니다:

* **macOS**: 샌드박스 강제 적용을 위해 Seatbelt 사용
* **Linux**: 격리를 위해 [bubblewrap](https://github.com/containers/bubblewrap) 사용
* **WSL2**: Linux와 동일하게 bubblewrap 사용

WSL1은 bubblewrap에 WSL2에서만 가능한 커널 기능이 필요하기 때문에 지원되지 않습니다. 이러한 OS 수준의 제한은 Claude Code의 명령에 의해 생성된 모든 자식 프로세스가 동일한 보안 경계를 상속하도록 보장합니다.

동일한 프리미티브가 독립형 [`@anthropic-ai/sandbox-runtime`](https://github.com/anthropic-experimental/sandbox-runtime) 패키지로 제공되며, 이는 [Sandbox environments](/docs/en/sandbox-environments#sandbox-runtime) 페이지에서 전체 Claude Code 프로세스를 감싸는 별도의 접근 방식으로 다룹니다.

## 샌드박스와 권한 및 권한 모드의 관계

샌드박스, [권한 규칙](/docs/en/permissions) 및 [권한 모드](/docs/en/permission-modes)는 상호 보완적인 레이어입니다. 아래 섹션에서는 샌드박스가 각 항목과 상호작용하는 방식을 다룹니다.

### 권한 규칙

권한 규칙과 샌드박스는 서로 다른 대상을 제어합니다:

* **권한 규칙**은 Claude Code가 사용할 수 있는 도구를 제어하며 모든 도구가 실행되기 전에 평가됩니다. 모든 도구(Bash, Read, Edit, WebFetch, MCP 등)에 적용되며, 다른 도구가 남아있는 동안 deny 또는 ask 규칙이 [`EndConversation`](/docs/en/tools-reference#endconversation-tool-behavior)을 차단할 수 없다는 점만 예외입니다.
* **샌드박스**는 파일 시스템 및 네트워크 수준에서 Bash 명령이 접근할 수 있는 대상을 제한하는 OS 수준의 강제 적용을 제공합니다. Bash 명령과 그 자식 프로세스에만 적용됩니다.

두 레이어는 강제 적용 방식에서도 다릅니다. Claude Code는 명령 문자열과 오토 모드에서 명령이 안전한지 여부에 대한 별도 분류기의 판단에 기초하여 명령이 실행되기 전에 권한 결정을 평가합니다. 운영체제는 실행 중인 프로세스에 샌드박스 경계를 강제 적용하므로, 모델이 실행하기로 선택한 내용과 관계없이, 그리고 허용된 명령이이름이 시사하는 것보다 더 많은 작업을 수행하더라도 샌드박스 경계가 유지됩니다.

파일 시스템 및 네트워크 제한은 샌드박스 설정과 권한 규칙 모두를 통해 구성됩니다:

| 설정 또는 규칙 | 수행하는 작업 |
| :--------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------ |
| `sandbox.filesystem.allowWrite` | 하위 프로세스에 작업 디렉터리 외부 경로에 대한 쓰기 접근 권한 부여 |
| `sandbox.filesystem.denyWrite` 및 `sandbox.filesystem.denyRead` | 특정 경로에 대한 하위 프로세스 접근 차단 |
| `sandbox.filesystem.allowRead` | `denyRead` 영역 내의 특정 경로 읽기를 다시 허용 |
| [`sandbox.filesystem.disabled`](#disable-filesystem-isolation) | {/* min-version: 2.1.216 */}네트워크 격리를 유지하면서 파일 시스템 레이어를 완전히 끔; Claude Code v2.1.216 이상 필요 |
| `Edit` 허용 규칙 | `sandbox.filesystem.allowWrite`와 동일한 방식으로 특정 경로에 대한 쓰기 접근 권한 부여 |
| `Read` 및 `Edit` 거부 규칙 | 특정 파일 또는 디렉터리에 대한 접근 차단 |
| `WebFetch` 허용 및 거부 규칙 | 도메인 접근 제어 |
| 샌드박스 `allowedDomains` | Bash 명령이 도달할 수 있는 도메인 제어 |
| 샌드박스 `deniedDomains` | 더 넓은 `allowedDomains` 와일드카드가 허용하는 경우에도 특정 도메인 차단 |

`sandbox.filesystem` 설정과 권한 규칙의 경로는 최종 샌드박스 구성으로 함께 병합됩니다.

[claude-code 리포지토리의 예시 디렉터리](https://github.com/anthropics/claude-code/tree/main/examples/settings)에는 샌드박스 전용 예시를 포함하여 일반적인 배포 시나리오를 위한 기본 설정 구성이 포함되어 있습니다. 이를 출발점으로 사용하고 필요에 맞게 조정하세요.

### 권한 모드

`/sandbox`는 [권한 모드](/docs/en/permission-modes)가 아닙니다. 권한 모드는 도구 호출을 실행할지 여부와 사전에 프롬프트를 표시할지 여부를 결정하는 반면, 샌드박스는 실행된 Bash 명령이 접근할 수 있는 대상을 제한합니다. 두 가지는 제어 대상과 작업별 프롬프트를 대체하는 항목이 다릅니다:

| | 제어 대상 | 프롬프트를 대체하는 항목 |
| :----------------------------------------------------------------- | :------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `/sandbox` | 실행된 Bash 명령이 접근할 수 있는 대상 | [auto-allow 모드](#sandbox-modes)에서의 샌드박스 경계 자체 |
| [Auto mode](/docs/en/permission-modes#eliminate-prompts-with-auto-mode) | 각 도구 호출의 실행 여부 | 작업을 검토하는 분류기 |
| `--dangerously-skip-permissions` | 각 도구 호출의 실행 여부 | 없음. [보호된 경로](/docs/en/permission-modes#protected-paths) 검사도 건너뜁니다. 명시적인 [ask rules](/docs/en/permissions#manage-permissions), [조직에서 `ask`로 설정한](/docs/en/mcp#organization-controls-on-connector-tools) 커넥터 도구, [`requiresUserInteraction`](/docs/en/mcp#require-approval-for-a-specific-tool)으로 표시된 MCP 도구, 그리고 `/` 또는 홈 디렉터리 삭제에 대해서만 프롬프트합니다 |

샌드박스의 [auto-allow 모드](#sandbox-modes)는 [auto 모드](/docs/en/permission-modes#eliminate-prompts-with-auto-mode)와 별개입니다. auto-allow는 샌드박스 경계가 명령을 포함하고 있기 때문에 Bash 명령을 승인하지만, auto 모드는 분류기를 사용하여 작업을 검토합니다. 두 가지는 독립적으로 작동하며 조합될 수 있습니다. 무인 실행을 위한 격리 경계를 선택하려면 [Sandbox environments](/docs/en/sandbox-environments#how-isolation-relates-to-permission-modes)를 참조하세요.

## 조직을 위한 샌드박스 구성하기

관리자는 모든 사용자에게 샌드박스를 사용하도록 요구하고, 개발자가 정책을 넓히지 못하도록 방지하며, 샌드박스 트래픽을 기업 프록시로 라우팅할 수 있습니다.

### 관리 설정으로 샌드박스 강제 적용

모든 개발자에게 샌드박스를 요구하려면 MDM에서 관리하는 파일로 또는 Claude.ai의 [서버 관리 설정](/docs/en/server-managed-settings)을 통해 [관리 설정](/docs/en/settings#settings-files)으로 `sandbox` 키를 전달하세요.

다음 관리 설정 구성은 샌드박스를 활성화하고, 샌드박스를 초기화할 수 없는 경우 Claude Code 시작을 거부하며, 모델이 샌드박스 외부에서 명령을 재시도하지 못하도록 방지합니다:

```json theme={null}
{
  "sandbox": {
    "enabled": true,
    "failIfUnavailable": true,
    "allowUnsandboxedCommands": false
  }
}
```

`enabled` 이외의 두 키는 샌드박스가 명령을 실행할 수 없을 때 발생하는 상황을 제어합니다:

* **`failIfUnavailable`**: Linux의 bubblewrap과 같이 누락된 종속성이 있는 경우 경고를 표시하고 샌드박스 해제 실행으로 전환하는 대신 Claude Code의 시작을 차단합니다.
* **`allowUnsandboxedCommands: false`**: `dangerouslyDisableSandbox` 탈출구가 무시되므로 샌드박스 아래에서 실패한 명령은 샌드박스 외부에서 재시도될 수 없습니다.

두 가지를 함께 고려할 가치가 있는 추가 사항이 있습니다. 격리 없이 실행되어야 하는 조직 승인 도구에 대해 `excludedCommands`를 추가하세요. 기본 읽기 정책이 자격 증명을 여전히 허용하므로 `~/.aws` 및 `~/.ssh`와 같은 자격 증명 디렉터리 및 보안 환경 변수에 대해 [`sandbox.credentials`](#protect-credentials) 항목을 추가하세요.

샌드박스는 네이티브 Windows에서 실행되지 않으므로 환경에 Windows 호스트가 포함되어 있는 경우 이 구성을 macOS 및 Linux로 범위를 좁히거나 해당 사용자가 WSL2 또는 컨테이너 내부에서 Claude Code를 실행하도록 하세요.

### 개발자가 정책을 넓히지 못하도록 방지

`enabled` 및 `failIfUnavailable`과 같은 부울 키의 경우 Claude Code는 관리형 값을 사용하고 개발자가 로컬로 설정한 모든 항목을 무시합니다. `excludedCommands` 및 `allowRead`와 같은 배열 키의 경우 Claude Code는 모든 범위의 항목을 병합하므로 개발자가 정책을 넓히는 항목을 덧붙일 수 있습니다.

관리 설정에서 `allowManagedReadPathsOnly`를 `true`로 설정하여 관리 설정의 `allowRead` 항목만 준수되도록 하세요. 사용자, 프로젝트 및 로컬 `allowRead` 항목은 무시됩니다. 이는 개발자가 조직에서 승인한 경로 이상으로 읽기 접근 권한을 넓히는 것을 방지합니다. 네트워크 도메인을 동일한 방식으로 관리형 값으로 잠그려면 [`allowManagedDomainsOnly`](/docs/en/settings#sandbox-settings)를 설정하세요.

관리 설정이 `sandbox.filesystem`을 구성하거나 `sandbox.credentials.files` 항목을 나열하는 경우, 관리 설정만 [`filesystem.disabled`](#disable-filesystem-isolation)를 설정할 수 있으므로 개발자가 관리자가 배포한 파일 시스템 제한을 끌 수 없습니다.

`excludedCommands`에는 동등한 관리 전용 잠금이 없으므로 개발자는 항상 샌드박스 외부에서 추가 명령을 실행하는 항목을 덧붙일 수 있습니다. 관리 목록을 좁게 유지하세요.

### 커스텀 프록시 구성

고급 네트워크 보안이 필요한 조직의 경우 커스텀 프록시를 구현하여 다음을 수행할 수 있습니다:

* HTTPS 트래픽 암호 해독 및 검사
* 커스텀 필터링 규칙 적용
* 모든 네트워크 요청 기록
* 기존 보안 인프라와 통합

Claude Code가 프록시를 가리키도록 하려면 [샌드박스 설정](/docs/en/settings#sandbox-settings)에 프록시 포트를 설정하세요:

```json theme={null}
{
  "sandbox": {
    "network": {
      "httpProxyPort": 8080,
      "socksProxyPort": 8081
    }
  }
}
```

## 문제 해결

일부 명령은 샌드박스 외부에서 작동하더라도 샌드박스 내부에서는 실패합니다. 아래의 해결 방법은 가장 일반적인 경우를 다룹니다.

* **명령이 host-not-allowed 오류로 실패함**: 많은 CLI 도구가 특정 호스트에 도달해야 합니다. 프롬프트가 표시될 때 권한을 부여하면 해당 호스트가 허용 목록에 추가되어 향후 도구가 샌드박스 내부에서 실행됩니다.
* **`jest`가 중단되거나 실패함**: `watchman`은 샌드박스와 호환되지 않습니다. 대신 `jest --no-watchman`을 실행하세요.
* **macOS에서 Go 기반 CLI가 TLS 검증에 실패함**: `gh`, `gcloud`, `terraform`과 같은 도구는 Seatbelt 아래에서 TLS 검증에 실패할 수 있습니다. 이러한 도구를 `excludedCommands`에 나열하여 샌드박스 외부에서 실행하세요. MITM 프록시 및 커스텀 CA와 함께 `httpProxyPort`를 사용하는 경우 대신 [`enableWeakerNetworkIsolation`](/docs/en/settings#sandbox-settings)을 `true`로 설정하세요.
* **`open`, `osascript` 또는 브라우저 기반 인증 흐름이 macOS에서 `-600` 오류로 실패함**: 샌드박스는 기본적으로 Apple Events를 차단합니다. 이를 허용하려면 사용자, 관리자 또는 CLI 설정에서 [`allowAppleEvents`](/docs/en/settings#sandbox-settings)를 `true`로 설정하세요. 프로젝트 설정은 이 키에 대해 무시됩니다. 이를 활성화하면 샌드박스 처리된 명령이 사용자 프롬프트 없이 샌드박스 해제 상태로 다른 애플리케이션을 시작할 수 있고 macOS 자동화 동의 프롬프트(TCC)에 따라 실행 중인 애플리케이션에 AppleScript 명령을 보낼 수 있으므로 코드 실행 격리가 해제됩니다. 또는 샌드박스 외부에서 실행되도록 해당 명령을 `excludedCommands`에 추가하세요.
* **`docker` 명령 실패**: `docker`는 샌드박스와 호환되지 않습니다. 샌드박스 외부에서 실행되도록 `excludedCommands`에 `docker *`를 추가하세요.
* **컨테이너 내부에서 Bubblewrap 시작 실패**: 비권한 컨테이너에서 bubblewrap은 새로운 `/proc` 파일 시스템을 마운트할 수 없습니다. 내부 샌드박스가 컨테이너의 기존 `/proc`을 바인드 마운트하도록 [`enableWeakerNestedSandbox`](/docs/en/settings#sandbox-settings)를 `true`로 설정하세요. 외부 컨테이너가 이미 필요한 격리 경계를 제공하는 경우에만 이 설정을 사용하세요. 새로운 `/proc` 마운트가 숨기는 프로세스 정보가 샌드박스 처리된 명령에 노출되기 때문입니다.
* **Linux에서의 Seccomp 필터**: Unix 도메인 소켓을 차단하려면 seccomp 필터가 필요합니다. `/sandbox` 의 Dependencies 탭은 종속성이 누락되었을 때만 표시됩니다. 재시작 후 탭이 보이지 않으면 필터가 이미 설치된 것입니다. 필터가 누락된 경우 `npm install -g @anthropic-ai/sandbox-runtime`을 실행하여 헬퍼를 설치한 다음 Claude Code를 재시작하여 시작 종속성 검사가 이를 감지하도록 하세요.
* **root 사용자로서 `--dangerously-skip-permissions` 실패**: Linux 및 macOS에서 root로 또는 sudo를 통해 실행할 때 이 플래그가 차단됩니다. root 접근 권한과 권한 프롬프트 부재가 결합되면 시스템의 모든 파일이나 서비스를 수정할 수 있기 때문입니다. 인식된 샌드박스 내부에서는 이 검사가 자동으로 건너뛰어집니다. 컨테이너에서 자율적으로 실행하려면 Claude Code를 비-root 사용자로 실행하는 [dev container](/docs/en/devcontainer) 구성을 사용하세요.

## 제한 사항

샌드박스는 위험을 줄이지만 완전한 격리 경계는 아닙니다. 하드 보안 제어로 의존하기 전에 아래 제한 사항을 검토하세요.

### 보안 제한 사항

* **네트워크 필터링**: 샌드박스는 프로세스가 연결할 수 있는 도메인을 제한합니다. 기본적으로 내장 프록시는 아웃바운드 트래픽에서 TLS를 종료하거나 검사하지 않으므로 암호화된 연결 내용이 검사되지 않습니다. 실험적인 [`network.tlsTerminate`](/docs/en/settings#sandbox-settings) 설정은 [`mask` 자격 증명 대체](#protect-credentials)를 위해 프록시에서 TLS를 종료하지만 콘텐츠 필터링을 추가하지는 않습니다. 정책에서 신뢰할 수 있는 도메인만 허용되도록 하는 것은 사용자의 책임입니다.

<Warning>
  `github.com`과 같은 광범위한 도메인을 허용하면 데이터 유출 경로가 생성될 수 있습니다. 프록시는 TLS를 검사하지 않고 클라이언트가 제공한 호스트 이름으로 허용 여부를 결정하므로, 샌드박스 내부에서 실행되는 코드가 [도메인 프론팅(domain fronting)](https://en.wikipedia.org/wiki/Domain_fronting) 또는 유사한 기술을 사용하여 허용 목록 외의 호스트에 도달할 가능성이 있습니다. 위협 모델에 더 강력한 보장이 필요한 경우 TLS를 종료하고 트래픽을 검사하는 [커스텀 프록시](#custom-proxy-configuration)를 구성하고 샌드박스 내부에 CA 인증서를 설치하세요. TLS를 인식하는 더 강력한 네트워크 격리는 활발히 개발 중인 분야입니다.
</Warning>

* **Unix 소켓을 통한 권한 승격**: `allowUnixSockets` 구성은 실수로 강력한 시스템 서비스에 대한 접근 권한을 부여하여 샌드박스 우회로 이어질 수 있습니다. 예를 들어 `/var/run/docker.sock`에 대한 접근을 허용하면 Docker 소켓을 통해 호스트 시스템에 사실상 접근할 수 있게 됩니다. 샌드박스를 통해 허용하는 모든 Unix 소켓을 신중하게 고려하세요.
* **파일 시스템 권한 승격**: 과도하게 광범위한 파일 시스템 쓰기 권한은 권한 승격 공격을 가능하게 할 수 있습니다. `$PATH`의 실행 파일이 포함된 디렉터리, 시스템 구성 디렉터리, 또는 `.bashrc`나 `.zshrc`와 같은 사용자 쉘 구성 파일에 대한 쓰기를 허용하면 다른 사용자나 시스템 프로세스가 이러한 파일에 접근할 때 서로 다른 보안 컨텍스트에서 코드가 실행될 수 있습니다.
* **Linux 샌드박스 강도**: Linux 구현은 강력한 파일 시스템 및 네트워크 격리를 제공하지만, 권한 있는 네임스페이스 없이 Docker 환경 내부에서 작동하거나 비권한 사용자 네임스페이스가 sysctl에 의해 비활성화된 Linux 호스트에서 작동할 수 있도록 하는 `enableWeakerNestedSandbox` 모드를 포함합니다. 이 옵션은 보안을 상당히 약화시키므로 그렇지 않은 경우 추가 격리가 강제 적용될 때만 사용해야 합니다.
* **macOS에서의 Apple Events**: macOS 샌드박스는 기본적으로 Apple Events를 차단합니다. `allowAppleEvents` 설정은 이 제한을 해제하여 `open` 및 `osascript`와 같은 도구가 작동하도록 하지만 코드 실행 격리를 제거합니다. 샌드박스 처리된 명령은 사용자 프롬프트 없이 샌드박스 해제 상태로 다른 애플리케이션을 시작할 수 있으며 앱별 macOS 자동화 동의 프롬프트(TCC)에 따라 실행 중인 애플리케이션에 AppleScript 명령을 보낼 수 있습니다. 사용자, 관리자 또는 CLI 설정에서만 수용됩니다. 프로젝트 설정으로는 이를 활성화할 수 없습니다.
* **보호되는 설정 파일**: 샌드박스는 모든 범위의 Claude Code `settings.json` 파일 및 관리 설정 디렉터리에 대한 쓰기 접근을 자동으로 거부하므로, 샌드박스 처리된 명령은 이러한 거부 규칙을 끄는 [파일 시스템 격리를 비활성화](#disable-filesystem-isolation)하지 않는 한 자체 정책을 수정할 수 없습니다. {/* min-version: 2.1.210 */}거부 규칙은 심볼릭 링크를 해제합니다. 시작 후 보호된 설정 파일 경로에 심볼릭 링크가 나타나면 샌드박스는 다음 명령을 위해 대상 경로를 거부 목록에 추가하므로 연결된 설정 파일을 링크를 통해 편집할 수 없습니다. v2.1.210 이전에는 거부 규칙이 심볼릭 링크를 해제하지 않았습니다.

### 플랫폼 및 도구 호환성

* **플랫폼 지원**: macOS, Linux 및 WSL2를 지원합니다. WSL1 및 네이티브 Windows는 지원되지 않습니다.
* **성능 오버헤드**: 최소화되어 있지만 일부 파일 시스템 작업이 약간 느려질 수 있습니다.
* **도구 호환성**: 특정 시스템 접근 패턴이 필요한 일부 도구는 구성 조정이 필요하거나 샌드박스 외부에서 실행해야 할 수 있습니다.

### 범위 (Scope)

샌드박스는 Bash 하위 프로세스를 격리합니다. 다른 도구는 서로 다른 경계 하에서 작동합니다:

* **내장 파일 도구**: Read, Edit 및 Write는 샌드박스를 통해 실행되지 않고 권한 시스템을 직접 사용합니다. [permissions](/docs/en/permissions)를 참조하세요.
* **Computer use**: Claude가 앱을 열고 화면을 제어할 때 격리된 환경이 아니라 실제 데스크톱에서 실행됩니다. 앱별 권한 프롬프트가 각 애플리케이션을 제어합니다. [computer use in the CLI](/docs/en/computer-use) 또는 [computer use in Desktop](/docs/en/desktop#let-claude-use-your-computer)을 참조하세요.
* **환경 변수**: 샌드박스 처리된 Bash 명령은 기본적으로 거기에 설정된 자격 증명을 포함하여 부모 프로세스 환경을 상속받습니다. 샌드박스 명령에 대해 특정 변수를 해제하거나 마스킹하려면 [`sandbox.credentials`](#protect-credentials)를 사용하고, 모든 하위 프로세스에서 Anthropic 및 클라우드 제공업체 자격 증명을 제거하려면 [`CLAUDE_CODE_SUBPROCESS_ENV_SCRUB`](/docs/en/env-vars)를 설정하세요.
* **서브에이전트**: [서브에이전트](/docs/en/sub-agents)는 부모 세션과 동일한 프로세스에서 실행되며 동일한 샌드박스 구성을 사용합니다. 서브에이전트 내부의 Bash 명령은 부모 세션에서 샌드박스가 활성화되어 있을 때 샌드박스 처리됩니다.

<Warning>
  효과적인 샌드박스 적용을 위해서는 파일 시스템 및 네트워크 격리가 모두 필요합니다. 네트워크 격리가 없으면 손상된 에이전트가 SSH 키와 같은 민감한 파일을 유출할 수 있습니다. 관대 한 정책이나 [파일 시스템 레이어 비활성화](#disable-filesystem-isolation)로 인해 파일 시스템 격리가 없으면, 손상된 에이전트가 시스템 리소스에 백도어를 설치하여 네트워크 접근 권한을 얻을 수 있습니다. 기본값을 넓힐 때 `allowWrite` 경로, 넓은 `allowedDomains` 항목 또는 `excludedCommands` 예외가 반대편의 제한을 취소하지 않는지 확인하세요.
</Warning>

## 참고 항목

* [Sandbox environments](/docs/en/sandbox-environments): 내장 샌드박스를 개발 컨테이너, 컨테이너 및 VM과 비교
* [Security](/docs/en/security): 포괄적인 보안 기능 및 모범 사례
* [Permissions](/docs/en/permissions): 권한 구성 및 접근 제어
* [Settings](/docs/en/settings): 전체 구성 참조 문서
* [CLI reference](/docs/en/cli-reference): 명령줄 옵션
