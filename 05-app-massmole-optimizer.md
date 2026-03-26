# Apps: MassMole Converter & Feature Optimizer

**Last Generated:** 2026-03-26

---

## Part 1: Mass Mole Converter (MassMole)

### Overview

The Mass Mole Converter handles mole%↔mass% conversions for **160 chemical compounds** across **15 categories**. It calculates stream properties (average molecular weight, LHV) and includes 5 industry-standard presets.

**URL:** https://eprom-portal.xyz/apps/massmole/
**Container:** `eprom_massmole`
**Technology:** Express.js (Node 20 Alpine)
**Port:** 3006 (behind Nginx at `/apps/massmole/` or `/_proxy/massmole/`)

### Engine

| File | Purpose |
|------|---------|
| `engine/compounds.js` | 160 compounds across 15 categories — each has: name, chemical formula, molecular weight (MW), lower heating value (LHV), CAS number |
| `engine/calculations.js` | `moleToMass()`, `massToMole()`, `calculateStreamProperties()` — core conversion functions |
| `engine/presets.js` | 5 presets: Natural Gas, Biogas, Syngas, LPG, Refinery Fuel Gas |
| `engine/massmole-engine.js` | Engine class: search, calculate, validate, apply presets |

### 15 Compound Categories

Alkanes, Alkenes, Alkynes, Aromatics, Alcohols, Acids, Aldehydes, Ketones, Ethers, Esters, Amines, Inorganic Gases, Sulfur Compounds, Halogenated, Other.

### Stream Properties (KPIs)

| Property | Unit |
|----------|------|
| **Average Molecular Weight** | g/mol |
| **LHV Mass Basis** | MJ/kg |
| **LHV Volume Basis** | MJ/m³ |

### 5 Presets

| Preset | Description |
|--------|-------------|
| Natural Gas | Typical pipeline natural gas composition |
| Biogas | Anaerobic digestion output |
| Syngas | Synthesis gas from gasification |
| LPG | Liquefied petroleum gas |
| Refinery Fuel Gas | Mixed refinery off-gas |

### AI Chat — 8 Tools

| Tool | What It Does |
|------|-------------|
| `search_compound` | Search compounds by name, formula, or CAS number |
| `get_compound_details` | Get full details for a specific compound |
| `fill_composition` | Fill compound percentages in the form |
| `trigger_calculate` | Execute mole/mass conversion + stream properties |
| `load_preset` | Load one of 5 preset compositions |
| `clear_composition` | Clear all entered compounds |
| `switch_basis` | Toggle between mole% and mass% input basis |
| `show_insight` | Display insight cards with engineering context |

### Iframe Mode

When loaded inside the portal iframe (via `/_proxy/massmole/`):
- **Hides:** `.app-header`
- **Creates:** Custom `#iframe-ai-btn` floating button (white background with two-drops SVG)
- FAB hides when chat is open via `body.chat-open` class

### API Endpoints

| Endpoint | Method | Auth | Rate Limit | Description |
|----------|--------|------|------------|-------------|
| `/api/health` | GET | No | No | Health check |
| `/api/defaults` | GET | Yes | No | Compound metadata + categories + presets |
| `/api/presets/:id` | GET | Yes | No | Load preset composition + auto-calculate |
| `/api/calculate` | POST | Yes | 30/min | Mole/mass conversion + stream properties |
| `/api/chat` | POST | Yes | 10/min | AI tool-use loop with 8 tools |

---

## Part 2: Feature Optimizer (4 Independent Apps)

### Overview

As of Session 55, the single optimizer was split into **4 fully independent apps**, each with its own directory, Docker container, AI chatbot (with domain-specific system prompt), and database log tables. They share the same ML engine (Express.js + PHP hybrid) but are tailored to different engineering domains.

