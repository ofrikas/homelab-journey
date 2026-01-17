# Copilot Instructions for homelab-journey

A **personal learning journal** documenting my journey building a self-hosted homelab. This is both a story and a technical reference — written so I can revisit it when I forget, and others can learn from my experience.

## Purpose & Tone

This repo tells my project story — personal yet professional. Every chapter should:
- Explain **what I learned** and **why things work** (not just commands to copy)
- Document **failures and troubleshooting** — the struggle is part of the story
- Be useful as a future reference when I forget how something works
- Help others understand what I went through and learn from it

## Repository Structure

- `chapters/` — Sequential journey (00-03 are complete; later chapters are ideas that will evolve)
- `configs/` — Working configuration files, organized by component
- `knowledge-base/` — Reference docs and cheatsheets (commands, networking, Docker)
- `assets/screenshots/chapter-XX/` — Screenshots created per chapter as it's written
- `assets/diagrams/` — ASCII and Mermaid architecture diagrams

## Writing Conventions

### Chapter Files
- Use emoji prefixes for section headers (🎯 🐳 📦 🔧 etc.)
- Start with **"Why"** before **"How"** — explain the reasoning, then the steps
- Document **decisions as tables**: Prompt | Choice | Reasoning
- Include **what went wrong** and how I fixed it — not just the happy path
- Reference configs with relative links: `[docker-compose.yml](../configs/docker/n8n-stack/docker-compose.yml)`
- Add screenshots for UI steps: `![Description](../assets/screenshots/chapter-XX/filename.png)`

### Configuration Files
- Use `.env` files with `.template` versions committed (never commit secrets)
- Pin specific versions in Docker (e.g., `n8nio/n8n:2.3.4`), not `latest`
- Include health checks for database dependencies
- Configs in `configs/system/` are populated when their chapter is written

### Commands Documentation
- Every command in `knowledge-base/commands-used.md` includes what it does and when to use it
- Update commands-used.md when writing new chapters

## Architecture Quick Reference

```
Internet → Cloudflare DNS → Cloudflare Tunnel (outbound only)
    → Home Network (10.100.102.0/24)
    → Proxmox Host (10.100.102.200)
    → LXC Container 100 (10.100.102.15)
    → Docker: cloudflared ↔ n8n:5678 ↔ postgres:5432
```

Key IPs: Proxmox `10.100.102.200` | Docker LXC `10.100.102.15` | Gateway `10.100.102.1`

## Key Patterns

1. **No port forwarding** — External access via Cloudflare Tunnel only
2. **LXC containers** over VMs for Docker workloads
3. **Community scripts** for Proxmox: `github.com/community-scripts/ProxmoxVE`
4. **Directory convention**: `/opt/<service>` for deployment, `/opt/repos/<stack>` for git repos
