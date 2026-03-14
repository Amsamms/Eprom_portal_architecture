# AI Chat System

**Last Generated:** 2026-03-14

---

## Overview

All 4 calculation apps (Heater, Pump, MassMole, Optimizer) share a common AI chatbot library and UX pattern. The AI assistant is called "EPROM AI Companion" and is powered by Anthropic Claude (Haiku by default, with secret access to Sonnet and Opus).

---

## Shared Library: `chat-base.js`

All apps use `chat-base.js` — a shared JavaScript library that provides the complete chat UI and interaction logic via a configuration pattern. Each app creates a `ChatBase` instance with app-specific config.

### Configuration Pattern

```javascript
const chat = new ChatBase({
  // Required
  apiEndpoint: '/api/chat',
  appName: 'heater',

  // Avatar
  avatarUrl: '/assets/images/eprom-logo.gif',
  avatarAlt: 'EPROM AI',

  // Quick Chips (suggested prompts)
  quickChips: [
    'What is the thermal efficiency?',
    'Load the base case scenario',
    'Compare all scenarios'
  ],

  // Visual Actions (optional)
  visualActionsConfig: heaterVisualActionsConfig,

  // Callbacks
  onActionReceived: (actions) => { /* handle AI actions */ },
  onError: (error) => { /* handle errors */ },

  // Selectors
  chatContainerId: 'chat-container',
  toggleButtonId: 'chat-toggle-btn',
});
```

### What `chat-base.js` Provides

| Feature | Description |
|---------|-------------|
| **Chat panel** | Full chat UI — messages, input, send button, header |
| **Message rendering** | Markdown support, code blocks with copy button, timestamps |
| **Quick chips** | Suggested prompts below the input (configurable per app) |
| **Insight cards** | Styled cards for engineering insights returned by AI |
| **Typing indicator** | Animated dots during AI response |
| **Stop generation** | Button to cancel ongoing AI response |
| **Keyboard shortcuts** | `Escape` to close chat, `Ctrl+/` to toggle |
| **Auto-resize textarea** | Input grows with content |
| **localStorage history** | Chat messages persist across page refreshes (per conversation) |
| **Clear chat** | Trash icon button to clear history and reset conversation |
| **EPROM avatar** | Consistent branding on AI messages |
| **Desktop side panel** | 400px fixed width, push-aside layout (>1200px) |
| **Mobile bottom sheet** | 3-state: peek (~60px), half (55vh), full (~95vh) |
| **Model selector** | Hidden Easter egg (10 clicks on logo) for Haiku/Sonnet/Opus |

---

## Visual Actions: `visual-actions-base.js`

The visual actions library provides animated cursor interactions — when the AI calls a `fill_input_fields` tool, the frontend shows:

1. A floating cursor appears
2. Cursor moves smoothly to the target input field
3. Field highlights (glow effect)
4. Value types character by character
5. Cursor moves to the next field
6. When done, cursor fades out

Each app provides a `visual-actions-config.js` that maps field IDs to DOM selectors and defines pre/post hooks.

**Note:** MassMole and Optimizer do not use visual actions (by design — their form interactions are simpler).

---

## Tool-Use Flow

```
User types message
    │
    ▼
Frontend sends POST /api/chat
  { message, conversationId, model }
    │
    ▼
Server route (chat.js / chat.py):
  1. Authenticate user (JWT + session + app permission)
  2. Rate limit check (10/min)
  3. Detect prompt injection (log but currently don't block)
  4. Load conversation history from in-memory Map
  5. Build messages array: system prompt + history + user message
  6. Call Anthropic Claude API with tools
    │
    ▼
Claude responds (may include tool_use):
  ├── If tool_use → execute tool server-side
  │   ├── tool-executor.js runs the tool
  │   ├── Returns tool result to Claude
  │   ├── Claude may call more tools (loop)
  │   └── Eventually Claude responds with text
  │
  └── If text → return to frontend
    │
    ▼
Server returns:
  { response, actions[], tokensUsed }
    │
    ▼
Frontend:
  1. Renders AI message (markdown, code blocks)
  2. Executes actions array (visual animations, chart updates, form fills)
  3. Stores in localStorage for persistence
```

