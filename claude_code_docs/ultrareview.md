> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Ultrareview로 버그 찾기

> 병합 전 버그를 찾고 검증하기 위해 /code-review ultra를 사용하여 클라우드에서 심층 멀티 에이전트 코드 리뷰를 실행하세요.

<Note>
  Ultrareview는 리서치 프리뷰(research preview) 기능입니다. 기능, 가격 및 가용성은 피드백에 따라 변경될 수 있습니다. 명령어는 `/code-review ultra`입니다. 계정에서 ultrareview를 사용할 수 있게 되면 `/ultrareview`가 별칭으로 제공됩니다.
</Note>

Ultrareview는 Claude Code on the web 인프라에서 실행되는 심층 코드 리뷰입니다. `/code-review ultra`를 실행하면 Claude Code가 브랜치 또는 풀 리퀘스트의 버그를 찾기 위해 원격 샌드박스에서 리뷰어 에이전트 플릿(fleet)을 스폰합니다.

로컬 `/code-review` 또는 `/review`와 비교했을 때 ultrareview는 다음을 제공합니다:

* **더 높은 신호 대 잡음비(Higher signal)**: 보고된 모든 발견 사항은 독립적으로 재현되고 검증되므로 결과는 스타일 제안보다는 실제 버그에 집중됩니다
* **더 넓은 커버리지**: 더 큰 리뷰어 에이전트 플릿이 변경 사항을 병렬로 탐색하므로 로컬 리뷰에서 놓칠 수 있는 문제를 표출합니다
* **로컬 리소스 사용 없음**: 리뷰가 원격 샌드박스에서 완전히 실행되므로 실행되는 동안 터미널은 다른 작업을 위해 자유롭게 유지됩니다

Ultrareview는 Claude Code on the web 인프라에서 실행되므로 claude.ai 계정 인증이 필요합니다. API 키로만 로그인한 경우 먼저 `/login`을 실행하고 claude.ai로 인증하세요. Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry에서 Claude Code를 사용할 때는 Ultrareview를 사용할 수 없으며, Zero Data Retention을 활성화한 조직도 사용할 수 없습니다. Ultrareview를 사용할 수 없을 때 `/code-review ultra`는 세션에서 로컬 리뷰를 대신 실행합니다.

## CLI에서 ultrareview 실행하기

Claude Code CLI의 모든 git 리포지토리에서 리뷰를 시작하세요.

```text theme={null}
/code-review ultra
```

인수 없이 실행하면 ultrareview는 작업 트리의 커밋되지 않고 스테이징된 변경 사항을 포함하여 현재 브랜치와 기본 브랜치 간의 diff를 리뷰합니다. Claude Code는 리포지토리 상태를 번들링하여 리뷰를 위해 원격 샌드박스로 업로드합니다. 통합 브랜치가 `develop` 또는 `trunk`인 리포지토리에서와 같이 다른 베이스(base)와 비교하려면 대신 브랜치 이름을 전달하세요: `/code-review ultra develop`. {/* min-version: 2.1.212 */} `origin`에만 존재하는 베이스 브랜치를 가져오며 오타가 있는 이름은 가장 가까운 브랜치를 제안합니다. 두 동작 모두 Claude Code v2.1.212 이상이 필요합니다.

두 기록이 관련이 없는 경우와 같이 브랜치가 베이스 브랜치와 병합 베이스(merge base)를 공유하지 않는 경우 Claude Code는 대신 리포지토리의 모든 추적 대상 파일을 리뷰하겠다고 제안합니다. 전체 리포지토리 폴백은 전체 클론이 필요하며 브랜치 리뷰와 동일한 크기 제한이 적용됩니다. v2.1.214 이전에는 `/code-review ultra`가 병합 베이스 없이 실행되는 것을 거부했습니다.

대신 GitHub 풀 리퀘스트를 리뷰하려면 PR 번호를 전달하세요.

```text theme={null}
/code-review ultra 1234
```

이 명령은 `#1234`, `PR 1234`, 붙여넣은 PR URL도 허용합니다. 붙여넣은 URL은 현재 디렉터리의 리포지토리를 가리켜야 합니다. v2.1.212 이전에는 이 명령이 순수 숫자만 허용하고 다른 형식은 브랜치가 아니라는 오류와 함께 거부했습니다.

PR 모드에서 원격 샌드박스는 로컬 작업 트리를 번들링하는 대신 호스트에서 풀 리퀘스트를 직접 클론합니다. PR 모드는 `github.com`의 리포지토리 및 소유자가 Claude Code에 연결한 [GitHub Enterprise Server](/docs/en/github-enterprise-server) 인스턴스에서 작동합니다.

