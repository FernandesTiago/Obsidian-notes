# WireGuard Server

VPN server running on the Pi. Allows remote access to the home network from any peer (phone, notebook) over a single UDP port.

## Objective

- Encrypted tunnel to home network from anywhere
- Force DNS through Pi-hole (ad blocking on the go)
- Single point of remote access for SSH, dashboards, future services

## State

- Service: `wg-quick@wg0.service` (active, enabled at boot)
- Interface: `wg0`
- Listen port: UDP `<VPN_PORT>` (custom, non-default — see [[IPs]])
- VPN range: `<VPN_RANGE>` (see [[IPs]])
- Server config: `/etc/wireguard/wg0.conf` (mode `600`, owner root)
- Keys directory: `/etc/wireguard/` — root-only access

Active peers:
- iPhone (Tiago)
- Notebook Mint

Both Pi-hole DNS forwarding (`DNS = 1.1.1.1` is fallback; actual DNS goes through Pi-hole because peers route `AllowedIPs = 0.0.0.0/0` and the Pi-hole sits in the routed range).

## Network topology requirement

WireGuard server must sit behind **only one NAT layer** to receive incoming UDP from the internet. Original setup had double NAT (modem → router → Pi) and handshake never completed from external peers.

Current topology:

```
Internet
   │
   ▼
[Modem/Router Claro]  ← port forward UDP <VPN_PORT> → Pi
   │
   ├── [Switch] ── Pi (WireGuard server)
   └── [Switch] ── [OpenWRT — bedroom]
```

Previous (broken) topology:

```
Modem Claro (NAT) → Switch → TP-Link (NAT) → Pi
	    ↑                          ↑
	double NAT, peers couldn't reach Pi
```

The fix was repositioning the Pi to sit directly off the modem's switch, bypassing the TP-Link's NAT. (I could have allowed the port on both NAT, but I preferred only having one NAT)

## Setup

### 1. Install

```bash
sudo apt update
sudo apt install wireguard
```

### 2. Generate server keys

```bash
cd /etc/wireguard
sudo wg genkey | sudo tee server_privatekey | wg pubkey | sudo tee server_publickey
sudo chmod 600 server_privatekey server_publickey
```

### 3. Create server config

`/etc/wireguard/wg0.conf`:

```ini
[Interface]
Address = <VPN_SERVER_IP>/24
ListenPort = <VPN_PORT>
DNS = 1.1.1.1
PrivateKey = <REDACTED>

PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# iPhone Tiago
PublicKey = <peer-pubkey>
AllowedIPs = <peer-vpn-ip>/32

[Peer]
# Notebook Mint
PublicKey = <peer-pubkey>
AllowedIPs = <peer-vpn-ip>/32
```

`PostUp` / `PostDown` rules enable IP forwarding and NAT masquerading so VPN traffic can reach the LAN and exit through `eth0`.

### 4. Enable IP forwarding (kernel-level)

`/etc/sysctl.conf`:

```
net.ipv4.ip_forward = 1
```

Apply:

```bash
sudo sysctl -p
```

### 5. Start and enable service

```bash
sudo systemctl enable --now wg-quick@wg0
```

### 6. Port forward on modem

Forward UDP `<VPN_PORT>` from modem → Pi LAN IP. See [[IPs]].

### 7. Dynamic DNS

Endpoint resolves through DuckDNS so peers don't need to track the changing public IP. See [[duckdns]].

## Adding a new peer

### On the peer device

```bash
wg genkey | tee peer_privatekey | wg pubkey > peer_publickey
```

Build peer config (example for a Linux peer):

```ini
[Interface]
PrivateKey = <peer-private-key>
Address = <new-peer-vpn-ip>/24
DNS = <PI_LAN_IP>

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <duckdns-subdomain>.duckdns.org:<VPN_PORT>
AllowedIPs = <LAN_RANGE>, <VPN_RANGE>
PersistentKeepalive = 25
```

`AllowedIPs` here controls what gets routed *through* the VPN on the peer side. Including `0.0.0.0/0` would route all traffic; including only the LAN+VPN ranges is split-tunnel (more efficient).

