# Docker Commands Reference

Quick reference for Docker and Docker Compose commands used in this homelab project.

---

## 🐳 Docker Compose Commands

### Stack Management

| Command | Description |
|---------|-------------|
| `docker compose up -d` | Start all services in detached (background) mode |
| `docker compose down` | Stop and remove all containers |
| `docker compose down -v` | Stop containers AND remove volumes (fresh start) |
| `docker compose ps` | List running containers and their status |
| `docker compose restart` | Restart all services |

### Logs & Debugging

| Command | Description |
|---------|-------------|
| `docker compose logs` | View all service logs |
| `docker compose logs -f` | Follow logs in real-time (Ctrl+C to exit) |
| `docker compose logs -f n8n` | Follow logs for specific service |
| `docker compose logs --tail=200 postgres` | Show last 200 lines for a service |

### Updates & Maintenance

| Command | Description |
|---------|-------------|
| `docker compose pull` | Download latest images (based on compose file) |
| `docker compose pull && docker compose up -d` | Update and restart with new images |

---

## 🔍 Docker Core Commands

### Container Management

| Command | Description |
|---------|-------------|
| `docker ps` | List running containers |
| `docker ps -a` | List all containers (including stopped) |
| `docker exec -it <container> bash` | Open shell inside running container |
| `docker stop <container>` | Stop a running container |
| `docker rm <container>` | Remove a stopped container |

### Volume Management

| Command | Description |
|---------|-------------|
| `docker volume ls` | List all volumes |
| `docker volume rm <volume_name>` | Remove a specific volume |
| `docker volume prune` | Remove all unused volumes (dangerous!) |

### Image Management

| Command | Description |
|---------|-------------|
| `docker images` | List downloaded images |
| `docker rmi <image>` | Remove an image |
| `docker image prune` | Remove unused images |

### System Information

| Command | Description |
|---------|-------------|
| `docker --version` | Check Docker version |
| `docker info` | Detailed Docker system information |
| `docker system df` | Show Docker disk usage |

---

## 🖥️ Proxmox LXC Commands

| Command | Description |
|---------|-------------|
| `pct enter <vmid>` | Enter an LXC container shell (e.g., `pct enter 100`) |
| `pct list` | List all LXC containers |
| `pct start <vmid>` | Start a container |
| `pct stop <vmid>` | Stop a container |

---

## 🌐 Network Debugging

| Command | Description |
|---------|-------------|
| `ip a` | Show all network interfaces and IPs |
| `hostname -I` | Show only IP addresses |
| `curl -I http://127.0.0.1:5678/` | Check HTTP response headers |
| `curl -sS -o /dev/null -w "%{http_code}\n" <url>` | Get only HTTP status code |
| `docker network ls` | List all Docker networks |
| `docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"` | List containers with clean formatting |

---

## 🚇 Cloudflare Tunnel Commands

| Command | Description |
|---------|-------------|
| `docker run -d --name cloudflared --restart unless-stopped --network <network> cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <token>` | Run Cloudflare tunnel container |
| `docker logs cloudflared` | Check tunnel connection status |
| `docker restart cloudflared` | Restart tunnel after config changes |

**Tunnel flags explained:**
- `--network n8n_default` — Join same network as services to expose
- `--restart unless-stopped` — Auto-restart on crash/reboot
- `--no-autoupdate` — Disable auto-updates (manual control)

---

## 🔐 Git Commands

### Repository Setup

| Command | Description |
|---------|-------------|
| `git init` | Initialize a new Git repository |
| `git add .` | Stage all files for commit |
| `git commit -m "message"` | Commit staged changes |
| `git remote add origin <url>` | Add remote repository |
| `git remote set-url origin <url>` | Change remote URL |

### Push & Pull

| Command | Description |
|---------|-------------|
| `git push -u origin main` | Push to remote (first time, sets upstream) |
| `git push -u origin main --force` | Force push (overwrite remote) |
| `git pull` | Fetch and merge remote changes |

### SSH Key Setup

| Command | Description |
|---------|-------------|
| `ssh-keygen -t ed25519 -C "comment" -f ~/.ssh/id_ed25519 -N ""` | Generate SSH key pair |
| `cat ~/.ssh/id_ed25519.pub` | Display public key (add to GitHub) |
| `ssh -T git@github.com` | Test GitHub SSH connection |

---

## 🔧 System Commands

| Command | Description |
|---------|-------------|
| `apt update` | Refresh package lists |
| `apt install -y <package>` | Install package without confirmation |
| `nano <file>` | Edit file in terminal (Ctrl+O save, Ctrl+X exit) |
| `mkdir -p /path/to/dir` | Create directory (and parents if needed) |
| `cat > file << 'EOF' ... EOF` | Create file with heredoc content |
| `openssl rand -hex 32` | Generate 32-byte random hex string |
| `openssl rand -base64 24` | Generate 24-byte random base64 string |

---

## ⚠️ Common Gotchas

1. **Volume persistence:** PostgreSQL only reads `POSTGRES_PASSWORD` on first initialization. Changing it later requires removing the volume.

2. **Detached mode:** Always use `-d` flag for production (`docker compose up -d`), otherwise containers stop when you close the terminal.

3. **Force flag:** `docker compose down -v` removes volumes! All data is lost. Use carefully.

4. **SSH keys:** The public key (`.pub`) goes to GitHub. Never share the private key.

5. **Environment variables:** Avoid special characters (`$`, `!`, `` ` ``) in passwords — they cause shell escaping issues.

6. **Docker networks:** Containers on different networks can't communicate. Use `--network` flag to join existing networks.

7. **Cloudflare tokens:** Never commit tunnel tokens to Git. Use environment variables or `.env` files (in `.gitignore`).
