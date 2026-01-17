# Networking Cheatsheet

Essential networking concepts and troubleshooting for homelab deployments.

---

## 🌐 IP Address Basics

### IPv4 Address Structure

```
10.100.102.200/24
│   │   │   │   └─ Subnet mask (CIDR notation)
│   │   │   └───── Host address
│   │   └───────── Network segment
│   └───────────── Network segment
└───────────────── Private IP range (10.0.0.0/8)
```

### Private IP Ranges (RFC 1918)

| Range | CIDR | Usage |
|-------|------|-------|
| 10.0.0.0 - 10.255.255.255 | 10.0.0.0/8 | Large networks |
| 172.16.0.0 - 172.31.255.255 | 172.16.0.0/12 | Medium networks |
| 192.168.0.0 - 192.168.255.255 | 192.168.0.0/16 | Home networks (common) |

### Subnet Masks

| CIDR | Subnet Mask | Hosts | Usage |
|------|-------------|-------|-------|
| /24 | 255.255.255.0 | 254 | Standard home/small office |
| /16 | 255.255.0.0 | 65,534 | Large private networks |
| /8 | 255.0.0.0 | 16,777,214 | Very large networks |

---

## 🔍 DNS Concepts

### DNS Hierarchy

```
Internet
  ↓
Root Servers (.)
  ↓
TLD Servers (.com, .dev, .org)
  ↓
Authoritative Nameservers
  ↓
Your Domain (ofrikas.dev)
```

### Key Terminology

| Term | Definition | Example |
|------|------------|---------|
| **Registrar** | Where you buy the domain | name.com |
| **Nameserver** | Tells the internet who controls DNS | Cloudflare's NS |
| **DNS Record** | Maps names to addresses | A, CNAME, TXT |
| **Resolver** | Your DNS lookup service | 8.8.8.8 (Google) |

### Common DNS Record Types

| Type | Purpose | Example |
|------|---------|---------|
| **A** | Map domain to IPv4 | `example.com → 1.2.3.4` |
| **AAAA** | Map domain to IPv6 | `example.com → 2001:db8::1` |
| **CNAME** | Alias one domain to another | `www → example.com` |
| **MX** | Mail server | `mail.example.com priority 10` |
| **TXT** | Text data (verification, SPF) | `v=spf1 include:_spf.google.com` |

---

## 🚇 Cloudflare Tunnel Architecture

### Traditional Approach (Port Forwarding)

```
Internet → Your Public IP:443
  ↓
Router (NAT + Port Forward)
  ↓
Firewall (allow port 443)
  ↓
Reverse Proxy (nginx/Caddy)
  ↓
Your Application
```

**Problems:** Exposed IP, SSL management, firewall holes, DDoS risk

### Cloudflare Tunnel Approach

```
Internet → Cloudflare Edge
  ↓ (managed TLS)
Cloudflare Tunnel (encrypted)
  ↓ (outbound connection only!)
cloudflared container
  ↓ (Docker network)
Your Application
```

**Benefits:** Hidden IP, automatic SSL, no port forwarding, DDoS protection

### How Tunnels Work

1. **Outbound Connection:** `cloudflared` container connects TO Cloudflare (no inbound ports!)
2. **Registration:** Tunnel registers with Cloudflare using a token
3. **Traffic Flow:** Cloudflare routes traffic through the tunnel
4. **Encryption:** All traffic is encrypted end-to-end

---

## 🔧 Network Troubleshooting

### Step-by-Step Debugging

```bash
# 1. Check local network interface
ip a

# 2. Ping localhost
ping 127.0.0.1

# 3. Ping gateway
ping 10.100.102.1

# 4. Ping external IP
ping 8.8.8.8

# 5. Test DNS resolution
ping google.com

# 6. Check DNS server
cat /etc/resolv.conf

# 7. Test specific port
curl -I http://localhost:5678

# 8. Check listening ports
netstat -tuln | grep 5678
```

### Common Issues & Solutions

| Problem | Likely Cause | Solution |
|---------|--------------|----------|
| Can't ping gateway | Cable/interface down | Check physical connection, `ip a` |
| Can ping IP but not domain | DNS failure | Check `/etc/resolv.conf` |
| Different subnets | IP mismatch | Verify network config matches router |
| Port not accessible | Firewall/not listening | Check `netstat`, firewall rules |
| Container can't talk | Wrong network | Use `docker network` commands |

---

## 🐳 Docker Networking

### Network Types

| Type | Use Case | Isolation |
|------|----------|-----------|
| **bridge** | Default, containers on same host | Yes |
| **host** | Container uses host network directly | No |
| **none** | No networking | Complete |
| **custom** | User-defined bridge | Yes, configurable |

### Docker Network Commands

```bash
# List networks
docker network ls

# Inspect network
docker network inspect n8n_default

# Create network
docker network create my-network

# Connect container to network
docker network connect my-network container-name

# Disconnect
docker network disconnect my-network container-name
```

### Container Communication

**Same Network:**
```yaml
services:
  app:
    container_name: app
    networks:
      - my-net
  db:
    container_name: db
    networks:
      - my-net
```

App can reach DB at: `http://db:5432` (uses container name as hostname)

**Different Networks:**
Containers can't communicate unless both are on a shared network.

---

## 📊 Network Architecture (This Homelab)

```
Internet
  ↓
Cloudflare DNS (nameservers)
  ↓
Cloudflare Edge (TLS termination)
  ↓
Cloudflare Tunnel (encrypted WebSocket)
  ↓
Home Router (10.100.102.1)
  ↓
Proxmox Host (10.100.102.200)
  ↓
LXC Container 100 (10.100.102.15)
  ↓
Docker Bridge Network (n8n_default)
  ├─ n8n:5678
  ├─ postgres:5432
  └─ cloudflared (outbound only)
```

---

## 🔐 Security Best Practices

1. **Never expose Proxmox directly** — Keep 8006 on local network only
2. **Use tunnels over port forwarding** — Cloudflare/Tailscale/WireGuard
3. **Segment networks** — Use VLANs for IoT, guests, servers
4. **Change default ports** — But don't rely on security through obscurity
5. **Use strong DNS** — 8.8.8.8 (Google), 1.1.1.1 (Cloudflare), 9.9.9.9 (Quad9)
6. **Monitor logs** — Watch for failed connection attempts

---

## 📚 Useful Tools

| Tool | Purpose |
|------|---------|
| `ping` | Test basic connectivity |
| `traceroute` | Show network path |
| `dig` | DNS lookup tool |
| `nslookup` | DNS query tool |
| `netstat` | Show network connections |
| `ss` | Modern alternative to netstat |
| `tcpdump` | Packet capture |
| `nmap` | Network scanner |
| `curl` | HTTP client |
| `wget` | Download files |

---

## ⚠️ Common Mistakes

1. **Forgetting subnet masks:** `10.100.102.200/24` ≠ `192.168.100.200/24` (different networks!)
2. **Mixing public/private IPs:** Can't route private IPs over the internet
3. **DNS vs Nameservers:** Nameservers control DNS, DNS records point to IPs
4. **Cloudflare proxy on/off:** Orange cloud = proxied (secure), grey = DNS only
5. **Port conflicts:** Only one service can listen on a port at a time
