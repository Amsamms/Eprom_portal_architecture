# Decisions & Constraints

**Last Generated:** 2026-03-14

---

## IT Meeting Outcomes

A formal IT strategy meeting established the deployment framework for the "Smart Engine" AI project. The meeting balanced the engineering team's need for rapid development velocity against the IT department's mandate for corporate security.

---

## The Six Pillars of Transformation

The "Smart Engine" project represents a transition from manual, Excel-based calculations to an integrated AI-driven platform. Six strategic pillars define the scope:

| Pillar | Strategic Value |
|--------|----------------|
| **Equipment Analysis (Fired Heaters)** | Eliminates human error by codifying 20 years of institutional knowledge into real-time thermal and mechanical calculations |
| **Soft Sensors (ML)** | Operational continuity — predicts critical outcomes (98% accuracy) during instrumentation failure using historical data relationships |
| **Automatic Carbon Footprint** | Regulatory compliance — automates labor-intensive environmental reporting to meet international standards |
| **Process Safety** | Proactive risk mitigation — uses AI to predict hazards before escalation |
| **Functional Safety** | Standardized reliability — ensures automated compliance and reliability reporting |
| **Operational Standards** | Global competitiveness — embeds international operating standards directly into software logic |

---

## Infrastructure Decisions

### Server Architecture
- **Hardware:** A dedicated physical server (32 GB RAM) from the company warehouse
- **Virtualization:** KVM virtual machine running Docker + Nginx
- **Network placement:** DMZ (Demilitarized Zone) — the server sees only the internet, protecting the internal corporate network from potential compromises

### Access Control
- **SSH with private keys only** — the approved method for remote administration
- **AnyDesk explicitly rejected** — forbidden by the Ministry, lacks encryption. The IT department described it as "standing in the street talking" (بتقف في الشارع بتتكلم)
- **IT manages SSH** — developers should not modify SSH configuration or harden it independently

### Domain & DNS
- **Subdomain:** `ese.eprom.com.eg` (or similar variant) — to be hosted on the company domain
- **DNS coordination:** Requires coordination with WE (Telecom Egypt) for IP mapping and DNS propagation. This is identified as a high-risk administrative bottleneck that could delay launch regardless of technical readiness
- **Staging domain:** `eprom-portal.xyz` (registered via Namecheap, independent of company DNS)

---

## Security Requirements

The IT department mandated these security measures:

| Requirement | Status |
|------------|--------|
| SQL injection prevention | ✅ Done — all queries parameterized |
| Complex passwords | ✅ Done — bcrypt with cost factor 12 |
| HTTPS enforcement | ✅ Done on staging, pending on production |
| Web Application Firewall (WAF) | Pending — IT to provide network-level WAF, or ModSecurity to be installed |
| Two-Factor Authentication (2FA) | Planned — via Orange/SMS, not yet implemented |

### Reputation Risk
The IT department emphasized that a compromised server poses more than a project-level threat:
- If the server is used for external attacks, the corporate IP address will be **blacklisted globally**
- This would **halt all corporate communications** (email, website, external services) for months
- The organization would be treated as a "suspect entity" in international markets

This is why security is prioritized even at the cost of development velocity.

---

## Bandwidth Constraints

| Fact | Detail |
|------|--------|
| **Requested bandwidth** | 200 MB |
| **IT assessment** | "200 MB is imaginary" (خيالي) given current infrastructure |
| **Current lines** | 10 MB and 20 MB dedicated lines |
| **Upgrade cost** | ~70,000 EGP per quarter for fiber line upgrade |
| **Mitigation** | IT will offload non-essential traffic to the Pre-WiMAX line |

This limits the portal to being a relatively lightweight web application. Heavy assets (large file uploads, video streaming) should be avoided. The 370 MB of static HTML reports are served locally from the VM, not transferred over the WAN.

---

## EGYPS Exhibition Risk

The EGYPS exhibition presents a specific connectivity risk:
- **Signal jamming** at the conference center during high-ranking official visits
- **Signal saturation** from thousands of attendees using mobile data
- The demonstration may fail due to local internet suppression, regardless of server-side bandwidth
- **Mitigation:** Prepare for offline demo capability or local hotspot fallback

---

## Financial Constraints

| Issue | Detail |
|-------|--------|
| **AI token funding** | Developers use personal credit cards for OpenAI/Anthropic tokens and domain names |
| **Out-of-pocket costs** | One developer paid 11,000 EGP out-of-pocket with a 3-month reimbursement delay |
| **Procurement process** | Traditional corporate procurement conflicts with agile AI development needs |
| **Short-term solution** | Submit bank statements with highlighted transactions + formal memo to Chairman |
| **Long-term goal** | Corporate payment gateway for AI "On-Demand" services to avoid "double costing" across personal accounts |

This financial friction directly impacts the AI credits system design — the system must carefully meter and control token usage because every token costs real money that may not be reimbursed promptly.

---

## Design Constraints (Technical)

These constraints emerged from 44 sessions of development and should not be violated:

1. **Server-side computation only** — All formulas, AI logic, and engineering calculations must run on the server. Zero business logic in browser JavaScript. This is non-negotiable for IP protection.

2. **Shared JWT authentication** — All apps share a single JWT secret and cookie (`eprom_token`). Do not introduce separate auth systems per app.

3. **Shared chat library** — All 4 apps use `chat-base.js` (config pattern). New apps must use this library, not create their own chat implementation.

4. **Vendor CSS override pattern** — The heater app uses Bootstrap (dark theme) with `!important` overrides to match EPROM corporate styling. This pattern should be used for any app that imports a CSS framework.

5. **No `rsync --delete`** — This previously destroyed critical files on the VM. Always sync additively.

6. **Docker Compose v2** — Use `docker compose` (v2 plugin). `docker-compose` (v1) is broken and should never be used.

7. **Mobile-first** — The majority of users access the portal on mobile devices. All design decisions should prioritize mobile usability with 44px touch targets minimum.
