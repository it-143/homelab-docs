# Docker Configuration

All services run as Docker containers, providing isolation, easy updates, and consistent deployments.

## Installation

Docker installed via the official convenience script:

```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
# Log out and back in for group membership to take effect
```

## Container Overview

```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

## Restart Policies

All containers should have restart policies to survive reboots.

### Check Current Policies
```bash
docker inspect --format '{{.Name}}: {{.HostConfig.RestartPolicy.Name}}' $(docker ps -q)
```

### Fix Missing Policy
```bash
docker update --restart unless-stopped <container_name>
```

### Policy Types
- `no`: Never restart (default if not specified)
- `always`: Always restart, including on Docker daemon start
- `unless-stopped`: Restart unless explicitly stopped
- `on-failure`: Only restart on non-zero exit code

**Recommendation:** Use `unless-stopped` for most services.

## Volume Mounts

### Directory Structure Example

```
/home/youruser/
├── jellyfin/
│   ├── config/
│   └── cache/
├── immich/
│   ├── library/
│   └── docker-compose.yml
├── audiobookshelf/
│   ├── config/
│   └── metadata/
├── kavita/
│   └── config/
├── caddy/
│   ├── Caddyfile
│   ├── data/
│   └── config/
├── wireguard/
├── media/
│   ├── movies/
│   ├── tvshows/
│   ├── music/
│   └── books/
├── audiobooks/
└── podcasts/
```

### Container vs Host Paths

When configuring services:
- **Host path:** `/home/youruser/media` (where files live on server)
- **Container path:** `/media` (what the application sees)

The `docker run -v` flag maps between them: `-v ~/media:/media`

## Resource Limits

### CPU Limit Example (Immich ML)

Critical for thermal management on limited hardware:

```yaml
immich-machine-learning:
  container_name: immich_machine_learning
  deploy:
    resources:
      limits:
        cpus: '1.0'
```

### Apply After Editing Compose
```bash
docker compose up -d
```

## Common Operations

### View All Containers
```bash
docker ps -a  # Including stopped
```

### Start/Stop/Restart
```bash
docker start <container>
docker stop <container>
docker restart <container>
```

### View Logs
```bash
docker logs <container>
docker logs -f <container>        # Follow
docker logs --tail 100 <container>  # Last 100 lines
```

### Enter Container Shell
```bash
docker exec -it <container> bash
# or for Alpine-based
docker exec -it <container> sh
```

### Update Container Image
```bash
docker pull <image>
docker stop <container>
docker rm <container>
# Recreate with original command
```

### Clean Up
```bash
docker system prune          # Remove unused data
docker image prune          # Remove dangling images
docker volume prune         # Remove unused volumes (careful!)
```

## Docker Compose vs Docker Run

### Docker Compose
- Defined in `docker-compose.yml`
- Multiple related containers
- Commands: `docker compose up -d`, `docker compose down`

### Docker Run
- Single containers
- Started with `docker run` command
- Must document the run command for recreation

## Troubleshooting

### Container Won't Start After Reboot

1. Check if container exists: `docker ps -a | grep <name>`
2. If exists but stopped: `docker start <name>`
3. View logs for errors: `docker logs <name>`

### "No Configuration File" Error

If service was created with `docker run`, there's no compose file. Use `docker start <name>` instead of `docker compose up`.

### Container Using Too Much Resources

```bash
# Check usage
docker stats

# Add limits
docker update --cpus 1.5 <container>
docker update --memory 2g <container>
```

### Disk Space Issues

```bash
# Check Docker disk usage
docker system df

# Clean up
docker system prune -a --volumes
```

## Backup Considerations

Critical directories:
- Service config directories
- Media files
- Database volumes
- WireGuard keys (critical!)
- Caddy data (certificates)

Lower priority:
- Cache directories
- Logs