| App | Container | Port | Domain Focus |
|-----|-----------|------|-------------|
| **Optimizer — General** | `eprom_optimizer_general` | 3007 | General-purpose ML optimization |
| **Optimizer — Energy** | `eprom_optimizer_energy` | 3008 | Power plant thermal efficiency |
| **Optimizer — Distillation** | `eprom_optimizer_distillation` | 3009 | Distillation column purity |
| **Optimizer — Flare** | `eprom_optimizer_flare` | 3010 | Flare gas recovery rate |

Each runs Express.js (Node.js) for auth and AI chat + PHP/Apache (port 8080, internal) for ML computation. Based on Amr Abu Mady's upstream repo (`github.com/amrabumady/ml-feature-optimizer`), with EPROM additions: auth, AI chat, theming, preprocessing upgrades, one-click analysis, paste from Excel, feature response charts, and domain-specific system prompts.

### Two-Flow Data Input (Session 56)

Users can load data two ways via a method selector UI:
1. **Paste from Excel** — Paste tab-separated data with auto-delimiter and auto-transpose detection, live preview table
2. **Upload Data File** — Upload a CSV dataset file

A **welcome banner** with config-driven app name and description appears per variant. Sample datasets can be loaded to try the tool immediately.

### 5-Step Workflow

| Step | What Happens |
|------|-------------|
| **1. Upload** | User uploads CSV or pastes from Excel |
| **2. Data Quality & Preprocessing** | `DataPreprocessor` class scans: missing values, text in numeric, outliers, duplicates, infinite values, constant columns |
| **3. Feature Importance** | Correlation analysis ranks features. Configurable threshold + elbow method auto-cutoff |
| **4. Optimization** | Random search (50,000 iterations) with feature normalization. Linear or polynomial regression. `CrossValidator` 5-fold CV (R², RMSE, MAE) |
| **5. Results** | Optimized parameters, predicted target, improvement %, model metrics, feature response charts |

### One-Click Analysis

Select target column + goal (maximize/minimize) → "One-Click Analysis" runs preprocessing → importance → polynomial optimization in a single request. Endpoint: `php/api/one-click.php`.

### AI Chat — 9 Tools (per variant)

| Tool | What It Does |
|------|-------------|
| `upload_data` | Upload CSV data as text |
| `set_target_column` | Set the target (Y) variable |
| `run_preprocessing` | Clean data (6 issue types) |
| `calculate_importance` | Pearson correlation analysis |
| `run_optimization` | Random search optimization (50K iterations) |
| `one_click_analysis` | Full pipeline in one call |
| `get_results` | Get session state by section |
| `get_data_summary` | Dataset summary statistics |
| `show_insight` | ML educational insight card |

Each variant has a **unique AI system prompt** with domain-specific terminology, welcome message, and quick chips tailored to its engineering area.

### Key PHP Classes

| Class | File | Purpose |
|-------|------|---------|
| `DataPreprocessor` | `php/preprocessing.php` | Scans and preprocesses data (6 issue types) |
| `CrossValidator` | `php/validation.php` | 5-fold cross-validation with R², RMSE, MAE |

### Frontend

Each optimizer is a PHP-rendered page (`php/index.php`) with the 5-step workflow UI, `eprom-theme.css` for EPROM light theme styling, `chat-base.js` from the shared library, and Chart.js for feature response charts. The EPROM tears PNG logo (`eprom-tears.png`) is used for all chatbot icons.

### Database Logging

Each optimizer variant shares the same table structure:
- **`optimizer_calculation_logs`**: Dual-purpose with `calc_type` column (importance or optimization). Importance rows: feature_count, total_features, threshold, data_rows, target_column. Optimization rows: predicted_y, objective, model_type, features_optimized, improvement_pct, iterations.
- **`optimizer_chat_logs`**: Standard chat log schema (conversation_id, role, content, tool_calls, model, tokens).

### E2E Testing

200 E2E tests across all 4 optimizer variants on production — 199 pass, 1 fail (99.5% pass rate).
