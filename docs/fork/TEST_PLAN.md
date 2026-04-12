# LibreChat Fork — Test Plan

**Status:** Draft
**Date:** 2026-04-08
**PRD Reference:** `docs/fork/PRD.md`
**HLD Reference:** `docs/fork/HLD.md`
**Standard:** WPR-103

---

## 1. Discovery Phase: Multi-Model Capability Audit

### 1.1 Independent Enumeration

Three LLM CLIs (Claude, Gemini, Codex) independently read the full codebase — PRD, HLD, and every file in the HLD §6 Component Inventory — and produced a complete list of testable capabilities.

| CLI | Model Family | Output |
|-----|-------------|--------|
| Claude | Anthropic Claude | `3cli-sessions/capability-audit/round-1-claude.md` |
| Gemini | Google Gemini | `3cli-sessions/capability-audit/round-1-gemini.md` |
| Codex | OpenAI Codex | `3cli-sessions/capability-audit/round-1-codex.md` |

### 1.2 Aggregated Master Capability List

Merged from all three independent audits. Items found by multiple CLIs are noted; items unique to one CLI are retained per WPR-103 §1.2.

#### Feature 1: Default-Enabled MCP Toggle

| ID | Capability | Found By | Layer |
|----|-----------|----------|-------|
| MC-01 | Toggle MCP server as default-enabled (switch UI on MCPServerCard) | All 3 | Frontend |
| MC-02 | Persist default-enabled list in localStorage (`mcpDefaultEnabledAtom`) | All 3 | Frontend |
| MC-03 | Bump MCP generation counter on new chat (`mcpNewChatGenAtom`) | Claude, Codex | Frontend |
| MC-04 | Apply defaults on new chat (filter by configuredServers, write to mcpValues + ephemeralAgent) | All 3 | Frontend |
| MC-05 | Idempotency guard (`lastAppliedGenRef` prevents double-application) | Claude, Codex | Frontend |
| MC-06 | Defaults do not apply to agent conversations (backend guard in `applyContextToAgent`) | All 3 | Backend |
| MC-07 | Defaults do not affect existing conversations (conversation-keyed atoms) | Claude | Frontend |
| MC-08 | Ephemeral MCP ↔ Jotai atom sync (strips unconfigured servers) | Claude, Codex | Frontend |
| MC-09 | Extract MCP servers from agent tools (`extractMCPServers` — `_mcp_` delimiter) | All 3 | Backend |
| MC-10 | Get MCP instructions for servers (`getMCPInstructionsForServers`) | Claude | Backend |
| MC-11 | Build agent instructions (concatenate sharedRunContext + base + MCP) | Claude | Backend |
| MC-12 | Apply context to agent (full orchestration flow) | All 3 | Backend |
| MC-13 | Toggle tooltip text ("Enable {server} by default in new chats") | Claude | Frontend |
| MC-14 | MCP selection dual-write + timestamping (localStorage + atom + timestamp) | Codex | Frontend |
| MC-15 | Tab-isolated MCP storage (`createTabIsolatedStorage`) | Gemini | Frontend |
| MC-16 | `keepAddedConvos` suppresses gen bump (defaults won't apply in multi-convo mode) | Claude, Codex | Frontend |

#### Feature 2: Proactive Context Compaction

| ID | Capability | Found By | Layer |
|----|-----------|----------|-------|
| CC-01 | Auto Compact toggle UI (Switch in AdvancedPanel) | All 3 | Frontend |
| CC-02 | Compact threshold input (number field, appears when toggle on) | All 3 | Frontend |
| CC-03 | AutoCompact rendered in AdvancedPanel (clearErrors on mount) | Claude | Frontend |
| CC-04 | Agent form includes auto_compact/compact_threshold in payload | All 3 | Frontend |
| CC-05 | AgentSelect seeds defaults (auto_compact: false, compact_threshold: 80, type guards) | Claude, Codex | Frontend |
| CC-06 | defaultAgentFormValues includes compaction fields | Codex | Frontend |
| CC-07 | Zod validation (auto_compact: boolean, compact_threshold: 10-99) | All 3 | Backend |
| CC-08 | Mongoose schema (Boolean default false, Number default 80, min 10 max 99) | All 3 | Backend |
| CC-09 | IAgent TypeScript interface includes compaction fields | Claude | Backend |
| CC-10 | data-provider types (Agent, AgentCreateParams, AgentUpdateParams) | Claude | Both |
| CC-11 | AgentForm type includes compaction fields | Claude | Frontend |
| CC-12 | i18n keys for auto_compact UI (3 keys) | Claude | Frontend |
| CC-13 | AgentClient constructor reads auto_compact settings (sets contextStrategy, shouldSummarize, compactThreshold) | All 3 | Backend |
| CC-14 | Proactive threshold check in handleContextStrategy (totalTokens >= max * threshold → reduce window) | All 3 | Backend |
| CC-15 | Previous summary reuse (skip re-summarize when only 1 message discarded + messageId match) | Claude, Codex | Backend |
| CC-16 | AgentClient.summarizeMessages (formats transcript, calls model, targets 5% budget) | All 3 | Backend |
| CC-17 | COMPACT_PROMPT template (PromptTemplate with new_lines, max_tokens) | All 3 | Backend |
| CC-18 | Summary stored on oldest message (updateMessage with summary + summaryTokenCount) | Claude, Codex | Backend |
| CC-19 | /compact slash command interception (useSubmitMessage.ts, strict equality check) | All 3 | Frontend |
| CC-20 | /compact API endpoint (POST /convos/:id/compact, oldest 70%, ownership check) | All 3 | Backend |
| CC-21 | /compact uses configurable model (COMPACT_MODEL env, default gpt-4o-mini) | Claude, Codex | Backend |
| CC-22 | /compact token count estimation (Math.ceil(length / 4) heuristic) | Claude | Backend |
| CC-23 | useCompactConversation mutation (no onSuccess/onError callbacks — silent) | Claude, Codex | Frontend |
| CC-24 | Summary token budget calculation (5% of maxContextTokens) | Claude, Codex | Backend |
| CC-25 | Message transcript formatting (string-only content, skips array/tool content) | Claude | Backend |
| CC-26 | BaseClient.summarizeMessages abstract throws if not overridden | Claude | Backend |
| CC-27 | Fallback to discard strategy when summarization fails | Claude, Codex | Backend |

#### Feature 3: File Browser Panel (not yet implemented)

| ID | Capability | Found By | Layer |
|----|-----------|----------|-------|
| FB-01 | FileBrowserPanel component (tree view, mount points as roots) | All 3 | Frontend |
| FB-02 | FileTreeNode component (folder expand/collapse, file select) | Claude | Frontend |
| FB-03 | GET /api/files/browse endpoint (directory listing, path validation) | All 3 | Backend |
| FB-04 | GET /api/files/browse/content endpoint (file serving, Content-Type, Content-Disposition) | All 3 | Backend |
| FB-05 | interface.ts fileBrowser config flag (opt-in gating) | All 3 | Both |
| FB-06 | useSideNavLinks File Browser link (conditional rendering) | Claude | Frontend |
| FB-07 | Insert Path button (pastes path at cursor in prompt textarea) | Gemini | Frontend |
| FB-08 | Path traversal prevention (validate within fileBrowserPaths) | Gemini | Backend |

#### Infrastructure

| ID | Capability | Found By | Layer |
|----|-----------|----------|-------|
| IF-01 | .dockerignore excludes **/dist (build from source) | All 3 | Infra |
| IF-02 | Dockerfile builds all TS packages via npm run frontend | All 3 | Infra |
| IF-03 | Container includes bash + openssh-client (MCP tool support) | All 3 | Infra |
| IF-04 | Configurable NODE_MAX_OLD_SPACE_SIZE for build | Gemini, Codex | Infra |
| IF-05 | jemalloc LD_PRELOAD configuration | Gemini | Infra |

#### Cross-Feature Interactions

| ID | Capability | Found By | Layer |
|----|-----------|----------|-------|
| XF-01 | context.ts serves both F1 (MCP source) and F2 (instructions for compaction) | Claude | Backend |
| XF-02 | Auto-compact with MCP tools (tool calls summarized away, tools re-injected per request) | Claude | Backend |
| XF-03 | /compact on agent vs non-agent conversations (different models/keys) | Claude | Backend |
| XF-04 | Default MCP + agent selection timing (UI briefly shows defaults that won't be used) | Claude | Frontend |

**Totals:** 56 capabilities (16 F1 + 27 F2 + 8 F3 + 5 Infra + 4 Cross-feature)

### 1.3 Coverage Assessment

| ID | Capability | Coverage | Notes |
|----|-----------|----------|-------|
| MC-01 | Toggle default-enabled | NONE | |
| MC-02 | Persist in localStorage | NONE | |
| MC-03 | Bump gen on new chat | NONE | |
| MC-04 | Apply defaults on new chat | NONE | Bug #553 |
| MC-05 | Idempotency guard | NONE | |
| MC-06 | Agent guard (backend) | NONE | Bug #554 |
| MC-07 | Existing chat isolation | NONE | |
| MC-08 | Ephemeral ↔ Jotai sync | NONE | |
| MC-09 | extractMCPServers | NONE | |
| MC-10 | getMCPInstructionsForServers | NONE | |
| MC-11 | buildAgentInstructions | NONE | |
| MC-12 | applyContextToAgent | NONE | Bug #554 |
| MC-13 | Toggle tooltip | NONE | |
| MC-14 | Dual-write + timestamp | NONE | |
| MC-15 | Tab-isolated storage | NONE | |
| MC-16 | keepAddedConvos suppression | NONE | |
| CC-01 | Auto Compact toggle UI | NONE | |
| CC-02 | Threshold input | NONE | |
| CC-03 | AdvancedPanel clearErrors | NONE | |
| CC-04 | Payload includes fields | NONE | |
| CC-05 | AgentSelect seeds defaults | NONE | |
| CC-06 | defaultAgentFormValues | NONE | |
| CC-07 | Zod validation | NONE | |
| CC-08 | Mongoose schema | NONE | |
| CC-09 | IAgent interface | NONE | Type-only |
| CC-10 | data-provider types | NONE | Type-only |
| CC-11 | AgentForm type | NONE | Type-only |
| CC-12 | i18n keys | NONE | |
| CC-13 | AgentClient constructor | NONE | |
| CC-14 | Proactive threshold check | NONE | |
| CC-15 | Previous summary reuse | NONE | |
| CC-16 | summarizeMessages | NONE | |
| CC-17 | COMPACT_PROMPT | NONE | |
| CC-18 | Summary stored on message | NONE | |
| CC-19 | /compact interception | NONE | Bug #555 |
| CC-20 | /compact endpoint | REAL | `scripts/test_compact.py` passes |
| CC-21 | Configurable model | NONE | |
| CC-22 | Token count heuristic | NONE | |
| CC-23 | Mutation (no callbacks) | NONE | Bug #555 |
| CC-24 | Summary token budget | NONE | |
| CC-25 | Transcript formatting | NONE | |
| CC-26 | Base abstract throws | NONE | |
| CC-27 | Fallback to discard | NONE | |
| FB-01 | FileBrowserPanel | NONE | Not implemented |
| FB-02 | FileTreeNode | NONE | Not implemented |
| FB-03 | GET /browse | NONE | Not implemented |
| FB-04 | GET /browse/content | NONE | Not implemented |
| FB-05 | fileBrowser config | NONE | Not implemented |
| FB-06 | Nav link | NONE | Not implemented |
| FB-07 | Insert Path button | NONE | Not implemented |
| FB-08 | Path traversal prevention | NONE | Not implemented |
| IF-01 | .dockerignore **/dist | NONE | |
| IF-02 | npm run frontend build | NONE | |
| IF-03 | bash + openssh-client | NONE | |
| IF-04 | NODE_MAX_OLD_SPACE_SIZE | NONE | |
| IF-05 | jemalloc LD_PRELOAD | NONE | |
| XF-01 | context.ts dual purpose | NONE | |
| XF-02 | Compact + MCP interaction | NONE | |
| XF-03 | /compact agent vs non-agent | NONE | |
| XF-04 | Default MCP + agent timing | NONE | |

**Coverage summary:** 1 REAL, 0 MOCK, 55 NONE (out of 56 capabilities). Target: zero NONE entries for implemented capabilities.

---

## 2. Engineering Requirements Gate

Before functional testing, verify compliance with applicable engineering requirements (ERQ-001, scoped per PRD §6):

| Check | Scope | Method |
|-------|-------|--------|
| Code formatting | All fork-added/modified files | ESLint + Prettier |
| TypeScript strict mode | All `.ts`/`.tsx` files in fork scope | `tsc --noEmit` on modified packages |
| No `any` types | All fork TypeScript code | Grep audit |
| No hardcoded secrets | All fork code | Grep for API keys, passwords, tokens |
| Input validation | All new endpoints, form fields | Code review + test scenarios |
| Null checking | All new code paths | Code review |
| Import order | All modified files | ESLint rule check |

These checks are prerequisites — functional tests do not start until all pass.

---

## 3. Test Strategy: Two Layers

### 3.1 Mock Tests (Unit)

Not used for this fork. Per PRD §6 (Known Constraints §3): "Jest tests in the `api` workspace fail due to dependency resolution issues." The monorepo's workspace configuration prevents reliable unit test execution.

**Justification for exclusion:** The fork modifies upstream files that depend on the full workspace dependency tree. Attempting to run Jest in isolation produces false failures unrelated to our code. All testing is done via live integration tests against the deployed container.

### 3.2 Live Tests (Integration)

All test scenarios are **live** — executed against the deployed `henson-librechat` container on M5 (192.168.1.120). Two test execution methods:

| Method | Tool | Purpose |
|--------|------|---------|
| Python scripts | `requests` + SSH | API endpoints, database state, backend behavior |
| Malory browser automation | Playwright via MCP | UI interactions, form submissions, end-to-end flows |

**"Deployed" means:** The `henson-librechat` container running via Docker Compose at `/mnt/storage/projects/Joshua26/docker-compose.yml` on M5, with volumes mounted per the compose configuration.

### 3.3 Mock Boundary

Since all tests are live:
- **Not mocked:** MongoDB, Express routes, filesystem, MCP servers, OpenAI SDK calls
- **Controlled:** Test uses a dedicated test agent and test conversations created via API, cleaned up after test runs
- **LLM calls:** For auto-compact testing, conversations are seeded with enough messages to trigger the threshold — the LLM summarization call is real (uses the agent's configured model)

---

## 4. Test Infrastructure

### 4.1 Isolation Strategy

Tests run against the **production fork deployment** on M5 — there is no separate test instance. Isolation is achieved by:
- Test agents are created with a `[TEST]` prefix and deleted after test runs
- Test conversations are created via API and deleted after test runs
- Test user: `j@rmdev.pro` (existing account)
- No test data persists after a successful run

### 4.2 Configuration

| Setting | Value |
|---------|-------|
| Container | `henson-librechat` on M5 (192.168.1.120) |
| Base URL | `http://192.168.1.120:3080` |
| Auth | JWT token obtained via `/api/auth/login` |
| Compact model | `COMPACT_MODEL` env var (default `gpt-4o-mini`) |
| MCP servers | As configured in container's `librechat.yaml` |
| File browser paths | As configured in `interface.fileBrowserPaths` |

### 4.3 Data Loading

Test data is created programmatically at test start:
1. Authenticate via `/api/auth/login` → JWT token
2. Create test agent with `auto_compact=true, compact_threshold=80` via `POST /api/agents`
3. Create test conversations with seeded messages via direct MongoDB insertion (SSH to M5)
4. After tests: delete test agent, delete test conversations

Idempotency: each test run checks for and cleans up `[TEST]`-prefixed agents/conversations before creating new ones.

### 4.4 Stack Lifecycle

- **Start:** Tests assume the container is already running. Pre-flight check: `GET /api/health` must return 200.
- **Ready detection:** Health endpoint + successful auth login.
- **Teardown:** Test data only — container is not restarted between test runs.
- **Full rebuild:** For fresh-container testing (§13b scope), use: `ssh aristotle9@192.168.1.120 "cd /mnt/storage/projects/Joshua26 && docker compose build henson-librechat && docker compose up -d henson-librechat"`

---

## 5. Coverage Targets

### 5.1 Capability Audit

See §1.2 for the full master capability list (56 capabilities) and §1.3 for coverage status. Current: 1 REAL, 55 NONE. Target: zero NONE for all implemented capabilities (48 of 56 — 8 are Feature 3, not yet implemented).

### 5.2 Coverage by Layer

| Layer | Target |
|-------|--------|
| API endpoints (new) | 100% — POST /compact, GET /browse, GET /browse/content |
| API endpoints (modified) | 100% — PATCH /agents/:id (auto_compact fields) |
| Agent constructor changes | 100% — auto_compact/compact_threshold initialization |
| Context strategy changes | 100% — proactive threshold in handleContextStrategy |
| UI components (new) | 100% via Malory — AutoCompact toggle, File Browser panel |
| UI components (modified) | 100% via Malory — MCP toggle, AgentPanel save, AdvancedPanel |
| Configuration options | 100% — fileBrowser, fileBrowserPaths, auto_compact, compact_threshold |
| Atom/state management | 100% via Malory — mcpDefaultEnabledAtom, mcpNewChatGenAtom |
| /compact slash command | 100% via Malory — interception, API call, toast feedback |

---

## 6. Test Cases by Component

### 6.1 Feature 1: Default-Enabled MCP Toggle

| ID | Scenario | Input | Expected | Condition | Gray-Box |
|----|----------|-------|----------|-----------|----------|
| T-MC-01 | Toggle on | Click switch for a configured server | Server added to localStorage default list | Happy | Check `localStorage['librechat_mcp_default_enabled']` |
| T-MC-02 | Toggle off | Unclick switch for a default-enabled server | Server removed from list | Happy | Check localStorage |
| T-MC-03 | Persist across reload | Toggle on → reload page | Toggle still on | Happy | Check localStorage |
| T-MC-04 | Apply to new chat | Toggle on → click New Chat | Server auto-selected in MCP picker | Happy | Check ephemeralAgent.mcp via Malory |
| T-MC-05 | Don't apply to agent chat | Toggle on → start agent conversation | Agent uses own tools, not defaults | Happy | Container logs: `extractMCPServers` used |
| T-MC-06 | Don't affect existing chat | Toggle on → switch to existing chat | Existing MCP selection unchanged | Happy | |
| T-MC-07 | Unconfigured server filtered | Toggle on server A → remove server A from config → new chat | Server A not selected | Edge | |
| T-MC-08 | No defaults set | No toggles on → new chat | No MCP servers auto-selected | Edge | |
| T-MC-09 | Multiple defaults | Toggle on 3 servers → new chat | All 3 selected | Happy | |
| T-MC-10 | localStorage unavailable | Block localStorage → toggle | Graceful handling (no crash) | Error | |

### 6.2 Feature 2: Auto Compact

| ID | Scenario | Input | Expected | Condition | Gray-Box |
|----|----------|-------|----------|-----------|----------|
| T-CC-01 | Toggle visible in Advanced | Open agent → Advanced panel | Auto Compact switch visible | Happy | |
| T-CC-02 | Toggle on shows threshold | Toggle Auto Compact on | Number input appears, default 80 | Happy | |
| T-CC-03 | Toggle off hides threshold | Toggle off | Number input disappears | Happy | |
| T-CC-04 | Save from Advanced | Toggle on, set 75, save | Fields in MongoDB: auto_compact=true, compact_threshold=75 | Happy | MongoDB query |
| T-CC-05 | Save auto_compact=false | Toggle off, save | auto_compact=false in MongoDB | Happy | MongoDB query |
| T-CC-06 | Threshold at min (10) | Set 10, save | Accepted, persisted | Boundary | MongoDB query |
| T-CC-07 | Threshold at max (99) | Set 99, save | Accepted, persisted | Boundary | MongoDB query |
| T-CC-08 | Threshold below min (9) | Set 9, save | Rejected by Zod, 400 response | Error | |
| T-CC-09 | Threshold above max (100) | Set 100, save | Rejected by Zod, 400 response | Error | |
| T-CC-10 | Reload preserves settings | Save → reload → reopen agent → Advanced | Saved values displayed | Happy | |
| T-CC-11 | New agent gets defaults | Create new agent → Advanced | auto_compact=false, compact_threshold=80 | Happy | |
| T-CC-12 | Proactive compact fires | Agent with auto_compact=true, threshold=80, send enough messages | Summary appears on oldest message | Happy | MongoDB: summary field |
| T-CC-13 | Summary within budget | After T-CC-12 | summaryTokenCount ≤ 5% of maxContextTokens | Happy | MongoDB query |
| T-CC-14 | Agent works after compact | After T-CC-12, send another message | Agent responds coherently | Happy | |
| T-CC-15 | MCP tools work after compact | After T-CC-12, ask agent to use MCP tool | Tool invocation succeeds | Happy | Container logs |
| T-CC-16 | Summary reuse | After T-CC-12, send one more message | No re-summarize (reuses stored summary) | Happy | Container logs: no new API call |
| T-CC-17 | Summarization failure fallback | Agent with invalid model for summary | Falls back to discard strategy | Error | Container logs |
| T-CC-18 | 3 compact cycles | Fill → compact → fill → compact → fill → compact | No state accumulation bugs, context stays usable | Stress | MongoDB: summary field each cycle |

### 6.3 Feature 2: /compact Slash Command

| ID | Scenario | Input | Expected | Condition | Gray-Box |
|----|----------|-------|----------|-----------|----------|
| T-CC-19 | /compact intercepted | Type "/compact", press Enter | Input cleared, no message to LLM | Happy | |
| T-CC-20 | /compact triggers API | Type "/compact" | POST to /api/convos/:id/compact fires | Happy | Network tab or container logs |
| T-CC-21 | /compact feedback | Type "/compact" | Toast notification shown | Happy | |
| T-CC-22 | /compact stores summary | Type "/compact" in conversation with messages | Summary on oldest message | Happy | MongoDB query |
| T-CC-23 | /compact in new chat | Type "/compact" with no conversationId | No crash, no request sent | Edge | |
| T-CC-24 | /compact case insensitive | Type "/COMPACT" | Same as "/compact" | Edge | |
| T-CC-25 | /compact with whitespace | Type "  /compact  " | Intercepted (trim handles it) | Edge | |
| T-CC-26 | /compact with extra text | Type "/compact now" | NOT intercepted (strict equality) — sent to LLM | Edge | |

### 6.4 Feature 3: File Browser (not yet implemented — tests to be written after implementation)

| ID | Scenario | Input | Expected | Condition | Gray-Box |
|----|----------|-------|----------|-----------|----------|
| T-FB-01 | Panel button visible (config on) | `fileBrowser: true` in config | Button in right panel | Happy | |
| T-FB-02 | Panel button hidden (config off) | `fileBrowser: false` in config | No button | Happy | |
| T-FB-03 | Root listing | Open panel | Mount points shown as root nodes | Happy | |
| T-FB-04 | Folder expand | Click folder | Children load | Happy | |
| T-FB-05 | Folder collapse | Click expanded folder | Children hidden | Happy | |
| T-FB-06 | File select | Click file | Highlighted, Insert Path enabled | Happy | |
| T-FB-07 | Insert Path | Select file → click Insert Path | Path in prompt textarea | Happy | |
| T-FB-08 | Folder path insert | Select folder → click Insert Path | Folder path in textarea | Happy | |
| T-FB-09 | Insert Path disabled | No selection | Button gray/disabled | Happy | |
| T-FB-10 | Double-click viewable file | Double-click .jpg | Opens in new tab | Happy | |
| T-FB-11 | Double-click binary | Double-click .zip | Download triggers | Happy | |
| T-FB-12 | Path outside allowed dirs | Request /etc/passwd via API | 403 | Error | |
| T-FB-13 | Path traversal | Request /storage/../etc/passwd | 403 | Error | |
| T-FB-14 | Non-existent path | Request /storage/nonexistent | 404 | Error | |

### 6.5 Infrastructure

| ID | Scenario | Input | Expected | Condition | Gray-Box |
|----|----------|-------|----------|-----------|----------|
| T-IF-01 | Docker build succeeds | `docker compose build henson-librechat` | Exit 0, image created | Happy | |
| T-IF-02 | Fresh container starts | `docker compose up -d` after build | Container healthy | Happy | Health endpoint |
| T-IF-03 | TS packages built from source | Fresh build with no host dist/ | All packages compile | Happy | Container logs during build |
| T-IF-04 | bash available in container | `docker exec ... bash --version` | Bash responds | Happy | |
| T-IF-05 | openssh-client available | `docker exec ... ssh -V` | SSH version output | Happy | |

### 6.6 Cross-Feature Interactions

| ID | Scenario | Input | Expected | Condition | Gray-Box |
|----|----------|-------|----------|-----------|----------|
| T-XF-01 | Agent MCP tools survive compaction | Agent with MCP tools, auto_compact fires | Tools still available post-compact | Happy | Container logs |
| T-XF-02 | /compact on agent conversation | /compact in agent chat | Uses app-level OpenAI key, not agent's | Happy | Container logs |
| T-XF-03 | Default MCP + immediate agent select | Toggle defaults → new chat → select agent | Agent uses own tools (defaults overridden) | Happy | Container logs |

---

## 7. Endpoint and Tool Testing

### 7.1 POST /api/convos/:conversationId/compact

| Scenario | Input | Expected | Type |
|----------|-------|----------|------|
| Happy path | Valid conversationId, authenticated user | 200, summary stored on oldest message | Happy |
| Invalid conversationId | Non-existent ID | 404 | Error |
| Unauthorized | No JWT token | 401 | Error |
| Wrong user | ConversationId owned by different user | 403 | Error |
| Empty conversation | ConversationId with 0-1 messages | Graceful handling (no crash) | Edge |
| Short conversation | ConversationId with < 3 messages | Graceful handling | Edge |

### 7.2 PATCH /api/agents/:id (auto_compact fields)

| Scenario | Input | Expected | Type |
|----------|-------|----------|------|
| Set auto_compact=true | `{ auto_compact: true, compact_threshold: 80 }` | Fields persisted in MongoDB | Happy |
| Set auto_compact=false | `{ auto_compact: false }` | Field persisted, threshold unchanged | Happy |
| Threshold at min | `{ compact_threshold: 10 }` | Accepted | Boundary |
| Threshold at max | `{ compact_threshold: 99 }` | Accepted | Boundary |
| Threshold below min | `{ compact_threshold: 9 }` | Rejected by Zod validation | Error |
| Threshold above max | `{ compact_threshold: 100 }` | Rejected by Zod validation | Error |
| Threshold non-numeric | `{ compact_threshold: "abc" }` | Rejected by Zod validation | Error |
| auto_compact non-boolean | `{ auto_compact: "yes" }` | Rejected by Zod validation | Error |

### 7.3 GET /api/files/browse (Feature 3 — not yet implemented)

| Scenario | Input | Expected | Type |
|----------|-------|----------|------|
| Root listing | `?path=/` | Configured root directories | Happy |
| Valid subdirectory | `?path=/storage/files` | Directory children | Happy |
| Path outside allowed | `?path=/etc/passwd` | 403 | Error |
| Non-existent path | `?path=/storage/nonexistent` | 404 | Error |
| No auth | No JWT | 401 | Error |
| Path traversal | `?path=/storage/../etc/passwd` | 403 | Error |

### 7.4 GET /api/files/browse/content (Feature 3 — not yet implemented)

| Scenario | Input | Expected | Type |
|----------|-------|----------|------|
| Image file | `?path=/storage/files/image.jpg` | 200 + Content-Type: image/jpeg + inline | Happy |
| Text file | `?path=/storage/files/readme.txt` | 200 + Content-Type: text/plain + inline | Happy |
| Binary file | `?path=/storage/files/archive.zip` | 200 + Content-Type: application/zip + attachment | Happy |
| Path outside allowed | `?path=/etc/shadow` | 403 | Error |
| Non-existent file | `?path=/storage/files/nope.txt` | 404 | Error |

---

## 8. Pipeline End-to-End Verification

### 8.1 Auto-Compact Pipeline

The auto-compact feature is a multi-stage pipeline triggered during message processing:

| Stage | Verification | Method |
|-------|-------------|--------|
| 1. Agent constructor reads config | `shouldSummarize`, `compactThreshold` set correctly | Gray-box: container logs |
| 2. Token count exceeds threshold | `handleContextStrategy` triggers summarization path | Gray-box: container logs |
| 3. Messages selected for summarization | Oldest messages (all but newest 10%) identified | Gray-box: log output |
| 4. Summarization call made | OpenAI SDK called with COMPACT_PROMPT | Gray-box: container logs showing API call |
| 5. Summary stored | `summary` field written to oldest message in MongoDB | Gray-box: MongoDB query |
| 6. Subsequent request uses summary | Summary prepended as system message instead of re-summarizing | Gray-box: container logs |

**Completion detection:** Check MongoDB for `summary` field on the target message after sending enough messages to trigger the threshold.

**Performance:** Measure time from threshold-triggering message send to summary storage in MongoDB.

---

## 9. Non-Deterministic System Testing

### 9.1 Behavioral Assertions

The auto-compact summarization and manual /compact both call LLMs. Tests verify behavioral outcomes, not text content:

| Test | Observable Side Effect |
|------|----------------------|
| Auto-compact fires | `summary` field exists on oldest message in MongoDB |
| Summary is within budget | `summaryTokenCount` ≤ 5% of `maxContextTokens` |
| /compact endpoint | `summary` field written, response includes "Compacted N messages" |
| Post-compact conversation works | Agent responds coherently after compaction (no crash, no empty response) |

### 9.2 Quality Evaluation

Not applicable for initial release. Summary quality is verified by:
1. Summary exists (non-empty)
2. Summary token count is within budget
3. Agent still responds coherently after compaction

Future: LLM-as-judge evaluation of summary quality (retains key facts, drops noise).

### 9.3 Agent Tool Testing

| Tool Behavior | Verification |
|--------------|-------------|
| MCP tools work before compaction | Agent uses filesystem/other MCP tools successfully |
| MCP tools work after compaction | Same tools still available and functional post-compact |
| Agent's own MCP tools used (not ephemeral) | Gray-box: verify `extractMCPServers(agent)` used, not `ephemeralAgent.mcp` |

---

## 10. Gray-Box Verification Strategy

### 10.1 Database Queries

| Check | Collection | Query | Columns | Why |
|-------|-----------|-------|---------|-----|
| Auto-compact fields saved | `agents` | `{ _id: agentId }` | `auto_compact`, `compact_threshold` | API response alone doesn't prove MongoDB persistence |
| Summary stored | `messages` | `{ conversationId, messageId }` | `summary`, `summaryTokenCount` | Verify actual DB write, not just API response |
| Agent MCP tools | `agents` | `{ _id: agentId }` | `tools` | Verify tool format matches `toolName_mcp_serverName` |

**Access method:** SSH to M5, `docker exec henson-librechat mongosh` or Python `pymongo` via SSH tunnel.

### 10.2 Container Log Inspection

| Pattern | Meaning | Never Acceptable |
|---------|---------|-----------------|
| `[AgentContext] Applied context to agent` | MCP instructions injected | — |
| `[AgentContext] Failed to apply context` | MCP injection failed | In happy path tests |
| `handleContextStrategy` with summarize | Auto-compact triggered | — |
| Stack traces / unhandled rejections | Crash | Always |

**Access method:** `ssh aristotle9@192.168.1.120 "docker logs henson-librechat --tail 100"`

### 10.3 Filesystem Inspection (Feature 3)

| Check | Path | Why |
|-------|------|-----|
| Browse endpoint returns real files | `/storage/files/` inside container | Verify mount is accessible |
| Path traversal blocked | Attempt `../` in path | Security: must not escape allowed dirs |

---

## 11. Runtime Issue Logging

All findings are logged as GitHub Issues on `rmdevpro/Joshua26`:

| Severity | Criteria | Label |
|----------|----------|-------|
| **Bug** | Assertion failure — system did not do what it should | `bug` |
| **Warning** | Non-fatal but unexpected behavior (degraded response, slow performance) | `investigation` |
| **Performance** | Latency > 30s for compact, > 5s for browse | `performance` |

Threshold for quality: if an agent cannot use its MCP tools after compaction, that is a bug, not a warning.

---

## 12. Hot-Reload Testing

**Excluded.** Per PRD §6: "Upstream's hot-reload. Not our feature."

---

## 13. UI Testing

### 13.1 UI-First Priority

LibreChat is a UI-first system. Backend API tests verify plumbing; Malory browser tests verify the product works. The engineering gate does not pass on API tests alone.

### 13.2 UI Test Scenarios (via Malory)

#### Feature 1: Default-Enabled MCP Toggle

| ID | Scenario | Steps | Expected |
|----|----------|-------|----------|
| UI-F1-01 | Toggle server as default-enabled | Open MCP Settings → find server card → toggle switch | Toggle state persists in localStorage |
| UI-F1-02 | New chat applies defaults | Toggle on → start new non-agent chat | Server auto-selected in MCP picker |
| UI-F1-03 | Untoggle removes default | Toggle off → start new chat | Server NOT selected |
| UI-F1-04 | Defaults survive reload | Toggle on → reload page → check toggle state | Toggle still on |
| UI-F1-05 | Agent chat ignores defaults | Toggle on → start agent chat | Agent uses own MCP tools, not defaults |
| UI-F1-06 | Existing chat not affected | Toggle on → switch to existing chat | Existing chat MCP selection unchanged |

#### Feature 2: Auto Compact UI

| ID | Scenario | Steps | Expected |
|----|----------|-------|----------|
| UI-F2-01 | Auto Compact toggle visible | Open Agent Builder → Advanced | "Auto Compact" toggle visible |
| UI-F2-02 | Toggle on shows threshold | Toggle Auto Compact on | Threshold input appears, default 80 |
| UI-F2-03 | Save auto_compact | Toggle on → set threshold → Save | Fields persisted (verify via API GET) |
| UI-F2-04 | Toggle off hides threshold | Toggle Auto Compact off | Threshold input disappears |
| UI-F2-05 | Save from Advanced panel | Navigate to Advanced → modify → Save | Save succeeds (no stale validation) |
| UI-F2-06 | Threshold validation | Enter 5 (below min) → Save | Validation error or clamped to 10 |
| UI-F2-07 | Reload preserves settings | Save → reload → reopen agent → Advanced | Fields show saved values |

#### Feature 2: /compact Slash Command

| ID | Scenario | Steps | Expected |
|----|----------|-------|----------|
| UI-F2-08 | /compact intercepted | Type "/compact" → press Enter | Input cleared, no message sent to LLM |
| UI-F2-09 | /compact triggers API | Type "/compact" → Enter | POST to /api/convos/:id/compact fires |
| UI-F2-10 | /compact feedback | Type "/compact" → Enter | Toast notification shown |
| UI-F2-11 | /compact in empty chat | Type "/compact" with no conversationId | Graceful handling (no crash) |

#### Feature 3: File Browser Panel (not yet implemented)

| ID | Scenario | Steps | Expected |
|----|----------|-------|----------|
| UI-F3-01 | Panel button visible | Config `fileBrowser: true` → reload | "File Browser" button in right panel |
| UI-F3-02 | Panel button hidden | Config `fileBrowser: false` → reload | No button |
| UI-F3-03 | Panel opens | Click "File Browser" button | Panel renders with root nodes |
| UI-F3-04 | Folder expand | Click folder node | Children load and display |
| UI-F3-05 | Folder collapse | Click expanded folder | Children hidden |
| UI-F3-06 | File select | Click file node | File highlighted, Insert Path activates |
| UI-F3-07 | Insert Path disabled | No selection | Button gray/disabled |
| UI-F3-08 | Insert Path click | Select file → click Insert Path | Path appears in prompt textarea |
| UI-F3-09 | Folder path insert | Select folder → click Insert Path | Folder path appears in prompt textarea |
| UI-F3-10 | Double-click image | Double-click .jpg file | Opens in new browser tab |
| UI-F3-11 | Double-click binary | Double-click .zip file | Download triggers |

### 13.3 Real-World Usability Tests

| Test | Task | Success Criteria |
|------|------|-----------------|
| U-01 | Create an agent with auto-compact, have a long conversation, verify it stays coherent | Agent responds meaningfully after compaction fires |
| U-02 | Use /compact mid-conversation, continue chatting | No errors, agent retains context from summary |
| U-03 | Browse filesystem, insert path, ask agent to read file | Agent uses MCP filesystem tool on the inserted path |
| U-04 | Toggle MCP defaults, start new chat, use MCP tools | Default-enabled tools are available and functional |

---

## 13a. Context Stress Testing

### 13a.1 Progressive Fill

| Threshold | Expected Behavior |
|-----------|------------------|
| 50% | No action |
| 79% | No action (threshold=80) |
| 80% | Auto-compact triggers — summary generated |
| Post-compact (~15%) | Free space restored |

### 13a.2 Pipeline Stage Verification

Per §8.1 — each stage of the auto-compact pipeline verified independently.

### 13a.3 Multiple Cycles

Test must trigger the full compact cycle at least 3 times:
1. Fill to 80% → compact → verify ~15% used
2. Fill to 80% again → compact → verify summary updated
3. Fill to 80% again → compact → verify no state accumulation bugs

### 13a.4 Cold Recall

After each compaction cycle, ask the agent about topics discussed before compaction. Verify the summary retained key facts.

---

## 13b. Fresh-Container Testing

**Scope: limited.** Per PRD §6: "Good practice but heavyweight for a fork. We do verify Dockerfile builds correctly."

Verification:
1. `docker compose build henson-librechat` succeeds (no build errors)
2. `docker compose up -d henson-librechat` starts successfully
3. `GET /api/health` returns 200
4. Auth login succeeds
5. Agent creation with auto_compact fields succeeds (Mongoose schema loaded correctly from fresh build)

---

## 14. Failure Investigation Strategy

### 14.1 Root Cause Before Fix

Every test failure is traced to root cause before any code change. Root cause = exact code path + why it executed + reproduction proof.

### 14.2 Full Analysis Before Action

Read actual error, actual data, actual code path. No summaries or assumptions. For 500 errors: read container logs, find the exception, trace the code.

### 14.3 Second Opinion

For non-trivial bugs, consult a second LLM via 3-CLI pattern with the evidence gathered. Present: failing test, error output, relevant code, proposed root cause.

### 14.4 Never Weaken Tests

If a test fails, the application code has a bug. Fix the application, not the test. No threshold lowering, assertion broadening, skip adding, or "flaky" reclassification.

### 14.5 No Hacks

Fixes must be precise and correct. No regex workarounds, no silent data loss, no behavioral changes to unrelated functionality.

### 14.6 Every Fix Verified

Deploy fix → run failing test → confirm fixed → run full suite → confirm no regressions.

All bug fixes follow PROC-01: 3 CLIs diagnose → fix → 3 CLIs review → verify via Malory → close issue.

---

## 15. What Is Not Tested

| Area | Type | Justification |
|------|------|---------------|
| Upstream LibreChat features | Exclusion | Not our code. Upstream's responsibility. |
| Unit tests (Jest) | Blocker | Workspace dependency resolution issues prevent reliable Jest execution. Filed as known constraint, not fixable without upstream changes. |
| LLM summary quality | Exclusion | Initial release verifies summary exists and is within token budget. Quality evaluation deferred to future iteration. |
| Hot-reload | Exclusion | Upstream feature, not modified by fork. |
| Performance benchmarks | Exclusion | No SLA defined. We measure compact latency but don't gate on it. |
| Multi-user concurrency | Exclusion | Single-user deployment. No concurrent usage expected. |

---

## 16. Traceability Matrix

All test IDs map to §6 test cases and §13 UI scenarios. Capabilities from §1.2 are covered by one or more test cases.

| ID | Scenario | Test File | Layer | Status | Last Run | Result | Capabilities |
|----|----------|-----------|-------|--------|----------|--------|-------------|
| T-MC-01 | Toggle on default-enabled | — | Malory | Not started | — | — | MC-01 |
| T-MC-02 | Toggle off | — | Malory | Not started | — | — | MC-01 |
| T-MC-03 | Persist across reload | — | Malory | Not started | — | — | MC-02 |
| T-MC-04 | Apply to new chat | — | Malory | Not started | — | — | MC-03, MC-04, MC-05 |
| T-MC-05 | Agent ignores defaults | — | Malory | Not started | — | — | MC-06, MC-09, MC-12 |
| T-MC-06 | Existing chat unaffected | — | Malory | Not started | — | — | MC-07 |
| T-MC-07 | Unconfigured server filtered | — | Malory | Not started | — | — | MC-04 |
| T-MC-08 | No defaults set | — | Malory | Not started | — | — | MC-04 |
| T-MC-09 | Multiple defaults | — | Malory | Not started | — | — | MC-04 |
| T-MC-10 | localStorage unavailable | — | Malory | Not started | — | — | MC-01, MC-02 |
| T-CC-01 | Auto Compact toggle visible | — | Malory | Not started | — | — | CC-01, CC-03 |
| T-CC-02 | Toggle on shows threshold | — | Malory | Not started | — | — | CC-02 |
| T-CC-03 | Toggle off hides threshold | — | Malory | Not started | — | — | CC-02 |
| T-CC-04 | Save auto_compact=true | — | Malory+Python | Not started | — | — | CC-04, CC-07, CC-08 |
| T-CC-05 | Save auto_compact=false | — | Malory+Python | Not started | — | — | CC-04, CC-07, CC-08 |
| T-CC-06 | Threshold min (10) | — | Python | Not started | — | — | CC-07 |
| T-CC-07 | Threshold max (99) | — | Python | Not started | — | — | CC-07 |
| T-CC-08 | Threshold below min | — | Python | Not started | — | — | CC-07 |
| T-CC-09 | Threshold above max | — | Python | Not started | — | — | CC-07 |
| T-CC-10 | Reload preserves | — | Malory | Not started | — | — | CC-05 |
| T-CC-11 | New agent defaults | — | Malory | Not started | — | — | CC-06 |
| T-CC-12 | Proactive compact fires | — | Python+Malory | Not started | — | — | CC-13, CC-14, CC-16, CC-17 |
| T-CC-13 | Summary within budget | — | Python | Not started | — | — | CC-24, CC-18 |
| T-CC-14 | Agent works after compact | — | Malory | Not started | — | — | CC-14 |
| T-CC-15 | MCP tools after compact | — | Malory | Not started | — | — | XF-02 |
| T-CC-16 | Summary reuse | — | Python | Not started | — | — | CC-15 |
| T-CC-17 | Summarization failure | — | Python | Not started | — | — | CC-27 |
| T-CC-18 | 3 compact cycles | — | Python+Malory | Not started | — | — | CC-14, CC-18 |
| T-CC-19 | /compact intercepted | — | Malory | Not started | — | — | CC-19 |
| T-CC-20 | /compact triggers API | — | Malory | Not started | — | — | CC-20, CC-23 |
| T-CC-21 | /compact feedback | — | Malory | Not started | — | — | CC-23 |
| T-CC-22 | /compact stores summary | `scripts/test_compact.py` | Python | Not started | — | — | CC-20, CC-18 |
| T-CC-23 | /compact in new chat | — | Malory | Not started | — | — | CC-19 |
| T-CC-24 | /compact case insensitive | — | Malory | Not started | — | — | CC-19 |
| T-CC-25 | /compact with whitespace | — | Malory | Not started | — | — | CC-19 |
| T-CC-26 | /compact with extra text | — | Malory | Not started | — | — | CC-19 |
| T-FB-01 | Panel button visible | — | Malory | Not started | — | — | FB-01, FB-05, FB-06 |
| T-FB-02 | Panel button hidden | — | Malory | Not started | — | — | FB-05 |
| T-FB-03 | Root listing | — | Malory | Not started | — | — | FB-01, FB-03 |
| T-FB-04 | Folder expand | — | Malory | Not started | — | — | FB-02 |
| T-FB-05 | Folder collapse | — | Malory | Not started | — | — | FB-02 |
| T-FB-06 | File select | — | Malory | Not started | — | — | FB-02, FB-07 |
| T-FB-07 | Insert Path | — | Malory | Not started | — | — | FB-07 |
| T-FB-08 | Folder path insert | — | Malory | Not started | — | — | FB-07 |
| T-FB-09 | Insert Path disabled | — | Malory | Not started | — | — | FB-07 |
| T-FB-10 | Double-click viewable | — | Malory | Not started | — | — | FB-04 |
| T-FB-11 | Double-click binary | — | Malory | Not started | — | — | FB-04 |
| T-FB-12 | Path outside allowed | — | Python | Not started | — | — | FB-08 |
| T-FB-13 | Path traversal | — | Python | Not started | — | — | FB-08 |
| T-FB-14 | Non-existent path | — | Python | Not started | — | — | FB-03 |
| T-IF-01 | Docker build succeeds | — | SSH | Not started | — | — | IF-01, IF-02 |
| T-IF-02 | Fresh container starts | — | SSH | Not started | — | — | IF-02 |
| T-IF-03 | TS built from source | — | SSH | Not started | — | — | IF-01 |
| T-IF-04 | bash available | — | SSH | Not started | — | — | IF-03 |
| T-IF-05 | openssh-client available | — | SSH | Not started | — | — | IF-03 |
| T-XF-01 | MCP tools survive compaction | — | Malory | Not started | — | — | XF-01, XF-02 |
| T-XF-02 | /compact on agent convo | — | Malory | Not started | — | — | XF-03 |
| T-XF-03 | Default MCP + agent select | — | Malory | Not started | — | — | XF-04 |
| U-01 | Long convo with auto-compact | — | Malory | Not started | — | — | CC-14, CC-16 |
| U-02 | /compact mid-conversation | — | Malory | Not started | — | — | CC-20 |
| U-03 | File browser → agent reads | — | Malory | Not started | — | — | FB-07 |
| U-04 | MCP defaults → use tools | — | Malory | Not started | — | — | MC-04, MC-12 |