### On the server

Append to `/etc/wireguard/wg0.conf`:

```ini
[Peer]
# <peer-name>
PublicKey = <peer-public-key>
AllowedIPs = <new-peer-vpn-ip>/32
```

`AllowedIPs` on the server side is restrictive — only this peer's VPN IP. This prevents IP spoofing between peers.

Apply without dropping existing connections:

```bash
sudo wg syncconf wg0 <(wg-quick strip wg0)
```

(A `systemctl restart wg-quick@wg0` also works but briefly disconnects all peers.)

## Verify

```bash
sudo wg show
sudo systemctl status wg-quick@wg0
```

Healthy peer shows recent `latest handshake` (within minutes). If a peer has been idle, handshake timestamp grows but reconnects on next traffic.

From a peer device, after connecting:

```bash
ping <PI_LAN_IP>          # should succeed
nslookup google.com       # should return through Pi-hole
```

## Troubleshoot

### Peer connects but no traffic flows

**Symptom:** handshake succeeds, but `ping` to LAN IPs times out.

**Cause:** IP forwarding disabled, or iptables rules missing.

**Fix:**

```bash
sysctl net.ipv4.ip_forward            # must be 1
sudo iptables -L FORWARD -v           # must show ACCEPT rules
sudo iptables -t nat -L POSTROUTING   # must show MASQUERADE on eth0
```

If forwarding is 0, re-apply `sysctl -p`. If iptables rules are missing, the `PostUp` from `wg0.conf` didn't fire — restart the service.

### External peer never establishes handshake

**Symptom:** handshake stays at "(none)" or never updates from external networks. Works only on LAN.

**Cause:** double NAT (Pi sits behind a second router), or port forward misconfigured.

**Fix:**
- Confirm the Pi sits directly behind the modem (one NAT layer)
- Verify the modem's port forward rule sends UDP `<VPN_PORT>` to the Pi's LAN IP
- Check the Pi's LAN IP is static/reserved (DHCP renewal would break the forward)

### DNS leaks (peer uses ISP DNS instead of Pi-hole)

**Symptom:** peer connected, ads not blocked.

**Cause:** peer's `DNS = ` points elsewhere, or OS overrides VPN DNS (common on iOS/Android).

**Fix:**
- Set `DNS = <PI_LAN_IP>` in the peer's `[Interface]` section
- On iOS, ensure "Include All Networks" is OFF (kills connectivity) but VPN DNS does apply by default
- Test with `nslookup pi.hole` on peer — should resolve to Pi's LAN IP

### Conflict with ProtonVPN on notebook

**Symptom:** WireGuard up, but no traffic; or ProtonVPN won't connect.

**Cause:** both VPNs try to set default route — mutually exclusive.

**Fix:** disable one before enabling the other:

```bash
sudo wg-quick down wg0      # before connecting Proton
sudo wg-quick up wg0        # before reconnecting Pi VPN
```

See also [[notebook-mint-i3]] for peer config.

## Revert

To stop and disable:

```bash
sudo systemctl disable --now wg-quick@wg0
```

To remove entirely:

```bash
sudo systemctl disable --now wg-quick@wg0
sudo rm -rf /etc/wireguard
sudo apt remove --purge wireguard
```

Don't forget to remove the modem's port forward rule too.

## Security notes

- Server `PrivateKey` is the master credential. Anyone with it can impersonate the server. File mode `600`, owner `root` only.
- VPN traffic is encrypted, but the Pi's residential IP is still visible to destinations — WireGuard hides traffic *content*, not *origin*. This is unlike commercial VPNs.
- Custom UDP port reduces exposure to mass scanners (most scan only common ports). Not security-by-itself, but useful layer.
- No password auth on WireGuard — pure public-key crypto. Compromise requires stealing a private key, not guessing.

## References

- [[IPs]] — VPN port, ranges, peer assignments
- [[duckdns]] — dynamic DNS for peer endpoint
- [[Pihole Private]] — DNS resolution for VPN peers
- [[notebook-mint-i3]] — notebook peer config
- [[backup-system]] — `/etc/wireguard/` is included in the backup