# Quick Reference: URLs, Commands, Network

> ⚠️ **If you lose internet connection:** Pi-hole provides DNS for your LAN. If it goes down (e.g., during `docker compose down`), you'll lose DNS resolution and internet access. To recover:
> 1. Connect to mobile hotspot (or manually set DNS to 8.8.8.8)
> 2. SSH to NAS and run: `docker compose -f docker-compose.arr-stack.yml up -d pihole`
> 3. Switch back to your normal network
>
> **Tip:** When doing full stack restarts, use mobile hotspot first, or restart with a single command:
> ```bash
> docker compose -f docker-compose.arr-stack.yml up -d  # Recreates without full down
> ```

## Local Access (.lan domains)

Port-free access from any device on your LAN:

| URL | Service |
|-----|---------|
| `http://jellyfin.lan` | Jellyfin |
| `http://jellyseerr.lan` | Jellyseerr |
| `http://sonarr.lan` | Sonarr |
| `http://radarr.lan` | Radarr |
| `http://prowlarr.lan` | Prowlarr |
| `http://bazarr.lan` | Bazarr |
| `http://qbit.lan` | qBittorrent |
| `http://sabnzbd.lan` | SABnzbd |
| `http://traefik.lan` | Traefik Dashboard |
| `http://pihole.lan/admin` | Pi-hole |
| `http://wg.lan` | WireGuard |
| `http://uptime.lan` | Uptime Kuma |
| `http://duc.lan` | duc (disk usage) |

> **Setup required:** See [Local DNS section](SETUP.md#local-dns-lan-domains--optional) in Setup guide.

## External Access (via Cloudflare Tunnel)

| URL | Service | Auth |
|-----|---------|------|
| `https://jellyfin.${DOMAIN}` | Jellyfin | ✅ Built-in |
| `https://jellyseerr.${DOMAIN}` | Jellyseerr | ✅ Built-in |
| `https://wg.${DOMAIN}` | WireGuard | ✅ Password |

All other services are **LAN-only** (not exposed to internet).

## Services & Network

| Service | IP | Port | Notes |
|---------|-----|------|-------|
| Traefik | 172.20.0.2 | 80, 443 | Reverse proxy |
| **Gluetun** | **172.20.0.3** | — | VPN gateway |
| ↳ qBittorrent | (via Gluetun) | 8085 | Download client |
| ↳ Sonarr | (via Gluetun) | 8989 | TV shows |
| ↳ Radarr | (via Gluetun) | 7878 | Movies |
| ↳ Prowlarr | (via Gluetun) | 9696 | Indexer manager |
| Jellyfin | 172.20.0.4 | 8096 | Media server |
| Pi-hole | 172.20.0.5 | 8081 | DNS ad-blocking (`/admin`) |
| WireGuard | 172.20.0.6 | 51820/udp | Remote VPN access |
| Jellyseerr | 172.20.0.8 | 5055 | Request management |
| Bazarr | 172.20.0.9 | 6767 | Subtitles |
| FlareSolverr | 172.20.0.10 | 8191 | Cloudflare bypass |
| ↳ SABnzbd | (via Gluetun) | 8082 | Usenet downloads (VPN) |

**+ remote access** (cloudflared.yml):

| Service | IP | Port | Notes |
|---------|-----|------|-------|
| Cloudflared | 172.20.0.12 | — | Tunnel (no ports exposed) |

**Optional** (utilities.yml):

| Service | IP | Port | Notes |
|---------|-----|------|-------|
| Uptime Kuma | 172.20.0.13 | 3001 | Monitoring |
| duc | 172.20.0.14 | 8838 | Disk usage |

### Service Connection Guide

**VPN-protected services** (qBittorrent, Sonarr, Radarr, Prowlarr) share Gluetun's network via `network_mode: service:gluetun`. This means:

| From | To | Use | Why |
|------|-----|-----|-----|
| Sonarr | qBittorrent | `localhost:8085` | Same network stack |
| Radarr | qBittorrent | `localhost:8085` | Same network stack |
| Prowlarr | Sonarr | `localhost:8989` | Same network stack |
| Prowlarr | Radarr | `localhost:7878` | Same network stack |
| Prowlarr | FlareSolverr | `http://172.20.0.10:8191` | Direct IP (outside gluetun) |
| Jellyseerr | Sonarr | `gluetun:8989` | Must go through gluetun |
| Jellyseerr | Radarr | `gluetun:7878` | Must go through gluetun |
| Jellyseerr | Jellyfin | `jellyfin:8096` | Both have own IPs |
| Bazarr | Sonarr | `gluetun:8989` | Must go through gluetun |
| Bazarr | Radarr | `gluetun:7878` | Must go through gluetun |
| Sonarr | SABnzbd | `localhost:8080` | Same network stack |
| Radarr | SABnzbd | `localhost:8080` | Same network stack |

> **Why `gluetun` not `sonarr`?** Services sharing gluetun's network don't get their own Docker DNS entries. Jellyseerr/Bazarr must use `gluetun` hostname (or `172.20.0.3`) to reach them.

## Common Commands

```bash
# All commands below run on your NAS via SSH

# View all containers
docker ps

# View logs
docker logs -f <container_name>

# Restart single service
docker compose -f docker-compose.arr-stack.yml restart <service_name>

# Restart entire stack (safe - Pi-hole restarts immediately)
docker compose -f docker-compose.arr-stack.yml up -d --force-recreate

# Pull repo updates then redeploy
git pull origin main
docker compose -f docker-compose.arr-stack.yml up -d --force-recreate

# Update container images
docker compose -f docker-compose.arr-stack.yml pull
docker compose -f docker-compose.arr-stack.yml up -d
```

> ⚠️ **Never use `docker compose down`** - this stops Pi-hole which kills DNS for your entire network if your router uses Pi-hole. Use `up -d --force-recreate` instead to restart the stack safely.

## Networks

| Network | Subnet | Purpose |
|---------|--------|---------|
| arr-stack | 172.20.0.0/24 | Service communication |
| vpn-net | 10.8.1.0/24 | Internal VPN routing (WireGuard peers) |
| traefik-lan | (your LAN)/24 | macvlan - gives Traefik its own LAN IP for .lan domains |

## Startup Order

Services start in dependency order (handled automatically by `depends_on`):

1. **Pi-hole** → DNS ready
2. **Gluetun** → VPN connected (uses Pi-hole for DNS)
3. **Sonarr, Radarr, Prowlarr, qBittorrent, SABnzbd** → VPN-protected services
4. **Jellyseerr, Bazarr** → Connect to Sonarr/Radarr via Gluetun
5. **Jellyfin, WireGuard, FlareSolverr** → Independent, start anytime
