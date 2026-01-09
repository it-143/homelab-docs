# Caddy Reverse Proxy

Caddy is a modern web server that automatically handles HTTPS certificates via Let's Encrypt. It acts as a reverse proxy, routing external requests to internal services.

## Why Caddy?

Before Caddy:
- Each service exposed on different ports
- No HTTPS - login credentials sent in plain text
- Browsers increasingly block HTTP connections
- Each port needed separate forwarding

After Caddy:
- Single entry point (ports 80/443)
- Automatic HTTPS with real certificates
- Clean subdomain URLs
- Only two ports to forward

## Installation

### Create Directories
```bash
mkdir -p ~/caddy/config ~/caddy/data
```

### Create Caddyfile
```bash
nano ~/caddy/Caddyfile
```

Example configuration handling both local and external access:

```
# Local access (requires Pi-hole DNS)
jellyfin.home {
    reverse_proxy 192.168.x.x:8096
    tls internal
}

immich.home {
    reverse_proxy 192.168.x.x:2283
    tls internal
}

audiobookshelf.home {
    reverse_proxy 192.168.x.x:13378
    tls internal
}

kavita.home {
    reverse_proxy 192.168.x.x:5000
    tls internal
}

pihole.home {
    reverse_proxy 192.168.x.2:80
    tls internal
}

# External access (real Let's Encrypt certificates)
jellyfin.yourserver.example.com {
    reverse_proxy 192.168.x.x:8096
}

immich.yourserver.example.com {
    reverse_proxy 192.168.x.x:2283
}

audiobookshelf.yourserver.example.com {
    reverse_proxy 192.168.x.x:13378
}

kavita.yourserver.example.com {
    reverse_proxy 192.168.x.x:5000
}
```

### Run Caddy Container
```bash
docker run -d \
  --name caddy \
  --restart unless-stopped \
  -p 80:80 \
  -p 443:443 \
  -v ~/caddy/Caddyfile:/etc/caddy/Caddyfile \
  -v ~/caddy/data:/data \
  -v ~/caddy/config:/config \
  caddy:latest
```

### Reload After Config Changes
```bash
docker exec caddy caddy reload --config /etc/caddy/Caddyfile
```

## DNS Configuration

### For Local Access (.home domains)

Pi-hole Local DNS must resolve these to the server IP.

### For External Access (subdomains)

Your dynamic DNS provider needs to resolve subdomains to your public IP. Many providers support wildcard DNS where all subdomains resolve automatically.

## Port Forwarding

### Router Configuration

Forward only these ports to your server:
| External | Internal | Protocol |
|----------|----------|----------|
| 80 | 80 | TCP |
| 443 | 443 | TCP |

### Close Old Ports

After Caddy is working, close individual service ports (8096, 2283, 13378, 5000, etc.) to reduce attack surface.

## Troubleshooting

### Port 443 Check

Before setup, verify ISP isn't blocking port 443:
```bash
# External checker
# https://www.yougetsignal.com/tools/open-ports/

# Or from terminal
curl -v --max-time 5 http://yourserver.example.com
```

If ISP blocks 443, use alternate port like 8443.

### Certificate Warnings (Local .home domains)

**Expected behavior:** Browsers show certificate warning for `.home` domains because they use self-signed certs (`tls internal`).

Click through once per service. Traffic is still encrypted.

### Certificate Issues (External domains)

If Let's Encrypt fails:

1. Check ports 80/443 are forwarded correctly
2. Verify DNS resolves to your IP
3. Check Caddy logs: `docker logs caddy`
4. Ensure dynamic DNS is current

### Config Format Warning

```
WARN    Caddyfile input is not formatted
```

Just a style warning—config still loads. Auto-format with:
```bash
docker exec caddy caddy fmt --overwrite /etc/caddy/Caddyfile
```

## Useful Commands

```bash
# Check status
docker ps | grep caddy

# View logs
docker logs caddy
docker logs -f caddy  # Follow

# Reload config
docker exec caddy caddy reload --config /etc/caddy/Caddyfile

# Validate config
docker exec caddy caddy validate --config /etc/caddy/Caddyfile

# Restart
docker restart caddy
```

## Security Benefits

1. **HTTPS everywhere** - All traffic encrypted
2. **Single entry point** - Only ports 80/443 exposed
3. **Automatic cert renewal** - Let's Encrypt handles it
4. **Defense in depth** - Services not directly exposed
