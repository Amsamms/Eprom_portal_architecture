# Database Schema & Authentication

**Last Generated:** 2026-03-14

---

## Database: PostgreSQL 16

Runs as `eprom_db` container (postgres:16-alpine) on the internal Docker network. Not exposed to the host or internet. All services connect using the same `eprom_user` credentials via the internal Docker bridge network (`eprom_net`).

---

## Table Overview (14 Tables)

### Core Tables

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| `users` | User accounts | `id`, `email` (unique), `full_name`, `password_hash`, `role` (viewer/admin), `is_active`, `email_verified`, `verification_token`, `company_name`, `department`, `job_title`, `years_of_experience`, `user_number` (USR-XXXX, auto-generated) |
| `apps` | Registered applications | `id`, `name` (unique), `display_name`, `description`, `url_slug`, `icon`, `port`, `is_active`. Seeded with 7 apps: heater, pump, massmole, carbon, sensors, leak, optimizer |
| `user_app_permissions` | Per-user app access | `user_id`, `app_name`, `can_access`, `granted_by`, `granted_at`. Has UNIQUE constraint on `(user_id, app_name)` |
| `sessions` | Active login sessions | `user_id`, `token_hash` (unique), `expires_at`, `ip_address`, `user_agent`. Used for DB-level session validation |
| `activity_logs` | General activity tracking | `user_id`, `app_name`, `action`, `input_summary`, `result_summary`, `duration_ms`. Tracks logins, logouts, app opens, report clicks |

### Per-App Calculation Logs

| Table | App | KPI Columns |
|-------|-----|-------------|
| `heater_calculation_logs` | Heater | `kpi_thermal_eff`, `kpi_fuel_eff`, `kpi_excess_air`, `kpi_ghg_rate`, `formula_count`, `error_count` |
| `pump_calculation_logs` | Pump | `kpi_pump_eff`, `kpi_bep_dev`, `kpi_power`, `kpi_npsh_margin`, `kpi_specific_speed`, `kpi_wire_to_water` |
| `massmole_calculation_logs` | MassMole | `input_basis`, `compound_count`, `avg_mw`, `lhv_mass`, `lhv_volume`, `total_percent`, `is_valid` |
| `optimizer_calculation_logs` | Optimizer | `calc_type` (importance/optimization), `feature_count`, `threshold`, `predicted_y`, `objective`, `model_type`, `improvement_pct` |

All calculation log tables store `inputs` and `outputs` as JSONB, plus `duration_ms` and `ip_address`.

### Per-App Chat Logs

| Table | App |
|-------|-----|
| `heater_chat_logs` | Heater |
| `pump_chat_logs` | Pump |
| `massmole_chat_logs` | MassMole |
| `optimizer_chat_logs` | Optimizer |

All chat log tables share the same schema: `user_id`, `conversation_id`, `role`, `content`, `tool_calls` (JSONB), `model`, `tokens_input`, `tokens_output`, `ip_address`.

### Subscription Tables

| Table | Purpose |
|-------|---------|
| `subscription_plans` | Defines tiers: Basic, Professional, Enterprise. Columns: `name`, `display_name`, `apps` (JSONB array), `has_ai_access`, `color`, `sort_order` |
| `user_subscriptions` | Links users to plans. Columns: `user_id`, `plan_id`, `starts_at`, `expires_at`, `is_active`, `granted_by`, `notes` |

---

## Subscription Tiers

| Tier | Apps Included | AI Access | Color |
|------|--------------|-----------|-------|
| **Basic** | Reports only | No | `#5A6A85` (gray) |
| **Professional** | Heater, Pump, MassMole, Reports | No | `#00529B` (blue) |
| **Enterprise** | All apps (heater, pump, massmole, carbon, sensors, leak, optimizer, reports) | Yes | `#00A651` (green) |

---

## Authentication Flow

```
1. User submits email + password
        │
2. Login API: POST /api/auth/login
   ├── Rate limit check (5 attempts / 15 min per IP)
   ├── Find user by email
   ├── bcrypt.compare(password, password_hash) — cost factor 12
   ├── Check user.is_active === true
   ├── Generate JWT with claims: { userId, email, role, jti }
   ├── Hash JWT token → store in sessions table (24h expiry)
   └── Set httpOnly cookie: eprom_token
        │
3. Cookie attributes:
   ├── httpOnly: true (not accessible via JavaScript)
   ├── secure: true (when PORTAL_URL starts with https)
   ├── sameSite: 'lax' (cross-site POST protection)
   ├── path: '/' (available to all routes)
   └── maxAge: 86400 (24 hours)
        │
4. Subsequent requests:
   ├── Portal middleware: jwtVerify(token, secret) — signature + expiry check
   │   (Note: middleware does NOT check DB session — page shell only)
   └── API routes: getTokenFromRequest() + validateSession()
       ├── Verify JWT signature (jose / jsonwebtoken / python-jose)
       ├── Hash token → query sessions table for matching active session
       ├── Check session not expired
       └── Return decoded user claims
        │
5. App-specific access:
   └── Check user_app_permissions for can_access = true
       (e.g., user must have "heater" permission to use heater app)
        │
6. Admin-only routes:
   └── requireAdmin() middleware: decoded.role === 'admin'
```

---

## Rate Limiting

| Endpoint | Limit | Implementation |
|----------|-------|---------------|
| `POST /api/auth/login` | 5 attempts / 15 min per IP | In-memory Map (portal) |
| `POST /api/calculate` | 30 requests / min per user | In-memory (heater/massmole), slowapi (pump) |
| `POST /api/chat` | 10 requests / min per user | In-memory (heater/massmole), slowapi (pump) |
| `POST /api/auth/signup` | No rate limit (known gap) | — |

Rate limiting is in-memory (JavaScript `Map` or Python slowapi) — resets on container restart. IP extraction reads `x-forwarded-for` header (known spoofable — see security posture document).

---

## Key Auth Files

| File | Purpose |
|------|---------|
| `portal/src/lib/auth.js` | `hashPassword()`, `verifyPassword()`, `signToken()`, `verifyToken()`, `validateSession()`, `requireAdmin()`, `getTokenFromRequest()` |
| `portal/src/middleware.js` | Next.js Edge middleware — JWT signature verification for protected page routes |
| `portal/src/hooks/useUser.js` | React hook — fetches user data from `/api/auth/me` (server-verified role, never trusts localStorage) |
| `heater/lib/auth.js` | JWT verify + session check + heater permission check |
| `pump/src/lib/auth.py` | JWT verify + session check + pump permission check (python-jose) |
| `massmole/lib/auth.js` | JWT verify + session check + massmole permission check |
| `optimizer/lib/auth.js` | JWT verify + session check + optimizer permission check |
