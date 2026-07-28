---
slug: ide-implementation-plan
section: Antigravity IDE
title: 계획 (Plan)
path:
  - Antigravity IDE
  - 아티팩트
  - 계획 (Plan)
---

# 구현 계획 (Implementation Plan)

에이전트는 구현 계획 아티팩트를 활용하여 작업을 완료하기 위해 코드베이스 내의 변경 사항을 설계합니다. 이러한 계획에는 필요한 수정 사항에 대한 기술적 세부 정보가 포함되어 있으며 사용자가 검토할 수 있도록 되어 있습니다. 아래는 에이전트가 생성한 구현 계획의 예입니다.

![아티팩트 구현 계획](assets/image/docs/artifacts/artifact-implementation-plan.png)

아티팩트 검토 정책을 "Always Proceed"[이 설정에 대한 문서 링크]로 설정하지 않은 한, 에이전트는 일반적으로 작업을 완료하는 데 필요한 변경을 수행하기 전에 구현 계획에 대한 검토를 요청합니다. 대화 내의 또는 아티팩트 헤더의 "Proceed" 버튼을 클릭하여 즉시 에이전트의 계획을 계속 진행할 수 있습니다.

![아티팩트 구현 계획 진행](assets/image/docs/artifacts/artifact-implementation-plan-proceed.png)

종종 에이전트가 사용자가 정확히 원하는 것과 약간 다른 계획을 세울 수 있습니다. Antigravity는 변경 범위 축소, 다른 기술 스택 사용 또는 에이전트의 불일치 수정 등 어떤 이유로든 에이전트에게 피드백을 제공할 수 있도록 이러한 아티팩트에 대한 댓글 작성을 지원합니다.

![아티팩트 구현 계획 댓글](assets/image/docs/artifacts/artifact-implementation-plan-comments.png)
 
구현 계획에 댓글을 남긴 후에도 "Proceed"를 사용하여 에이전트의 계획을 계속 진행할 수 있습니다. 또는 아티팩트 헤더에서 "Review" 버튼을 토글하여 필요한 경우 직접 진행하는 대신 모든 댓글을 검토하고 피드백 메시지를 남길 수도 있습니다.

![아티팩트 구현 계획 댓글 제출](assets/image/docs/artifacts/artifact-implementation-plan-submit-comments.png)

진행하거나 검토 의견을 남기면 에이전트는 구현 계획을 반복하여 검토를 다시 요청하거나 작업을 시작하여 작업을 계속 진행합니다!

![아티팩트 구현 계획 진행됨](assets/image/docs/artifacts/artifact-implementation-plan-proceeded.png)
