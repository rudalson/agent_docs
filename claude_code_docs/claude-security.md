> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 코드베이스 취약점 스캔하기

> Claude Security 플러그인을 설치하여 Claude Code 세션에서 코드베이스의 취약점을 스캔하고, 발견된 결과를 직접 검토하고 적용할 수 있는 패치로 변환하세요.

Claude Security 플러그인은 Claude Code 세션 내에서 코드베이스의 다중 에이전트(multi-agent) 취약점 스캔을 실행합니다. Claude 에이전트 팀이 아키텍처를 매핑하고, 위협 모델을 구축하며, 취약점을 찾아내고, 보고서를 작성하기 전에 각 발견 항목을 독립적으로 검토합니다. 플러그인을 사용하여 전체 리포지토리 또는 브랜치 diff, 풀 리퀘스트 diff, 단일 커밋과 같은 [특정 변경 사항 세트만 스캔](#scan-only-your-changes)한 다음, 선택한 발견 항목을 직접 검토하고 적용할 수 있는 패치로 변환하세요.

플러그인은 세션 내에서 로컬로 실행되며, 각 스캔은 요금제의 사용량 한도에 반영됩니다. 리포지토리를 모니터링하는 관리형 서비스를 원하신다면 Enterprise 요금제에서 제공되는 [Claude Security](https://claude.com/product/claude-security) 제품을 확인하세요. 플러그인은 GitLab이나 Bitbucket에 호스팅되거나 인바운드 연결을 허용하지 않는 네트워크에 있는 리포지토리 등 관리형 제품이 접근할 수 없는 코드에 접근할 수 있습니다.

또한 플러그인은 Claude Code에 이미 포함된 검토 도구와 구별됩니다. [보안 지침 플러그인](/docs/en/security-guidance)은 Claude가 코드를 작성하는 시점에 코드를 검토하고, [`/security-review`](/docs/en/commands#all-commands)는 현재 브랜치를 단일 패스로 검토하며, [Code Review](/docs/en/code-review)는 풀 리퀘스트를 검토합니다. 레이어가 어떻게 구성되는지는 [타 보안 도구와 플러그인의 관계](#how-the-plugin-fits-with-other-security-tools)를 참조하세요.

## 사전 요구 사항

플러그인을 실행하려면 다음이 필요합니다.

* {/* min-version: 2.1.154 */}스캔이 에이전트를 조율하는 데 사용하는 [동적 워크플로우](/docs/en/workflows)를 위해 유료 요금제의 Claude Code v2.1.154 이상 버전이 필요합니다. Pro에서는 `/config` 독의 동적 워크플로우 행에서 이를 켜세요.
* `PATH`에서 `python3`으로 사용할 수 있는 Python 3.9.6 이상 버전. `python3 --version`으로 확인하세요. 플러그인의 툴링은 Python 표준 라이브러리만 사용하므로 별도로 설치되는 것은 없습니다.
* Linux, macOS 또는 Windows.
* 변경 사항 스캔 및 발견 항목의 패치 변환을 위한 Git. 이러한 작업은 다른 버전 관리 시스템을 지원하지 않습니다. 전체 스캔은 버전 관리 여부와 관계없이 모든 디렉토리에서 작동합니다.

## 플러그인 설치하기

Claude Code 세션에서 [공식 Anthropic 마켓플레이스](/docs/en/discover-plugins#official-anthropic-marketplace)를 통해 설치합니다.

```text theme={null}
/plugin install claude-security@claude-plugins-official
```

<Note>
  Claude Code에서 마켓플레이스를 찾을 수 없다고 보고하면, 먼저 `/plugin marketplace add anthropics/claude-plugins-official`을 실행한 다음 다시 설치를 시도하세요.
</Note>

그런 다음 재시작 없이 보류 중인 플러그인 변경 사항을 적용하는 `/reload-plugins` 명령으로 현재 세션에서 플러그인을 활성화합니다.

```text theme={null}
/reload-plugins
```

플러그인이 이제 활성화되었으며, [코드베이스 스캔 및 문제 수정하기](#scan-and-fix-your-codebase)를 진행할 준비가 되었습니다.

### 플러그인 삭제하기

플러그인을 제거하려면 `/plugin` 메뉴에서 언인스톨하거나 터미널에서 `claude plugin uninstall claude-security`를 실행하세요.

## 코드베이스 스캔 및 문제 수정하기

플러그인은 코드베이스 스캔, 변경 사항 세트 스캔, 패치 제안 등 세 가지 작업 메뉴를 여는 `/claude-security` 명령 하나를 추가합니다. 일반적인 권장 흐름은 전체 스캔을 실행한 다음 발견 항목을 패치로 변환하는 것입니다.

<Steps>
  <Step title="Claude Security 메뉴 열기">
    `/claude-security`를 실행하고 **Scan codebase**를 선택합니다.
  </Step>

  <Step title="스캔 대상 선택">
    플러그인이 먼저 리포지토리를 읽은 후 전체 리포지토리 또는 집중 영역을 제안하며, 각 옵션의 파일 수와 상대적 비용이 표시됩니다. 전체 리포지토리를 선택하거나 "I don't know"라고 답하면 플러그인이 리포지토리 크기에 적합한 기본값을 선택합니다.
  </Step>

  <Step title="실행 확인">
    스캔은 시간이 걸릴 수 있고 상당한 수의 토큰을 사용할 수 있으며 완료될 때까지 Claude Code를 열어두어야 합니다. 확인하기 전까지는 아무것도 실행되지 않습니다.
  </Step>

  <Step title="보고서 읽기">
    스캔이 실행되는 동안 각 단계가 시작될 때마다 상황을 보고하며, 세부 정보는 [`/workflows`](/docs/en/workflows) 아래에서 확인할 수 있습니다. 결과는 [스캔 결과 읽기](#read-the-scan-results)에 설명된 대로 리포지토리의 타임스탬프 디렉토리에 저장됩니다.
  </Step>

  <Step title="발견 항목을 패치로 변환">
    `/claude-security`를 다시 실행하고 **Suggest patches**를 선택한 다음 해결할 발견 항목을 선택합니다. 검토된 패치는 보고서의 `patches/` 폴더에 저장됩니다. [발견 항목 수정하기](#fix-findings)에서 각 패치가 어떻게 생성되고 검토되는지 다룹니다.
  </Step>

  <Step title="수락한 패치 적용">
    각 패치를 셸에서 `git apply`를 사용하여 별도의 풀 리퀘스트로 적용합니다. 패치는 절대로 자동으로 적용되지 않습니다.
  </Step>
</Steps>

메뉴에서 시작할 필요는 없습니다. `/claude-security scan my branch`와 같은 명령 인수로 작업을 직접 요청하거나, "scan commit abc1234"와 같이 일반 언어로 요청할 수도 있습니다. 플러그인은 각 단계에서 권한 프롬프트 없이 스캔 에이전트가 진행할 수 있는 [자동 모드(auto mode)](/docs/en/permission-modes)에서 가장 잘 작동합니다. 작업이 시작될 때 플러그인이 이를 활성화하는 방법을 안내합니다.

### 변경 사항만 스캔하기

브랜치에 베이스에 없는 커밋이 있는 경우, `/claude-security` 메뉴는 해당 diff만 스캔하는 옵션을 제공하므로 머지하기 전에 브랜치를 확인할 수 있습니다. 또한 열려 있는 풀 리퀘스트 중 하나를 스캔하거나 "scan commit abc1234"와 같이 요청하여 단일 커밋을 스캔할 수도 있습니다. 커밋된 변경 사항만 스캔되므로 진행 중인 편집은 먼저 커밋하거나 스태시(stash)하세요. 작업 트리를 읽는 전체 스캔을 실행할 수도 있습니다.

변경 사항 스캔에는 git 리포지토리가 필요합니다. 버전 관리되지 않는 디렉토리의 전체 스캔은 여전히 작동합니다. 열려 있는 풀 리퀘스트를 찾는 단계는 네트워크에 접근하는 유일한 단계이며, 세션에 이미 GitHub CLI 실행 권한이 있고 `gh`가 로그인되어 있을 때만 제공됩니다.

### 대규모 코드베이스 범위 지정하기

대규모 리포지토리에서는 전체 트리가 아닌 한 번에 한 영역씩 스캔하세요. API 레이어나 인증 코드 등 플러그인이 제공하는 집중 범위 중 하나를 선택하면 선택한 범위에 맞춰 실행 규모가 조정됩니다. 보고서의 범주(coverage) 섹션에 검사된 부분과 검사되지 않은 부분이 설명됩니다. 언제든지 다른 영역에 대해 스캔을 다시 실행할 수 있습니다.

### 스캔 결과 읽기

모든 스캔은 리포지토리의 타임스탬프가 지정된 `CLAUDE-SECURITY-<timestamp>/` 디렉토리에 결과를 기록합니다.

* **`CLAUDE-SECURITY-RESULTS.md`**: `F1`과 같은 각 발견 항목의 ID, 영향, 익스플로잇 시나리오, 심각도, 신뢰도 및 권장 사항이 포함된 보고서
* **`CLAUDE-SECURITY-RESULTS.jsonl`**: 줄당 하나의 JSON 객체로 구성된 시스템 읽기용 형신의 동일한 발견 항목
* **`CLAUDE-SECURITY-REVISION-<commit>.json`**: 스캔된 커밋, 투입된 작업 수준, 미커밋 변경 사항이 스캔 대상 트리에 포함되었는지 여부, 실행이 얼마나 철저하게 검증되었는지를 기록하는 리비전 스탬프로, 보고서가 항상 설명하는 코드와 연결되도록 합니다. 버전 관리를 벗어난 스캔은 커밋 위치에 `UNVERSIONED`를 스탬프합니다.

이 디렉토리는 스캔이 체크아웃에 만드는 유일한 변경 사항이며, 자체 `.gitignore`를 포함하므로 실수로 `git add`를 실행해도 보고서가 커밋에 포함되지 않습니다. 감사 기록을 위해 보고서를 기록에 남기려면 해당 `.gitignore` 파일 하나만 삭제하고 다른 파일처럼 디렉토리를 커밋하세요.

발견 항목은 독립적인 검증자(verifier) 에이전트가 분석한 후에만 보고서에 나타나므로 보고서가 짧고 읽을 가치가 있게 유지됩니다. 스캔은 비결정적(nondeterministic)입니다. 동일한 코드에 대한 두 번의 스캔에서 서로 다른 발견 항목이 표출될 수 있습니다. 정기적으로 스캔을 실행하고 리비전 스탬프를 사용하여 각 보고서가 다루는 정확한 코드와 설정을 확인하세요.

## 발견 항목 수정하기

`/claude-security` 메뉴에서 **Suggest patches**를 선택하거나 "fix finding F3"과 같이 일반 언어로 요청하여 수정 흐름을 시작한 다음 보고서에서 해결할 발견 항목을 선택합니다. 패치는 커밋된 코드를 기반으로 생성되며, 보고서는 여전히 현재 보유한 코드를 설명해야 합니다. 코드가 변경된 발견 항목은 메모와 함께 건너뛰고 플러그인은 오래된 보고서에서 패치를 생성하는 대신 새로운 스캔을 제공합니다. 각 패치는 리포지토리의 임시 복사본에서 초안이 작성되므로 직접 패치를 적용할 때까지 소스 파일은 수정되지 않은 상태로 유지됩니다.

전달되기 전에 각 패치는 패치를 작성한 에이전트와 독립된 에이전트에 의해 검토되며, 해당 에이전트는 코드에 테스트가 있는 경우 변경 사항에 대해 프로젝트 테스트를 실행하고 도입될 수 있는 새로운 요소에 대해 diff를 직접 읽어봅니다. 패치는 검토에서 변경 사항이 한 가지 발견 항목을 해결하고, 새로운 취약점을 도입하지 않으며, 그렇지 않은 경우 동작을 변경하지 않고 유지함을 보증할 수 있을 때만 작성됩니다. 세 가지 모두를 보증할 수 없을 때는 패치 대신 이유를 설명하는 짧은 메모가 제공됩니다.

### 패치는 자동으로 적용되지 않습니다

패치를 적용하는 것은 항상 사용자의 결정입니다. 패치는 보고서의 `patches/` 폴더에 위치하며, 발견 항목당 하나의 `F<n>.patch`와 변경 사항을 설명하는 메모가 함께 제공됩니다. 셸에서 패치를 적용하거나 Claude에게 적용하고 풀 리퀘스트를 열도록 요청하세요.

```bash theme={null}
git apply CLAUDE-SECURITY-<timestamp>/patches/F1.patch
```

패치된 코드에 테스트가 없는 경우 패치 메모에 해당 내용이 안내되므로 테스트 통과 없이 검토가 실행되었음을 알 수 있습니다. 개별적으로 검토하고 테스트할 수 있도록 각 패치를 자체 풀 리퀘스트로 적용하세요.

## 타 보안 도구와 플러그인의 관계

Claude Security 플러그인은 [보안 지침 플러그인](/docs/en/security-guidance), [`/security-review`](/docs/en/commands#all-commands), [Code Review](/docs/en/code-review), 관리형 [Claude Security](https://claude.com/product/claude-security) 제품 및 기존 스캐너와 함께 심층 방어(defense-in-depth) 스택의 온디맨드 심층 스캔 레이어 역할을 합니다.

| 단계 | 도구 | 범위 |
| :--- | :--- | :--- |
| 세션 내부 | [보안 지침 플러그인](/docs/en/security-guidance) | Claude가 작성하는 코드의 공통 취약점, 동일 세션에서 수정 |
| 온디맨드, 단일 패스 | [`/security-review`](/docs/en/commands#all-commands) | 현재 브랜치에 대한 일회성 보안 패스 |
| 온디맨드, 심층 스캔 | Claude Security 플러그인 | 독립적으로 검토된 발견 항목 및 패치가 포함된 리포지토리/diff의 다중 에이전트 스캔 |
| 풀 리퀘스트 시 | [Code Review](/docs/en/code-review) (Team 및 Enterprise 요금제) | 전체 코드베이스 컨텍스트를 활용한 다중 에이전트 정확성 및 보안 검토 |
| 관리형 | [Claude Security](https://claude.com/product/claude-security) (Enterprise 요금제) | 연결된 리포지토리를 모니터링하는 호스팅 스캐닝 |
| CI 내부 | 기존 정적 분석 및 종속성 스캐너 | 언어별 규칙, 공급망 검사 및 정책 적용 |

플러그인은 기존 소스 코드 보안 도구를 대체하지 않습니다. 정적 분석, 종속성 스캐닝, 코드 검토와 함께 함께 실행하세요. 인간 보안 연구원이 하듯 코드를 추론하므로 이러한 도구가 제공하는 결정적 검사를 보완합니다.

## 문제 해결

**`/claude-security` 메뉴가 Python 경고와 함께 열립니다.** 플러그인은 `PATH`에 Python 3.9.6 이상의 `python3`이 필요합니다. `python3`을 전혀 찾을 수 없는 경우 설치될 때까지 Claude Security가 작동하지 않는다는 경고가 메뉴에 표시되며, `PATH`에 있는 첫 번째 `python3`이 더 이전 버전인 경우 발견된 버전이 경고에 표시됩니다. Python 3을 설치하거나 더 새로운 `python3`을 `PATH`에 최우선으로 지정한 다음 새 세션을 시작하세요.

## 관련 리소스

이 페이지에서 다루는 내용을 더 자세히 알아보려면 다음 리소스를 참조하세요.

* [보안 지침 플러그인](/docs/en/security-guidance): 동일 세션에서 Claude가 작성할 때 코드의 문제를 즉시 포착
* [Code Review](/docs/en/code-review): PR 시점의 다중 에이전트 검토 설정
* [Claude Security](https://claude.com/product/claude-security): 연결된 리포지토리를 모니터링하는 관리형 서비스
* [Claude Code 보안](/docs/en/security): Claude Code가 신뢰, 권한 및 보호 조치에 접근하는 방식
* [플러그인 탐색 및 설치](/docs/en/discover-plugins#official-anthropic-marketplace): 기타 공식 플러그인 둘러보기
