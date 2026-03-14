# EPROM Portal Architecture — Knowledge Base Repository

## What This Repo Is

This repository contains **13 markdown knowledge base files + 1 instructions file + 1 HTML visual guide** that are uploaded to a ChatGPT shared project. The purpose is to give collaborators full context about the EPROM Portal so they can make design decisions with AI mediation.

## Source of Truth

The actual EPROM Portal codebase lives at: `/home/amsamms/projects/Eprom_portal/`

This repo (`Eprom_portal_architecture`) contains **generated documentation** derived from that codebase. When the portal changes, run `/update-knowledge-base` to regenerate these files.

## Files

| File | Content |
|------|---------|
| `00-project-overview.md` | Executive summary, business context |
| `01-architecture-overview.md` | System topology, containers, tech stack |
| `02-design-system.md` | Colors, typography, CSS tokens, components |
| `03-app-heater.md` | Heater app deep dive |
| `04-app-pump.md` | Pump app deep dive |
| `05-app-massmole-optimizer.md` | MassMole + Optimizer apps |
| `06-ai-chat-system.md` | Shared chat library, AI tools, token optimization |
| `07-database-and-auth.md` | Database schema, auth flow, subscriptions |
| `08-security-posture.md` | Security strengths, known issues, audit checklist |
| `09-roadmap-and-ideas.md` | Prioritized backlog, implemented features |
| `10-deployment-and-infrastructure.md` | VMs, Docker, rsync, domains, backups |
| `11-decisions-and-constraints.md` | IT meeting outcomes, constraints |
| `PROJECT-INSTRUCTIONS.md` | ChatGPT project custom instructions |
| `eprom-architecture-guide.html` | Visual architecture diagram (open in browser) |

## Workflow

1. Make changes to the EPROM Portal (`/home/amsamms/projects/Eprom_portal/`)
2. Come to this folder and run `/update-knowledge-base`
3. Re-upload the updated `.md` files to the ChatGPT shared project
