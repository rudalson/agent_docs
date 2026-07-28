> ## 문서 목차
> 문서 전체 목차 보기: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude가 코드를 작성하는 동안 보안 문제 포착하기

> 보안 지침(security-guidance) 플러그인을 설치하면 Claude가 자신의 코드 변경 사항에서 취약점을 검토하고 동일한 세션 내에서 수정하도록 할 수 있습니다.

보안 지침 플러그인은 Claude가 작업하는 동안 일반적인 취약점이 있는지 자체 코드 변경 사항을 검토하고 동일한 세션에서 발견된 문제를 수정하도록 합니다. 이 플러그인은 인젝션(injection), 안전하지 않은 역직렬화(unsafe deserialization), 안전하지 않은 DOM API와 같은 문제를 코드가 풀 리퀘스트에 도달하기 전에 포착하여 다운스트림의 사람 리뷰어에게 전달되는 보안 검토 부담을 줄여줍니다.

설치되면 플러그인이 자동으로 실행됩니다. 호출할 작업이나 별도로 기억해야 할 명령어가 없습니다.

이 플러그인은 풀 리퀘스트에서 실행되는 [코드 리뷰](/docs/en/code-review)의 세션 내 동반자입니다. 플러그인은 PR에 도달하는 항목을 줄여주며, 코드 리뷰는 도달한 항목을 포착합니다. 이 플러그인이 주문형 검토 및 CI 스캐닝과 어떻게 어우러지는지는 [다른 보안 도구와 조화되는 방식](#다른-보안-도구와-조화되는-방식)을 참조하세요.

## 사전 요구 사항

* Claude Code CLI 버전 2.1.144 이상
* `PATH`에 Python 3.7 이상 등록. 에이전틱 커밋 검토는 Python 3.10 이상이 필요하며, Claude Code가 Amazon Bedrock이나 Google Cloud's Agent Platform과 같은 서드파티 제공업체를 사용하는 경우의 모든 모델 기반 검토도 동일합니다. 플러그인은 버전이 지정된 인터프리터 `python3.13` ~ `python3.10`을 선호하며, 그 후 `python3`, `python`, `py -3`으로 대체됩니다.
* 작업하는 디렉터리에 대한 Git 리포지토리. 차례 종료(end-of-turn) 및 커밋 검토는 Git 상태와 비교 diff를 수행하며 리포지토리 외부에서는 메시지 없이 건너뜁니다. 편집당 패턴 검사(per-edit pattern check)는 어디서나 작동합니다.

첫 실행 시 플러그인은 `~/.claude/security/` 아래에 가상 환경을 생성하고 Claude Agent SDK를 설치하며, 이 과정에서 `pip` 및 네트워크 액세스가 필요합니다. 해당 설치가 실패하거나 사용 가능한 Python 버전이 3.10보다 구버전인 경우, 퍼스트 파티 인증에서의 커밋 검토는 에이전틱 검토 대신 단일 실행(single-shot) 검토로 대체됩니다. Amazon Bedrock이나 Google Cloud's Agent Platform과 같은 서드파티 제공업체에서는 모델 기반 검토에 SDK 자체가 필요하므로 검토를 건너뜁니다. 이전 버전의 Python이 원인인 경우 플러그인에서 일회성 알림을 표시합니다.

## 플러그인 설치

터미널 Claude Code 세션에서 [공식 Anthropic 마켓플레이스](/docs/en/discover-plugins#official-anthropic-marketplace)를 통해 설치합니다:

```text theme={null}
/plugin install security-guidance@claude-plugins-official
```

`/plugin`은 대화형 패널을 열며 터미널 CLI에서만 사용할 수 있습니다. 이 환경에서 `/plugin`을 사용할 수 없다는 Claude의 응답이 나오면 다른 방법으로 설치하세요:

* **Claude 데스크톱 앱, 로컬 또는 SSH 세션**: 프롬프트 옆의 **+** 버튼을 클릭한 다음 **Plugins**, **Add plugin**을 클릭하여 [플러그인 브라우저](/docs/en/desktop#install-plugins)를 엽니다.
* **웹 기반 Claude Code 또는 데스크톱 클라우드 세션**: [클라우드 세션 및 공유 리포지토리에서 활성화](#클라우드-세션-및-공유-리포지토리에서-활성화)에 설명된 대로 `.claude/settings.json`에 플러그인을 선언합니다.

터미널 설치 시 스코프(scope)를 묻는 프롬프트가 표시됩니다. 사용자 스코프(user scope)를 선택하면 사용자 설정에 플러그인이 기록되어 이 머신에서 시작하는 모든 새 로컬 세션에 로드됩니다. Claude Code에서 `Marketplace "claude-plugins-official" not found`라고 보고하는 경우 `/plugin marketplace add anthropics/claude-plugins-official` 명령으로 마켓플레이스를 추가하세요. 플러그인을 마켓플레이스에서 찾을 수 없다고 보고하는 경우 로컬 사본이 오래된 것이므로 `/plugin marketplace update claude-plugins-official`로 새로 고친 후 설치를 다시 시도하세요.

그런 다음 재시작 없이 보류 중인 플러그인 변경 사항을 적용하는 `/reload-plugins` 명령으로 현재 세션에서 활성화합니다:

```text theme={null}
/reload-plugins
```

### 클라우드 세션 및 공유 리포지토리에서 활성화

사용자 스코프 플러그인은 [웹 기반 Claude Code](/docs/en/claude-code-on-the-web)로 전달되지 않습니다. 해당 세션은 사용자의 머신이 아닌 Anthropic 인프라에서 실행되기 때문입니다. 그곳에서 플러그인을 활성화하거나 리포지토리를 복제하는 모든 사람에게 플러그인을 켜려면 프로젝트의 체크인된 설정에 플러그인을 선언하세요:

```json .claude/settings.json theme={null}
{
  "enabledPlugins": {
    "security-guidance@claude-plugins-official": true
  }
}
```

관리자는 [관리형 설정](/docs/en/admin-setup)에서 [`enabledPlugins`](/docs/en/settings#plugin-settings)을 설정하여 조직 전체에 플러그인을 활성화할 수 있습니다.

## 플러그인이 검사하는 항목

플러그인은 Claude의 작업을 세 시점에서 각기 다른 깊이로 검토합니다:

* [각 파일 편집 시](#각-파일-편집-시): 모델 호출 없이 위험한 호출을 빠르게 패턴 매칭
* [각 차례(turn) 종료 시](#각-차례turn-종료-시): 해당 차례에서 변경된 모든 내용에 대한 백그라운드 모델 검토
* [Claude가 수행하는 각 커밋 또는 푸시 시](#claude가-수행하는-각-커밋-또는-푸시-시): 주변 코드를 읽는 더 깊은 에이전틱 검토

[자신만의 규칙 추가](#자신만의-규칙-추가)를 통해 각 레이어를 확장할 수 있습니다. 내장된 검사를 개별적으로 제거할 수는 없지만 각 레이어를 독립적으로 [비활성화하거나 제거](#비활성화-또는-제거)할 수 있습니다.

### 각 파일 편집 시

Claude가 파일에 작성할 때 플러그인은 알맞은 위험 패턴이 있는지 새 콘텐츠를 스캔합니다. 이는 모델 호출이 없는 패턴 매칭이므로 추가 사용 비용이 발생하지 않습니다.

예시 패턴 범주:

* 동적 코드 실행: `eval(`, `new Function`, `os.system`, `child_process.exec`
* 안전하지 않은 역직렬화: `pickle`
* DOM 인젝션: `dangerouslySetInnerHTML`, `.innerHTML =`, `document.write`
* 워크플로 파일: 리포지토리 수준 권한을 부여할 수 있는 `.github/workflows/` 아래의 편집

검사는 편집이 적용된 후 실행되며 다음 단계를 위한 Claude의 컨텍스트에 경고를 추가합니다. 각 경고는 세션당 파일당 패턴당 한 번만 발생하므로 동일한 파일에서 반복되는 일치 항목이 대화를 도배하지 않습니다.

`security-patterns.yaml` 파일을 사용하여 이 레이어에 [자신만의 패턴을 추가](#사용자-정의-편집당-패턴-추가)할 수 있습니다.

### 각 차례(turn) 종료 시

차례(turn)는 Claude가 응답하는 한 라운드입니다. 즉, 사용자가 메시지를 보내고, Claude가 작업하고 응답하며, 차례가 끝납니다. 각 차례가 끝난 후 플러그인은 Claude의 편집 도구, Bash 명령, 서브에이전트의 변경 사항을 포함하여 작업 트리에 변경된 모든 항목의 git diff를 계산하고 보안에 집중된 별도의 Claude 검토로 전송합니다. 검토는 백그라운드에서 실행되므로 Claude의 응답이 지연되지 않습니다. 검토에서 문제를 발견하면 Claude에게 결과가 다시 프롬프트로 전달되어 후속 작업으로 해결하도록 합니다.

이는 문자열 일치로 포착할 수 없는 다음과 같은 문제를 포착합니다:

* 권한 우회 (Authorization bypass)
* 안전하지 않은 직간접 객체 참조 (Insecure direct object references)
* 인젝션 (Injection)
* 서버 측 요청 위조 (Server-side request forgery)
* 취약한 암호화 (Weak cryptography)

결과와 Claude의 해결 방법을 세션에서 직접 모두 확인할 수 있습니다. 검토는 차례당 최대 30개의 변경된 파일을 처리하며, 사용자에게 다시 양도하기 전에 연속으로 최대 3번까지 발생합니다.

### Claude가 수행하는 각 커밋 또는 푸시 시

Claude가 Bash 도구를 통해 `git commit` 또는 `git push`를 실행하면 플러그인은 백그라운드에서 변경 사항에 대해 더 깊은 에이전틱 검토를 실행합니다. 이 검토는 호출자, 새니타이저(sanitizer) 및 관련 파일을 포함한 주변 코드를 읽어 결과를 보고하기 전에 실제 문제인지 판단합니다. 추가 컨텍스트는 격리 상태에서는 위험해 보이지만 코드베이스에서는 안전한 패턴의 오탐(false positive)을 낮게 유지합니다.

이 레이어는 Claude가 Bash 도구를 통해 수행하는 커밋 및 푸시에서만 실행됩니다. 세션 내부의 `!` 셸 이스케이프를 포함하여 사용자가 직접 셸에서 실행하는 커밋은 검토되지 않습니다. 커밋 및 푸시 검토는 롤링 시간당 20회로 제한됩니다. 커밋 검토의 결과가 차례 종료 검토에서 이미 보고한 내용과 중복되는 경우 Claude에게 다시 프롬프트가 표시되지 않으므로 깨끗한 커밋은 이 레이어에서 눈에 보이는 출력을 생성하지 않습니다.

### 검토의 독립성 및 한계

플러그인은 코드를 작성한 동일한 Claude 인스턴스에 스스로 채점하도록 요청하지 않습니다. 편집당 검사는 모델이 참여하지 않는 확정적(deterministic) 문자열 일치입니다. 차례 종료 및 커밋 검토는 새로운 컨텍스트와 보안 중심 프롬프트를 사용하여 별도의 Claude 호출로 실행됩니다. 검토자는 diff에서 시작하여 원래 접근 방식에 대한 고정관념이 없으며 오직 문제를 찾도록 지시받습니다.

어떤 레이어도 쓰기나 커밋을 차단하지 않습니다. 결과는 지시 사항 형태로 코드를 작성하는 Claude에게 도달하고, Claude는 대화 속에서 이를 처리하며, 검토 모델이 문제를 놓칠 수도 있습니다. 플러그인을 심층 방어의 한 레이어로 다루어야 하며 완전한 보안 솔루션으로 생각해서는 안 됩니다. [다른 보안 도구와 조화되는 방식](#다른-보안-도구와-조화되는-방식)을 참조하세요.

## 자신만의 규칙 추가

플러그인에는 두 가지 확장 지점이 있습니다. 모델 기반 검토를 위한 Markdown 지침 파일과 편집당 문자열 일치를 위한 YAML 또는 JSON 패턴 파일입니다. 둘 다 가산적(additive)입니다. 검사를 추가할 수는 있지만 이 파일에서 내장 검사를 비활성화할 수는 없습니다.

### 모델 기반 검토에 대한 지침 추가

프로젝트에 `.claude/claude-security-guidance.md`를 생성하고 위협 모델과 검토 체크리스트를 일반 언어로 작성하세요. 모델 기반 검토는 내장 취약점 체크리스트와 함께 이를 추가 컨텍스트로 로드합니다.

다음 예시는 역할 게이트 관리자 경로와 고객 데이터 로깅 정책이 있는 웹 서비스에 대한 예시입니다:

```markdown .claude/claude-security-guidance.md theme={null}
# 이 리포지토리에 대한 보안 지침

- INFO 수준 이상에서 `customer_id` 또는 `account_number`를 로그에 기록하지 마세요.
- `/admin` 아래의 모든 경로는 데이터베이스 읽기 전에 `require_role("admin")`을 호출해야 합니다.
- 토큰 비교 시 `===` 대신 `crypto.timingSafeEqual`을 사용하세요.
```

이 규칙들은 검토자를 위한 지침이며 확정적인 가드레일이 아닙니다. 플러그인은 Claude가 수정할 수 있도록 위반 사항을 결과로 노출하지만, 쓰기를 차단하거나 모든 위반이 포착됨을 보장하지 않습니다. 지침은 오직 가산적입니다. 취약점 클래스를 무시하라는 규칙이 있더라도 해당 결과가 억제되지 않습니다. 엄격한 강제를 위해서는 플러그인을 [편집을 차단하는 훅](/docs/en/hooks-guide#block-edits-to-protected-files) 또는 CI 검사와 함께 사용하세요.

### 사용자 정의 편집당 패턴 추가

`.claude/security-patterns.yaml`을 생성하여 [편집당 패턴 검사](#각-파일-편집-시)에 정규식 또는 부분 문자열 규칙을 추가합니다. 이 규칙들은 내장 패턴과 함께 확정적 문자열 일치로 실행됩니다:

```yaml .claude/security-patterns.yaml theme={null}
patterns:
  - rule_name: internal_api_key
    substrings: ["sk_live_", "AKIA"]
    reminder: "하드코딩된 API 키 접두사입니다. 시크릿 매니저에서 자격 증명을 로드하세요."
  - rule_name: tenant_unfiltered_query
    regex: "\\.objects\\.all\\(\\)"
    paths: ["**/src/tenants/**"]
    reminder: "다중 테넌트 코드는 org_id로 필터링해야 합니다."
```

| 필드 | 타입 | 설명 |
| :--- | :--- | :--- |
| `rule_name` | string | 경고에 표시되는 식별자 |
| `reminder` | string | Claude의 컨텍스트에 추가되는 경고 텍스트 (최대 1 KB) |
| `regex` | string | 편집된 콘텐츠와 일치시키는 Python 정규식 |
| `substrings` | list | 리터럴 부분 문자열; 이 항목 또는 `regex`를 제공 |
| `paths` | list | 선택적 글롭(glob) 패턴; 규칙은 일치하는 파일에만 적용됨. 글롭은 전체 파일 경로에 대해 일치하므로 프로젝트 상대 패턴 앞에는 `**/`를 붙이세요. |
| `exclude_paths` | list | 건너뛸 선택적 글롭 패턴; `paths`와 동일한 일치 방식 |

플러그인은 동일한 스키마로 `.claude/security-patterns.yml` 및 `.claude/security-patterns.json`도 읽습니다. JSON은 모든 Python 설치에서 작동합니다. YAML 형식은 PyYAML을 임포트할 수 있어야 하지만 플러그인이 이를 대신 설치해주지는 않습니다. 플러그인은 최대 50개의 사용자 정의 규칙을 로드하며 치명적인 백트래킹(catastrophic backtracking)을 유발할 수 있는 정규식은 건너뜁니다.

### 규칙 파일 탐색 위치

플러그인은 플러그인이 활성화된 방식과 무관하게 동일한 위치에서 `claude-security-guidance.md` 및 `security-patterns.yaml`을 탐색합니다:

| 스코프 | 경로 | 참고 사항 |
| :--- | :--- | :--- |
| 사용자 | `~/.claude/claude-security-guidance.md` | 머신의 모든 프로젝트에 적용됨 |
| 프로젝트 | `.claude/claude-security-guidance.md` | 리포지토리에 체크인됨 |
| 프로젝트 로컬 | `.claude/claude-security-guidance.local.md` | Git 무시됨, 개인 오버라이드용 |

플러그인은 존재하는 모든 위치를 로드하고 이를 연결하며, 지침 파일의 결합 한도는 8 KB입니다. 관리자는 디바이스 관리를 통해 사용자 스코프 파일을 `~/.claude/`로 푸시하여 조직 전체 규칙을 배포할 수 있습니다. 동일한 경로가 `security-patterns.yaml`에도 적용됩니다.

## 사용 비용

[편집당 패턴 검사](#각-파일-편집-시)는 모델 호출을 하지 않으며 비용이 추가되지 않습니다. [차례 종료](#각-차례turn-종료-시) 및 [커밋](#claude가-수행하는-각-커밋-또는-푸시-시) 검토는 각각 다른 Claude 요청과 마찬가지로 [사용량](/docs/en/costs)에 합산되는 추가 모델 사용량을 지출합니다. 커밋 검토는 에이전틱 방식으로 커밋당 여러 모델 차례가 걸릴 수 있으며, 롤링 시간당 20회 검토로 제한됩니다. 파일을 변경하는 차례당 대략 한 번의 검토 호출과 커밋당 한 번의 더 깊은 검토가 발생할 것으로 예상되며, 둘 다 위의 제한이 적용됩니다.

두 모델 기반 검토는 기본적으로 Claude Opus 4.7을 사용합니다. 차례 종료 검토의 경우 `SECURITY_REVIEW_MODEL`을 설정하고 커밋 검토의 경우 `SG_AGENTIC_MODEL`을 설정하여 다른 모델을 선택할 수 있습니다.

이 플러그인은 모든 요금제에서 사용할 수 있습니다.

## 비활성화 또는 제거

나머지는 유지하면서 개별 레이어를 끄려면 해당 환경 변수를 설정하세요:

| 변수 | 효과 |
| :--- | :--- |
| `ENABLE_PATTERN_RULES=0` | [편집당 패턴 검사](#각-파일-편집-시) 비활성화 |
| `ENABLE_STOP_REVIEW=0` | [차례 종료 diff 검토](#각-차례turn-종료-시) 비활성화 |
| `ENABLE_COMMIT_REVIEW=0` | [커밋 및 푸시 검토](#claude가-수행하는-각-커밋-또는-푸시-시) 비활성화 |
| `ENABLE_CODE_SECURITY_REVIEW=0` | 모든 모델 기반 검토를 한 번에 비활성화 |
| `SECURITY_GUIDANCE_DISABLE=1` | 제거하지 않고 플러그인 전체를 비활성화 |

사용자 스코프에서 플러그인을 일시 정지하려면:

```text theme={null}
/plugin disable security-guidance@claude-plugins-official
```

사용자 스코프에서 제거하려면:

```text theme={null}
/plugin uninstall security-guidance@claude-plugins-official
```

프로젝트의 `.claude/settings.json`을 통해 플러그인이 활성화된 경우, `/plugin`에서 이를 비활성화하면 체크인된 파일을 편집하는 대신 `.claude/settings.local.json`에 오버라이드를 기록하므로 팀원에게 주지 않고 본인에게만 플러그인이 꺼진 상태로 유지됩니다. {/* min-version: 2.1.203 */}동일한 대화 상자에서 공유 `.claude/settings.json`에서 제거하여 모든 사람에 대해 플러그인을 제거하는 기능도 제공합니다. 이 옵션은 Claude Code v2.1.203 이상이 필요합니다. [관리형 설정](/docs/en/admin-setup)을 통해 활성화된 경우 관리자만 비활성화할 수 있습니다.

## 플러그인이 Claude Code와 통합되는 방식

이 플러그인은 Claude 루프의 특정 시점에서 사용자 고유의 코드를 실행하는 메커니즘인 [훅(hooks)](/docs/en/hooks)을 기반으로 완전히 구축되었습니다. 다음 이벤트들을 등록합니다:

| 훅 이벤트 | 목적 |
| :--- | :--- |
| `SessionStart` | 플러그인의 Python 환경 부트스트랩 |
| `UserPromptSubmit` | 차례 종료 검토가 diff를 수행할 작업 트리 기준선 캡처 |
| `Edit`, `Write`, `NotebookEdit`에 대한 `PostToolUse` | 편집당 패턴 매칭 |
| `Stop` | 백그라운드에서 실행되는 차례 종료 diff 검토 |
| `git commit` 및 `git push`로 필터링된 `Bash`에 대한 `PostToolUse` | 백그라운드에서 실행되는 커밋 및 푸시 검토 |

자신만의 훅을 만드는 경우, [플러그인의 소스](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/security-guidance)는 훅에서 별도의 모델 호출을 실행하고 결과를 세션에 다시 피드백하는 작업 예시가 됩니다.

## 다른 보안 도구와 조화되는 방식

이 플러그인은 심층 방어 접근 방식의 한 레이어입니다. 코드가 편집기에 있는 동안 가장 빠르게 문제를 포착하지만 보장하지는 않으며 이후의 검사를 대체하지 않습니다. 일반적인 스택:

| 단계 | 도구 | 커버하는 내용 |
| :--- | :--- | :--- |
| 세션 내 | 보안 지침 플러그인 | Claude가 작성하는 코드의 일반적인 취약점, 동일 세션 내 수정 |
| 주문형, 단일 패스 | [`/security-review`](/docs/en/commands#all-commands) | 요청 시 실행되는 현재 브랜치에 대한 일회성 보안 패스 |
| 주문형, 정밀 스캔 | [Claude Security 플러그인](/docs/en/claude-security) | 독립적으로 검토된 결과 및 패치를 포함하는 리포지토리 또는 diff의 다중 에이전트 취약점 스캔 |
| 풀 리퀘스트 시 | [코드 리뷰](/docs/en/code-review), Team 및 Enterprise 플랜 | 전체 코드베이스 컨텍스트를 활용한 다중 에이전트 정확성 및 보안 검토 |
| CI 내 | 기존 정적 분석 및 종속성 스캐너 | 언어 특정 규칙, 공급망 검사, 플러그인이 시도하지 않는 정책 강제 적용 |

이후의 각 단계는 이전 단계가 놓친 것을 포착합니다. 플러그인의 가치는 거기에 도달하는 볼륨을 줄이는 것이지 필요성을 없애는 것이 아닙니다.

## 문제 해결

플러그인은 `~/.claude/security/log.txt`에 런타임 진단을 기록합니다. 검토가 나타나지 않는 경우 이곳을 먼저 확인하세요.

검토 레이어가 대화에 메시지 없이 건너뛰는 일반적인 이유:

* 디렉터리가 Git 리포지토리가 아님: 차례 종료 및 커밋 검토는 Git 상태가 필요하며 리포지토리 외부에서는 건너뜁니다.
* 세션에 Anthropic 인증이 없고 구성된 서드파티 제공업체가 없음: 모델 기반 검토는 건너뛰고 편집당 패턴 검사만 실행됩니다.
* `security-patterns.yaml` 파일이 존재하지만 PyYAML을 임포트할 수 없음: 파일이 무시됩니다. 대신 `security-patterns.json`을 사용하세요.

## 관련 리소스

이 페이지에서 다루는 내용을 더 자세히 알아보려면:

* [코드 리뷰](/docs/en/code-review): PR 시점 다중 에이전트 검토 설정
* [훅으로 작업 자동화](/docs/en/hooks-guide): 동일한 수명 주기 시점에 자신만의 검사 구축
* [플러그인 탐색 및 설치](/docs/en/discover-plugins#official-anthropic-marketplace): 다른 공식 플러그인 둘러보기
