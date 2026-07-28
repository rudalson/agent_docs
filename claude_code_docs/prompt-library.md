> ## 문서 목차
> 전체 문서 목차는 다음 URL에서 가져올 수 있습니다: https://code.claude.com/docs/llms.txt
> 더 자세히 탐색하기 전에 이 파일을 사용하여 사용 가능한 모든 페이지를 확인하세요.

# 프롬프트 라이브러리

> 태스크 및 역할별로 태그가 지정된 Claude Code용 복사-붙여넣기 프롬프트 모음입니다.

export const PromptLibrary = ({text = {}, labels = {}, tagLabels = {}, phaseLabels = {}, sourceLabels = {}, catLabels = {}}) => {
  const RAW = useMemo(() => [{
    id: 'get-oriented-in-a',
    sdlc: 'discover',
    cat: 'Onboard',
    startN: 1,
    roles: [],
    prompt: 'give me an overview of this codebase: architecture, key directories, and how the pieces connect',
    nextHref: '/en/memory',
    src: 'workflows'
  }, {
    id: 'explain-unfamiliar-code',
    sdlc: 'discover',
    cat: 'Understand',
    roles: [],
    prompt: 'explain what {path} does and how data flows through it. write it up as {format}',
    slots: {
      path: 'src/scheduler/queue.ts',
      format: 'an HTML page with a diagram, then open it in my browser'
    },
    nextHref: '/en/output-styles',
    src: 'workflows'
  }, {
    id: 'find-where-something-happens',
    sdlc: 'discover',
    cat: 'Understand',
    startN: 2,
    roles: [],
    prompt: 'where do we {behavior}?',
    slots: {
      behavior: 'validate uploaded file types'
    },
    src: 'workflows'
  }, {
    id: 'see-what-depends-on',
    sdlc: 'discover',
    cat: 'Understand',
    roles: [],
    prompt: 'what would break if I deleted {target}?',
    slots: {
      target: 'the retryWithBackoff helper'
    },
    src: 'workflows'
  }, {
    id: 'trace-how-code-evolved',
    sdlc: 'discover',
    cat: 'Understand',
    roles: [],
    prompt: 'look through the commit history of {path} and summarize how it evolved and why',
    slots: {
      path: 'internal/auth/session.go'
    },
    src: 'best-practices'
  }, {
    id: 'scope-a-change-before',
    sdlc: 'discover',
    cat: 'Understand',
    roles: ['pm', 'design'],
    prompt: 'which files would I need to touch to {change}?',
    slots: {
      change: 'add a dark mode toggle to settings'
    },
    src: 'teams'
  }, {
    id: 'ask-the-codebase-a',
    sdlc: 'discover',
    cat: 'Understand',
    roles: ['pm'],
    prompt: 'I am a {role}. walk me through what happens when a user {action}, from the UI down to the result',
    slots: {
      role: 'PM',
      action: 'clicks Export to PDF'
    },
    nextHref: '/en/output-styles',
    src: 'teams'
  }, {
    id: 'plan-a-multi-file',
    sdlc: 'design',
    cat: 'Plan',
    roles: ['pm', 'design'],
    prompt: 'plan how to refactor the {target} to {goal}. list the files you would change, but don\'t edit anything yet',
    slots: {
      target: 'payment module',
      goal: 'support multiple currencies'
    },
    src: 'workflows'
  }, {
    id: 'draft-a-spec-by',
    sdlc: 'design',
    cat: 'Plan',
    roles: ['pm'],
    prompt: 'I want to build {feature}. interview me about implementation, UX, edge cases, and tradeoffs until we have covered everything, then write the spec to SPEC.md',
    slots: {
      feature: 'per-workspace rate limits'
    },
    nextHref: '/en/skills',
    src: 'best-practices'
  }, {
    id: 'turn-a-meeting-into',
    sdlc: 'design',
    cat: 'Plan',
    roles: ['pm'],
    prompt: 'read {input} and write up the action items, then create a {tracker} ticket for each with acceptance criteria',
    slots: {
      input: '@meeting-notes.md',
      tracker: 'Linear'
    },
    needs: 'tracker',
    nextHref: '/en/skills',
    src: 'teams'
  }, {
    id: 'map-edge-cases-before',
    sdlc: 'design',
    cat: 'Plan',
    roles: ['design', 'pm'],
    prompt: 'list the error states, empty states, and edge cases for {feature} that the design needs to cover',
    slots: {
      feature: 'the file upload flow'
    },
    src: 'teams'
  }, {
    id: 'turn-a-mockup-into',
    sdlc: 'design',
    cat: 'Prototype',
    roles: ['design', 'pm', 'marketing'],
    paste: 'mockup',
    prompt: 'here is a mockup. build a working prototype I can click through, matching the layout and states shown',
    src: 'teams'
  }, {
    id: 'implement-from-a-screenshot',
    sdlc: 'design',
    cat: 'Prototype',
    roles: ['design'],
    paste: 'design',
    needs: 'browser',
    prompt: 'implement this design, then take a screenshot of the result, compare it to the original, and fix any differences',
    nextHref: '/en/goal',
    src: 'best-practices'
  }, {
    id: 'follow-an-existing-pattern',
    sdlc: 'build',
    cat: 'Implement',
    roles: [],
    prompt: 'look at how {example} is implemented to understand the pattern, then build {new} the same way',
    slots: {
      example: 'the GitHub webhook handler',
      new: 'a Stripe webhook handler'
    },
    nextHref: '/en/memory',
    src: 'best-practices'
  }, {
    id: 'generate-docs-for-code',
    sdlc: 'build',
    cat: 'Implement',
    roles: ['docs'],
    prompt: 'find {scope} without {format} comments and add them, matching the style already used in the file',
    slots: {
      scope: 'the public functions in src/auth/',
      format: 'JSDoc'
    },
    src: 'workflows'
  }, {
    id: 'add-a-small-well',
    sdlc: 'build',
    cat: 'Implement',
    roles: [],
    prompt: 'add a {endpoint} endpoint that returns {payload}',
    slots: {
      endpoint: '/health',
      payload: 'the app version and uptime'
    },
    src: 'workflows'
  }, {
    id: 'build-a-small-internal',
    sdlc: 'build',
    cat: 'Implement',
    roles: ['pm', 'design', 'marketing', 'docs'],
    prompt: 'create a {tool} using HTML, CSS, and vanilla JavaScript, then open it in my browser',
    slots: {
      tool: 'drag-and-drop Kanban board with three columns'
    },
    src: 'teams'
  }, {
    id: 'work-an-issue-end',
    sdlc: 'build',
    cat: 'Implement',
    roles: [],
    prompt: 'read issue #{issue}, implement the fix, and run the tests',
    slots: {
      issue: '312'
    },
    needs: 'gh',
    src: 'workflows'
  }, {
    id: 'find-and-update-copy',
    sdlc: 'build',
    cat: 'Implement',
    roles: ['design', 'docs', 'marketing'],
    prompt: 'find every place we say "{copy}" or a close variant, show me each one in context, then update them all to "{new}". leave tests and the changelog alone',
    slots: {
      copy: 'Sign up free',
      new: 'Start free trial'
    },
    src: 'teams'
  }, {
    id: 'draft-from-past-examples',
    sdlc: 'build',
    cat: 'Implement',
    roles: ['docs', 'marketing', 'pm'],
    prompt: 'read the {examples} in {folder} to learn the structure and voice, then draft a new one for {topic}',
    slots: {
      examples: 'privacy impact assessments',
      folder: 'legal/pia/',
      topic: 'the new analytics integration'
    },
    nextHref: '/en/skills',
    src: 'legal'
  }, {
    id: 'write-tests-run-them',
    sdlc: 'build',
    cat: 'Test',
    startN: 4,
    roles: [],
    prompt: 'write tests for {path}, run them, and fix any failures',
    slots: {
      path: 'app/parsers/feed.py'
    },
    nextHref: '/en/memory',
    src: 'workflows'
  }, {
    id: 'drive-implementation-from-tests',
    sdlc: 'build',
    cat: 'Test',
    roles: [],
    prompt: 'write tests for {feature} first, then implement it until they pass',
    slots: {
      feature: 'the password reset flow'
    },
    src: 'ebook'
  }, {
    id: 'fill-gaps-from-a',
    sdlc: 'build',
    cat: 'Test',
    roles: [],
    prompt: 'read {report} and add tests for the lowest-covered files until each is above {target}%',
    slots: {
      report: 'coverage/coverage-summary.json',
      target: '80'
    },
    nextHref: '/en/goal',
    src: 'workflows'
  }, {
    id: 'migrate-a-pattern-across',
    sdlc: 'build',
    cat: 'Refactor',
    roles: [],
    prompt: 'migrate everything from {from} to {to}: identify every place that needs to change, then make the changes',
    slots: {
      from: 'the old logging API',
      to: 'the structured logger'
    },
    src: 'workflows'
  }, {
    id: 'port-code-between-languages',
    sdlc: 'build',
    cat: 'Refactor',
    roles: [],
    prompt: 'port {source} to {target}, keeping the same {keep}',
    slots: {
      source: 'this Python module',
      target: 'Rust',
      keep: 'public API and test behavior'
    },
    src: 'teams'
  }, {
    id: 'optimize-against-a-measurable',
    sdlc: 'build',
    cat: 'Refactor',
    roles: ['data'],
    prompt: 'optimize {target} to bring {metric} from {current} down to under {goal}',
    slots: {
      target: 'the search query',
      metric: 'p95 latency',
      current: '2s',
      goal: '500ms'
    },
    nextHref: '/en/goal',
    src: 'ebook'
  }, {
    id: 'fix-a-precise-visual',
    sdlc: 'build',
    cat: 'Refactor',
    roles: ['design'],
    prompt: 'the {element} extends {amount} beyond the {container} on {viewport}. fix it.',
    slots: {
      element: 'login button',
      amount: '20px',
      container: 'card border',
      viewport: 'mobile'
    },
    nextHref: '/en/desktop#preview-your-app',
    src: 'ebook'
  }, {
    id: 'review-your-changes-before',
    sdlc: 'build',
    cat: 'Review',
    startN: 5,
    roles: [],
    prompt: 'review my uncommitted changes and flag anything that looks risky before I commit',
    nextHref: '/en/commands',
    src: 'workflows'
  }, {
    id: 'review-a-pull-request',
    sdlc: 'build',
    cat: 'Review',
    roles: [],
    prompt: 'review PR #{pr} and summarize what changed, then list any concerns',
    slots: {
      pr: '247'
    },
    needs: 'gh',
    nextHref: '/en/code-review',
    src: 'workflows'
  }, {
    id: 'review-infrastructure-changes-before',
    sdlc: 'build',
    cat: 'Review',
    roles: ['security', 'ops'],
    paste: 'plan',
    prompt: 'here is my Terraform plan output. what is this going to do, and is anything here going to cause problems?',
    src: 'teams'
  }, {
    id: 'run-a-security-review',
    sdlc: 'build',
    cat: 'Review',
    roles: ['security'],
    prompt: 'use a subagent to review {path} for security issues and report what it finds',
    slots: {
      path: 'src/api/'
    },
    nextHref: '/en/sub-agents',
    src: 'best-practices'
  }, {
    id: 'review-content-before-sending',
    sdlc: 'build',
    cat: 'Review',
    roles: ['marketing', 'docs'],
    prompt: 'review {file} for {concerns} and list anything I should fix before it goes to {reviewer}',
    slots: {
      file: 'launch-post.md',
      concerns: 'unsupported claims, missing attributions, and brand-guideline issues',
      reviewer: 'legal'
    },
    nextHref: '/en/skills',
    src: 'legal'
  }, {
    id: 'course-correct-a-wrong',
    sdlc: 'build',
    cat: 'Steer',
    roles: [],
    prompt: 'that is not right: {feedback}. try a different approach',
    slots: {
      feedback: 'the function signature needs to stay backward-compatible'
    },
    nextHref: '/en/checkpointing',
    src: 'best-practices'
  }, {
    id: 'narrow-the-scope-of',
    sdlc: 'build',
    cat: 'Steer',
    roles: [],
    prompt: 'that is too much. keep only the changes to {scope} and undo your other edits',
    slots: {
      scope: 'the validation logic in src/forms/'
    },
    src: 'best-practices'
  }, {
    id: 'turn-a-correction-into',
    sdlc: 'build',
    cat: 'Steer',
    roles: [],
    prompt: 'you keep {mistake}. add a rule to CLAUDE.md so this stops happening',
    slots: {
      mistake: 'using default exports when this project uses named exports'
    },
    nextHref: '/en/memory',
    src: 'best-practices'
  }, {
    id: 'resolve-merge-conflicts',
    sdlc: 'ship',
    cat: 'Git',
    roles: [],
    prompt: 'resolve the merge conflicts in this branch and explain what you kept from each side',
    src: 'workflows'
  }, {
    id: 'commit-with-a-generated',
    sdlc: 'ship',
    cat: 'Git',
    roles: [],
    prompt: 'commit these changes with a message that summarizes what I did',
    src: 'workflows'
  }, {
    id: 'open-a-pull-request',
    sdlc: 'ship',
    cat: 'Git',
    roles: [],
    prompt: 'find the {tracker} ticket about {topic} and open a PR that implements it',
    slots: {
      tracker: 'Linear',
      topic: 'the login timeout'
    },
    needs: 'tracker',
    src: 'workflows'
  }, {
    id: 'draft-release-notes-from',
    sdlc: 'ship',
    cat: 'Release',
    roles: ['pm', 'docs', 'marketing'],
    prompt: 'compare {from} to {to} and draft release notes grouped by feature, fix, and breaking change',
    slots: {
      from: 'v2.3.0',
      to: 'v2.4.0'
    },
    nextHref: '/en/skills',
    src: 'workflows'
  }, {
    id: 'write-a-ci-workflow',
    sdlc: 'ship',
    cat: 'Release',
    roles: ['ops'],
    prompt: 'write a GitHub Actions workflow that {steps} on every push to {branch}',
    slots: {
      steps: 'runs the tests and deploys to staging',
      branch: 'main'
    },
    src: 'workflows'
  }, {
    id: 'find-and-fix-a',
    sdlc: 'operate',
    cat: 'Debug',
    startN: 3,
    roles: [],
    prompt: 'the {test} test is failing, find out why and fix it',
    slots: {
      test: 'UserAuth'
    },
    src: 'workflows'
  }, {
    id: 'investigate-a-reported-error',
    sdlc: 'operate',
    cat: 'Debug',
    roles: ['ops'],
    prompt: 'users are seeing {symptom} on {where}. investigate and tell me what is going on',
    slots: {
      symptom: '500 errors',
      where: '/api/settings'
    },
    nextHref: '/en/web-quickstart#pre-fill-sessions',
    src: 'workflows'
  }, {
    id: 'fix-a-build-error',
    sdlc: 'operate',
    cat: 'Debug',
    roles: ['ops'],
    paste: 'error',
    prompt: 'here is a build error. fix the root cause and verify the build succeeds',
    src: 'best-practices'
  }, {
    id: 'investigate-a-production-incident',
    sdlc: 'operate',
    cat: 'Incident',
    roles: ['ops', 'security'],
    prompt: '{symptom}. check the logs, recent deploys, and config changes, then tell me the most likely cause',
    slots: {
      symptom: 'the checkout endpoint started returning 500s an hour ago'
    },
    nextHref: '/en/mcp',
    src: 'workflows'
  }, {
    id: 'diagnose-from-a-console',
    sdlc: 'operate',
    cat: 'Incident',
    roles: ['ops', 'data'],
    paste: 'screenshot',
    prompt: 'here is a screenshot of {console}. walk me through why {resource} is failing and give me the exact commands to fix it',
    slots: {
      console: 'the GCP Kubernetes dashboard',
      resource: 'this pod'
    },
    src: 'teams'
  }, {
    id: 'query-logs-in-plain',
    sdlc: 'operate',
    cat: 'Incident',
    roles: ['security', 'ops', 'data'],
    prompt: 'show me all {events} for {scope} over {timeframe}. write the query, run it, and tell me what stands out',
    slots: {
      events: 'failed logins',
      scope: 'the auth service',
      timeframe: 'the past 24 hours'
    },
    needs: 'db',
    src: 'cybersecurity'
  }, {
    id: 'analyze-a-data-file',
    sdlc: 'operate',
    cat: 'Data',
    roles: ['data', 'pm', 'marketing'],
    paste: 'csv',
    prompt: 'read {file}, summarize the key patterns, and write the results to {output}',
    slots: {
      file: '@reports/q1-signups.csv',
      output: 'an HTML page with charts, then open it in my browser'
    },
    nextHref: '/en/mcp',
    src: 'teams'
  }, {
    id: 'generate-variations-from-performance',
    sdlc: 'operate',
    cat: 'Data',
    roles: ['marketing', 'data'],
    paste: 'csv',
    prompt: 'read {file}, find the underperforming {items}, and generate {n} new variations that stay under {limit} characters',
    slots: {
      file: '@ads-performance.csv',
      items: 'headlines',
      n: '20',
      limit: '90'
    },
    nextHref: '/en/mcp',
    src: 'teams'
  }, {
    id: 'turn-a-recurring-task',
    sdlc: 'operate',
    cat: 'Automate',
    roles: [],
    prompt: 'create a /{name} skill for this project that {steps}',
    slots: {
      name: 'ship',
      steps: 'runs the linter and tests, then drafts a commit message'
    },
    src: 'workflows'
  }, {
    id: 'add-a-hook-for',
    sdlc: 'operate',
    cat: 'Automate',
    roles: [],
    prompt: 'write a hook that {action} after every {event}',
    slots: {
      action: 'runs prettier',
      event: 'edit to a .ts or .tsx file'
    },
    src: 'best-practices'
  }, {
    id: 'connect-a-tool-with',
    sdlc: 'operate',
    cat: 'Automate',
    roles: [],
    prompt: 'set up the {server} MCP server so you can read my {data} directly',
    slots: {
      server: 'Sentry',
      data: 'error reports'
    },
    src: 'workflows'
  }, {
    id: 'capture-what-to-remember',
    sdlc: 'operate',
    cat: 'Automate',
    roles: ['pm', 'docs'],
    prompt: 'summarize what we did this session and suggest what to add to CLAUDE.md',
    src: 'teams'
  }], []);
  const PROMPTS = useMemo(() => {
    if (typeof window !== 'undefined') {
      const rawIds = new Set(RAW.map(p => p.id));
      RAW.forEach(p => {
        if (!text[p.id]) console.warn('[prompt-library] no text[] entry for id:', p.id);
      });
      Object.keys(text).forEach(k => {
        if (!rawIds.has(k)) console.warn('[prompt-library] orphaned text[] key:', k);
      });
    }
    return RAW.map(p => ({
      ...p,
      title: p.id,
      teaches: '',
      ...text[p.id] || ({})
    }));
  }, [RAW, text]);
  const L = labels;
  const TL = k => tagLabels[k] || k;
  const CAT_TAG = useMemo(() => ({
    Onboard: 'understand',
    Understand: 'understand',
    Plan: 'plan',
    Prototype: 'prototype',
    Implement: 'build',
    Test: 'test',
    Refactor: 'refactor',
    Review: 'review',
    Steer: 'steer',
    Git: 'git',
    Release: 'release',
    Debug: 'debug',
    Incident: 'debug',
    Data: 'data',
    Automate: 'automate'
  }), []);
  const TAGS = useMemo(() => ['understand', 'plan', 'prototype', 'build', 'test', 'refactor', 'review', 'steer', 'debug', 'git', 'release', 'data', 'automate', 'pm', 'design', 'docs', 'marketing', 'security', 'ops'], []);
  const tagsOf = p => [CAT_TAG[p.cat], ...p.roles || []];
  const doc = useMemo(() => {
    const p = typeof window !== 'undefined' ? window.location.pathname : '';
    const base = p.startsWith('/docs/') ? '/docs' : '';
    const m = p.slice(base.length).match(/^\/([a-z]{2}(?:-[A-Z]{2})?)\//);
    const locale = m ? m[1] : 'en';
    return href => {
      if (!href || href[0] !== '/' || href[1] === '/') return href;
      return base + (href.startsWith('/en/') ? '/' + locale + href.slice(3) : href);
    };
  }, []);
  const linkify = s => {
    const out = [];
    let last = 0;
    const re = /\[([^\]]+)\]\(([^)]+)\)/g;
    for (let m; m = re.exec(s); ) {
      if (m.index > last) out.push(s.slice(last, m.index));
      out.push(<a key={m.index} href={doc(m[2])}>{m[1]}</a>);
      last = re.lastIndex;
    }
    if (last < s.length) out.push(s.slice(last));
    return out;
  };
  const codeify = s => s.split(/(`[^`]+`)/g).map((part, i) => part[0] === '`' ? <code key={i}>{part.slice(1, -1)}</code> : part);
  const SOURCES = useMemo(() => ({
    'workflows': '/en/common-workflows',
    'teams': 'https://claude.com/blog/how-anthropic-teams-use-claude-code',
    'legal': 'https://claude.com/blog/how-anthropic-uses-claude-legal',
    'cybersecurity': 'https://claude.com/blog/how-anthropic-uses-claude-cybersecurity',
    'best-practices': '/en/best-practices',
    'ebook': 'https://resources.anthropic.com/hubfs/Scaling%20agentic%20coding%20across%20your%20organization.pdf'
  }), []);
  const [mounted, setMounted] = useState(false);
  const [q, setQ] = useState('');
  const [start, setStart] = useState(true);
  const [sel, setSel] = useState(null);
  const [openId, setOpenId] = useState(null);
  const [copied, setCopied] = useState(null);
  const [fills, setFills] = useState({});
  const copyTimer = useRef(null);
  useEffect(() => {
    setMounted(true);
    return () => clearTimeout(copyTimer.current);
  }, []);
  const setFill = (id, key, val) => setFills(f => ({
    ...f,
    [id + '.' + key]: val
  }));
  const fillOf = (p, key) => {
    const v = fills[p.id + '.' + key];
    return v !== undefined ? v : p.slots && p.slots[key] !== undefined ? p.slots[key] : '';
  };
  const assemble = p => p.prompt.replace(/\{(\w+)\}/g, (_, k) => fillOf(p, k) || p.slots && p.slots[k] || k);
  const preview = p => p.prompt.replace(/\{(\w+)\}/g, (_, k) => p.slots && p.slots[k] || k);
  const bodyText = p => preview(p) + ' ' + p.teaches.replace(/\[([^\]]+)\]\([^)]+\)/g, '$1') + ' ' + (p.next || '');
  const widthFor = s => (s || '').length + 3 + 'ch';
  const ql = q.trim().toLowerCase();
  const toggleTag = k => {
    setStart(false);
    setSel(s => !ql && s === k ? null : k);
  };
  const clear = () => {
    setStart(false);
    setSel(null);
    setQ('');
  };
  const results = useMemo(() => {
    const list = PROMPTS.filter(p => {
      if (ql) return p.title.toLowerCase().includes(ql) || bodyText(p).toLowerCase().includes(ql);
      if (start) return !!p.startN;
      if (sel) return tagsOf(p).includes(sel);
      return true;
    });
    if (ql) return list;
    if (start) return list.sort((a, b) => a.startN - b.startN);
    if (sel) return list.sort((a, b) => (a.roles || []).length - (b.roles || []).length || (b.sdlc === 'operate') - (a.sdlc === 'operate'));
    return list;
  }, [PROMPTS, ql, start, sel]);
  const matchSnippet = p => {
    if (!ql || p.title.toLowerCase().includes(ql)) return null;
    const txt = bodyText(p);
    const at = txt.toLowerCase().indexOf(ql);
    if (at < 0) return null;
    const lo = Math.max(0, at - 30), hi = Math.min(txt.length, at + ql.length + 50);
    return [lo > 0 ? '…' : '', txt.slice(lo, at), <mark key="m">{txt.slice(at, at + ql.length)}</mark>, txt.slice(at + ql.length, hi), hi < txt.length ? '…' : ''];
  };
  const grouped = useMemo(() => {
    if (start && !q.trim()) return [];
    const g = {};
    for (const p of results) {
      const key = p.sdlc + '|' + p.cat;
      (g[key] = g[key] || ({
        sdlc: p.sdlc,
        cat: p.cat,
        items: []
      })).items.push(p);
    }
    return Object.values(g);
  }, [results, start, q]);
  const copy = async (str, id) => {
    try {
      await navigator.clipboard.writeText(str);
    } catch {
      const ta = document.createElement('textarea');
      ta.value = str;
      ta.setAttribute('readonly', '');
      ta.style.position = 'fixed';
      ta.style.opacity = '0';
      document.body.appendChild(ta);
      ta.select();
      document.execCommand('copy');
      document.body.removeChild(ta);
    }
    clearTimeout(copyTimer.current);
    setCopied(id);
    copyTimer.current = setTimeout(() => setCopied(null), 1600);
  };
  const promptBody = p => {
    if (!p.slots) return <code>{p.prompt}</code>;
    const parts = p.prompt.split(/(\{\w+\})/g);
    return <code>
        {parts.map((part, idx) => {
      const m = part.match(/^\{(\w+)\}$/);
      if (!m) return <span key={idx}>{part}</span>;
      const k = m[1];
      const val = fillOf(p, k);
      return <input key={idx} type="text" className="pl-slot" value={val} placeholder={p.slots[k] || k} aria-label={k} style={{
        width: widthFor(val || p.slots[k])
      }} onChange={e => setFill(p.id, k, e.target.value)} onFocus={e => e.target.select()} onClick={e => e.stopPropagation()} />;
    })}
      </code>;
  };
  const card = p => {
    const open = openId === p.id;
    const srcHref = SOURCES[p.src];
    const srcLabel = sourceLabels[p.src];
    const snip = matchSnippet(p);
    return <div key={p.id} className={'pl-card' + (open ? ' pl-open' : '')}>
        <button type="button" className="pl-head" onClick={() => setOpenId(open ? null : p.id)} aria-expanded={open}>
          <span className="pl-title">{p.title}</span>
          {!!p.startN && <span className="pl-chip">{L.startHere} · {p.startN}</span>}
        </button>
        {snip ? <div className="pl-match">{snip}</div> : <code className="pl-prompt-preview">{preview(p)}</code>}
        {open && <div className="pl-body">
            <div className="pl-label">{p.slots ? L.fillAndCopy : L.copyThis}</div>
            {p.needs && L.needs && L.needs[p.needs] && <div className="pl-hint pl-needs">
                <span className="pl-needs-label">{L.needsLabel}</span> {linkify(L.needs[p.needs])}
              </div>}
            {p.paste && L.paste && L.paste[p.paste] && <div className="pl-hint pl-paste">{L.paste[p.paste]}</div>}
            {p.slots && <div className="pl-hint">
                {L.hintBefore} <span className="pl-hint-chip">{L.hintChip}</span> {L.hintAfter}
              </div>}
            <div className="pl-prompt-box">
              <span className="pl-caret">{'❯'}</span>
              {promptBody(p)}
              <button type="button" className="pl-copy" onClick={() => copy(assemble(p), p.id)}>
                {copied === p.id ? L.copied : L.copy}
              </button>
            </div>
            <div className="pl-label">{L.whyWorks}</div>
            <div className="pl-teaches">{linkify(p.teaches)}</div>
            {p.nextHref && p.next && <div className="pl-next">
                <span className="pl-next-label">{L.makeItStick}</span>
                <a href={doc(p.nextHref)}>{codeify(p.next)} →</a>
              </div>}
            {srcLabel && <div className="pl-src">{L.from} {srcHref ? <a href={doc(srcHref)}>{srcLabel}</a> : srcLabel}</div>}
          </div>}
      </div>;
  };
  const STYLES = useMemo(() => `
.pl {
  --pl-accent: #D97757;
  --pl-accent-bg: rgba(217,119,87,0.07);
  --pl-bg: #fff;
  --pl-surface: #FAFAF7;
  --pl-border: #E8E6DC;
  --pl-border-subtle: rgba(31,30,29,0.08);
  --pl-text: #141413;
  --pl-text-2: #5E5D59;
  --pl-text-3: #73726C;
  --pl-text-4: #9C9A92;
  --pl-mono: var(--font-mono, ui-monospace, SFMono-Regular, Menlo, monospace);
  font-family: 'Anthropic Sans', -apple-system, BlinkMacSystemFont, sans-serif;
  margin: 1.5rem 0;
}
.dark .pl {
  --pl-accent: #E08363;
  --pl-accent-bg: rgba(224,131,99,0.12);
  --pl-bg: #181816;
  --pl-surface: #1F1E1B;
  --pl-border: #33322E;
  --pl-border-subtle: rgba(255,255,255,0.08);
  --pl-text: #EEEEEC;
  --pl-text-2: #B0AEA5;
  --pl-text-3: #8C8A81;
  --pl-text-4: #66645D;
}
.pl-controls { margin-bottom: 1.25rem; }
.pl-search { position: relative; margin-bottom: 0.75rem; }
.pl-search-input {
  width: 100%;
  padding: 0.625rem 2.25rem 0.625rem 0.875rem;
  background: var(--pl-surface);
  border: 1px solid var(--pl-border);
  border-radius: 8px;
  font-size: 0.9375rem;
  color: var(--pl-text);
  outline: none;
  transition: border-color 0.15s;
}
.pl-search-input:focus { border-color: var(--pl-accent); }
.pl-search-input::placeholder { color: var(--pl-text-4); }
.pl-clear {
  position: absolute;
  right: 0.625rem;
  top: 50%;
  transform: translateY(-50%);
  border: none;
  background: transparent;
  color: var(--pl-text-3);
  font-size: 1.125rem;
  cursor: pointer;
  padding: 0 0.25rem;
  line-height: 1;
}
.pl-clear:hover { color: var(--pl-text); }
.pl-tags { display: flex; flex-wrap: wrap; gap: 0.375rem; }
.pl-tag {
  border: 1px solid var(--pl-border);
  background: var(--pl-surface);
  color: var(--pl-text-2);
  font-size: 0.8125rem;
  padding: 0.25rem 0.625rem;
  border-radius: 9999px;
  cursor: pointer;
  transition: all 0.15s;
  user-select: none;
}
.pl-tag:hover { border-color: var(--pl-accent); color: var(--pl-text); }
.pl-tag.pl-sel {
  background: var(--pl-accent);
  border-color: var(--pl-accent);
  color: #fff;
  font-weight: 500;
}
.dark .pl-tag.pl-sel { color: #141413; }
.pl-section { margin-bottom: 1.75rem; }
.pl-section-head {
  display: flex;
  align-items: baseline;
  gap: 0.5rem;
  margin-bottom: 0.625rem;
  padding-bottom: 0.375rem;
  border-bottom: 1px solid var(--pl-border-subtle);
}
.pl-phase {
  font-size: 0.6875rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--pl-accent);
}
.pl-cat {
  font-size: 0.9375rem;
  font-weight: 600;
  color: var(--pl-text);
}
.pl-grid { display: flex; flex-direction: column; gap: 0.5rem; }
.pl-card {
  background: var(--pl-surface);
  border: 1px solid var(--pl-border);
  border-radius: 8px;
  overflow: hidden;
  transition: border-color 0.15s;
}
.pl-card:hover { border-color: var(--pl-text-4); }
.pl-card.pl-open { border-color: var(--pl-accent); }
.pl-head {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  padding: 0.625rem 0.875rem;
  background: transparent;
  border: none;
  text-align: left;
  cursor: pointer;
}
.pl-title { font-size: 0.875rem; font-weight: 600; color: var(--pl-text); }
.pl-chip {
  font-size: 0.6875rem;
  font-weight: 600;
  color: var(--pl-accent);
  background: var(--pl-accent-bg);
  padding: 0.125rem 0.5rem;
  border-radius: 9999px;
  white-space: nowrap;
}
.pl-prompt-preview {
  display: block;
  padding: 0 0.875rem 0.625rem;
  font-family: var(--pl-mono);
  font-size: 0.8125rem;
  color: var(--pl-text-2);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.pl-match {
  padding: 0 0.875rem 0.625rem;
  font-size: 0.8125rem;
  color: var(--pl-text-2);
}
.pl-match mark { background: var(--pl-accent-bg); color: var(--pl-accent); font-weight: 600; padding: 0 0.125rem; border-radius: 2px; }
.pl-body {
  padding: 0.875rem;
  border-top: 1px solid var(--pl-border);
  background: var(--pl-bg);
}
.pl-label {
  font-size: 0.6875rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--pl-text-3);
  margin-top: 0.875rem;
  margin-bottom: 0.375rem;
}
.pl-label:first-child { margin-top: 0; }
.pl-hint { font-size: 0.8125rem; color: var(--pl-text-2); margin-bottom: 0.5rem; }
.pl-hint a { color: var(--pl-accent); text-decoration: underline; }
.pl-hint-chip { background: var(--pl-accent-bg); color: var(--pl-accent); font-weight: 600; padding: 0.0625rem 0.375rem; border-radius: 4px; font-size: 0.75rem; }
.pl-needs { background: var(--pl-surface); border: 1px dashed var(--pl-border); padding: 0.5rem 0.625rem; border-radius: 6px; }
.pl-needs-label { font-weight: 600; color: var(--pl-text); }
.pl-paste { background: var(--pl-accent-bg); color: var(--pl-text); padding: 0.5rem 0.625rem; border-radius: 6px; border-left: 3px solid var(--pl-accent); }
.pl-prompt-box {
  position: relative;
  display: flex;
  align-items: flex-start;
  gap: 0.5rem;
  padding: 0.75rem 0.875rem;
  background: var(--pl-surface);
  border: 1px solid var(--pl-border);
  border-radius: 6px;
  font-family: var(--pl-mono);
  font-size: 0.8125rem;
  color: var(--pl-text);
  line-height: 1.5;
}
.pl-caret { color: var(--pl-accent); font-weight: bold; user-select: none; }
.pl-prompt-box code { display: block; flex: 1; white-space: pre-wrap; word-break: break-word; background: transparent; padding: 0; color: inherit; font-size: inherit; }
.pl-slot {
  font-family: var(--pl-mono);
  font-size: 0.8125rem;
  color: var(--pl-accent);
  background: var(--pl-accent-bg);
  border: 1px solid var(--pl-accent);
  border-radius: 4px;
  padding: 0.0625rem 0.25rem;
  outline: none;
  font-weight: 600;
  max-width: 100%;
}
.pl-copy {
  margin-left: auto;
  align-self: flex-start;
  border: 1px solid var(--pl-border);
  background: var(--pl-bg);
  color: var(--pl-text-2);
  font-size: 0.75rem;
  font-weight: 600;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  cursor: pointer;
  white-space: nowrap;
  transition: all 0.15s;
}
.pl-copy:hover { border-color: var(--pl-accent); color: var(--pl-accent); }
.pl-teaches { font-size: 0.84375rem; color: var(--pl-text-2); line-height: 1.5; }
.pl-teaches a { color: var(--pl-accent); text-decoration: underline; }
.pl-next { margin-top: 0.75rem; font-size: 0.8125rem; color: var(--pl-text-2); display: flex; align-items: center; gap: 0.375rem; }
.pl-next-label { font-weight: 600; color: var(--pl-text-3); text-transform: uppercase; font-size: 0.6875rem; letter-spacing: 0.06em; }
.pl-next a { color: var(--pl-accent); text-decoration: none; font-weight: 500; }
.pl-next a:hover { text-decoration: underline; }
.pl-next code { font-family: var(--pl-mono); font-size: 0.75rem; background: var(--pl-surface); padding: 0.0625rem 0.25rem; border-radius: 3px; border: 1px solid var(--pl-border); }
.pl-src { margin-top: 0.75rem; font-size: 0.75rem; color: var(--pl-text-4); }
.pl-src a { color: var(--pl-text-3); text-decoration: underline; }
.pl-empty { padding: 2rem; text-align: center; color: var(--pl-text-3); font-size: 0.875rem; }
`, []);
  if (!mounted) return null;
  return <div className="pl">
      <style>{STYLES}</style>
      <div className="pl-controls">
        <div className="pl-search">
          <input type="text" className="pl-search-input" value={q} placeholder={L.searchPlaceholder || 'Search prompts...'} onChange={e => {
          setQ(e.target.value);
          setStart(false);
        }} />
          {(q || sel || !start) && <button type="button" className="pl-clear" onClick={clear} title={L.clear}>×</button>}
        </div>
        <div className="pl-tags">
          <button type="button" className={'pl-tag' + (start && !q.trim() && !sel ? ' pl-sel' : '')} onClick={() => {
          setQ('');
          setSel(null);
          setStart(true);
        }}>
            {L.startHereTag || '⭐ Start here'}
          </button>
          <button type="button" className={'pl-tag' + (!start && !sel && !q.trim() ? ' pl-sel' : '')} onClick={() => {
          setQ('');
          setSel(null);
          setStart(false);
        }}>
            {L.allTag || 'All'}
          </button>
          {TAGS.map(k => <button key={k} type="button" className={'pl-tag' + (sel === k ? ' pl-sel' : '')} onClick={() => toggleTag(k)}>
              {TL(k)}
            </button>)}
        </div>
      </div>

      {grouped.length > 0 ? grouped.map(g => <div key={g.sdlc + '|' + g.cat} className="pl-section">
            <div className="pl-section-head">
              <span className="pl-phase">{phaseLabels[g.sdlc] || g.sdlc}</span>
              <span className="pl-cat">{catLabels[g.cat] || g.cat}</span>
            </div>
            <div className="pl-grid">
              {g.items.map(card)}
            </div>
          </div>) : results.length > 0 ? <div className="pl-grid">{results.map(card)}</div> : <div className="pl-empty">{L.noResults || 'No matching prompts found.'}</div>}
    </div>;
};

