# Troubleshooting & Lessons Learned

A compilation of problems encountered during this home lab build and their solutions.

## Ubuntu Installation

### Problem: Installer Stuck on DHCP

**Symptoms:**
- DHCP lease errors flooding screen
- Can't proceed with installation

**Root Cause:** No Ethernet cable connected. WiFi often needs drivers not in base installer.

**Solution:** Connect Ethernet, even temporarily for installation.

**Lesson:** Always have Ethernet ready for Linux server installations.

---

### Problem: Hardware Read Errors During Install

**Symptoms:**
- Kernel messages about I/O errors
- Installation seems slow

**Root Cause:** Some hardware (SD cards, older drives) have quirky Linux support.

**Solution:** Errors are often noisy but not fatal. Test if things work before deep troubleshooting.

---

## Networking

### Problem: ISP Blocking Ports

**Symptoms:**
- Port forward configured correctly
- External port checker shows port closed
- Works on LAN but not from internet

**Root Cause:** ISPs block common ports to prevent home servers.

**Solution:** Use alternate ports. After reverse proxy setup, only need 80/443.

**Lesson:** If port forward "doesn't work" despite correct config, try a different port.

---

### Problem: SSL Protocol Error for Remote Users

**Symptoms:**
- Remote user can't access server
- Browser shows `ERR_SSL_PROTOCOL_ERROR`

**Root Cause:** Modern browsers auto-upgrade HTTP to HTTPS.

**Solution:** Set up HTTPS properly with Caddy.

**Lesson:** HTTP-only servers are increasingly problematic. HTTPS should be default.

---

### Problem: Static IP Didn't Take on Raspberry Pi

**Symptoms:**
- Edited config file with static IP
- After reboot, old DHCP address still used

**Root Cause:** Newer Raspberry Pi OS uses NetworkManager, not dhcpcd.

**Solution:** Use `nmcli` commands instead of editing `/etc/dhcpcd.conf`.

**Lesson:** Verify which network manager is active before editing configs.

---

## Docker

### Problem: Container Not Starting After Reboot

**Symptoms:**
- Server rebooted
- Some containers not running

**Root Cause:** Containers created without restart policy.

**Solution:**
```bash
docker update --restart unless-stopped <container>
```

**Lesson:** Always include restart policy when creating containers.

---

### Problem: "No Configuration File" Error

**Symptoms:**
- `docker compose up -d` shows error
- But container works fine

**Root Cause:** Service was created with `docker run`, not compose.

**Solution:** Use `docker start <name>` instead.

**Lesson:** Document how each service was deployed.

---

## Thermal Management

### Problem: Server Overheating During Photo Import

**Symptoms:**
- CPU temps hitting 90°C+
- Fan maxed out
- System sluggish

**Root Cause:** ML processing using all available CPU.

**Solutions:**
1. Improve physical airflow
2. Add CPU limits to containers:
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '1.0'
   ```

**Lesson:** 
- Old hardware needs thermal consideration
- Limit resource-intensive containers
- Monitor temps during first big workloads

---

## File Organization

### Problem: Files Not Appearing in Applications

**Symptoms:**
- Added files via Samba
- Scanned library
- Files don't show up

**Causes Found:**

1. **macOS junk files:**
   ```bash
   find ~/media -name '._*' -delete
   ```

2. **Files in root directory:** Some apps need files in subfolders

3. **Long filenames:** Rename to something simple

**Lesson:** Keep filenames simple, organize in folders, clean macOS artifacts.

---

### Problem: Can't Write to Samba Share

**Symptoms:**
- Can browse share
- Can't paste/drag files

**Troubleshooting:**
1. Check `read only = no` in config
2. Verify user in Samba DB
3. Check file permissions
4. Restart Samba, reconnect fresh

**Root Cause:** Often stale macOS SMB connection.

**Lesson:** Disconnect completely and reconnect fresh when permissions seem wrong.

---

## Pi-hole

### Problem: Network Location Shortcut Fails

**Symptoms:**
- macOS Shortcut gives "Exit code 4"

**Root Cause:** Location name didn't match exactly.

**Solution:**
```bash
networksetup -listlocations  # Get exact names
```

**Lesson:** Shell commands are literal. Copy exact names.

---

## Caddy

### Problem: Subdomain Not Resolving

**Symptoms:**
- Base domain works
- Subdomains don't resolve

**Solution:** Many DDNS providers support wildcard DNS automatically. Test properly before assuming it's broken.

---

## WireGuard

### Problem: VPN Connects But Can't Reach Services

**Symptoms:**
- WireGuard shows connected
- Can't access anything

**Root Cause:** Device resolving to IPv6, VPN only routes IPv4.

**Solution:** Use IPv4 address directly in Endpoint field.

---

## General Lessons

### Document Everything
Write down every command, config change, and decision.

### Test From External Network
Local testing isn't sufficient. Use phone on cellular or friend's connection.

### One Change at a Time
Multiple simultaneous changes make debugging impossible.

### Backups Before Changes
Even just `cp file file.bak` helps.

### Read Error Messages Carefully
"Connection refused" ≠ "Connection timed out" ≠ "SSL error"

### Hardware Limits Matter
Know your hardware's limits and configure accordingly.

### Security Through Layers
- Reverse proxy (single entry point)
- HTTPS (encryption)
- VPN (secure remote access)
- Minimal port exposure
