# Architecture Overview

**Last Generated:** 2026-03-26

---

## System Topology

```
┌──────────────────────────────────────────────────────────┐
│  User's Browser                                          │
│  (Mobile or Desktop)                                     │
└──────────────┬───────────────────────────────────────────┘
               │ HTTPS (port 443)
               ▼
┌──────────────────────────────────────────────────────────┐
│  Nginx (System Service)                                  │
│  - TLS termination (Let's Encrypt)                       │
│  - Reverse proxy to Docker containers                    │
│  - Security headers (HSTS, X-Frame-Options, etc.)        │
│  - Static report serving (/reports/)                     │
│  - Iframe proxy paths (/_proxy/*)                        │
└──────────────┬───────────────────────────────────────────┘
               │ HTTP (internal, 127.0.0.1 only)
               ▼
┌──────────────────────────────────────────────────────────┐
│  Docker Compose v2 (10 containers on eprom_net bridge)   │
│                                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐         │
│  │ portal     │  │ heater     │  │ pump       │         │
│  │ :3000      │  │ :3001      │  │ :3005      │         │
│  │ Next.js    │  │ Express.js │  │ FastAPI    │         │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘         │
│        │               │               │                │
│  ┌────────────┐  ┌──────────────────┐                   │
│  │ massmole   │  │ 4 × optimizer    │                   │
│  │ :3006      │  │ general  :3007   │                   │
│  │ Express.js │  │ energy   :3008   │                   │
│  └─────┬──────┘  │ distill. :3009   │                   │
│        │         │ flare    :3010   │                   │
│        │         │ Express+PHP each │                   │
│        │         └────────┬─────────┘                   │
│        └───────┬──────────┘                             │
│                ▼                                        │
│  ┌──────────────────────┐                               │
│  │ db (PostgreSQL 16)   │                               │
│  │ :5432                │                               │
│  │ eprom_db             │                               │
│  └──────────────────────┘                               │
└──────────────────────────────────────────────────────────┘
```

---

## Container Table

| Container | Port | Technology | Role |
|-----------|------|-----------|------|
| `eprom_db` | 5432 | PostgreSQL 16 (Alpine) | Shared database for all services |
| `eprom_portal` | 3000 | Next.js 14.2.5 (App Router) | Auth, dashboard, admin panel, API gateway, Dashboard AI Companion |
| `eprom_heater` | 3001 | Express.js (Node 20) | Fired heater calculations (406 formulas) |
| `eprom_pump` | 3005 | FastAPI (Python 3.11) | Pump efficiency calculations (53 formulas) |
| `eprom_massmole` | 3006 | Express.js (Node 20) | Mass/mole conversion (160 compounds) |
| `eprom_optimizer_general` | 3007 | Express.js + PHP | General-purpose ML feature importance & optimization |
| `eprom_optimizer_energy` | 3008 | Express.js + PHP | Energy/power plant domain optimizer |
| `eprom_optimizer_distillation` | 3009 | Express.js + PHP | Distillation column domain optimizer |
| `eprom_optimizer_flare` | 3010 | Express.js + PHP | Flare gas recovery domain optimizer |

All application containers bind to `127.0.0.1` only — they are not directly accessible from the internet. Only Nginx exposes ports 80/443. The old single `optimizer/` directory is deprecated — the 4 domain-specific optimizer apps replaced it in Session 55.

---

## Nginx Routing Table

| URL Path | Proxied To | Notes |
|----------|-----------|-------|
| `/` | `http://127.0.0.1:3000` | Portal (catch-all) |
| `/apps/heater/` | `http://127.0.0.1:3001/` | Heater app (direct access) |
| `/apps/pump/` | `http://127.0.0.1:3005/` | Pump app (direct access) |
| `/apps/massmole/` | `http://127.0.0.1:3006/` | MassMole app (direct access) |
| `/apps/optimizer/` | `http://127.0.0.1:3007/` | Optimizer (20M upload limit) |
| `/_proxy/heater/` | `http://127.0.0.1:3001/` | Heater via iframe (portal sidebar visible) |
| `/_proxy/pump/` | `http://127.0.0.1:3005/` | Pump via iframe |
| `/_proxy/massmole/` | `http://127.0.0.1:3006/` | MassMole via iframe |
| `/_proxy/optimizer/` | `http://127.0.0.1:3007/` | Optimizer via iframe |
| `/reports/` | Filesystem | Static HTML reports (Nginx direct) |

