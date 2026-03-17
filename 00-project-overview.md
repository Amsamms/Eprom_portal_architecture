# EPROM Portal — Project Overview

**Last Generated:** 2026-03-17

---

## What is EPROM?

EPROM (Egyptian Projects Operation & Maintenance) is the largest operation and maintenance company in Egypt and North Africa, operating across the entire oil & gas value chain — from upstream gas processing and SCADA pipelines to downstream refineries, petrochemical plants, fertilizer factories, and marine terminals.

Originally founded in 2002 as MIDOM (Middle East for Operation & Maintenance of Oil Refineries), the company operates under the Egyptian General Petroleum Corporation (EGPC). Led by Chairman Eng. Hossam Asaad, EPROM employs over 1,000 staff and offers services spanning O&M, commissioning, process safety management, CEMS, and training (300+ courses, 150+ instructors). The company also has international projects in Nigeria and across the Middle East and Africa.

## Why This Project Exists

The EPROM Portal was built to digitize and protect 20+ years of process engineering knowledge, replacing Excel-based manual calculations with a secure AI-powered platform — to be demonstrated at EGYPES 2026 for the Chairman, the Authority, and the Ministry of Petroleum.

Beyond internal use, the project serves three strategic goals:

1. **Lead AI implementation in Egypt and North Africa.** EPROM is already a leader in energy transition, process safety, carbon footprint reporting, and operational standards. The portal positions EPROM as the first O&M company in the region to deploy AI-driven engineering tools at scale.

2. **Sell the product to other Egyptian petroleum companies.** The portal is designed as a licensable product. Other companies under EGPC and beyond can subscribe to access the same calculation and AI tools — generating direct revenue for EPROM.

3. **Use usage data to identify and sell what each company needs.** Calculation logs, AI chat interactions, and app usage patterns reveal what each client company focuses on — whether it's heater efficiency, pump analysis, emissions reporting, or process optimization. EPROM can use this data to offer tailored consulting, training, and engineering services — because EPROM already has the experience, the people, and the tools.

---

## What is the EPROM Portal?

The EPROM Portal is a secure, mobile-first web platform that gives authorized employees and stakeholders access to multiple engineering and data applications through a unified interface. It was built as a solo project by Ahmed Mohamed Sabri (Process Engineer / Developer) over 53 development sessions.

The portal's core innovation is the integration of "Process Knowledge" (20+ years of operational experience) with "System Knowledge" (AI/ML capabilities). The resulting tools achieve 97–98% accuracy for predictive calculations. Even if the source code were compromised, the underlying expert assumptions — derived from decades of EPROM operational data — remain a proprietary barrier to entry.

**Live URL:** https://eprom-portal.xyz (staging on AWS EC2)

---

## The Five Live Apps

| App | What It Does | Tech |
|-----|-------------|------|
| **Heater Efficiency Calculator** | Evaluates fired heater thermal performance using 406 engineering formulas across 43 inputs. Produces KPIs like thermal efficiency, fuel efficiency, excess air %, and GHG emission rate. Includes a professional SVG process schematic with animated KPI annotations. | Express.js, ApexCharts |
| **Pump Efficiency Calculator** | Analyzes centrifugal pump performance with 53 formulas covering curve fitting, system curves, NPSH, affinity laws, and BEP. Generates 5 interactive Plotly charts. | Python 3.11, FastAPI, SciPy |
| **Mass Mole Converter** | Converts between mole% and mass% for 160 chemical compounds across 15 categories. Calculates stream properties (avg molecular weight, LHV). Includes 5 industry presets. | Express.js |
| **Feature Optimizer** | ML-based feature importance analysis and parameter optimization. Upload CSV or paste from Excel → preprocess → analyze → optimize. Includes one-click analysis and feature response charts. | Express.js + PHP hybrid, Chart.js |
| **Interactive Reports** | Library of 25 petroleum industry reports (370 MB of static HTML) served via Nginx. Searchable, filterable by 5 categories. | Static HTML, Nginx |

### Coming Soon (12 apps)

The dashboard shows 12 additional apps across 7 pillars, currently marked as "Coming Soon":

