> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 일정에 따라 프롬프트 실행하기

> `/loop` 및 cron 예약 도구를 사용하여 Claude Code 세션 내에서 프롬프트를 반복 실행하고, 상태를 폴링하거나, 1회성 알림을 설정하세요.

예약된 작업(Scheduled tasks)을 사용하면 Claude가 일정 간격으로 프롬프트를 자동으로 재실행할 수 있습니다. 배포 폴링, PR 감시, 오래 걸리는 빌드 재확인, 또는 세션 후반에 수행할 작업 알림 설정 등에 활용하세요. 폴링 대신 이벤트 발생 시 즉시 반응하려면 [Channels](/docs/en/channels)를 참조하세요(CI가 실패를 세션에 직접 푸시할 수 있음). 일정 간격 대신 조건이 충족될 때까지 세션이 차례대로 계속 작업하게 하려면 [`/goal`](/docs/en/goal)을 참조하세요.

작업의 범위는 세션 단위입니다. 즉, 현재 대화 내에 존재하며 새 대화를 시작하면 중지됩니다. `--resume` 또는 `--continue`로 재개하면 [7일 만료](#seven-day-expiry)되지 않은 모든 작업(지난 7일 이내에 생성된 반복 작업 또는 아직 예약 시간이 지나지 않은 1회성 작업)이 복원됩니다. 세션과 독립적으로 유지되는 예약을 하려면 Anthropic 관리 인프라에서 루틴을 생성하는 [Routines](/docs/en/routines)를 사용하거나, [Desktop scheduled task](/docs/en/desktop-scheduled-tasks)를 설정하거나, [GitHub Actions](/docs/en/github-actions)를 사용하세요.

## 예약 옵션 비교

Claude Code는 반복 또는 1회성 작업을 예약하는 세 가지 방법을 제공합니다:

| | [Cloud](/docs/en/routines) | [Desktop](/docs/en/desktop-scheduled-tasks) | [`/loop`](/docs/en/scheduled-tasks) |
| :------------------------- | :----------------------------- | :------------------------------------- | :---------------------------------- |
| 실행 위치 | Anthropic Cloud | 사용자 머신 | 사용자 머신 |
| 머신 켜짐 필요 여부 | 아니오 | 예 | 예 |
| 열린 세션 필요 여부 | 아니오 | 아니오 | 예 |
| 재시작 후 유지 여부 | 예 | 예 | 만료되지 않은 경우 `--resume` 시 복원 |
| 로컬 파일 접근 | 아니오 (새로 클론) | 예 | 예 |
| MCP 서버 | 작업별 구성된 커넥터 | [구성 파일](/docs/en/mcp) 및 커넥터 | 세션에서 상속 |
| 권한 프롬프트 | 아니오 (자율 실행) | 작업별 구성 가능 | 세션에서 상속 |
| 커스텀 일정 | CLI의 `/schedule` 통해 | 예 | 예 |
| 최소 간격 | 1시간 | 1분 | 1분 |

<Tip>
  사용자 머신 없이 안정적으로 실행되어야 하는 작업에는 **클라우드 작업**을 사용하세요. 로컬 파일 및 도구에 접근해야 할 때는 **Desktop 작업**을 사용하세요. 세션 중 빠른 폴링에는 **`/loop`**를 사용하세요.
</Tip>

## /loop로 프롬프트 반복 실행하기

`/loop` [번들 skill](/docs/en/commands)은 세션이 열려 있는 동안 프롬프트를 반복 실행하는 가장 빠른 방법입니다. 간격과 프롬프트는 모두 옵션이며 제공하는 내용에 따라 루프의 동작 방식이 결정됩니다.

| 제공하는 내용 | 예시 | 작동 방식 |
| :------------------------ | :-------------------------- | :------------------------------------------------------------------------------------------------------------ |
| 간격 및 프롬프트 | `/loop 5m check the deploy` | 프롬프트가 [고정된 일정](#run-on-a-fixed-interval)에 따라 실행됨 |
| 프롬프트만 제공 | `/loop check the deploy` | 각 반복마다 [Claude가 선택한 간격](#let-claude-choose-the-interval)으로 프롬프트가 실행됨 |
| 간격만 제공 또는 아무것도 제공하지 않음 | `/loop` | [내장 유지 관리 프롬프트](#run-the-built-in-maintenance-prompt)가 실행되거나 `loop.md`가 존재하는 경우 해당 파일이 실행됨 |

각 반복 시 skill을 재실행하기 위해 프롬프트로 skill을 전달할 수도 있습니다(예: `/loop 20m /review-pr 1234`). {/* min-version: 2.1.196 */}v2.1.196부터 예약된 실행은 Claude가 [자체적으로 호출하도록 허용된](/docs/en/skills#control-who-invokes-a-skill) skill만 실행합니다. 다음 항목들은 실행되는 대신 일반 텍스트로 Claude에게 전달됩니다:

* `/permissions`, `/model`, 또는 `/clear`와 같은 내장 명령
* 번들로 제공되는 `/verify` 및 `/code-review` skill을 포함하여 [`disable-model-invocation: true`](/docs/en/skills#frontmatter-reference)로 표시된 skill
* [`skillOverrides`](/docs/en/skills#override-skill-visibility-from-settings) 설정 또는 `Skill` [deny 규칙](/docs/en/skills#restrict-claude’s-skill-access)에 의해 Claude로부터 제한된 skill
* `/mcp__github__list_prs`와 같은 [MCP 프롬프트](/docs/en/mcp#use-mcp-prompts-as-commands)

### 고정 간격으로 실행

간격을 제공하면 Claude는 이를 cron 표현식으로 변환하고, 작업을 예약하며, 주기와 작업 ID를 확인합니다.

```text theme={null}
/loop 5m check if the deployment finished and tell me what happened
```

간격은 프롬프트 앞에 `30m`과 같은 단일 토큰으로 위치하거나, 뒤에 `every 2 hours`와 같은 절로 위치할 수 있습니다. 지원되는 단위는 초 `s`, 분 `m`, 시간 `h`, 일 `d`입니다.

cron은 1분 단위 해상도를 가지므로 초 단위는 가장 가까운 분으로 올림됩니다. `7m` 또는 `90m`과 같이 깔끔한 cron 단계로 매핑되지 않는 간격은 매핑되는 가장 가까운 간격으로 반올림되며 Claude가 선택한 결과를 알려줍니다.

### Claude가 간격을 선택하도록 설정

간격을 생략하면 Claude는 고정된 cron 일정 대신 동적으로 간격을 선택합니다. 각 반복이 끝난 후 관찰한 내용(빌드 완료 중이거나 PR이 활성화된 경우 짧은 대기, 보류 중인 작업이 없는 경우 긴 대기)을 바탕으로 1분에서 1시간 사이의 지연 시간을 선택합니다. 선택된 지연 시간과 그 이유는 각 반복의 끝에 출력됩니다.

아래 예시는 CI 및 리뷰 주석을 확인하며, PR 활동이 줄어들면 Claude가 반복 사이의 대기 시간을 늘립니다:

```text theme={null}
/loop check whether CI passed and address any review comments
```

동적 `/loop` 일정을 요청하면 Claude가 [Monitor 도구](/docs/en/tools-reference#monitor-tool)를 직접 사용할 수 있습니다. Monitor는 백그라운드 스크립트를 실행하고 각 출력 라인을 다시 스트리밍하므로 폴링을 완전히 피하고 간격에 따라 프롬프트를 재실행하는 것보다 토큰 효율적이고 반응성이 뛰어납니다.

동적으로 예약된 루프는 다른 작업과 마찬가지로 [예약된 작업 목록](#manage-scheduled-tasks)에 표시되므로 동일한 방식으로 목록을 확인하거나 취소할 수 있습니다. [지터(jitter) 규칙](#jitter)은 적용되지 않지만 [7일 만료](#seven-day-expiry) 규칙은 적용되므로 시작한 지 7일 후에 루프가 자동으로 종료됩니다.

<Note>
  Amazon Bedrock, Claude Platform on AWS, Google Cloud's Agent Platform 및 Microsoft Foundry에서는 프롬프트에 간격이 없는 경우 고정된 10분 일정으로 실행됩니다.
</Note>

### 내장 유지 관리 프롬프트 실행

프롬프트를 생략하면 Claude는 사용자가 제공한 프롬프트 대신 내장 유지 관리 프롬프트를 사용합니다. 각 반복에서 순서대로 다음 작업을 진행합니다:

* 대화의 미완료 작업 계속 수행
* 현재 브랜치의 풀 리퀘스트 처리: 리뷰 주석, 실패한 CI 실행, 병합 충돌
* 보류 중인 작업이 없을 때 버그 탐색이나 단순화와 같은 정리 작업 실행

Claude는 해당 범위를 벗어나는 새로운 이니셔티브를 시작하지 않으며, 푸시 또는 삭제와 같은 되돌릴 수 없는 작업은 트랜스크립트가 이미 승인한 작업을 계속할 때만 진행됩니다.

```text theme={null}
/loop
```

단독 `/loop`는 이 프롬프트를 [동적으로 선택된 간격](#let-claude-choose-the-interval)으로 실행합니다. 고정 일정으로 실행하려면 `/loop 15m`과 같이 간격을 추가하세요. 내장 프롬프트를 자체 기본값으로 교체하려면 [loop.md로 기본 프롬프트 커스텀하기](#customize-the-default-prompt-with-loop-md)를 참조하세요.

<Note>
  Amazon Bedrock, Claude Platform on AWS, Google Cloud's Agent Platform 및 Microsoft Foundry에서는 프롬프트가 없는 `/loop`가 유지 관리 프롬프트를 실행하는 대신 사용법 메시지를 출력합니다.
</Note>

### loop.md로 기본 프롬프트 커스텀하기

`loop.md` 파일은 내장 유지 관리 프롬프트를 사용자 고유의 지침으로 교체합니다. 이 파일은 분리된 예약 작업 목록이 아니라 단독 `/loop`를 위한 단일 기본 프롬프트를 정의하며, 명령줄에서 프롬프트를 제공할 때마다 무시됩니다. 그와 함께 추가 프롬프트를 예약하려면 `/loop <prompt>`를 사용하거나 [Claude에게 직접 요청](#manage-scheduled-tasks)하세요.

Claude는 두 위치에서 파일을 찾고 먼저 발견된 파일을 사용합니다.

| 경로 | 범위 |
| :------------------ | :--------------------------------------------------------------- |
| `.claude/loop.md` | 프로젝트 수준. 두 파일이 모두 존재할 때 우선 적용됩니다. |
| `~/.claude/loop.md` | 사용자 수준. 자체 파일이 정의되지 않은 모든 프로젝트에 적용됩니다. |

이 파일은 필수 구조가 없는 일반 Markdown입니다. `/loop` 프롬프트를 직접 입력하는 것처럼 작성하세요. 다음 예시는 릴리스 브랜치를 정상 상태로 유지합니다:

```markdown title=".claude/loop.md" theme={null}
Check the `release/next` PR. If CI is red, pull the failing job log,
diagnose, and push a minimal fix. If new review comments have arrived,
address each one and resolve the thread. If everything is green and
quiet, say so in one line.
```

`loop.md` 편집 사항은 다음 반복 시 적용되므로 루프가 실행되는 동안 지침을 수정할 수 있습니다. 두 위치에 모두 `loop.md`가 없으면 루프는 내장 유지 관리 프롬프트로 전환됩니다. 파일 내용을 간결하게 유지하세요. 25,000바이트를 초과하는 내용은 잘립니다.

<Note>
  Amazon Bedrock, Claude Platform on AWS, Google Cloud's Agent Platform 및 Microsoft Foundry에서는 `loop.md`를 읽지 않으며 프롬프트가 없는 `/loop`는 사용법 메시지를 출력합니다.
</Note>

### 루프 중단하기

다음 반복을 기다리는 동안 `/loop`를 중단하려면 `Esc`키를 누릅니다. 그러면 보류 중인 대기가 지워져 루프가 다시 실행되지 않습니다. [Claude에게 직접 요청](#manage-scheduled-tasks)하여 예약한 작업은 `Esc`의 영향을 받지 않으며 삭제할 때까지 유지됩니다.

[자율 주행 모드(self-paced mode)](#let-claude-choose-the-interval)에서는 작업이 완료되면 Claude가 스스로 루프를 종료할 수도 있습니다. Claude가 `stop: true`로 [`ScheduleWakeup` 도구](/docs/en/tools-reference)를 호출하면 보류 중인 대기가 즉시 취소됩니다. 재예약이나 중단 없이 반복이 종료되면 Claude Code는 약 20분 후에 하나의 대체 대기를 예약하고 해당 반복에서도 재예약되지 않으면 루프를 종료합니다. v2.1.202 이전에는 재예약하지 않는 것이 Claude가 스스로 루프를 종료할 수 있는 유일한 방법이었습니다.

고정 간격의 루프는 사용자가 중단하거나 [7일이 지날 때까지](#seven-day-expiry) 계속 실행됩니다.

## 1회성 알림 설정하기

1회성 알림의 경우 `/loop`를 사용하는 대신 자연어로 원하는 내용을 설명하세요. Claude는 실행 후 자동으로 삭제되는 단일 실행 작업을 예약합니다.

```text theme={null}
remind me at 3pm to push the release branch
```

```text theme={null}
in 45 minutes, check whether the integration tests passed
```

Claude는 cron 표현식을 사용하여 실행 시간을 특정 분과 시에 고정하고 실행 시점을 확인합니다.

## 예약된 작업 관리하기

자연어로 Claude에게 작업을 나열하거나 취소하도록 요청하거나, 밑바탕의 도구를 직접 참조하세요.

```text theme={null}
what scheduled tasks do I have?
```

```text theme={null}
cancel the deploy check job
```

내부적으로 Claude는 다음 도구를 사용합니다:

| 도구 | 목적 |
| :----------- | :-------------------------------------------------------------------------------------------------------------- |
| `CronCreate` | 새 작업 예약. 5개 필드의 cron 표현식, 실행할 프롬프트, 반복 여부 또는 1회성 실행 여부를 받습니다. |
| `CronList` | 모든 예약된 작업과 해당 ID, 일정, 프롬프트를 나열합니다. |
| `CronDelete` | ID로 작업을 취소합니다. |

각 예약 작업에는 `CronDelete`에 전달할 수 있는 8자 ID가 있습니다. 세션은 한 번에 최대 50개의 예약 작업을 가질 수 있습니다.

## 예약 작업 실행 방식

스케줄러는 매초 마감된 작업을 확인하고 낮은 우선순위로 대기열에 추가합니다. 예약된 프롬프트는 Claude가 응답하는 중간이 아니라 사용자의 턴 사이에 실행됩니다. 작업 마감 시점에 Claude가 바쁜 상태이면 프롬프트는 현재 턴이 끝날 때까지 기다립니다.

모든 시간은 로컬 시간대로 해석됩니다. `0 9 * * *`와 같은 cron 표현식은 UTC가 아니라 Claude Code를 실행 중인 위치의 오전 9시를 의미합니다.

### 지터 (Jitter)

모든 세션이 동일한 시계 시점에 API를 요청하는 것을 방지하기 위해 스케줄러는 실행 시간에 결정론적 오프셋을 추가합니다:

* 반복 작업은 예약된 시간보다 최대 30분 늦게 실행됩니다(매시간보다 더 자주 실행되는 작업의 경우 간격의 절반까지). `:00`에 예약된 매시간 작업은 최대 `:30` 사이의 임의 시점에 실행될 수 있습니다.
* 정시 또는 30분에 예약된 1회성 작업은 최대 90초 일찍 실행됩니다.

오프셋은 작업 ID에서 파생되므로 동일한 작업은 항상 동일한 오프셋을 받습니다. 정확한 타이밍이 중요한 경우 `:00`이나 `:30`이 아닌 분(예: `0 9 * * *` 대신 `3 9 * * *`)을 선택하면 1회성 지터가 적용되지 않습니다.

### 7일 만료

반복 작업은 생성 후 7일이 지나면 자동으로 만료됩니다. 작업이 마지막으로 한 번 실행된 후 스스로 삭제됩니다. 이는 잊혀진 루프가 실행될 수 있는 기간을 제한합니다. 반복 작업을 더 오래 유지해야 하는 경우 만료되기 전에 취소하고 다시 생성하거나, 지속적인 예약을 위해 [Routines](/docs/en/routines) 또는 [Desktop scheduled tasks](/docs/en/desktop-scheduled-tasks)를 사용하세요.

## Cron 표현식 참조

`CronCreate`는 표준 5-필드 cron 표현식(`minute hour day-of-month month day-of-week`)을 받아들입니다. 모든 필드는 와일드카드(`*`), 단일 값(`5`), 단계(`*/15`), 범위(`1-5`) 및 쉼표로 구분된 목록(`1,15,30`)을 지원합니다.

| 예시 | 의미 |
| :------------- | :--------------------------- |
| `*/5 * * * *` | 5분마다 |
| `0 * * * *` | 매시간 정각 |
| `7 * * * *` | 매시간 7분 |
| `0 9 * * *` | 매일 로컬 오전 9시 |
| `0 9 * * 1-5` | 평일 로컬 오전 9시 |
| `30 14 15 3 *` | 3월 15일 로컬 오후 2:30 |

요일(Day-of-week)은 일요일 `0` 또는 `7`부터 토요일 `6`까지 사용합니다. `L`, `W`, `?`와 같은 확장 구문 및 `MON`, `JAN`과 같은 이름 별칭은 지원되지 않습니다.

일자(day-of-month)와 요일(day-of-week)이 모두 제한된 경우 두 필드 중 하나라도 일치하면 날짜가 일치합니다. 이는 표준 vixie-cron 의미론을 따릅니다.

## 예약 작업 비활성화

스케줄러를 완전히 비활성화하려면 환경 변수에 `CLAUDE_CODE_DISABLE_CRON=1`을 설정하세요. cron 도구 및 `/loop`를 사용할 수 없게 되며 이미 예약된 작업의 실행이 중지됩니다. 전체 비활성화 플래그 목록은 [Environment variables](/docs/en/env-vars)를 참조하세요.

## 제한 사항

세션 범위 예약에는 고유한 제약 사항이 있습니다:

* 작업은 Claude Code가 실행 중이고 대기 상태일 때만 실행됩니다. 터미널을 닫거나 세션을 종료하면 실행이 중지됩니다. [세션을 백그라운드로 전환](/docs/en/agent-view#from-inside-a-session)하면 `/loop` 작업이 백그라운드 세션으로 전달되어 터미널 없이 계속 실행됩니다.
* 놓친 실행에 대한 소급 실행 없음. 실행 시간이 오래 걸리는 요청으로 Claude가 바쁜 동안 작업의 예약 시간이 지난 경우, missed interval마다 실행되지 않고 Claude가 대기 상태가 될 때 한 번 실행됩니다.
* 새 대화를 시작하면 모든 세션 범위 작업이 지워집니다. `claude --resume` 또는 `claude --continue`로 재개하면 만료되지 않은 작업(생성 후 7일 이내의 반복 작업 및 예약 시간이 아직 지나지 않은 1회성 작업)이 복원됩니다. 백그라운드 Bash 및 monitor 작업은 재개 시 복원되지 않습니다.
* {/* min-version: 2.1.216 */}Claude Code는 프로젝트의 `.claude` 디렉터리에 예약 작업 목록을 저장하며, 해당 디렉터리나 내부의 작업 파일이 심볼릭 링크인 경우 작업 예약이 오류와 함께 실패합니다. v2.1.216 이전에는 Claude Code가 링크를 통해 파일을 썼습니다.

무인으로 실행해야 하는 cron 기반 자동화의 경우:

* [Routines](/docs/en/routines): Anthropic 관리 인프라에서 일정, API 호출 또는 GitHub 이벤트에 따라 실행
* [GitHub Actions](/docs/en/github-actions): CI에서 `schedule` 트리거 사용
* [Desktop scheduled tasks](/docs/en/desktop-scheduled-tasks): 사용자 머신에서 로컬로 실행
