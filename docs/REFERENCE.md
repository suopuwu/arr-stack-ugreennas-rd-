# Quick Reference

## Local Access URLs

| Service | URL | Default Credentials |
|---------|-----|---------------------|
| Jellyfin | http://HOST_IP:8096 | (create during setup) |
| Jellyseerr | http://HOST_IP:5055 | (use Jellyfin login) |
| qBittorrent | http://HOST_IP:8085 | admin / adminadmin |
| Sonarr | http://HOST_IP:8989 | (none by default) |
| Radarr | http://HOST_IP:7878 | (none by default) |
| Prowlarr | http://HOST_IP:9696 | (none by default) |
| Bazarr | http://HOST_IP:6767 | (none by default) |
| Pi-hole | http://HOST_IP/admin | (from PIHOLE_UI_PASS) |

**Optional utilities** (if deployed with `docker-compose.utilities.yml`):

| Service | URL | Default Credentials |
|---------|-----|---------------------|
| Uptime Kuma | http://HOST_IP:3001 | (create during setup) |
| duc | http://HOST_IP:8838 | (none) |

## Common Commands

```bash
# View all containers
docker ps

# View logs
docker logs -f <container_name>

# Restart service
docker compose -f docker-compose.arr-stack.yml restart <service_name>

# Pull repo updates then redeploy
git pull origin main
docker compose -f docker-compose.arr-stack.yml down
docker compose -f docker-compose.arr-stack.yml up -d

# Update container images
docker compose -f docker-compose.arr-stack.yml pull
docker compose -f docker-compose.arr-stack.yml up -d

# Stop everything
docker compose -f docker-compose.arr-stack.yml down
```

## Network Information

| Network | Subnet | Purpose |
|---------|--------|---------|
| traefik-proxy | 192.168.100.0/24 | Service communication |
| vpn-net | 10.8.1.0/24 | Internal VPN routing |

## IP Allocation (traefik-proxy)

| IP | Service |
|----|---------|
| .1 | Gateway |
| .2 | Traefik |
| .3 | Gluetun |
| .4 | Jellyfin |
| .5 | Pi-hole |
| .6 | WireGuard |
| .8 | Jellyseerr |
| .9 | Bazarr |
| .10 | FlareSolverr |
| .12 | Cloudflared |
| .13 | Uptime Kuma* |

*Optional (utilities.yml)
