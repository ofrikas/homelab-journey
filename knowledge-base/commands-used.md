# Commands Used in Homelab Project

All commands actually used throughout chapters 0-3, organized chronologically with explanations.

---

## Chapter 1: Proxmox Installation & Network Troubleshooting

### Network Diagnostics

```bash
ip a
```
Show all network interfaces and their IP addresses. Used to diagnose interface mismatch (nic0 vs nic1).

```bash
ping 8.8.8.8
```
Test internet connectivity by pinging Google's DNS. Used to verify physical connection works.

```bash
ping google.com
```
Test DNS resolution. Used to discover DNS configuration was broken.

### Network Configuration

```bash
nano /etc/network/interfaces
```
Edit network interface configuration file. Changed IP from 192.168.100.200/24 to 10.100.102.200/24 to match home network.

**Key steps in nano:**
- `Ctrl+O` then `Enter` to save
- `Ctrl+X` to exit

```bash
ifreload -a
```
Reload network configuration without rebooting. Applied new IP address settings immediately.

```bash
nano /etc/resolv.conf
```
Edit DNS resolver configuration. Added `nameserver 8.8.8.8` to fix domain name resolution.

### Post-Install Script

```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/tools/pve/post-pve-install.sh)"
```
Run Proxmox post-install script. Removed subscription nag, enabled free repositories, updated system.

---

## Chapter 2: Docker & n8n Setup

### Docker Installation

```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/ct/docker.sh)"
```
Create and configure LXC container with Docker pre-installed. Community script that sets up everything automatically.

### Container Access

```bash
pct enter 100
```
Enter LXC container 100 (Docker host). Faster than SSH for local Proxmox management.

### Network Verification

```bash
ip a | grep 10.100.102.15 -n || hostname -I
```
Check container's IP address. The `||` means "if grep fails, run hostname -I instead".

```bash
docker --version
```
Verify Docker installation. Confirmed Docker 29.1.4 was installed.

### Directory Setup

```bash
mkdir -p /opt/n8n && cd /opt/n8n
```
Create n8n directory and navigate into it. The `-p` flag creates parent directories if needed.

```bash
mkdir -p /opt/repos/n8n-stack
```
Create directory for Git repository. Organized structure: `/opt/n8n` for deployment, `/opt/repos/n8n-stack` for version control.

```bash
cd /opt/repos/n8n-stack
```
Navigate to repository directory.

### Docker Compose Operations

```bash
docker compose up -d
```
Start all services in detached mode (background). The `-d` flag is critical — without it, containers stop when you close the terminal.

```bash
docker compose down
```
Stop and remove all containers. Doesn't remove volumes by default.

```bash
docker compose down -v
```
Stop containers AND remove volumes. **Dangerous!** Used when starting fresh after changing PostgreSQL password or encryption keys.

```bash
docker compose ps
```
List running containers and their status. Quick health check for the stack.

```bash
docker compose logs --tail=200 postgres
```
View last 200 lines of PostgreSQL logs. Used for debugging database startup issues.

```bash
docker compose logs --tail=50 n8n
```
View last 50 lines of n8n logs. Used to check for connection errors.

### Docker Volume Management

```bash
docker volume rm n8n_n8n_data
```
Remove n8n data volume. Used when encryption key changed and n8n wouldn't start.

```bash
docker volume rm n8n_postgres_data
```
Remove PostgreSQL data volume. Used when password mismatch occurred (Postgres initializes password only once).

### Testing n8n

```bash
curl -I http://127.0.0.1:5678/ | head -n 1
```
Get HTTP status code from n8n. Returns `HTTP/1.1 200 OK` if working. The `| head -n 1` shows only the first line.

### File Creation

```bash
cat > .env.example << 'EOF'
POSTGRES_USER=n8n
POSTGRES_PASSWORD=REPLACE_ME
...
EOF
```
Create file using heredoc. Everything between `EOF` markers goes into the file. The quotes around `'EOF'` prevent variable expansion.

```bash
cat > .gitignore << 'EOF'
.env
*.log
EOF
```
Create .gitignore file to prevent committing secrets.

```bash
cat > README.md << 'EOF'
# n8n stack (home)
...
EOF
```
Create README with usage instructions.

```bash
cp /opt/n8n/docker-compose.yml .
```
Copy docker-compose file from deployment to repository. The `.` means "current directory".

### Git Setup

```bash
apt update
```
Refresh package lists. Always run before installing new packages.

```bash
apt install -y git
```
Install Git. The `-y` flag auto-confirms installation (no prompt).