Security headers applied globally: `X-Frame-Options: SAMEORIGIN`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`. The `X-Powered-By` header is stripped.

### Iframe Integration Pattern

Apps can be accessed two ways:
1. **Direct:** `/apps/heater/` — full standalone app with its own header/navigation
2. **Iframe:** `/view/heater` — Next.js page wraps the app in an iframe via `/_proxy/heater/`, adding the portal sidebar. The app detects iframe embedding and hides its own navigation (header, breadcrumbs) while keeping the AI chat FAB visible.

---

## Two VMs

| Property | EC2 (Staging) | Company KVM (Production) |
|----------|--------------|-------------------------|
| **IP** | 18.198.1.231 | 192.168.240.3 |
| **URL** | https://eprom-portal.xyz | https://ese.eprom.com.eg (pending) |
| **OS** | Ubuntu 24.04 LTS | Ubuntu 24.04.4 LTS |
| **CPU** | 2 vCPU (t3.micro) | 8 cores (KVM) |
| **RAM** | 1 GB + 2 GB swap | 24 GB + 4 GB swap |
| **Disk** | 30 GB gp3 | 80 GB |
| **Docker** | 28.2.2 | 29.3.0 |
| **Compose** | v2 plugin | v5.1.0 |
| **SSL** | Let's Encrypt (auto-renew) | Pending |
| **Resource limits** | None (small VM) | docker-compose.override.yml (19 GB / 8 CPU) |
| **Backups** | Manual | Daily cron at 2 AM, 7-day retention |
| **Playwright tests** | Ongoing | 1,437/1,437 passed (Session 55) — 99.5% pass rate |

---

## Tech Stack

| Layer | Technology | Version/Notes |
|-------|-----------|--------------|
| Portal Frontend | Next.js | 14.2.5, App Router, Tailwind CSS |
| App Frontends | Vanilla JS | Thin clients, no framework, Bootstrap 5 |
| Charts | ApexCharts (Heater), Plotly (Pump), Chart.js (Optimizer), Recharts (Portal) | |
| Authentication | JWT + httpOnly cookies | jose (portal), jsonwebtoken (heater/massmole), python-jose (pump) |
| Session Management | PostgreSQL sessions table | Token hash stored, validated on every API call |
| AI | Anthropic Claude SDK | claude-haiku-4-5-20251001 (default), Sonnet/Opus via secret selector |
| Dashboard AI | Anthropic REST API | claude-haiku-4-5-20251001, dynamic system prompt from pillars config |
| Email | Nodemailer | Gmail App Password, CSS-only HTML templates |
| Database | PostgreSQL 16 | Alpine image, Docker volume persistence |
| Pump Backend | Python 3.11 + FastAPI + uvicorn | numpy, scipy, plotly, asyncpg |
| Optimizer Backend | PHP | DataPreprocessor, CrossValidator, genetic algorithm, Chart.js |
| Containers | Docker Compose v2 | 10 services on bridge network (db + portal + heater + pump + massmole + 4 optimizers) |
| Web Server | Nginx | System service, reverse proxy + static files |
| SSL | Let's Encrypt (Certbot) | Auto-renewal on staging |

---

## Key Architectural Decisions

1. **Server-side computation:** All formulas, engineering calculations, and AI logic run exclusively on the server. Zero business logic in browser JavaScript. This protects EPROM's intellectual property (the expert assumptions and formula calibrations).

2. **Shared JWT authentication:** All apps share a single JWT secret and validate against the portal's PostgreSQL sessions table. A user logs in once at the portal and gets seamless access to all permitted apps via a shared `eprom_token` httpOnly cookie.

3. **Microservice boundaries:** Each app is a standalone Docker container with its own server, routes, and engine. They share only the database and JWT secret. This allows independent deployment and scaling.

4. **Thin client pattern:** App frontends are vanilla JS SPAs that call server APIs. They render results and charts but contain no calculation logic. The heater has 406 formulas — all 406 run server-side.

5. **AI tool-use pattern:** The AI chatbot uses Claude's tool-use capability to interact with each app's engine. The AI calls server-side tools (e.g., `run_calculation`, `fill_input_fields`) and returns structured action objects that the frontend renders (animated cursor, field filling, chart updates).

6. **Iframe integration:** Apps are embedded within the portal via iframes using `/_proxy/` Nginx paths. The portal's sidebar and navigation remain visible while the app runs in an iframe. Apps detect embedding via `window.parent !== window` and hide their own navigation to avoid duplication.

7. **7-pillar dashboard:** The dashboard organizes all apps (live and coming-soon) into 7 strategic pillars, each with collapsible sections. A centralized `pillars-config.js` defines all app metadata, pillars, and coming-soon status.

8. **AI Credits metering:** A credit system (1 unit = 10K tokens) gates AI chat access across all apps. Credits are checked pre-chat via `requireAIAccess()` middleware and deducted post-chat. Three tiers: Basic (10 trial), Professional (50/month), Enterprise (500/month).

9. **SMTP Email Relay:** The company VM firewall blocks outbound SMTP. A relay route (`/api/internal/relay-email`) on the portal accepts email data over HTTPS and sends via Gmail SMTP on EC2, secured with an API key and rate limiting.
