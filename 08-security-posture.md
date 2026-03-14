# Security Posture

**Last Generated:** 2026-03-14

---

## What's Strong

The platform has solid security fundamentals for an internal engineering tool:

1. **Zero OWASP code vulnerabilities** — Semgrep scan with 7 rulesets found no SQL injection, XSS, SSRF, or insecure deserialization in application code
2. **All SQL queries parameterized** — `$1, $2` placeholders throughout, no string interpolation anywhere
3. **Solid auth architecture** — JWT + httpOnly/secure cookies + DB session validation + bcrypt (cost factor 12)
4. **Server-side IP protection** — All 406 heater formulas, 53 pump formulas, 160 compound calculations, and ML optimization run server-side only; zero business logic in browser JavaScript
5. **TLS properly configured** — TLSv1.3, Let's Encrypt certificate, HSTS with includeSubDomains, HTTP→HTTPS redirect
6. **5 of 7 security headers present** — X-Frame-Options, X-Content-Type-Options, HSTS, Referrer-Policy, Permissions-Policy
7. **Network isolation** — All Docker containers bind to `127.0.0.1` only, Nginx proxies all traffic
8. **Firewall locked down** — UFW allows only ports 22, 80, 443
9. **System hardening** — fail2ban active for SSH brute-force protection, unattended-upgrades enabled

---

## Known Issues — HIGH Severity

### H1. Outdated Dependencies with CVEs

| Package | Version | Risk | Fix |
|---------|---------|------|-----|
| `next` | 14.2.5 | **Authorization bypass in middleware** (GHSA-f82v-jwr5-mffw) — could let attackers bypass JWT verification entirely. Also: cache poisoning, SSRF. Total: 1 critical + 5 high CVEs. | Upgrade to `next@14.2.35+` |
| `python-jose` | 3.3.0 | **JWT algorithm confusion** — attacker can craft JWTs that bypass signature verification. Unmaintained since 2021. | Replace with `PyJWT` |
| `fastapi` | 0.104.1 | ReDoS via Content-Type parsing | Upgrade to `≥0.115` |
| `starlette` | 0.27.0 | Unbounded multipart form memory (OOM DoS) | Upgrade to `≥0.47.2` |

### H2. Dynamic Code Execution — `new Function()`

**File:** `heater/engine/heater-engine.js:139`

The heater engine translates Excel-style formulas to JavaScript and executes them via `new Function()` (equivalent to `eval()`). Validation is limited to balanced-parentheses checking. Formulas come from internal spec files (not user input), but if the spec source is ever tampered with, arbitrary JavaScript executes with full server privileges.

**Recommended fix:** Replace with a strict math expression evaluator (e.g., `mathjs` or `expr-eval`).

### H3. All 4 Containers Run as Root

All Dockerfiles lack a `USER` directive. Verified: `docker exec <container> whoami` → `root` for all 4 app containers. If an attacker achieves code execution inside any container, they run as root.

### H4. Rate Limiter IP Spoofable

Rate limiters read `x-forwarded-for` (a client-controlled header) for IP identification. An attacker can set this header to any value, completely bypassing rate limits. Express services don't call `app.set('trust proxy', 1)`.

---

## Known Issues — MEDIUM Severity

| ID | Issue | Risk |
|----|-------|------|
| M1 | **No CSRF token protection** | SameSite=Lax blocks most cross-site POSTs, but top-level navigation POSTs and older browsers remain vulnerable |
| M2 | **In-memory rate limiting** resets on container restart | Attacker can trigger restart via resource exhaustion, then brute-force |
| M3 | **No Content-Security-Policy header** | Any XSS has unrestricted capability — can load external scripts |
| M4 | **Prompt injection detected but not blocked** | Detected injection patterns are logged but still sent to the AI model. System prompt guardrails are advisory only |
| M5 | **No Docker resource limits** (EC2 only) | One container can consume all memory (company VM has override limits) |
| M6 | **Email templates interpolate unsanitized user data** | XSS in admin notification emails |
| M7 | **Unbounded in-memory conversation store** | Chat history grows forever, no eviction, lost on restart |
| M8 | **No lock files committed** | Non-reproducible builds, vulnerable to dependency substitution |
| M9 | **No signup rate limiting** | Unlimited signup requests enable email bombing and junk accounts |
| M10 | **Nginx server version disclosed** | `server_tokens off;` is commented out |

---

## Pre-Production Audit Checklist

Before the domain `ese.eprom.com.eg` goes live, a 9-phase security audit must be completed:

1. **Infrastructure & OS Hardening** — Updates, permissions, cron audit
2. **Firewall & Network** — UFW rules, Docker iptables bypass check, fail2ban
3. **SSL/TLS & Nginx** — Certificate, HTTPS redirect, cipher suites, security headers
4. **Docker Security** — Non-root users, no privileged containers, resource limits, health checks
5. **Portal Application Security** — JWT cookies, session validation, admin middleware, CORS
6. **App API Security** — Auth on all routes, permission checks, no client-side formulas
7. **Database Security** — Network isolation, strong credentials, backup encryption
8. **Secrets & Environment** — `.env` permissions, strong JWT_SECRET, no hardcoded secrets
9. **Monitoring & Incident Response** — Log rotation, activity logging, breach response plan

**60+ checklist items total.** All critical/high issues must be resolved before go-live.

---

## IT Meeting Security Requirements

The IT department explicitly mandated:
- **WAF** (Web Application Firewall) for filtering
- **HTTPS** enforced on all traffic
- **2FA** planned for future implementation
- **SSH with private keys** only (AnyDesk explicitly rejected — "forbidden by the Ministry, lacks encryption")
- **DMZ hosting** — server sees only the internet, not the internal corporate network
- **Reputation risk awareness** — a compromised server could blacklist EPROM's corporate IP globally, halting all email and web communications
