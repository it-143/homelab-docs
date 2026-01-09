# Glances System Monitoring

Glances is a cross-platform system monitoring tool with a web interface.

## Installation

### Install lm-sensors First

For temperature monitoring:
```bash
sudo apt install lm-sensors -y
sudo sensors-detect --auto
sensors  # Test that it works
```

On Mac mini, `applesmc` and `coretemp` kernel modules provide sensor data.

### Run Glances Container
```bash
docker run -d \
  --name glances \
  --restart unless-stopped \
  -p 61208:61208 \
  -e GLANCES_OPT="-w" \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  -v /etc/os-release:/etc/os-release:ro \
  --pid host \
  nicolargo/glances:latest-full
```

**Flags explained:**
- `-e GLANCES_OPT="-w"` enables web server mode
- Docker socket mount shows container stats
- `--pid host` allows process monitoring

## Access

- **Web UI:** http://192.168.x.x:61208
- **Internal only** - not exposed externally

## Understanding the Dashboard

### CPU Section
- **User %:** Application CPU usage
- **System %:** Kernel/system CPU usage
- **IOWait %:** Time waiting on disk I/O (high = storage bottleneck)
- **Idle %:** Available CPU capacity

### Memory
- Total, used, free, buffers/cache
- Swap usage (should stay low)

### Load Average
Three numbers showing system load over 1, 5, and 15 minutes.

**Rule of thumb (for 4-thread CPU):**
- Under 4.0: Healthy
- 4.0-8.0: Under load but okay
- Over 8.0: System stressed

### Temperatures

**Healthy ranges:**
| State | Temperature |
|-------|-------------|
| Idle | 50-65°C |
| Normal use | 65-80°C |
| Heavy load | 80-90°C |
| **Warning** | 90°C+ |
| Critical | 100°C (throttling) |

### Fan Speed
Monitor fan RPM to gauge thermal load.

### Container Stats
Shows all Docker containers with CPU, memory, and network usage.

## Alerts/Warnings

Glances shows warnings for:
- **LOAD:** System load exceeded threshold
- **CPU_IOWAIT:** High disk wait time
- **MEM:** Memory usage high
- **SWAP:** Swap usage

### Common Warnings

**LOAD (brief spike):** Normal during batch operations. Concerning only if sustained.

**CPU_IOWAIT:** CPU waiting on disk. Common during large file transfers or database operations. Expected with HDD storage.

## Thermal Management Lessons

During Immich ML processing, observed:
- CPU at 90%+ usage
- Temperatures hitting 91-94°C
- Fan maxed out
- Load average exceeding 9.0

**Solutions:**
1. Physical: Improve airflow, consider cooling pad
2. Software: CPU limit on resource-intensive containers
3. Long-term: Replace thermal paste on older hardware

**Result after fixes:** Idle temps 72-78°C, fan quiet, load under 1.0

## Useful Commands

```bash
# Check status
docker ps | grep glances

# View logs
docker logs glances

# Restart
docker restart glances
```

## Alternative: Netdata

For more comprehensive monitoring with historical data:

```bash
docker run -d --name netdata \
  --restart=unless-stopped \
  --pid=host --network=host \
  -v netdataconfig:/etc/netdata \
  -v netdatalib:/var/lib/netdata \
  -v netdatacache:/var/cache/netdata \
  -v /:/host/root:ro,rslave \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --cap-add SYS_PTRACE \
  netdata/netdata
```

Access at http://192.168.x.x:19999

Glances is lighter weight—better for constrained hardware.
