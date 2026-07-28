> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 코드 리뷰 (Code Review)

> 전체 코드베이스의 다중 에이전트 분석을 사용하여 로직 오류, 보안 취약점 및 리그레션을 포착하는 자동화된 PR 리뷰를 설정하세요.

<Note>
  Code Review는 현재 리서치 프리뷰 상태로, [Team 및 Enterprise](https://claude.ai/admin-settings/claude-code) 구독에서 이용할 수 있습니다. [데이터 보존 보장 없음(Zero Data Retention)](/docs/en/zero-data-retention)이 활성화된 조직에서는 이용할 수 없습니다. 다른 요금제에서는 `/code-review` 명령으로 [로컬에서 diff를 리뷰](#review-a-diff-locally)할 수 있습니다.
</Note>

Code Review는 GitHub 풀 리퀘스트를 분석하고 문제를 포착한 코드 줄에 인라인 주석으로 발견 항목을 게시합니다. 전문 에이전트 군단이 전체 코드베이스 컨텍스트에서 코드 변경 사항을 검사하여 로직 오류, 보안 취약점, 미처리 엣지 케이스 및 미세한 리그레션을 찾습니다.

발견 항목은 심각도별로 태그가 지정되며 PR을 승인하거나 차단하지 않으므로 기존 리뷰 워크플로우가 그대로 유지됩니다. 리포지토리에 `CLAUDE.md` 또는 `REVIEW.md` 파일을 추가하여 Claude가 지적하는 내용을 조정할 수 있습니다.

이 관리형 서비스 대신 자체 CI 인프라에서 Claude를 실행하려면 [GitHub Actions](/docs/en/github-actions) 또는 [GitLab CI/CD](/docs/en/gitlab-ci-cd)를 참조하세요. 자체 호스팅 GitHub 인스턴스의 리포지토리는 [GitHub Enterprise Server](/docs/en/github-enterprise-server)를 참조하세요.

이 페이지에서 다루는 내용:

* [리뷰 작동 방식](#how-reviews-work)
* [설정](#set-up-code-review)
* `@claude review` 및 `@claude review always`로 [수동으로 리뷰 트리거하기](#manually-trigger-reviews)
* `CLAUDE.md` 및 `REVIEW.md`로 [리뷰 커스터마이징하기](#customize-reviews)
* [요금제 (Pricing)](#pricing)
* 실패한 실행 및 누락된 주석에 대한 [문제 해결](#troubleshooting)
* `/code-review` 명령으로 [로컬에서 diff 리뷰하기](#review-a-diff-locally)

<Note>
  GitHub 앱을 설치하지 않고 터미널에서 로컬로 diff를 리뷰하려면 모든 Claude Code 세션에서 `/code-review` 명령을 실행하세요. [로컬에서 diff 리뷰하기](#review-a-diff-locally)를 참조하세요.
</Note>

## 리뷰 작동 방식

소유자(Owner)가 조직에 대해 [Code Review를 활성화](#set-up-code-review)하면, 리포지토리에 구성된 동작에 따라 PR이 열릴 때, 모든 푸시 시 또는 수동으로 요청될 때 리뷰가 트리거됩니다. `@claude review`를 주석으로 작성하면 모든 모드에서 [PR 리뷰가 시작](#manually-trigger-reviews)됩니다.

리뷰가 실행되면 여러 에이전트가 Anthropic 인프라에서 병렬로 diff 및 주변 코드를 분석합니다. 각 에이전트는 서로 다른 유형의 문제를 검색한 다음, 검증 단계에서 후보를 실제 코드 동작과 비교하여 오탐(false positive)을 필터링합니다. 결과는 중복 제거되고 심각도순으로 정렬되어 문제가 발견된 특정 줄에 인라인 주석으로 게시되며 리뷰 본문에 요약이 제공됩니다. 문제가 발견되지 않으면 Code Review는 GitHub 체크 실행을 업데이트하여 문제가 감지되지 않았음을 보여줍니다. Claude는 PR에 짧은 확인 주석을 게시할 수도 있습니다.

리뷰 비용은 PR 크기 및 복잡성에 따라 확장되며, 평균 20분 내에 완료됩니다. 소유자는 [분석 대시보드](#view-usage)를 통해 리뷰 활동 및 지출을 모니터링할 수 있습니다.

### 심각도 수준

각 발견 항목에는 심각도 수준 태그가 지정됩니다.

| 표식 | 심각도 | 의미 |
| :--- | :--- | :--- |
| 🔴 | Important (중요) | 머지 전에 수정해야 하는 버그 |
| 🟡 | Nit (자잘한 지적) | 사소한 문제로, 수정할 가치는 있지만 블로킹되지는 않음 |
| 🟣 | Pre-existing (기존 문제) | 코드베이스에 존재하지만 이 PR로 인해 도입되지는 않은 버그 |

발견 항목에는 Claude가 왜 해당 문제를 지적했는지, 어떻게 문제를 검증했는지 이해하기 위해 펼쳐볼 수 있는 접혀진 추론 확장 섹션이 포함되어 있습니다.

### 발견 항목 평가 및 답글

Claude의 각 리뷰 주석에는 👍 및 👎가 이미 첨부된 상태로 도착하므로 원클릭 평가를 위해 GitHub UI에 두 버튼이 모두 표시됩니다. 발견 항목이 유용한 경우 👍를 누르고, 틀렸거나 노이즈인 경우 👎를 누르세요. Anthropic은 PR이 머지된 후 반응 횟수를 수집하고 리뷰어를 조정하는 데 사용합니다. 반응이 재리뷰를 트리거하거나 PR의 내용 항목을 변경하지는 않습니다.

인라인 주석에 답글을 달아도 Claude가 응답하거나 PR을 업데이트하도록 요청되지 않습니다. 발견 항목에 대응하려면 코드를 수정하고 푸시하세요. PR이 푸시 트리거 리뷰 구독 상태인 경우, 문제가 수정되면 다음 실행 시 해당 스레드가 자동으로 해결됩니다. 푸시하지 않고 새로운 리뷰를 요청하려면 [PR 최상위 주석](#manually-trigger-reviews)으로 `@claude review`를 남기세요.

### 체크 실행 출력

인라인 리뷰 주석 외에도 각 리뷰는 CI 체크 항목과 함께 표시되는 **Claude Code Review** 체크 실행을 채웁니다. **Details** 링크를 펼치면 심각도별로 정렬된 모든 발견 항목의 요약을 한 곳에서 볼 수 있습니다.

| 심각도 | 파일:줄 | 문제 |
| :--- | :--- | :--- |
| 🔴 Important | `src/auth/session.ts:142` | 토큰 새로고침이 로그아웃과 경쟁 상태(race condition)를 일으켜 오래된 세션이 활성 상태로 남음 |
| 🟡 Nit | `src/auth/session.ts:88` | `parseExpiry`가 잘못된 형식의 입력에서 자동으로 0을 반환함 |

또한 각 발견 항목은 **Files changed** 탭에 주석(annotation)으로 표시되어 관련 diff 줄에 직접 기록됩니다. Important 발견 항목은 빨간색 표식으로, Nit은 노란색 경고로, 기존 버그는 회색 알림으로 렌더링됩니다. 주석과 심각도 테이블은 인라인 리뷰 주석과 독립적으로 체크 실행에 기록되므로, 이동된 줄의 인라인 주석을 GitHub이 거부하더라도 계속 이용할 수 있습니다.

체크 실행은 항상 중립(neutral) 결과로 완료되므로 브랜치 보호 규칙을 통해 머지가 차단되지 않습니다. Code Review 발견 항목에 따라 머지를 제한하려면 자체 CI에서 체크 실행 출력의 심각도 내역을 읽어오세요. Details 텍스트의 마지막 줄은 워크플로우가 `gh` 및 jq로 구문 분석할 수 있는 시스템 읽기용 주석입니다. 체크 실행 ID를 찾으려면 `gh api repos/OWNER/REPO/commits/<commit-sha>/check-runs --jq '.check_runs[] | {id, name}'`로 커밋의 체크 실행을 나열하고 `Claude Code Review` 실행의 `id`를 가져오세요. `OWNER`, `REPO` 및 `CHECK_RUN_ID`를 사용자 리포지토리 소유자, 리포지토리 이름 및 해당 ID로 교체하세요.

```bash theme={null}
gh api repos/OWNER/REPO/check-runs/CHECK_RUN_ID \
  --jq '.output.text | split("bughunter-severity: ")[1] | split(" -->")[0] | fromjson'
```

이는 심각도별 개수가 포함된 JSON 객체(예: `{"normal": 2, "nit": 1, "pre_existing": 0}`)를 반환합니다. `normal` 키는 Important 발견 항목 수이며, 0이 아닌 값은 Claude가 머지 전에 수정할 가치가 있는 버그를 하나 이상 찾았음을 의미합니다.

### Code Review 검사 항목

기본적으로 Code Review는 서식 지정 선호도나 누락된 테스트 커버리지가 아니라 프로덕션을 중단시킬 수 있는 버그와 같은 정확성에 중점을 둡니다. 리포지토리에 [지침 파일 추가](#customize-reviews)를 진행하여 검사 범위를 확장할 수 있습니다.

## Code Review 설정하기

소유자(Owner)가 조직에 대해 Code Review를 한 번 활성화하고 포함할 리포지토리를 선택합니다.

<Steps>
  <Step title="Claude Code 관리자 설정 열기">
    [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)로 이동하여 Code Review 섹션을 찾습니다. Claude 조직의 Owner 또는 Primary Owner 역할과 GitHub 조직에 GitHub 앱을 설치할 수 있는 권한이 필요합니다.
  </Step>

  <Step title="설정 시작">
    **Setup**을 클릭합니다. GitHub 앱 설치 흐름이 시작됩니다.
  </Step>

  <Step title="Claude GitHub 앱 설치">
    안내에 따라 GitHub 조직에 Claude GitHub 앱을 설치합니다. 앱은 다음 리포지토리 권한을 요청합니다.

    * **Contents**: 읽기 및 쓰기
    * **Issues**: 읽기 및 쓰기
    * **Pull requests**: 읽기 및 쓰기

    Code Review는 콘텐츠 읽기 권한과 풀 리퀘스트 쓰기 권한을 사용합니다. 더 광범위한 권한 세트는 나중에 활성화할 경우 [GitHub Actions](/docs/en/github-actions)도 지원합니다.
  </Step>

  <Step title="리포지토리 선택">
    Code Review를 활성화할 리포지토리를 선택합니다. 리포지토리가 보이지 않는 경우 설치 중에 Claude GitHub 앱에 접근 권한을 부여했는지 확인하세요. 나중에 더 많은 리포지토리를 추가할 수 있습니다.
  </Step>

  <Step title="리포지토리별 리뷰 트리거 설정">
    설정이 완료된 후 Code Review 섹션의 테이블에 리포지토리가 표시됩니다. 각 리포지토리에 대해 **Review Behavior** 드롭다운을 사용하여 리뷰 실행 시점을 선택하세요.

    * **Once after PR creation**: PR이 열리거나 리뷰 준비 완료(ready for review)로 표시될 때 리뷰가 한 번 실행됨
    * **After every push**: PR 브랜치에 푸시할 때마다 리뷰가 실행되어 PR이 진행됨에 따라 새 문제를 포착하고 지적된 문제를 수정하면 스레드를 자동 해결함
    * **Manual**: 누군가 [PR에 `@claude review` 주석을 남길 때만](#manually-trigger-reviews) 리뷰가 시작됨; `@claude review always`는 리뷰를 시작하고 후속 푸시 시 PR을 리뷰 구독에 등록함

    Every push 리뷰는 가장 많은 리뷰를 실행하며 비용이 가장 많이 듭니다. Manual 모드는 특정 PR만 리뷰 대상으로 지정하려는 트래픽이 많은 리포지토리나 PR이 준비되었을 때만 리뷰를 시작하려는 경우에 유용합니다.
  </Step>
</Steps>

리포지토리 테이블에는 최근 활동을 기반으로 한 각 리포지토리의 리뷰당 평균 비용도 표시됩니다. 행 작업 메뉴를 사용하여 리포지토리별로 Code Review를 켜거나 끄거나 리포지토리를 완전히 제거하세요.

설정을 확인하려면 테스트 PR을 여세요. 자동 트리거를 선택한 경우 몇 분 이내에 **Claude Code Review**라는 체크 실행이 나타납니다. Manual을 선택한 경우 PR에 `@claude review` 주석을 남겨 첫 번째 리뷰를 시작하세요. 체크 실행이 나타나지 않으면 리포지토리가 관리자 설정에 나열되어 있는지, Claude GitHub 앱이 접근 권한을 가지고 있는지 확인하세요.

## 수동으로 리뷰 트리거하기

주석 명령은 필요에 따라 리뷰를 시작합니다. 리포지토리에 구성된 트리거와 관계없이 작동하므로 Manual 모드에서 특정 PR을 리뷰 대상으로 선택하거나 다른 모드에서 즉시 재리뷰를 받는 데 사용할 수 있습니다.

| 명령 | 동작 |
| :--- | :--- |
| `@claude review` | 향후 푸시 구독 없이 단일 리뷰 시작 |
| `@claude review always` | 리뷰를 시작하고 향후 푸시 트리거 리뷰에 PR을 구독 등록 |
| `@claude review once` | `@claude review`와 동일: 구독 없이 단일 리뷰 시작 |

Manual 모드로 설정된 리포지토리의 우선순위가 높은 PR 등 PR에 대한 모든 후속 푸시마다 새 리뷰가 시작되도록 하려면 `@claude review always`를 사용하세요. 기본 명령은 PR을 구독시키지 않으므로 이후 푸시가 리뷰를 트리거할지 여부를 변경하지 않고 일회성 2차 의견을 요청할 수 있습니다.

<Note>
  2026년 7월 업데이트 이전에는 `@claude review`가 PR을 푸시 트리거 리뷰에 구독시켰습니다. 해당 동작에 의존했다면 대신 `@claude review always`를 주석으로 남기세요. `@claude review once`는 여전히 동작하며 기본 명령과 동일하게 동작합니다.
</Note>

이러한 명령이 리뷰를 트리거하려면:

* diff 줄의 인라인 주석이 아닌 최상위 PR 주석으로 게시하세요.
* 주석 시작 부분에 명령을 두고, `once` 또는 `always`는 명령의 나머지 부분과 동일한 줄에 두세요.
* 리포지토리에 대한 소유자(owner), 멤버(member) 또는 협력자(collaborator) 접근 권한이 있어야 합니다.
* PR이 열려(open) 있어야 합니다.

자동 트리거와 달리 수동 트리거는 드래프트 상태와 관계없이 지금 리뷰를 원한다는 명시적 요청이므로 드래프트 PR에서도 실행됩니다.

해당 PR에서 리뷰가 이미 실행 중인 경우 진행 중인 리뷰가 완료될 때까지 요청이 대기열에 추가됩니다. PR의 체크 실행을 통해 진행 상황을 모니터링할 수 있습니다.

## 리뷰 커스터마이징하기

Code Review는 무엇을 지적할지 안내하기 위해 리포지토리에서 두 개의 파일을 읽습니다. 이 파일들은 리뷰에 미치는 영향력의 강도가 다릅니다.

* **`CLAUDE.md`**: 리뷰뿐만 아니라 모든 작업에 Claude Code가 사용하는 공유 프로젝트 지침입니다. Code Review는 이를 프로젝트 컨텍스트로 읽고 새롭게 도입된 위반 사항을 Nit 수준으로 지적합니다.
* **`REVIEW.md`**: 리뷰 전용 지침으로, 리뷰 파이프라인의 모든 에이전트에 최우선 순위로 직접 주입됩니다. 지적 대상, 심각도 및 발견 항목 보고 방식을 변경하는 데 사용하세요.

### CLAUDE.md

Code Review는 리포지토리의 `CLAUDE.md` 파일을 읽고 새롭게 도입된 위반 사항을 [Nit 수준](#severity-levels) 발견 항목으로 취급합니다. 이는 양방향으로 작동합니다. PR이 `CLAUDE.md` 설명을 오래된 것으로 만드는 방식으로 코드를 변경하면 Claude는 문서도 업데이트해야 한다고 지적합니다.

Claude는 디렉토리 계층 구조의 모든 수준에서 `CLAUDE.md` 파일을 읽으므로 하위 디렉토리의 `CLAUDE.md` 규칙은 해당 경로 아래의 파일에만 적용됩니다. `CLAUDE.md` 작동 방식에 대한 자세한 내용은 [메모리 문서](/docs/en/memory)를 참조하세요.

일반 Claude Code 세션에 적용하고 싶지 않은 리뷰 전용 지침의 경우 대신 [`REVIEW.md`](#review-md)를 사용하세요.

### REVIEW.md

`REVIEW.md`는 리포지토리 루트에 위치하여 리포지토리에서 Code Review가 동작하는 방식을 재정의하는 파일입니다. 해당 내용은 기본 리뷰 지침보다 우선하여 가장 높은 우선순위 지침 블록으로 리뷰 파이프라인의 모든 에이전트의 시스템 프롬프트에 성문 그대로 주입됩니다.

성문 그대로 붙여넣어지므로 `REVIEW.md`는 일반 지침입니다. [`@` 임포트 구문](/docs/en/memory#import-additional-files)은 확장되지 않으며 참조된 파일은 프롬프트로 읽혀지지 않습니다. 적용하려는 규칙을 파일에 직접 작성하세요.

#### 조정할 수 있는 항목

`REVIEW.md`는 자유 형식의 마크다운이므로 리뷰 지침으로 표현할 수 있는 모든 것이 범위에 포함됩니다. 아래 패턴이 실제 실행에서 가장 큰 영향을 미칩니다.

**심각도**: 리포지토리에 대해 🔴 Important의 의미를 재정의합니다. 기본 보정은 프로덕션 코드를 대상으로 합니다. 문서 리포지토리, 구성 리포지토리 또는 프로토타입은 훨씬 더 좁은 정의를 원할 수 있습니다. 어떤 클래스의 발견 항목이 Important이고 어떤 항목이 최대 Nit인지 명시적으로 서술하세요. 또한 다른 방향으로 에스컬레이션할 수도 있습니다(예: 모든 `CLAUDE.md` 위반 사항을 기본 Nit 대신 Important로 취급).

**Nit 수량**: 단일 리뷰가 게시하는 🟡 Nit 주석 수를 제한합니다. 줄글 및 구성 파일은 무한히 다듬어질 수 있습니다. "최대 5개의 Nit만 보고하고 나머지는 요약에 개수로 언급"과 같은 제한을 두면 리뷰를 실용적으로 유지할 수 있습니다.

**건너뛰기 규칙**: Claude가 발견 항목을 게시하지 않아야 하는 경로, 브랜치 패턴 및 발견 항목 카테고리를 나열합니다. 일반적인 대상은 생성된 코드, 락파일(lockfile), 벤더링된 종속성, 머신이 작성한 브랜치, 린팅이나 스펠링 체크와 같이 CI가 이미 적용 중인 항목입니다. 어느 정도 리뷰가 필요하지만 완전한 검수까지는 필요하지 않은 경로의 경우 완전히 건너뛰는 대신 더 높은 기준을 설정하세요: "`scripts/` 내부에서는 거의 확실하고 심각한 경우에만 보고."

**리포지토리 특화 검사**: "새 API 라우트에는 통합 테스트가 있어야 함"과 같이 모든 PR에서 지적되기를 원하는 규칙을 추가합니다. `REVIEW.md`가 최우선 순위로 주입되기 때문에 긴 `CLAUDE.md`에 있는 동일한 규칙보다 더 확실하게 적용됩니다.

**검증 기준**: 발견 항목 카테고리가 게시되기 전에 증거를 요구합니다. 예를 들어 "동작 주장은 명명 규칙에서의 추론이 아니라 소스 코드 내 `file:line` 인용이 필요함"은 불필요한 질의응답 소모를 줄여줍니다.

**재리뷰 수렴**: PR이 이미 리뷰되었을 때 Claude가 어떻게 동작할지 지시합니다. "첫 번째 리뷰 이후에는 새 Nit을 억제하고 Important 발견 항목만 게시"와 같은 규칙은 한 줄 수정이 스타일 문제만으로 7번째 라운드까지 이어지는 것을 방지합니다.

**요약 형태**: 리뷰 본문이 `2 factual, 4 style`과 같이 한 줄 집계로 시작하고, 결함이 없는 경우 "no factual issues"로 시작하도록 요청합니다. 작성자는 세부 정보를 보기 전에 작업의 윤곽을 알고 싶어 합니다.

#### 예시

이 `REVIEW.md`는 백엔드 서비스에 대한 심각도를 재보정하고, Nit 수량을 제한하며, 생성된 파일을 건너뛰고, 리포지토리 특화 검사를 추가합니다.

```markdown theme={null}
# Review instructions

## What Important means here

Reserve Important for findings that would break behavior, leak data,
or block a rollback: incorrect logic, unscoped database queries, PII
in logs or error messages, and migrations that aren't backward
compatible. Style, naming, and refactoring suggestions are Nit at
most.

## Cap the nits

Report at most five Nits per review. If you found more, say "plus N
similar items" in the summary instead of posting them inline. If
everything you found is a Nit, lead the summary with "No blocking
issues."

## Do not report

- Anything CI already enforces: lint, formatting, type errors
- Generated files under `src/gen/` and any `*.lock` file
- Test-only code that intentionally violates production rules

## Always check

- New API routes have an integration test
- Log lines don't include email addresses, user IDs, or request bodies
- Database queries are scoped to the caller's tenant
```

#### 핵심 위주로 유지하기

길이에는 비용이 따릅니다. 긴 `REVIEW.md`는 가장 중요한 규칙을 희석시킵니다. 리뷰 동작을 변경하는 지침으로 유지하고, 일반적인 프로젝트 컨텍스트는 `CLAUDE.md`에 두세요.

## 사용량 보기

조직 전체의 Code Review 활동을 보려면 [claude.ai/analytics/code-review](https://claude.ai/analytics/code-review)로 이동하세요. 대시보드에 표시되는 항목:

| 섹션 | 표시 내용 |
| :--- | :--- |
| PRs reviewed | 선택한 시간 범위 동안 리뷰된 일별 풀 리퀘스트 수 |
| Cost weekly | Code Review에 지출된 주간 비용 |
| Feedback | 개발자가 문제를 해결하여 자동 해결된 리뷰 주석 수 |
| Repository breakdown | 리포지토리별 리뷰된 PR 수 및 해결된 주석 수 |

관리자 설정의 리포지토리 테이블에는 각 리포지토리의 리뷰당 평균 비용도 표시됩니다. 대시보드 비용 수치는 활동 모니터링을 위한 추정치입니다. 청구서와 일치하는 정밀 지출은 Anthropic 청구서를 참조하세요.

## 요금제 (Pricing)

Code Review는 토큰 사용량을 기준으로 청구됩니다. 각 리뷰는 PR 크기, 코드베이스 복잡성 및 검증이 필요한 문제 수에 따라 확장되어 평균 \$15-25의 비용이 듭니다. Code Review 사용량은 [사용 크레딧](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)을 통해 별도로 청구되며 요금제에 포함된 기본 사용량에 반영되지 않습니다.

선택한 리뷰 트리거는 전체 비용에 영향을 미칩니다.

* **Once after PR creation**: PR당 한 번 실행됨
* **After every push**: 각 푸시마다 실행되어 푸시 횟수만큼 비용이 배가됨
* **Manual**: 누군가 PR에 `@claude review` 주석을 달 때까지 리뷰 없음

Once after PR creation 또는 Manual 모드에서 `@claude review always` 주석을 남기면 [PR이 푸시 트리거 리뷰 구독에 등록](#manually-trigger-reviews)되므로 해당 주석 이후 푸시당 추가 비용이 발생합니다. After every push 모드에서는 푸시가 이미 리뷰를 트리거하므로 구독 등록이 푸시당 비용을 변경하지 않습니다. `@claude review` 주석을 남기면 향후 푸시 구독 없이 단일 리뷰만 실행됩니다.

조직이 다른 Claude Code 기능에 Amazon Bedrock 또는 Google Cloud Agent Platform을 사용하는지 여부와 관계없이 비용은 Anthropic 청구서에 표시됩니다. Code Review의 월간 지출 한도를 설정하려면 [claude.ai/admin-settings/usage](https://claude.ai/admin-settings/usage)로 이동하여 Claude Code Review 서비스에 대한 한도를 구성하세요.

[분석](#view-usage)의 주간 비용 차트 또는 관리자 설정의 리포지토리별 평균 비용 열을 통해 지출을 모니터링하세요.

## 문제 해결

리뷰 실행은 최선형(best-effort)입니다. 실패한 실행이 PR을 차단하지는 않지만 자동으로 재시도되지도 않습니다. 이 섹션에서는 실패한 실행에서 복구하는 방법과 체크 실행에서 보고되었으나 찾을 수 없는 문제를 찾는 곳을 다룹니다.

### 실패하거나 타임아웃된 리뷰 다시 트리거하기

리뷰 인프라에 내부 오류가 발생하거나 시간 제한을 초과하면 체크 실행이 **Code review encountered an error** 또는 **Code review timed out**이라는 제목으로 완료됩니다. 결과는 여전히 중립이므로 머지가 차단되지는 않지만 발견 항목이 게시되지 않습니다.

리뷰를 다시 실행하려면 PR에 `@claude review` 주석을 남기세요. 향후 푸시 구독 없이 새 리뷰가 시작됩니다. PR이 이미 푸시 트리거 리뷰를 구독 중인 경우 새 커밋을 푸시해도 새 리뷰가 시작됩니다.

GitHub Checks 탭의 **Re-run** 버튼은 Code Review를 다시 트리거하지 않습니다. 대신 주석 명령이나 새 푸시를 사용하세요.

### 리뷰가 실행되지 않고 PR에 지출 한도 메시지가 표시됨

조직의 월간 지출 한도에 도달하면 Code Review는 리뷰가 건너뛰어졌음을 설명하는 단일 주석을 PR에 게시합니다. 다음 청구 주기가 시작될 때 자동으로 리뷰가 재개되거나 관리자가 [claude.ai/admin-settings/usage](https://claude.ai/admin-settings/usage)에서 한도를 증액하면 즉시 재개됩니다.

### 인라인 주석으로 표시되지 않는 문제 찾기

체크 실행 제목에 문제가 발견되었다고 되어 있지만 diff에서 인라인 리뷰 주석이 보이지 않는 경우, 발견 항목이 표출되는 다른 위치를 확인하세요.

* **체크 실행 Details**: Checks 탭에서 Claude Code Review 체크 옆의 **Details**를 클릭하세요. 심각도 테이블에는 인라인 주석이 수락되었는지 여부와 관계없이 파일, 줄 및 요약과 함께 모든 발견 항목이 나열됩니다.
* **Files changed 주석(annotations)**: PR의 **Files changed** 탭을 보세요. 발견 항목은 리뷰 주석과 별도로 diff 줄에 직접 첨부된 주석(annotation)으로 렌더링됩니다.
* **리뷰 본문**: 리뷰가 실행되는 동안 PR에 푸시한 경우 일부 발견 항목이 현재 diff에 더 이상 존재하지 않는 줄을 참조할 수 있습니다. 이러한 항목은 인라인 주석이 아닌 리뷰 본문 텍스트의 **Additional findings** 제목 아래에 표시됩니다.

## 로컬에서 diff 리뷰하기

[`/code-review` 명령](/docs/en/commands)은 GitHub 앱을 설치하지 않고 터미널에서 diff를 리뷰합니다. 모든 Claude Code 세션에서 실행할 수 있으며, 정확성 버그 및 {/* min-version: 2.1.151 */}재사용, 단순화 및 효율성 클린업 항목을 보고합니다. 기본적으로 로컬 리뷰는 업스트림 이전의 브랜치 커밋과 작업 트리의 미커밋 변경 사항을 다룹니다. 발견 항목을 인라인 PR 주석으로 게시하려면 `--comment`를 전달하고, 리뷰 후 작업 트리에 발견 항목을 적용하려면 `--fix`를 전달하세요.

로컬 명령은 다른 Claude Code 세션과 마찬가지로 `CLAUDE.md`를 따르지만 [`REVIEW.md`](#review-md)는 읽지 않습니다.

낮은 [작업 투입 수준(effort level)](/docs/en/model-config#adjust-effort-level)은 더 적지만 신뢰도가 높은 발견 항목을 반환하는 반면, `high`에서 `max`까지는 더 광범위한 범위를 제공하며 불확실한 발견 항목이 포함될 수 있습니다. 작업 투입 인수가 없으면 리뷰는 세션의 현재 투입 수준을 사용합니다. 기본 diff 이외의 항목을 리뷰하려면 타겟(파일 경로, PR 번호, 브랜치 이름 또는 `main...my-feature`와 같은 참조 범위)을 전달하세요. 참조 범위 형식은 브랜치의 업스트림 구성 방식에 관계없이 `my-feature`에서 `main`으로의 풀 리퀘스트에 포함될 커밋된 diff를 리뷰합니다.

`/code-review ultra --fix`는 클라우드에서 더 깊이 있는 [ultrareview](/docs/en/ultrareview)를 실행한 다음, 세션으로 다시 돌아왔을 때 작업 트리에 발견 항목을 적용합니다. Ultrareview는 자체 범위를 사용합니다: 리포지토리의 기본 브랜치에 대한 현재 브랜치 및 작업 트리의 미커밋/스테이징된 변경 사항. 다른 베이스와 비교하려면 `/code-review ultra develop`과 같이 브랜치 이름을 전달하세요.

Ultrareview는 claude.ai 계정 인증이 필요하며 Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry 또는 데이터 보존 보장이 활성화된 조직에서는 이용할 수 없습니다. Ultrareview를 사용할 수 없는 경우 `/code-review ultra`는 세션 내에서 대신 로컬 리뷰를 실행합니다.

이 명령은 기본적으로 수정 사항을 적용하던 v2.1.147 이전에는 `/simplify`라는 이름이었습니다. {/* min-version: 2.1.154 */}v2.1.154부터 `/simplify`는 버그를 탐색하지 않고 수정 사항만 적용하는 별도의 정리 전용 리뷰를 실행합니다. 버그 탐색을 위해 `/simplify`를 스크립팅한 경우 동일하게 유지된 `/code-review --fix`로 전환하세요.

## 관련 리소스

Code Review는 Claude Code의 나머지 기능과 함께 작동하도록 설계되었습니다. PR을 열기 전에 로컬에서 리뷰를 실행하고 싶거나, 자체 호스팅 설정이 필요하거나, `CLAUDE.md`가 도구 전반에서 Claude의 동작을 어떻게 형성하는지 더 깊이 파고들고 싶다면 다음 페이지가 좋은 다음 단계입니다.

* [명령](/docs/en/commands): 푸시하기 전에 로컬 Claude Code 세션에서 `/code-review`를 실행하여 diff 확인
* [GitHub Actions](/docs/en/github-actions): 코드 리뷰를 넘어선 커스텀 자동화를 위해 자체 GitHub Actions 워크플로우에서 Claude 실행
* [GitLab CI/CD](/docs/en/gitlab-ci-cd): GitLab 파이프라인을 위한 자체 호스팅 Claude 연동
* [메모리](/docs/en/memory): Claude Code 전반에서 `CLAUDE.md` 파일이 작동하는 방식
* [분석](/docs/en/analytics): 코드 리뷰를 넘어선 Claude Code 사용량 추적