<Tip>
  리포지토리가 너무 커서 번들링할 수 없는 경우 Claude Code는 대신 PR 모드를 사용하도록 프롬프트를 표시합니다. 브랜치를 푸시하고 드래프트 PR을 연 다음 `/code-review ultra <PR-number>`를 실행하세요.

  풀 리퀘스트의 diff가 너무 크면 Claude Code는 리뷰 작업이 실행되기 전에 범위 지정 힌트와 함께 리뷰를 거부합니다.
</Tip>

시작하기 전에 Claude Code는 리뷰 범위(브랜치를 리뷰할 때 파일 및 줄 수 포함), 남은 무료 실행 횟수, 예상 비용이 포함된 확인 대화 상자를 표시합니다. 확인하면 리뷰가 백그라운드에서 계속되며 세션을 계속 사용할 수 있습니다. 이 명령은 `/code-review ultra`로 호출할 때만 실행되며 Claude가 스스로 ultrareview를 시작하지는 않습니다.

## 요금제 및 무료 실행

Ultrareview는 플랜에 포함된 사용량이 아닌 사용 크레딧(usage credits)에 따라 청구되는 프리미엄 기능입니다.

| 플랜 | 포함된 무료 실행 | 무료 실행 이후 |
| ------------------- | ------------------ | ------------------------------------------------------------------------------------------------------------ |
| Pro | 3회 무료 실행 | [usage credits](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)로 청구 |
| Max | 3회 무료 실행 | [usage credits](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)로 청구 |
| Team 및 Enterprise | 없음 | [usage credits](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)로 청구 |

* **무료 실행**: Pro 및 Max의 3회 실행은 계정당 1회성 할당량이며 새로 고쳐지지 않습니다.
* **리뷰당 비용**: 무료 실행을 사용한 후 또는 무료 실행 기간이 종료된 후 각 실행 전에 시작 대화 상자에 표시되는 예상 비용과 일치하게 변경 크기에 따라 일반적으로 사용 크레딧에서 \$5 ~ \$25가 청구됩니다.
* **실행이 카운트되는 시점**: 클라우드 세션이 시작되면 카운트됩니다. 일찍 중지하거나 완료하지 못한 리뷰도 무료 실행을 사용하며, 유료 리뷰는 실행된 부분에 대해서만 청구됩니다.

Ultrareview는 무료 실행 외에는 항상 사용 크레딧으로 청구되므로 유료 리뷰를 시작하려면 먼저 계정이나 조직에서 사용 크레딧을 켜야 합니다. 사용 크레딧이 켜져 있지 않으면 Claude Code가 시작을 차단하며 켜는 방법은 청구 접근 권한에 따라 다릅니다:

* 계정의 청구를 관리할 수 있는 경우 Claude Code는 사용 크레딧을 켤 수 있는 청구 설정 링크를 제공합니다.
* Team 및 Enterprise 플랜에서 청구 접근 권한이 없는 팀원은 관리자에게 사용 크레딧을 켜달라고 요청하는 메시지를 CLI에서 보냅니다.

`/usage-credits`를 실행하여 사용 크레딧 설정을 확인하거나 변경할 수도 있습니다.

Claude Code는 대화당 한 번 사용 크레딧 청구를 확인하도록 요청합니다. `/clear` 등을 통해 새 대화를 시작하면 Claude Code는 다음 유료 리뷰에 대해 청구 확인을 다시 표시합니다. v2.1.212 이전에는 이전 대화의 확인이 이월되어 `/clear` 이후의 유료 리뷰가 청구 확인 표시 없이 시작되었습니다.

## 실행 중인 리뷰 추적

리뷰는 일반적으로 5~10분이 소요됩니다. 리뷰는 백그라운드 작업으로 실행되므로 세션에서 계속 작업하거나, 다른 명령을 시작하거나, 터미널을 완전히 닫을 수 있습니다.

실행 중이거나 완료된 리뷰를 보고, 리뷰에 대한 세부 정보 뷰를 열거나, 진행 중인 리뷰를 중지하려면 `/tasks`를 사용하세요. 리뷰를 중지하면 클라우드 세션이 아카이브되고 부분 발견 사항은 반환되지 않습니다. 리뷰가 완료되면 검증된 발견 사항이 세션에 알림으로 표시됩니다. 각 발견 사항에는 파일 위치와 문제 설명이 포함되어 있으므로 Claude에게 직접 수정을 요청할 수 있습니다.

## 비대화형으로 ultrareview 실행하기

대화형 세션 없이 CI 또는 스크립트에서 ultrareview를 시작하려면 `claude ultrareview` 하위 명령을 사용하세요. 하위 명령은 `/code-review ultra`와 동일한 리뷰를 시작하고, 원격 리뷰가 완료될 때까지 차단(block)되며, 발견 사항을 stdout에 출력하고, 성공 시 코드 0으로, 실패 시 코드 1로 종료합니다.

```bash theme={null}
claude ultrareview
claude ultrareview 1234
claude ultrareview origin/main
```

