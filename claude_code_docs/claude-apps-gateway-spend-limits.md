> ## 문서 색인
> 다음 위치에서 전체 문서 색인을 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude apps gateway 지출 한도 (Spend limits)

> Claude apps gateway를 통한 각 개발자의 지출을 일별, 주별 또는 월별로 제한합니다. 관리자 API를 통해 한도를 설정하면 게이트웨이가 모든 요청에서 실시간으로 한도를 적용합니다.

지출 한도는 일별, 주별 또는 월별로 특정 [Claude apps gateway](/docs/en/claude-apps-gateway)를 통한 각 개발자의 지출 상한선을 정합니다. 개발자가 자신의 한도를 초과하면 게이트웨이는 다음 요청에서 `429`를 반환하고 기간이 리셋되거나 관리자가 상한선을 올릴 때까지 차단합니다. 지출 한도를 사용하여 모든 사람이 공유하는 자격 증명에 대해 각 개발자, 그룹 또는 조직 전체에 상한선을 부여하세요.

Claude apps gateway는 하나의 공유된 상류(upstream) 자격 증명을 통해 모든 추론을 전달하므로 제공업체의 청구서는 개별 개발자가 아닌 해당 자격 증명에 모든 금액을 귀속시킵니다. 개발자별 한도가 없으면 하나의 폭주하는 에이전트 플릿이 조직의 전체 약정 금액을 지출할 수 있습니다. 지출 한도는 공유 청구서 위에 적용되는 게이트웨이의 개발자별 보기 및 서킷 브레이커(circuit breaker)입니다.

## 한도 설정

