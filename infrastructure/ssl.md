# SSL/HTTPS Configuration

All external services are protected with HTTPS via Caddy's automatic certificate management.

## Certificate Types

### External Access: Let's Encrypt (Automatic)

For your public domain, Caddy automatically:
1. Requests certificates from Let's Encrypt
2. Completes HTTP-01 challenge (requires port 80)
3. Renews before expiration

**Requirements:**
- Port 80 must be forwarded to Caddy
- DNS must resolve to your public IP
- Domain must be publicly accessible

### Internal Access: Self-Signed (Internal CA)

For `.home` domains, Caddy uses `tls internal`:
1. Generates certificates from internal CA
2. Browsers show warning (expected)
3. Traffic is still encrypted

## How It Works

```
External Request:
https://jellyfin.yourserver.example.com
    │
    ▼
[Internet] → [Router :443] → [Caddy]
                                │
                                ├── Terminates TLS
                                ├── Decrypts request
                                ├── Routes to internal service
                                │
                                ▼
                            [Service]
```

Caddy handles all encryption. Backend services run on HTTP internally.

## Troubleshooting

### Certificate Not Issuing

1. **Port 80 forwarded?** Let's Encrypt needs port 80
2. **DNS resolving correctly?** `nslookup yourdomain.com`
3. **Caddy logs:** `docker logs caddy`
4. **Rate limits?** Let's Encrypt has limits—wait if exceeded

### Certificate Renewal Failing

1. Check dynamic DNS is current
2. Port 80 still forwarded?
3. Check Caddy logs for ACME errors

### ERR_SSL_PROTOCOL_ERROR

**Cause:** Client trying HTTPS but server only speaks HTTP.

**Solution:** Use correct HTTPS URL through Caddy.

### Certificate Warning for .home Domains

**Expected behavior.** Self-signed certificates aren't trusted by default.

Click through—traffic is still encrypted.

## Verifying HTTPS

### Browser Check
- Lock icon in address bar
- Click for certificate details

### Command Line
```bash
echo | openssl s_client -connect yourdomain.com:443 2>/dev/null | openssl x509 -noout -dates
```

## Security Benefits

1. **Encryption:** All data encrypted in transit
2. **Authentication:** Certificate proves server identity
3. **Integrity:** Detects tampering
4. **Browser trust:** No warnings for external access
5. **App compatibility:** Many apps require HTTPS
