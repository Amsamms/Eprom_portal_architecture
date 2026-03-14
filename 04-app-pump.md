# App: Pump Efficiency Calculator

**Last Generated:** 2026-03-14

---

## Overview

The Pump Efficiency Calculator analyzes centrifugal pump performance using **53 engineering formulas** across **9 calculation categories**. It's the only Python-based app on the platform, using FastAPI with NumPy and SciPy for scientific computation. It generates 5 interactive Plotly charts server-side.

**URL:** https://eprom-portal.xyz/apps/pump/
**Container:** `eprom_pump` (~150 MB RAM with numpy/scipy)
**Technology:** Python 3.11 + FastAPI + uvicorn
**Port:** 3005 (behind Nginx at `/apps/pump/`)

---

## Calculation Engine

### 9 Calculation Categories

| Category | What It Calculates |
|----------|-------------------|
| **Fluid Properties** | Water/fluid density, viscosity at operating temperature |
| **Pipe Hydraulics** | Reynolds number, friction factor (Colebrook-White), pipe losses (Darcy-Weisbach) |
| **Curve Fitting** | Polynomial fit of manufacturer H-Q, efficiency, power, NPSHr curves (numpy polyfit) |
| **System Curve** | Static head + friction losses as a function of flow rate |
| **Operating Point** | Intersection of pump curve and system curve |
| **BEP Analysis** | Best Efficiency Point detection, deviation from BEP |
| **Power** | Hydraulic power, brake power, motor power, wire-to-water |
| **NPSH** | NPSHa (available), NPSHr (required), margin, cavitation risk check |
| **Affinity Laws** | Speed and diameter ratio transformations for pump curves |

### Key Libraries

| Library | Usage |
|---------|-------|
| **NumPy** | Polynomial curve fitting (`numpy.polyfit`), array operations |
| **SciPy** | Root finding for operating point intersection, Colebrook-White solver |
| **Plotly** | Server-side chart generation as JSON (5 chart types) |
| **asyncpg** | Async PostgreSQL connection pool |
| **python-jose** | JWT verification (note: has known CVEs, scheduled for replacement with PyJWT) |

### Engine Files

| File | Purpose |
|------|---------|
| `src/engine/pump_engine.py` | Core 53-formula orchestrator |
| `src/engine/curve_fitter.py` | NumPy polyfit, BEP finding, curve intersection |
| `src/engine/system_curve.py` | Darcy-Weisbach, Colebrook-White, Reynolds number |
| `src/engine/affinity.py` | Speed/diameter affinity law transformations |
| `src/engine/npsh.py` | NPSHa, NPSHr, margin, cavitation check |
| `src/engine/constants.py` | Physical constants, water property tables |

---

## KPIs (6)

| KPI | Unit | Description |
|-----|------|-------------|
| **Pump Efficiency** | % | Overall pump efficiency at operating point |
| **BEP Deviation** | % | How far the operating point is from Best Efficiency Point |
| **Power Consumption** | kW | Brake horsepower at operating point |
| **NPSH Margin** | m | NPSHa − NPSHr (positive = safe from cavitation) |
| **Specific Speed** | dimensionless | Pump type classification number |
| **Wire-to-Water Efficiency** | % | Overall efficiency from electrical input to hydraulic output |

---

## Charts (5 Plotly Charts)

| Chart | What It Shows |
|-------|-------------- |
| **H-Q + System Curve** | Pump head curve, system curve, and operating point intersection |
| **Efficiency Curve** | Pump efficiency vs. flow rate with BEP marked |
| **Power Curve** | Power consumption vs. flow rate |
| **NPSH Curve** | NPSHa and NPSHr vs. flow rate, with margin visualization |
| **Affinity Curves** | Transformed curves at different speeds/diameters |

Charts are generated as Plotly JSON on the server and rendered client-side using Plotly.js. This keeps the computation server-side while allowing interactive chart features in the browser.

---

## AI Chat Assistant

### 6 Tools

| Tool | What It Does |
|------|-------------|
| `run_calculation` | Executes all 53 formulas, returns KPIs + chart data |
| `fill_input_fields` | Fills pump input fields (triggers animated cursor) |
| `get_current_inputs` | Reads current form values |
| `compare_scenarios` | Runs multiple parameter sets and produces comparison |
| `analyze_sensitivity` | Varies one input parameter, measures KPI sensitivity |
| `detect_anomalies` | Checks inputs against expected ranges |

### Visual Actions

The pump app has full visual action support (animated cursor filling fields), with custom `beforeFillFields` and `afterFillFields` hooks in the visual-actions config for pump-specific behavior (e.g., updating curve data tables before calculation).

---

## API Endpoints

| Endpoint | Method | Auth | Rate Limit | Description |
|----------|--------|------|------------|-------------|
| `/api/health` | GET | No | No | Health check |
| `/api/defaults` | GET | Yes | No | `INPUT_METADATA` + `DEFAULT_INPUTS` + `CURVE_DATA` (26 inputs + 7 curve datapoints) |
| `/api/calculate` | POST | Yes | 30/min (slowapi) | 53 formulas + JSON sanitization (inf/nan) |
| `/api/chat` | POST | Yes | 10/min (slowapi) | AI tool-use loop with 6 tools |

---

## Key Differences from Node.js Apps

| Aspect | Pump (Python) | Heater/MassMole (Node.js) |
|--------|--------------|--------------------------|
| **Runtime** | Python 3.11, FastAPI, uvicorn | Node 20, Express.js |
| **Database** | asyncpg (async connection pool) | pg (synchronous pool) |
| **JWT Library** | python-jose (has CVEs) | jsonwebtoken |
| **Rate Limiting** | slowapi (built on limits library) | Custom in-memory Map |
| **JSON Sanitization** | Custom inf/nan → null replacement | Not needed (JavaScript handles) |
| **Charts** | Plotly (server-side JSON) | ApexCharts (client config) |
| **Scientific Computing** | NumPy, SciPy | Not needed |
| **Image size** | ~150 MB (python:3.11-slim + numpy/scipy) | ~80 MB (node:20-alpine) |

### JSON Sanitization

Python's `float('inf')` and `float('nan')` are not valid JSON. The pump engine has a custom sanitizer that replaces these with `null` before sending responses. This was a bug that caused chat crashes in early sessions.

---

## Frontend

Vanilla JS SPA matching the same thin-client pattern as heater:

| File | Purpose |
|------|---------|
| `public/index.html` | Entry point with `<base href="/apps/pump/">`, loads Bootstrap 5 + Plotly.js |
| `public/js/app.js` | Thin orchestrator |
| `public/js/ui-manager.js` | Form generation, curve data table, results rendering |
| `public/js/chart-renderer.js` | Plotly chart rendering from server JSON |
| `public/js/chat-base.js` | Shared chatbot library |
| `public/js/chat-config.js` | Pump-specific ChatBase configuration |
| `public/js/visual-actions-base.js` | Shared cursor animation library |
| `public/js/visual-actions-config.js` | Pump-specific field mapping with before/after hooks |
| `public/css/main.css` | Blue/teal industrial theme + AI animation classes |

---

## Database Logging

- **`pump_calculation_logs`**: inputs (JSONB), outputs (JSONB), 6 KPI columns, formula_count, error_count, duration_ms
- **`pump_chat_logs`**: conversation_id (UUID), role, content, tool_calls (JSONB), model, tokens
