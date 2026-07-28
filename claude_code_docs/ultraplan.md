> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Ultraplan으로 클라우드에서 계획 세우기

> CLI에서 계획 수립을 시작하고, 웹의 Claude Code에서 드래프트한 다음, 원격으로 또는 터미널로 돌아와 실행하세요.

<Note>
  Ultraplan은 리서치 프리뷰(research preview) 상태입니다. 피드백에 따라 동작 및 기능이 변경될 수 있습니다.
</Note>

Ultraplan은 로컬 CLI의 계획 수립 작업을 [plan mode](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)로 실행되는 [Claude Code on the web](/docs/en/claude-code-on-the-web) 세션에 전달합니다. 사용자가 터미널에서 계속 작업하는 동안 Claude가 클라우드에서 계획을 드래프트합니다. 계획이 준비되면 브라우저에서 열어 특정 섹션에 의견을 남기고, 수정을 요청하며, 실행할 위치를 선택합니다.

이는 터미널이 제공하는 것보다 더 풍부한 리뷰 인터페이스를 원할 때 유용합니다:

* **타겟팅된 피드백**: 전체에 답변하는 대신 계획의 개별 섹션에 주석을 달기
* **직접 개입이 필요 없는 드래프트**: 계획이 원격으로 생성되므로 터미널은 다른 작업을 위해 자유롭게 유지됨
* **유연한 실행**: 웹에서 실행되도록 계획을 승인하고 풀 리퀘스트를 생성하거나, 터미널로 다시 보내기

Ultraplan에는 [Claude Code on the web](/docs/en/claude-code-on-the-web) 계정과 GitHub 리포지토리가 필요합니다. Anthropic의 클라우드 인프라에서 실행되기 때문에 Amazon Bedrock, Google Cloud's Agent Platform 또는 Microsoft Foundry를 사용할 때는 지원되지 않습니다. 클라우드 세션은 계정의 기본 [cloud environment](/docs/en/claude-code-on-the-web#the-cloud-environment)에서 실행됩니다. 클라우드 환경이 아직 없는 경우 ultraplan이 처음 시작될 때 자동으로 환경을 생성합니다.

## CLI에서 ultraplan 시작하기

로컬 CLI 세션에서 다음 세 가지 방법으로 ultraplan을 시작할 수 있습니다:

* **명령어**: `/ultraplan` 뒤에 프롬프트를 입력하여 실행
* **키워드**: 일반 프롬프트의 어느 위치에나 `ultraplan` 단어를 포함
* **로컬 플랜에서**: Claude가 로컬 플랜을 완료하고 승인 대화 상자를 표시할 때 **No, refine with Ultraplan on Claude Code on the web**을 선택하여 추가 반복을 위해 드래프트를 클라우드로 전송

예를 들어 명령어로 서비스 마이그레이션을 계획하려면:

```
/ultraplan migrate the auth service from sessions to JWTs
```

명령어 및 키워드 경로는 시작하기前に 확인 대화 상자를 엽니다. 로컬 플랜 경로는 해당 선택이 이미 확인 역할을 하므로 이 대화 상자를 건너끕니다. [Remote Control](/docs/en/remote-control)이 활성화되어 있는 경우 두 기능 모두 claude.ai/code 인터페이스를 점유하고 한 번에 하나만 연결될 수 있으므로 ultraplan이 시작될 때 연결이 해제됩니다.

클라우드 세션이 시작된 후 클라우드 세션이 작업하는 동안 CLI의 프롬프트 입력에 상태 표시기가 표시됩니다:

| 상태 | 의미 |
| :----------------------------- | :----------------------------------------------------------------- |
| `◇ ultraplan` | Claude가 코드베이스를 조사하고 계획을 드래프트하는 중입니다 |
| `◇ ultraplan needs your input` | Claude에게 명확히 해야 할 질문이 있습니다. 세션 링크를 열어 응답하세요 |
| `◆ ultraplan ready` | 브라우저에서 계획을 리뷰할 준비가 되었습니다 |

`/tasks`를 실행하고 ultraplan 항목을 선택하면 세션 링크, 에이전트 활동, **Stop ultraplan** 작업이 포함된 세부 정보 뷰가 열립니다. 중지하면 클라우드 세션이 아카이브되고 표시기가 지워지며 터미널에는 아무것도 저장되지 않습니다.

## 브라우저에서 계획 리뷰 및 수정

상태가 `◆ ultraplan ready`로 변경되면 세션 링크를 열어 claude.ai에서 계획을 확인하세요. 계획은 전용 리뷰 뷰에 표시됩니다:

* **인라인 주석**: 구절을 하이라이트하고 Claude가 처리할 주석 남기기
* **이모지 반응**: 전체 주석을 작성하지 않고 섹션에 반응하여 승인 또는 우려 표시하기
* **개요 사이드바**: 계획의 섹션 간 이동하기

Claude에게 주석을 처리하도록 요청하면 계획을 수정하고 업데이트된 드래프트 상태를 제시합니다. 실행 위치를 선택하기 전에 필요한 만큼 반복할 수 있습니다.

## 실행 위치 선택

계획이 제대로 작성되었으면 Claude가 동일한 클라우드 세션에서 구현할지, 아니면 대기 중인 터미널로 다시 보낼지를 브라우저에서 선택합니다.

### 웹에서 실행

동일한 Claude Code on the web 세션에서 Claude가 이를 구현하도록 하려면 브라우저에서 **Approve Claude's plan and start coding**을 선택하세요. 터미널에 확인이 표시되고 상태 표시기가 지워지며 클라우드에서 작업이 계속됩니다. 구현이 완료되면 [diff를 리뷰](/docs/en/claude-code-on-the-web#review-changes)하고 웹 인터페이스에서 풀 리퀘스트를 생성합니다.

### 계획을 터미널로 다시 전송

환경에 대한 전체 접근 권한을 가지고 로컬에서 계획을 구현하려면 브라우저에서 **Approve plan and teleport back to terminal**을 선택하세요. 이 옵션은 CLI에서 세션이 시작되었고 터미널이 여전히 폴링 중일 때 표시됩니다. 웹 세션은 병렬로 계속 작업하지 않도록 아카이브됩니다.

터미널에는 세 가지 옵션이 포함된 **Ultraplan approved**라는 제목의 대화 상자에 계획이 표시됩니다:

* **Implement here**: 현재 대화에 계획을 주입하고 중단했던 부분부터 계속 진행
* **Start new session**: 현재 대화를 지우고 계획만 컨텍스트로 가지고 새로 시작
* **Cancel**: 실행하지 않고 파일에 계획 저장. 나중에 다시 돌아올 수 있도록 Claude가 파일 경로를 출력함

새 세션을 시작하면 Claude가 상단에 `claude --resume` 명령을 출력하여 나중에 이전 대화로 돌아갈 수 있도록 합니다.

## 관련 리소스

* [Claude Code on the web](/docs/en/claude-code-on-the-web): ultraplan이 실행되는 클라우드 인프라
* [Plan mode](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode): 로컬 세션에서 계획 수립이 작동하는 방식
* [Find bugs with ultrareview](/docs/en/ultrareview): 병합 전 문제를 잡기 위한 ultraplan의 코드 리뷰 대응물
* [Remote Control](/docs/en/remote-control): 자체 머신에서 실행되는 세션으로 claude.ai/code 인터페이스 사용
