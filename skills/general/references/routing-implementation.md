# Routing Implementation (zoom-general)

This reference provides a concrete implementation model for routing a complex developer query from `zoom-general` to specialized skills.

## Runtime Assumptions

- Runtime: Node.js 18+.
- Language: TypeScript 5+.
- Input: free-form developer prompt.
- Output: deterministic handoff contract with primary skill, chained skills, rationale, and follow-up questions (if required).

## TypeScript Router Example

```ts
export type SkillId =
  | 'zoom-general'
  | 'zoom-rest-api'
  | 'zoom-mcp'
  | 'zoom-mcp/whiteboard'
  | 'zoom-mcp/team-chat'
  | 'zoom-webhooks'
  | 'zoom-websockets'
  | 'zoom-meeting-sdk'
  | 'zoom-meeting-sdk-web'
  | 'zoom-meeting-sdk-web-component-view'
  | 'zoom-video-sdk'
  | 'zoom-video-sdk-web'
  | 'zoom-plugin-sdk'
  | 'zoom-plugin-sdk-macos'
  | 'zoom-plugin-sdk-windows'
  | 'zoom-apps-sdk'
  | 'scribe'
  | 'summarizer'
  | 'translator'
  | 'zoom-rtms'
  | 'zoom-team-chat'
  | 'contact-center'
  | 'virtual-agent'
  | 'phone'
  | 'rivet-sdk'
  | 'probe-sdk'
  | 'zoom-ui-toolkit'
  | 'zoom-cobrowse-sdk'
  | 'zoom-oauth';

export interface RouteDecision {
  primarySkill: SkillId;
  chainedSkills: SkillId[];
  resourceHints: string[];
  confidence: number;
  rationale: string[];
  needsClarification: string[];
  warnings: string[];
}

interface Signals {
  meetingEmbed: boolean;
  meetingCustomUi: boolean;
  customVideo: boolean;
  pluginSdk: boolean;
  pluginMacos: boolean;
  pluginWindows: boolean;
  restApi: boolean;
  mcp: boolean;
  whiteboardMcp: boolean;
  teamChatMcp: boolean;
  webhooks: boolean;
  websockets: boolean;
  zoomApps: boolean;
  scribe: boolean;
  summarizer: boolean;
  translator: boolean;
  oauth: boolean;
  rtms: boolean;
  teamChat: boolean;
  contactCenter: boolean;
  virtualAgent: boolean;
  phone: boolean;
  rivet: boolean;
  preflight: boolean;
  uiToolkit: boolean;
  cobrowse: boolean;
}

const hasAny = (q: string, words: string[]): boolean => words.some((w) => q.includes(w));

export function detectSignals(rawQuery: string): Signals {
  const q = rawQuery.toLowerCase();
  return {
    meetingEmbed: hasAny(q, ['meeting sdk', 'embed meeting', 'join meeting ui', 'client view', 'component view']),
    meetingCustomUi: hasAny(q, [
      'custom meeting ui',
      'custom zoom meeting ui',
      'custom meeting video ui',
      'custom video ui for meeting',
      'zoommtgembedded',
      'zoomapproot',
      'embeddable meeting ui',
      'component view',
    ]),
    customVideo: hasAny(q, ['video sdk', 'custom video', 'attachvideo', 'peer-video-state-change']),
    pluginSdk: hasAny(q, [
      'plugin sdk',
      'zoom workplace ipc',
      'control zoom workplace',
      'installed zoom client',
      'desktop companion',
      'zoomtoolsuite',
      'zmtoolsuiteproxy',
    ]),
    pluginMacos: hasAny(q, ['plugin sdk macos', 'zoomtoolsuiteapi', 'zoomtoolsuite framework']),
    pluginWindows: hasAny(q, ['plugin sdk windows', 'zmtoolsuiteproxy', 'ztoolsuiteipcproxy']),
    restApi: hasAny(q, ['rest api', 'api create meeting', 'api list meetings', '/v2/', 'list users', 's2s oauth', 'meeting endpoint']),
    mcp: hasAny(q, ['zoom mcp', 'mcp server', 'agentic retrieval', 'tools/list', 'tools/call', 'semantic meeting search', 'search zoom', 'zoom docs search', 'zoom chat search']),
    whiteboardMcp: hasAny(q, ['whiteboard mcp', 'zoom whiteboard mcp', 'list whiteboards', 'get a whiteboard', 'wb/db', 'whiteboard_id']),
    teamChatMcp: hasAny(q, ['team chat mcp', 'zoom chat mcp', 'send zoom chat via mcp', 'edit zoom chat message mcp', 'zoom_chat_message_send', 'zoom_chat_channel_create']),
    webhooks: hasAny(q, ['webhook', 'x-zm-signature', 'event subscription', 'crc']),
    websockets: hasAny(q, ['websocket', 'real-time events', 'persistent connection']),
    zoomApps: hasAny(q, ['zoom apps sdk', 'in-client app', 'layers api', 'collaborate mode']),
    scribe: hasAny(q, ['scribe', 'transcribe file', 'transcribe recording', 'batch transcription', 'aiservices/scribe']),
    summarizer: hasAny(q, ['summarizer', 'summarize transcript', 'meeting recap api', 'action items', 'aiservices/summarizer']),
    translator: hasAny(q, ['translator', 'translate text', 'batch translation', 'target_languages', 'aiservices/translator']),
    oauth: hasAny(q, ['oauth', 'pkce', 'authorization code', 'account_credentials', 'token refresh']),
    rtms: hasAny(q, ['rtms', 'real-time media streams', 'live transcript stream', 'audio stream']),
    teamChat: hasAny(q, ['team chat', 'chatbot', 'chat card', 'chat message']),
    contactCenter: hasAny(q, ['contact center', 'engagement context', 'contact center smart embed', 'zcc']),
    virtualAgent: hasAny(q, ['virtual agent', 'zva', 'knowledge base sync', 'virtual assistant sdk']),
    phone: hasAny(q, ['zoom phone', 'phone smart embed', 'phone api', 'click to dial']),
    rivet: hasAny(q, ['rivet', 'zoom rivet']),
    preflight: hasAny(q, ['probe sdk', 'preflight', 'diagnostics', 'network readiness']),
    uiToolkit: hasAny(q, ['ui toolkit', 'prebuilt video ui']),
    cobrowse: hasAny(q, ['cobrowse', 'co-browse', 'shared browsing']),
  };
}

function pickPrimarySkill(s: Signals): SkillId {
  // Hard guardrails: SDK embed/custom-video requests should not fall back to REST.
  if (s.pluginMacos) return 'zoom-plugin-sdk-macos';
  if (s.pluginWindows) return 'zoom-plugin-sdk-windows';
  if (s.pluginSdk) return 'zoom-plugin-sdk';
  if (s.meetingCustomUi) return 'zoom-meeting-sdk-web-component-view';
  if (s.meetingEmbed && !s.customVideo) return 'zoom-meeting-sdk-web';
  if (s.meetingEmbed) return 'zoom-meeting-sdk';
  if (s.customVideo && !s.meetingEmbed) return 'zoom-video-sdk-web';
  if (s.customVideo) return 'zoom-video-sdk';

  if (s.virtualAgent) return 'virtual-agent';
  if (s.contactCenter) return 'contact-center';
  if (s.zoomApps) return 'zoom-apps-sdk';
  if (s.rtms) return 'zoom-rtms';
  if (s.summarizer) return 'summarizer';
  if (s.translator) return 'translator';
  if (s.scribe) return 'scribe';
  if (s.teamChatMcp) return 'zoom-mcp/team-chat';
  if (s.teamChat) return 'zoom-team-chat';
  if (s.phone) return 'phone';
  if (s.cobrowse) return 'zoom-cobrowse-sdk';
  if (s.uiToolkit) return 'zoom-ui-toolkit';
  if (s.preflight) return 'probe-sdk';
  if (s.websockets) return 'zoom-websockets';
  if (s.webhooks) return 'zoom-webhooks';
  if (s.whiteboardMcp) return 'zoom-mcp/whiteboard';
  if (s.mcp) return 'zoom-mcp';
  if (s.restApi) return 'zoom-rest-api';
  if (s.oauth) return 'zoom-oauth';

  return 'zoom-general';
}

function buildChain(primary: SkillId, s: Signals): SkillId[] {
  const chain = new Set<SkillId>();

  if (primary === 'zoom-meeting-sdk-web-component-view') chain.add('zoom-meeting-sdk-web');

  // Auth chaining.
  if (s.oauth || s.pluginSdk || s.pluginMacos || s.pluginWindows || s.restApi || s.mcp || s.whiteboardMcp || s.teamChatMcp || s.webhooks || s.websockets || s.phone || s.teamChat || s.virtualAgent) {
    chain.add('zoom-oauth');
  }

  // Optional server framework.
  if (s.rivet) chain.add('rivet-sdk');

  // Cross-surface chaining.
  if (primary === 'contact-center' && s.virtualAgent) chain.add('virtual-agent');
  if (primary === 'virtual-agent' && s.contactCenter) chain.add('contact-center');
  if (primary === 'summarizer' && s.scribe) chain.add('scribe');
  if (primary === 'translator' && s.scribe) chain.add('scribe');
  if (primary === 'translator' && s.summarizer) chain.add('summarizer');
  if (primary === 'summarizer' && s.translator) chain.add('translator');

  if (s.scribe || s.summarizer || s.translator) {
    chain.add('zoom-rest-api');
  }

  // Event channels often pair with REST resource management.
  if (s.webhooks || s.websockets) chain.add('zoom-rest-api');
  if (s.mcp && s.restApi) {
    chain.add('zoom-rest-api');
    chain.add('zoom-mcp');
  }
  if (s.teamChatMcp && s.teamChat) {
    chain.add('zoom-team-chat');
  }

  // Avoid redundant primary in chain.
  chain.delete(primary);

  return [...chain];
}

function buildResourceHints(primary: SkillId, s: Signals): string[] {
  const hints: string[] = [];

  if (primary === 'zoom-plugin-sdk' || primary === 'zoom-plugin-sdk-macos' || primary === 'zoom-plugin-sdk-windows' || s.pluginSdk || s.pluginMacos || s.pluginWindows) {
    hints.push('plugin-sdk/faq.md');
    hints.push('plugin-sdk/capabilities.md');
  }
  if ((s.pluginSdk || s.pluginMacos || s.pluginWindows) && s.meetingEmbed) {
    hints.push('meeting-sdk/SKILL.md');
  }
  if (primary === 'zoom-plugin-sdk-macos' || s.pluginMacos) {
    hints.push('plugin-sdk/macos/capabilities.md');
    hints.push('plugin-sdk/macos/references/package-and-api-map.md');
  }
  if (primary === 'zoom-plugin-sdk-windows' || s.pluginWindows) {
    hints.push('plugin-sdk/windows/capabilities.md');
    hints.push('plugin-sdk/windows/references/package-and-api-map.md');
  }

  return hints;
}

function validateDecision(primary: SkillId, s: Signals): string[] {
  const warnings: string[] = [];

  if (s.meetingEmbed && !['zoom-meeting-sdk', 'zoom-meeting-sdk-web', 'zoom-meeting-sdk-web-component-view'].includes(primary)) {
    warnings.push('meeting embed intent detected but primary skill is not zoom-meeting-sdk');
  }
  if (s.meetingCustomUi && primary !== 'zoom-meeting-sdk-web-component-view') {
    warnings.push('custom meeting UI intent detected but primary skill is not zoom-meeting-sdk-web-component-view');
  }
  if (s.customVideo && !['zoom-video-sdk', 'zoom-video-sdk-web'].includes(primary)) {
    warnings.push('custom video intent detected but primary skill is not zoom-video-sdk');
  }
  if ((s.pluginSdk || s.pluginMacos || s.pluginWindows) && !['zoom-plugin-sdk', 'zoom-plugin-sdk-macos', 'zoom-plugin-sdk-windows'].includes(primary)) {
    warnings.push('Plugin SDK intent detected but primary skill is not zoom-plugin-sdk');
  }
  if (s.meetingCustomUi && s.customVideo) {
    warnings.push('meeting UI intent and custom video intent both detected; prefer Meeting SDK Component View unless the user explicitly wants a non-meeting session');
  }
  if (s.restApi && (s.meetingEmbed || s.customVideo)) {
    warnings.push('mixed SDK + REST intent; keep SDK as primary and use REST only for resource workflows');
  }
  if (s.pluginSdk && s.meetingEmbed) {
    warnings.push('Plugin SDK and Meeting SDK signals both detected; ask whether the app controls installed Zoom Workplace or embeds the meeting');
  }

  return warnings;
}

function confidenceFromSignals(s: Signals): number {
  const hits = Object.values(s).filter(Boolean).length;
  if (hits >= 4) return 0.9;
  if (hits >= 2) return 0.78;
  if (hits === 1) return 0.65;
  return 0.5;
}

export function routeComplexQuery(query: string): RouteDecision {
  const signals = detectSignals(query);
  const primarySkill = pickPrimarySkill(signals);
  const chainedSkills = buildChain(primarySkill, signals);
  const resourceHints = buildResourceHints(primarySkill, signals);
  const warnings = validateDecision(primarySkill, signals);

  const needsClarification: string[] = [];
  if (signals.mcp && signals.restApi) {
    needsClarification.push('Do you want deterministic REST API automation, AI-agent MCP tooling, or a hybrid of both?');
  }
  if (signals.pluginSdk && signals.meetingEmbed) {
    needsClarification.push('Should the native app control installed Zoom Workplace over IPC, or embed Zoom meeting functionality inside the app?');
  }
  if (primarySkill === 'zoom-general') {
    needsClarification.push('Do you need SDK embed behavior, API resource automation, or event ingestion?');
  }

  const rationale = [
    `primary=${primarySkill}`,
    `signals=${JSON.stringify(signals)}`,
    `chained=${chainedSkills.join(',') || 'none'}`,
    `resources=${resourceHints.join(',') || 'none'}`,
  ];

  return {
    primarySkill,
    chainedSkills,
    resourceHints,
    confidence: confidenceFromSignals(signals),
    rationale,
    needsClarification,
    warnings,
  };
}
```

## Handoff Contract (Example Output)

```json
{
  "primarySkill": "zoom-plugin-sdk-windows",
  "chainedSkills": ["zoom-oauth"],
  "resourceHints": [
    "plugin-sdk/capabilities.md",
    "plugin-sdk/faq.md",
    "plugin-sdk/windows/capabilities.md",
    "plugin-sdk/windows/references/package-and-api-map.md"
  ],
  "confidence": 0.9,
  "rationale": [
    "primary=zoom-plugin-sdk-windows",
    "signals={\"pluginSdk\":true,\"pluginWindows\":true,\"oauth\":true,...}",
    "chained=zoom-oauth",
    "resources=plugin-sdk/faq.md,plugin-sdk/capabilities.md,plugin-sdk/windows/capabilities.md,plugin-sdk/windows/references/package-and-api-map.md"
  ],
  "needsClarification": [],
  "warnings": []
}
```

## Error Handling Expectations

- Unknown/low-signal prompts route to `zoom-general` with one clarifying question.
- Conflicting signals do not fail hard; produce warnings and preserve guardrails.
- Routing should be deterministic for the same normalized prompt.
