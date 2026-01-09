# Kavita

Kavita is a self-hosted digital library for ebooks, manga, and comics with a built-in reader.

## Installation

### Using Docker Compose (Recommended)

Create `~/kavita/docker-compose.yml`:
```yaml
services:
  kavita:
    image: jvmilazz0/kavita:latest
    container_name: kavita
    volumes:
      - ~/kavita/config:/kavita/config
      - ~/media/books:/books
    ports:
      - "5000:5000"
    restart: unless-stopped
```

### Alternative: Docker Run
```bash
mkdir -p ~/books ~/kavita/config

docker run -d \
  --name=kavita \
  --restart unless-stopped \
  -p 5000:5000 \
  -v ~/books:/books \
  -v ~/kavita/config:/kavita/config \
  jvmilazz0/kavita
```

### Start
```bash
cd ~/kavita && docker compose up -d
```

## Configuration

### Directory Structure

```
~/media/books/
├── Manga/
│   └── Series Name/
│       ├── Series Name Vol 01.cbz
│       └── Series Name Vol 02.cbz
├── Comics/
│   └── Series Name/
│       ├── Issue 001.cbz
│       └── Issue 002.cbz
└── Books/
    └── Author Name/
        └── Book Title/
            └── Book Title.epub
```

**Key points:**
- Books need to be in subfolders, not loose in the root
- Series should be in their own folders
- Kavita uses folder names for organization

### Initial Setup

1. Access http://192.168.x.x:5000
2. Create admin account
3. Add library:
   - Server Settings (gear icon) → Libraries
   - Add Library
   - Name: "Books" (or "Manga", "Comics")
   - Type: Select appropriate type
   - Folder: `/books` (container path)
4. Kavita will scan automatically

## Access

| Access Type | URL |
|-------------|-----|
| Local | http://192.168.x.x:5000 |
| External (via Caddy) | https://kavita.yourserver.example.com |
| Internal (via Pi-hole) | https://kavita.home |

## Supported Formats

**Ebooks:** epub, pdf
**Comics/Manga:** cbz, cbr, cb7, zip, rar, tar.gz, 7zip
**Images:** Raw images in folders (for manga/comics)

## Adding Content

### Via Samba

Connect to your server's Samba share and navigate to books folder.

### After Adding Files

1. Go to Server Settings → Libraries
2. Click refresh/scan icon on your library
3. Or wait for scheduled scan

## Troubleshooting

### Book Not Appearing After Adding

**Causes and solutions:**

1. **macOS junk files:** Remove `._*` files created by Finder
   ```bash
   find ~/media/books -name '._*' -delete
   ```

2. **File in root directory:** Kavita doesn't always pick up loose files. Put books in subfolders:
   ```bash
   cd ~/media/books
   mkdir -p "Book Title"
   mv "Book Title.epub" "Book Title/"
   ```

3. **Filename too long:** Rename to something simple:
   ```bash
   mv 'Very Long Filename...' "Simple Title.epub"
   ```

4. **Check logs:**
   ```bash
   docker logs kavita
   ```

### Library Won't Add

**Problem:** Can't find folder when setting up library.

**Solution:** Use the container path (`/books`) not the host path (`~/media/books`).

## Useful Commands

```bash
# Check status
docker ps | grep kavita

# View logs
docker logs kavita

# Restart
docker restart kavita
```

## Tips

- **Reading progress syncs** across devices when logged into the same account
- **OPDS support** allows connecting reader apps like Panels or Chunky
- **Metadata can be edited** per series/book in the web UI
- **Multiple libraries** can separate manga, comics, and ebooks