<PromptLibrary
  labels={{
    startHere: 'Start here',
    startHereTag: '⭐ Start here',
    allTag: 'All',
    searchPlaceholder: 'Search prompts by task, framework, or keyword...',
    clear: 'Clear filters',
    fillAndCopy: 'Fill parameters and copy',
    copyThis: 'Copy prompt',
    copy: 'Copy',
    copied: 'Copied!',
    hintBefore: 'Edit the highlighted',
    hintChip: 'fields',
    hintAfter: 'above to customize the prompt for your codebase.',
    needsLabel: 'Prerequisite:',
    whyWorks: 'Why this works',
    makeItStick: 'Learn more:',
    from: 'Source:',
    noResults: 'No prompts match your search.'
  }}
  tagLabels={{
    understand: 'Understand code',
    plan: 'Plan & spec',
    prototype: 'Prototype',
    build: 'Build feature',
    test: 'Test',
    refactor: 'Refactor',
    review: 'Review',
    steer: 'Course-correct',
    debug: 'Debug',
    git: 'Git & PRs',
    release: 'Release',
    data: 'Data & CSVs',
    automate: 'Automate',
    pm: 'PM',
    design: 'Design',
    docs: 'Docs',
    marketing: 'Marketing',
    security: 'Security',
    ops: 'Ops & Infra'
  }}
  phaseLabels={{
    discover: 'Phase 1: Discovery',
    design: 'Phase 2: Design',
    build: 'Phase 3: Build',
    ship: 'Phase 4: Ship',
    operate: 'Phase 5: Operate'
  }}
  catLabels={{
    Onboard: 'Onboarding',
    Understand: 'Code Understanding',
    Plan: 'Planning & Specs',
    Prototype: 'Prototypes',
    Implement: 'Implementation',
    Test: 'Testing',
    Refactor: 'Refactoring',
    Review: 'Code Review',
    Steer: 'Steering & Corrections',
    Git: 'Git Workflows',
    Release: 'Release & CI',
    Debug: 'Debugging',
    Incident: 'Incidents & Logs',
    Data: 'Data Analysis',
    Automate: 'Automation'
  }}
  sourceLabels={{
    workflows: 'Common Workflows',
    teams: 'How Anthropic Teams Use Claude Code',
    legal: 'How Anthropic Uses Claude Legal',
    cybersecurity: 'How Anthropic Uses Claude Cybersecurity',
    'best-practices': 'Best Practices',
    ebook: 'Scaling Agentic Coding'
  }}
  text={{
    'get-oriented-in-a': {
      title: 'Get oriented in a new codebase',
      teaches: 'Claude Code maps the structure and entry points before diving into changes.'
    },
    'explain-unfamiliar-code': {
      title: 'Explain unfamiliar code visually',
      teaches: 'Asking Claude Code for an HTML diagram opens an interactive browser page.'
    },
    'find-where-something-happens': {
      title: 'Find where a feature or behavior lives',
      teaches: 'Searches codebase semantics, not just exact string matches.'
    },
    'see-what-depends-on': {
      title: 'See what depends on a function or file',
      teaches: 'Traces imports and usages across the entire project before you refactor.'
    },
    'trace-how-code-evolved': {
      title: 'Trace how code evolved over time',
      teaches: 'Claude Code uses git log and blame to explain historical decisions.'
    },
    'scope-a-change-before': {
      title: 'Scope a change before committing to it',
      teaches: 'Lists affected files and risks before writing any code.'
    },
    'ask-the-codebase-a': {
      title: 'Ask the codebase a domain question from your perspective',
      teaches: 'Explains complex data flows tailoring the depth to your specific role.'
    },
    'plan-a-multi-file': {
      title: 'Plan a multi-file refactor without touching code',
      teaches: 'Forces Claude Code to produce a plan first, keeping your working tree clean.'
    },
    'draft-a-spec-by': {
      title: 'Draft a spec by interviewing me',
      teaches: 'Turns Claude Code into an interviewer to uncover edge cases and tradeoffs.'
    },
    'turn-a-meeting-into': {
      title: 'Turn meeting notes into issue tickets',
      teaches: 'Extracts action items and interfaces with issue trackers like Linear or Jira.'
    },
    'map-edge-cases-before': {
      title: 'Map edge cases before building UI',
      teaches: 'Anticipates empty, error, and loading states upfront.'
    },
    'turn-a-mockup-into': {
      title: 'Turn a mockup into a working prototype',
      teaches: 'Builds interactive HTML/JS prototypes directly from image attachments.'
    },
    'implement-from-a-screenshot': {
      title: 'Implement UI from a screenshot and self-correct',
      teaches: 'Claude Code compares browser screenshots against original designs to iterate.'
    },
    'follow-an-existing-pattern': {
      title: 'Follow an existing codebase pattern',
      teaches: 'Ensures new code matches existing architectural patterns and conventions.'
    },
    'generate-docs-for-code': {
      title: 'Generate docs for undocumented code',
      teaches: 'Scans for missing docstrings and fills them while preserving surrounding style.'
    },
    'add-a-small-well': {
      title: 'Add a small, well-defined endpoint',
      teaches: 'Great for routine API additions following existing routing patterns.'
    },
    'build-a-small-internal': {
      title: 'Build a small internal tool in one prompt',
      teaches: 'Generates single-file HTML apps you can use immediately in your browser.'
    },
    'work-an-issue-end': {
      title: 'Work an issue end-to-end',
      teaches: 'Fetches issue context via GitHub CLI, writes code, and verifies with tests.'
    },
    'find-and-update-copy': {
      title: 'Find and update copy everywhere',
      teaches: 'Updates user-facing text safely across strings, translations, and components.'
    },
    'draft-from-past-examples': {
      title: 'Draft new documents matching past examples',
      teaches: 'Reads existing files to adopt your organization\'s formatting and tone.'
    },
    'write-tests-run-them': {
      title: 'Write tests, run them, and fix failures',
      teaches: 'Executes your test runner autonomously until all tests pass.'
    },
    'drive-implementation-from-tests': {
      title: 'Drive implementation from failing tests (TDD)',
      teaches: 'Enforces test-driven development by writing specs as tests first.'
    },
    'fill-gaps-from-a': {
      title: 'Fill coverage gaps from a coverage report',
      teaches: 'Parses coverage JSON and targets untested lines systematically.'
    },
    'migrate-a-pattern-across': {
      title: 'Migrate a pattern across the codebase',
      teaches: 'Performs large-scale, systematic API or library migrations.'
    },
    'port-code-between-languages': {
      title: 'Port code between languages while keeping behavior',
      teaches: 'Translates logic while preserving API contracts and edge-case handling.'
    },
    'optimize-against-a-measurable': {
      title: 'Optimize code against a measurable goal',
      teaches: 'Iteratively benchmarks and refactors until performance targets are met.'
    },
    'fix-a-precise-visual': {
      title: 'Fix a precise visual layout bug',
      teaches: 'Fixes CSS and responsive layout issues from exact visual feedback.'
    },
    'review-your-changes-before': {
      title: 'Review uncommitted changes before committing',
      teaches: 'Acts as a pre-commit sanity check to catch accidental edits and security bugs.'
    },
    'review-a-pull-request': {
      title: 'Review a pull request',
      teaches: 'Uses GitHub CLI to inspect diffs and summarize changes concisely.'
    },
    'review-infrastructure-changes-before': {
      title: 'Review infrastructure changes before applying',
      teaches: 'Analyzes dry-run plans (like Terraform) to prevent accidental outages.'
    },
    'run-a-security-review': {
      title: 'Run a security review on a directory',
      teaches: 'Spawns an isolated subagent to audit code for OWASP vulnerabilities.'
    },
    'review-content-before-sending': {
      title: 'Review content before sending to reviewers',
      teaches: 'Checks copy or documentation against compliance and brand guidelines.'
    },
    'course-correct-a-wrong': {
      title: 'Course-correct a wrong direction',
      teaches: 'Redirects Claude Code cleanly without restarting the session.'
    },
    'narrow-the-scope-of': {
      title: 'Narrow the scope of an overly broad change',
      teaches: 'Instructs Claude Code to discard unwanted edits while keeping core fixes.'
    },
    'turn-a-correction-into': {
      title: 'Turn a correction into a permanent rule',
      teaches: 'Saves your preferences to CLAUDE.md so Claude Code remembers them forever.'
    },
    'resolve-merge-conflicts': {
      title: 'Resolve merge conflicts cleanly',
      teaches: 'Analyzes both sides of conflict markers and resolves them intelligently.'
    },
    'commit-with-a-generated': {
      title: 'Commit with a generated summary message',
      teaches: 'Inspects `git diff` to craft concise, conventional commit messages.'
    },
    'open-a-pull-request': {
      title: 'Open a pull request from an issue',
      teaches: 'Automates branching, committing, and opening PRs tied to issue tickets.'
    },
    'draft-release-notes-from': {
      title: 'Draft release notes from git history',
      teaches: 'Categorizes commits between tags into user-facing release notes.'
    },
    'write-a-ci-workflow': {
      title: 'Write a CI workflow file',
      teaches: 'Creates production-ready GitHub Actions or CI pipeline configurations.'
    },
    'find-and-fix-a': {
      title: 'Find and fix a failing test',
      teaches: 'Reads failure stack traces, locates the root cause, and applies a fix.'
    },
    'investigate-a-reported-error': {
      title: 'Investigate a reported error or 500',
      teaches: 'Traces runtime exceptions from log outputs down to source code.'
    },
    'fix-a-build-error': {
      title: 'Fix a build or compiler error',
      teaches: 'Parses complex compiler error outputs and fixes underlying type or syntax bugs.'
    },
    'investigate-a-production-incident': {
      title: 'Investigate a production incident',
      teaches: 'Correlates logs, recent commits, and environment changes during outages.'
    },
    'diagnose-from-a-console': {
      title: 'Diagnose an issue from a dashboard screenshot',
      teaches: 'Interprets visual cloud/K8s console errors to give exact CLI fix commands.'
    },
    'query-logs-in-plain': {
      title: 'Query logs in plain English',
      teaches: 'Translates natural language questions into database or log queries.'
    },
    'analyze-a-data-file': {
      title: 'Analyze a CSV or data file visually',
      teaches: 'Processes tabular data and generates interactive visual reports.'
    },
    'generate-variations-from-performance': {
      title: 'Generate copy variations from performance data',
      teaches: 'Combines data analysis with creative copy generation.'
    },
    'turn-a-recurring-task': {
      title: 'Turn a recurring task into a reusable skill',
      teaches: 'Creates custom `/slash` commands to automate frequent workflows.'
    },
    'add-a-hook-for': {
      title: 'Add an automated hook for file events',
      teaches: 'Configures background event handlers for automatic formatting or validation.'
    },
    'connect-a-tool-with': {
      title: 'Connect external tools via MCP',
      teaches: 'Sets up Model Context Protocol servers for third-party integrations.'
    },
    'capture-what-to-remember': {
      title: 'Capture session learnings into project memory',
      teaches: 'Summarizes session insights and updates project-level instructions.'
    }
  }}
/>
