---
slug: implementation-plan
section: Antigravity 2.0
title: 계획서
path:
  - Antigravity 2.0
  - 아티팩트
  - 계획서
---

# 구현 계획서 (Implementation Plan)

에이전트는 코드베이스 내에서 작업을 완료하기 위해 변경 사항을 설계하는 과정에서 구현 계획서 아티팩트를 활용합니다. 이 계획서에는 필요한 수정 사항에 대한 기술적인 세부 정보가 포함되어 있으며 사용자가 검토할 수 있도록 제공됩니다. 아래는 에이전트가 생성한 계획서의 예시입니다.

![아티팩트 구현 계획서](../../.assets/image/artifact-implementation-plan.png)

아티팩트 검토 정책이 "항상 진행(Always Proceed)" [이 설정에 관한 문서 링크]으로 설정되어 있지 않은 한, 에이전트는 일반적으로 작업을 완료하는 데 필요한 변경 사항을 적용하기 전에 구현 계획서에 대한 사용자의 검토를 요청합니다. 대화창 내 버튼이나 아티팩트 헤더에 있는 "진행(Proceed)" 버튼을 클릭하면 에이전트의 계획대로 작업을 즉시 계속 진행할 수 있습니다.

![아티팩트 구현 계획서 진행](../../.assets/image/artifact-implementation-plan-proceed.png)

때로는 에이전트가 사용자가 정확히 원하는 방향과 약간 다른 계획을 수립할 수 있습니다. Antigravity는 이러한 아티팩트에 댓글을 다는 기능을 지원하므로, 변경 범위 축소, 다른 기술 스택 사용 또는 에이전트의 불일치 사항 수정 등 어떤 이유로든 에이전트에게 피드백을 제공할 수 있습니다.

![아티팩트 구현 계획서 댓글](../../.assets/image/artifact-implementation-plan-comments.png)
 
구현 계획서에 댓글을 남긴 후에도 "진행(Proceed)"을 눌러 에이전트의 계획대로 계속 진행할 수 있습니다. 또는 아티팩트 헤더의 "검토(Review)" 버튼을 토글하여 작성된 모든 댓글을 검사하고, 바로 진행하는 대신 피드백 메시지를 남겨서 전송할 수도 있습니다.

![아티팩트 구현 계획서 댓글 제출](../../.assets/image/artifact-implementation-plan-submit-comments.png)

사용자가 진행을 수락하거나 검토 의견을 제출하면, 에이전트는 작업을 계속 수행하며 구현 계획서를 재수정하여 검토를 다시 요청하거나 본격적인 작업을 시작하게 됩니다!

![아티팩트 구현 계획서 진행됨](../../.assets/image/artifact-implementation-plan-proceeded.png)
