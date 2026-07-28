> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# Claude Code의 프롬프트 캐싱 활용 방식

> Claude Code는 프롬프트 캐싱을 자동으로 관리합니다. 모델 전환이 왜 느리고 캐시되지 않은 턴을 트리거하는지, `/compact` 비용은 얼마인지, 세션 중간에 CLAUDE.md를 편집해도 왜 바로 적용되지 않는지, 그리고 캐시 적중률(hit rate)을 확인하는 방법을 알아보세요.

프롬프트 캐싱(Prompt caching)은 Claude Code를 더 빠르고 비용 효율적으로 만들어 줍니다. 캐싱이 없으면 API는 매 턴마다 전체 기록을 다시 처리해야 합니다. 캐싱이 있으면 이미 처리한 내용을 재사용하고 변경된 내용에 대해서만 새 작업을 수행합니다.

사용자가 [직접 비활성화](#disable-prompt-caching)하지 않는 한 Claude Code가 프롬프트 캐싱을 대신 처리합니다. 그럼에도 특정 작업이 캐시를 무효화(invalidate)하여 다음 응답을 재구축하는 동안 느리고 비싸게 만든다는 점을 알 수 있으므로 프롬프트 캐싱의 작동 방식을 파악하는 것은 여전히 유용합니다. 이 페이지에서는 이러한 작업이 무엇인지, 일부 설정이 왜 적용을 위해 재시작을 기다리는지, 사용량이 높아 보일 때 캐시 성능을 점검하는 방법에 대해 다룹니다.

## 캐시 구성 방식

Claude Code에서 메시지를 보낼 때마다 새로운 API 요청이 생성됩니다. 모델은 요청과 요청 사이에 아무것도 기억하지 못하므로 Claude Code는 시스템 프롬프트, 프로젝트 컨텍스트, 이전의 모든 메시지 및 도구 결과, 그리고 새로운 메시지를 포함한 전체 컨텍스트를 다시 전송합니다. 새 내용은 끝에 덧붙여지므로 각 요청의 대부분은 이전 요청과 동일합니다. 프롬프트 캐싱은 API가 변경되지 않은 부분을 다시 처리하는 것을 방지하는 기술입니다.

API는 각 요청의 시작 부분(접두사, prefix)을 최근에 처리한 내용과 일치시켜 캐싱합니다. 일반적인 턴에서 접두사는 이전 요청 전체에 해당하며 최신 대화 내용만 새로 추가된 부분입니다. 일치 방식은 완전 일치(exact match)이므로 접두사 임의의 위치에서 변경이 발생하면 그 뒤의 모든 내용을 다시 계산합니다. 파일별 또는 세그먼트별 캐싱은 존재하지 않습니다. 기본적인 메커니즘은 API 참조 문서의 [프롬프트 캐싱 작동 방식](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#how-prompt-caching-works)을 참조하세요.

<img src="https://mintcdn.com/claude-code/VbDJw--l6T9a9Wvm/images/prompt-caching-prefix.svg?fit=max&auto=format&n=VbDJw--l6T9a9Wvm&q=85&s=f2e8f0b8298a50305fe428ca3f1d1594" className="dark:hidden" alt="Four turns shown as growing horizontal bars. Each turn's request contains everything from the previous turn plus the latest exchange appended at the end. On turns two and three, the unchanged prefix is read from cache and only the new exchange is processed. On turn four, the system prompt changed, so the prefix no longer matches and the entire request is reprocessed and written." width="720" height="454" data-path="images/prompt-caching-prefix.svg" />

<img src="https://mintcdn.com/claude-code/VbDJw--l6T9a9Wvm/images/prompt-caching-prefix-dark.svg?fit=max&auto=format&n=VbDJw--l6T9a9Wvm&q=85&s=7434a04e08187edd26ec6c3dd332f624" className="hidden dark:block" alt="Four turns shown as growing horizontal bars. Each turn's request contains everything from the previous turn plus the latest exchange appended at the end. On turns two and three, the unchanged prefix is read from cache and only the new exchange is processed. On turn four, the system prompt changed, so the prefix no longer matches and the entire request is reprocessed and written." width="720" height="454" data-path="images/prompt-caching-prefix-dark.svg" />

접두사 매칭의 이점을 극대화하기 위해 Claude Code는 턴과 턴 사이에 거의 변경되지 않는 내용이 먼저 오도록 각 요청을 정렬합니다:

| 레이어           | 내용                                              | 변경 시점                                                              |
| ---------------- | ------------------------------------------------- | ---------------------------------------------------------------------- |
| 시스템 프롬프트  | 핵심 지침, 도구 정의, 출력 스타일                 | 로드된 도구 정의 세트가 변경되거나, Claude Code가 업그레이드될 때      |
| 프로젝트 컨텍스트| CLAUDE.md, 자동 메모리, 범위 미지정 규칙          | 세션 시작 시, 또는 `/clear`나 `/compact` 수행 후                       |
| 대화 내용        | 사용자 메시지, Claude의 응답, 도구 결과           | 매 턴마다                                                              |

대화 레이어가 변경되더라도 시스템 프롬프트 및 프로젝트 컨텍스트는 캐시된 상태로 유지됩니다. 시스템 프롬프트가 변경되면 이후의 모든 내용이 다른 접두사 뒤에 위치하게 되므로 전체가 무효화됩니다. 세 번째 열은 망라된 전체 목록이라기보다 흔한 원인들을 나열한 것이며, 아래 섹션들에서는 세션 시작 시 고정되는 출력 스타일과 같은 내용을 포함하여 전체적인 원인들을 다룹니다.

접두사 매칭 규칙은 이 페이지에 설명된 대부분의 동작을 설명합니다. 예를 들어 [계획 모드(Plan mode)](/docs/en/permission-modes#analyze-before-you-edit-with-plan-mode)와 [스킬 로딩](/docs/en/skills)은 자신들의 지침을 대화 메시지 형태로 덧붙이므로 캐시된 접두사가 손상되지 않고 유지됩니다.

두 가지 설정은 프롬프트 텍스트의 일부가 전혀 아니어서 레이어 표에 나타나지 않지만, 둘 다 캐시 키의 일부에 해당합니다:

* **모델 (Model)**: 각 모델은 자체 캐시를 가집니다. 모델을 전환하면 내용이 완전히 동일하더라도 전체 요청을 다시 계산합니다. 아래의 [모델 전환하기](#switching-models)를 참조하세요.
* **노력 수준 (Effort level)**: 동일한 모델에 대해서도 각 노력 수준마다 자체 캐시가 있습니다. 세션 중간에 이를 변경하면 전체 요청을 다시 계산하며, Claude Code가 변경을 적용하기 전에 확인을 요청합니다. 아래의 [노력 수준 변경하기](#changing-effort-level)를 참조하세요.

<Tip>
  세션 상단에서 모델과 노력 수준을 선택한 다음 태스크 사이의 자연스러운 중단 시점에 `/compact`를 활용하세요. 태스크 중간에 변경을 적게 할수록 캐시 적중률(hit rate)이 높아집니다.
</Tip>

### 캐시가 위치하는 곳

캐싱은 모델을 서비스하는 인프라의 서버 측에서 일어납니다. 해당 위치가 어디인가는 인증 방식에 따라 달라집니다:

* **API 키, Claude 구독, 또는 [AWS 상의 Claude Platform](/docs/en/claude-platform-on-aws)**: 캐시는 Anthropic 인프라에 존재하며 [Claude API](https://platform.claude.com/docs)를 통해 접근함
* **Amazon Bedrock 또는 Google Cloud's Agent Platform**: 캐시는 클라우드 프로바이더의 서빙 인프라에 존재함
* **Microsoft Foundry**: 배포의 [호스팅 옵션](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry#hosting-options)에 따라 다름. Hosted on Azure 배포는 Azure 인프라에서 서빙되고; Hosted on Anthropic 배포는 Anthropic 인프라에서 서빙됨
* **커스텀 `ANTHROPIC_BASE_URL` 또는 [LLM 게이트웨이](/docs/en/llm-gateway)**: 캐시는 요청이 전달되는 위치에 존재하며, 캐싱 작동 여부는 게이트웨이에 따라 다름

Claude Code가 대화 중간에 덧붙이는 시스템 컨텍스트(파일 변경 알림 등)는 Claude API에서와 동일하게 Amazon Bedrock과 그 [Mantle 엔드포인트](/docs/en/amazon-bedrock#use-the-mantle-endpoint), Google Cloud's Agent Platform, Microsoft Foundry에서도 캐싱됩니다. v2.1.211 이전에는 이들 프로바이더가 해당 덧붙여진 시스템 컨텍스트를 매 요청마다 캐시되지 않은 입력 토큰으로 청구했었습니다.

요청이 [LLM 게이트웨이](/docs/en/llm-gateway)나 커스텀 `ANTHROPIC_BASE_URL`을 통과할 때 Claude Code는 덧붙여진 시스템 컨텍스트를 동일한 방식으로 캐싱 대상으로 표시하며, 캐시 적용 여부는 게이트웨이에 따라 다릅니다. 게이트웨이가 해당 블록에 대한 [캐시 브레이크포인트(cache breakpoint)](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#explicit-cache-breakpoints)를 거부하면 Claude Code가 브레이크포인트 없이 요청을 다시 시도하고 대화의 남은 기간 동안 해당 블록을 캐시되지 않은 상태로 둡니다.

각 프로바이더가 저장하고 처리하는 내용에 대해서는 [데이터 사용량](/docs/en/data-usage)을 참조하세요. 캐시가 어디에 있든 항목은 비활동 기간이 지나면 만료되며, 아래 [캐시 수명](#cache-lifetime)에서 TTL과 이를 연장하는 방법을 다룹니다.

## 캐시를 무효화하는 작업들

다음 작업들은 이후 요청이 캐시의 일부 또는 전체를 놓치게 만듭니다. 새로 접두사가 캐시되기 전까지 일시적으로 더 느리고 비싼 턴을 경험하게 됩니다. 이러한 작업들은 대부분 비용이 발생함을 파악하면 태스크 중간에 회피할 수 있습니다. 모델 전환은 그 뒤에 따르는 느린 턴을 인지하기 전까지는 비용이 들지 않는 것처럼 느껴질 수 있습니다.

* [모델 전환하기](#switching-models)
* [노력 수준 변경하기](#changing-effort-level)
* [빠른 모드 켜기](#turning-on-fast-mode)
* [MCP 서버 연결 또는 연결 해제](#connecting-or-disconnecting-an-mcp-server)
* [플러그인 활성화 또는 비활성화](#enabling-or-disabling-a-plugin)
* [도구 전체 거부하기](#denying-an-entire-tool)
* [대화 내용 압축하기](#compacting-the-conversation)
* [Claude Code 업그레이드](#upgrading-claude-code)

### 모델 전환하기

각 모델은 자체 캐시를 가집니다. [`/model`](/docs/en/model-config#setting-your-model)로 전환하면 내용이 완전히 동일하더라도 다음 요청이 캐시 적중 없이 전체 대화 기록을 읽게 됩니다.

[`opusplan` 모델 설정](/docs/en/model-config#opusplan-model-setting)은 계획 모드 동안 Opus로, 실행 동안 Sonnet으로 해석되므로 각 계획 모드 토글은 모델 전환에 해당하며 새로운 캐시를 시작합니다.

Fable 5에서의 [자동 모델 폴백](/docs/en/model-config#automatic-model-fallback) 역시 모델 전환에 해당합니다. 안전 분류기가 요청에 플래그를 지정하면 Claude Code가 기본 Opus 모델에서 이를 다시 구동하고 세션이 거기서 계속됩니다.

### 노력 수준 변경하기

캐시는 모델뿐만 아니라 [노력 수준](/docs/en/model-config#adjust-effort-level)에 의해서도 키가 지정되므로 `/effort`로 전환하면 다음 요청이 캐시 적중 없이 전체 대화 기록을 읽게 됩니다. 대화가 일단 시작된 후 Claude Code는 캐시를 무효화하는 노력 변경을 적용하기 전에 확인 대화 상자를 보여줍니다. 모델의 기본값을 명시적으로 설정하는 등 이미 적용 중인 수준과 동일한 수준으로 해석되는 변경은 대화 상자를 건너뛰고 캐시를 유지합니다.

### 빠른 모드 켜기

[빠른 모드](/docs/en/fast-mode)를 활성화하면 캐시 키의 일부인 요청 헤더가 추가되므로 다음 요청이 캐시 적중 없이 전체 대화 기록을 읽게 됩니다. 캐시되지 않은 해당 입력 토큰들은 [빠른 모드 요율](/docs/en/fast-mode#understand-the-cost-tradeoff)로 청구되므로 세션 시작 시 켜는 것이 긴 세션 깊숙한 곳에서 켜는 것보다 비용이 적게 듭니다. 비 Opus 모델에서 빠른 모드를 활성화하는 것은 [모델 전환](#switching-models)도 수반하므로 자체적으로 새 캐시를 시작하게 됩니다.

이 비용은 대화당 한 번 적용됩니다. 첫 빠른 모드 턴 이후 Claude Code는 헤더를 계속 전송하며 캐시 키의 일부가 아닌 요청의 속도 설정만을 다르게 적용합니다. 빠른 모드를 끄거나, 비율 제한 후 [표준 속도로 자동 폴백](/docs/en/fast-mode#handle-rate-limits)되거나, 나중에 다시 켜는 것은 모두 캐시를 유지합니다. `/clear` 및 `/compact`는 어차피 해당 시점에 캐시를 재구축하므로 이를 리셋합니다.

### MCP 서버 연결 또는 연결 해제

도구 정의는 시스템 프롬프트 레이어에 위치하므로 요청 내의 도구 정의 세트가 턴 사이에 변경되면 캐시가 무효화됩니다. [어드바이저 도구](/docs/en/advisor) 토글은 예외입니다: 그 정의는 캐시 브레이크포인트 뒤에 위치하므로 `/advisor`를 활성화하거나 비활성화해도 캐시된 접두사가 손상되지 않고 유지됩니다. [MCP 서버](/docs/en/mcp) 변경이 캐시를 무효화하는지 여부는 해당 서버의 도구들이 [도구 검색](/docs/en/mcp#scale-with-mcp-tool-search)에 의해 지연되는지 아니면 접두사 안으로 로드되는지에 따라 다릅니다:

* **지연된 도구 (Deferred tools)**: 지원되는 모델에서의 기본값으로, 서버 연결, 연결 해제, 도구 목록 변경 시 새로운 내용만 덧붙여지며 이미 캐시된 내용을 방해하지 않습니다.
* **접두사 안으로 로드된 도구 (Tools loaded into the prefix)**: 변경 시 캐시가 무효화됩니다. 이는 Google Cloud's Agent Platform이나 커스텀 `ANTHROPIC_BASE_URL` 게이트웨이처럼 [도구 검색을 이용할 수 없거나 비활성화되어 있을 때](/docs/en/mcp#configure-tool-search) 발생합니다. 또한 [`alwaysLoad`](/docs/en/mcp#exempt-a-server-from-deferral)로 표시된 서버나 도구, [임계값 기반 로딩](/docs/en/mcp#configure-tool-search)으로 처음에 유지되는 정의에 대해서도 발생합니다.

도구가 접두사로 로드될 때 무효화의 가장 흔한 원인은 세션 중간에 서버가 연결되거나 연결 해제되는 것이며, 이는 사용자의 아무런 동작 없이도 발생할 수 있습니다: stdio 서버의 프로세스 종료, HTTP 세션 만료, [일시적 오류 후 서버의 자동 재연결](/docs/en/mcp#automatic-reconnection). 연결된 서버가 도구 목록을 변경하는 [동적 도구 업데이트](/docs/en/mcp#dynamic-tool-updates)를 푸시할 수도 있습니다.

MCP 구성을 편집하는 것 자체는 캐시를 변경하지 않습니다. 새 구성은 재시작 후에만 적용되며, 재시작 시점이 서버가 연결되거나 연결 해제되는 시점입니다.

### 플러그인 활성화 또는 비활성화

[플러그인](/docs/en/plugins)은 여러 구성 요소 유형을 번들로 다루며, 변경 비용은 플러그인이 제공하는 구성 요소에 따라 다릅니다. 스킬, 명령, 에이전트, 훅, LSP 서버, 모니터, 테마는 캐시를 절대 무효화하지 않습니다: 이들이 요청에 추가하는 모든 내용은 기존 대화 뒤에 덧붙여지므로 다음 요청이 새 내용에 대한 비용을 지불하지만 이전의 모든 내용은 캐시에서 읽어옵니다.

[MCP 서버](/docs/en/plugins-reference#mcp-servers)를 제공하는 플러그인은 예외입니다. 활성화 또는 비활성화는 [MCP 서버 연결 또는 연결 해제](#connecting-or-disconnecting-an-mcp-server)와 동일한 규칙을 따릅니다: 서버 도구가 지연될 때 캐시가 유지되고, 접두사 안으로 로드될 때 다음 요청이 전체 대화를 다시 읽습니다.

플러그인 변경 사항은 [`/reload-plugins`](/docs/en/discover-plugins#apply-plugin-changes-without-restarting)를 실행하거나 새 세션을 시작할 때 적용됩니다. 덧붙여진 공지이든 전체 다시 읽기이든 비용은 `/plugin install`, `/plugin enable`, `/plugin disable`을 실행할 때가 아니라 새로고침 후 첫 턴에 나타납니다. v2.1.163부터 새로고침이 전체 다시 읽기를 트리거하게 될 때 `/reload-plugins`가 경고를 출력하고 새로고침을 적용하지 않습니다. 강제로 적용하려면 `--force`를 전달하세요.

세션 초반에 활성화했던 플러그인을 비활성화하면 이전 요청 형태가 복원됩니다. 해당 접두사가 여전히 [캐시 수명](#cache-lifetime) 이내에 있다면 다음 요청이 다시 작성하는 대신 이전 캐시 항목을 읽어옵니다.

### 도구 전체 거부하기

`Bash`나 `WebFetch`와 같은 순수 도구 이름을 [거부 규칙(deny rule)](/docs/en/permissions#manage-permissions)으로 추가하면 Claude의 컨텍스트에서 해당 도구가 완전히 제거됩니다. 내장 도구 정의는 시스템 프롬프트 레이어에 로드되므로 세션 중간에 이러한 규칙을 추가하거나 제거하면 캐시가 무효화됩니다. 변경 사항은 `/permissions`를 통해 추가하든 [설정 파일을 직접 편집](/docs/en/settings#when-edits-take-effect)하든 다음 턴에 적용됩니다.

도구 이름 위치에서 매칭되는 거부 규칙만 이러한 효과를 가집니다: 순수 도구 이름, 동등한 `Bash(*)` 형태, 또는 `"mcp__*"`와 같은 [도구 이름 글로브](/docs/en/permissions#tool-name-wildcards). `"mcp__*"`처럼 MCP 도구와만 매칭되는 글로브는 동일한 방식으로 해당 도구들을 제거하지만 매칭된 도구들이 [지연된 상태](#connecting-or-disconnecting-an-mcp-server)(기본값)일 때는 지연된 정의가 캐시된 접두사에 전혀 들어있지 않았으므로 캐시를 손상시키지 않고 유지를 해줍니다. `Bash(rm *)`와 같은 범위 지정 거부 규칙 및 모든 허용/질문 규칙은 Claude가 보는 도구를 변경하지 않습니다. Claude Code는 Claude가 호출을 시도할 때 이들을 검사하므로 접두사가 손상되지 않고 유지됩니다.

### 대화 내용 압축하기

[압축(Compaction)](/docs/en/context-window#what-survives-compaction)은 메시지 기록을 요약으로 대체합니다. 다음 요청이 이전 기록과 접두사를 공유하지 않는 새롭고 짧은 기록을 가지게 되므로 설계상 대화 레이어가 무효화됩니다. Claude Code는 시스템 프롬프트 레이어를 재사용하고 디스크에서 프로젝트 컨텍스트를 다시 로드하는데, 이는 세션 시작 후 CLAUDE.md와 메모리가 변경되지 않은 경우에만 캐시 적중을 보장합니다.

요약을 생성하기 위해 Claude Code는 대화와 동일한 시스템 프롬프트, 도구, 기록에 최종 사용자 메시지로 요약 지침을 덧붙인 일회성 요청을 보냅니다. 접두사를 공유하므로 해당 요청은 전체 기록을 다시 처리하는 대신 기존 캐시를 읽습니다. 압축 시간의 대부분은 캐시 미스가 아닌 요약 생성에 소요됩니다. 뒤따르는 턴은 훨씬 짧은 요약에 대해서만 대화 캐시를 다시 구축하므로 압축 후 턴은 느린 부분이 아닙니다.

<Tip>
  버리는 컨텍스트가 더 이상 필요하지 않은 내용일 때 압축은 사용자에게 유리하게 작동합니다. 오버헤드가 발생하는 시점을 선택하려면 태스크 중간에 자동 압축이 트리거되기를 기다리는 대신 태스크 사이와 같은 작업의 자연스러운 중단 시점에 `/compact`를 실행하세요. abandoned 하고 싶은 경로로 접어들었다면 대신 이전 턴으로 [`/rewind`](#rewinding-the-conversation) 하세요. 되돌리기는 압축처럼 새 접두사를 구축하는 대신 이미 캐시된 접두사로 잘라냅니다.
```
</Tip>

### Claude Code 업그레이드

새 Claude Code 버전은 일반적으로 시스템 프롬프트나 도구 정의를 업데이트하므로 업그레이드 후 첫 요청은 처음부터 캐시를 다시 구축합니다. [자동 업데이트](/docs/en/setup#auto-updates)는 백그라운드에서 새 버전을 다운로드하지만 세션 중간에는 절대 적용하지 않고 다음 실행 시 적용하므로, 세션 도중의 뜻밖의 상황보다는 재시작 후 캐시되지 않은 첫 턴으로 경험하게 됩니다. 업그레이드가 적용되는 시점을 제어하려면 `DISABLE_AUTOUPDATER=1`을 설정하세요.

<Note>
  업그레이드 후 [세션을 다시 시작(resume)](/docs/en/sessions#resume-a-session)하면 대화 기록이 이제 다른 시스템 프롬프트 뒤에 위치하므로 캐시 적중 없이 전체 대화 기록을 다시 처리합니다. 비용은 재개된 대화가 얼마나 긴가에 비례하므로 긴 세션으로 다시 돌아가는 첫 턴이 사용자가 전송하는 가장 비싼 요청이 될 수 있습니다.
</Note>

## 캐시를 유지하는 작업들

다음 작업들은 대화 끝에 덧붙여지거나 요청에 전혀 손을 대지 않습니다. CLAUDE.md 편집이나 출력 스타일 변경처럼 일부 작업은 설정 변경이 적용되기 위해 재시작을 기다리는 이유이기도 합니다.

* [저장소의 파일 편집하기](#editing-files-in-your-repository)
* [세션 중간에 CLAUDE.md 편집하기](#editing-claude-md-mid-session)
* [출력 스타일 변경하기](#changing-output-style)
* [권한 모드 변경하기](#changing-permission-mode)
* [스킬 및 명령 호출하기](#invoking-skills-and-commands)
* [`/recap` 실행하기](#running-%2Frecap)
* [대화 내용 되돌리기 (rewind)](#rewinding-the-conversation)
* [Subagent 생성하기](#subagents-and-the-cache)

### 저장소의 파일 편집하기

파일 내용은 Claude가 이를 읽을 때만 컨텍스트에 들어오며, 읽기 동작은 대화 끝에 덧붙여집니다. Claude가 이전에 읽은 파일을 편집하더라도 기록에 있던 이전 읽기가 소급하여 변경되지는 않습니다. 대신 Claude Code가 파일이 변경되었음을 나타내는 `<system-reminder>`를 덧붙이고, Claude가 필요시 이를 다시 읽습니다.

### 세션 중간에 CLAUDE.md 편집하기

프로젝트 루트 및 사용자 수준 CLAUDE.md 파일은 세션 시작 시 한 번 읽혀 메모리에 유지됩니다. 세션 중간에 이들을 편집해도 캐시가 무효화되지 않지만 편집 내용도 적용되지 않습니다. Claude는 세션 시작 시 로드된 버전으로 계속 작업합니다. 새 내용은 다음 `/clear`, `/compact`, 또는 재시작 시 로드됩니다.

[하위 디렉토리의 중첩 CLAUDE.md 파일](/docs/en/memory) 및 [`paths:` 프론트매터가 있는 규칙](/docs/en/memory#path-specific-rules)은 Claude가 일치하는 파일을 처음 읽을 때 나중에 로드됩니다. 로드되기 전에 편집하는 것은 적용됩니다. 로드된 후 해당 내용은 대화 기록의 일부가 되므로 세션 중간 편집이 이를 소급 변경하지는 못합니다.

### 출력 스타일 변경하기

[출력 스타일](/docs/en/output-styles)은 시스템 프롬프트의 일부로, Claude Code가 세션 시작 시 한 번 읽습니다. 세션 중간에 `/config`나 `outputStyle` 설정을 통해 변경해도 캐시가 무효화되지 않지만 변경 내용 역시 적용되지 않습니다. Claude는 세션 시작 시 로드된 스타일을 계속 사용합니다. 새 스타일은 다음 `/clear` 또는 재시작 시 로드됩니다.

### 권한 모드 변경하기

기본 모드에서 accept edits 모드로 바꾸는 것처럼 [권한 모드](/docs/en/permission-modes) 간을 전환하는 것은 시스템 프롬프트나 도구 정의를 변경하지 않으므로 캐시에 안전합니다. [`opusplan`](/docs/en/model-config#opusplan-model-setting) 모델 설정을 사용한 계획 모드는 예외로, 계획 모드에 진입하거나 이탈할 때 모델을 Opus와 Sonnet 간에 전환합니다. 이는 모드 토글을 [모델 전환](#switching-models)으로 만듭니다.

### 스킬 및 명령 호출하기

[스킬](/docs/en/skills)과 [명령](/docs/en/commands)은 호출 시점에 자신의 지침을 사용자 메시지로 주입합니다. 이전 대화의 어떤 것도 변경되지 않습니다.

### `/recap` 실행하기

[`/recap`](/docs/en/interactive-mode#session-recap)은 터미널 표시용 요약을 생성합니다. `/compact`와 달리 메시지 기록을 대체하는 것이 아니라 요약을 명령 출력으로 덧붙이므로 캐시된 접두사가 손상되지 않고 유지됩니다.

### 대화 내용 되돌리기 (rewind)

[`/rewind`](/docs/en/checkpointing)는 대화 내용을 이전 턴으로 되돌려 잘라냅니다. 남은 기록은 해당 시점에 캐시가 구축되었던 동일한 내용이고 시스템 프롬프트 및 프로젝트 컨텍스트 레이어가 변경되지 않았으므로 다음 요청이 이전 캐시 항목에 적중하게 됩니다. 그 이후의 모든 턴이 해당 접두사를 통과해 읽어왔으므로 원래 턴이 TTL보다 오래전이었더라도 해당 항목이 따뜻하게(warm) 유지되었습니다.

대화와 함께 파일 체크포인트를 복원하는 것은 캐시에 별도 영향이 없습니다. 파일 내용은 [저장소의 파일 편집하기](#editing-files-in-your-repository)와 동일하게 Claude가 이를 읽을 때만 컨텍스트에 들어옵니다.

## 캐시 수명 (Cache lifetime)

캐시된 접두사는 비활동 기간이 지나면 만료됩니다. 캐시에 적중하는 각 요청은 타이머를 리셋하므로 작업을 계속하는 동안에는 캐시가 따뜻하게 유지됩니다. 충분히 긴 공백이 지나면 다음 요청이 전체 입력을 다시 계산하고 캐시를 재확립하며, 이것이 한참 자리를 비웠다가 돌아온 후의 첫 턴이 눈에 띄게 느릴 수 있는 이유입니다.

TTL(Time to live)은 캐시가 유지되는 공백 기간을 제어합니다. API는 두 가지를 제공합니다: 5분 TTL, 그리고 긴 휴식에도 캐시를 따뜻하게 유지해주지만 [캐시 쓰기를 더 높은 요율로 청구](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#pricing)하는 [1시간 TTL](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#1-hour-cache-duration). Claude Code는 인증 방식에 따라 TTL을 자동으로 선택하며 환경 변수로 이를 재정의할 수 있습니다.

### Claude 구독 사용 시

Claude 구독 상에서 Claude Code는 1시간 TTL을 자동으로 요청합니다. 사용량이 토큰당 청구가 아닌 요금제에 포함되어 있으므로 더 긴 TTL이 추가 비용을 발생시키지 않고 캐시가 따뜻하게 유지되는 기간에만 영향을 줍니다.

요금제의 사용량 제한을 초과하여 Claude Code가 [사용량 크레딧](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans)을 끌어다 쓰고 있다면 해당 사용량에 대해 요금이 청구되므로 Claude Code가 TTL을 5분으로 자동 낮춥니다.

### API 키 또는 서드파티 프로바이더 사용 시

API 키, Amazon Bedrock, Google Cloud's Agent Platform, Microsoft Foundry, 또는 AWS 상의 Claude Platform에서는 토큰당 요율을 지불하므로 기본적으로 더 저렴한 5분 TTL이 유지됩니다. [1시간 TTL](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#1-hour-cache-duration)을 옵트인하려면 `ENABLE_PROMPT_CACHING_1H=1`을 설정하세요.

Amazon Bedrock에서는 프롬프트 캐싱 지원 여부, 캐시 가능한 최소 접두사 길이, 1시간 TTL 수용 여부가 모델별로 다릅니다. 캐시 토큰 수가 0으로 계속 유지된다면 Amazon Bedrock 문서의 [지원되는 모델, 리전, 제한](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html#prompt-caching-models)을 확인하세요.

### TTL 재정의하기

인증 방식에 상관없이 5분 TTL을 강제하려면 `FORCE_PROMPT_CACHING_5M=1`을 설정하세요. 이는 캐시 동작을 디버깅하거나, 두 TTL을 비교하거나, [관리형 설정](/docs/en/settings#settings-files)에 설정된 `ENABLE_PROMPT_CACHING_1H`를 재정의할 때 유용합니다.

## 캐시 스코프

Claude Code에서 캐시는 사실상 하나의 머신 및 디렉토리에 국한(scoped)됩니다. 시스템 프롬프트가 작업 디렉토리, 플랫폼, 셸, OS 버전, 자동 메모리 경로를 내포하므로 서로 다른 디렉토리의 두 세션은 다른 접두사를 구축하고 서로의 캐시를 미스하게 됩니다. 각 worktree가 자체 작업 디렉토리를 가지므로 동일한 저장소의 worktree 간에도 이것이 포함됩니다.

동일한 디렉토리에서 병렬로 구동하는 세션들은 일치하는 접두사를 구축하고 서로의 캐시를 읽어옵니다. 순차적 세션은 시작 시의 git 상태 스냅샷이 일치할 때만 접두사를 공유하는데, 시스템 프롬프트가 브랜치 및 최근 커밋도 포착하기 때문입니다.

기반 API 캐시는 더 넓습니다. 조직 간에 캐시가 격리되며 일부 프로바이더에서는 [조직 내의 워크스페이스 간에도 격리](https://platform.claude.com/docs/en/build-with-claude/prompt-caching#cache-storage-and-sharing)됩니다. 해당 경계 내부에서는 동일한 모델과 접두사를 가진 임의의 두 요청이 동일한 캐시를 읽어옵니다. 자동화 프로세스 플릿을 구동하는 Agent SDK 호출자의 경우, 시스템 프롬프트의 머신별 섹션을 억제하고 머신 간에 캐시를 공유하려면 [사용자 및 머신 간 프롬프트 캐싱 향상](/docs/en/agent-sdk/modifying-system-prompts#improve-prompt-caching-across-users-and-machines)을 참조하세요.

## 캐시 성능 점검

캐시 성능은 API가 매 응답마다 보고하는 두 토큰 수치로 나타납니다. 이를 라이브로 관찰하는 가장 직접적인 방법은 `current_usage` 객체를 읽는 [statusline 스크립트](/docs/en/statusline)입니다:

| 필드                         | 의미                                                                                         |
| ----------------------------- | --------------------------------------------------------------------------------------------- |
| `cache_creation_input_tokens` | 이번 턴에 캐시에 작성된 토큰으로, 캐시 쓰기 요율로 청구됨                                     |
| `cache_read_input_tokens`     | 이번 턴에 캐시로부터 서빙된 토큰으로, 표준 입력 요율의 약 10% 수준으로 청구됨                |

생성(creation) 대비 읽기(read) 비율이 높다는 것은 캐싱이 잘 작동하고 있음을 의미합니다. 턴을 거듭해도 생성 수치가 높게 유지된다면 접두사에서 무언가가 계속 변경되고 있는 것입니다. [캐시를 무효화하는 작업들](#actions-that-invalidate-the-cache) 섹션에 일반적인 원인들이 나열되어 있습니다.

조직 전체에 대한 가시성을 위해 OpenTelemetry 내보내기 도구(exporter)가 사용자 및 세션별 캐시 읽기 및 생성 토큰을 보고합니다. 메트릭 및 이벤트 속성 참조는 [사용량 모니터링](/docs/en/monitoring-usage)을 참조하세요.

## Subagents 및 캐시

[subagent](/docs/en/sub-agents)는 부모와 별개로 자체 시스템 프롬프트 및 도구 세트를 갖춘 자체 대화를 시작합니다. 첫 호출 시 캐시 적중 없이 시작하여 자체 턴을 거듭하면서 따뜻해지는 자체 캐시를 구축합니다. 자동 1시간 TTL이 메인 대화에 적용되므로 subagent는 구독 상에서도 5분 TTL을 사용합니다.

부모의 캐시는 영향을 받지 않습니다. 부모 측면에서 subagent의 호출과 결과는 대화 끝에 덧붙여지므로 부모의 접두사가 손상되지 않고 유지됩니다.

대조적으로 [포크 (fork)](/docs/en/sub-agents#fork-the-current-conversation)는 부모의 시스템 프롬프트, 도구, 대화 기록을 정확히 상속받으므로 첫 요청이 부모의 캐시를 읽습니다. [대화 내용 압축하기](#compacting-the-conversation)에 설명된 압축 요약 호출도 동일한 접두사 공유 접근 방식을 사용합니다.

## 프롬프트 캐싱 비활성화

특정 모델이나 프로바이더에서 캐싱 동작을 디버깅할 때 캐싱을 비활성화하는 것이 간혹 유용할 수 있습니다. 끄려면 다음 환경 변수 중 하나를 `1`로 설정하세요:

| 환경 변수                       | 효과                    |
| ------------------------------- | ----------------------- |
| `DISABLE_PROMPT_CACHING`        | 모든 모델에 대해 비활성화 |
| `DISABLE_PROMPT_CACHING_HAIKU`  | Haiku에 대해서만 비활성화 |
| `DISABLE_PROMPT_CACHING_SONNET` | Sonnet에 대해서만 비활성화 |
| `DISABLE_PROMPT_CACHING_OPUS`   | Opus에 대해서만 비활성화 |
| `DISABLE_PROMPT_CACHING_FABLE`  | Fable에 대해서만 비활성화 |

조직 전체에 걸쳐 캐싱 정책을 설정하려면 [관리형 설정](/docs/en/settings#settings-files)의 `env` 블록에 이들 변수나 [TTL 변수](#cache-lifetime)를 위치시키세요. 일반적인 용도의 경우 캐싱을 활성화 상태로 두세요.

## 관련 리소스

* [Claude Code 구축의 교훈: 프롬프트 캐싱이 전부다](https://claude.com/blog/lessons-from-building-claude-code-prompt-caching-is-everything): 계획 모드, 지연 도구 로딩, 압축에 대한 설계 근거
* [컨텍스트 창 둘러보기](/docs/en/context-window): 언제 무엇이 컨텍스트로 로드되는지
* [토큰 사용량 줄이기](/docs/en/costs#reduce-token-usage): 컨텍스트 크기 관리를 위한 캐싱 이외의 전략들
* [비용 추적 및 절감](/docs/en/agent-sdk/cost-tracking): Agent SDK 호출자를 위한 캐시 토큰 추적 및 TTL 구성
* [프롬프트 캐싱](https://platform.claude.com/docs/en/build-with-claude/prompt-caching): 기반 API 메커니즘, 브레이크포인트, 가격 정책
