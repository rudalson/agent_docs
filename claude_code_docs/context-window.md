> ## 문서 색인
> 전체 문서 색인은 다음 주소에서 확인하세요: https://code.claude.com/docs/llms.txt
> 더 찾아보기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 컨텍스트 창 탐색하기

> 세션 동안 Claude Code의 컨텍스트 창이 채워지는 방식을 보여주는 대화형 시뮬레이션입니다. 자동으로 로드되는 내용, 각 파일 읽기의 비용, 규칙 및 훅이 실행되는 시점을 확인하세요.

export const ContextWindow = () => {
  const MAX = 200000;
  const STARTUP_END = 0.2;
  {}
  const EVENTS = useMemo(() => [{}, {
    t: 0.015,
    kind: 'auto',
    label: '시스템 프롬프트',
    tokens: 4200,
    color: '#6B6964',
    vis: 'hidden',
    desc: '동작, 도구 사용 및 응답 형식 지정을 위한 핵심 지침입니다. 항상 가장 먼저 로드되며 터미널에는 보이지 않습니다.',
    link: null
  }, {
    t: 0.035,
    kind: 'auto',
    label: '자동 메모리 (MEMORY.md)',
    tokens: 680,
    color: '#E8A45C',
    vis: 'hidden',
    desc: '이전 세션에서 학습한 빌드 명령, 탐지한 패턴, 피해야 할 실수 등 Claude 스스로 남긴 메모입니다. 첫 200줄 또는 25KB 중 먼저 도달하는 분량이 대화 컨텍스트로 로드됩니다.',
    link: '/en/memory#auto-memory'
  }, {
    t: 0.06,
    kind: 'auto',
    label: '환경 정보',
    tokens: 280,
    color: '#6B6964',
    vis: 'hidden',
    desc: '작업 디렉토리, 플랫폼, 셸, OS 버전 및 git 리포지토리 여부입니다. Git 브랜치, 상태 및 최근 커밋은 시스템 프롬프트의 맨 끝에 별도 블록으로 로드됩니다.',
    link: null
  }, {
    t: 0.08,
    kind: 'auto',
    label: 'MCP 도구 (지연 로드)',
    tokens: 120,
    color: '#9B7BC4',
    vis: 'hidden',
    desc: 'Claude가 사용 가능한 도구를 알 수 있도록 나열된 MCP 도구 이름입니다. 기본적으로 전체 스키마는 지연 유지되며, 작업에 필요할 때 도구 검색을 통해 특정 스키마를 온디맨드로 로드합니다. 스키마가 컨텍스트 창의 10% 이내에 들어올 때 미리 로드하려면 `ENABLE_TOOL_SEARCH=auto`, 모두 로드하려면 `ENABLE_TOOL_SEARCH=false`를 설정하세요.',
    link: '/en/mcp#scale-with-mcp-tool-search'
  }, {
    t: 0.1,
    kind: 'auto',
    label: '스킬 설명 목록',
    tokens: 450,
    color: '#D4A843',
    vis: 'hidden',
    noSurviveCompact: true,
    desc: 'Claude가 호출할 수 있는 항목을 알 수 있도록 사용 가능한 스킬의 한 줄 설명 목록입니다. 전체 스킬 내용은 Claude가 실제로 스킬을 사용할 때만 로드됩니다. `disable-model-invocation: true`가 설정된 스킬은 이 목록에 포함되지 않으며 `/name`으로 호출할 때까지 컨텍스트 외부에 완전히 남아 있습니다. 다른 시작 콘텐츠와 달리 이 목록은 `/compact` 후 다시 주입되지 않습니다.',
    link: '/en/skills'
  }, {
    t: 0.12,
    kind: 'auto',
    label: '~/.claude/CLAUDE.md',
    tokens: 320,
    color: '#6A9BCC',
    vis: 'hidden',
    desc: '전역 환경설정입니다. 모든 프로젝트에 적용됩니다. 모든 대화가 시작될 때 프로젝트 지침과 함께 로드됩니다.',
    link: '/en/memory#choose-where-to-put-claude-md-files'
  }, {
    t: 0.14,
    kind: 'auto',
    label: '프로젝트 CLAUDE.md',
    tokens: 1800,
    color: '#6A9BCC',
    vis: 'hidden',
    desc: '프로젝트 규칙, 빌드 명령, 아키텍처 메모입니다. 생성할 수 있는 가장 중요한 파일입니다. 프로젝트 루트에 위치하므로 전체 팀이 동일한 지침을 공유합니다.',
    tip: '200줄 이하로 유지하세요. 참조 콘텐츠는 필요할 때만 로드되도록 스킬이나 경로 한정 규칙으로 이동하세요.',
    link: '/en/memory'
  }, {}, {
    t: 0.22,
    kind: 'user',
    label: '사용자 프롬프트',
    tokens: 45,
    color: '#558A42',
    vis: 'full',
    desc: '"Fix the auth bug where users get 401 after token refresh"',
    link: null
  }, {}, {
    t: 0.28,
    kind: 'claude',
    label: 'src/api/auth.ts 읽기',
    tokens: 2400,
    color: '#8A8880',
    vis: 'brief',
    desc: '메인 인증 파일입니다. 터미널에는 "Read auth.ts"만 보이지만, 2,400 토큰의 파일 내용은 Claude에게만 전달됩니다.',
    tip: '파일 읽기가 컨텍스트 사용량의 대부분을 차지합니다. Claude가 파일 읽기를 줄일 수 있도록 프롬프트에 구체적으로 작성하세요. 조사가 많은 작업에는 하위 에이전트를 사용하세요.',
    link: null
  }, {
    t: 0.32,
    kind: 'claude',
    label: 'src/lib/tokens.ts 읽기',
    tokens: 1100,
    color: '#8A8880',
    vis: 'brief',
    desc: '토큰 모듈로 임포트를 추적하여 읽습니다. 터미널에는 한 줄로 표시됩니다.',
    link: null
  }, {
    t: 0.35,
    kind: 'auto',
    label: '규칙: api-conventions.md',
    tokens: 380,
    color: '#4A9B8E',
    vis: 'brief',
    desc: '`.claude/rules/`의 이 규칙은 `src/api/**`와 일치하는 `paths:` 패턴을 가지고 있습니다. Claude가 해당 디렉토리의 파일을 읽을 때 자동으로 로드되었습니다. 터미널에는 "Loaded .claude/rules/api-conventions.md"로 표시되며 규칙 내용은 표시되지 않습니다.',
    link: '/en/memory#path-specific-rules'
  }, {
    t: 0.38,
    kind: 'claude',
    label: 'middleware.ts 읽기',
    tokens: 1800,
    color: '#8A8880',
    vis: 'brief',
    desc: '인증 흐름을 더 깊이 추적합니다.',
    link: null
  }, {
    t: 0.41,
    kind: 'claude',
    label: 'auth.test.ts 읽기',
    tokens: 1600,
    color: '#8A8880',
    vis: 'brief',
    desc: '예상되는 동작에 대해 기존 테스트를 확인합니다.',
    link: null
  }, {
    t: 0.44,
    kind: 'auto',
    label: '규칙: testing.md',
    tokens: 290,
    color: '#4A9B8E',
    vis: 'brief',
    desc: '`*.test.ts` 파일과 일치하는 또 다른 경로 지정 규칙입니다. Claude가 auth.test.ts를 읽을 때 트리거되었습니다.',
    link: '/en/memory#path-specific-rules'
  }, {
    t: 0.47,
    kind: 'claude',
    label: 'grep "refreshToken"',
    tokens: 600,
    color: '#A09E96',
    vis: 'brief',
    desc: '코드베이스 전반의 검색 결과입니다. 전체 출력이 아닌 명령 실행 여부만 터미널에 표시됩니다.',
    link: null
  }, {}, {
    t: 0.53,
    kind: 'claude',
    label: 'Claude의 분석',
    tokens: 800,
    color: '#D97757',
    vis: 'full',
    desc: '버그 원인 설명: 로테이션에서 토큰이 너무 일찍 무효화됨. 이 텍스트는 터미널에 표시됩니다.',
    link: null
  }, {
    t: 0.57,
    kind: 'claude',
    label: 'auth.ts 편집',
    tokens: 400,
    color: '#D97757',
    vis: 'full',
    desc: '토큰 로테이션 순서를 수정합니다. diff가 터미널에 표시됩니다.',
    link: null
  }, {
    t: 0.59,
    kind: 'hook',
    label: '훅: prettier',
    tokens: 120,
    color: '#B8860B',
    vis: 'hidden',
    desc: '`settings.json`의 PostToolUse 훅이 파일 편집마다 prettier를 실행하고 `hookSpecificOutput.additionalContext`를 통해 보고합니다. 해당 필드는 Claude 컨텍스트에 들어가지만 종료 코드 0의 일반 stdout은 들어가지 않으며 디버그 로그에만 기록됩니다.',
    tip: 'Claude에게 정보를 보내려면 `additionalContext`를 포함한 JSON을 출력하세요. 요약되거나 잘리지 않고 컨텍스트에 포함되므로 간결하게 유지하세요.',
    link: '/en/hooks-guide'
  }, {
    t: 0.62,
    kind: 'claude',
    label: 'auth.test.ts 편집',
    tokens: 600,
    color: '#D97757',
    vis: 'full',
    desc: '수정 사항에 대한 리그레션 테스트를 추가합니다. diff가 터미널에 표시됩니다.',
    link: null
  }, {
    t: 0.64,
    kind: 'hook',
    label: '훅: prettier',
    tokens: 100,
    color: '#B8860B',
    vis: 'hidden',
    desc: '테스트 파일에 대해 동일한 훅이 다시 실행됩니다.',
    link: '/en/hooks-guide'
  }, {
    t: 0.67,
    kind: 'claude',
    label: 'npm test 출력',
    tokens: 1200,
    color: '#A09E96',
    vis: 'brief',
    desc: '테스트 수트를 실행합니다. 전체 1,200 토큰 출력이 아닌 "Running npm test..." 및 통과 수만 보입니다.',
    link: null
  }, {
    t: 0.70,
    kind: 'claude',
    label: '요약',
    tokens: 400,
    color: '#D97757',
    vis: 'full',
    desc: '"Fixed token rotation. Added regression test. All tests pass."',
    link: null
  }, {}, {
    t: 0.72,
    kind: 'user',
    label: '추가 요청',
    tokens: 40,
    color: '#558A42',
    vis: 'full',
    desc: '"Use a subagent to research session timeout handling, then fix it"',
    tip: '추가 질문은 동일한 컨텍스트에 누적됩니다. 조사를 하위 에이전트에 위임하면 큰 파일 읽기가 메인 창에 들어오지 않습니다.',
    link: null
  }, {
    t: 0.79,
    kind: 'claude',
    label: '조사 하위 에이전트 생성',
    tokens: 80,
    color: '#D97757',
    vis: 'brief',
    desc: 'Claude는 신선하고 별도의 컨텍스트 창을 가진 하위 에이전트에 조사를 위임합니다. CLAUDE.md 및 동일한 MCP/스킬을 로드하지만 메인 대화 기록이나 자동 메모리 없이 시작합니다.',
    link: '/en/sub-agents'
  }, {
    t: 0.795,
    kind: 'sub',
    label: '시스템 프롬프트',
    tokens: 0,
    subTokens: 900,
    color: '#6B6964',
    vis: 'hidden',
    desc: '하위 에이전트는 메인 세션보다 짧은 자체 시스템 프롬프트를 가져옵니다. 메인 세션의 자동 메모리는 포함되지 않습니다.',
    link: '/en/sub-agents#enable-persistent-memory'
  }, {
    t: 0.80,
    kind: 'sub',
    label: '프로젝트 CLAUDE.md (자체 복사본)',
    tokens: 0,
    subTokens: 1800,
    color: '#6A9BCC',
    vis: 'hidden',
    desc: '하위 에이전트도 CLAUDE.md를 로드합니다. 사용자 메인 컨텍스트가 아닌 하위 에이전트 컨텍스트에만 반영됩니다.',
    link: '/en/sub-agents'
  }, {
    t: 0.805,
    kind: 'sub',
    label: 'MCP 도구 + 스킬',
    tokens: 0,
    subTokens: 970,
    color: '#9B7BC4',
    vis: 'hidden',
    desc: '하위 에이전트는 동일한 MCP 서버 및 스킬에 접근할 수 있습니다.',
    link: '/en/sub-agents'
  }, {
    t: 0.81,
    kind: 'sub',
    label: '메인 세션의 작업 프롬프트',
    tokens: 0,
    subTokens: 120,
    color: '#558A42',
    vis: 'hidden',
    desc: '하위 에이전트는 사용자 프롬프트 대신 Claude가 작성한 작업 프롬프트를 받습니다.',
    link: '/en/sub-agents'
  }, {
    t: 0.82,
    kind: 'sub',
    label: 'session.ts 읽기',
    tokens: 0,
    subTokens: 2200,
    color: '#8A8880',
    vis: 'hidden',
    desc: '하위 에이전트가 작업을 수행합니다. 파일 읽기는 하위 에이전트 컨텍스트만 채웁니다.',
    link: '/en/sub-agents'
  }, {
    t: 0.825,
    kind: 'sub',
    label: 'timeouts.ts 읽기',
    tokens: 0,
    subTokens: 800,
    color: '#8A8880',
    vis: 'hidden',
    desc: '하위 에이전트 컨텍스트의 추가 파일 읽기입니다.',
    link: '/en/sub-agents'
  }, {
    t: 0.83,
    kind: 'sub',
    label: 'config/*.ts 읽기',
    tokens: 0,
    subTokens: 3100,
    color: '#8A8880',
    vis: 'hidden',
    desc: '필요한 만큼 파일을 읽을 수 있으며 메인 컨텍스트에는 영향을 주지 않습니다.',
    link: '/en/sub-agents'
  }, {
    t: 0.85,
    kind: 'claude',
    label: '하위 에이전트 요약 반환',
    tokens: 420,
    color: '#D97757',
    vis: 'brief',
    desc: '하위 에이전트의 최종 텍스트 응답과 소량의 메타데이터만 메인 컨텍스트로 반환됩니다. 6,100 토큰의 파일 읽기 대신 420 토큰 결과만 절약되었습니다.',
    link: '/en/sub-agents'
  }, {
    t: 0.86,
    kind: 'claude',
    label: 'Claude의 응답',
    tokens: 1200,
    color: '#D97757',
    vis: 'full',
    desc: '세션 타임아웃에 대한 분석 및 수정 내용입니다. 터미널에 표시됩니다.',
    link: null
  }, {}, {
    t: 0.875,
    kind: 'user',
    label: '!git status',
    tokens: 180,
    color: '#558A42',
    vis: 'full',
    desc: 'Claude가 수정한 파일을 확인하기 위해 ! 접두사로 셸 명령을 실행했습니다. 명령과 출력이 메시지의 일부로 컨텍스트에 포함됩니다.',
    link: '/en/interactive-mode#bash-mode-with-prefix'
  }, {
    t: 0.89,
    kind: 'user',
    label: '/commit-push',
    tokens: 620,
    color: '#558A42',
    vis: 'brief',
    desc: '`disable-model-invocation: true`가 설정된 스킬을 호출했습니다. 이때 전체 스킬 내용이 로드되고 명령이 실행됩니다.',
    tip: '커밋, 배포, 메시지 전송과 같은 사이드 이펙트가 있는 스킬에는 `disable-model-invocation: true`를 설정하여 필요할 때만 컨텍스트에 로드되도록 하세요.',
    link: '/en/skills#control-who-invokes-a-skill'
  }, {}, {
    t: 0.93,
    kind: 'compact',
    label: '/compact',
    tokens: 0,
    color: '#D97757',
    vis: 'brief',
    desc: '대화를 구조화된 요약으로 대체합니다. "Conversation compacted" 메시지가 표시됩니다.',
    link: '/en/how-claude-code-works#the-context-window'
  }].filter(e => e.t !== undefined), []);
  const VIS_META = {
    hidden: {
      label: '터미널에 보이지 않음',
      sub: '이 콘텐츠는 터미널에 나타나지 않습니다.'
    },
    brief: {
      label: '터미널에 한 줄로 표시됨',
      sub: '간략한 언급만 보이며 전체 내용은 보이지 않습니다.'
    },
    full: {
      label: '터미널에 표시됨',
      sub: '실제 콘텐츠가 터미널에 표출됩니다.'
    }
  };
  {}
  const GATES = [{
    at: 0.18,
    kind: 'prompt',
    text: 'Fix the auth bug where users get 401 after token refresh',
    resumeTo: 0.22
  }, {
    at: 0.705,
    kind: 'prompt',
    text: 'Use a subagent to research session timeout handling, then fix it',
    resumeTo: 0.72
  }, {
    at: 0.865,
    kind: 'bang',
    text: '!git status',
    resumeTo: 0.875
  }, {
    at: 0.88,
    kind: 'slash',
    text: '/commit-push',
    resumeTo: 0.89
  }, {
    at: 0.90,
    kind: 'compact',
    text: '/compact',
    resumeTo: 1
  }];
  const KIND_META = {
    auto: {
      badge: 'auto',
      detail: '자동 로드됨',
      badgeBg: 'rgba(94,93,89,0.15)',
      badgeColor: '#8A8880'
    },
    user: {
      badge: 'you',
      detail: '사용자 입력',
      badgeBg: 'rgba(85,138,66,0.15)',
      badgeColor: '#6BA656'
    },
    claude: {
      badge: 'claude',
      detail: 'Claude의 작업',
      badgeBg: 'rgba(217,119,87,0.12)',
      badgeColor: '#D97757'
    },
    hook: {
      badge: 'hook',
      detail: '훅 (자동)',
      badgeBg: 'rgba(184,134,11,0.15)',
      badgeColor: '#CCA020'
    },
    compact: {
      badge: 'compact',
      detail: '축소(Compaction)',
      badgeBg: 'rgba(217,119,87,0.12)',
      badgeColor: '#D97757'
    },
    sub: {
      badge: 'subagent',
      detail: '하위 에이전트 컨텍스트',
      badgeBg: 'rgba(155,123,196,0.12)',
      badgeColor: '#9B7BC4'
    }
  };
  const LEGEND = [{
    c: '#6B6964',
    l: 'System'
  }, {
    c: '#6A9BCC',
    l: 'CLAUDE.md'
  }, {
    c: '#E8A45C',
    l: 'Memory'
  }, {
    c: '#D4A843',
    l: 'Skills'
  }, {
    c: '#9B7BC4',
    l: 'MCP'
  }, {
    c: '#4A9B8E',
    l: 'Rules'
  }, {
    c: '#558A42',
    l: 'You'
  }, {
    c: '#8A8880',
    l: 'Files'
  }, {
    c: '#A09E96',
    l: 'Output'
  }, {
    c: '#D97757',
    l: 'Claude'
  }, {
    c: '#B8860B',
    l: 'Hooks'
  }];
  const fmt = n => n >= 1000 ? (n / 1000).toFixed(1).replace(/\.0$/, '') + 'K' : n + '';
  const [time, setTime] = useState(0);
  const [playing, setPlaying] = useState(false);
  const [hovIdx, setHovIdx] = useState(null);
  const [selIdx, setSelIdx] = useState(null);
  const [hovCat, setHovCat] = useState(null);
  const [gatesPassed, setGatesPassed] = useState(0);
  const [mounted, setMounted] = useState(false);
  const [hasInteracted, setHasInteracted] = useState(false);
  const lastRef = useRef(null);
  const scrollRef = useRef(null);
  const detailRef = useRef(null);
  useEffect(() => setMounted(true), []);
  const activeGate = GATES.find((g, i) => i >= gatesPassed && time >= g.at && time < g.resumeTo);
  useEffect(() => {
    if (!playing) return;
    let raf;
    let stopped = false;
    const tick = ts => {
      if (stopped) return;
      if (!lastRef.current) lastRef.current = ts;
      const dt = (ts - lastRef.current) / 1000;
      lastRef.current = ts;
      setTime(prev => {
        const next = prev + dt * 0.032;
        const gate = GATES.find((g, i) => i >= gatesPassed && next >= g.at && prev < g.resumeTo);
        if (gate) {
          stopped = true;
          setPlaying(false);
          return gate.at;
        }
        if (next >= 1) {
          stopped = true;
          setPlaying(false);
          return 1;
        }
        return next;
      });
      if (!stopped) raf = requestAnimationFrame(tick);
    };
    raf = requestAnimationFrame(tick);
    return () => {
      stopped = true;
      cancelAnimationFrame(raf);
      lastRef.current = null;
    };
  }, [playing, gatesPassed]);
  const sendPrompt = () => {
    if (!activeGate) return;
    const isCompact = activeGate.kind === 'compact';
    setGatesPassed(n => n + 1);
    setTime(activeGate.resumeTo);
    setSelIdx(null);
    setHovIdx(null);
    if (!isCompact) setPlaying(true);
  };
  const visibleCount = EVENTS.filter(e => e.t <= time).length;
  const preCompactVisible = useMemo(() => EVENTS.slice(0, visibleCount), [EVENTS, visibleCount]);
  const compactGateIdx = GATES.length - 1;
  const isCompacted = gatesPassed > compactGateIdx && preCompactVisible.some(e => e.kind === 'compact');
  const {visible, preCompactTotal} = useMemo(() => {
    const nonCompact = preCompactVisible.filter(e => e.kind !== 'compact');
    if (!isCompacted) {
      return {
        visible: preCompactVisible,
        preCompactTotal: 0
      };
    }
    {}
    const autoLoads = nonCompact.filter(e => e.kind === 'auto' && e.t < STARTUP_END && !e.noSurviveCompact);
    const summarized = nonCompact.filter(e => e.t >= STARTUP_END && e.kind !== 'sub');
    const sumTokens = summarized.reduce((s, e) => s + e.tokens, 0);
    const summaryBlock = {
      t: STARTUP_END,
      kind: 'compact',
      label: '대화 요약 (Conversation summary)',
      tokens: Math.round(sumTokens * 0.12),
      color: '#A09E96',
      vis: 'hidden',
      desc: `${summarized.length}개의 대화 이벤트가 하나의 구조화된 요약으로 압축되었습니다. 요약은 사용자의 요청과 의도, 주요 기술 개념, 수정된 파일 및 스니펫, 오류와 수정 방법, 보류 중인 작업을 유지합니다.`,
      link: '/en/how-claude-code-works#the-context-window'
    };
    return {
      visible: [...autoLoads, summaryBlock],
      preCompactTotal: nonCompact.reduce((s, e) => s + e.tokens, 0)
    };
  }, [preCompactVisible, isCompacted]);
  const {blocks, totalTokens} = useMemo(() => {
    const bl = visible.map((e, visIdx) => ({
      ...e,
      id: e.label + e.t,
      visIdx
    })).filter(e => e.tokens > 0 || e.label === '대화 요약 (Conversation summary)');
    return {
      blocks: bl,
      totalTokens: bl.reduce((s, b) => s + b.tokens, 0)
    };
  }, [visible]);
  const subTotal = useMemo(() => visible.filter(e => e.kind === 'sub').reduce((s, e) => s + (e.subTokens || 0), 0), [visible]);
  useEffect(() => {
    if (!scrollRef.current) return;
    if (isCompacted) scrollRef.current.scrollTo({
      top: 0,
      behavior: 'smooth'
    }); else if (playing || activeGate) scrollRef.current.scrollTop = scrollRef.current.scrollHeight;
  }, [visible.length, !!activeGate, isCompacted]);
  const rootRef = useRef(null);
  const keyStateRef = useRef({});
  const [isFullscreen, setIsFullscreen] = useState(false);
  keyStateRef.current = {
    time,
    activeGate,
    sendPrompt,
    hasInteracted
  };
  useEffect(() => {
    const onFsChange = () => setIsFullscreen(!!document.fullscreenElement);
    document.addEventListener('fullscreenchange', onFsChange);
    return () => document.removeEventListener('fullscreenchange', onFsChange);
  }, []);
  const toggleFullscreen = () => {
    if (!rootRef.current) return;
    if (document.fullscreenElement) document.exitFullscreen(); else rootRef.current.requestFullscreen().catch(() => {});
  };
  useEffect(() => {
    const onKey = e => {
      const tag = e.target.tagName;
      if (tag === 'INPUT' || tag === 'BUTTON' || tag === 'TEXTAREA' || tag === 'SELECT' || e.target.isContentEditable) return;
      if (!rootRef.current) return;
      const rect = rootRef.current.getBoundingClientRect();
      if (rect.width === 0 && rect.height === 0) return;
      if (rect.bottom < 0 || rect.top > window.innerHeight) return;
      if (e.code === 'Space') {
        const {time: t, activeGate: g, sendPrompt: send, hasInteracted: hi} = keyStateRef.current;
        if (!hi) return;
        e.preventDefault();
        if (t === 0) setPlaying(true); else if (g) send(); else if (t >= 1) {
          setTime(0);
          setGatesPassed(0);
          setSelIdx(null);
          setHovIdx(null);
          setPlaying(true);
        } else setPlaying(p => !p);
      }
    };
    window.addEventListener('keydown', onKey);
    return () => window.removeEventListener('keydown', onKey);
  }, []);
  const pct = totalTokens / MAX * 100;
  const barColor = pct > 75 ? '#D97757' : pct > 50 ? '#B8860B' : '#558A42';
  const activeIdx = selIdx !== null ? selIdx : hovIdx;
  const hovEvent = activeIdx !== null ? visible[activeIdx] : null;
  useEffect(() => {
    if (detailRef.current) detailRef.current.scrollTop = 0;
  }, [hovEvent]);
  const focusT = hovEvent ? hovEvent.t : time;
  const takeaway = isCompacted ? '축소(Compaction)는 대화를 구조화된 요약으로 대체합니다. 시스템 프롬프트, CLAUDE.md, 메모리 및 MCP 도구는 자동으로 다시 로드됩니다.' : focusT < STARTUP_END ? '입력하기 전에 많은 내용이 로드됩니다. CLAUDE.md, 메모리, 스킬 및 MCP 도구가 첫 프롬프트 전 이미 컨텍스트에 들어옵니다.' : focusT < 0.28 ? '이미 로드된 내용에 비하면 프롬프트는 매우 작습니다. Claude 컨텍스트의 대부분은 사용자 단어가 아닌 프로젝트 지식입니다.' : focusT < 0.50 ? 'Claude가 파일 하나를 읽을 때마다 컨텍스트가 늘어납니다. 경로 지정 규칙은 파일 읽기와 함께 자동으로 로드됩니다.' : focusT < 0.71 ? '훅은 도구 이벤트 시 자동으로 실행됩니다. 출력은 additionalContext JSON을 통해 Claude에게 전달됩니다.' : focusT < 0.79 ? '추가 질문은 동일한 컨텍스트에 계속 누적됩니다.' : focusT < 0.87 ? '하위 에이전트는 별도의 컨텍스트 창에서 작동합니다. 파일 읽기가 메인 컨텍스트에 영향을 주지 않으며 최종 요약만 반환됩니다.' : focusT < 0.88 ? '뱅(!) 명령은 셸에서 실행되고 다음 메시지 앞에 출력을 덧붙입니다.' : focusT < 0.90 ? '사용자 전용 스킬은 직접 호출할 때까지 컨텍스트 외부에 유지됩니다.' : '/compact는 핵심 정보를 유지하면서 공간을 확보하기 위해 대화를 요약합니다.';
  const terminalView = isCompacted ? '"Conversation compacted" 메시지가 표시되며 요약은 보이지 않게 수행됩니다.' : focusT < STARTUP_END ? '첫 메시지를 기다리는 입력 상자입니다. 입력 전에 위 내용들이 로드됩니다.' : focusT < 0.28 ? '작성한 프롬프트입니다. Claude는 아직 작업을 시작하지 않았습니다.' : focusT < 0.52 ? '프롬프트 및 "Reading files...". 규칙은 내용이 아닌 "Loaded" 알림으로 표시됩니다.' : focusT < 0.72 ? 'Claude의 응답 및 diff. npm test 같은 도구 출력은 요약으로 보입니다.' : focusT < 0.79 ? '추가 프롬프트입니다.' : focusT < 0.86 ? '하위 에이전트 작업 알림 후 결과가 보입니다. 하위 에이전트의 개별 파일 읽기는 보이지 않습니다.' : focusT < 0.90 ? 'Claude의 응답, git status 출력 및 commit-push 스킬 실행 화면입니다.' : '전체 대화입니다. /compact를 실행할 수 있습니다.';
  const mono = 'var(--font-mono, ui-monospace, SFMono-Regular, Menlo, monospace)';
  const renderWithCode = s => s.split('`').map((part, i) => i % 2 === 1 ? <code key={i} style={{
    fontFamily: mono,
    fontSize: '0.92em',
    background: 'var(--cw-track)',
    padding: '1px 4px',
    borderRadius: 3
  }}>{part}</code> : part);
  if (!mounted) return null;
  return <>
    <div className="cw-mobile-fallback">
      이 대화형 타임라인은 큰 화면에서 가장 잘 보입니다. 세부 내용은 <a href="#what-the-timeline-shows" style={{
    color: '#D97757'
  }}>아래의 텍스트 설명</a>을 참조하세요.
    </div>
    <div className="cw-root" ref={rootRef} onClickCapture={() => setHasInteracted(true)} style={isFullscreen ? {
    height: '100vh',
    borderRadius: 0,
    display: 'flex',
    flexDirection: 'column'
  } : {}}>
      <style>{`
        .cw-root {
          --cw-bg: #FAFAF8;
          --cw-text: #1A1918;
          --cw-text-2: #3D3C38;
          --cw-text-3: #5E5D59;
          --cw-text-dim: #6E6C64;
          --cw-text-faint: #8A8880;
          --cw-surface: rgba(0,0,0,0.025);
          --cw-surface-2: rgba(0,0,0,0.04);
          --cw-border: rgba(0,0,0,0.08);
          --cw-track: rgba(0,0,0,0.04);
          --cw-hover: rgba(0,0,0,0.04);
          --cw-rail: rgba(0,0,0,0.08);
          --cw-scrollbar: rgba(0,0,0,0.22);
          background: var(--cw-bg);
          border-radius: 12px;
          overflow: hidden;
          font-family: var(--font-sans, -apple-system, BlinkMacSystemFont, sans-serif);
          color: var(--cw-text);
          border: 1px solid var(--cw-border);
        }
        .dark .cw-root {
          --cw-bg: #111110;
          --cw-text: #E8E6DC;
          --cw-text-2: #B8B6AE;
          --cw-text-3: #9C9A92;
          --cw-text-dim: #8A8880;
          --cw-text-faint: #6E6C64;
          --cw-surface: rgba(255,255,255,0.02);
          --cw-surface-2: rgba(255,255,255,0.015);
          --cw-border: rgba(255,255,255,0.06);
          --cw-track: rgba(255,255,255,0.03);
          --cw-hover: rgba(255,255,255,0.04);
          --cw-rail: rgba(255,255,255,0.04);
          --cw-scrollbar: rgba(255,255,255,0.18);
        }
        .cw-scroll::-webkit-scrollbar { width: 6px; }
        .cw-scroll::-webkit-scrollbar-track { background: transparent; }
        .cw-scroll::-webkit-scrollbar-thumb { background: var(--cw-scrollbar); border-radius: 3px; }
        @keyframes cw-blink { 50% { opacity: 0; } }
        @keyframes cw-fadein { from { opacity: 0; transform: translateY(-4px); } to { opacity: 1; transform: translateY(0); } }
        .cw-compacted-row { animation: cw-fadein 0.3s ease-out backwards; }
        .cw-mobile-fallback { display: none; padding: 14px 16px; border-radius: 8px; font-size: 14px; border: 1px solid rgba(0,0,0,0.1); background: rgba(0,0,0,0.03); }
        .dark .cw-mobile-fallback { border-color: rgba(255,255,255,0.15); background: rgba(255,255,255,0.04); }
        @media (max-width: 700px) {
          .cw-root { display: none !important; }
          .cw-mobile-fallback { display: block; }
        }
      `}</style>

      {}
      <div style={{
    padding: '16px 20px 12px',
    display: 'flex',
    alignItems: 'flex-end',
    gap: 24
  }}>
        <div style={{
    flex: 1,
    minWidth: 0
  }}>
          <div style={{
    fontSize: 18,
    fontWeight: 600,
    letterSpacing: -0.3,
    lineHeight: 1
  }}>
            컨텍스트 창 탐색하기
          </div>
          <div style={{
    fontSize: 14,
    color: 'var(--cw-text-dim)',
    marginTop: 4
  }}>
            컨텍스트에 들어오는 내용과 비용을 보여주는 세션 시뮬레이션
          </div>
        </div>
        <div style={{
    textAlign: 'right',
    flexShrink: 0
  }}>
          <div style={{
    fontFamily: mono,
    fontSize: 20,
    fontWeight: 600,
    color: barColor,
    letterSpacing: -0.5,
    lineHeight: 1
  }}>
            ~{fmt(totalTokens)}<span style={{
    fontSize: 15,
    fontWeight: 500,
    marginLeft: 4
  }}>tokens</span>
          </div>
          <div style={{
    fontFamily: mono,
    fontSize: 13,
    color: 'var(--cw-text-dim)',
    marginTop: 2
  }} title="Token counts are illustrative. Actual values vary with your CLAUDE.md size, MCP servers, and file lengths.">
            / {fmt(MAX)} · 예시용
          </div>
        </div>
      </div>

      {}
      <div style={{
    padding: '0 20px'
  }}>
        <div style={{
    height: 4,
    borderRadius: 2,
    background: 'var(--cw-track)',
    overflow: 'hidden',
    marginBottom: 6
  }}>
          <div style={{
    width: pct + '%',
    height: '100%',
    background: barColor,
    transition: 'width 0.6s cubic-bezier(0.4, 0, 0.2, 1), background 0.3s'
  }} />
        </div>
        <div style={{
    height: 28,
    borderRadius: 5,
    background: 'var(--cw-track)',
    border: '1px solid var(--cw-border)',
    overflow: 'hidden',
    display: 'flex'
  }}>
          {blocks.map((b, i) => {
    const w = Math.max(b.tokens / MAX * 100, 0.15);
    const isHov = b.visIdx === activeIdx;
    const catMatch = hovCat && b.color === hovCat;
    const dimmed = hovCat ? !catMatch : activeIdx !== null && !isHov;
    return <div key={b.id} onMouseEnter={() => setHovIdx(b.visIdx)} onMouseLeave={() => setHovIdx(null)} onClick={() => setSelIdx(selIdx === b.visIdx ? null : b.visIdx)} style={{
      width: w + '%',
      height: '100%',
      background: b.color,
      opacity: isHov || catMatch ? 1 : dimmed ? 0.25 : 0.65,
      borderRight: i < blocks.length - 1 ? '0.5px solid var(--cw-border)' : 'none',
      transition: 'opacity 0.15s',
      cursor: 'pointer'
    }} />;
  })}
        </div>
        <div style={{
    display: 'flex',
    gap: 12,
    marginTop: 6,
    flexWrap: 'wrap',
    justifyContent: 'space-between'
  }}>
          <div style={{
    display: 'flex',
    gap: 12,
    flexWrap: 'wrap'
  }}>
            {LEGEND.map(x => {
    const active = hovCat === x.c;
    return <div key={x.l} onMouseEnter={() => setHovCat(x.c)} onMouseLeave={() => setHovCat(null)} style={{
      display: 'flex',
      alignItems: 'center',
      gap: 4,
      padding: '2px 6px',
      borderRadius: 4,
      cursor: 'pointer',
      background: active ? 'var(--cw-hover)' : 'transparent',
      transition: 'background 0.1s'
    }}>
                  <div style={{
      width: 6,
      height: 6,
      borderRadius: 1.5,
      background: x.c,
      opacity: active ? 1 : 0.7
    }} />
                  <span style={{
      fontSize: 12,
      color: active ? 'var(--cw-text)' : 'var(--cw-text-dim)'
    }}>{x.l}</span>
                </div>;
  })}
          </div>
          <div style={{
    display: 'flex',
    gap: 6,
    alignItems: 'center',
    fontSize: 12,
    color: 'var(--cw-text-dim)'
  }}>
            <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="#558A42" strokeWidth="2.5">
              <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z" /><circle cx="12" cy="12" r="3" />
            </svg>
            <span>= 터미널에 표시됨</span>
          </div>
        </div>
      </div>

      {}
      <div style={{
    display: 'flex',
    padding: '14px 20px 0',
    gap: 16,
    height: isFullscreen ? 'calc(100vh - 240px)' : 420
  }}>

        {}
        <div ref={scrollRef} className="cw-scroll" style={{
    flex: 1,
    minWidth: 0,
    overflowY: 'auto',
    paddingRight: 8,
    scrollBehavior: 'smooth'
  }}>
          {visible.length === 0 && !playing && <div style={{
    height: '100%',
    display: 'flex',
    flexDirection: 'column',
    alignItems: 'center',
    justifyContent: 'center',
    gap: 16
  }}>
              <div style={{
    fontFamily: mono,
    fontSize: 16,
    color: 'var(--cw-text-dim)',
    display: 'flex',
    alignItems: 'center',
    gap: 8
  }}>
                <span style={{
    color: 'var(--cw-text-faint)'
  }}>$</span>
                <span>claude</span>
                <span style={{
    display: 'inline-block',
    width: 8,
    height: 16,
    background: 'var(--cw-text-dim)',
    opacity: 0.5,
    animation: 'cw-blink 1s step-end infinite'
  }} />
              </div>
              <button onClick={() => setPlaying(true)} style={{
    padding: '10px 20px',
    borderRadius: 8,
    border: '1px solid rgba(217,119,87,0.3)',
    background: 'rgba(217,119,87,0.08)',
    color: '#D97757',
    fontSize: 15,
    fontWeight: 600,
    cursor: 'pointer',
    display: 'flex',
    alignItems: 'center',
    gap: 8
  }}>
                <span>▶</span>
                <span>세션 시작</span>
              </button>
              <div style={{
    fontSize: 13,
    color: 'var(--cw-text-faint)',
    maxWidth: 280,
    textAlign: 'center',
    lineHeight: 1.5
  }}>
                <code style={{
    fontFamily: mono
  }}>claude</code>를 실행하는 순간부터 전체 대화까지 컨텍스트에 로드되는 내용을 확인하세요.
              </div>
            </div>}
          {isCompacted && <div style={{
    marginBottom: 10,
    padding: '10px 12px',
    borderRadius: 6,
    background: 'rgba(217,119,87,0.05)',
    border: '1px solid rgba(217,119,87,0.15)'
  }}>
              <div style={{
    fontSize: 13,
    fontWeight: 600,
    color: '#D97757',
    marginBottom: 3
  }}>
                /compact 실행 후
              </div>
              <div style={{
    fontSize: 13,
    color: 'var(--cw-text-3)',
    lineHeight: 1.5,
    fontFamily: mono
  }}>
                {fmt(preCompactTotal)} → {fmt(totalTokens)} 토큰 · 절약됨: {fmt(preCompactTotal - totalTokens)}
              </div>
              <div style={{
    fontSize: 13,
    color: 'var(--cw-text-dim)',
    lineHeight: 1.5,
    marginTop: 4
  }}>
                컨텍스트에 남아 있는 항목: 메시지 기록 외부에 존재하고 축소 후 다시 로드되는 시작 콘텐츠와 전체 대화의 구조화된 요약입니다. 스킬 설명 목록은 다시 로드되지 않습니다.
              </div>
            </div>}
          {time > 0 && visible.length > 0 && <div style={{
    fontSize: 12,
    fontWeight: 700,
    color: 'var(--cw-text-faint)',
    textTransform: 'uppercase',
    letterSpacing: 0.6,
    marginBottom: 6,
    paddingLeft: 28
  }}>
              {isCompacted ? '축소 후 다시 로드됨' : '입력하기 전'}
            </div>}

          {time > 0 && visible.map((evt, i) => {
    const meta = KIND_META[evt.kind];
    const isHov = hovIdx === i;
    const prevKind = i > 0 ? visible[i - 1].kind : null;
    const isSub = evt.kind === 'sub';
    const enteringSubagent = isSub && prevKind !== 'sub';
    const leavingSubagent = prevKind === 'sub' && !isSub;
    let showPhase = null;
    if (evt.kind === 'user' && prevKind !== 'user') showPhase = '사용자'; else if (evt.kind === 'claude' && prevKind === 'user') showPhase = 'Claude 작업 중'; else if (evt.label === '대화 요약 (Conversation summary)') showPhase = '/compact로 요약됨';
    const isNewRow = isCompacted && !(evt.kind === 'auto' && evt.t < STARTUP_END);
    return <div key={evt.label + evt.t} className={isNewRow ? 'cw-compacted-row' : ''} style={isNewRow ? {
      animationDelay: `${i * 60}ms`
    } : {}}>
                {showPhase && <div style={{
      fontSize: 12,
      fontWeight: 700,
      color: 'var(--cw-text-faint)',
      textTransform: 'uppercase',
      letterSpacing: 0.6,
      marginTop: 14,
      marginBottom: 6,
      paddingLeft: 28
    }}>
                    {showPhase}
                  </div>}
                {enteringSubagent && <div style={{
      marginLeft: 28,
      marginTop: 6,
      marginBottom: 2,
      paddingLeft: 10,
      borderLeft: '2px solid rgba(155,123,196,0.4)',
      fontSize: 12,
      fontWeight: 600,
      color: '#9B7BC4',
      textTransform: 'uppercase',
      letterSpacing: 0.5
    }}>
                    하위 에이전트의 별도 컨텍스트 창
                  </div>}
                {leavingSubagent && <div style={{
      marginLeft: 28,
      marginBottom: 6,
      paddingLeft: 10,
      paddingBottom: 6,
      borderLeft: '2px solid rgba(155,123,196,0.4)',
      fontSize: 12,
      color: 'var(--cw-text-dim)',
      fontFamily: mono
    }}>
                    ↓ {fmt(subTotal)} 토큰이 하위 에이전트 컨텍스트에 남아 있음 · 요약만 반환됨
                  </div>}
                <div onMouseEnter={() => setHovIdx(i)} onMouseLeave={() => setHovIdx(null)} onClick={() => setSelIdx(selIdx === i ? null : i)} style={{
      display: 'flex',
      alignItems: 'flex-start',
      borderRadius: 6,
      cursor: 'pointer',
      background: selIdx === i || isHov ? 'var(--cw-hover)' : 'transparent',
      outline: selIdx === i ? '1px solid rgba(217,119,87,0.4)' : 'none',
      opacity: hovCat && evt.color !== hovCat ? 0.35 : 1,
      transition: 'background 0.1s, opacity 0.15s',
      marginLeft: isSub ? 28 : 0,
      paddingLeft: isSub ? 10 : 0,
      borderLeft: isSub ? '2px solid rgba(155,123,196,0.4)' : 'none'
    }}>
                  <div style={{
      width: 28,
      display: 'flex',
      flexDirection: 'column',
      alignItems: 'center',
      paddingTop: 8,
      flexShrink: 0
    }}>
                    <div style={{
      width: evt.kind === 'user' || evt.kind === 'compact' ? 10 : 7,
      height: evt.kind === 'user' || evt.kind === 'compact' ? 10 : 7,
      borderRadius: '50%',
      background: evt.color,
      opacity: isHov ? 1 : 0.6,
      transition: 'opacity 0.15s',
      boxShadow: isHov ? `0 0 8px ${evt.color}40` : 'none'
    }} />
                    {i < visible.length - 1 && <div style={{
      width: 1.5,
      flex: 1,
      background: 'var(--cw-rail)',
      marginTop: 2,
      minHeight: 6
    }} />}
                  </div>
                  <div style={{
      flex: 1,
      minWidth: 0,
      padding: '5px 10px 5px 4px',
      display: 'flex',
      alignItems: 'center',
      gap: 8
    }}>
                    <span style={{
      fontSize: 12,
      fontWeight: 600,
      padding: '1px 5px',
      borderRadius: 3,
      background: meta.badgeBg,
      color: meta.badgeColor,
      flexShrink: 0,
      fontFamily: mono
    }}>
                      {meta.badge}
                    </span>
                    <span style={{
      fontSize: 15,
      fontFamily: mono,
      color: isHov ? 'var(--cw-text)' : evt.kind === 'user' ? '#558A42' : evt.kind === 'auto' ? 'var(--cw-text-dim)' : 'var(--cw-text-2)',
      flex: 1,
      minWidth: 0,
      overflow: 'hidden',
      textOverflow: 'ellipsis',
      whiteSpace: 'nowrap',
      fontWeight: evt.kind === 'user' ? 550 : 400
    }}>
                      {evt.label}
                    </span>
                    {evt.tokens > 0 && <span style={{
      fontSize: 12,
      fontFamily: mono,
      color: 'var(--cw-text-faint)',
      flexShrink: 0
    }}>
                        +{fmt(evt.tokens)}
                      </span>}
                    {evt.subTokens > 0 && <span style={{
      fontSize: 12,
      fontFamily: mono,
      color: '#9B7BC4',
      flexShrink: 0,
      opacity: 0.6
    }}>
                        +{fmt(evt.subTokens)}
                      </span>}
                    {evt.tokens > 0 && <div style={{
      width: 50,
      height: 5,
      borderRadius: 2,
      background: 'var(--cw-track)',
      flexShrink: 0,
      overflow: 'hidden'
    }}>
                        <div style={{
      width: Math.min(evt.tokens / 5000 * 100, 100) + '%',
      height: '100%',
      background: evt.color,
      opacity: isHov ? 0.8 : 0.4,
      transition: 'opacity 0.15s'
    }} />
                      </div>}
                    <span style={{
      width: 14,
      flexShrink: 0,
      display: 'flex',
      justifyContent: 'center'
    }} title={VIS_META[evt.vis].label}>
                      {evt.vis !== 'hidden' && <svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke={evt.vis === 'full' ? '#558A42' : 'currentColor'} style={{
      color: 'var(--cw-text-faint)',
      opacity: evt.vis === 'full' ? 1 : 0.5
    }} strokeWidth="2">
                          <path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8-11-8-11-8z" /><circle cx="12" cy="12" r="3" />
                        </svg>}
                    </span>
                  </div>
                </div>
              </div>;
  })}

          {activeGate && (activeGate.kind === 'prompt' || activeGate.kind === 'bang' || activeGate.kind === 'slash') && <div style={{
    paddingLeft: 28,
    marginTop: 12,
    paddingRight: 8
  }}>
              <div style={{
    fontSize: 11,
    fontWeight: 600,
    color: '#6BA656',
    fontFamily: mono,
    textTransform: 'uppercase',
    letterSpacing: 0.5,
    marginBottom: 4,
    paddingLeft: 2
  }}>
                터미널 입력
              </div>
              <div style={{
    display: 'flex',
    alignItems: 'flex-start',
    gap: 8,
    padding: '10px 12px',
    borderRadius: 6,
    background: 'rgba(85,138,66,0.06)',
    border: '1px solid rgba(85,138,66,0.2)'
  }}>
                <span style={{
    color: '#558A42',
    fontSize: 15,
    fontFamily: mono,
    flexShrink: 0
  }}>❯</span>
                <span style={{
    fontSize: 15,
    fontFamily: mono,
    color: 'var(--cw-text-2)',
    flex: 1,
    lineHeight: 1.5
  }}>
                  {activeGate.text}
                  <span style={{
    display: 'inline-block',
    width: 7,
    height: 13,
    marginLeft: 2,
    background: '#558A42',
    opacity: 0.5,
    verticalAlign: 'middle',
    animation: 'cw-blink 1s step-end infinite'
  }} />
                </span>
                <button onClick={sendPrompt} style={{
    padding: '5px 12px',
    borderRadius: 5,
    border: 'none',
    background: '#558A42',
    color: '#fff',
    fontSize: 13,
    fontWeight: 600,
    cursor: 'pointer',
    flexShrink: 0
  }}>
                  {activeGate.kind === 'prompt' ? '전송 ↵' : '실행 ↵'}
                </button>
              </div>
            </div>}
          {activeGate && activeGate.kind === 'compact' && <div style={{
    paddingLeft: 28,
    marginTop: 12,
    paddingRight: 8
  }}>
              <div style={{
    padding: '12px 14px',
    borderRadius: 6,
    background: 'rgba(217,119,87,0.06)',
    border: '1px solid rgba(217,119,87,0.25)'
  }}>
                <div style={{
    fontSize: 13,
    color: 'var(--cw-text-3)',
    marginBottom: 8,
    lineHeight: 1.5
  }}>
                  컨텍스트 용량: <span style={{
    fontFamily: mono,
    fontWeight: 600,
    color: barColor
  }}>{fmt(totalTokens)} 토큰</span>.
                  이전 대화를 요약하고 공간을 확보하려면 <code style={{
    fontFamily: mono,
    background: 'var(--cw-track)',
    padding: '1px 4px',
    borderRadius: 3
  }}>/compact</code>를 실행하세요.
                </div>
                <div style={{
    display: 'flex',
    alignItems: 'center',
    gap: 8
  }}>
                  <span style={{
    color: '#D97757',
    fontSize: 15,
    fontFamily: mono
  }}>❯</span>
                  <span style={{
    fontSize: 15,
    fontFamily: mono,
    color: 'var(--cw-text-2)',
    flex: 1
  }}>
                    {activeGate.text}
                  </span>
                  <button onClick={sendPrompt} style={{
    padding: '5px 12px',
    borderRadius: 5,
    border: 'none',
    background: '#D97757',
    color: '#fff',
    fontSize: 13,
    fontWeight: 600,
    cursor: 'pointer',
    flexShrink: 0
  }}>
                    실행 ↵
                  </button>
                </div>
              </div>
            </div>}
        </div>

        {}
        <div style={{
    width: 300,
    flexShrink: 0,
    display: 'flex',
    flexDirection: 'column'
  }}>
          <div ref={detailRef} className="cw-scroll" style={{
    padding: '14px 16px',
    borderRadius: 10,
    background: 'var(--cw-surface)',
    border: '1px solid var(--cw-border)',
    flex: 1,
    minHeight: 0,
    overflowY: 'auto',
    display: 'flex',
    flexDirection: 'column',
    gap: 10
  }}>
            {hovEvent ? <div>
                <div style={{
    display: 'flex',
    alignItems: 'center',
    gap: 8,
    marginBottom: 8
  }}>
                  <div style={{
    width: 10,
    height: 10,
    borderRadius: 3,
    background: hovEvent.color,
    opacity: 0.8
  }} />
                  <span style={{
    fontSize: 16,
    fontWeight: 600
  }}>{hovEvent.label}</span>
                </div>
                <div style={{
    display: 'flex',
    width: 'fit-content',
    padding: '3px 8px',
    borderRadius: 4,
    marginBottom: 8,
    background: KIND_META[hovEvent.kind].badgeBg
  }}>
                  <span style={{
    fontSize: 12,
    fontWeight: 600,
    color: KIND_META[hovEvent.kind].badgeColor
  }}>
                    {KIND_META[hovEvent.kind].detail}
                  </span>
                </div>
                {hovEvent.tokens > 0 && <div style={{
    fontSize: 14,
    fontFamily: mono,
    color: 'var(--cw-text-dim)',
    marginBottom: 6
  }}>
                    {fmt(hovEvent.tokens)} 토큰
                  </div>}
                {hovEvent.subTokens > 0 && <div style={{
    fontSize: 14,
    fontFamily: mono,
    color: '#9B7BC4',
    marginBottom: 6
  }}>
                    하위 에이전트 컨텍스트의 {fmt(hovEvent.subTokens)} 토큰
                  </div>}
                <p style={{
    fontSize: 15,
    color: 'var(--cw-text-3)',
    lineHeight: 1.55,
    margin: 0
  }}>
                  {renderWithCode(hovEvent.desc)}
                </p>
                <div style={{
    marginTop: 10,
    padding: '8px 10px',
    borderRadius: 6,
    background: hovEvent.vis === 'full' ? 'rgba(85,138,66,0.08)' : 'var(--cw-surface-2)',
    border: '1px solid ' + (hovEvent.vis === 'full' ? 'rgba(85,138,66,0.2)' : 'var(--cw-border)')
  }}>
                  <div style={{
    display: 'flex',
    alignItems: 'center',
    gap: 6,
    marginBottom: 3
  }}>
                    <span style={{
    fontSize: 13,
    color: hovEvent.vis === 'full' ? '#558A42' : 'var(--cw-text-dim)'
  }}>
                      {hovEvent.vis === 'full' ? '●' : hovEvent.vis === 'brief' ? '◐' : '○'}
                    </span>
                    <span style={{
    fontSize: 12,
    fontWeight: 600,
    color: 'var(--cw-text-2)'
  }}>
                      {VIS_META[hovEvent.vis].label}
                    </span>
                  </div>
                  <div style={{
    fontSize: 13,
    color: 'var(--cw-text-dim)',
    lineHeight: 1.4
  }}>
                    {VIS_META[hovEvent.vis].sub}
                  </div>
                </div>
                {hovEvent.tip && <div style={{
    marginTop: 10,
    padding: '8px 10px',
    borderRadius: 6,
    background: 'rgba(85,138,66,0.06)',
    border: '1px solid rgba(85,138,66,0.15)'
  }}>
                    <div style={{
    fontSize: 12,
    fontWeight: 600,
    color: '#558A42',
    marginBottom: 3,
    display: 'flex',
    alignItems: 'center',
    gap: 4
  }}>
                      <span>💡</span> 컨텍스트 절약 팁
                    </div>
                    <div style={{
    fontSize: 13,
    color: 'var(--cw-text-3)',
    lineHeight: 1.5
  }}>
                      {renderWithCode(hovEvent.tip)}
                    </div>
                  </div>}
                {hovEvent.link && <a href={hovEvent.link} style={{
    display: 'inline-block',
    marginTop: 10,
    fontSize: 13,
    color: '#D97757',
    textDecoration: 'none',
    borderBottom: '1px solid rgba(217,119,87,0.3)'
  }}>
                    자세히 알아보기 →
                  </a>}
              </div> : <div style={{
    display: 'flex',
    flexDirection: 'column',
    alignItems: 'center',
    textAlign: 'center',
    gap: 4,
    padding: '12px 0 4px'
  }}>
                <div style={{
    fontSize: 22,
    opacity: 0.2
  }}>👁</div>
                <div style={{
    fontSize: 14,
    fontWeight: 500,
    color: 'var(--cw-text-dim)'
  }}>이벤트에 마우스를 올리거나 클릭하세요</div>
                <div style={{
    fontSize: 12,
    color: 'var(--cw-text-faint)',
    lineHeight: 1.4,
    maxWidth: 200
  }}>
                  마우스를 올려 미리보거나 클릭하여 고정 후 스크롤하세요.
                </div>
              </div>}

            <div style={{
    padding: '10px 12px',
    borderRadius: 8,
    background: 'rgba(217,119,87,0.05)',
    border: '1px solid rgba(217,119,87,0.12)'
  }}>
              <div style={{
    fontSize: 11,
    fontWeight: 700,
    color: '#D97757',
    textTransform: 'uppercase',
    letterSpacing: 0.5,
    marginBottom: 3
  }}>
                핵심 요약 (Key takeaway)
              </div>
              <div style={{
    fontSize: 13,
    color: 'var(--cw-text-3)',
    lineHeight: 1.5
  }}>
                {takeaway}
              </div>
            </div>

            <div style={{
    padding: '10px 12px',
    borderRadius: 8,
    background: 'var(--cw-surface-2)',
    border: '1px solid var(--cw-border)'
  }}>
              <div style={{
    fontSize: 11,
    fontWeight: 700,
    color: 'var(--cw-text-dim)',
    textTransform: 'uppercase',
    letterSpacing: 0.5,
    marginBottom: 3
  }}>
                터미널에 보이는 내용
              </div>
              <div style={{
    fontSize: 13,
    color: 'var(--cw-text-3)',
    lineHeight: 1.5
  }}>
                {terminalView}
              </div>
            </div>
          </div>
        </div>
      </div>

      {}
      <div style={{
    padding: '10px 20px 14px',
    display: 'flex',
    alignItems: 'center',
    gap: 10
  }}>
        <button aria-label={time >= 1 ? 'Restart' : activeGate ? 'Continue' : playing ? 'Pause' : 'Play'} onClick={() => {
    if (time >= 1) {
      setTime(0);
      setGatesPassed(0);
      setSelIdx(null);
      setHovIdx(null);
      setPlaying(true);
    } else if (activeGate) sendPrompt(); else setPlaying(!playing);
  }} style={{
    width: 30,
    height: 30,
    borderRadius: 6,
    border: 'none',
    background: 'rgba(217,119,87,0.1)',
    color: '#D97757',
    cursor: 'pointer',
    fontSize: 15,
    fontWeight: 700,
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center'
  }}>
          {time >= 1 ? '↺' : playing ? '⏸' : '▶'}
        </button>
        <div style={{
    flex: 1,
    height: 3,
    borderRadius: 2,
    background: 'var(--cw-track)',
    overflow: 'hidden'
  }}>
          <div style={{
    width: time * 100 + '%',
    height: '100%',
    background: '#D97757',
    transition: 'width 0.1s linear'
  }} />
        </div>
        <span style={{
    fontSize: 12,
    fontFamily: mono,
    color: 'var(--cw-text-faint)',
    minWidth: 30
  }}>
          {Math.round(time * 100)}%
        </span>
        <button onClick={toggleFullscreen} aria-label={isFullscreen ? 'Exit fullscreen' : 'Enter fullscreen'} title={isFullscreen ? 'Exit fullscreen' : 'Fullscreen'} style={{
    width: 28,
    height: 28,
    borderRadius: 6,
    border: '1px solid var(--cw-border)',
    background: 'var(--cw-surface)',
    color: 'var(--cw-text-dim)',
    cursor: 'pointer',
    fontSize: 15,
    flexShrink: 0,
    marginLeft: 4,
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'center'
  }}>
          {isFullscreen ? '⤡' : '⛶'}
        </button>
      </div>
    </div>
    </>;
};

