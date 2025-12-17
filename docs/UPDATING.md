# Updating the Stack

Since the stack is deployed via git, updates are straightforward.

## Pull Latest Changes

```bash
cd /volume1/docker/arr-stack  # or your deployment path
git pull origin main
```

## Redeploy Services

After pulling changes, redeploy to apply updates:

```bash
# Stop services (Docker volumes are preserved - your configs are safe)
docker compose -f docker-compose.arr-stack.yml down
docker compose -f docker-compose.cloudflared.yml down
docker compose -f docker-compose.traefik.yml down

# Start in correct order
docker compose -f docker-compose.traefik.yml up -d
docker compose -f docker-compose.cloudflared.yml up -d
docker compose -f docker-compose.arr-stack.yml up -d
```

## Update Container Images

To pull the latest Docker images (Sonarr, Radarr, etc.):

```bash
docker compose -f docker-compose.arr-stack.yml pull
docker compose -f docker-compose.arr-stack.yml up -d
```

**Note:** Docker named volumes persist across `down`/`up` cycles. All your service configurations (Sonarr settings, API keys, library data, etc.) are preserved.
