# Samba File Sharing

Samba enables Windows/macOS file sharing to the Ubuntu server.

## Installation

```bash
sudo apt install samba -y
```

## Configuration

### Main Config File

Edit `/etc/samba/smb.conf`:

```bash
sudo nano /etc/samba/smb.conf
```

### Example Shares

Add at the bottom:

```ini
[media]
   path = /home/youruser/media
   browseable = yes
   read only = no
   valid users = youruser

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

[photos]
   path = /home/youruser/immich/library
   browseable = yes
   read only = no
   valid users = youruser
```

### Apply Changes

```bash
sudo systemctl restart smbd
```

## Samba User Setup

Samba has its own user database, separate from Linux users.

### Add User
```bash
sudo smbpasswd -a youruser
# Enter password when prompted
```

### Verify User Exists
```bash
sudo pdbedit -L
```

## Connecting from macOS

### Via Finder

1. Finder → Go → Connect to Server (Cmd+K)
2. Enter: `smb://192.168.x.x`
3. Click Connect
4. Authenticate with Samba credentials
5. Select share(s) to mount

## Troubleshooting

### Can't Write to Share

**Causes and Solutions:**

1. **Share configured read-only:**
   Check smb.conf has `read only = no`

2. **User not in Samba database:**
   ```bash
   sudo pdbedit -L | grep youruser
   # If not listed:
   sudo smbpasswd -a youruser
   ```

3. **File permissions on host:**
   ```bash
   ls -la /home/youruser/media
   sudo chown -R youruser:youruser /home/youruser/media
   ```

4. **Stale connection:**
   Eject all shares, reconnect fresh

5. **Restart Samba:**
   ```bash
   sudo systemctl restart smbd
   ```

### macOS Junk Files

macOS creates hidden `._filename` files when copying.

**Remove them:**
```bash
find /home/youruser/media -name '._*' -delete
```

These files can confuse applications like Immich and Kavita.

### Config Test

```bash
testparm -s
```

## Security Notes

### Local Network Only

Samba (port 445) should NOT be forwarded through router. Shares are only accessible on local network.

### Per-Share Access Control

```ini
[private]
   path = /home/youruser/private
   browseable = no      # Hidden from browse list
   read only = no
   valid users = youruser

[shared]
   path = /home/youruser/shared
   read only = yes      # Read-only default
   valid users = youruser, guest
   write list = youruser   # Only youruser can write
```

## Useful Commands

```bash
# Check service status
sudo systemctl status smbd

# Restart Samba
sudo systemctl restart smbd

# View active connections
sudo smbstatus

# List configured shares
testparm -s 2>/dev/null | grep -E '^\[|path'
```
