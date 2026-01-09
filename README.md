# Home Lab Documentation

A self-hosted infrastructure project running on a 2011 Mac mini converted to Ubuntu Server. This documentation covers the complete setup process, troubleshooting, and lessons learned.

## Philosophy

This project is centered around privacy, control, and avoiding subscription services. The goal is to create a robust home server ecosystem that provides enterprise-level functionality while maintaining complete ownership of data and services.

## Hardware

| Component | Specification |
|-----------|---------------|
| Device | Mac mini Mid 2011 (Macmini5,2) |
| CPU | Intel Core i7-2620M @ 2.7 GHz (dual-core, 4 threads) |
| RAM | 8 GB |
| Storage | 700 GB internal HDD |
| Architecture | Sandy Bridge with Intel HD Graphics 3000 |

**Secondary Device:**
- Raspberry Pi 2 running Pi-hole for DNS and ad-blocking

## Network Configuration

| Setting | Value |
|---------|-------|
| Server IP | 192.168.x.x (static) |
| Gateway | 192.168.x.1 |
| Hostname | yourserver |
| Connection | Ethernet to mesh router |

> **Note:** Replace `x.x` with your actual network values when implementing.

## Services Overview

| Service | Purpose | Local Port | External Access |
|---------|---------|------------|-----------------|
| [Jellyfin](services/jellyfin.md) | Media server (movies, TV, music) | 8096* | Via reverse proxy |
| [Immich](services/immich.md) | Photo management | 2283 | Via reverse proxy |
| [Audiobookshelf](services/audiobookshelf.md) | Audiobooks & podcasts | 13378 | Via reverse proxy |
| [Kavita](services/kavita.md) | Ebooks, manga, PDFs | 5000 | Via reverse proxy |
| [Glances](services/glances.md) | System monitoring | 61208 | Internal only |
| [Pi-hole](services/pihole.md) | DNS & ad-blocking | 80 (on Pi) | Internal only |
| [Caddy](services/caddy.md) | Reverse proxy & HTTPS | 80, 443 | — |
| [WireGuard](services/wireguard.md) | VPN server | 51820/udp | — |

*May need alternate port if ISP blocks default

## Infrastructure Components

- **[Network Setup](infrastructure/network.md)** - Dynamic DNS, port forwarding, firewall
- **[Docker Configuration](infrastructure/docker.md)** - Container management, restart policies
- **[Samba File Sharing](infrastructure/samba.md)** - Network shares for Mac access
- **[SSL/HTTPS](infrastructure/ssl.md)** - Certificate management with Caddy

## Quick Reference

### SSH Access
```bash
# Local
ssh youruser@192.168.x.x

# Remote (via port forward)
ssh -p 2222 youruser@yourserver.example.com
```

### Service URLs (Internal - requires Pi-hole DNS)
- https://jellyfin.home
- https://immich.home
- https://audiobookshelf.home
- https://kavita.home
- https://pihole.home

### Useful Commands
```bash
# Check all containers
docker ps

# View container logs
docker logs <container_name>

# Restart a service
docker restart <container_name>

# System update
sudo apt update && sudo apt upgrade -y

# Check temperatures
sensors

# Check public IP
curl -s https://api.ipify.org
```

## Project Timeline

- **Week 1** - Initial Ubuntu Server installation, Jellyfin setup, discovered ISP port blocking
- **Week 1** - Added Audiobookshelf, Immich, Samba shares
- **Week 2** - Added Kavita for ebooks
- **Week 2** - Set up Pi-hole on Raspberry Pi, configured Caddy reverse proxy
- **Week 2** - Added WireGuard VPN, Glances monitoring
- **Week 2** - Discovered thermal issues, implemented CPU limits

## Lessons Learned

See [Troubleshooting & Lessons](troubleshooting/lessons-learned.md) for detailed documentation of problems encountered and their solutions.

Key takeaways:
- ISPs often block common ports - test with alternates
- Old hardware needs thermal management under heavy loads
- Document everything as you go
- HTTPS is essential for external access in 2025+

## Future Plans

- [ ] Add external hard drives for expanded storage
- [ ] Implement 3-2-1 backup strategy with rsync
- [ ] Deploy RustDesk for family tech support
- [ ] Replace thermal paste on Mac mini
- [ ] Explore friend-to-friend service sharing via WireGuard peering

## Adapting This Guide

This documentation uses placeholder values you'll need to replace:

| Placeholder | Replace With |
|-------------|--------------|
| `192.168.x.x` | Your server's static IP |
| `192.168.x.1` | Your router/gateway IP |
| `yourserver` | Your chosen hostname |
| `youruser` | Your Linux username |
| `yourserver.example.com` | Your dynamic DNS domain |
| `<your-token>` | Your actual API tokens |

## License

This documentation is provided as-is for educational purposes. Feel free to adapt for your own home lab.
