# DuckDNS

Dynamic DNS service that maps a free subdomain to the home's current public IP. Required because residential ISPs assign dynamic IPs that change without notice.

## Objective

Provide a stable hostname that WireGuard peers can use as `Endpoint`, regardless of how often the modem's public IP rotates.

## State

- Update script: `~/duckdns/duck.sh` (Pi user)
- Log file: `~/duckdns/duck.log`
- Schedule: cron, every 5 minutes
- Subdomain and token: see [[DuckDNS Private]] in private notes

## Setup

### 1. Create account and subdomain

At [duckdns.org](https://www.duckdns.org), sign in (GitHub/Google), create a subdomain. Note the token shown on the dashboard.

### 2. Create script directory

```bash
mkdir -p ~/duckdns
```

### 3. Create update script

`~/duckdns/duck.sh`:

```bash
echo url="https://www.duckdns.org/update?domains=<SUBDOMAIN>&token=<TOKEN>&ip=" | curl -k -o ~/duckdns/duck.log -K -
```

Real values from [[DuckDNS Private]]. The `ip=` is left empty so DuckDNS uses the request's source IP — which is the modem's public IP from the Pi's perspective.

```bash
chmod 700 ~/duckdns/duck.sh
```

### 4. Schedule via cron

```bash
crontab -e
```

```cron
*/5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1
```

Every 5 minutes is DuckDNS's recommended interval. Output is silenced because the script already writes to `duck.log`.

## Verify

After the first cron run (within 5 minutes):

```bash
cat ~/duckdns/duck.log
```

Should output `OK`. `KO` means failure (bad token, bad subdomain, or network issue).

External resolution:

```bash
nslookup <SUBDOMAIN>.duckdns.org
```

Should return the modem's current public IP. Compare against [whatismyip.com](https://whatismyip.com) on a device behind the same modem.

## Troubleshoot

### `KO` in `duck.log`

**Symptom:** log shows `KO` instead of `OK`.

**Cause:** wrong token, wrong subdomain, or DuckDNS rejected the request.

**Fix:**
- Verify subdomain and token in [[DuckDNS Private]] match the DuckDNS dashboard
- Test manually: `bash ~/duckdns/duck.sh && cat ~/duckdns/duck.log`
- Regenerate token at duckdns.org if compromised, update both [[DuckDNS Private]] (private notes) and Bitwarden, update `duck.sh`

### Subdomain resolves to wrong IP

**Symptom:** `nslookup` returns an outdated IP.

**Cause:** DNS cache, or DuckDNS hasn't been updated recently.

**Fix:**
- Force run: `bash ~/duckdns/duck.sh`
- Wait 1-2 minutes for DNS propagation
- Flush local DNS cache: `sudo systemd-resolve --flush-caches` (or restart Pi-hole if querying through it)

### WireGuard peers can't connect after IP change

**Symptom:** peers worked yesterday, today handshake fails.

**Cause:** modem rebooted and got a new IP, but DuckDNS update lagged.

**Fix:**
- Force update: `bash ~/duckdns/duck.sh`
- Reconnect peers after `OK` confirmation in log

## Revert

To stop updates:

```bash
crontab -e   # remove the duck.sh line
```

To remove entirely:

```bash
crontab -e   # remove line
rm -rf ~/duckdns
```

DuckDNS will keep the subdomain pointing at the last known IP indefinitely (or until manually deleted on the dashboard).

## Security notes

- The DuckDNS token is a credential — anyone with it can repoint the subdomain anywhere. Treat it like a password.
- Token lives in Bitwarden, referenced from [[DuckDNS Private]] private notes.
- Subdomain itself isn't sensitive on its own, but combined with a known service port it reveals attack surface. Avoid posting it publicly.
- Rotation procedure: regenerate token at duckdns.org → update Bitwarden → update `duck.sh` → run manually to confirm `OK`.

## References

- [[DuckDNS Private]] — subdomain and token reference (private)
- [[wireguard-server]] — uses this hostname as VPN endpoint
- [[IPs]] — Pi LAN IP (port forward target)