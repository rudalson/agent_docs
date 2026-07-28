> ## 문서 색인
> 전체 문서 색인을 확인하려면 다음 주소를 방문하세요: https://code.claude.com/docs/llms.txt
> 추가 탐색을 진행하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 애널리틱스(Analytics)로 팀 사용량 추적하기

> 애널리틱스 대시보드에서 Claude Code 사용량 지표를 확인하고, 도입률을 추적하며, 엔지니어링 속도를 측정하세요.

Claude Code는 조직이 개발자의 사용 패턴을 이해하고, 기여 지표를 추적하며, Claude Code가 엔지니어링 속도에 미치는 영향을 측정할 수 있도록 돕는 애널리틱스 대시보드를 제공합니다. 요금제에 맞는 대시보드에 접근해 보세요:

| 요금제 | 대시보드 URL | 포함 항목 | 자세히 보기 |
| ----------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- | ---------------------------------------------------- |
| Claude for Teams / Enterprise | [claude.ai/analytics/claude-code](https://claude.ai/analytics/claude-code) | 사용 지표, GitHub 연동 기여 지표, 리더보드, 데이터 내보내기 | [상세 보기](#access-analytics-for-team-and-enterprise) |
| API (Claude Console) | [platform.claude.com/claude-code](https://platform.claude.com/claude-code) | 사용 지표, 지출 추적, 팀 인사이트 | [상세 보기](#access-analytics-for-api-customers) |

## Team 및 Enterprise 요금제 애널리틱스

[claude.ai/analytics/claude-code](https://claude.ai/analytics/claude-code)로 이동합니다. 관리자(Admin) 및 소유자(Owner)가 대시보드를 볼 수 있습니다.

Team 및 Enterprise 대시보드 포함 항목:

* **사용 지표**: 수락된 코드 줄 수, 제안 수락률, 일일 활성 사용자 수 및 세션 수
* **기여 지표**: [GitHub 연동](#enable-contribution-metrics)을 통해 Claude Code의 도움을 받아 배포된 PR 및 코드 줄 수
* **리더보드**: Claude Code 사용량 기준 상위 기여자 순위
* **데이터 내보내기**: 커스텀 보고를 위해 기여 데이터를 CSV로 다운로드

사용자별 토큰 수 및 예상 비용은 [OpenTelemetry 내보내기](/docs/en/monitoring-usage)를 구성하거나 조직의 애널리틱스 설정에서 [지출 보고서](https://support.claude.com/en/articles/12883420-view-usage-analytics-for-team-and-enterprise-plans)를 내보내어 확인하세요.

### 기여 지표 활성화하기

<Note>
  기여 지표는 공개 베타 버전이며 Claude for Teams 및 Claude for Enterprise 요금제에서 사용할 수 있습니다. 이 지표는 claude.ai 조직 내의 사용자만 다룹니다. Claude Console API 또는 서드파티 연동을 통한 사용량은 포함되지 않습니다.
</Note>

사용량 및 도입 데이터는 모든 Claude for Teams 및 Claude for Enterprise 계정에서 사용할 수 있습니다. 기여 지표를 사용하려면 GitHub 조직을 연결하는 추가 설정이 필요합니다.

설정을 구성하려면 소유자(Owner) 역할이 필요하며, GitHub 관리자가 GitHub 앱을 설치해야 합니다.

<Warning>
  [Zero Data Retention](/docs/en/zero-data-retention)이 활성화된 조직에서는 기여 지표를 사용할 수 없습니다. 애널리틱스 대시보드에는 사용 지표만 표시됩니다.
</Warning>

<Steps>
  <Step title="GitHub 앱 설치">
    GitHub 관리자가 [github.com/apps/claude](https://github.com/apps/claude)에서 조직의 GitHub 계정에 Claude GitHub 앱을 설치합니다.
  </Step>

  <Step title="Claude Code 애널리틱스 활성화">
    Claude 소유자가 [claude.ai/admin-settings/claude-code](https://claude.ai/admin-settings/claude-code)로 이동하여 Claude Code 애널리틱스 기능을 활성화합니다.
  </Step>

  <Step title="GitHub 애널리틱스 활성화">
    동일한 페이지에서 "GitHub analytics" 토글을 활성화합니다.
  </Step>

  <Step title="GitHub 인증">
    GitHub 인증 흐름을 완료하고 분석에 포함할 GitHub 조직을 선택합니다.
  </Step>
</Steps>

데이터는 활성화 후 보통 24시간 이내에 나타나며 매일 업데이트됩니다.

### 요약 지표 검토

대시보드 상단에 표시되는 요약 지표:

* **PRs with CC**: Claude Code로 작성된 코드가 한 줄 이상 포함된 머지된 총 PR 수
* **Lines of code with CC**: Claude Code의 도움을 받아 작성되어 머지된 모든 PR의 총 코드 줄 수 (3자 이상의 유효한 코드 줄만 카운트)
* **PRs with Claude Code (%)**: 전체 머지된 PR 중 Claude Code의 도움을 받은 코드가 포함된 비율
* **Suggestion accept rate**: 사용자가 Edit, Write, NotebookEdit 도구 등의 코드 편집 제안을 수락한 비율
* **Lines of code accepted**: 세션에서 사용자가 수락한 Claude Code 작성 총 코드 줄 수

## API 고객용 애널리틱스

Claude Console을 사용하는 API 고객은 [platform.claude.com/claude-code](https://platform.claude.com/claude-code)에서 애널리틱스에 접근할 수 있습니다. UsageView 권한이 필요합니다.

Console 대시보드 표시 항목:

* **Lines of code accepted**: 수락된 코드 줄 수
* **Suggestion accept rate**: 코드 편집 도구 제안 수락률
* **Activity**: 일일 활성 사용자 및 세션 차트
* **Spend**: 일일 API 비용 및 사용자 수

## 관련 리소스

* [OpenTelemetry로 모니터링하기](/docs/en/monitoring-usage): 실시간 지표 및 이벤트를 관측 가능성 스택으로 내보내기
* [효율적인 비용 관리](/docs/en/costs): 지출 한도 설정 및 토큰 사용량 최적화
* [권한](/docs/en/permissions): 역할 및 권한 구성