Claude Code의 컨텍스트 창은 세션에 대해 Claude가 알고 있는 모든 것(지침, 읽는 파일, 자체 응답, 터미널에 전혀 나타나지 않는 콘텐츠)을 보관합니다. 아래 타임라인은 시작부터 축소(compaction)까지 전체 세션을 재생합니다: 입력하기 전에 로드되는 내용, Claude가 작업함에 따라 각 파일 읽기, 규칙 및 훅이 추가하는 내용, 하위 에이전트가 대용량 읽기를 컨텍스트에서 배제하는 방식을 보여줍니다. 동일한 내용의 목록은 [타임라인이 보여주는 내용](#what-the-timeline-shows)을 참조하세요.

<ContextWindow />

## 타임라인이 보여주는 내용

세션은 대표적인 토큰 수와 함께 현실적인 흐름을 탐색합니다.

* **입력하기 전**: CLAUDE.md, 자동 메모리, MCP 도구 이름 및 스킬 설명이 모두 컨텍스트에 로드됩니다. 자체 설정에 따라 [출력 스타일](/docs/en/output-styles)이나 [`--append-system-prompt`](/docs/en/cli-reference)의 텍스트가 추가되어 동일한 방식으로 시스템 프롬프트에 들어갈 수 있습니다.
* **Claude 작업 중**: 각 파일 읽기가 컨텍스트에 추가되고, [경로 한정 규칙](/docs/en/memory#path-specific-rules)이 일치하는 파일과 함께 자동으로 로드되며, 각 편집 후 [PostToolUse 훅](/docs/en/hooks-guide)이 실행됩니다.
* **후속 프롬프트**: [하위 에이전트](/docs/en/sub-agents)가 자체 별도 컨텍스트 창에서 조사를 처리하므로 대용량 파일 읽기가 메인 창에 들어오지 않습니다. 요약과 소량의 메타데이터만 반환됩니다.
* **끝 부분**: `/compact`가 대화를 구조화된 요약으로 대체합니다. 대부분의 시작 콘텐츠는 자동으로 다시 로드됩니다. 각 메커니즘에 어떤 일이 일어나는지는 아래 표에 표시됩니다.

## 축소(Compaction) 후 유지되는 항목

긴 세션이 축소되면 Claude Code는 컨텍스트 창에 맞게 대화 기록을 요약합니다. {/* min-version: 2.1.198 */}v2.1.198부터 요약 요청은 세션의 [extended thinking](/docs/en/model-config#extended-thinking) 구성을 상속받으므로 세션에서 활성화되어 있을 때 thinking이 켜진 상태로 추론합니다. Thinking은 요약 생성 방식에만 영향을 주며 이후 세션 설정은 변경되지 않은 상태로 유지됩니다. 지침에 어떤 일이 일어나는지는 로드된 방식에 따라 다릅니다.

| 메커니즘 | 축소 후 |
| :--- | :--- |
| 시스템 프롬프트 및 출력 스타일 | 변경 없음; 메시지 기록의 일부가 아님 |
| 프로젝트 루트 CLAUDE.md 및 범위 없는 규칙 | 디스크에서 다시 주입됨 |
| 자동 메모리 | 디스크에서 다시 주입됨 |
| `paths:` frontmatter가 있는 규칙 | 일치하는 파일을 다시 읽을 때까지 손실됨 |
| 하위 디렉토리의 중첩 CLAUDE.md | 해당 하위 디렉토리의 파일을 다시 읽을 때까지 손실됨 |
| 호출된 스킬 본문 | 다시 주입되며, 스킬당 5,000 토큰, 전체 25,000 토큰으로 제한됨 (오래된 것부터 삭제) |
| 훅 | 해당 없음; 훅은 컨텍스트가 아닌 코드로 실행됨 |

경로 한정 규칙 및 중첩된 CLAUDE.md 파일은 대상 파일을 읽을 때 메시지 기록으로 로드되므로 축소 시 다른 모든 항목과 함께 요약되어 사라집니다. 다음 번에 일치하는 파일을 읽을 때 다시 로드됩니다. 규칙이 축소 후에도 유지되어야 하는 경우 `paths:` frontmatter를 제거하거나 프로젝트 루트 CLAUDE.md로 이동하세요.

스킬 본문은 축소 후 다시 주입되지만 대형 스킬은 제한에 맞춰 절단되며, 전체 예산이 초과되면 가장 오래 전에 호출된 스킬부터 제거됩니다. 절단 시 파일 시작 부분이 보존되므로 `SKILL.md` 상단 부근에 가장 중요한 지침을 배치하세요.

## 컨텍스트가 가득 찼을 때

Claude Code는 한도에 도달하면 자동으로 축소(compaction)하므로 컨텍스트 창이 가득 차도 세션이 종료되지 않습니다. 자동 패스는 타임라인의 `/compact` 단계와 동일한 방식으로 작동합니다. 보존되는 항목은 [컨텍스트가 가득 찼을 때](/docs/en/how-claude-code-works#when-context-fills-up)를 참조하세요.

자동 패스가 실행되기 전에 조치를 취할 수도 있습니다.

* **초점을 맞춘 축소**: 긴 새 작업을 시작하기 전에 `/compact focus on the auth bug fix`와 같이 지침을 제공하여 `/compact`를 실행하세요. 요약은 자동 패스가 중요하다고 예상하는 것 대신 사용자가 선택한 내용을 보존합니다.
* **작업 간 초기화**: 관련 없는 작업으로 전환할 때 `/clear`를 실행하세요. 이전 대화는 다음에 필요한 파일을 몰아내고 모든 메시지에서 토큰 비용을 발생시킵니다.
* **대용량 읽기 위임**: 파일 내용이 사용자 창이 아닌 [하위 에이전트](/docs/en/sub-agents)의 컨텍스트 창에 남도록 조사를 전달하세요.

작은 대화 대신 더 큰 창이 필요한 경우 Fable 5, Sonnet 5, Opus 4.6 이상 및 Sonnet 4.6은 100만 토큰 컨텍스트 창을 지원합니다. 요금제별 이용 가능 여부 및 `[1m]` 모델 변형 선택 방법은 [확장된 컨텍스트](/docs/en/model-config#extended-context)를 참조하세요. Sonnet 5는 별도의 `[1m]` 변형 없이 1M에서 동작합니다. 자동 축소 임계값 및 LLM 게이트웨이 예외는 [Sonnet 5 컨텍스트 창](/docs/en/model-config#sonnet-5-context-window)을 참조하세요. 축소는 더 큰 한도에서도 동일한 방식으로 작동합니다.

## 자신의 세션 확인하기

시각화는 대표적인 수치를 사용합니다. 임의의 시점에 실제 컨텍스트 사용량을 보려면 `/context`를 실행하여 로드된 CLAUDE.md 및 자동 메모리 파일을 포함한 최적화 제안과 함께 카테고리별 실시간 내역을 확인하세요. 해당 파일을 열고 편집하려면 `/memory`를 실행하세요.

## 관련 리소스

타임라인에 표시된 기능에 대한 자세한 내용은 다음 페이지를 참조하세요.

* [Claude Code 확장](/docs/en/features-overview): CLAUDE.md vs 스킬 vs 규칙 vs 훅 vs MCP 사용 시점
* [지침 및 메모리 저장](/docs/en/memory): CLAUDE.md 계층 구조 및 자동 메모리
* [하위 에이전트](/docs/en/sub-agents): 별도의 컨텍스트 창에 조사 위임
* [모범 사례](/docs/en/best-practices): 주요 제약 조건으로서의 컨텍스트 관리
* [프롬프트 캐싱](/docs/en/prompt-caching): 캐시된 접두사를 무효화하는 동작
* [토큰 사용량 줄이기](/docs/en/costs#reduce-token-usage): 컨텍스트 사용량을 낮게 유지하기 위한 전략