인수 없이 하위 명령을 실행하면 병합 베이스가 없을 때 `/code-review ultra`와 동일한 [전체 리포지토리 폴백](#run-ultrareview-from-the-cli)을 사용하여 현재 브랜치와 기본 브랜치 간의 diff를 리뷰합니다. 풀 리퀘스트를 리뷰하려면 PR 번호를 전달하고, 해당 브랜치와의 diff를 리뷰하려면 베이스 브랜치를 전달하세요. 하위 명령을 호출하면 전체 리포지토리 폴백과 대화형 명령이 표시하는 청구 및 약관 프롬프트에 동의하는 것으로 간주되므로 입력을 기다리지 않고 실행이 시작됩니다.

전달한 베이스 브랜치가 `origin`에는 존재하지만 로컬 클론에는 존재하지 않는 경우 Claude Code는 해당 브랜치를 가져와 계속 진행합니다. 이름이 어떤 브랜치와도 일치하지 않는 경우 오류 메시지가 가장 가까운 브랜치 이름을 제안합니다. v2.1.212 이전에는 두 경우 모두 브랜치가 아니라는 오류와 함께 실패했습니다.

진행 메시지 및 라이브 세션 URL은 stderr로 이동하므로 stdout은 구문 분석 가능하게 유지됩니다. 다음 플래그를 사용하여 출력 및 시간 초과를 제어하세요:

| 플래그 | 설명 |
| --------------------- | ------------------------------------------------------------------- |
| `--json` | 형식이 지정된 발견 사항 대신 날것의 `bugs.json` 페이로드 출력 |
| `--timeout <minutes>` | 리뷰 완료를 기다릴 최대 시간(분). 기본값은 30분 |

`claude ultrareview`를 실행하려면 `/code-review ultra`와 동일한 인증 및 사용 크레딧 구성이 필요합니다. 발견 사항의 유무에 관계없이 리뷰가 완료되면 코드 0으로 종료하고, 리뷰를 시작하지 못하거나 클라우드 세션 오류가 발생하거나 시간 초과가 지나는 경우 코드 1로 종료하며, Ctrl-C로 중단되면 코드 130으로 종료합니다. 하위 명령을 중단하더라도 원격 리뷰는 계속 실행되므로 브라우저에서 관찰하려면 stderr에 출력된 세션 URL을 따르세요.

GitHub 풀 리퀘스트에 대한 자동 리뷰의 경우 [Code Review](/docs/en/code-review)가 리포지토리와 직접 통합되어 CLI 단계 없이 인라인 PR 주석으로 발견 사항을 게시합니다.

## ultrareview와 /code-review 및 /review의 비교

세 명령어 모두 코드를 리뷰하지만 워크플로의 다양한 단계를 타겟팅합니다.

| | `/code-review` | `/review <pr>` | `/code-review ultra` |
| -------- | ------------------------------- | -------------------------------------------- | --------------------------------------------------------------- |
| 대상 | 작업 중인 diff | GitHub 풀 리퀘스트 | 작업 중인 diff 또는 풀 리퀘스트 |
| 실행 위치 | 세션의 로컬 환경 | 세션의 로컬 환경 | 클라우드 샌드박스의 원격 환경 |
| 심도 | 노력 인수(effort argument)에 따라 확장됨 | 세션의 노력 수준에서 단일 패스 리뷰 | 독립적인 검증을 포함한 멀티 에이전트 플릿 |
| 소요 시간 | 몇 초 ~ 몇 분 | 몇 초 ~ 몇 분 | 약 5 ~ 10분 |
| 비용 | 일반 사용량에 카운트됨 | 일반 사용량에 카운트됨 | 무료 실행 후 사용 크레딧으로 리뷰당 약 \$5 ~ \$25 |
| 최적 환경 | 작업 중 빠른 피드백 | 승인 전 동료의 PR 검토 | 상당한 변경 사항에 대해 병합 전 신뢰 확보 |

작업하면서 빠른 피드백을 얻으려면 `/code-review`를 사용하세요. 승인 전 풀 리퀘스트를 훑어보듯 검토하려면 `/review <pr>`을 사용하세요. 로컬 리뷰에서 놓칠 수 있는 문제를 포착하는 더 깊은 패스를 원할 때 상당한 변경 사항을 병합하기 전에 `/code-review ultra`를 사용하세요.

## 관련 리소스

* [Claude Code on the web](/docs/en/claude-code-on-the-web): 클라우드 세션 및 클라우드 샌드박스가 작동하는 방식에 대해 알아보기
* [Plan complex changes with ultraplan](/docs/en/ultraplan): 사전 설계 작업을 위한 ultrareview의 계획 수립 대응물
* [Manage costs effectively](/docs/en/costs): 사용량 추적 및 지출 한도 설정
