# Roadmap & Ideas

**Last Generated:** 2026-03-26

---

## Priority Backlog

### Critical — Must Do Before Go-Live

| Idea | Summary |
|------|---------|
| **Pre-Production Security Audit** | Full 9-phase security audit. **13/14 checks pass** (Session 55). Only Docker-runs-as-root remains. Mostly complete — low priority remaining item. |

### High Priority

| Idea | Summary |
|------|---------|
| **API Rate Limit Queue** | Haiku 4.5 rate limit mitigation for 200 users. Currently at Tier 1 (50 RPM). Tier 2 ($22 more deposit) solves 95% of the problem. Plus: module-scope client, SDK fix, timeouts, token buckets, prompt caching. |

### Medium Priority

| Idea | Summary |
|------|---------|
| **PDF Report Export** | Branded EPROM PDF export using jsPDF (client-side). Phase 1 covers Heater + MassMole. Intended as a credits-based premium feature (Professional tier and above). Full content: KPIs + Charts + All Inputs + All Outputs. Depends on the AI credits system being implemented first. |

### Low Priority

| Idea | Summary |
|------|---------|
| **Saved Calculations** | Save, name, and revisit past calculations per user. Currently all results are lost on page refresh. |
| **Bulk Calculations** | Upload CSV of multiple input sets, get batch results. Enables parametric studies. |
| **Arabic Language Support** | Arabic RTL interface — UI labels, buttons, navigation. The AI already responds in Arabic when asked. |
| **User Onboarding Tour** | Full guided tour with step-by-step overlays per app. Basic onboarding tips (4 visits, auto-dismiss) are already implemented. |
| **Comparison Mode** | Side-by-side comparison of 2–3 saved scenarios with highlighted differences. |
| **AI Chat Export** | Export AI chat conversation as PDF or Markdown for project documentation. |
| **Webhook/Email Notifications** | Email alerts for threshold breaches, new signups, etc. |
| **PWA (Progressive Web App)** | Add manifest + service worker for "install to home screen" and offline caching. |
| **Favicon & Meta Tags** | Add favicon.ico, apple-touch-icon, and proper meta tags (og:image, description). Currently 404 on `/favicon.ico`. |
| **Accessibility Audit** | Password mismatch fields lack `aria-invalid`. Chat panel needs better ARIA roles. Keyboard navigation between form sections. Found via Session 54 E2E testing. |

---

## Implemented Features (Sessions 27–57)

