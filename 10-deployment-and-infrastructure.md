# Deployment & Infrastructure

**Last Generated:** 2026-03-14

---

## Two VMs

### EC2 — Staging

| Property | Value |
|----------|-------|
| **IP** | 18.198.1.231 |
| **URL** | https://eprom-portal.xyz |
| **Region** | Frankfurt (eu-central-1b) |
| **OS** | Ubuntu 24.04 LTS |
| **Instance** | t3.micro (2 vCPU, 1 GB RAM + 2 GB swap, 30 GB gp3) |
| **SSH** | `ssh -i ~/eprom-aws-key.pem ubuntu@18.198.1.231` |
| **Docker** | 28.2.2 |
| **SSL** | Let's Encrypt (auto-renew via Certbot), valid until May 2026 |
| **Firewall** | UFW: ports 22, 80, 443 |
| **Services** | Docker Compose v2: 6 containers (db, portal, heater, pump, massmole, optimizer) |

### Company KVM — Production

| Property | Value |
|----------|-------|
| **IP** | 192.168.50.202 (private network) |
| **URL** | https://ese.eprom.com.eg (pending DNS + SSL) |
| **OS** | Ubuntu 24.04.4 LTS |
| **Virtualization** | KVM (QEMU i440FX) |
| **Specs** | 8 CPU, 24 GB RAM + 4 GB swap, 80 GB disk |
| **SSH** | `sshpass -p '123456' ssh ai@192.168.50.202` |
| **Docker** | 29.3.0, Compose v5.1.0 |
| **SSL** | Pending (company cert or Let's Encrypt) |
| **Firewall** | UFW: ports 22, 80, 443 |
| **Resource limits** | docker-compose.override.yml: 19 GB RAM / 8 CPU allocated to containers |
| **Backups** | Daily cron at 2 AM, 7-day retention (~20 MB each) |
| **Reports** | 370 MB static HTML in /opt/eprom/reports/ (27 reports) |
| **Nginx** | 1.24.0, proxying all 5 apps + serving static reports |

---

## Docker Compose

**Important:** Use `docker compose` (v2 plugin, no hyphen) — NOT `docker-compose` (v1 is broken with `KeyError: 'ContainerConfig'`).

**Compose file:** `/opt/eprom/docker-compose.yml`

| Service | Image | Port (host:container) | Depends On |
|---------|-------|----------------------|------------|
| `db` | postgres:16-alpine | (internal only) :5432 | — |
| `portal` | Built from `./portal` | 127.0.0.1:3000:3000 | db (healthy) |
| `heater` | Built from `./heater` | 127.0.0.1:3001:3001 | db (healthy) |
| `pump` | Built from `./pump` | 127.0.0.1:3005:3005 | db (healthy) |
| `massmole` | Built from `./massmole` | 127.0.0.1:3006:3006 | db (healthy) |
| `optimizer` | Built from `./optimizer` | 127.0.0.1:3007:3007 | db (healthy) |

All services use `restart: unless-stopped`. The database has a health check (`pg_isready`); application containers do not (known gap).

**Network:** `eprom_net` (bridge driver). All containers communicate internally. Only Nginx exposes services to the internet.

**Volume:** `db_data` persists PostgreSQL data across container restarts. Database initialization SQL is mounted from `./db/init.sql`.

---

## Deployment Workflow

### Step 1: Sync Code (rsync)

**CRITICAL: NEVER use `rsync --delete`** — it previously destroyed critical VM-only files (db.js, auth.js, logout, me endpoints). Recovery was only possible because the Docker container was still running with the old code.

```bash
# To EC2 (staging):
rsync -avz -e "ssh -i ~/eprom-aws-key.pem" \
  portal/src/ ubuntu@18.198.1.231:/opt/eprom/portal/src/

# To Company VM (production):
rsync -avz -e "sshpass -p '123456' ssh -o StrictHostKeyChecking=no" \
  portal/src/ ai@192.168.50.202:/opt/eprom/portal/src/
```

Replace `portal/src/` with the appropriate app directory:
- `heater/` (exclude `node_modules`, `.git`)
- `pump/` (exclude `__pycache__`, `.venv`, `.git`)
- `massmole/` (exclude `node_modules`, `.git`)
- `optimizer/` (exclude `node_modules`, `.git`)

### Step 2: Build + Restart

```bash
# On EC2:
ssh -i ~/eprom-aws-key.pem ubuntu@18.198.1.231 \
  "cd /opt/eprom && docker compose up -d --build portal"

# On Company VM:
sshpass -p '123456' ssh -o StrictHostKeyChecking=no ai@192.168.50.202 \
  "cd /opt/eprom && docker compose up -d --build portal"
```

### Step 3: Verify

```bash
# Check container is running:
docker logs eprom_portal --tail 20

# Check health endpoint:
curl http://localhost:3000/api/health
```

---

## Files That Are NEVER Synced Between VMs

| File | Why |
|------|-----|
| `.env` | Different passwords, URLs, SMTP credentials per environment |
| `docker-compose.override.yml` | Only exists on company VM (resource limits) |
| Nginx site config | Different server_name, SSL configuration |
| `/opt/eprom/backups/` | Local to each VM |
| `/opt/eprom/reports/` | 370 MB static HTML, local to each VM |

---

## Domains & SSL

| Domain | Points To | Status |
|--------|----------|--------|
| `eprom-portal.xyz` | EC2 (18.198.1.231) | Active, Let's Encrypt cert, auto-renew |
| `ese.eprom.com.eg` | Company VM (pending) | DNS coordination with Telecom Egypt (WE) required |

The Let's Encrypt certificate on EC2 is valid until May 22, 2026 and auto-renews via Certbot. The company VM will need either a company-provided SSL certificate or a new Let's Encrypt setup once the domain is pointed.

---

## Nginx Configuration

Nginx runs as a system service (not in Docker). It handles:
- TLS termination
- Reverse proxying to all 5 app containers
- Static file serving for reports
- Security header injection
- Proxy buffer sizing (128k buffer, 256k busy)

On EC2, two Nginx configs exist:
- `eprom-portal` — domain-based (HTTPS, eprom-portal.xyz)
- `eprom-ip` — IP-based fallback (HTTP, 18.198.1.231)

---

## Backup Strategy

**Company VM only** (production):

```bash
# Daily cron at 2 AM:
docker exec eprom_db pg_dump -U $DB_USER $DB_NAME > /opt/eprom/backups/eprom_db_$DATE.sql
find /opt/eprom/backups/ -name "*.sql" -mtime +7 -delete
```

**Known issues:**
- Backups are unencrypted plain SQL
- Stored on the same disk as production data (not off-site)
- ~20 MB per dump, 7-day retention