```bash
git init
```
Initialize new Git repository in current directory.

```bash
git add .
```
Stage all files for commit. The `.` means "everything in current directory".

```bash
git commit -m "Initial n8n stack"
```
Commit staged files with message. The `-m` flag provides commit message inline.

### SSH Key Generation

```bash
ssh-keygen -t ed25519 -C "n8n-stack" -f ~/.ssh/id_ed25519 -N ""
```
Generate SSH key pair. Flags explained:
- `-t ed25519` = key type (modern, secure)
- `-C "n8n-stack"` = comment (identifies key purpose)
- `-f ~/.ssh/id_ed25519` = filename
- `-N ""` = no passphrase (empty string)

```bash
cat ~/.ssh/id_ed25519.pub
```
Display public key. Copy this to GitHub → Settings → SSH Keys.

```bash
ssh -T git@github.com
```
Test SSH connection to GitHub. Returns "Hi username!" if working.

### Git Remote Operations

```bash
git remote add origin git@github.com:ofrikas/homelab-n8n-stack.git
```
Add remote repository. Named "origin" by convention.

```bash
git remote set-url origin git@github.com:ofrikas/homelab-n8n-stack.git
```
Change remote URL. Used when repository was created after initial attempt.

```bash
git push -u origin main
```
Push to remote and set upstream. The `-u` flag remembers the remote for future `git push` commands.

```bash
git push -u origin main --force
```
Force push to override remote. Used because GitHub repo had a README that conflicted with local. **Dangerous!** Only use when you know what you're doing.

### Security

```bash
openssl rand -hex 32
```
Generate 32-byte random hex string. Used for encryption keys (64 characters output).

```bash
openssl rand -base64 24
```
Generate 24-byte random base64 string. Alternative for passwords (32 characters output).

```bash
nano .env
```
Edit environment file with secrets. Never commit this file!

---

## Chapter 3: Cloudflare Tunnel

### Cloudflare Tunnel Setup

```bash
docker run -d \
  --name cloudflared \
  --restart unless-stopped \
  --network n8n_default \
  cloudflare/cloudflared:latest \
  tunnel --no-autoupdate run --token <YOUR_TUNNEL_TOKEN>
```
Run Cloudflare tunnel container. Flags explained:
- `-d` = detached mode (background)
- `--name cloudflared` = container name
- `--restart unless-stopped` = auto-restart on crash/reboot
- `--network n8n_default` = join n8n's Docker network
- `--no-autoupdate` = manual control of updates
- `--token <...>` = authentication token from Cloudflare

### Docker Inspection

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```
List containers with custom formatting. Shows only name, image, and ports in a clean table.

```bash
docker network ls
```
List all Docker networks. Used to verify n8n_default network exists.

---

## Useful Search Commands

### Find Docker Compose Files

```bash
find / -maxdepth 4 -type f \( -name "docker-compose.yml" -o -name "compose.yml" \) 2>/dev/null
```
Search for docker-compose files across system. Flags explained:
- `/` = start from root
- `-maxdepth 4` = only search 4 levels deep (faster)
- `-type f` = files only (not directories)
- `-name "docker-compose.yml" -o -name "compose.yml"` = match either filename
- `2>/dev/null` = hide permission errors

---

## Quick Reference

### Most Used Commands

| Command | Purpose |
|---------|---------|
| `docker compose up -d` | Start services (background) |
| `docker compose down` | Stop services |
| `docker compose ps` | Check container status |
| `docker compose logs -f <service>` | Follow service logs |
| `pct enter 100` | Enter Docker LXC container |
| `ip a` | Show network interfaces |
| `nano <file>` | Edit file (Ctrl+O save, Ctrl+X exit) |
| `cat ~/.ssh/id_ed25519.pub` | Show SSH public key |

### Emergency Recovery

```bash
# If n8n won't start due to encryption key:
docker compose down
docker volume rm n8n_n8n_data
docker compose up -d

# If PostgreSQL password mismatch:
docker compose down
docker volume rm n8n_postgres_data
docker compose up -d
```

---

## Tips & Tricks

1. **Tab completion:** Press `Tab` to auto-complete filenames and commands
2. **Command history:** Press `↑` to recall previous commands
3. **Cancel command:** Press `Ctrl+C` to stop a running command
4. **Exit logs:** Press `Ctrl+C` when following logs with `-f` flag
5. **Copy from terminal:** Select text to auto-copy (Linux), then middle-click to paste

ls -la
docker ps --format "table {{.Names}}\t{{.Ports}}"