| Pillar | Coming Soon Apps |
|--------|-----------------|
| **AI Equipment Performance** | Turbine Analyzer, Boiler Monitor, Cooling Tower, Distillation Tower, Air Cooler, Separators, Compressors, Heat Exchanger |
| **Process Smart Tools** | Unit Converter, Fluid Properties, Pipe Sizer, Relief Valve Sizing |

Every calculation app includes an **AI Chat Assistant** ("EPROM AI Companion") powered by Anthropic Claude. The AI can fill input forms, run calculations, load scenarios, and provide engineering analysis — all through natural language conversation with tool-use capabilities. A separate Dashboard AI Companion helps users navigate the portal and understand the available tools.

---

## The 7 Pillars

The dashboard organizes all apps into 7 strategic pillars:

| Pillar | Apps |
|--------|------|
| **AI Equipment Performance** | Heater (live), + 8 coming soon |
| **Process Smart Tools** | Pump (live), MassMole (live), + 4 coming soon |
| **Virtual Sensors** | Quality Analyzer (coming soon) |
| **Optimization Engine** | Feature Optimizer (live) |
| **Smart Sustainability** | Carbon Footprint (coming soon) |
| **Safety Intelligence** | Leak Detection (coming soon) |
| **Knowledge Hub** | Interactive Reports (live) |

---

## Business Context

### EGYPES Exhibition
The immediate catalyst for the project was the upcoming **EGYPES** (Egypt Energy Show), 30 March – 1 April 2026 at the Egypt International Exhibition Center. The portal serves as a high-visibility demonstration for the Chairman, the Authority, and the Ministry of Petroleum. A free one-week trial was planned to allow officials to observe live system outputs and validate ROI.

### The "Smart Engine" Vision
The portal is part of a broader initiative called **"Smart Engine"** — the strategic convergence of Process Engineering and Digital Transformation. It covers six pillars:

1. **Equipment Analysis (Fired Heaters)** — Codifies 20 years of institutional knowledge
2. **Soft Sensors (ML)** — Predicts outcomes during instrumentation failure (98% accuracy)
3. **Automatic Carbon Footprint** — Automates environmental reporting
4. **Process Safety** — AI-based hazard prediction
5. **Functional Safety** — Automated compliance reporting
6. **Operational Standards** — Embeds international standards into software logic

### IT Meeting Outcomes
A formal IT strategy meeting established the deployment framework:
- **Hosting:** A dedicated 32 GB RAM physical server in a DMZ (Demilitarized Zone)
- **Access:** SSH with private keys only (AnyDesk explicitly rejected)
- **Security:** WAF, HTTPS mandatory, 2FA planned, SQL injection prevention
- **Domain:** `ese.eprom.com.eg` (pending DNS coordination with Telecom Egypt)

---

## Key People

| Role | Who | Notes |
|------|-----|-------|
| Developer | Ahmed Mohamed Sabri | Solo developer, built entire platform over 53 sessions |
| IT Department | Company IT team | Manages infrastructure, SSH access, network security |
| Chairman | Eng. Hossam Asaad | Key stakeholder, EGYPES demo audience |
| Friend/Colleague | Amr Abu Mady | Contributed ML Feature Optimizer upstream repo |

---

## Current State (as of Session 53)

- **4 calculation apps** live and tested on staging and production (heater, pump, massmole, optimizer)
- **12 coming-soon apps** displayed on dashboard across 7 pillars
- **Heater process schematic** — professional SVG with 8 animated KPI annotations
- **Optimizer V2** — one-click analysis, paste from Excel, feature response charts
- **5 unified AI chatbots** — consistent icons (two-drops SVG), white headers, mobile bottom sheets, onboarding tips
- **7-pillar dashboard** with collapsible sections and universal sidebar
- **Iframe app integration** — apps load inside portal with consistent navigation
- **Admin dashboard overhaul** complete — global stats, session lifecycle tracking, user journey timeline
- **AI token optimization** achieving 63–86% cost reduction across all apps
- **Company VM migration** complete (192.168.50.202, 8 CPU / 24 GB RAM), 130/130 Playwright tests passed
- **Pre-production security audit** pending before domain goes live
- **SSL and domain** active on staging (eprom-portal.xyz), pending on production (ese.eprom.com.eg)
