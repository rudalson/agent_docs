> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 조직을 위한 MCP 서버 액세스 제어하기

> 관리형 설정 파일, 허용 목록(allowlists) 및 거부 목록(denylists)을 사용하여 사용자가 추가하거나 연결할 수 있는 MCP 서버를 제한합니다.

기본적으로 Claude Code를 구동하는 사람은 누구나 본인이 선택한 [MCP 서버](/docs/en/mcp)에 연결할 수 있습니다. Anthropic은 커넥터를 [Anthropic Directory](https://claude.ai/directory)에 추가하기 전에 [등재 기준](https://claude.com/docs/connectors/building/review-criteria)에 따라 검토하지만, MCP 서버에 대해 보안 감사나 직접 관리를 수행하지는 않습니다. 관리자는 고정된 승인 세트를 배포하는 것부터 MCP를 완전히 비활성화하는 것까지 조직 내에서 구동되는 서버를 제한할 수 있습니다.

이 페이지에서는 다음 내용을 다룹니다:

* 필요한 제어 수준에 맞는 [패턴 선택하기](#choose-a-pattern)
* `managed-mcp.json`으로 [고정 서버 세트 배포하기](#exclusive-control-with-managed-mcp-json) ([MCP 완전히 비활성화하기](#disable-mcp-entirely) 포함)
* 허용 목록 및 거부 목록으로 [서버 제어하기](#policy-based-control-with-allowlists-and-denylists)
* 제약 조건으로 인해 서버가 차단되었을 때 [사용자에게 표시되는 내용 안내하기](#how-restrictions-appear-to-users)
* 조직에서 실제 사용 중인 [서버 모니터링하기](#monitor-mcp-usage)

<Note>
  [보안](/docs/en/security) 페이지에서는 MCP 위협 모델과 서버 승인 전 평가 방법을 다룹니다. [강제 적용 대상 결정하기](/docs/en/admin-setup#decide-what-to-enforce)에서는 다른 관리 제어 기능과 함께 MCP 제한을 설명합니다.
</Note>

## 패턴 선택하기

Claude Code는 다양한 제한 수준을 지원합니다. 각 패턴은 아래에서 다루는 메커니즘 중 하나 이상을 사용합니다: 고정 세트 배포를 위한 `managed-mcp.json`, 그리고 사용자가 구성한 내용을 필터링하기 위한 `allowedMcpServers`/`deniedMcpServers`.

| 패턴                    | 역할                                                                                       | 구성 방법                                                                                            |
| :---------------------- | :----------------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------- |
| **MCP 비활성화**        | 어디서도 서버가 로드되지 않음                                                               | 빈 서버 맵을 가진 `managed-mcp.json`                                                                 |
| **고정 배포**           | 모든 사용자가 동일한 서버를 제공받으며 다른 서버를 추가할 수 없음                           | 원하는 서버가 포함된 `managed-mcp.json`                                                              |
| **승인된 카탈로그**     | 승인된 서버 목록을 게시; 사용자는 원하는 서버를 추가하며 그 외에는 차단됨                   | `allowedMcpServers` + `allowManagedMcpServersOnly: true`                                             |
| **플러그인 서버 전용**  | 플러그인 제공 서버만 허용; 사용자가 직접 추가 불가                                         | `mcp`가 목록에 포함된 [`strictPluginOnlyCustomization`](/docs/en/settings#strictpluginonlycustomization) |
| **소프트 허용 목록**    | 사용자가 자체 설정에서 확장할 수 있는 허용 목록 강제 적용                                  | `allowManagedMcpServersOnly` 없이 `allowedMcpServers` 설정                                          |
| **거부 목록 전용**      | 알려진 악성 서버를 차단하고 나머지 전체 허용                                               | `deniedMcpServers`                                                                                   |
| **제한 없음**           | 사용자가 모든 항목 추가 가능                                                               | 관리형 MCP 구성을 배포하지 않음                                                                      |

<Note>
  Claude Code에는 사용자가 찾아보고 설치할 수 있는 내장 MCP 서버 레지스트리가 없습니다. 승인된 카탈로그 패턴의 경우 사내 위키 등 사용자가 볼 수 있는 곳에 승인 목록과 `claude mcp add` 명령어를 공유하거나, 사용자가 `/plugin`에서 탐색 및 설치할 수 있도록 [관리형 플러그인 마켓플레이스](/docs/en/plugin-marketplaces#managed-marketplace-restrictions)를 통해 플러그인 형태로 배포하세요.
</Note>

## managed-mcp.json을 통한 독점적 제어

`managed-mcp.json` 파일을 배포하면 Claude Code는 해당 파일이 정의한 서버만 로드합니다. 사용자는 플러그인이 제공하는 서버를 포함하여 다른 어떠한 MCP 서버도 추가, 수정 또는 사용할 수 없습니다. 이 파일은 [관리형 세트와 함께 허용하도록 설정](#allow-claude-ai-connectors-alongside-the-managed-set)하지 않는 한 claude.ai 커넥터도 억제(suppress)합니다.

두 가지 다른 설정으로 관리형 세트를 추가로 필터링할 수 있습니다:

* `allowedMcpServers` 및 `deniedMcpServers`는 관리형 서버에도 적용되므로 이를 통과하지 못한 관리형 서버는 로드되지 않습니다.
* 사용자의 자체 `deniedMcpServers`는 사용자 설정에서 병합되므로, 사용자가 스스로를 위해 관리형 서버를 차단할 수 있습니다.

전체 검사 순서는 [서버 평가 방식](#how-a-server-is-evaluated)을 참조하세요.

`managed-mcp.json`은 독립된 파일이므로 [서버 관리형 설정](/docs/en/server-managed-settings)을 통해 전달될 수 없습니다. 관리자 권한으로 시스템 경로에 쓸 수 있는 모든 프로세스가 이를 배포할 수 있습니다. 대규모 환경에서는 일반적으로 macOS의 Jamf나 구성 프로필, Windows의 그룹 정책(GPO)이나 Intune, Linux의 플릿 관리 도구 등 장치 관리 툴을 사용합니다. Claude Code는 다음 경로 중 하나에서 이 파일을 탐색합니다:

| 플랫폼        | 경로                                                       |
| :------------ | :--------------------------------------------------------- |
| macOS         | `/Library/Application Support/ClaudeCode/managed-mcp.json` |
| Linux 및 WSL  | `/etc/claude-code/managed-mcp.json`                        |
| Windows       | `C:\Program Files\ClaudeCode\managed-mcp.json`             |

이 파일은 프로젝트 [`.mcp.json`](/docs/en/mcp#project-scope) 파일과 동일한 형식을 사용합니다:

```json theme={null}
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "sentry": {
      "type": "http",
      "url": "https://mcp.sentry.dev/mcp"
    },
    "company-internal": {
      "type": "stdio",
      "command": "/usr/local/bin/company-mcp-server",
      "args": ["--config", "/etc/company/mcp-config.json"],
      "env": {
        "COMPANY_API_URL": "https://internal.example.com"
      }
    }
  }
}
```

### 사용자별 자격 증명으로 인증

머신의 모든 사용자가 이 파일을 읽을 수 있으므로 `env` 블록에 API 키나 기타 자격 증명을 저장하지 마세요. 대신 다음 중 하나를 사용하여 사용자별 자격 증명을 전달하세요:

* 각 사용자의 환경에서 시크릿을 읽는 [`${VAR}` 치환(expansion)](/docs/en/mcp#environment-variable-expansion-in-mcp-json) 사용
* 각 사용자가 본인 계정으로 인증하도록 [OAuth 또는 사용자별 헤더](/docs/en/mcp#authenticate-with-remote-mcp-servers) 사용
* 연결 시점에 자격 증명을 생성하는 [`headersHelper`](/docs/en/mcp#use-dynamic-headers-for-custom-authentication) 사용

### 구성 검증

파일이 정상 적용되었는지 확인하려면 관리 대상 머신에서 두 가지 검사를 실행하세요:

1. `claude mcp list`를 실행했을 때 `managed-mcp.json`에 정의된 서버만 표시되어야 합니다. 사용자의 자체 서버가 여전히 표시된다면 파일이 읽히지 않는 것이므로 경로와 권한을 확인하세요.
2. `claude mcp add --transport http test https://example.com/mcp` 실행 시 `Cannot add MCP server: enterprise MCP configuration is active and has exclusive control over MCP servers` 오류와 함께 실패해야 합니다. 정책 검사가 모든 연결 시도 전에 명령을 거부하므로 URL이 실제 서버일 필요는 없습니다.

### MCP 완전히 비활성화하기

모든 MCP 서버를 차단하려면 빈 서버 맵을 가진 `managed-mcp.json`을 배포하세요:

```json theme={null}
{
  "mcpServers": {}
}
```

사용자는 `/mcp`에서 어떠한 MCP 서버도 볼 수 없으며, `claude mcp add`는 위의 기업 정책 오류를 발생시키며 실패합니다. 사용자가 이전에 구성했던 서버들도 다음 세션 시작 시 정책이 원인이라는 경고 없이 로드가 중지됩니다.

### 관리형 세트와 함께 claude.ai 커넥터 허용

`managed-mcp.json`을 배포하면 기본적으로 claude.ai 관리자 콘솔에서 조직을 위해 관리자가 구성한 커넥터를 포함하여 [claude.ai 커넥터](/docs/en/mcp#use-mcp-servers-from-claude-ai)가 억제(suppress)됩니다. `managed-mcp.json` 내부의 서버와 함께 해당 커넥터들을 로드하려면 [관리형 설정 소스](/docs/en/admin-setup#decide-how-settings-reach-devices)에 `"allowAllClaudeAiMcps": true`를 설정하세요. 사용하려면 Claude Code v2.1.149 이상이 필요합니다.

이 설정이 활성화되면 Claude Code는 `managed-mcp.json`이 배포되지 않았을 때 로드하는 동일한 claude.ai 커넥터들을 로드합니다. 이들 커넥터에도 [허용 목록 및 거부 목록](#policy-based-control-with-allowlists-and-denylists)이 적용되므로 `deniedMcpServers`로 특정 커넥터를 차단할 수 있습니다. 이 설정은 claude.ai 커넥터에만 영향을 주며, 플러그인 제공 서버는 계속 억제 상태를 유지합니다.

Claude Code는 관리자가 제어하는 정책 계층(서버 관리형 설정, MDM 배포 plist 또는 HKLM 레지스트리 키, 시스템 `managed-settings.json` 파일)에서만 이 설정을 읽습니다. 사용자나 프로젝트 설정에 추가하는 것은 아무런 효과가 없으므로 사용자가 독점적 제어로 억제된 커넥터를 임의로 다시 활성화할 수 없습니다.

## 허용 목록 및 거부 목록을 통한 정책 기반 제어

허용 목록과 거부 목록은 구성된 서버 중 로드가 허용되는 서버를 필터링합니다. 이들은 레지스트리가 아닙니다: 허용 목록이나 거부 목록이 적용되기 전에 사용자, 플러그인 또는 `managed-mcp.json`에 의해 서버가 먼저 추가되어야 합니다. 사용자에게 서버를 배포하려면 [`managed-mcp.json`](#exclusive-control-with-managed-mcp-json)을 사용하세요. 두 목록 모두 [`--mcp-config` CLI 플래그](/docs/en/cli-reference#cli-flags)로 전달된 서버를 필터링합니다; `--strict-mcp-config`는 로드되는 설정 파일을 제한할 뿐 두 목록 중 어느 것도 우회하지 않습니다.

허용 목록에 결정 권한을 부여하려면 서버 관리형 설정이나 배포된 `managed-settings.json` 파일과 같은 [관리형 설정 소스](/docs/en/admin-setup#decide-how-settings-reach-devices)에 `allowedMcpServers`와 `allowManagedMcpServersOnly: true`를 함께 설정하세요. [허용 목록을 관리형 설정으로만 제한하기](#restrict-the-allowlist-to-managed-settings-only)에서 구성을 다룹니다. `allowManagedMcpServersOnly`가 없으면 사용자의 자체 `~/.claude/settings.json`을 포함한 모든 설정 소스의 허용 목록이 병합되므로 사용자가 허용 범위를 확장할 수 있습니다. 거부 목록은 설정 소스와 관계없이 항상 병합됩니다.

<Note>
  `allowManagedMcpServersOnly`는 [권한 규칙](/docs/en/permissions#managed-settings)만 잠그는 `allowManagedPermissionRulesOnly`와 별개입니다. 해당 플래그를 설정한다고 해서 MCP 허용 목록이 강제 적용되지는 않습니다.
</Note>

### URL, 명령, 이름으로 서버 매칭

`allowedMcpServers` 및 `deniedMcpServers`는 항목들의 목록입니다. 각 항목은 URL, 명령, 또는 이름으로 서버를 식별하는 단일 키를 가진 객체입니다:

| 키               | 매칭 대상                                                             | 사용 목적                              |
| :-------------- | :-------------------------------------------------------------------- | :------------------------------------- |
| `serverUrl`     | 원격 서버 URL (정확한 일치 또는 `*` 와일드카드 포함)                    | HTTP 및 SSE 서버                       |
| `serverCommand` | stdio 서버를 시작하는 정확한 명령 및 인수                              | Stdio 서버                             |
| `serverName`    | 사용자가 지정한 레이블. 정확한 일치만 지원하며 와일드카드는 확장 안 됨  | 두 유형 모두 가능하나 아래 경고 참조   |

`allowedMcpServers`를 설정하지 않은 것과 빈 배열로 설정하는 것은 다릅니다:

| 설정                | 미설정 (기본값)     | 빈 배열 `[]`       | 값이 채워짐                   |
| :------------------ | :------------------ | :----------------- | :---------------------------- |
| `allowedMcpServers` | 모든 서버 허용      | 모든 서버 허용 안 됨| 매칭되는 서버만 허용          |
| `deniedMcpServers`  | 차단되는 서버 없음  | 차단되는 서버 없음 | 매칭되는 서버 차단            |

항목이 스키마 검증에 실패할 때 발생하는 현상은 [관리형 설정 내의 유효하지 않은 항목](/docs/en/settings#invalid-entries-in-managed-settings)을 참조하세요.

<Warning>
  두 목록 중 어디에 있든 `serverName` 항목은 보안 제어 수단이 아닙니다. 이 이름은 사용자가 `claude mcp add`를 실행하거나 설정 파일을 편집할 때 부여하는 레이블일 뿐 원본 서버 자체가 아니므로, 사용자가 임의의 서버를 `github`이라고 이름 붙일 수 있습니다. claude.ai 커넥터의 경우 이름은 claude.ai가 반환하는 표시 이름이며 변경될 수 있습니다. 실행되는 서버를 확실히 강제하려면 `serverCommand` 또는 `serverUrl` 항목을 추가하세요.
</Warning>

`serverName` 검증 방식은 두 목록 간에 다릅니다:

* `deniedMcpServers`에서 `serverName`은 비어 있지 않은 모든 문자열을 허용하므로, 표시 이름으로 [claude.ai 커넥터](/docs/en/mcp#use-mcp-servers-from-claude-ai)를 차단할 수 있습니다. 예를 들어 `{ "serverName": "claude.ai Slack" }`은 Slack 커넥터를 차단합니다. 이름 변경에 영향받지 않는 강력한 거부가 필요하거나 커넥터 이름 충돌로 ` (N)` 접미사가 붙는 경우 `serverUrl` 항목을 사용하는 것이 좋습니다.
* `allowedMcpServers`에서 `serverName`은 영문자, 숫자, 하이픈, 언더스코어로 제한됩니다. claude.ai 커넥터를 허용 목록에 추가하려면 `serverUrl`을 사용하세요.

모든 claude.ai 커넥터를 끄려면 [`disableClaudeAiConnectors`](/docs/en/mcp#disable-claude-ai-connectors)를 참조하세요.

### 서버 평가 방식

Claude Code는 `managed-mcp.json`에서 제공된 서버를 포함하여 서버를 로드하기 전에 다음 3가지 검사를 순서대로 진행합니다:

1. **목록 병합.** 모든 설정 소스의 허용 목록 및 거부 목록 항목이 하나의 허용 목록과 하나의 거부 목록으로 결합됩니다. `allowManagedMcpServersOnly`가 `true`일 때는 관리형 허용 목록만 유지됩니다; 거부 목록은 항상 모든 소스에서 병합됩니다.
2. **거부 목록 검사.** URL, 명령 또는 이름에 의해 거부 목록 항목 중 하나라도 매칭되는 서버는 차단됩니다. 어떠한 설정도 거부 목록 매칭을 재정의할 수 없습니다.
3. **허용 목록 검사.** `allowedMcpServers`가 어디에도 설정되지 않았다면 거부 목록 검사를 통과한 모든 서버가 로드됩니다. 설정되어 있다면 아래 표에 표시된 서버 유형에 맞춰 매칭되어야만 로드됩니다.

| 서버 유형            | 허용되는 매칭 조건                                                                                               |
| :------------------- | :--------------------------------------------------------------------------------------------------------------- |
| 원격 (HTTP 또는 SSE) | `serverUrl` 항목과 매칭. `serverName` 매칭은 허용 목록에 `serverUrl` 항목이 하나도 없을 때만 적용됨              |
| Stdio                | `serverCommand` 항목과 매칭. `serverName` 매칭은 허용 목록에 `serverCommand` 항목이 하나도 없을 때만 적용됨      |

해당 검사 과정에는 다음 3가지 매칭 규칙이 적용됩니다:

* **명령은 정확히 일치해야 함.** 순서대로 모든 인수가 일치해야 합니다. `["npx", "-y", "server"]`는 `["npx", "server"]`나 `["npx", "-y", "server", "--flag"]`와 일치하지 않습니다.
* **`serverCommand` 및 `serverUrl` 값은 매칭 전 치환됨.** 정책 항목과 서버의 구성 값 모두 `.mcp.json`과 동일한 [`${VAR}` 및 `${VAR:-default}` 치환](/docs/en/mcp#environment-variable-expansion-in-mcp-json)을 거칩니다. 따라서 `["${HOME}/bin/server"]`로 작성된 항목은 동일한 참조를 사용하거나 치환된 경로를 사용하는 서버 구성과 일치합니다. Windows의 경우 `${HOME}` 대신 `${USERPROFILE}`과 같이 설정된 환경 변수를 참조하세요. `serverName` 값은 문자 그대로 매칭되며 치환되지 않습니다.
* **URL은 패턴의 어디서나 `*` 와일드카드를 지원함.** 스킴(scheme)을 포함하여 어디든 가능합니다. 호스트명 매칭은 대소문자를 구분하지 않으며 후미 FQDN 점을 무시하므로 `https://Mcp.Example.com/*`는 `https://mcp.example.com/api`와 일치합니다. 경로는 대소문자를 구분합니다.

| 패턴                        | 허용 대상                                                              |
| :-------------------------- | :--------------------------------------------------------------------- |
| `https://mcp.example.com/*` | 특정 도메인의 모든 경로                                                |
| `https://mcp.example.com`   | 해당 도메인의 모든 경로. 경로가 없는 패턴은 모든 경로에 일치함        |
| `https://*.example.com/*`   | `example.com` 하위 도메인 전체                                         |
| `http://localhost:*/*`      | localhost의 모든 포트                                                  |
| `*://mcp.example.com/*`     | 특정 도메인에 대한 모든 스킴                                           |

`${VAR}` 치환은 Claude Code 자체의 프로세스 환경을 읽으므로 변수를 참조하는 `serverCommand`나 `serverUrl` 정책 항목은 사용자가 설정한 값으로 치환됩니다. 강제 적용에 의존하는 항목에는 문자 그대로의 URL과 명령을 사용하세요.

### 구성 예시

아래 구성은 거부 목록이 포함된 엄격한 허용 목록을 설정합니다. 강조 표시된 행은 목록의 나머지 항목이 평가되는 방식을 변경하며, 블록 뒤의 설명에서 각각을 해설합니다:

```json {3,5,11} theme={null}
{
  "allowedMcpServers": [
    { "serverUrl": "https://api.githubcopilot.com/*" },
    { "serverUrl": "https://mcp.sentry.dev/*" },
    { "serverCommand": ["npx", "-y", "@modelcontextprotocol/server-filesystem", "."] },
    { "serverCommand": ["python", "/usr/local/bin/approved-server.py"] },
    { "serverUrl": "https://mcp.example.com/*" },
    { "serverUrl": "https://*.internal.example.com/*" }
  ],
  "deniedMcpServers": [
    { "serverName": "dangerous-server" },
    { "serverCommand": ["npx", "-y", "unapproved-package"] },
    { "serverUrl": "https://*.untrusted.example.com/*" }
  ]
}
```

* **3행**: 첫 번째 `serverUrl` 항목. 이 항목이 포함되면 모든 원격 서버가 URL 패턴과 일치해야 하므로, 사용자가 목록에 없는 원격 서버에 허용된 이름을 부여하여 우회할 수 없습니다.
* **5행**: 첫 번째 `serverCommand` 항목. stdio 서버에 대해 동일한 효과를 내므로 모든 로컬 서버는 나열된 명령과 정확히 일치해야 합니다.
* **11행**: 거부 목록의 `serverName` 항목. 거부 목록 항목은 항상 적용되므로 `dangerous-server`라는 이름의 서버는 URL이나 명령에 상관없이 차단됩니다.

이 허용 목록의 `serverName` 항목은 두 전송 유형 모두에 대해 더 엄격한 항목이 이미 존재하므로 매칭되지 않습니다.

아래 아코디언 항목들에서 다른 허용 목록 및 거부 목록 조합에 대해 서버가 평가되는 방식을 확인할 수 있습니다.

<Accordion title="URL 전용 허용 목록">
  ```json theme={null}
  {
    "allowedMcpServers": [
      { "serverUrl": "https://mcp.example.com/*" },
      { "serverUrl": "https://*.internal.example.com/*" }
    ]
  }
  ```

  | 서버                                                  | 결과                                         |
  | :---------------------------------------------------- | :------------------------------------------- |
  | `https://mcp.example.com/api` 위치의 HTTP 서버        | 허용: URL 패턴 일치                          |
  | `https://api.internal.example.com/mcp` 위치의 HTTP 서버| 허용: 와일드카드 하위 도메인 일치            |
  | `https://external.example.com/mcp` 위치의 HTTP 서버   | 차단: 일치하는 URL 패턴 없음                 |
  | 임의 명령을 가진 Stdio 서버                           | 차단: 매칭할 이름이나 명령 항목 없음         |
</Accordion>

<Accordion title="명령 전용 허용 목록">
  ```json theme={null}
  {
    "allowedMcpServers": [
      { "serverCommand": ["npx", "-y", "approved-package"] }
    ]
  }
  ```

  | 서버                                                  | 결과                              |
  | :---------------------------------------------------- | :-------------------------------- |
  | `["npx", "-y", "approved-package"]` 명령의 Stdio 서버 | 허용: 명령 일치                   |
  | `["node", "server.js"]` 명령의 Stdio 서버             | 차단: 명령 일치하지 않음          |
  | `my-api` 이름의 HTTP 서버                             | 차단: 매칭할 이름 항목 없음       |
</Accordion>

<Accordion title="이름 및 명령 혼합 허용 목록">
  ```json theme={null}
  {
    "allowedMcpServers": [
      { "serverName": "github" },
      { "serverCommand": ["npx", "-y", "approved-package"] }
    ]
  }
  ```

  | 서버                                                                     | 결과                                                                  |
  | :----------------------------------------------------------------------- | :-------------------------------------------------------------------- |
  | `["npx", "-y", "approved-package"]` 명령, `local-tool` 이름의 Stdio 서버| 허용: 명령 일치                                                       |
  | `["node", "server.js"]` 명령, `local-tool` 이름의 Stdio 서버             | 차단: 명령 항목이 존재하나 일치하지 않음                               |
  | `["node", "server.js"]` 명령, `github` 이름의 Stdio 서버                 | 차단: 명령 항목이 존재할 경우 Stdio 서버는 반드시 명령과 일치해야 함 |
  | `github` 이름의 HTTP 서버                                                | 허용: 이름 일치                                                       |
  | `other-api` 이름의 HTTP 서버                                             | 차단: 이름 일치하지 않음                                              |
</Accordion>

<Accordion title="이름 전용 허용 목록">
  ```json theme={null}
  {
    "allowedMcpServers": [
      { "serverName": "github" },
      { "serverName": "internal-tool" }
    ]
  }
  ```

  | 서버                                                | 결과                             |
  | :-------------------------------------------------- | :------------------------------- |
  | 임의의 명령을 가지며 `github` 이름인 Stdio 서버     | 허용: 명령 제약 조건 없음        |
  | 임의의 명령을 가지며 `internal-tool` 이름인 Stdio 서버| 허용: 명령 제약 조건 없음      |
  | `github` 이름의 HTTP 서버                           | 허용: 이름 일치                  |
  | `other` 이름의 임의 서버                            | 차단: 이름 일치하지 않음         |
</Accordion>

<Accordion title="거부 목록 재정의가 포함된 허용 목록">
  ```json theme={null}
  {
    "allowedMcpServers": [
      { "serverUrl": "https://*.example.com/*" }
    ],
    "deniedMcpServers": [
      { "serverUrl": "https://staging.example.com/*" }
    ]
  }
  ```

  | 서버                                             | 결과                                                      |
  | :----------------------------------------------- | :-------------------------------------------------------- |
  | `https://mcp.example.com/api` 위치의 HTTP 서버   | 허용: 허용 목록 URL 패턴 일치, 거부 목록 일치 항목 없음   |
  | `https://staging.example.com/api` 위치의 HTTP 서버| 차단: 둘 다 일치하나 거부 목록이 우선 적용됨              |
  | `https://other.com/mcp` 위치의 HTTP 서버         | 차단: 허용 목록에 일치하지 않음                           |
</Accordion>

### 허용 목록을 관리형 설정으로만 제한하기

관리형 허용 목록만 적용되도록 하려면 관리형 설정 파일에 `allowManagedMcpServersOnly`를 설정하세요:

```json theme={null}
{
  "allowManagedMcpServersOnly": true,
  "allowedMcpServers": [
    { "serverUrl": "https://api.githubcopilot.com/*" },
    { "serverUrl": "https://*.internal.example.com/*" }
  ]
}
```

`allowManagedMcpServersOnly`가 `true`이면 사용자, 프로젝트, 로컬 설정의 허용 목록이 무시됩니다. 거부 목록은 여전히 모든 소스에서 병합되므로 사용자는 스스로를 위해 항상 서버를 차단할 수 있습니다.

## 제한이 사용자에게 표시되는 방식

제한으로 인해 서버가 차단되면 사용자는 `claude mcp add` 실행 시 오류를 보거나 서버 로드가 소리 없이 중단되는 현상을 겪게 됩니다. 다음 표를 참고하여 보고된 현상을 구별하고 변경사항 배포 전에 사용자에게 예고하세요:

| 제약 조건                                                            | 사용자에게 표시되는 내용                                                                                    |
| :------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------- |
| `managed-mcp.json`이 존재할 때 `claude mcp add` 실행                 | `Cannot add MCP server: enterprise MCP configuration is active and has exclusive control over MCP servers` |
| 서버가 거부 목록에 있을 때 `claude mcp add` 실행                     | `Cannot add MCP server "<name>": server is explicitly blocked by enterprise policy`                        |
| 서버가 허용 목록에 없을 때 `claude mcp add` 실행                     | `Cannot add MCP server "<name>": not allowed by enterprise policy`                                         |
| 이전에 구성된 서버가 정책에 의해 새로 차단된 경우                    | 아무런 경고 없이 `/mcp` 및 `claude mcp list`에서 소리 없이 삭제됨                                           |

마지막 경우 사용자는 서버 삭제 원인이 정책 때문인지 알 수 없으므로, 새 제한사항을 배포할 때 차단되는 서버에 대해 해당 사용자에게 안내하세요.

## MCP 사용량 모니터링

[OpenTelemetry 내보내기](/docs/en/monitoring-usage)가 설정된 경우 Claude Code는 사용자가 호출하는 MCP 서버 및 도구를 기록할 수 있습니다. `OTEL_LOG_TOOL_DETAILS=1`을 설정하여 도구 이벤트에 MCP 서버 및 도구 이름을 포함한 다음 콜렉터(collector)에서 이를 집계하면 사용자가 실제로 연결하는 서버를 파악할 수 있습니다. 내보내기 설정 방법 및 이벤트 스키마 전문은 [모니터링](/docs/en/monitoring-usage)을 참조하세요.

## 구성 요약

이 페이지에서 다룬 파일 및 설정, 제어 대상, 배포 방법 정리:

| 항목                         | 제어 대상                                                                           | 위치                                                                                                                         | 배포 방법                                                                                                                                                                   |
| :--------------------------- | :---------------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `managed-mcp.json`           | 고정 서버 세트, 독점적 제어                                                         | 시스템 경로: `/Library/Application Support/ClaudeCode/`, `/etc/claude-code/`, 또는 `C:\Program Files\ClaudeCode\`            | MDM, GPO, 플릿 관리 도구 또는 관리자 권한이 있는 프로세스. 서버 관리형 설정을 통해서는 설정 불가능                                                                            |
| `allowedMcpServers`          | 허용된 서버의 허용 목록                                                             | 모든 [설정 파일](/docs/en/settings#settings-files); `allowManagedMcpServersOnly`가 설정되지 않는 한 모든 소스 항목 병합    | 강제 적용을 위해서는 [관리형 설정 소스](/docs/en/admin-setup#decide-how-settings-reach-devices) 사용: 서버 관리형 설정, `managed-settings.json`, MDM 프로필 또는 레지스트리 |
| `deniedMcpServers`           | 차단된 서버의 거부 목록                                                             | 모든 설정 파일; 모든 소스의 항목이 병합됨                                                                                    | `allowedMcpServers`와 동일                                                                                                                                                  |
| `allowManagedMcpServersOnly` | 허용 목록을 관리형 소스로만 잠금                                                    | 관리형 설정 소스 전용; 다른 위치에서는 아무런 효과가 없음                                                                    | `allowedMcpServers`와 동일                                                                                                                                                  |
| `allowAllClaudeAiMcps`       | `managed-mcp.json`과 함께 claude.ai 커넥터를 억제하지 않고 함께 로드                | 관리형 설정 소스 전용; 다른 위치에서는 아무런 효과가 없음                                                                    | `allowedMcpServers`와 동일                                                                                                                                                  |

## 관련 리소스

* [강제 적용 대상 결정하기](/docs/en/admin-setup#decide-what-to-enforce): 권한 규칙, 샌드박싱 및 기타 관리 제어 기능과 함께 다루는 MCP 제한사항
* [MCP를 통해 Claude를 도구에 연결하기](/docs/en/mcp): 트랜스포트, 스코프, 인증을 포함한 MCP 참조 문서
* [설정](/docs/en/settings): 설정 계층 구조 및 관리형 설정의 우선 적용 방식
* [서버 관리형 설정](/docs/en/server-managed-settings): Claude.ai 관리자 콘솔에서 `allowedMcpServers` 및 `deniedMcpServers` 전달하기
* [보안](/docs/en/security): 이 제어 기능들이 방어하는 위협 모델
* [Claude Enterprise Administrator Guide](https://claude.com/resources/tutorials/claude-enterprise-administrator-guide): SSO, SCIM, 계정 관리 및 배포 플레이북
