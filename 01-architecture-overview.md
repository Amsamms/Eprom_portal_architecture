# Architecture Overview

**Last Generated:** 2026-03-14

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
└──────────────┬───────────────────────────────────────────┘
               │ HTTP (internal, 127.0.0.1 only)
               ▼
┌──────────────────────────────────────────────────────────┐
│  Docker Compose v2 (6 containers on eprom_net bridge)    │
│                                                          │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐         │
│  │ portal     │  │ heater     │  │ pump       │         │
│  │ :3000      │  │ :3001      │  │ :3005      │         │
│  │ Next.js    │  │ Express.js │  │ FastAPI    │         │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘         │
│        │               │               │                │
│  ┌────────────┐  ┌────────────┐                         │
│  │ massmole   │  │ optimizer  │                         │
│  │ :3006      │  │ :3007      │                         │
│  │ Express.js │  │ Express+PHP│                         │
│  └─────┬──────┘  └─────┬──────┘                         │
│        │               │                                │
│        └───────┬───────┘                                │
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
| `eprom_portal` | 3000 | Next.js 14.2.5 (App Router) | Auth, dashboard, admin panel, API gateway |
| `eprom_heater` | 3001 | Express.js (Node 20) | Fired heater calculations (406 formulas) |
| `eprom_pump` | 3005 | FastAPI (Python 3.11) | Pump efficiency calculations (53 formulas) |
| `eprom_massmole` | 3006 | Express.js (Node 20) | Mass/mole conversion (160 compounds) |
| `eprom_optimizer` | 3007 | Express.js + PHP | ML feature importance & optimization |

All application containers bind to `127.0.0.1` only — they are not directly accessible from the internet. Only Nginx exposes ports 80/443.

---

## Nginx Routing Table

| URL Path | Proxied To | Notes |
|----------|-----------|-------|
| `/` | `http://127.0.0.1:3000` | Portal (catch-all) |
| `/apps/heater/` | `http://127.0.0.1:3001/` | Heater app |
| `/apps/pump/` | `http://127.0.0.1:3005/` | Pump app |
| `/apps/massmole/` | `http://127.0.0.1:3006/` | MassMole app |
| `/apps/optimizer/` | `http://127.0.0.1:3007/` | Optimizer (20M upload limit) |
| `/reports/` | Filesystem | Static HTML reports (Nginx direct) |

Security headers applied globally: `X-Frame-Options: SAMEORIGIN`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`. The `X-Powered-By` header is stripped.

---

## Two VMs

| Property | EC2 (Staging) | Company KVM (Production) |
|----------|--------------|-------------------------|
| **IP** | 18.198.1.231 | 192.168.50.202 |
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

---

## Tech Stack

| Layer | Technology | Version/Notes |
|-------|-----------|--------------|
| Portal Frontend | Next.js | 14.2.5, App Router, Tailwind CSS |
| App Frontends | Vanilla JS | Thin clients, no framework, Bootstrap 5 |
| Charts | ApexCharts (Heater), Plotly (Pump), Recharts (Portal) | |
| Authentication | JWT + httpOnly cookies | jose (portal), jsonwebtoken (heater/massmole), python-jose (pump) |
| Session Management | PostgreSQL sessions table | Token hash stored, validated on every API call |
| AI | Anthropic Claude SDK | claude-haiku-4-5-20251001 (default), Sonnet/Opus via secret selector |
| Email | Nodemailer | Gmail App Password, CSS-only HTML templates |
| Database | PostgreSQL 16 | Alpine image, Docker volume persistence |
| Pump Backend | Python 3.11 + FastAPI + uvicorn | numpy, scipy, plotly, asyncpg |
| Optimizer Backend | PHP | DataPreprocessor, CrossValidator, genetic algorithm |
| Containers | Docker Compose v2 | 6 services on bridge network |
| Web Server | Nginx | System service, reverse proxy + static files |
| SSL | Let's Encrypt (Certbot) | Auto-renewal on staging |

---

## Key Architectural Decisions

1. **Server-side computation:** All formulas, engineering calculations, and AI logic run exclusively on the server. Zero business logic in browser JavaScript. This protects EPROM's intellectual property (the expert assumptions and formula calibrations).

2. **Shared JWT authentication:** All apps share a single JWT secret and validate against the portal's PostgreSQL sessions table. A user logs in once at the portal and gets seamless access to all permitted apps via a shared `eprom_token` httpOnly cookie.

3. **Microservice boundaries:** Each app is a standalone Docker container with its own server, routes, and engine. They share only the database and JWT secret. This allows independent deployment and scaling.

4. **Thin client pattern:** App frontends are vanilla JS SPAs that call server APIs. They render results and charts but contain no calculation logic. The heater has 406 formulas — all 406 run server-side.

5. **AI tool-use pattern:** The AI chatbot uses Claude's tool-use capability to interact with each app's engine. The AI calls server-side tools (e.g., `run_calculation`, `fill_input_fields`) and returns structured action objects that the frontend renders (animated cursor, field filling, chart updates).
