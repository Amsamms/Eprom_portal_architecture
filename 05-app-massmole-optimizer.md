# Apps: MassMole Converter & Feature Optimizer

**Last Generated:** 2026-03-14

---

## Part 1: Mass Mole Converter (MassMole)

### Overview

The Mass Mole Converter handles mole%↔mass% conversions for **160 chemical compounds** across **15 categories**. It calculates stream properties (average molecular weight, LHV) and includes 5 industry-standard presets.

**URL:** https://eprom-portal.xyz/apps/massmole/
**Container:** `eprom_massmole`
**Technology:** Express.js (Node 20 Alpine)
**Port:** 3006 (behind Nginx at `/apps/massmole/`)

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

### API Endpoints

| Endpoint | Method | Auth | Rate Limit | Description |
|----------|--------|------|------------|-------------|
| `/api/health` | GET | No | No | Health check |
| `/api/defaults` | GET | Yes | No | Compound metadata + categories + presets |
| `/api/presets/:id` | GET | Yes | No | Load preset composition + auto-calculate |
| `/api/calculate` | POST | Yes | 30/min | Mole/mass conversion + stream properties |
| `/api/chat` | POST | Yes | 10/min | AI tool-use loop with 8 tools |

---

## Part 2: Feature Optimizer

### Overview

The Feature Optimizer is an ML-based tool for feature importance analysis and parameter optimization. It uses a unique Express.js + PHP hybrid architecture — Node.js handles routing, auth, and AI chat (port 3007), while PHP/Apache handles the ML computation (port 8080, internal only). Uses Pearson correlation for importance ranking and random search (10,000 iterations) for optimization.

**URL:** https://eprom-portal.xyz/apps/optimizer/
**Container:** `eprom_optimizer`
**Technology:** Express.js + PHP
**Port:** 3007 (behind Nginx at `/apps/optimizer/`)

### 5-Step Workflow

| Step | What Happens |
|------|-------------|
| **1. Upload** | User uploads a CSV dataset |
| **2. Data Quality & Preprocessing** | `DataPreprocessor` class scans for issues: missing values, text in numeric columns, outliers, duplicates, infinite values, constant columns. User can auto-fix or manually adjust. |
| **3. Feature Importance** | Correlation analysis ranks features by importance relative to a target variable. Configurable threshold for filtering. |
| **4. Optimization** | Random search (10,000 iterations) optimizes feature values to maximize/minimize the target variable. Uses linear or polynomial regression models. `CrossValidator` with 5-fold CV validates model quality (R², RMSE, MAE). |
| **5. Results** | Optimized parameter values, predicted target value, improvement percentage, model metrics (R², RMSE, MAE). |

### Key PHP Classes

| Class | File | Purpose |
|-------|------|---------|
| `DataPreprocessor` | `php/preprocessing.php` | Scans and preprocesses uploaded data (6 issue types) |
| `CrossValidator` | `php/validation.php` | 5-fold cross-validation with R², RMSE, MAE metrics |

### AI Chat — 8 Tools

| Tool | What It Does |
|------|-------------|
| `upload_data` | Upload CSV data as text (headers + data rows) |
| `set_target_column` | Set the target (Y) variable column name |
| `run_preprocessing` | Clean data: handle missing values, text in numeric columns, outliers, duplicates |
| `calculate_importance` | Pearson correlation analysis to rank features; auto-detects cutoff via elbow method |
| `run_optimization` | Random search (10,000 iterations) to find optimal X values; linear or polynomial model |
| `get_results` | Get session state by section (data, importance, optimization, all) |
| `get_data_summary` | Get loaded dataset summary: columns, row count, basic statistics |
| `show_insight` | Display ML educational insight card (tip, warning, info, benchmark) |

The AI tool executor routes tool calls to PHP API endpoints on port 8080:
- `php/api/run-importance.php`
- `php/api/run-optimization.php`
- `php/api/run-preprocessing.php`
- `php/api/get-session-state.php`

### Upstream Repository

The optimizer is based on a friend's (Amr Abu Mady) open-source repo: `github.com/amrabumady/ml-feature-optimizer`. The EPROM version adds auth integration, AI chat, EPROM theming, and preprocessing upgrades.

### Frontend

Unlike other apps that use vanilla HTML, the optimizer's frontend is a PHP-rendered page (`php/index.php`) with the 5-step workflow UI. It uses its own `eprom-theme.css` for styling and loads `chat-base.js` from the shared library.

The optimizer originally had a dark theme and was the last app migrated to the EPROM light theme (Session 40). Its mobile CSS is considered the most complete — it has zero framework conflicts and exhaustive responsive coverage, making it the reference for mobile improvements to other apps.

### Database Logging

- **`optimizer_calculation_logs`**: Dual-purpose table with `calc_type` column (importance or optimization). Importance rows store: feature_count, total_features, threshold, data_rows, target_column. Optimization rows store: predicted_y, objective, model_type, features_optimized, improvement_pct, iterations.
- **`optimizer_chat_logs`**: Standard chat log schema (conversation_id, role, content, tool_calls, model, tokens).
