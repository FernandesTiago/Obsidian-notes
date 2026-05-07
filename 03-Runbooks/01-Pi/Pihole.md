# Pi-hole

Network-wide ad and tracker blocker. DNS sinkhole that intercepts queries to known ad/tracker domains and returns null responses.

## Objective

- Block ads and trackers across all devices on the LAN without per-device configuration
- Provide DNS resolution for the LAN (replaces ISP/router DNS)
- Provide DNS resolution for VPN peers (so ad blocking works remotely too)

## State

- Versions: Core v6.4.1, Web v6.5, FTL v6.6 (check `pihole -v`)
- Listening on port 53 (TCP/UDP, IPv4/IPv6)
- Web admin: `http://<PI_IP>/admin`
- Config files: `/etc/pihole/`
- Main DB: `/etc/pihole/gravity.db` (block lists)
- Stats DB: `/etc/pihole/pihole-FTL.db` (queries log)

DNS clients reach Pi-hole because:
- Direct LAN clients: OpenWRT pushes Pi-hole IP via DHCP option 6 (see [[dhcp-option-6]])
- VPN peers: WireGuard config sets `DNS = <PI_IP>` (see [[wireguard-server]])

## Setup

### 1. Install (one-line installer)

```bash
curl -sSL https://install.pi-hole.net | bash
```

During install:
- Select upstream DNS providers (e.g., Cloudflare 1.1.1.1)
- Accept default block list (StevenBlack hosts)
- Enable web interface
- Enable web server (lighttpd)
- Enable query logging
- Set privacy level (default: show everything)

After install, set admin password:

```bash
pihole -a -p
```

### 2. Configure additional adlists

Web UI → Adlists → add URLs. Or via CLI/SQL:

```bash
sqlite3 /etc/pihole/gravity.db "INSERT INTO adlist (address, enabled, comment) VALUES ('<URL>', 1, '<NAME>');"
pihole -g    # rebuild gravity database
```

Active adlists currently:
- `https://adaway.org/hosts.txt` — AdAway mobile ads
- `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts` — StevenBlack unified
- `https://v.firebog.net/hosts/AdguardDNS.txt` — AdGuard DNS
- `https://v.firebog.net/hosts/Easyprivacy.txt` — EasyPrivacy
- `https://v.firebog.net/hosts/Prigent-Ads.txt` — Prigent Ads
- `https://raw.githubusercontent.com/PolishFiltersTeam/KADhosts/master/KADhosts.txt` — KAD scam/fraud blocking

### 3. Force LAN clients through Pi-hole

DHCP option 6 in OpenWRT pushes `<PI_IP>` as DNS to all DHCP clients. See [[dhcp-option-6]].

### 4. Force VPN peers through Pi-hole

In `/etc/wireguard/wg0.conf` (server side), peer configs include `DNS = <PI_IP>`. See [[wireguard-server]].

## Verify

### Service health

```bash
pihole status
pihole -v
```

Should show all four DNS listeners (TCP/UDP × IPv4/IPv6) active and "Pi-hole blocking is enabled".

### From a LAN client

```bash
nslookup pi.hole
# should return <PI_IP>

nslookup doubleclick.net
# should return 0.0.0.0 (blocked) or NXDOMAIN
```

### From the web UI

Navigate to `http://<PI_IP>/admin`. Dashboard shows:
- Total queries
- Queries blocked
- Percentage blocked
- Top clients
- Top blocked domains

## Common operations

### Update gravity (refresh blocklists)

```bash
pihole -g
```

Runs automatically once a week by default. Force after editing adlists.

### Whitelist a domain

```bash
pihole allow <domain>
```

Or web UI → Domains → Allow.

### Blacklist a specific domain

```bash
pihole deny <domain>
```

### Pause blocking temporarily

```bash
pihole disable 5m
pihole enable
```

Useful when something legitimate is being blocked and the test is faster than diagnosing.

### View live query log

```bash
pihole tail
```

Or web UI → Tools → Query Log.

### Reset web admin password

```bash
pihole -a -p
```

## Troubleshoot

### Some clients ignore Pi-hole

**Symptom:** ads still appear on certain devices.

**Cause:** device hardcodes its own DNS (smart TVs, Chromecast, some IoT) and ignores DHCP option 6.

**Fix:**
- Block port 53 on OpenWRT for that device's MAC, except to `<PI_IP>` — forces all DNS through Pi-hole
- Or block known DNS-over-HTTPS endpoints (Cloudflare DoH, Google DoH) at firewall level
- For Chromecast/Google devices: known to bypass via hardcoded `8.8.8.8` — firewall rule required

### DNS resolution slow after Pi-hole install

**Symptom:** browsing feels laggy, especially on first request to a new domain.

**Cause:** upstream DNS slow, or Pi-hole hitting cache misses.

**Fix:**
- Check upstream DNS in web UI → Settings → DNS. Try Cloudflare (1.1.1.1) or Quad9 (9.9.9.9)
- Increase cache size: web UI → Settings → DNS → Advanced → Cache size
- Consider Unbound for recursive resolution (planned)

### Web UI inaccessible

**Symptom:** `http://<PI_IP>/admin` doesn't load.

**Cause:** lighttpd stopped, or port 80 conflict.

**Fix:**

```bash
sudo systemctl status lighttpd
sudo systemctl restart lighttpd
sudo ss -tlnp | grep :80    # check what's listening on port 80
```

### Gravity update fails

**Symptom:** `pihole -g` errors out.

**Cause:** network issue, blocklist URL changed/dead, or disk full.

**Fix:**

```bash
df -h /etc/pihole         # disk space
pihole -g 2>&1 | tee /tmp/gravity.log    # full log
```

Check log for which URL failed; remove or replace the dead adlist via web UI.

### Updating Pi-hole itself

```bash
pihole -up
```

Updates Core, Web, and FTL components. Configs are preserved across updates.

## Backup

Pi-hole's `/etc/pihole/` is included in the main backup (see [[backup-system]]). To export blocklist DB separately:

```bash
sudo cp /etc/pihole/gravity.db ~/gravity-backup-$(date +%F).db
```

Restore by copying back and running `pihole -g`.

## Revert

To temporarily disable:

```bash
pihole disable
```

To uninstall completely:

```bash
pihole uninstall
```

This removes Pi-hole but preserves `/etc/pihole/` configs in case of reinstall.

## Security notes

- `cli_pw` and TLS cert/key files in `/etc/pihole/` are root-only. Don't expose them.
- Web admin password: stored hashed in Pi-hole config. Reset with `pihole -a -p`.
- Don't expose port 80/443 of the Pi to the internet. The web UI is for LAN/VPN access only.
- Pi-hole logs DNS queries — privacy implication if shared. Privacy level is configurable in web UI → Settings → Privacy.

## References

- [[wireguard-server]] — VPN peers use Pi-hole as DNS
- [[dhcp-option-6]] — OpenWRT pushes Pi-hole as DNS to LAN clients
- [[backup-system]] — `/etc/pihole/` backed up daily
- [[IPs]] — Pi LAN IP
- [Pi-hole docs](https://docs.pi-hole.net/)