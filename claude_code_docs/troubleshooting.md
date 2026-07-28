> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 문제 해결 (Troubleshooting)

> Claude Code에서 높은 CPU 또는 메모리 사용량, 멈춤 현상, 자동 요약(auto-compact) 래싱 및 검색 문제를 해결하고 기타 문제에 대한 적절한 페이지를 찾으세요.

이 페이지는 Claude Code가 실행된 후의 성능, 안정성 및 검색 문제를 다룹니다. 다른 문제의 경우 막힌 부분과 일치하는 페이지에서 시작하세요:

| 증상 | 이동할 페이지 |
| :--------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------- |
| `command not found`, 설치 실패, PATH 문제, `EACCES`, TLS 오류 | [Troubleshoot installation and login](/docs/en/troubleshoot-install) |
| `The connection dropped while downloading the update` 또는 `aborted` 오류와 함께 업데이트 또는 설치 다운로드 실패 | [Error reference](/docs/en/errors#the-connection-dropped-while-downloading-the-update) |
| 로그인 루프, OAuth 오류, `403 Forbidden`, "organization disabled", Amazon Bedrock, Google Cloud's Agent Platform, 또는 Microsoft Foundry 자격 증명 | [Troubleshoot installation and login](/docs/en/troubleshoot-install#login-and-authentication) |
| 설정 미적용, 훅 미작동, MCP 서버 로드 불가 | [Debug your configuration](/docs/en/debug-your-config) |
| `API Error: 5xx`, `529 Overloaded`, `429`, 요청 검증 오류 | [Error reference](/docs/en/errors) |
| `model not found` 또는 `you may not have access to it` | [Error reference](/docs/en/errors#theres-an-issue-with-the-selected-model) |
| VS Code 확장 프로그램이 연결되지 않거나 Claude를 감지하지 못함 | [VS Code integration](/docs/en/vs-code#fix-common-issues) |
| JetBrains 플러그인 또는 IDE가 감지되지 않음 | [JetBrains integration](/docs/en/jetbrains#troubleshooting) |
| 높은 CPU 또는 메모리 사용량, 느린 응답, 멈춤, 검색에서 파일을 찾지 못함 | 아래의 [성능 및 안정성](#performance-and-stability) 참조 |

어느 것이 적용되는지 확실하지 않은 경우 Claude Code 내부에서 `/doctor`를 실행하여 설치, 설정, 확장 프로그램 및 컨텍스트 사용량을 자동으로 검사하세요. 확인 후 적용할 수 있는 수정 사항을 제안합니다. `claude`가 전혀 시작되지 않으면 쉘에서 `claude doctor`를 대신 실행하세요. MCP 서버 상태를 확인하려면 `/mcp`를 실행하세요.

## 성능 및 안정성

이 섹션에서는 리소스 사용량, 응답성 및 검색 동작과 관련된 문제를 다룹니다.

### 높은 CPU 또는 메모리 사용량

Claude Code는 대부분의 개발 환경에서 작동하도록 설계되었지만 대규모 코드베이스를 처리할 때 상당한 리소스를 소비할 수 있습니다. 성능 문제가 발생하는 경우:

1. 정기적으로 `/compact`를 사용하여 컨텍스트 크기 줄이기
2. 주요 작업 사이에 Claude Code를 닫고 다시 시작하기
3. `.gitignore` 파일에 대형 빌드 디렉터리를 추가하는 방안 고려하기
4. 플러그인, MCP 서버 또는 훅이 원인인지 확인하기 위해 [`claude --safe-mode`](/docs/en/cli-reference#cli-flags)로 다시 시작하기. 세션의 모든 커스터마이징을 비활성화합니다. 사용량이 감소하면 [Debug your configuration](/docs/en/debug-your-config#test-against-a-clean-configuration)을 참조하여 원인을 찾으세요.

이러한 단계 후에도 메모리 사용량이 높게 유지되면 `/heapdump`를 실행하여 JavaScript 힙 스냅샷과 메모리 분석을 `~/Desktop`에 작성하세요. Desktop 폴더가 없는 Linux에서는 파일이 홈 디렉터리에 작성됩니다.

분석에는 RSS(resident set size), JS 힙, 배열 버퍼, 추적되지 않은 네이티브 메모리가 표시되므로 증가가 JavaScript 개체에 있는지 또는 네이티브 코드에 있는지 확인하는 데 도움이 됩니다. 리테이너(retainers)를 검사하려면 Chrome DevTools의 Memory → Load에서 `.heapsnapshot` 파일을 열세요. 분석 결과는 `-diagnostics.json`으로 끝나는 파일입니다.

<Warning>
  `.heapsnapshot` 파일에는 프로세스의 모든 문자열이 포함되어 있습니다. 공개 이슈에 첨부하거나 공유하지 마세요. [GitHub](https://github.com/anthropics/claude-code/issues)에 메모리 문제를 보고할 때는 `-diagnostics.json` 파일만 첨부하세요. 해당 파일에는 메모리 통계가 포함되어 있으며 대화 내용이나 자격 증명은 포함되어 있지 않습니다.
```
</Warning>

### 대형 테이블이 터미널에서 잘림

200행이 넘는 Markdown 테이블은 처음 200행을 렌더링한 후 `… N more rows not shown` 라인을 렌더링합니다. 표시만 제한되며 전체 테이블은 대화 내용에 그대로 남아있고 [`/copy`](/docs/en/commands)는 모든 행을 복사합니다. 터미널에서 읽기에는 너무 큰 테이블의 경우 Claude에게 대신 파일로 작성하도록 요청하세요. v2.1.208 이전에는 Claude Code가 모든 행을 렌더링했으므로 매우 큰 테이블이 포함된 세션을 재개하면 다시 렌더링하는 동안 멈출 수 있었습니다.

### 자동 요약(Auto-compaction)이 래싱 오류로 중지됨

`Autocompact is thrashing: the context refilled to the limit...`가 보인다면 자동 요약에는 성공했지만 파일 또는 도구 출력이 연속으로 여러 번 즉시 컨텍스트 창을 다시 채운 것입니다. Claude Code는 진전이 없는 루프에서 API 호출을 낭비하지 않도록 재시도를 중지합니다.

복구하려면:

1. Claude에게 전체 파일 대신 특정 줄 범위나 함수와 같이 더 작은 단위로 대형 파일을 읽도록 요청하기
2. 큰 출력을 삭제하는 초점으로 `/compact` 실행하기(예: `/compact keep only the plan and the diff`)
3. 대형 파일 작업을 별도의 컨텍스트 창에서 실행되도록 [서브에이전트](/docs/en/sub-agents)로 이동하기
4. 이전 대화가 더 이상 필요하지 않은 경우 `/clear` 실행하기

### 명령이 멈추거나 응답하지 않음

Claude Code가 응답하지 않는 것처럼 보이면:

1. Ctrl+C를 눌러 현재 작업을 취소해 보세요.
2. 응답이 없으면 터미널을 닫고 다시 시작해야 할 수도 있습니다.

다시 시작해도 대화는 손실되지 않습니다. 세션을 다시 시작하려면 동일한 디렉터리에서 `claude --resume`을 실행하세요.

### 에디터의 통합 터미널에서 텍스트가 깨지거나 손상됨

VS Code, Cursor 또는 Devin Desktop 통합 터미널에서 Claude Code를 실행할 때 문자가 상자로 렌더링되거나 번지거나 잘못된 글리프로 표시되는 경우 터미널의 GPU 렌더러가 원인일 가능성이 높습니다. Claude Code 내부에서 `/terminal-setup`을 실행하여 `terminal.integrated.gpuAcceleration`을 `"off"`로 설정하거나 에디터 설정에서 수동으로 설정하고 창을 다시 로드하세요. `/terminal-setup`이 작성하는 다른 설정은 [Terminal configuration](/docs/en/terminal-config)을 참조하세요.

### 검색 및 탐색 문제

Search 도구, `@file` 멘션, 커스텀 에이전트 또는 커스텀 skill이 파일을 찾지 못하는 경우 번들로 제공된 `ripgrep` 바이너리가 시스템에서 실행되지 않는 것일 수 있습니다. 해당 플랫폼의 `ripgrep` 패키지를 설치하고 Claude Code에 대신 해당 패키지를 사용하도록 지정하세요:

<Tabs>
  <Tab title="macOS">
    ```bash theme={null}
    brew install ripgrep
    ```
  </Tab>

  <Tab title="Ubuntu/Debian">
    ```bash theme={null}
    sudo apt install ripgrep
    ```
  </Tab>

  <Tab title="Alpine">
    ```bash theme={null}
    apk add ripgrep
    ```

    `ripgrep`은 Alpine의 커뮤니티 리포지토리에 있습니다. `apk`에서 패키지가 누락되었다고 보고하는 경우 [Alpine Linux setup](/docs/en/setup#alpine-linux-and-musl-based-distributions)을 참조하세요.
  </Tab>

  <Tab title="Arch">
    ```bash theme={null}
    pacman -S ripgrep
    ```
  </Tab>

  <Tab title="Windows">
    ```powershell theme={null}
    winget install BurntSushi.ripgrep.MSVC
    ```
  </Tab>
</Tabs>

그런 다음 [환경 변수](/docs/en/env-vars)에 `USE_BUILTIN_RIPGREP=0`을 설정하세요. 전환이 적용되었는지 확인하려면 터미널에서 `claude doctor`를 실행하고 Search 라인에 `OK (bundled)` 대신 시스템 ripgrep 경로가 표시되는지 확인하세요.

### WSL에서 느리거나 불완전한 검색 결과

[WSL에서 파일 시스템을 교차하여 작업할 때](https://learn.microsoft.com/en-us/windows/wsl/filesystems) 발생하는 디스크 읽기 성능 저하로 인해 WSL에서 Claude Code를 사용할 때 예상보다 적은 일치 결과가 발생할 수 있습니다. 검색은 여전히 작동하지만 네이티브 파일 시스템보다 적은 결과를 반환합니다.

<Note>
  `claude doctor`는 이 경우 Search를 OK로 표시합니다.
</Note>

**해결 방법:**

1. **더 구체적인 검색 제출**: 디렉터리나 파일 유형을 지정하여 검색되는 파일 수를 줄이세요. 예: "Search for JWT validation logic in the auth-service package" 또는 "Find use of md5 hash in JS files".

2. **프로젝트를 Linux 파일 시스템으로 이동**: 가능한 경우 프로젝트가 Windows 파일 시스템(`/mnt/c/`)이 아닌 Linux 파일 시스템(`/home/`)에 있는지 확인하세요.

3. **대신 네이티브 Windows 사용**: 더 나은 파일 시스템 성능을 위해 WSL 대신 Windows에서 네이티브로 Claude Code를 실행하는 것을 고려해 보세요.

## 추가 도움받기

여기에 적용되지 않은 문제가 발생하는 경우:

1. 설정 점검을 위해 `/doctor`를 실행하고 MCP 서버 상태를 확인하기 위해 `/mcp`를 실행하세요.
2. 문제를 Anthropic에 직접 보고하려면 Claude Code 내부의 `/feedback` 명령을 사용하세요.
3. 알려진 문제는 [GitHub repository](https://github.com/anthropics/claude-code)를 확인하세요.
4. Claude에게 해당 기능 및 특성에 대해 직접 물어보세요. Claude는 자체 문서에 내장 접근 권한을 가지고 있습니다.

계정, 청구 또는 구독 문제의 경우 Anthropic 지원팀에 대신 문의하세요: [claude.ai](https://claude.ai) (Console 사용자는 [platform.claude.com](https://platform.claude.com))에 로그인하고 좌측 하단의 이니셜을 클릭한 후 **Get help**를 선택하세요. 각 플랜에서 인간 에이전트에 도달할 수 있는 사람을 포함한 전체 절차는 [How to get support](https://support.claude.com/en/articles/9015913-how-to-get-support)를 참조하세요.
