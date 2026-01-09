# Jellyfin Media Server

Jellyfin is a free, open-source media server for streaming movies, TV shows, and music.

## Installation

### Prerequisites
```bash
mkdir -p ~/jellyfin/config ~/jellyfin/cache ~/media/movies ~/media/tvshows ~/media/music ~/media/musicvideos
```

### Docker Run Command
```bash
docker run -d \
  --name jellyfin \
  --restart unless-stopped \
  -p 8096:8096 \
  -v ~/jellyfin/config:/config \
  -v ~/jellyfin/cache:/cache \
  -v ~/media:/media \
  jellyfin/jellyfin
```

> **Note:** If your ISP blocks port 8096, use an alternate like 8920: `-p 8920:8096`

## Configuration

### Directory Structure
```
~/media/
├── movies/
│   └── Movie Name (Year)/
│       └── Movie Name (Year).mkv
├── tvshows/
│   └── Show Name/
│       └── Season 01/
│           └── Show Name - S01E01 - Episode Title.mkv
├── music/
│   └── Artist Name/
│       └── Album Name/
│           └── 01 - Track Title.mp3
└── musicvideos/
    └── Artist Name/
        └── Song Title.mp4
```

### Recommended Media Organization

**Movies:** Use `Movie Name (Year)/Movie Name (Year).ext` format for best metadata scraping.

**TV Shows:** `Show Name/Season XX/Show Name - SXXEXX - Episode Title.ext`

**Music:** `Artist/Album/## - Track.ext` - Jellyfin reads ID3 tags, but folder structure helps with organization.

**Music Videos:** Create a separate library with "Music Videos" content type for proper metadata scraping.

### Network Settings

In Jellyfin Dashboard → Networking:
- Allow remote connections: Enabled
- Local network addresses: Leave default
- Remote IP address filter: Configure if needed for security

## Access

| Access Type | URL |
|-------------|-----|
| Local | http://192.168.x.x:8096 |
| External (via Caddy) | https://jellyfin.yourserver.example.com |
| Internal (via Pi-hole) | https://jellyfin.home |

## Troubleshooting

### ISP Port Blocking

**Problem:** Port 8096 showed as closed from external port checkers even with port forwarding configured correctly.

**Solution:** ISPs often block common media server ports. Switch to an alternate port (e.g., 8920 external → 8096 internal).

**How to identify:** Use an external port checker like yougetsignal.com. If the port shows closed despite correct router config, try an alternate port.

### SSL Protocol Error for Remote Users

**Problem:** Remote user couldn't access server - browser showed `ERR_SSL_PROTOCOL_ERROR`.

**Cause:** Modern browsers auto-upgrade HTTP to HTTPS. When server only speaks HTTP, protocol mismatch occurs.

**Solutions:**
1. Have user explicitly type `http://` (workaround)
2. **Best:** Set up Caddy reverse proxy with real HTTPS certificates

### Container Not Starting After Reboot

**Problem:** After power outage/reboot, Jellyfin didn't auto-start.

**Solution:** Verify restart policy:
```bash
docker inspect --format '{{.Name}}: {{.HostConfig.RestartPolicy.Name}}' jellyfin
```

Should show `unless-stopped`. If not:
```bash
docker update --restart unless-stopped jellyfin
```

## File Format Recommendations

**Best for storage and quality:** MKV (Matroska)
- Container format supporting any video/audio codec
- Multiple audio tracks and subtitle options
- Chapter markers
- No quality loss

**Considerations:**
- Some devices (older smart TVs, Apple devices) don't play MKV natively
- Jellyfin will transcode on-the-fly, but this taxes the CPU
- For direct play on all devices: MP4 with H.264 video and AAC audio

## Mobile Apps

- **iOS:** Swiftfin or official Jellyfin app
- **Android:** Official Jellyfin app
- **TV:** Apps available for most smart TV platforms

When updating server address in apps, use the full HTTPS URL.

## Useful Commands

```bash
# Check status
docker ps | grep jellyfin

# View logs
docker logs jellyfin

# Restart
docker restart jellyfin

# Access shell inside container
docker exec -it jellyfin bash
```
