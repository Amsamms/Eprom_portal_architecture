# EPROM Portal — Project Overview

**Last Generated:** 2026-03-14

---

## What is EPROM?

EPROM (Egyptian Petroleum Research Organization for Midstream) is an Egyptian petroleum company operating in the midstream sector. The organization has decades of institutional knowledge in process engineering, equipment analysis, and operational standards.

## What is the EPROM Portal?

The EPROM Portal is a secure, mobile-first web platform that gives authorized employees and stakeholders access to multiple engineering and data applications through a unified interface. It was built as a solo project by Ahmed Mohamed Sabri (Process Engineer / Developer) over 44 development sessions.

The portal's core innovation is the integration of "Process Knowledge" (20+ years of operational experience) with "System Knowledge" (AI/ML capabilities). The resulting tools achieve 97–98% accuracy for predictive calculations. Even if the source code were compromised, the underlying expert assumptions — derived from decades of EPROM operational data — remain a proprietary barrier to entry.

**Live URL:** https://eprom-portal.xyz (staging on AWS EC2)

---

## The Five Apps

| App | What It Does | Tech |
|-----|-------------|------|
| **Heater Efficiency Calculator** | Evaluates fired heater thermal performance using 406 engineering formulas across 43 inputs. Produces KPIs like thermal efficiency, fuel efficiency, excess air %, and GHG emission rate. | Express.js, ApexCharts |
| **Pump Efficiency Calculator** | Analyzes centrifugal pump performance with 53 formulas covering curve fitting, system curves, NPSH, affinity laws, and BEP. Generates 5 interactive Plotly charts. | Python 3.11, FastAPI, SciPy |
| **Mass Mole Converter** | Converts between mole% and mass% for 160 chemical compounds across 15 categories. Calculates stream properties (avg molecular weight, LHV). Includes 5 industry presets. | Express.js |
| **Feature Optimizer** | ML-based feature importance analysis and parameter optimization using correlation analysis and genetic algorithms. Upload CSV → preprocess → analyze → optimize. | Express.js + PHP hybrid |
| **Interactive Reports** | Library of 25 petroleum industry reports (370 MB of static HTML) served via Nginx. Searchable, filterable by 5 categories. | Static HTML, Nginx |

Every calculation app includes an **AI Chat Assistant** ("EPROM AI Companion") powered by Anthropic Claude. The AI can fill input forms, run calculations, load scenarios, and provide engineering analysis — all through natural language conversation with tool-use capabilities.

---

## Business Context

### EGYPS Exhibition
The immediate catalyst for the project was the upcoming **EGYPS** (Egypt Petroleum Show) exhibition. The portal serves as a high-visibility demonstration for the Chairman, the Authority, and the Ministry of Petroleum. A free one-week trial was planned to allow officials to observe live system outputs and validate ROI.

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
| Developer | Ahmed Mohamed Sabri | Solo developer, built entire platform over 44 sessions |
| IT Department | Company IT team | Manages infrastructure, SSH access, network security |
| Chairman | EPROM Chairman | Key stakeholder, EGYPS demo audience |
| Friend/Colleague | Amr Abu Mady | Contributed ML Feature Optimizer upstream repo |

---

## Current State (as of Session 44)

- **4 calculation apps** live and tested on staging (heater, pump, massmole, optimizer)
- **AI token optimization** achieving 63–86% cost reduction across all apps
- **Company VM migration** complete (192.168.50.202, 8 CPU / 24 GB RAM)
- **Pre-production security audit** pending before domain goes live
- **SSL and domain** active on staging (eprom-portal.xyz), pending on production (ese.eprom.com.eg)
