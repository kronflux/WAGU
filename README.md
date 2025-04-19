# WAGU - WireGuard, AdGuard, Unbound + Cloudflared

An all-in-one hardened, privacy-focused VPN + DNS stack powered by Docker Compose.

WAGU bundles:
- **[WireGuard](https://github.com/wireguard) + Web UI ([wg-easy](https://github.com/wg-easy/wg-easy))**: simple client management
- **[AdGuard Home](https://github.com/AdguardTeam/AdGuardHome)**: ad/tracker blocking DNS server with web UI
- **[Unbound](https://github.com/NLnetLabs/unbound)**: DNSSEC-validating, caching DNS resolver
- **[Cloudflared](https://github.com/cloudflare/cloudflared)**: secure DNS-over-HTTPS proxy (to Cloudflare, Quad9, etc.)

---

## 🌐 Features

- **Hardened container security**
  - Runs as non-root with `cap_drop`, `init`, and Docker secrets
  - Health checks for service readiness

- **Smart DNS routing architecture:**
  - `WireGuard clients` → `AdGuard` → `Unbound` → `Cloudflared` → DoH (Cloudflare)

---

## 🚀 Setup Instructions

### 1. Clone and Prepare Directory Structure
```bash
git clone https://github.com/kronflux/WAGU.git
cd WAGU

mkdir -p adguard/conf \
         adguard/work \
         wg-easy \
         unbound/conf.d \
         unbound/zones.d

echo "your_password" > .wg_password
chmod 600 .wg_password
```

### 2. Run AdGuard Home Setup Wizard
```bash
docker compose up -d --no-deps adguard
```

- Visit: `http://<hostIP>:3000`
- Choose a **username** and **password**
- For both DNS interfaces, select: `eth0`
- Finish the setup and log in at: `http://<hostIP>:80`

### 3. Configure AdGuard DNS Upstream
- Go to **Settings → DNS Settings**
- Set **Upstream DNS servers** to:
  ```
  10.2.0.200:5335
  ```
- Set **Bootstrap DNS server** to `1.1.1.1`
- Click **Apply**

### 4. Finalize Full Stack
```bash
docker compose down adguard
docker compose up -d
```

---

## 🔗 Accessing Services

Use the IP address of your Docker host (e.g., `http://192.168.X.X`) to access services:

- **AdGuard Home Web UI:** `http://<hostIP>:80`
- **AdGuard Setup Wizard (first time only):** `http://<hostIP>:3000`
- **WireGuard Web UI:** `http://<hostIP>:51821`
- **VPN Clients DNS Resolver:** `10.2.0.100`

---

## 🧪 Troubleshooting

### Cloudflared not resolving?
```bash
docker logs cloudflared
```

### Unbound health check failing?
```bash
docker exec -it unbound /usr/local/unbound/sbin/healthcheck.sh
```

### Can't resolve DNS via AdGuard?
```bash
docker exec -it adguard dig @10.2.0.200 example.com
```

---

## 🧱 Split Tunnel Mode
Only tunnel WAGU-related traffic:
```ini
AllowedIPs = 10.2.0.0/24
```

---

## 🔄 Updating
```bash
docker compose pull && docker compose up -d
```
To clean old images:
```bash
docker image prune
```

---

## 🌐 Dynamic DNS Setup
In `docker-compose.yml`, set your hostname:
```yaml
environment:
  WG_HOST: "your.ddns.net"
```

---

## 📒 FAQ

### Where do I set custom hostnames?
**AdGuard → Settings → DNS Settings → Custom DNS entries**

### Recommended Blocklists?
- https://firebog.net/
- https://github.com/anudeepND/whitelist

### Want to change DoH provider?
Modify Cloudflared command in `docker-compose.yml`.

---

## 🙏 Credits
Based on work by [@IAmStoxe](https://github.com/IAmStoxe) (WireHole), [@10h30](https://github.com/10h30) (WireHole UI)

---