---

## Guardrails

### Identity Enforcement

The system prompt enforces:
- AI identifies as "EPROM AI Companion" — never "Claude" or "Anthropic"
- Stays within the engineering domain of the specific app
- Provides professional, helpful responses

### Anti-Injection

Each app has `guardrails.js` with **15+ detection patterns** for prompt injection:
- "ignore previous instructions"
- "you are now..."
- "system prompt override"
- Role-playing attempts
- Encoding-based bypasses

**Current behavior:** Detected injections are **logged but not blocked** — the message is still forwarded to Claude. The system prompt's guardrail instructions are the primary defense. This is a known MEDIUM-severity security issue (advisory-only guardrails).

---

## Token Optimization

Two strategies implemented in Session 44, achieving **63–86% token reduction**:

### Strategy B: `cleanForClaude()`

Before sending tool results to Claude, strip out data the AI doesn't need:
- Remove chart data (Plotly/ApexCharts JSON — thousands of points)
- Remove raw data arrays and individual entries
- Remove curve fitting coefficients
- Keep: KPIs, summary statistics, error messages

The frontend's `_action` data is **not** touched — visual actions still work correctly.

### Strategy A: `summarizeOldToolResults()`

On subsequent conversation turns, replace old `tool_result` messages in the history with one-line text summaries:

```
Before: { role: "tool", content: "{ ... 15KB of JSON ... }" }
After:  { role: "tool", content: "Calculation done: thermal_eff=87.3%, fuel_eff=91.2%, ..." }
```

### Measured Results

| Metric | Before | After | Reduction |
|--------|--------|-------|-----------|
| 4-turn pump conversation | ~92K tokens | ~13K tokens | **86%** |
| Average across apps | Variable | Variable | **63%** |
| Response quality | Baseline | Improved (more KPIs, better analysis) | Better |

---

## UX Features

### Quick Chips
Suggested prompts displayed below the chat input. Each app has domain-specific chips:
- Heater: "What is the thermal efficiency?", "Load the base case"
- Pump: "Calculate pump efficiency", "Show NPSH analysis"
- MassMole: "Load natural gas preset", "Convert to mass basis"
- Optimizer: "Run feature importance", "Optimize parameters"

### Insight Cards
Styled cards that the AI can return for key findings — formatted with title, value, description, and a colored accent bar. Used for KPI summaries and engineering recommendations.

### Chat History Persistence
Messages are stored in `localStorage` keyed by `${appName}_chat_history`. Persists across page refreshes but clears on explicit "Clear Chat" action. Each app has its own key to prevent cross-app history leaks (fixed in Session 36 — was previously a shared key collision bug).

### Model Selector Easter Egg
Click the EPROM logo 10 times → reveals a dropdown menu with:
- Claude Haiku (default, cheapest)
- Claude Sonnet (smarter, more expensive)
- Claude Opus (most capable, most expensive)

Selection is saved to localStorage. This is for internal testing only — not exposed to regular users.

---

## Conversation Storage

Chat conversations are stored in two places:
1. **In-memory Map** (server-side): Keyed by `conversationId`. Used for the active tool-use loop. No eviction policy — grows unbounded (known issue). Lost on container restart.
2. **PostgreSQL** (per-app chat_logs table): Every message (user and assistant) logged with conversation_id, role, content, tool_calls, model, token counts. Permanent record for admin audit.

---

## Per-App Chat Configuration

| App | Tools | Visual Actions | Model Default |
|-----|-------|---------------|---------------|
| Heater | 7 | Yes (cursor fills form fields) | Haiku |
| Pump | 6 | Yes (with before/after hooks) | Haiku |
| MassMole | 8 | No (uses action callbacks) | Haiku |
| Optimizer | 4 | No (routes to PHP API) | Haiku |
