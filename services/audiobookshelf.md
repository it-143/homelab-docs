# Audiobookshelf

Audiobookshelf is a self-hosted audiobook and podcast server with excellent mobile apps.

## Installation

### Prerequisites
```bash
mkdir -p ~/audiobooks ~/podcasts ~/audiobookshelf/config ~/audiobookshelf/metadata
```

### Docker Run Command
```bash
docker run -d \
  --name=audiobookshelf \
  --restart unless-stopped \
  -p 13378:80 \
  -v ~/audiobooks:/audiobooks \
  -v ~/podcasts:/podcasts \
  -v ~/audiobookshelf/config:/config \
  -v ~/audiobookshelf/metadata:/metadata \
  ghcr.io/advplyr/audiobookshelf
```

## Configuration

### Directory Structure

**Audiobooks:**
```
~/audiobooks/
├── Author Name/
│   └── Book Title/
│       ├── Chapter 01.mp3
│       ├── Chapter 02.mp3
│       └── cover.jpg
└── Another Author/
    └── Series Name/
        └── Book 1/
            └── ...
```

**Podcasts:**
```
~/podcasts/
└── Podcast Name/
    ├── Episode 1.mp3
    └── Episode 2.mp3
```

### Initial Setup

1. Access http://192.168.x.x:13378
2. Create admin account
3. Add library:
   - Click gear icon → Libraries
   - Add Library
   - Type: Audiobooks (or Podcast)
   - Folder: `/audiobooks` (container path)

### Metadata Configuration

Audiobookshelf can fetch metadata automatically:
- Settings → Libraries → your library → Edit
- Enable "Auto-scan for new content"
- Enable metadata providers (Audible, Google Books, etc.)

## Access

| Access Type | URL |
|-------------|-----|
| Local | http://192.168.x.x:13378 |
| External (via Caddy) | https://audiobookshelf.yourserver.example.com |
| Internal (via Pi-hole) | https://audiobookshelf.home |

## Mobile App

Audiobookshelf has dedicated iOS and Android apps with:
- Progress sync across devices
- Offline downloads
- Sleep timer
- Playback speed control
- Chapter navigation
- **Apple Watch support** (play/pause, skip, volume, sleep timer)

**Setup:**
1. Download "Audiobookshelf" from App Store/Play Store
2. Enter server URL
3. Log in

## Adding Content via Samba

Add audiobooks share to Samba for easy drag-and-drop from Mac.

### Samba Configuration
Add to `/etc/samba/smb.conf`:
```ini
[audiobooks]
   path = /home/youruser/audiobooks
   browseable = yes
   read only = no
   valid users = youruser

[podcasts]
   path = /home/youruser/podcasts
   browseable = yes
   read only = no
   valid users = youruser
```

### Connect from Mac
1. Finder → Go → Connect to Server (Cmd+K)
2. `smb://192.168.x.x`
3. Select `audiobooks` or `podcasts` share
4. Drag and drop content

### After Adding Files
Audiobookshelf auto-scans periodically, or manually:
- Settings → Libraries → Click scan icon

## Troubleshooting

### Config Directory Confusion

**Problem:** During setup, saw `/config` and `/metadata` paths.

**Explanation:** These are container-internal paths that map to your host directories:
- `/config` → `~/audiobookshelf/config`
- `/metadata` → `~/audiobookshelf/metadata`
- `/audiobooks` → `~/audiobooks`
- `/podcasts` → `~/podcasts`

You only configure library paths using the container paths.

### Container Not Auto-Starting

**Solution:** Verify restart policy:
```bash
docker inspect --format '{{.Name}}: {{.HostConfig.RestartPolicy.Name}}' audiobookshelf
```

Should show `unless-stopped`.

## Useful Commands

```bash
# Check status
docker ps | grep audiobookshelf

# View logs
docker logs audiobookshelf

# Restart
docker restart audiobookshelf
```

## Supported Formats

**Audio:** mp3, m4b, m4a, flac, opus, ogg, mp4, aac, wma, aiff, wav
**Ebooks:** epub, pdf, mobi, azw3, cbr, cbz

Audiobookshelf can also handle ebooks, making it a versatile media server for books in all formats.
