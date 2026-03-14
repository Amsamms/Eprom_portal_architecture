---
name: update-knowledge-base
description: "Regenerate all 13 ChatGPT knowledge base .md files, the PROJECT-INSTRUCTIONS.md, and the eprom-architecture-guide.html by deeply exploring the EPROM Portal codebase for changes. Use when the user says 'update knowledge base', 'update structure', 'refresh docs', or 'sync knowledge base'."
---

# Update EPROM Portal Knowledge Base

You are updating the ChatGPT knowledge base files in this repository (`Eprom_portal_architecture`). These files are uploaded to a ChatGPT shared project so collaborators have full context about the EPROM Portal.

## Step 1: Explore the EPROM Portal Codebase for Changes

The source codebase lives at `/home/amsamms/projects/Eprom_portal/`. Read these files to detect what changed:

1. **Always read first:**
   - `/home/amsamms/projects/Eprom_portal/PROGRESS.md` — current status, session history, app architecture, tech stack
   - `/home/amsamms/projects/Eprom_portal/CLAUDE.md` — project instructions, deployment info
   - `/home/amsamms/projects/Eprom_portal/ideas/README.md` — current ideas backlog

2. **Read if relevant sections changed:**
   - `docker-compose.yml` — container changes
   - `db/init.sql` — database schema changes
   - `nginx/sites/portal.conf` — routing changes
   - `portal/src/app/globals.css` — design system changes
   - `security/` folder — security scan updates
   - `ideas/` folder — new/implemented ideas
   - Meeting minutes `.md` files — business context changes
   - `VM_MIGRATION_PLAN.md` — infrastructure changes

3. **Deep explore apps for detailed changes** (use Agent tool with Explore subagent for each):
   - `heater/` — engine, AI tools, routes, CSS
   - `pump/` — engine, AI tools, routes
   - `massmole/` — engine, AI tools, compounds
   - `optimizer/` — AI tools, PHP API, workflow
   - `shared/` — chat-base.js, visual-actions-base.js
   - `portal/src/` — auth, middleware, hooks, API routes

## Step 2: Compare with Current Knowledge Base

Read each of the 13 existing `.md` files + `PROJECT-INSTRUCTIONS.md` + `eprom-architecture-guide.html` in this repo. Identify what's outdated or missing.

## Step 3: Update All Files

Update each file that needs changes. The 13 knowledge base files are:

| File | Primary Sources |
|------|----------------|
| `00-project-overview.md` | PROGRESS.md, Meeting minutes |
| `01-architecture-overview.md` | docker-compose.yml, nginx config, PROGRESS.md |
| `02-design-system.md` | heater/public/css/main.css, portal globals.css, optimizer CSS |
| `03-app-heater.md` | heater/ source files |
| `04-app-pump.md` | pump/src/ source files |
| `05-app-massmole-optimizer.md` | massmole/ and optimizer/ source files |
| `06-ai-chat-system.md` | shared/chat-base.js, shared/visual-actions-base.js, app chat configs |
| `07-database-and-auth.md` | db/init.sql, portal/src/lib/auth.js, portal/src/middleware.js |
| `08-security-posture.md` | security/ folder, ideas/pre-production-security-audit.md |
| `09-roadmap-and-ideas.md` | ideas/README.md, ideas/*.md, ideas/implemented/ |
| `10-deployment-and-infrastructure.md` | CLAUDE.md, VM_MIGRATION_PLAN.md |
| `11-decisions-and-constraints.md` | Meeting minutes, PROGRESS.md |
| `PROJECT-INSTRUCTIONS.md` | Synthesized from all sources |

### Rules for updating:

- Update the `Last Generated` date at the top of each file to today's date
- Keep files self-contained and readable without the others
- Don't repeat the same information across files
- Verify accuracy: hex codes, port numbers, table names, file paths must match actual codebase
- No single file should exceed ~5 pages
- Use exact tool names, formula counts, and KPI names from the actual code (not from PROGRESS.md summaries)
- For AI tools: read the actual `tool-definitions.js` or `tool_definitions.py` files to get correct tool names

### Rules for the HTML file:

- Update `eprom-architecture-guide.html` with any structural changes (new containers, new apps, updated counts)
- Update the "Last updated" date
- Keep the same visual style and collapsible sections

## Step 4: Git Commit and Push

After all files are updated:

1. `git add` all changed files
2. Commit with message: `docs: update knowledge base — [brief summary of what changed]`
3. `git push origin main`

## Step 5: Report

Tell the user:
- Which files were updated and why
- Any new information discovered
- Anything that needs manual attention (e.g., the user needs to re-upload files to ChatGPT project)
