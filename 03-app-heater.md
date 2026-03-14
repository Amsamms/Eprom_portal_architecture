# App: Heater Efficiency Calculator

**Last Generated:** 2026-03-14

---

## Overview

The Heater Efficiency Calculator evaluates fired heater thermal performance using **406 engineering formulas** running server-side. It takes **43 inputs** organized across **5 sections** and produces comprehensive efficiency analysis with KPIs, charts, and detailed output tables.

**URL:** https://eprom-portal.xyz/apps/heater/
**Container:** `eprom_heater` (~20 MB RAM)
**Technology:** Express.js (Node 20 Alpine)
**Port:** 3001 (behind Nginx at `/apps/heater/`)

---

## Calculation Engine

### Architecture

The engine is the core intellectual property — it runs entirely server-side and is never served to the browser.

| File | Purpose | Size |
|------|---------|------|
| `engine/formula-specs.js` | All 406 formula specifications | 4,136 lines |
| `engine/heater-engine.js` | Core engine — evaluates all formulas | ~400 lines |
| `engine/translator.js` | Translates Excel-style formulas to JavaScript | ~120 lines |
| `engine/dependency-resolver.js` | Topological sort (Kahn's algorithm) for formula evaluation order | ~80 lines |

### How It Works

1. User submits 43 inputs → `POST /api/calculate`
2. `formula-specs.js` defines each formula with: ID, name, Excel-style expression, dependencies, section, unit
3. `translator.js` converts Excel expressions (e.g., `IF(A1>0, A1*B2, 0)`) to JavaScript
4. `dependency-resolver.js` sorts formulas by dependencies (topological sort)
5. `heater-engine.js` evaluates all 406 formulas in dependency order using `new Function()` for dynamic evaluation
6. Results grouped into KPIs, charts, and output tables

### Key KPIs

| KPI | Unit | Description |
|-----|------|-------------|
| **Thermal Efficiency** | % | Overall heater thermal efficiency |
| **Fuel Efficiency** | % | Fuel utilization efficiency |
| **Excess Air** | % | Excess combustion air percentage |
| **GHG Emission Rate** | kg CO₂/GJ | Greenhouse gas emission rate |
| **Liberated Heat** | MW | Total heat liberated from fuel combustion |
| **Absorbed Duty** | MW | Heat absorbed by the process fluid |

### 5 Input Sections

The 43 inputs are organized into 5 sections covering fuel composition, ambient conditions, heater geometry, operating parameters, and process fluid properties. Default values and metadata (labels, units, ranges, tooltips) are served via the `/api/defaults` endpoint.

---

## 7 Scenarios

The heater includes **7 predefined scenarios** — each is a complete set of input values representing different operating conditions. Scenarios are served via `GET /api/scenarios` (names and descriptions only) and loaded by the AI chat assistant.

---

## AI Chat Assistant

### 7 Tools

The AI (Claude Haiku by default) has access to 7 server-side tools:

| Tool | What It Does |
|------|-------------|
| `fill_inputs` | Fills one or more input fields with values (triggers visual cursor animation) |
| `trigger_calculate` | Clicks the Calculate button to run all 406 formulas |
| `get_current_results` | Reads current calculation results by section (kpis, fuel_data, performance, energy_balance, all) |
| `get_current_inputs` | Reads all 43 current input field values by section |
| `load_scenario` | Loads one of 7 predefined scenario presets (with optional auto-calculate) |
| `scroll_to_section` | Scrolls the page to a specific UI section (kpi-section, charts-section, etc.) |
| `show_insight` | Displays an engineering insight card in chat (tip, warning, info, benchmark) |

### Visual Actions

When the AI calls `fill_inputs`, the frontend plays an animated cursor that:
1. Moves to the target input field
2. Highlights the field
3. Types the new value character by character
4. Moves to the next field

This is powered by `visual-actions-base.js` (shared library) + `visual-actions-config.js` (heater-specific field mapping).

### Model Selector (Easter Egg)

Clicking the EPROM logo 10 times reveals a dropdown to switch between Claude Haiku (default), Sonnet, and Opus. The selection is saved to localStorage. This is a hidden feature for internal testing.

---

## API Endpoints

| Endpoint | Method | Auth | Rate Limit | Description |
|----------|--------|------|------------|-------------|
| `/api/health` | GET | No | No | Health check (returns `{ status: 'ok' }`) |
| `/api/defaults` | GET | Yes | No | `INPUT_METADATA` + `DEFAULT_INPUTS` for form generation |
| `/api/scenarios` | GET | Yes | No | Scenario names and descriptions (no input data) |
| `/api/calculate` | POST | Yes | 30/min | Runs 406 formulas, returns KPIs + charts + output tables |
| `/api/chat` | POST | Yes | 10/min | AI tool-use loop, returns response + action array |

All protected endpoints require the `eprom_token` httpOnly cookie and validate against:
1. JWT signature verification
2. Database session existence check
3. Heater app permission check (`user_app_permissions`)

---

## Charts

The heater uses **ApexCharts** for client-side rendering. Chart data (series, labels, colors) is generated server-side and returned as JSON from the calculate endpoint.

---

## Frontend

The heater frontend is a **vanilla JavaScript SPA** (no framework). Key files:

| File | Purpose |
|------|---------|
| `public/index.html` | Entry point with `<base href="/apps/heater/">` |
| `public/js/app.js` | Thin orchestrator — API calls only, no calculations |
| `public/js/ui-manager.js` | Form generation from metadata, chart rendering, KPI display |
| `public/js/chat-base.js` | Shared chatbot library (copy from `shared/`) |
| `public/js/chat-config.js` | Heater-specific ChatBase configuration |
| `public/js/visual-actions-base.js` | Shared cursor animation library |
| `public/js/visual-actions-config.js` | Heater-specific field mapping for animations |
| `public/vendor/` | Bootstrap, jQuery, ApexCharts |
| `public/css/main.css` | Bootstrap dark theme overrides + EPROM design tokens |

**Key constraint:** The frontend contains zero calculation logic. All 406 formulas run server-side. The browser only renders results.

---

## Database Logging

Every calculation and chat message is logged:

- **`heater_calculation_logs`**: inputs (JSONB), outputs (JSONB), KPIs (thermal_eff, fuel_eff, excess_air, ghg_rate), formula_count, error_count, duration_ms
- **`heater_chat_logs`**: conversation_id, role, content, tool_calls (JSONB), model, tokens_input, tokens_output
