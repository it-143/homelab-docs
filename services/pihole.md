# Pi-hole

Pi-hole is a network-wide ad blocker that acts as a DNS sinkhole. Runs on a dedicated Raspberry Pi.

## Hardware

- **Device:** Raspberry Pi 2/3/4
- **OS:** Raspberry Pi OS
- **IP:** 192.168.x.2 (static recommended)
- **Connection:** Ethernet to router

## Installation

### Set Static IP First

Newer Raspberry Pi OS uses NetworkManager instead of dhcpcd. Check which is active:
```bash
sudo systemctl status dhcpcd
# If "could not be found", you're on NetworkManager
```

**For NetworkManager:**
```bash
# Find connection name
nmcli con show

# Set static IP (adjust connection name and IPs as needed)
sudo nmcli con mod "netplan-eth0" \
  ipv4.addresses 192.168.x.2/24 \
  ipv4.gateway 192.168.x.1 \
  ipv4.dns "1.1.1.1" \
  ipv4.method manual

sudo reboot
```

### Install Pi-hole
```bash
curl -sSL https://install.pi-hole.net | sudo bash
```

**During setup:**
- Interface: `eth0`
- Upstream DNS: Cloudflare (1.1.1.1) or your preference
- Blocklists: Yes, use default
- Admin web interface: Yes
- Web server: Yes
- Logging: Yes
- Privacy mode: 0 (show everything) for home use

**Save the admin password** shown at the end!

## Access

| Interface | URL |
|-----------|-----|
| Admin Dashboard | http://192.168.x.2/admin |
| Via Caddy (HTTPS) | https://pihole.home |

## Client Configuration

Pi-hole only filters DNS for devices configured to use it. Two approaches:

### Option 1: Per-Device

**macOS:**
1. System Settings → Network → Wi-Fi → Details
2. DNS tab
3. Remove existing, add Pi-hole IP

**iOS:**
1. Settings → Wi-Fi → tap ⓘ on network
2. Configure DNS → Manual
3. Delete existing, Add Server: Pi-hole IP

### Option 2: Router-Wide

Configure router's DHCP to hand out Pi-hole's IP as DNS server.

**Pros:** Set and forget, covers all devices
**Cons:** Pi-hole becomes single point of failure

## Network Location Switching (macOS)

Create two network locations to easily toggle Pi-hole on/off.

### Setup
1. System Settings → Network → Location dropdown → Edit Locations
2. Create "home-pihole" (DNS: Pi-hole IP)
3. Create "home-normal" (DNS: automatic/router)

### Shortcuts for Desktop Widget

**Pi-hole ON:**
```bash
networksetup -switchtolocation "home-pihole"
```

**Pi-hole OFF:**
```bash
networksetup -switchtolocation "home-normal"
```

In Shortcuts app:
1. New shortcut → Run Shell Script
2. Paste command
3. Check "Run as Administrator"
4. Add to desktop via Widgets

## Local DNS Records

Pi-hole can resolve custom domain names for your local network.

### Setup
1. Local DNS → DNS Records
2. Add entries:

| Domain | IP |
|--------|-----|
| jellyfin.home | (your server IP) |
| immich.home | (your server IP) |
| audiobookshelf.home | (your server IP) |
| kavita.home | (your server IP) |
| pihole.home | (Pi-hole IP) |

These resolve to your server's IP, then Caddy handles routing based on hostname.

## Troubleshooting

### Static IP Didn't Take (Raspberry Pi)

**Problem:** After setting static IP and rebooting, Pi is unreachable.

**Cause:** Edited wrong config file. Newer Pi OS uses NetworkManager, not `/etc/dhcpcd.conf`.

**Solution:** Use `nmcli` commands. Check which service is active first.

### Blocklist Error (-2)

**Problem:** Dashboard shows "Domains on Lists - Error (-2)"

**Solution:**
```bash
pihole -g   # Refresh blocklists (update gravity)
```

### Site Broken Due to Blocking

**Quick fix:** Disable blocking temporarily from dashboard.

**Permanent fix:** Whitelist the domain in Domains settings.

## Useful Commands

```bash
# SSH to Pi
ssh youruser@192.168.x.2

# Check status
pihole status

# Update blocklists
pihole -g

# View recent queries
pihole -t

# Restart DNS
pihole restartdns

# Update Pi-hole
pihole -up
```

## Maintenance

- **Blocklists update automatically** (weekly by default)
- **Check dashboard periodically** for query patterns
- **Keep Pi-hole updated:** `pihole -up`

## Backup

Key files:
- `/etc/pihole/` - Configuration
- `/etc/dnsmasq.d/` - DNS settings

Or use built-in teleporter: Settings → Teleporter → Export
