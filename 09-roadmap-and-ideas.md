# Roadmap & Ideas

**Last Generated:** 2026-03-17

---

## Priority Backlog

### Critical — Must Do Before Go-Live

| Idea | Summary |
|------|---------|
| **Pre-Production Security Audit** | Full 9-phase security audit of the company VM before `ese.eprom.com.eg` goes live. 60+ checklist items covering OS, firewall, SSL, Docker, auth, APIs, database, secrets, and monitoring. All critical/high findings must be resolved first. |

### High Priority

| Idea | Summary |
|------|---------|
| **AI Credits System** | Metered AI credit system: 1 unit = 10K tokens. Basic: 10 trial units, Professional: 50/month, Enterprise: 500/month. Users see a percentage gauge, admins can grant top-ups, monthly credits consumed first before top-ups. Covers DB schema changes, middleware enforcement, chat token deduction, frontend gauge widget, and admin panel management. Key for revenue control and AI cost management. |

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

---

## Implemented Features (Sessions 27–53)

| Feature | Session | Summary |
|---------|---------|---------|
| **Heater Process Schematic** | 53 | Professional SVG schematic of fired heater with 8 KPI annotations (Stack Temp, Flue Gas Flow, Thermal Eff, Fuel Eff, Excess Air, GHG Rate, Absorbed Duty, Radiant/Conv). Count-up animations with ease-out cubic, animated flow arrows, responsive at 3 breakpoints. Replaced unused Trends inputs with 4 user-configurable Design Data fields. |
| **AI Chatbot Consistency** | 51–52 | Unified all 5 AI chatbots: standardized two-drops SVG icons (blue `#1565C0` + green `#4CAF50`), white headers, FAB dismiss on open, mobile bottom sheet (3-state), session-based history clearing, onboarding tips (4 visits), dynamic dashboard system prompt from pillars config, 70 Playwright tests. |
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
| Company VM Migration | Complete | 16/16 verification checks passed, all 6 containers running, 130/130 Playwright tests passed |
| AI Credits System | Not Started | High priority — needed for cost control and subscription monetization |
| Chatbot Consistency | Complete | All 5 chatbots unified (Sessions 51-52) |
| Optimizer V2 | Complete | One-click analysis, paste from Excel, feature response charts (Session 50) |
| 7-Pillar Dashboard | Complete | Iframe integration, sidebar, collapsible sections (Sessions 48-49) |
