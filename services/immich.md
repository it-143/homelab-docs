# Immich Photo Server

Immich is a self-hosted Google Photos alternative with mobile app backup, facial recognition, and smart search.

## Installation

Immich requires Docker Compose due to multiple services (app server, machine learning, PostgreSQL, Redis).

### Download Official Compose File
```bash
mkdir ~/immich && cd ~/immich
wget https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
```

### Configure Environment
Edit `.env`:
```bash
UPLOAD_LOCATION=./library
DB_PASSWORD=<strong-password>
```

### Start Services
```bash
docker compose up -d
```

## Critical Configuration: CPU Limits

⚠️ **Important for older/limited hardware**

The Mac mini 2011 runs extremely hot under ML processing loads. Without CPU limits, Immich's machine learning container can push CPU temps to 90°C+.

### Add CPU Limit to docker-compose.yml

```yaml
immich-machine-learning:
  container_name: immich_machine_learning
  deploy:
    resources:
      limits:
        cpus: '1.0'  # Limit to 1 core instead of all available
```

After editing:
```bash
cd ~/immich
docker compose up -d
```

### Why This Matters

When uploading photos, Immich runs:
- Thumbnail generation
- Facial recognition
- Smart search embeddings (CLIP model)

On limited hardware, this can cause:
- CPU temps exceeding 90°C
- Fan running at maximum
- System throttling
- Potential hardware damage long-term

With the CPU limit, processing takes longer but temps stay in safe range.

## External Libraries

To import existing photos (not just mobile uploads), set up an external library.

### Add Volume Mount

In `docker-compose.yml`, add to `immich-server` volumes:
```yaml
volumes:
  - ${UPLOAD_LOCATION}:/data
  - /etc/localtime:/etc/localtime:ro
  - /home/youruser/immich/library:/external:ro  # External library
```

### Configure in Web UI

1. Go to Administration → External Libraries
2. Create Library
3. Set import path to `/external` (the container path, not host path)
4. Click Scan

### Folder Structure Recommendations

```
library/
├── 2024/
│   ├── 01/
│   └── 12/
└── 2025/
```

Or by source:
```
library/
├── iphone/
├── camera/
└── scans/
```

Immich organizes photos by date in the UI regardless of folder structure—it reads EXIF metadata.

## Access

| Access Type | URL |
|-------------|-----|
| Local | http://192.168.x.x:2283 |
| Admin | http://192.168.x.x:2283/admin |
| External (via Caddy) | https://immich.yourserver.example.com |
| Internal (via Pi-hole) | https://immich.home |

## Mobile App

The Immich iOS/Android app supports:
- Automatic photo backup
- Background upload
- Selective album backup
- Timeline view synced with server

**Setup:**
1. Download "Immich" from App Store/Play Store
2. Enter server URL (HTTPS recommended)
3. Log in with your account
4. Enable backup in settings

## Troubleshooting

### Cannot Add External Library Path `/data`

**Problem:** Immich rejects `/data` as an external library path.

**Cause:** `/data` is Immich's internal upload folder—it won't accept it as an external library.

**Solution:** Mount your library as a separate path (e.g., `/external`) as shown above.

### High CPU and Temperature During Upload

**Problem:** After uploading photos, CPU usage hits 90%+, temps exceed 90°C.

**Solutions:**
1. Add CPU limit to ML container (recommended)
2. Let it finish if this is a one-time bulk upload
3. Disable facial recognition in admin settings if you don't need it
4. Monitor with Glances or similar tool

### Photos Not Appearing After Samba Upload

**Solutions:**
1. Go to External Libraries and click "Scan"
2. Check file permissions—Immich container needs read access
3. Remove macOS junk files: `find ~/immich/library -name '._*' -delete`
4. Ensure files are in a subfolder, not library root
5. Check logs: `docker logs immich_server`

## Containers

Immich runs as 4 containers:
- `immich_server` - Main application
- `immich_machine_learning` - AI processing (the CPU-intensive one)
- `immich_postgres` - Database
- `immich_redis` - Cache

## Useful Commands

```bash
# Check all Immich containers
docker ps | grep immich

# View server logs
docker logs immich_server

# View ML processing logs
docker logs immich_machine_learning

# Restart all Immich services
cd ~/immich && docker compose restart

# Full restart
cd ~/immich && docker compose down && docker compose up -d
```

## Backup Considerations

Important data locations:
- `~/immich/library/` - Actual photo files
- PostgreSQL data - Database (metadata, faces, albums)

For backup, prioritize the library folder. Database can be recreated by re-scanning, though you'd lose album organization and face names.
