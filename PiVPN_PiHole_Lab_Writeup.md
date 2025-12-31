# Home Lab DNS + VPN Stack (Pi-hole + PiVPN WireGuard)
## Network-Wide Ad Blocking & Secure Remote Access

**Author:** Alex Lux  
**Date:** December 2025  
**Environment:** Home Lab - Raspberry Pi / Linux VM  
**OS:** Linux pivpn 6.12.47+rpt-rpi-v7 #1 SMP Raspbian 1:6.12.47-1+rpt1~bookworm (2025-09-16) armv7l GNU/Linux

> **Quick Reference Available**: For command cheat sheets and quick troubleshooting, see `QUICK_REFERENCE.md` in this directory.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Goals & Secure Defaults](#goals--secure-defaults)
4. [Prerequisites](#prerequisites)
5. [Step 1: Stabilize Host IP](#step-1--stabilize-the-host-ip-important)
6. [Step 2: Install PiVPN (WireGuard)](#step-2--install-pivpn-wireguard)
7. [Step 3: Install Pi-hole (LAN-only DNS)](#step-3--install-pi-hole-lan-only-dns)
8. [Step 4: Configure Router DNS](#step-4--point-eero-dns-to-pi-hole-lan-only)
9. [Operational Walkthrough: Reading Pi-hole Logs](#operational-walkthrough--reading-pi-hole-logs)
10. [PiVPN Client Provisioning](#pivpn-client-provisioning-wireguard)
11. [Router Port Forward Configuration](#router-eero--port-forward-for-wireguard)
12. [Validation Checklist](#validation-checklist-copypaste)
13. [Troubleshooting Playbook](#troubleshooting-playbook)
14. [Key Skills Demonstrated](#key-skills-demonstrated)
15. [Quick Build Script](#quick-copypaste-build-script-high-level)

---

## Project Overview

### Objective
Deploy a reproducible home-lab stack combining network-wide ad/tracker blocking via Pi-hole DNS filtering and secure remote access via WireGuard VPN (PiVPN) on a single Linux host. This setup provides:

- **Network-wide ad blocking**: All LAN devices automatically use Pi-hole for DNS filtering
- **Secure VPN access**: Remote access to home network via WireGuard
- **Minimal attack surface**: Only WireGuard port exposed to WAN
- **Easy client management**: QR codes and simple config files for VPN clients

### Why This Stack?
- **Pi-hole**: Industry-standard DNS filtering with web UI, blocklist management, and query logging
- **PiVPN**: Automated WireGuard setup with client provisioning tools
- **Single host**: Reduces hardware requirements and simplifies management
- **Production-ready**: Secure defaults, proper network isolation, operational tooling

### Architecture Summary
```
Internet → Eero Router → LAN Devices (use Pi-hole DNS)
                    ↓
              PiVPN Host (192.168.4.127)
              ├── Pi-hole (Port 53, 80 - LAN only)
              └── WireGuard (Port 51820/UDP - WAN exposed)
                    ↓
              VPN Clients (10.155.185.0/24)
```

---

## Architecture Overview

### Components

**Amazon eero Router:**
- Router + DHCP (hands out IP addresses)
- Port forwarding for WireGuard (UDP 51820)

**PiVPN Host (Raspberry Pi or Linux VM):**
- **Pi-hole**: LAN DNS filtering + local DNS (Port 53, 80)
- **PiVPN (WireGuard)**: Remote VPN access (Port 51820/UDP)
- **Static IP**: 192.168.4.127/22 (via DHCP reservation)

**Clients:**
- **LAN devices** (phones, TVs, laptops): Use Pi-hole for DNS automatically
- **VPN clients** (phone/laptop when away): Connect via WireGuard to access LAN

### Network Configuration (Example)

Example configuration:

- **LAN subnet**: 192.168.4.0/22 (covers 192.168.4.0 – 192.168.7.255)
- **PiVPN host**:
  - `eth0`: 192.168.4.127/22 (static via DHCP reservation)
  - `wg0`: 10.155.185.1/24 (WireGuard VPN subnet)
- **MAC address**: b8:27:eb:51:7c:e7 (for DHCP reservation)

### Traffic Flow

**LAN DNS (Pi-hole LAN-only):**
```
Client → DNS query → Pi-hole (192.168.4.127:53) → upstream DNS → response
```

**Remote VPN:**
```
Phone/Laptop → WireGuard tunnel → PiVPN host → LAN resources
(Optional) VPN client DNS can also use Pi-hole when connected
```

---

## Goals & Secure Defaults

### Goals

1. **Pi-hole is LAN-only** (no WAN exposure of DNS/UI)
2. **WireGuard is the only service exposed** via port-forward on the router (UDP)
3. **Strong keys, minimal open ports**, easy client provisioning
4. **Stable IP addressing** via DHCP reservation

### Secure Defaults

**Do NOT port-forward:**
- ❌ TCP/80 (Pi-hole UI)
- ❌ TCP/UDP 53 (DNS)

**Only port-forward:**
- ✅ UDP 51820 (WireGuard) from WAN → PiVPN host

**Additional Security:**
- Set a stable IP for the PiVPN host (DHCP reservation recommended)
- Use strong WireGuard keys (PiVPN generates these automatically)
- VPN clients can optionally use Pi-hole DNS when connected

---

## Prerequisites

### Hardware / OS

**Raspberry Pi** (or any Debian/Ubuntu host) running:
- Raspberry Pi OS Lite / Debian / Ubuntu Server
- Minimum 1GB RAM (2GB+ recommended)
- 8GB+ storage (SD card or disk)

### Prerequisites

1. **Local shell access** (SSH)
2. **A stable LAN IP** for the host (set via DHCP reservation)
3. **eero admin access** (to set DHCP reservation + custom DNS)
4. Root or sudo access on the host

---

## Step 1 — Stabilize the Host IP (Important)

Example `ip a` output:
```
inet 192.168.4.127/22 ... dynamic
```

**Fix:** Set a DHCP reservation in eero so the Pi always gets 192.168.4.127.

### Identify MAC Address

From the output:
```
eth0 MAC: b8:27:eb:51:7c:e7
```

### Configure DHCP Reservation in Eero

1. **Access eero admin interface**
   - Open eero app or web interface
   - Navigate to **Network Settings** > **DHCP & NAT** or **Reservations**

2. **Add reservation**
   - **Device**: Select the Pi (or add manually)
   - **MAC Address**: `b8:27:eb:51:7c:e7`
   - **IP Address**: `192.168.4.127`
   - **Save**

### Outcome

- ✅ Pi-hole stays reachable at 192.168.4.127
- ✅ DNS settings in eero don't break after reboots
- ✅ VPN clients can reliably connect to the host

**Verify after reboot:**
```bash
ip a show eth0
# Should show: inet 192.168.4.127/22
```

---

## Step 2 — Install PiVPN (WireGuard)

PiVPN is a guided installer that configures WireGuard cleanly.

### Install

```bash
sudo apt-get update
sudo apt-get -y upgrade
curl -L https://install.pivpn.io | bash
```

### Installer Choices (Recommended)

**VPN type:**
- Select **WireGuard** (modern, faster than OpenVPN)

**WireGuard port:**
- Default: **51820/UDP** (recommended)

**DNS for VPN clients:**
- **Option 1**: If you want VPN clients to use Pi-hole later: choose **Custom** and set `192.168.4.127`
- **Option 2**: If VPN is "remote access only" and you don't care about DNS filtering remotely: choose **Cloudflare** (1.1.1.1) or **Quad9** (9.9.9.9)

**User for management:**
- Select the current user (or create a dedicated user)

### Enable IP Forwarding

PiVPN normally handles this automatically, but verify:

```bash
sysctl net.ipv4.ip_forward
```

**Expected output:**
```
net.ipv4.ip_forward = 1
```

If it shows `0`, enable it:
```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Verify WireGuard is Up

```bash
# Check service status
sudo systemctl status wg-quick@wg0 --no-pager

# Show WireGuard configuration
sudo wg show

# Show interface details
ip a show wg0
```

**Example output (`ip a show wg0`):**
```
4: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP> mtu 1420 qdisc noqueue state UNKNOWN group default qlen 1000
    link/none
    inet 10.155.185.1/24 scope global wg0
       valid_lft forever preferred_lft forever
```

**Example output (`sudo wg show`):**
```
interface: wg0
  public key: <server_public_key>
  private key: (hidden)
  listening port: 51820

peer: <client_public_key>
  endpoint: <client_ip>:<port>
  allowed ips: 10.155.185.2/32
  latest handshake: 2 minutes ago
  transfer: 12.34 MiB received, 45.67 MiB sent
```

---

## Step 3 — Install Pi-hole (LAN-only DNS)

### Install

```bash
curl -sSL https://install.pi-hole.net | bash
```

### Installer Choices

**Interface:**
- Select **eth0** (or the primary LAN interface)

**IP:**
- Should auto-detect: **192.168.4.127/22**
- Verify this matches the static IP

**Upstream DNS:**
- Choose from:
  - **Cloudflare**: 1.1.1.1, 1.0.0.1
  - **Quad9**: 9.9.9.9, 149.112.112.112
  - **Google**: 8.8.8.8, 8.8.4.4
  - **OpenDNS**: 208.67.222.222, 208.67.220.220

**Web admin UI:**
- **Enabled** (recommended for management)

**Query logging:**
- **Enabled** (useful for troubleshooting and monitoring)

### If Install Warns "Port 53 in Use"

Check who is listening:
```bash
sudo ss -lntup | grep ':53'
```

**If it's systemd-resolved** (common on Ubuntu):
```bash
sudo systemctl disable --now systemd-resolved
sudo rm -f /etc/resolv.conf
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf >/dev/null
```

Then rerun the Pi-hole installer.

### Post-Installation

**Note the admin password:**
- The installer will display a random password
- Save it for accessing the web UI
- Can be changed later: `pihole -a -p`

**Access web UI:**
- URL: `http://192.168.4.127/admin`
- Or: `http://pi.hole/admin`

---

## Step 4 — Point Eero DNS to Pi-hole (LAN-only)

### Configure Custom DNS in Eero

1. **Access eero admin interface**
   - Open eero app or web interface
   - Navigate to **Network Settings** > **DNS**

2. **Set Custom DNS**
   - **Primary DNS**: `192.168.4.127`
   - **Secondary DNS**:
     - Ideally **blank** (if allowed)
     - If required: Consider a second Pi-hole instance later; otherwise some clients may bypass filtering occasionally

3. **Save settings**

### Validate a LAN Client is Using Pi-hole

On a laptop on the LAN:

```bash
# Test normal DNS resolution
nslookup example.com 192.168.4.127

# Test blocked domain (should return 0.0.0.0 or NXDOMAIN)
nslookup doubleclick.net 192.168.4.127
```

**Expected output for blocked domain:**
```
Server:		192.168.4.127
Address:	192.168.4.127#53

Name:	doubleclick.net
Address: 0.0.0.0
```

Or:
```
** server can't find doubleclick.net: NXDOMAIN
```

**Note:** The exact response depends on your Pi-hole block mode configuration.

---

## Operational Walkthrough — Reading Pi-hole Logs

### Live Tail

```bash
sudo pihole -t
```

### Example Output (Annotated)

Based on the pasted output:

```
16:51:38: query[A] app-analytics-v2.snapchat.com from 192.168.5.2
```

**Meaning:** A device at 192.168.5.2 asked Pi-hole for IPv4 (A record)

```
16:51:38: gravity blocked app-analytics-v2.snapchat.com is 0.0.0.0
16:51:38: gravity blocked app-analytics-v2.snapchat.com is ::
```

**Meaning:** Pi-hole blocklists ("gravity") blocked it. Returned 0.0.0.0 for IPv4 and :: for IPv6.

```
16:51:56: forwarded pancake.g.aaplimg.com to 8.8.8.8
```

**Meaning:** Pi-hole didn't answer locally (or refreshed), forwarded to upstream DNS (Google here)

```
reply pancake.apple.com is <CNAME>
reply pancake.g.aaplimg.com is 17.253.5.201
```

**Meaning:** Apple domain uses an alias (CNAME), final answer is an IP.

```
reply get-bx.g.aaplimg.com is NODATA
```

**Meaning:** That query type had no records at that moment (common with mixed record types). Not an error by itself.

### Understanding Log Entries

- **`query[A]`**: IPv4 address query
- **`query[AAAA]`**: IPv6 address query
- **`gravity blocked`**: Domain is in blocklist
- **`forwarded`**: Query sent to upstream DNS
- **`cached`**: Answer served from cache (not shown in this example)
- **`reply`**: Response from upstream DNS

---

## PiVPN Client Provisioning (WireGuard)

### Create a New Client Profile

```bash
pivpn add
```

**Prompts:**
- **Name**: Enter a descriptive name (e.g., `alexPhone`, `workyLaptop`)
- **Password**: Optional, for encrypting the config file

**Output:**
- Config file created: `~/configs/<name>.conf`
- QR code displayed in terminal (for mobile setup)

### List Clients

```bash
pivpn -c
```

**Example output:**
```
::: Connected Clients List :::
Name           Remote IP        Virtual IP      Last Seen
alexPhone       x.x.x.x         10.155.185.2    2 minutes ago
workyLaptop     x.x.x.x         10.155.185.3    1 hour ago
```

### Show Client Config Path

```bash
pivpn -qr alexPhone
```

**Output:**
- Displays QR code in terminal
- Shows file path: `~/configs/alexPhone.conf`

### Deliver Client Configs to Devices

The config files are available:
```bash
ls ~/configs
# alexPhone.conf  workyLaptop.conf
```

#### Option A — SCP to a Laptop (macOS/Linux)

From the laptop, pull the file:
```bash
scp alexlux58@192.168.4.127:~/configs/workyLaptop.conf .
```

Or push it somewhere:
```bash
scp workyLaptop.conf alexlux58@192.168.4.127:/home/alexlux58/configs/
```

Then import it in the WireGuard app (desktop GUI) as a tunnel config.

#### Option B — QR Code for Phone (Best UX)

On the PiVPN host:
```bash
pivpn -qr alexPhone
```

That prints a QR code in the terminal.

On the phone:
1. Install **WireGuard** app (iOS/Android)
2. **Add tunnel** → **Create from QR code**
3. Scan the QR shown in your SSH terminal
4. No file transfer needed!

### Example WireGuard Client Config

A typical `alexPhone.conf` looks like:

```ini
[Interface]
PrivateKey = <client_private_key>
Address = 10.155.185.2/24
DNS = 192.168.4.127

[Peer]
PublicKey = <server_public_key>
Endpoint = <your_public_ip_or_ddns>:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```

### Notes

- **`DNS = 192.168.4.127`**: When VPN is connected, use Pi-hole for DNS
- **`AllowedIPs = 0.0.0.0/0, ::/0`**: Full-tunnel (all traffic through VPN)

**For split tunnel** (only route LAN traffic through VPN):
```ini
AllowedIPs = 192.168.4.0/22, 10.155.185.0/24
```

This allows:
- LAN traffic (192.168.4.0/22) → VPN
- VPN subnet (10.155.185.0/24) → VPN
- Internet traffic → Direct (bypasses VPN)

---

## Router (Eero) — Port Forward for WireGuard

To connect from outside the house:

### Configure Port Forward

1. **Access eero admin interface**
   - Open eero app or web interface
   - Navigate to **Network Settings** > **Port Forwarding** or **Reservations**

2. **Add port forward**
   - **Name**: WireGuard VPN
   - **Protocol**: UDP
   - **External Port**: 51820
   - **Internal IP**: 192.168.4.127
   - **Internal Port**: 51820
   - **Save**

### Test from Mobile Data

1. **Disable Wi-Fi** on the phone
2. **Connect WireGuard** tunnel
3. **Test connectivity**:
   ```bash
   # From phone (if terminal access is available)
   ping 192.168.4.127
   
   # Or access a LAN-only service (e.g., Pi-hole web UI)
   # Open browser: http://192.168.4.127/admin
   ```

### Dynamic DNS (Optional)

If the public IP changes, consider setting up Dynamic DNS:

1. **Services**: DuckDNS, No-IP, or your domain registrar
2. **Update client config**: Replace `<your_public_ip>` with `<your-ddns-hostname>`
3. **PiVPN can integrate with DDNS**: Check `pivpn -h` for options

---

## Validation Checklist (Copy/Paste)

### On the PiVPN Host

```bash
# Check network interfaces
ip a | egrep 'eth0|wg0' -A2

# Verify WireGuard
sudo wg show

# Check Pi-hole status
pihole status

# Verify listening ports
sudo ss -lntup | egrep ':53|:80|:51820'
```

**Expected:**
- `eth0` has the stable LAN IP (192.168.4.127/22)
- `wg0` is up with 10.155.185.1/24
- Port 53 and 80 listening (Pi-hole)
- Port 51820 listening (WireGuard)

### From a LAN Client

```bash
# Test Pi-hole DNS resolution
nslookup pi.hole 192.168.4.127

# Test blocked domain
nslookup doubleclick.net 192.168.4.127
```

**Expected:**
- `pi.hole` resolves to 192.168.4.127
- `doubleclick.net` returns 0.0.0.0 or NXDOMAIN

### From a VPN Client (When Away from Home)

1. **Connect WireGuard**
2. **Hit a LAN-only service** (example: Grafana, NAS UI, Pi-hole admin)
3. **Confirm VPN IP**:
   - **On client**: Check tunnel "Address" (should be 10.155.185.x)
   - **On server**: `sudo wg show` (should show connected peer)

**Example `wg show` snippet:**
```
peer: <client_pubkey>
  endpoint: <client_public_ip>:54321
  allowed ips: 10.155.185.2/32
  latest handshake: 29 seconds ago
  transfer: 12.34 MiB received, 45.67 MiB sent
```

---

## Troubleshooting Playbook

### A) Pi-hole Not Blocking Anything

**Confirm client DNS is actually Pi-hole:**
```bash
# On client device
nslookup example.com
# Should show: Server: 192.168.4.127
```

**Tail live log:**
```bash
sudo pihole -t
```

If you see no queries, the client is bypassing Pi-hole.

**Common causes:**
- Router still using default DNS
- Device using hardcoded DNS (8.8.8.8)
- Device using DoH (DNS-over-HTTPS)

### B) Some Devices Bypass Pi-hole

**Common causes:**
- eero forces a secondary DNS (clients choose fastest)
- Devices use DoH (DNS-over-HTTPS)
- Devices have hardcoded DNS

**Fix path:**
1. Run Pi-hole as the only DNS option (or deploy a second Pi-hole)
2. Block DoH at firewall/router later (advanced)
3. Check device network settings for hardcoded DNS

### C) WireGuard Connects But Can't Reach LAN

**Check server forwarding + NAT:**
```bash
# Verify IP forwarding
sysctl net.ipv4.ip_forward
# Should show: net.ipv4.ip_forward = 1

# Check NAT rules (if any)
sudo iptables -t nat -S | head -n 50
```

**Confirm AllowedIPs is correct on client:**
- For full tunnel: `AllowedIPs = 0.0.0.0/0, ::/0`
- For split tunnel: `AllowedIPs = 192.168.4.0/22, 10.155.185.0/24`

**Check firewall rules:**
```bash
# On PiVPN host
sudo iptables -L -n -v
```

### D) WireGuard Won't Connect from Outside

**Verify port forward UDP 51820:**
- Check eero port forwarding settings
- Test with `nc -u -v <your_public_ip> 51820` (from outside)

**Verify public IP / DDNS matches:**
- Check client config `Endpoint` matches the public IP
- If using DDNS, verify it resolves correctly

**Confirm server is listening:**
```bash
sudo ss -lunp | grep 51820
```

**Expected:**
```
UNCONN 0 0 0.0.0.0:51820 0.0.0.0:* users:(("wg",pid=1234,fd=3))
```

### E) Pi-hole Web UI Not Accessible

**Check service status:**
```bash
sudo systemctl status lighttpd
pihole status
```

**Check firewall:**
```bash
sudo iptables -L -n | grep 80
```

**Verify listening:**
```bash
sudo ss -lntup | grep ':80'
```

**Access from LAN:**
- URL: `http://192.168.4.127/admin`
- Or: `http://pi.hole/admin`

### F) VPN Client Can't Resolve DNS

**If using Pi-hole DNS in VPN config:**
- Verify Pi-hole is accessible from VPN subnet
- Check Pi-hole firewall rules allow VPN subnet
- Test: `nslookup example.com 192.168.4.127` from VPN client

**Alternative:** Use public DNS in VPN config:
```ini
DNS = 1.1.1.1
```

---

## Key Skills Demonstrated

### What This Project Shows

1. **Network architecture design:**
   - Designed a clean home network service boundary
   - Implemented LAN DNS filtering without exposing DNS to WAN
   - WireGuard-only exposure, minimal attack surface

2. **Operational expertise:**
   - Can operate and troubleshoot production-style services
   - Validations (`wg show`, `pihole -t`, `ss -lntup`)
   - Log interpretation (blocked vs forwarded vs cached)

3. **Security mindset:**
   - Understanding of secure defaults + operational ergonomics
   - Minimal port exposure (only WireGuard)
   - Proper network isolation (LAN-only DNS)

4. **Automation and tooling:**
   - Used automated installers (PiVPN, Pi-hole)
   - Client provisioning workflows (QR codes, config files)
   - DHCP reservation for stability

### Nice Enhancements (Optional)

**Add Unbound for local recursive DNS:**
- Reduces dependency on upstream DNS
- Improves privacy (no DNS queries to third parties)
- Faster response times (local caching)

**Run a second Pi-hole for redundancy:**
- High availability for DNS service
- Load distribution
- Failover capability

**Export metrics (Pi-hole + WireGuard) into Grafana/Prometheus:**
- Visualize DNS query patterns
- Monitor VPN connection statistics
- Alert on anomalies

**Integrate with Security Onion:**
- Monitor DNS queries for security events
- Correlate VPN connections with network activity
- Enhanced threat detection

---

## Quick "Copy/Paste" Build Script (High Level)

```bash
#!/bin/bash
# Update host
sudo apt-get update && sudo apt-get -y upgrade

# Install PiVPN (WireGuard)
curl -L https://install.pivpn.io | bash

# Install Pi-hole
curl -sSL https://install.pi-hole.net | bash

# Validate services
sudo wg show
pihole status
sudo ss -lntup | egrep ':53|:80|:51820'

# Add a VPN client + show QR
pivpn add
pivpn -qr <clientName>
```

**Note:** This is a high-level script. The interactive installers will prompt you for configuration choices. See the detailed steps above for recommended settings.

---

## Key Takeaways

### Critical Success Factors

1. **DHCP Reservation**: Ensures stable IP for Pi-hole and VPN endpoint
2. **Port Forwarding**: Only WireGuard (UDP 51820) exposed to WAN
3. **DNS Configuration**: Router must point LAN clients to Pi-hole
4. **IP Forwarding**: Required for VPN clients to reach LAN resources
5. **Client Provisioning**: QR codes make mobile setup trivial

### Common Pitfalls

1. **Dynamic IP**: Without DHCP reservation, IP changes break DNS/VPN
2. **Wrong DNS in Router**: Clients bypass Pi-hole if router uses default DNS
3. **Firewall Blocking**: VPN can't reach LAN if firewall rules are too restrictive
4. **Port Forward Wrong Protocol**: WireGuard is UDP, not TCP
5. **DoH Bypassing Pi-hole**: Some devices use DNS-over-HTTPS, bypassing Pi-hole

### Performance Considerations

- **RAM**: 1GB minimum, 2GB+ recommended for Pi-hole + WireGuard
- **Storage**: 8GB+ for OS and logs
- **Network**: Gigabit Ethernet recommended for low latency
- **CPU**: Any modern Raspberry Pi or VM is sufficient

---

## Conclusion

This project deployed a combined Pi-hole + PiVPN WireGuard stack on a single Linux host, providing network-wide ad blocking and secure remote access. The system provides production-ready network services with secure defaults and operational tooling.

**Final Status:**
- ✅ Pi-hole filtering LAN DNS queries
- ✅ WireGuard VPN providing remote access
- ✅ Minimal attack surface (only WireGuard exposed)
- ✅ Easy client provisioning (QR codes)
- ✅ Stable IP addressing (DHCP reservation)

**Next Steps:**
- Monitor Pi-hole logs for blocked domains
- Add custom blocklists for enhanced filtering
- Set up Dynamic DNS for VPN endpoint
- Integrate metrics with monitoring stack
- Deploy second Pi-hole for redundancy

---

## Related Projects

This project integrates with other lab services:

- **Security Onion**: Can monitor DNS queries from Pi-hole and correlate with network traffic
- **Network Observability Stack**: Export Pi-hole metrics to Prometheus for visualization in Grafana
- **DevOps Tools**: Automate Pi-hole blocklist updates via Ansible playbooks
- **NSOT (NetBox/Nautobot)**: Document PiVPN host IP and services in network documentation

For complete lab documentation, see `README.md` in the HomeLabWriteup directory.

---

**End of Lab Writeup**