`gateway.yaml`에 [`admin:`](/docs/en/claude-apps-gateway-config#admin) 블록을 구성하면 게이트웨이는 `/v1/organizations/spend_limits`에서 관리자 API를 제공하고 모든 추론 요청에 실시간으로 상한선을 적용합니다. 상한선 자체는 `gateway.yaml`이 아닌 해당 API를 통해 설정됩니다. 각 `POST /v1/organizations/spend_limits` 요청은 `{scope, amount, period}`에서 하나의 상한선을 생성하거나 교체합니다. 이 API는 Anthropic 공개 [관리자 API](https://platform.claude.com/docs/en/manage-claude/admin-api) 지출 한도 엔드포인트의 통신 형식을 미러링하므로, 해당 계약에 맞춰 작성된 HTTP 클라이언트는 베이스 URL을 변경하여 게이트웨이를 타겟팅할 수 있습니다.

다음 요청은 모든 개발자에 대해 월 \$500의 조직 전체 기본값을 설정합니다:

```bash theme={null}
curl -sS https://claude-gateway.internal.example.com/v1/organizations/spend_limits \
  -H "x-api-key: $GATEWAY_ADMIN_WRITE_KEY" \
  -H "Content-Type: application/json" \
  -d '{"scope": {"type": "organization"}, "amount": "50000", "period": "monthly"}'
```

다음 요청은 `contractors` 그룹의 각 구성원에 대해 일 \$100의 더 엄격한 상한선을 계층화합니다:

```bash theme={null}
curl -sS https://claude-gateway.internal.example.com/v1/organizations/spend_limits \
  -H "x-api-key: $GATEWAY_ADMIN_WRITE_KEY" \
  -H "Content-Type: application/json" \
  -d '{"scope": {"type": "rbac_group", "rbac_group_id": "contractors"}, "amount": "10000", "period": "daily"}'
```

| 필드         | 값                                          | 설명                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ------------ | ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `scope.type` | `user`, `rbac_group`, `organization`        | `user`는 정체성 제공업체가 할당하는 고유 사용자 ID인 OpenID Connect (OIDC) `sub`를 기반으로 특정 개발자를 지정합니다(`scope.user_id`로 전달). `rbac_group`은 이름으로 [IdP 그룹](/docs/en/claude-apps-gateway-config#managed)을 지정합니다(`scope.rbac_group_id`로 전달). `organization`은 조직 전체 기본값입니다. 게이트웨이는 3가지 모두를 수락합니다. Anthropic 공개 `POST`는 현재 `user` 전용입니다. |
| `amount`     | USD 센트 단위의 정수 문자열, 또는 `null`    | `null`은 제한 없음입니다. `"0"`은 무조건 거부(zero cap)를 의미하며 모든 요청을 차단합니다.                                                                                                                                                                                                                                                                                                    |
| `period`     | `daily`, `weekly`, `monthly`                | 스코프는 기간당 하나의 상한선을 가질 수 있으며 각 상한선은 독립적으로 적용됩니다. 개발자가 이 중 하나라도 넘어서면 차단됩니다.                                                                                                                                                                                                                                                                |

그룹 또는 조직 상한선은 공유 풀이 아닌 각 구성원이 상속하는 사용자별 기본값입니다. 기간별로 개발자의 적용 한도는 다음 순서로 해소됩니다: 사용자별 재정의, 그 다음 가장 제한적인 그룹 상한선, 그 다음 조직 기본값, 그 다음 제한 없음. [`admin.group_limit_mode: max`](/docs/en/claude-apps-gateway-config#admin)는 다중 그룹 동점 처리 방식을 가장 덜 제한적인 방식으로 전환합니다.

### 관리자 API 인증

다음 중 하나를 전송하세요:

* 전체 접근 권한의 경우 [`admin.write_keys`](/docs/en/claude-apps-gateway-config#admin)의 키와 일치하는 `x-api-key` 헤더를 전송하고, `GET` 전용 접근 권한의 경우 `admin.read_keys`를 전송합니다. 각 키는 감사 로그에 `admin-key:<id>`로 표시되는 `id`를 가지므로 Terraform, CI 및 각 자동화에 고유 키를 부여하세요.
* `groups` 클레임에 [`admin.admin_groups`](/docs/en/claude-apps-gateway-config#admin) 중 하나가 포함된 게이트웨이 전달자 토큰. 이는 전체 접근 권한이며 `oidc:<sub>`로 감사되므로 인간 관리자에게 이를 사용하세요.

## 적용 동작 방식

모든 `/v1/messages` 요청 시 게이트웨이는 단일 Postgres 쿼리로 개발자의 상한선과 현재 기간 누적 지출을 해소합니다. 임의의 상한선을 초과하는 경우 요청은 `error.type: billing_error` 및 `x-should-retry: false` 헤더와 함께 `429`를 반환합니다. 메시지는 `spend limit reached`이며, 설정된 경우 [`admin.blocked_message`](/docs/en/claude-apps-gateway-config#admin)가 뒤따릅니다.

`/v1/messages/count_tokens`는 제외됩니다. 토큰 카운팅은 무료이므로 상한선 상태와 상관없이 실행됩니다.

각 응답 후 사용량 측정기(meter)는 클라이언트로 스트리밍될 때 응답에서 토큰 수를 읽고, 이를 USD 정가로 책정하며, 3가지 기간 버킷 모두에 대해 Postgres 카운터를 증분합니다. 측정기는 스트림의 단일 리더(reader)이므로 클라이언트의 바이트는 변경되지 않으며 측정 실패가 응답을 깨뜨리지 않습니다.

지출 한도는 USD 정가의 토큰 수에서 지출을 추정합니다. 이는 서킷 브레이커이며 송장이 아닙니다. 정식 청구의 경우 Anthropic Usage & Cost Admin API, Amazon Bedrock의 호출 로그 또는 Google Cloud의 Cloud Monitoring과 같은 제공업체의 사용량 보고서와 대조하세요.

가격 책정은 Anthropic, Amazon Bedrock (`us.anthropic.…-v1:0`), Google Cloud Agent Platform (`claude-…@date`) 및 Microsoft Foundry ID 형식에 걸쳐 동일한 모델 ID 정규화를 적용하여 Claude Code CLI가 자체 비용 표시에 사용하는 것과 동일한 테이블을 사용합니다. Microsoft Foundry 배포 이름이나 추론 프로필 ARN과 같이 테이블이 분류할 수 없는 모델 ID는 0으로 책정되는 대신 입력/출력 백만 토큰당 \$5/\$25의 알 수 없는 모델 기본 티어로 책정되므로, 인식되지 않은 ID가 측정되지 않은 채 상한선을 우회할 수 없습니다. 게이트웨이는 모델이 폴백을 통해 가격 책정될 때 부팅 시 및 런타임에 ID당 한 번 경고를 기록합니다.

클라이언트의 중단(abort)도 청구됩니다. 상류는 스트림의 최종 프레임에서만 출력 토큰을 보고하므로 중단된 스트림은 해당 토큰을 전달하지 않습니다. 측정기는 스트리밍된 콘텐츠 크기에서 토큰당 약 4자의 보수적인 하한 추정치를 유지하고, 최종 사용량 프레임이 누락된 경우에만 이에 대한 비용을 청구합니다. 완료된 스트림은 항상 상류 보고 수를 청구합니다. 이것이 없으면 한도에 도달한 개발자가 출력을 스트리밍하고 각 요청이 끝나기 직전에 중단하여 집계되지 않고 지출할 수 있게 됩니다.

### Postgres 가용성

사전 검사는 2초 타임아웃으로 Postgres를 쿼리합니다. 저장소에 도달할 수 없거나 타임아웃이 발생하면 적용 로직은 기본적으로 fail-open으로 작동합니다: 요청이 진행되고 게이트웨이가 경고를 기록합니다. 대신 fail-closed로 변경하려면 [`enforcement.fail_closed_on_error: true`](/docs/en/claude-apps-gateway-config#enforcement)를 설정하세요. 그러면 `spend limit unavailable` 메시지와 함께 동일한 `429 billing_error`가 반환됩니다. Fail-open은 저장소 장애가 추론 장애로 이어지는 것을 방지하며, fail-closed는 측정되지 않는 지출이 없음을 보장합니다.

## 관리자 API 참조

아래 엔드포인트는 `/v1/organizations/spend_limits` 아래에서 제공됩니다.

| 메서드 및 경로                                 | 설명                                                         |
| ---------------------------------------------- | ------------------------------------------------------------ |
| `GET /v1/organizations/spend_limits`           | 구성된 상한선 목록 조회. 쿼리: `?limit=&after_id=&before_id=`.|
| `POST /v1/organizations/spend_limits`          | `{scope, period}`에 대한 상한선 생성 또는 교체.             |
| `GET /v1/organizations/spend_limits/{id}`      | `spl_` 접두사 ID로 하나의 상한선 가져오기.                   |
| `DELETE /v1/organizations/spend_limits/{id}`   | 하나의 상한선 삭제. `{type: "spend_limit_deleted", id}` 반환. |
| `GET /v1/organizations/spend_limits/effective` | 기간별 보안 주체별 적용 상한선 및 누적 지출 해소 결과.       |
| `GET /v1/organizations/spend_limits/audit`     | 관리자 변경 이력 (최신순). 쿼리: `?limit=`.                  |

관례는 Anthropic 관리자 API를 미러링합니다:

* 모든 객체의 `type`
* `spl_` 접두사 ID
* USD 센트 단위의 정수 문자열 형태의 금액 (`POST`는 다른 `currency`에 대해 `400`으로 거부)
* `{type: "error", error: {type, message}, request_id}` 오류 봉투
* 모든 관리자 응답(성공 또는 에러)의 `request-id` 응답 헤더(본문의 `request_id`와 일치)

모든 변경은 동일한 트랜잭션 내에서 `admin_audit`에 변경 전/후 행을 기록하며 `admin-key:<id>` 또는 `oidc:<sub>`로 귀속됩니다.

게이트웨이는 지출 한도 엔드포인트만 제공합니다. `spend_limit_increase_requests` 큐와 같은 다른 관리자 API 표면은 게이트웨이 관리자 API의 일부가 아닙니다.

### `/effective`

`GET /v1/organizations/spend_limits/effective`는 Anthropic의 `SpendSummary` 스키마를 반환합니다: 각 행은 적용 상한선, 현재 기간 누적 지출 및 `actor` 객체를 포함하는 기간별 보안 주체입니다. 게이트웨이 전용 차이점:

* `user_id`는 OIDC `sub`입니다.
* `actor.name` 및 `actor.email_address`는 게이트웨이를 통한 보안 주체의 첫 추론 요청 전까지 `null`입니다. 게이트웨이에는 사용자 디렉터리가 없으며, 각 사용자의 자체 세션 JWT에서 최근 확인된 값을 기록합니다.
* 각 행은 보안 주체의 최근 확인된 IdP 그룹인 `groups` 배열도 전달합니다. 이는 관리자 UI가 적용되는 모든 한도 계층을 표시할 수 있도록 하는 게이트웨이 확장이며 Anthropic 형식의 클라이언트는 이를 무시합니다.
* `user_ids[]` 필터가 없으면 게이트웨이가 모든 조직 구성원을 열거할 수 없으므로 지출이 기록된 보안 주체를 나열합니다.

그룹 기반 상한선은 적용 시 사용하는 것과 동일한 `group_limit_mode` 동점 처리 방식으로 해당 최근 확인된 그룹에 대해 해소되므로 조회자는 실제로 적용되는 상한선을 볼 수 있습니다.

| 쿼리 파라미터   | 설명                                                                                                    |
| --------------- | ------------------------------------------------------------------------------------------------------- |
| `user_ids[]`    | 반복 가능. OIDC `sub`를 통해 특정 보안 주체로 필터링.                                                  |
| `period[]`      | 반복 가능. `daily`, `weekly` 또는 `monthly` 행으로 필터링.                                               |
| `sort`          | `spend_desc`는 상위 지출자를 먼저 나열함. 정확히 하나의 `period[]`가 필요함.                            |
| `q`             | OIDC `sub`, 최근 확인된 이메일 및 최근 확인된 표시 이름에 대한 대소문자 구분 없는 부분 문자열 필터.   |
| `limit` / `page`| 페이지 크기(1–1000, 기본값 20) 및 이전 응답의 `next_page`에서 받은 불투명 커서.                         |

<Warning>
  `q=` 및 `user_ids[]=`는 GET 쿼리 문자열로 전달되므로 프런트엔드 프록시나 로드 밸런서가 접근 로그에 이를 캡처합니다. 개인정보 로그 정책이 엄격하다면 해당 파라미터를 로그에서 마스킹하세요.
</Warning>

### `/audit`

지출 한도 변경 이력(누가 어떤 상한선을 변경했는지, 변경 전/후 스냅샷 및 선택적 사유)을 최신순으로 반환합니다. `has_more`는 정확합니다. 이 엔드포인트는 퍼스트 파티 형태 대신 로컬 관리자 API 관례를 따릅니다.

### 페이지네이션

원시 목록은 상호 배타적인 `spl_…` ID인 `after_id` 및 `before_id`로 페이지네이션됩니다. 결과는 생성 순서대로 정렬되며 `has_more`는 탐색 방향을 반영합니다. `/effective`는 `?page=`로 다시 전달되는 불투명한 `next_page` 토큰으로 페이지네이션되며, 지출이 기록되는 동안 페이지가 안정적으로 유지되도록 보안 주체가 오름차순으로 정렬됩니다. 둘 다 `limit`는 1–1000, 기본값 20입니다.

## 데이터 수명주기

게이트웨이는 4개의 지출 관련 테이블을 보유하며 매시간 수행되는 정리가 보존 기간을 적용합니다:

| 테이블             | 내용                                                                          | 보존 정책                                                                                               |
| ------------------ | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `spend`            | 센트 단위의 보안 주체별 현재 기간 누적 카운터                                | [`admin.spend_retention_months`](/docs/en/claude-apps-gateway-config#admin), 기본값 13                      |
| `spend_limits`     | 구성된 상한선                                                                 | API를 통해 삭제될 때까지                                                                                |
| `admin_audit`      | 변경 이력                                                                     | [`admin.audit_retention_days`](/docs/en/claude-apps-gateway-config#admin), 기본값 365                       |
| `principal_emails` | 각 보안 주체의 최근 확인된 이메일, 표시 이름 및 IdP 그룹 (개인정보 포함).     | 마지막 활동 후 [`admin.identity_retention_days`](/docs/en/claude-apps-gateway-config#admin), 기본값 90 |

`identity_retention_days`는 `spend_retention_months`보다 의도적으로 짧게 설정됩니다: 권한이 박탈된 정체성은 새로 고침을 중단하고 정리되는 반면, 연간 보고를 위해 익명 지출 카운터는 보존됩니다.

개발자가 퇴사하면 `DELETE /v1/organizations/spend_limits/{id}`를 통해 사용자별 상한선을 삭제하세요. 지출 및 정체성 행은 위의 보존 기간에 따라 만료 정리됩니다. 퇴사 처리 또는 개인정보 접근/삭제 요청(DSAR)으로 한 사람의 정보를 즉시 삭제하려면 게이트웨이 데이터베이스에 직접 `DELETE FROM principal_emails WHERE principal = '<sub>'`를 실행하세요. 그 사람의 이메일, 이름, 그룹을 보유한 유일한 테이블이 삭제됩니다. `spend` 및 `admin_audit` 행은 가명의 OIDC `sub`만 참조하며 자체 보존 기간에 따라 만료 정리됩니다.

## 관련 항목

* [`admin` 및 `enforcement` 설정](/docs/en/claude-apps-gateway-config#admin): 관리자 API 활성화 및 보존 기간 조정
* [배포 가이드](/docs/en/claude-apps-gateway-deploy#postgres): Postgres 스키마 및 백업 안내
