> ## 문서 색인
> 전체 문서 색인은 https://code.claude.com/docs/llms.txt 에서 가져올 수 있습니다.
> 더 자세히 탐색하기 전에 이 파일을 사용해 사용 가능한 모든 페이지를 확인하세요.

# 체크포인팅으로 파일 변경 사항 되돌리기(Rewind)

> 에이전트 세션 동안의 파일 변경 사항을 추적하고 파일 이전 상태로 복원합니다.

파일 체크포인팅(File checkpointing)은 에이전트 세션 동안 Write, Edit, NotebookEdit 도구를 통해 수정된 파일 변경 사항을 추적하여 파일 이전 상태로 되돌릴 수 있게 해줍니다. 직접 시도해보고 싶으신가요? [대화형 예제](#대화형-예제-시도해보기)로 이동하세요.

체크포인팅을 사용하면 다음을 수행할 수 있습니다:

* 알고 있는 양호한 상태로 파일을 복원하여 **원치 않는 변경 사항 취소**
* 체크포인트로 복원한 후 다른 접근 방식을 시도하여 **대안 탐색**
* 에이전트가 잘못 수정한 경우 **오류로부터 복구**

<Warning>
  Write, Edit, NotebookEdit 도구를 통한 변경 사항만 추적됩니다. Bash 명령(`echo > file.txt` 또는 `sed -i` 등)을 통한 변경 사항은 체크포인트 시스템에 포착되지 않습니다.
</Warning>

## 체크포인팅 작동 방식

파일 체크포인팅을 활성화하면 SDK는 Write, Edit 또는 NotebookEdit 도구를 통해 파일을 수정하기 전에 백업을 생성합니다. 응답 스트림의 사용자 메시지에는 복원 지점으로 사용할 수 있는 체크포인트 UUID가 포함됩니다.

체크포인트는 에이전트가 파일을 수정하는 데 사용하는 다음 내장 도구와 함께 작동합니다:

| 도구 | 설명 |
| ------------ | ------------------------------------------------------------------ |
| Write | 새 파일을 생성하거나 기존 파일을 새 콘텐츠로 덮어씁니다 |
| Edit | 기존 파일의 특정 부분에 목표로 하는 편집을 수행합니다 |
| NotebookEdit | Jupyter 노트북(`.ipynb` 파일)의 셀을 수정합니다 |

<Note>
  파일 되돌리기(rewind)는 디스크의 파일을 이전 상태로 복원합니다. 대화 자체를 되돌리지는 않습니다. `rewindFiles()` (TypeScript) 또는 `rewind_files()` (Python)를 호출한 후에도 대화 기록과 컨텍스트는 그대로 유지됩니다.
</Note>

체크포인트 시스템은 다음 사항을 추적합니다:

* 세션 동안 생성된 파일
* 세션 동안 수정된 파일
* 수정된 파일의 원본 콘텐츠

체크포인트로 되돌리면 생성된 파일은 삭제되고 수정된 파일은 해당 시점의 콘텐츠로 복원됩니다.

## 체크포인팅 구현하기

파일 체크포인팅을 사용하려면 옵션에서 이를 활성화하고, 응답 스트림에서 체크포인트 UUID를 캡처한 다음, 복원이 필요할 때 `rewindFiles()` (TypeScript) 또는 `rewind_files()` (Python)를 호출하세요.

다음 예제는 전체 흐름을 보여줍니다: 체크포인팅 활성화, 응답 스트림에서 체크포인트 UUID 및 세션 ID 캡처, 나중에 세션을 다시 시작하여 파일 되돌리기. 각 단계는 아래에서 자세히 설명합니다. 이 섹션의 예제는 "Refactor the authentication module" 프롬프트를 사용합니다. 인증 모듈이 포함된 프로젝트에서 실행하거나, 파일 변경과 되돌리기 복원 과정을 관찰할 수 있도록 프롬프트를 기존 프로젝트 파일 이름으로 변경하여 실행하세요.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import (
      ClaudeSDKClient,
      ClaudeAgentOptions,
      UserMessage,
      ResultMessage,
  )


  async def main():
      # 1단계: 체크포인팅 활성화
      options = ClaudeAgentOptions(
          enable_file_checkpointing=True,
          permission_mode="acceptEdits",  # 프롬프트 없이 파일 편집 자동 승인
          extra_args={
              "replay-user-messages": None
          },  # 응답 스트림에서 체크포인트 UUID를 받기 위해 필수
      )

      checkpoint_id = None
      session_id = None

      # 쿼리를 실행하고 체크포인트 UUID 및 세션 ID 캡처
      async with ClaudeSDKClient(options) as client:
          await client.query("Refactor the authentication module")

          # 2단계: 첫 번째 사용자 메시지에서 체크포인트 UUID 캡처
          async for message in client.receive_response():
              if isinstance(message, UserMessage) and message.uuid and not checkpoint_id:
                  checkpoint_id = message.uuid
              if isinstance(message, ResultMessage) and not session_id:
                  session_id = message.session_id

      # 3단계: 나중에 빈 프롬프트로 세션을 다시 시작하여 되돌리기
      if checkpoint_id and session_id:
          async with ClaudeSDKClient(
              ClaudeAgentOptions(enable_file_checkpointing=True, resume=session_id)
          ) as client:
              await client.query("")  # 연결을 열기 위한 빈 프롬프트
              async for message in client.receive_response():
                  await client.rewind_files(checkpoint_id)
                  break
          print(f"Rewound to checkpoint: {checkpoint_id}")


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  async function main() {
    // 1단계: 체크포인팅 활성화
    const opts = {
      enableFileCheckpointing: true,
      permissionMode: "acceptEdits" as const, // 프롬프트 없이 파일 편집 자동 승인
      extraArgs: { "replay-user-messages": null } // 응답 스트림에서 체크포인트 UUID를 받기 위해 필수
    };

    const response = query({
      prompt: "Refactor the authentication module",
      options: opts
    });

    let checkpointId: string | undefined;
    let sessionId: string | undefined;

    // 2단계: 첫 번째 사용자 메시지에서 체크포인트 UUID 캡처
    try {
      for await (const message of response) {
        if (message.type === "user" && message.uuid && !checkpointId) {
          checkpointId = message.uuid;
        }
        if ("session_id" in message && !sessionId) {
          sessionId = message.session_id;
        }
      }
    } catch (error) {
      // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
      // 오류 결과였더라도 sessionId와 checkpointId는 위 루프에서 이미 캡처된 상태입니다.
      // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
      console.error(`Session ended with an error: ${error}`);
    }

    // 3단계: 나중에 빈 프롬프트로 세션을 다시 시작하여 되돌리기
    if (checkpointId && sessionId) {
      const rewindQuery = query({
        prompt: "", // 연결을 열기 위한 빈 프롬프트
        options: { ...opts, resume: sessionId }
      });

      for await (const msg of rewindQuery) {
        await rewindQuery.rewindFiles(checkpointId);
        break;
      }
      console.log(`Rewound to checkpoint: ${checkpointId}`);
    }
  }

  main();
  ```
</CodeGroup>

<Steps>
  <Step title="체크포인팅 활성화">
    체크포인팅을 활성화하고 체크포인트 UUID를 수신하도록 SDK 옵션을 구성하세요:

    | 옵션 | Python | TypeScript | 설명 |
    | ------------------------ | ------------------------------------------- | --------------------------------------------- | ------------------------------------------------ |
    | 체크포인팅 활성화 | `enable_file_checkpointing=True` | `enableFileCheckpointing: true` | 되돌리기를 위한 파일 변경 사항 추적 |
    | 체크포인트 UUID 수신 | `extra_args={"replay-user-messages": None}` | `extraArgs: { 'replay-user-messages': null }` | 스트림에서 사용자 메시지 UUID를 받기 위해 필요 |

    <CodeGroup>
      ```python Python theme={null}
      options = ClaudeAgentOptions(
          enable_file_checkpointing=True,
          permission_mode="acceptEdits",
          extra_args={"replay-user-messages": None},
      )

      async with ClaudeSDKClient(options) as client:
          await client.query("Refactor the authentication module")
      ```

      ```typescript TypeScript theme={null}
      const response = query({
        prompt: "Refactor the authentication module",
        options: {
          enableFileCheckpointing: true,
          permissionMode: "acceptEdits" as const,
          extraArgs: { "replay-user-messages": null }
        }
      });
      ```
    </CodeGroup>
  </Step>

  <Step title="체크포인트 UUID 및 세션 ID 캡처">
    `replay-user-messages` 옵션이 설정되면(위 참조) 응답 스트림의 각 사용자 메시지에는 체크포인트 역할을 하는 UUID가 있습니다.

    대부분의 유즈케이스에서는 첫 번째 사용자 메시지 UUID(`message.uuid`)를 캡처하면 됩니다. 이 UUID로 되돌리면 모든 파일이 원래 상태로 복원됩니다. 여러 체크포인트를 저장하고 중간 상태로 되돌리려면 [다중 복원 지점](#다중-복원-지점)을 참조하세요.

    세션 ID(`message.session_id`) 캡처는 선택 사항입니다. 스트림이 완료된 후 나중에 되돌리려는 경우에만 필요합니다. 메시지를 처리하는 동안 즉시 `rewindFiles()`를 호출하는 경우([위험한 작업 전 체크포인트](#위험한-작업-전-체크포인트) 예제 참조) 세션 ID 캡처를 건너뛸 수 있습니다.

    <CodeGroup>
      ```python Python theme={null}
      checkpoint_id = None
      session_id = None

      async for message in client.receive_response():
          # 첫 번째 사용자 메시지 UUID를 체크포인트로 캡처
          if isinstance(message, UserMessage) and message.uuid and checkpoint_id is None:
              checkpoint_id = message.uuid
          # 결과 메시지에서 세션 ID 캡처
          if isinstance(message, ResultMessage):
              session_id = message.session_id
      ```

      ```typescript TypeScript theme={null}
      let checkpointId: string | undefined;
      let sessionId: string | undefined;

      for await (const message of response) {
        // 첫 번째 사용자 메시지 UUID를 체크포인트로 캡처
        if (message.type === "user" && message.uuid && !checkpointId) {
          checkpointId = message.uuid;
        }
        // 세션 ID가 있는 메시지에서 캡처
        if ("session_id" in message) {
          sessionId = message.session_id;
        }
      }
      ```
    </CodeGroup>
  </Step>

  <Step title="파일 되돌리기(Rewind)">
    스트림이 완료된 후 되돌리려면 빈 프롬프트로 세션을 다시 시작하고 체크포인트 UUID와 함께 `rewind_files()` (Python) 또는 `rewindFiles()` (TypeScript)를 호출하세요. 스트림 도중에 되돌릴 수도 있습니다. 해당 패턴은 [위험한 작업 전 체크포인트](#위험한-작업-전-체크포인트)를 참조하세요.

    <CodeGroup>
      ```python Python theme={null}
      async with ClaudeSDKClient(
          ClaudeAgentOptions(enable_file_checkpointing=True, resume=session_id)
      ) as client:
          await client.query("")  # 연결을 열기 위한 빈 프롬프트
          async for message in client.receive_response():
              if checkpoint_id:
                  await client.rewind_files(checkpoint_id)
              break
      ```

      ```typescript TypeScript theme={null}
      const rewindQuery = query({
        prompt: "", // 연결을 열기 위한 빈 프롬프트
        options: { ...opts, resume: sessionId }
      });

      for await (const msg of rewindQuery) {
        if (checkpointId) {
          await rewindQuery.rewindFiles(checkpointId);
        }
        break;
      }
      ```
    </CodeGroup>

    세션 ID와 체크포인트 ID를 캡처하면 CLI에서 되돌릴 수도 있습니다. 이 명령은 [Claude Code 설치](/docs/en/setup)를 통해 제공되는 `claude` 실행 파일이 필요하며 SDK 패키지에는 설치되지 않습니다. SDK가 체크포인팅을 활성화하지만 `claude -p`를 직접 실행할 때는 `CLAUDE_CODE_ENABLE_SDK_FILE_CHECKPOINTING` 환경 변수를 설정해야 합니다:

    ```bash theme={null}
    CLAUDE_CODE_ENABLE_SDK_FILE_CHECKPOINTING=true claude -p --resume <session-id> --rewind-files <checkpoint-uuid>
    ```

    `--rewind-files` 플래그는 `claude --help` 출력에 표시되지 않지만 CLI는 위에 표시된 대로 이를 수용합니다.
  </Step>
</Steps>

## 일반적인 패턴

이 패턴들은 유즈케이스에 따라 체크포인트 UUID를 캡처하고 사용하는 다양한 방법을 보여줍니다.

### 위험한 작업 전 체크포인트

이 패턴은 각 에이전트 턴 전에 업데이트하여 가장 최근의 체크포인트 UUID만 유지합니다. 처리 중에 문제가 발생하면 즉시 마지막 안전 상태로 되돌리고 루프에서 벗어날 수 있습니다.

이 예제를 실행하기 전에 `your_revert_condition` (Python) 또는 `yourRevertCondition` (TypeScript)을 오류 탐지나 검증 실패 확인과 같은 고유한 확인 로직으로 교체하세요. 자리표시자는 예제에 정의되어 있지 않습니다.

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, UserMessage


  async def main():
      options = ClaudeAgentOptions(
          enable_file_checkpointing=True,
          permission_mode="acceptEdits",
          extra_args={"replay-user-messages": None},
      )

      safe_checkpoint = None

      async with ClaudeSDKClient(options) as client:
          await client.query("Refactor the authentication module")

          async for message in client.receive_response():
              # 각 에이전트 턴이 시작되기 전에 체크포인트 업데이트
              # 이전 체크포인트를 덮어쓰므로 최신만 유지됨
              if isinstance(message, UserMessage) and message.uuid:
                  safe_checkpoint = message.uuid

              # 로직에 따라 되돌릴 시점 결정
              # 예: 오류 탐지, 검증 실패, 사용자 입력 등
              if your_revert_condition and safe_checkpoint:
                  await client.rewind_files(safe_checkpoint)
                  # 되돌린 후 루프 종료, 파일이 복원됨
                  break


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  async function main() {
    const response = query({
      prompt: "Refactor the authentication module",
      options: {
        enableFileCheckpointing: true,
        permissionMode: "acceptEdits" as const,
        extraArgs: { "replay-user-messages": null }
      }
    });

    let safeCheckpoint: string | undefined;

    for await (const message of response) {
      // 각 에이전트 턴이 시작되기 전에 체크포인트 업데이트
      // 이전 체크포인트를 덮어쓰므로 최신만 유지됨
      if (message.type === "user" && message.uuid) {
        safeCheckpoint = message.uuid;
      }

      // 로직에 따라 되돌릴 시점 결정
      // 예: 오류 탐지, 검증 실패, 사용자 입력 등
      if (yourRevertCondition && safeCheckpoint) {
        await response.rewindFiles(safeCheckpoint);
        // 되돌린 후 루프 종료, 파일이 복원됨
        break;
      }
    }
  }

  main();
  ```
</CodeGroup>

### 다중 복원 지점

Claude가 여러 턴에 걸쳐 변경하는 경우 처음으로 완전히 돌아가는 대신 특정 시점으로 되돌리고 싶을 수 있습니다. 예를 들어 Claude가 1턴에서 파일을 리팩토링하고 2턴에서 테스트를 추가할 때, 리팩토링은 유지하고 테스트만 취소하고 싶을 수 있습니다.

이 패턴은 메타데이터와 함께 모든 체크포인트 UUID를 배열에 저장합니다. 세션이 완료된 후 이전 체크포인트 중 하나로 되돌릴 수 있습니다:

<CodeGroup>
  ```python Python theme={null}
  import asyncio
  from dataclasses import dataclass
  from datetime import datetime
  from claude_agent_sdk import (
      ClaudeSDKClient,
      ClaudeAgentOptions,
      UserMessage,
      ResultMessage,
  )


  # 더 나은 추적을 위해 체크포인트 메타데이터 저장
  @dataclass
  class Checkpoint:
      id: str
      description: str
      timestamp: datetime


  async def main():
      options = ClaudeAgentOptions(
          enable_file_checkpointing=True,
          permission_mode="acceptEdits",
          extra_args={"replay-user-messages": None},
      )

      checkpoints = []
      session_id = None

      async with ClaudeSDKClient(options) as client:
          await client.query("Refactor the authentication module")

          async for message in client.receive_response():
              if isinstance(message, UserMessage) and message.uuid:
                  checkpoints.append(
                      Checkpoint(
                          id=message.uuid,
                          description=f"After turn {len(checkpoints) + 1}",
                          timestamp=datetime.now(),
                      )
                  )
              if isinstance(message, ResultMessage) and not session_id:
                  session_id = message.session_id

      # 나중에: 세션을 다시 시작하여 임의의 체크포인트로 되돌리기
      if checkpoints and session_id:
          target = checkpoints[0]  # 임의의 체크포인트 선택
          async with ClaudeSDKClient(
              ClaudeAgentOptions(enable_file_checkpointing=True, resume=session_id)
          ) as client:
              await client.query("")  # 연결을 열기 위한 빈 프롬프트
              async for message in client.receive_response():
                  await client.rewind_files(target.id)
                  break
          print(f"Rewound to: {target.description}")


  asyncio.run(main())
  ```

  ```typescript TypeScript theme={null}
  import { query } from "@anthropic-ai/claude-agent-sdk";

  // 더 나은 추적을 위해 체크포인트 메타데이터 저장
  interface Checkpoint {
    id: string;
    description: string;
    timestamp: Date;
  }

  async function main() {
    const opts = {
      enableFileCheckpointing: true,
      permissionMode: "acceptEdits" as const,
      extraArgs: { "replay-user-messages": null }
    };

    const response = query({
      prompt: "Refactor the authentication module",
      options: opts
    });

    const checkpoints: Checkpoint[] = [];
    let sessionId: string | undefined;

    try {
      for await (const message of response) {
        if (message.type === "user" && message.uuid) {
          checkpoints.push({
            id: message.uuid,
            description: `After turn ${checkpoints.length + 1}`,
            timestamp: new Date()
          });
        }
        if ("session_id" in message && !sessionId) {
          sessionId = message.session_id;
        }
      }
    } catch (error) {
      // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
      // 오류 결과였더라도 sessionId와 checkpoints 배열은 위 루프에서 이미 채워진 상태입니다.
      // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
      console.error(`Session ended with an error: ${error}`);
    }

    // 나중에: 세션을 다시 시작하여 임의의 체크포인트로 되돌리기
    if (checkpoints.length > 0 && sessionId) {
      const target = checkpoints[0]; // 임의의 체크포인트 선택
      const rewindQuery = query({
        prompt: "", // 연결을 열기 위한 빈 프롬프트
        options: { ...opts, resume: sessionId }
      });

      for await (const msg of rewindQuery) {
        await rewindQuery.rewindFiles(target.id);
        break;
      }
      console.log(`Rewound to: ${target.description}`);
    }
  }

  main();
  ```
</CodeGroup>

## 대화형 예제 시도해보기

이 전체 예제는 소형 유틸리티 파일을 생성하고, 에이전트에게 문서 주석을 추가하도록 요청하고, 변경 사항을 보여준 뒤 되돌릴지 묻습니다.

시작하기 전에 [Claude Agent SDK가 설치되어 있는지](/docs/en/agent-sdk/quickstart) 확인하세요.

<Steps>
  <Step title="테스트 파일 생성">
    `utils.py` (Python) 또는 `utils.ts` (TypeScript)라는 새 파일을 생성하고 다음 코드를 붙여넣으세요:

    <CodeGroup>
      ```python utils.py theme={null}
      def add(a, b):
          return a + b


      def subtract(a, b):
          return a - b


      def multiply(a, b):
          return a * b


      def divide(a, b):
          if b == 0:
              raise ValueError("Cannot divide by zero")
          return a / b
      ```

      ```typescript utils.ts theme={null}
      export function add(a: number, b: number): number {
        return a + b;
      }

      export function subtract(a: number, b: number): number {
        return a - b;
      }

      export function multiply(a: number, b: number): number {
        return a * b;
      }

      export function divide(a: number, b: number): number {
        if (b === 0) {
          throw new Error("Cannot divide by zero");
        }
        return a / b;
      }
      ```
    </CodeGroup>
  </Step>

  <Step title="대화형 예제 실행">
    유틸리티 파일과 동일한 디렉토리에 `try_checkpointing.py` (Python) 또는 `try_checkpointing.ts` (TypeScript)라는 새 파일을 생성하고 다음 코드를 붙여넣으세요.

    이 스크립트는 Claude에게 유틸리티 파일에 문서 주석을 추가하도록 요청한 다음 원본으로 되돌리고 복원할 수 있는 옵션을 제공합니다.

    <CodeGroup>
      ```python try_checkpointing.py theme={null}
      import asyncio
      from claude_agent_sdk import (
          ClaudeSDKClient,
          ClaudeAgentOptions,
          UserMessage,
          ResultMessage,
      )


      async def main():
          # 체크포인팅이 활성화된 상태로 SDK 구성
          # - enable_file_checkpointing: 되돌리기를 위해 파일 변경 사항 추적
          # - permission_mode: 프롬프트 없이 파일 편집 자동 승인
          # - extra_args: 스트림에서 사용자 메시지 UUID를 받기 위해 필수
          options = ClaudeAgentOptions(
              enable_file_checkpointing=True,
              permission_mode="acceptEdits",
              extra_args={"replay-user-messages": None},
          )

          checkpoint_id = None  # 되돌리기를 위해 사용자 메시지 UUID 저장
          session_id = None  # 재시작을 위해 세션 ID 저장

          print("Running agent to add doc comments to utils.py...\n")

          # 에이전트를 실행하고 응답 스트림에서 체크포인트 데이터 캡처
          async with ClaudeSDKClient(options) as client:
              await client.query("Add doc comments to utils.py")

              async for message in client.receive_response():
                  # 첫 번째 사용자 메시지 UUID 캡처 - 이것이 복원 지점임
                  if isinstance(message, UserMessage) and message.uuid and not checkpoint_id:
                      checkpoint_id = message.uuid
                  # 나중에 다시 시작할 수 있도록 세션 ID 캡처
                  if isinstance(message, ResultMessage):
                      session_id = message.session_id

          print("Done! Open utils.py to see the added doc comments.\n")

          # 사용자에게 변경 사항을 되돌릴지 확인
          if checkpoint_id and session_id:
              response = input("Rewind to remove the doc comments? (y/n): ")

              if response.lower() == "y":
                  # 빈 프롬프트로 세션을 다시 시작한 다음 되돌리기
                  async with ClaudeSDKClient(
                      ClaudeAgentOptions(enable_file_checkpointing=True, resume=session_id)
                  ) as client:
                      await client.query("")  # 빈 프롬프트로 연결 열기
                      async for message in client.receive_response():
                          await client.rewind_files(checkpoint_id)  # 파일 복원
                          break

                  print(
                      "\n✓ File restored! Open utils.py to verify the doc comments are gone."
                  )
              else:
                  print("\nKept the modified file.")


      asyncio.run(main())
      ```

      ```typescript try_checkpointing.ts theme={null}
      import { query } from "@anthropic-ai/claude-agent-sdk";
      import * as readline from "readline";

      async function main() {
        // 체크포인팅이 활성화된 상태로 SDK 구성
        // - enableFileCheckpointing: 되돌리기를 위해 파일 변경 사항 추적
        // - permissionMode: 프롬프트 없이 파일 편집 자동 승인
        // - extraArgs: 스트림에서 사용자 메시지 UUID를 받기 위해 필수
        const opts = {
          enableFileCheckpointing: true,
          permissionMode: "acceptEdits" as const,
          extraArgs: { "replay-user-messages": null }
        };

        let sessionId: string | undefined; // 재시작을 위해 세션 ID 저장
        let checkpointId: string | undefined; // 되돌리기를 위해 사용자 메시지 UUID 저장

        console.log("Running agent to add doc comments to utils.ts...\n");

        // 에이전트를 실행하고 응답 스트림에서 체크포인트 데이터 캡처
        const response = query({
          prompt: "Add doc comments to utils.ts",
          options: opts
        });

        try {
          for await (const message of response) {
            // 첫 번째 사용자 메시지 UUID 캡처 - 이것이 복원 지점임
            if (message.type === "user" && message.uuid && !checkpointId) {
              checkpointId = message.uuid;
            }
            // 나중에 다시 시작할 수 있도록 세션 ID 캡처
            if ("session_id" in message) {
              sessionId = message.session_id;
            }
          }
        } catch (error) {
          // 단발성 query()는 오류 결과를 생성한 후 예외를 던집니다.
          // 오류 결과였더라도 checkpointId와 sessionId는 위 루프에서 이미 캡처된 상태입니다.
          // 연결 또는 프로세스 실패는 결과 메시지를 생성하지 않습니다.
          console.error(`Session ended with an error: ${error}`);
        }

        console.log("Done! Open utils.ts to see the added doc comments.\n");

        // 사용자에게 변경 사항을 되돌릴지 확인
        if (checkpointId && sessionId) {
          const rl = readline.createInterface({
            input: process.stdin,
            output: process.stdout
          });

          const answer = await new Promise<string>((resolve) => {
            rl.question("Rewind to remove the doc comments? (y/n): ", resolve);
          });
          rl.close();

          if (answer.toLowerCase() === "y") {
            // 빈 프롬프트로 세션을 다시 시작한 다음 되돌리기
            const rewindQuery = query({
              prompt: "", // 빈 프롬프트로 연결 열기
              options: { ...opts, resume: sessionId }
            });

            for await (const msg of rewindQuery) {
              await rewindQuery.rewindFiles(checkpointId); // 파일 복원
              break;
            }

            console.log("\n✓ File restored! Open utils.ts to verify the doc comments are gone.");
          } else {
            console.log("\nKept the modified file.");
          }
        }
      }

      main();
      ```
    </CodeGroup>

    이 예제는 완전한 체크포인팅 워크플로우를 보여줍니다:

    1. **체크포인팅 활성화**: `enable_file_checkpointing=True` 및 파일 편집 자동 승인을 위한 `permission_mode="acceptEdits"`로 SDK 구성
    2. **체크포인트 데이터 캡처**: 에이전트 실행 시 첫 번째 사용자 메시지 UUID(복원 지점) 및 세션 ID 저장
    3. **되돌리기 확인**: 에이전트가 끝난 후 유틸리티 파일을 확인하여 문서 주석을 살펴보고 변경 사항 취소 여부 결정
    4. **다시 시작 및 되돌리기**: 예 선택 시 빈 프롬프트로 세션을 다시 시작하고 `rewind_files()`를 호출하여 원본 파일 복원
  </Step>

  <Step title="예제 실행">
    유틸리티 파일이 있는 동일한 디렉토리에서 스크립트를 실행하세요.

    <Tip>
      스크립트를 실행하기 전에 IDE나 에디터에서 유틸리티 파일(`utils.py` 또는 `utils.ts`)을 열어두세요. 에이전트가 문서 주석을 추가할 때 실시간으로 파일이 업데이트되고, 되돌리기를 선택하면 원본으로 되돌아가는 모습을 볼 수 있습니다.
    </Tip>

    <Tabs>
      <Tab title="Python">
        ```bash theme={null}
        python try_checkpointing.py
        ```
      </Tab>

      <Tab title="TypeScript">
        ```bash theme={null}
        npx tsx try_checkpointing.ts
        ```
      </Tab>
    </Tabs>

    에이전트가 문서 주석을 추가한 후 되돌릴지 묻는 프롬프트가 표시됩니다. 예(y)를 선택하면 파일이 원본 상태로 복원됩니다.
  </Step>
</Steps>

## 제한 사항

파일 체크포인팅에는 다음과 같은 제한 사항이 있습니다:

| 제한 사항 | 설명 |
| ---------------------------------- | -------------------------------------------------------------------- |
| Write/Edit/NotebookEdit 도구 전용 | Bash 명령을 통해 이루어진 변경 사항은 추적되지 않음 |
| 동일한 세션 | 체크포인트는 이를 생성한 세션에 묶여 있음 |
| 파일 콘텐츠 전용 | 디렉토리 생성, 이동, 삭제는 되돌리기로 취소되지 않음 |
| 로컬 파일 | 원격 또는 네트워크 파일은 추적되지 않음 |

## 문제 해결

### 체크포인팅 옵션이 인식되지 않음

`enableFileCheckpointing` 또는 `rewindFiles()`를 사용할 수 없다면 이전 SDK 버전을 사용 중일 수 있습니다.

**해결 방법**: 최신 SDK 버전으로 업데이트하세요:

* **Python**: `pip install --upgrade claude-agent-sdk`
* **TypeScript**: `npm install @anthropic-ai/claude-agent-sdk@latest`

### 사용자 메시지에 UUID가 없음

`message.uuid`가 `undefined`이거나 누락된 경우 체크포인트 UUID를 수신하지 못하는 상태입니다.

**원인**: `replay-user-messages` 옵션이 설정되지 않았습니다.

**해결 방법**: 옵션에 `extra_args={"replay-user-messages": None}` (Python) 또는 `extraArgs: { 'replay-user-messages': null }` (TypeScript)를 추가하세요.

### "No file checkpoint found for message" 오류

이 오류는 지정된 사용자 메시지 UUID에 대한 체크포인트 데이터가 존재하지 않을 때 발생합니다.

**일반적인 원인**:

* 원본 세션에서 파일 체크포인팅이 활성화되지 않음(`enable_file_checkpointing` 또는 `enableFileCheckpointing`이 `true`로 설정되지 않음)
* 세션을 다시 시작하고 되돌리기 전에 세션이 정상적으로 완료되지 않음

**해결 방법**: 원본 세션에 `enable_file_checkpointing=True` (Python) 또는 `enableFileCheckpointing: true` (TypeScript)가 설정되어 있었는지 확인한 다음 예제에 표시된 패턴을 사용하세요: 첫 번째 사용자 메시지 UUID 캡처, 세션 완전 종료, 빈 프롬프트로 다시 시작 후 `rewindFiles()` 1회 호출.

### "File rewinding is not enabled" 오류

이 오류는 체크포인팅이 활성화되지 않은 상태에서 비대화형 되돌리기를 시도할 때 발생합니다: `--rewind-files`와 함께 순수 `claude -p`를 실행하거나, 옵션에서 체크포인팅을 활성화하지 않은 세션(다시 시작된 세션 포함)을 실행할 때 발생합니다. SDK는 되돌리기를 수행하는 세션에서 `enable_file_checkpointing` (Python) 또는 `enableFileCheckpointing` (TypeScript)이 활성화된 경우에만 내부적으로 `CLAUDE_CODE_ENABLE_SDK_FILE_CHECKPOINTING` 환경 변수를 설정하며, 순수 CLI는 이를 설정하지 않습니다.

**해결 방법**: 순수 CLI의 경우 명령 실행 시 환경 변수를 설정하세요:

```bash theme={null}
CLAUDE_CODE_ENABLE_SDK_FILE_CHECKPOINTING=true claude -p --resume <session-id> --rewind-files <checkpoint-uuid>
```

SDK의 경우 이 페이지의 예제처럼 다시 시작된 세션에 `enable_file_checkpointing=True` (Python) 또는 `enableFileCheckpointing: true` (TypeScript)를 설정하세요.

### "ProcessTransport is not ready for writing" 오류

이 오류는 응답 반복 처리가 완료된 후 `rewindFiles()` 또는 `rewind_files()`를 호출할 때 발생합니다. 루프가 완료되면 CLI 프로세스와의 연결이 닫힙니다.

**해결 방법**: 빈 프롬프트로 세션을 다시 시작한 후 새 쿼리에서 되돌리기를 호출하세요:

<CodeGroup>
  ```python Python theme={null}
  # 빈 프롬프트로 세션 다시 시작 후 되돌리기
  async with ClaudeSDKClient(
      ClaudeAgentOptions(enable_file_checkpointing=True, resume=session_id)
  ) as client:
      await client.query("")
      async for message in client.receive_response():
          if checkpoint_id:
              await client.rewind_files(checkpoint_id)
          break
  ```

  ```typescript TypeScript theme={null}
  // 빈 프롬프트로 세션 다시 시작 후 되돌리기
  const rewindQuery = query({
    prompt: "",
    options: { ...opts, resume: sessionId }
  });

  try {
    for await (const msg of rewindQuery) {
      if (checkpointId) {
        await rewindQuery.rewindFiles(checkpointId);
      }
      break;
    }
  } catch (error) {
    // 여기서 오류가 발생하면 되돌리기가 완료되지 않았음을 의미합니다 (예: 체크포인트를 찾을 수 없음, 세션을 다시 시작할 수 없음)
    console.error(`Rewind session ended with an error: ${error}`);
  }
  ```
</CodeGroup>

## 다음 단계

* **[세션](/docs/en/agent-sdk/sessions)**: 스트림 완료 후 되돌리기에 필요한 세션 다시 시작 방법을 알아봅니다. 세션 ID, 대화 다시 시작, 세션 포크를 다룹니다.
* **[권한](/docs/en/agent-sdk/permissions)**: Claude가 사용할 수 있는 도구와 파일 수정 승인 방식을 구성합니다. 편집 시점을 더 제어하고자 할 때 유용합니다.
* **[TypeScript SDK 참조](/docs/en/agent-sdk/typescript)**: `query()`의 모든 옵션과 `rewindFiles()` 메서드를 포함한 전체 API 참조.
* **[Python SDK 참조](/docs/en/agent-sdk/python)**: `ClaudeAgentOptions`의 모든 옵션과 `rewind_files()` 메서드를 포함한 전체 API 참조.
