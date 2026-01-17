# Proxmox VM/LXC Container Notes

## LXC Container 100 - Docker Host

**Purpose:** Docker container host for n8n, PostgreSQL, and Cloudflare tunnel

**Specifications:**
- Container ID: 100
- OS Template: Debian (created via community script)
- IP Address: 10.100.102.15/24
- Gateway: 10.100.102.1
- DNS: 8.8.8.8
- Memory: 2GB (adjust as needed)
- Disk: 8GB (adjust as needed)
- Features: Nesting enabled (for Docker)

**Installed Software:**
- Docker 29.1.4
- Docker Compose
- Portainer (optional, for web UI)
- Git (for version control)

**Running Services:**
- n8n (container: n8n, port 5678)
- PostgreSQL 18 (container: n8n_postgres)
- Cloudflare Tunnel (container: cloudflared)

**Access:**
```bash
pct enter 100
```

**Important Directories:**
- `/opt/n8n/` - n8n stack deployment
- `/opt/repos/n8n-stack/` - Git repository for n8n configs
- `/root/.ssh/` - SSH keys for GitHub access

**Docker Networks:**
- n8n_default (bridge network for n8n stack)

---

## Future Containers/VMs

To be added as the homelab grows:
- Monitoring stack (Prometheus, Grafana)
- AI/ML workloads (with GPU passthrough)
- Development environments
- Testing/staging instances
