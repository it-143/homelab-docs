# WireGuard VPN

WireGuard is a modern, fast VPN protocol. We use wg-easy for a web-based management interface.

## What WireGuard Does

When away from home, WireGuard creates an encrypted tunnel back to your network. Once connected:
- Your device acts like it's physically at home
- Access local-only services (Pi-hole admin, monitoring)
- Use .home domains from anywhere
- All traffic routes through your home network (secure on public WiFi)
- Can close SSH port and only access via VPN

## Installation

### Create Directory
```bash
mkdir -p ~/wireguard
```

### Run wg-easy Container

```bash
docker run -d \
  --name wg-easy \
  --restart unless-stopped \
  -e WG_HOST=yourserver.example.com \
  -e PASSWORD=YourSecurePassword \
  -e WG_DEFAULT_DNS=192.168.x.2 \
  -v ~/wireguard:/etc/wireguard \
  -p 51820:51820/udp \
  -p 51821:51821/tcp \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  ghcr.io/wg-easy/wg-easy
```

**Important:**
- Change `PASSWORD` to something secure
- Set `WG_HOST` to your dynamic DNS domain
- `WG_DEFAULT_DNS` routes DNS through Pi-hole for ad-blocking on VPN
- Port 51820/udp is for VPN connections
- Port 51821/tcp is for web management UI

### Port Forwarding

Add to router:
| External | Internal | Protocol |
|----------|----------|----------|
| 51820 | 51820 | **UDP** |

Note: This is UDP, not TCP like other services.

## Web UI Access

- http://192.168.x.x:51821

Log in with the PASSWORD you set.

## Adding Clients

### Via Web UI
1. Open management interface
2. Click "+ New Client"
3. Enter a name (e.g., "iPhone", "MacBook")
4. Click Create

### Mobile Setup
1. Download WireGuard app (iOS/Android)
2. In web UI, click QR code icon next to client
3. In app: Add Tunnel → Scan QR Code
4. Toggle VPN on to connect

### Desktop Setup
1. Download WireGuard (macOS/Windows/Linux)
2. In web UI, click download icon next to client
3. Import the .conf file into WireGuard app
4. Activate tunnel

## Troubleshooting

### Docker Command Breaks on Paste

**Problem:** Command has special characters, shell interprets incorrectly.

**Solution:** Quote values with special characters:
```bash
-e 'PASSWORD=Complex!Pass123'
```

### IPv6 Resolution Issue

**Problem:** VPN connects but can't reach services.

**Cause:** Device resolving domain to IPv6.

**Solution:** Force IPv4 in client config—use actual IP instead of domain for Endpoint.

### Can't Connect When IP Changes

**Cause:** Dynamic DNS hasn't updated.

**Solutions:**
1. Wait for cron job to update (typically every 5 min)
2. Force DNS update manually
3. Reconnect VPN to re-resolve domain

## VPN Use Cases

### Secure Remote Access
Access all services securely from anywhere.

### Pi-hole on Mobile Data
With Pi-hole set as WG_DEFAULT_DNS, even cellular data gets ad-blocking.

### Access .home Domains Anywhere
While connected, local DNS domains work from anywhere.

### Maximum Security
Close external SSH port entirely. SSH only via VPN.

## Network Configuration

| Port | Purpose |
|------|---------|
| 51820/UDP | WireGuard VPN (external) |
| 51821/TCP | wg-easy Web UI (internal only) |

## Backup

Critical files in `~/wireguard/`:
- Private keys for server and clients
- Client configurations

**If you lose these, all clients need reconfiguration.**

## Useful Commands

```bash
# Check container status
docker ps | grep wg-easy

# View logs
docker logs wg-easy

# Restart
docker restart wg-easy

# Check active connections
docker exec wg-easy wg show
```

## Security Notes

- WireGuard uses modern cryptography
- Minimal attack surface
- Each client has unique key pair—can revoke individually
- No persistent state on connection drops
