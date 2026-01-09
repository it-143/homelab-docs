# Network Infrastructure

## Overview

The home lab network uses a mesh router with the server on a static IP. External access is enabled through dynamic DNS and careful port management.

## Network Diagram

```
Internet
    │
    ▼
┌─────────────────────┐
│   ISP Router/Modem  │
└─────────────────────┘
    │
    ▼
┌─────────────────────┐      ┌─────────────────────┐
│   Mesh Router       │──────│   Raspberry Pi      │
│   (Gateway)         │      │   Pi-hole DNS       │
│   192.168.x.1       │      │   192.168.x.2       │
└─────────────────────┘      └─────────────────────┘
    │
    │ Ethernet
    ▼
┌─────────────────────┐
│  Server             │
│  192.168.x.x        │
│                     │
│  Services:          │
│  - Jellyfin         │
│  - Immich           │
│  - Audiobookshelf   │
│  - Kavita           │
│  - Glances          │
│  - Caddy            │
│  - WireGuard        │
└─────────────────────┘
```

## IP Assignments

| Device | IP Address | Purpose |
|--------|------------|---------|
| Router | 192.168.x.1 | Gateway |
| Raspberry Pi | 192.168.x.2 | DNS Server (Pi-hole) |
| Server | 192.168.x.x | Main Server |

## Dynamic DNS

Dynamic DNS maps a domain name to your changing home IP address.

### Popular Free Options
- FreeMyIP
- DuckDNS
- No-IP (free tier)

### Cron Job (Updates Every 5 Minutes)

```bash
crontab -e
```

Add (example for generic DDNS):
```
*/5 * * * * curl -s "https://your-ddns-provider.com/update?token=<your-token>&domain=<your-domain>" > /dev/null 2>&1
```

### Manual Update
```bash
curl -s "https://your-ddns-provider.com/update?..."
```

### Check Current Public IP
```bash
curl -s https://api.ipify.org
```

## Port Forwarding

### Active Forwards

| Name | External Port | Internal Port | Internal IP | Protocol |
|------|---------------|---------------|-------------|----------|
| HTTP | 80 | 80 | (server) | TCP |
| HTTPS | 443 | 443 | (server) | TCP |
| SSH | 2222 | 22 | (server) | TCP |
| WireGuard | 51820 | 51820 | (server) | **UDP** |

### Closed After Caddy Setup

Individual service ports no longer needed:
- ~~8096 (Jellyfin)~~
- ~~2283 (Immich)~~
- ~~13378 (Audiobookshelf)~~
- ~~5000 (Kavita)~~

## ISP Port Blocking

### The Problem

Some ISPs block common ports to prevent home servers.

### How to Identify

1. Configure port forward correctly in router
2. Test with external checker (yougetsignal.com)
3. If closed despite correct config, ISP is blocking

### Solution

Use alternate ports. After Caddy setup, only need 80/443 which ISPs typically don't block.

## DNS Architecture

### External Resolution

```
jellyfin.yourserver.example.com 
    → Public IP 
    → Router NAT 
    → Caddy 
    → Jellyfin
```

### Internal Resolution (Pi-hole)

For devices using Pi-hole as DNS:

```
jellyfin.home 
    → Pi-hole Local DNS 
    → Server IP 
    → Caddy 
    → Jellyfin
```

### Pi-hole Local DNS Records

| Domain | IP |
|--------|-----|
| jellyfin.home | (server IP) |
| immich.home | (server IP) |
| audiobookshelf.home | (server IP) |
| kavita.home | (server IP) |
| pihole.home | (Pi-hole IP) |

## VPN Considerations

### Friend-to-Friend Sharing

For sharing services with friends who have similar setups:

1. **WireGuard Site-to-Site:** Create tunnel between networks
2. **Both have dynamic IPs:** Both need DDNS and `PersistentKeepalive`
3. **Tailscale/Headscale:** Zero port forwarding, easier setup

### Maximum Security

Once comfortable with WireGuard, close SSH port entirely. Only access SSH via VPN.

## Troubleshooting

### Can't Access Server Externally

1. Check public IP: `curl -s https://api.ipify.org`
2. Verify DNS resolves correctly
3. Update DDNS manually
4. Test port with external checker
5. Check router port forwards

### DNS Not Resolving .home Domains

1. Verify device using Pi-hole DNS
2. Check Pi-hole Local DNS records
3. Test: `nslookup jellyfin.home <pihole-ip>`

### Services Accessible Internally But Not Externally

1. Caddy running? `docker ps | grep caddy`
2. Ports 80/443 forwarded?
3. ISP blocking ports?
4. Check Caddy logs