| Feature | Session | Summary |
|---------|---------|---------|
| **EPROM Tears Logo** | 57 | Replaced inline SVG drops with official EPROM tears PNG logo across all 8 apps (Dashboard + 7 app bots). 3 icon locations per app (FAB, header, avatar). 16 files modified. Deployed to EC2 + KVM. |
| **Optimizer Branch Integration** | 56 | Two-flow UI (Paste from Excel OR Upload File), welcome banner, sample dataset loader, clear & start over, auto-execution, feature normalization, 50K iterations. AI now correctly reports optimization results. |
| **SMTP Email Relay** | 56 | HTTPS relay for company VM (firewall blocks outbound SMTP). New `/api/internal/relay-email` route secured with API key + rate limiting. `dispatchEmail()` function auto-selects relay vs direct SMTP. |
| **User Deletion Fix** | 56 | Admin delete was silently failing — 17 FK constraints. DELETE handler now cleans all dependent tables. `validateSession()` checks `users.is_active`. |
| **Four Optimizer Apps** | 55 | Split single optimizer into 4 independent apps (General, Energy, Distillation, Flare). Each has own Docker container, AI chatbot with domain-specific system prompt, DB tables, admin log page. 200 E2E tests, 99.5% pass. |
| **Production Go-Live** | 55 | Full go-live checklist (8/9 phases). 1,437 E2E tests on production (99.5%). Security audit 13/14 pass. Nginx hardened. React hydration fix. Performance: 223ms page load. |
| **Comprehensive E2E Testing** | 54 | 1,400 Playwright MCP tests across 27 phases on staging, 97.9% pass rate. |
| **AI Credits System** | 53 | Metered credits: 1 unit = 10K tokens. DB tables, middleware, post-chat deduction, dashboard gauge, admin credits tab, grant modal, transaction history. 17 users auto-provisioned. Model selector removed. |
| **Heater Process Schematic** | 53 | Professional SVG schematic of fired heater with 8 KPI annotations (Stack Temp, Flue Gas Flow, Thermal Eff, Fuel Eff, Excess Air, GHG Rate, Absorbed Duty, Radiant/Conv). Count-up animations with ease-out cubic, animated flow arrows, responsive at 3 breakpoints. Replaced unused Trends inputs with 4 user-configurable Design Data fields. |
| **AI Chatbot Consistency** | 51–52 | Unified all 5 AI chatbots: standardized icons, white headers, FAB dismiss on open, mobile bottom sheet (3-state), session-based history clearing, onboarding tips (4 visits), dynamic dashboard system prompt from pillars config, 70 Playwright tests. |
| **Optimizer V2** | 50 | One-click analysis (preprocessing → importance → optimization in one call), paste from Excel input (with auto-delimiter and auto-transpose detection), feature response charts (Chart.js, 50-step sweep per feature), new `one_click_analysis` AI tool (now 9 tools total). |
| **Coming Soon Apps + Dashboard Overhaul** | 49 | 10 new coming-soon apps (6 equipment + 4 process tools), 7-pillar renaming, collapsible sidebar (DashboardShell component), collapsible dashboard pillar sections, heater action bar fix for iframe mode. |
| **Dashboard Redesign** | 48 | 7-pillar dashboard structure, universal sidebar with pillar navigation, iframe app integration (`/view/{slug}` + `/_proxy/` Nginx paths), Dashboard AI Companion (EpromCompanion.js rewrite to 270 lines), hide app nav in iframe mode. |
| **Login/Signup Refinements** | 47 | "Request Access" → "Create Account", removed "Engineering Portal" subtitle, lightened login card, browser autofill fix. |
| **KVM Deploy + Testing** | 46 | 130/130 Playwright tests on Company KVM, logo fix (local SVG replacing external URLs), report download fix (iframe sandbox `allow-downloads`), database migration sync. |
| **Admin Analytics Dashboard** | 45 | Full overhaul: global admin stats (Total Users, Active Sessions, Total Logins, Total Activity), session lifecycle tracking (soft-delete with `ended_at` + `end_reason`), user journey timeline, admin-only activity log with User column + dynamic filters (user/app/action/date), clickable session links, Top Active Users table. |
| **AI Token Optimization** | 44 | Two strategies: `cleanForClaude()` trims tool output before sending, `summarizeOldToolResults()` compresses history. Heaviest pump conversation: 92K → 13K tokens (86% reduction). All 4 apps. Quality improved. |
| **System Prompt Regression Audit** | 42 | Reverted session 32's damaging "brief summary" rule in all 4 system prompts. Restored original Important Rules, multi-case examples, and error recovery instructions. |
| **Clear Chat Button** | 41 | Trash-icon button in chat panel header. Clears DOM, removes localStorage history, resets conversationId. All 4 apps. |
| **Cross-App Visual Consistency** | 40 | Unified all 4 apps into one light theme: gradient headers, Inter font, EPROM design tokens, optimizer dark→light migration. 6 phases. |
| **Heater Video Removal** | 39 | Removed 17.8 MB of background videos + audio + ember particles. Kept refinery-fire.jpg as static background. Zero video/audio requests on load. |
| **Mobile-Friendly Redesign** | 36 | Full mobile redesign across all 5 apps. 6 phases + 39/39 Playwright tests passed. Fixed cross-app chat history leak from shared localStorage key collision. |
| **Post-Calc KPI Summary** | 34 | KPI summary card after calculation. Pump defaults corrected (static_head 15→60, elevation_diff 5→10). |
| **Chatbot Visual Actions Audit** | 32–33 | 8 new features: stop generation, Escape/Ctrl+/ shortcuts, textarea auto-resize, copy code, timestamps, chat history persistence, optimizer model selector. |
| **Shared Chatbot Library** | 32 | Extracted common chatbot code into `shared/chat-base.js` + `visual-actions-base.js`. Migrated all 4 apps to config pattern. Fixed async visual action bug. |
| **AI Chat Layout Consistency** | 29 | Standardized AI chat side-panel layout, toggle buttons, and content shift across all 4 apps. |
| **Optimizer Preprocessing Upgrade** | 31 | DataPreprocessor + CrossValidator classes (6 phases, 11 files). Auth redirect fix across all 4 apps. |

---

## Business Milestones

| Milestone | Status | Notes |
|-----------|--------|-------|
| EGYPES Exhibition Demo | Pending | Requires stable prototype with heater + soft sensors + carbon footprint (30 March – 1 April 2026) |
| Domain `ese.eprom.com.eg` Live | Pending | Blocked on DNS coordination with Telecom Egypt + SSL setup on company VM |
| Pre-Production Security Audit | Pending | Must complete before domain goes live |
| Company VM Migration | Complete | 16/16 verification checks, 9 containers running, 1,437 E2E tests (99.5%) |
| AI Credits System | Complete | Session 53 — metered usage, 3 tiers, gauge widget, admin panel, 17 users provisioned |
| Four Optimizer Apps | Complete | Session 55 — 4 independent apps from Amr's branch, 200 E2E tests (99.5%) |
| EPROM Tears Logo | Complete | Session 57 — official PNG logo across all 8 chatbots |
| Production Go-Live | Complete | Session 55 — go-live checklist 8/9 phases, security audit 13/14, Nginx hardened |
| Chatbot Consistency | Complete | All 8 chatbots unified (Sessions 51-52, 57) |
| SMTP Email Relay | Complete | Session 56 — HTTPS relay for company VM |
| 7-Pillar Dashboard | Complete | Iframe integration, sidebar, collapsible sections (Sessions 48-49) |
