# 📡 AREDN Mesh Integration with UniFi UDR-7

_Last updated: Feb 2026_

This document is a **complete reference** for integrating an **AREDN
mesh node** with a **UniFi Dream Router (UDR-7)**, including:

- Physical and logical **network topology**
- Rationale behind the topology design
- Routing, NAT, and firewall behavior
- DNS mirroring strategy (not forwarding)
- Persistence across reboot using **systemd**
- What is logged and why

This reflects the **final, stable configuration** in use.

---

## 1. Introduction

AREDN (Amateur Radio Emergency Data Network) enables licensed amateur
radio operators to interconnect private networks over RF links and VPN
tunnels, forming a distributed IP-based mesh network.

Key characteristics of AREDN:

- **No NAT inside the mesh** -- Nodes receive routable 10.x.x.x
  addresses derived from callsigns.
- **Layer 3 dynamic routing (OLSR-based)** -- Routes are automatically
  learned and adjusted.
- **Distributed hostname propagation** -- Services are advertised and
  discoverable across the mesh via `.local.mesh` hostnames.

Unlike traditional home networking, AREDN behaves like a routed backbone
rather than an ISP-style WAN.

---

## 2. Typical Home Deployment

In a simple setup:

    Home LAN → AREDN Node → RF / VPN → Mesh

The AREDN node connects directly to a basic home router.\
Segmentation is minimal, routing is simple, and the LAN effectively
behaves as a client of the mesh.

This model works well for uncomplicated networks.

However, You can only access the mesh network using a client of an AREDN node.
Although, you can use port forwarding and other mechanisms to make some services
exposed to your home network (the WAN side of the AREDN network), AREDN networks
maintain their own dynamic DNS system in authorative way, and DNS by design is
not forwarded or exposed to the WAN side.

---

## 3. My Network Environment

My environment is significantly more complex.

A UniFi router serves as the authoritative network controller, managing
multiple VLANs:

- Private LAN
- Work network
- Dedicated VPN network
- IoT network
- Additional segmented environments

UniFi handles:

- VLAN isolation\
- Firewall policies\
- Static routing\
- DNS control\
- Monitoring and observability

This segmentation and centralized control are intentional and required.

---

## 4. The Core Architectural Conflict

Integrating AREDN into this environment creates design tension.

### Option 1 -- Place LAN behind AREDN

This would make my internal network a client of the mesh and bypass
UniFi's centralized routing and segmentation model.

This also means that I have to throttle my internet connection to the maximum
connection speed available to the hardware of the AREDN node. That is 1Gbps
in some hardware, and 100Mbps on most others. I need at least 2.5Gbps.

### Option 2 -- Treat AREDN as a WAN on UniFi

This does not work conceptually or technically. AREDN is not an upstream
ISP --- it is a routed mesh with its own addressing and dynamic routing.

### Option 3 -- Fully isolate AREDN

This prevents seamless access to mesh services from controlled VLANs and
creates unnecessary friction.

---

## 5. The Actual Problem

The issue is not connectivity.

The issue is architectural alignment.

I need:

- Full participation in the AREDN mesh\
- Access to mesh-wide services\
- Clean routing without NAT conflicts\
- Preservation of UniFi VLAN segmentation\
- Centralized firewall and policy control\
- The high speed connections that are NOT accessible to AREDN nodes

The challenge is designing an integration model where:

- AREDN remains a proper mesh node\
- UniFi remains the authoritative internal router\
- Neither system is reduced to a workaround

The rest of this document describes how that balance was achieved.

---

## 6. Philosophy of the Solution

### 6.1 Design Principles

This solution is built around a few core principles:

- Clear separation of responsibilities\
- Deterministic routing\
- Centralized policy control\
- Controlled boundary between routing domains\
- Automation over manual maintenance

The goal was not just connectivity --- it was architectural alignment
between AREDN and UniFi.

### 6.2 Role Separation

The architecture assigns strict roles:

**AREDN Node** - Participates fully in the mesh - Handles RF/VPN mesh
routing - Advertises and consumes mesh services

**UniFi (UDR)** - Remains the authoritative router for all internal
VLANs - Enforces firewall and segmentation policies - Controls DNS and
routing decisions

Neither system is reduced to a workaround.\
Each operates within its intended design scope.

### 6.3 Dual-Network Integration Model

The integration introduces two dedicated networks on the UniFi router:

#### `aredn-lan` (Mesh-Facing LAN)

A dedicated internal network where:

- UniFi participates as a client of the AREDN mesh
- Mesh routing is explicitly handled
- The mesh domain is logically contained

This allows UniFi to understand and route to the mesh --- but only
within this controlled segment.

#### `aredn-wan` (AREDN Uplink Network)

A separate network used exclusively for the AREDN node uplink:

- The AREDN node connects to UniFi as a WAN-side client
- From UniFi's perspective, the AREDN node is simply another routed
  upstream network
- Routing between UniFi and AREDN is explicit and controlled

Separating uplink and mesh participation avoids ambiguity and keeps
traffic direction predictable.

### 6.4 Controlled Access via NAT Policy

To allow my segmented internal VLANs (private, work, IoT, VPN, etc.) to
access the AREDN mesh, a NAT policy is applied on UniFi.

This ensures:

- Internal VLANs are NATed when accessing the mesh
- The AREDN mesh sees only **one client**: the UniFi router
- No internal subnet is directly exposed
- Mesh nodes cannot traverse back into private segments

From the mesh's perspective, my entire environment appears as a single
routed endpoint.

This preserves outbound access while preventing unintended inbound
exposure.

### 6.5 DNS and Automation

Mesh hostnames are synchronized into the UniFi environment in a
controlled way using:

- Scripted mesh data retrieval
- Systemd services and timers
- Controlled publishing into dnsmasq
- Logging for observability

This ensures:

- Reliable hostname resolution
- Automatic updates when the mesh changes
- Recovery after reboots
- Long-term maintainability

### 6.6 🧭 Topology Overview

#### Physical / Logical Layout

```
    [ ISP Modem ]
          |
          |  (Public IP)
          v
    [ UniFi UDR-7 ]
          |
          |-- VLAN 1 (unifi-management) 172.20.1.0/24
          |
          |-- VLAN 10 (Personal) 172.21.10.0/24
          |
          |-- VLAN 11 (Personal-VPN) 172.21.11.0/24
          |
          |-- VLAN 20 (Work) 172.21.20.0/24
          |
          |-- VLAN 30 (IoT) 172.21.30.0/24
          |
          |-- VLAN 31 (Cameras) 172.21.31.0/24
          |
          |-- VLAN 40 (aredn-wan) 172.21.40.0/29
          |      |
          |      +--> AREDN Node WAN Port                   <---+
          |                                                     |
          |-- VLAN 41 (aredn-lan) 10.6.229.8/29           [ AREDN NODE ]
          |      |                                              |
          |      +--> AREDN Node LAN Port (10.6.229.9)      <---+
          |      |      |
          |      |      +--> AREDN Mesh (10.0.0.0/8)
          |      |
          |      +--> Mesh phone and other services
          |
          |-- VLAN 200 (guest) 172.21.200.0/24
```

### 6.7 Philosophy Summary

This solution respects the design philosophy of both systems:

- AREDN remains a decentralized, NAT-free mesh
- UniFi remains a segmented, policy-driven router

The boundary between them is intentional and controlled.

The result:

- Full mesh participation\
- Preserved VLAN isolation\
- Explicit routing\
- Controlled exposure\
- Operational stability

The guiding principle is simple:

> Integrate systems by respecting their design --- not by bending one to
> fit the other.

This reflects the **final, stable configuration** in use.

---

## 7 Implenetation

### 7.1 🌐 UniFi Network Configuration

#### 7.1.1 VLANs

| VLAN | Purpose   | Subnet         |
| ---- | --------- | -------------- |
| 40   | AREDN WAN | 172.21.40.0/29 |
| 41   | AREDN LAN | 10.6.229.8/29  |

AREDN LAN IP: `10.6.229.9`  
UDR VLAN 41 IP: `10.6.229.10`

#### 7.1.2 Routing

Route all mesh traffic via:

```
Destination: 10.0.0.0/8
Next Hop: 10.6.229.9
```

Why: - AREDN mesh uses large dynamic address space - Static route avoids
per-subnet churn - Keeps routing simple and deterministic

#### 7.1.3 NAT

Outbound NAT rule:

- Source: LAN networks
- Destination: 10.0.0.0/8
- Out Interface: VLAN 41

Purpose:

- Prevents home LAN routes from leaking into mesh
- Avoids asymmetric return traffic
- Enforces clean L3 boundary

#### 7.1.4 Firewall

Rules implemented:

- **Personal / VPN → AREDN**: Allow
- **AREDN → Personal / VPN (return)**: Allow
- **AREDN → Internal (unsolicited)**: Blocked by default

DNS is _not_ forwarded --- only routed traffic is allowed.

### 7.2 🔑 SSH Access to AREDN

- Port: **2222**
- Authentication: **Key-based only**
- Key stored on UDR:

#### 7.2.1 Generate the SSH Key (on the UDR)

Generate a dedicated keypair for this integration so it can be rotated
independently of any admin keys.

```bash
mkdir -p /data/aredn
ssh-keygen -t ed25519 -a 64 -f /data/aredn/id_ed25519 -C "udr-aredn-dns"
```

Notes:

- The `-a 64` option strengthens key derivation for the private key.
- This key is **read-only** in practice because the AREDN user will be
  restricted (see upload section below).

#### 7.2.2 Upload the Public Key via AREDN UI

1. Log into the AREDN node UI.
2. Navigate to **Administration → SSH Keys** (may appear as **Services → SSH**
   in some firmware builds).
3. Click **Add** or **Upload** key.
4. Paste the contents of the public key file:

   `/data/aredn/id_ed25519.pub`

5. Save and apply.

Verification (from UDR):

```bash
ssh -p 2222 -i /data/aredn/id_ed25519 -o BatchMode=yes root@10.6.229.9 "uname -a"
```

If this works, the key is correctly installed.

### 🧠 7.3 DNS Strategy

#### 7.3.1 Problem

- AREDN generates dynamic host files:

      /var/run/arednlink/hosts/*

- UniFi dnsmasq cannot read these directly

- DNS forwarding breaks AREDN locality assumptions

- Parent hostnames do not auto-resolve

#### 7.3.2 Solution

- Periodically **pull host data via SSH**
- Mirror into UniFi dnsmasq using `addn-hosts`
- Synthesize base hostnames

Example input (AREDN):

    10.6.229.9 lan.AL0Y-home.local.mesh

Generated output (UDR):

    10.6.229.9 lan.AL0Y-home.local.mesh
    10.6.229.9 AL0Y-home.local.mesh

No queries are forwarded into the mesh.

#### 7.3.3 Why Mirror Instead of Forward

Mirroring ensures:

- **Locality is preserved** (mesh clients still resolve mesh-only names)
- **No dependency on mesh reachability** for home LAN DNS
- **No query leakage** into the mesh from non-mesh networks

Forwarding, by contrast, introduces failure modes where DNS resolution depends
on mesh connectivity and can expose internal resolver behavior to the mesh.

### 📄 7.4 DNS Update Script

**File:** `/data/aredn/update-mesh-dns.sh`

```sh
#!/bin/sh
set -eu

AREDN_IP="10.6.229.9"
AREDN_PORT="2222"
SSH_KEY="/data/aredn/id_ed25519"

# Where we publish the mirrored mesh DNS
OUT_HOSTS="/run/aredn-mesh.hosts"
OUT_CONF="/run/dnsmasq.dhcp.conf.d/99-aredn-mesh.conf"

TMP_HOSTS="/tmp/aredn-mesh.hosts.$$"
TMP_CONF="/tmp/99-aredn-mesh.conf.$$"

cleanup() {
  rm -f "$TMP_HOSTS" "$TMP_CONF" 2>/dev/null || true
}
trap cleanup EXIT INT TERM

# Fetch all mesh hosts from AREDN
MESH_DATA="$(ssh \
  -p "$AREDN_PORT" \
  -i "$SSH_KEY" \
  -o BatchMode=yes \
  -o ConnectTimeout=3 \
  -o StrictHostKeyChecking=no \
  root@"$AREDN_IP" \
  'cat /var/run/arednlink/hosts/* 2>/dev/null' || true)"

# If fetch failed, do not clobber existing config
[ -n "$MESH_DATA" ] || exit 0

# Normalize, dedupe, and add base hostname aliases
echo "$MESH_DATA" \
  | tr '\t' ' ' \
  | sed 's/  */ /g' \
  | awk '
NF>=2 && $2 ~ /\.local\.mesh$/ {
  ip=$1
  name=$2
  print ip " " name

  # If hostname is lan.<node>.local.mesh, also add <node>.local.mesh
  if (name ~ /^lan\./) {
    base=name
    sub(/^lan\./, "", base)
    print ip " " base
  }
}' \
  | sort -u > "$TMP_HOSTS"

# If empty, don't clobber
[ -s "$TMP_HOSTS" ] || exit 0

# Ensure dnsmasq include directory exists (UniFi main dnsmasq uses this conf-dir)
mkdir -p /run/dnsmasq.dhcp.conf.d

# dnsmasq include to load our dynamic hosts
cat > "$TMP_CONF" <<EOF
# Auto-generated from AREDN mesh host list (do not edit)
addn-hosts=$OUT_HOSTS
local=/local.mesh/
EOF

changed=0

# Install hosts atomically ONLY if content changed
if [ ! -f "$OUT_HOSTS" ] || ! cmp -s "$TMP_HOSTS" "$OUT_HOSTS"; then
  mv "$TMP_HOSTS" "$OUT_HOSTS"
  changed=1
else
  rm -f "$TMP_HOSTS"
fi

# Install conf atomically ONLY if content changed
if [ ! -f "$OUT_CONF" ] || ! cmp -s "$TMP_CONF" "$OUT_CONF"; then
  mv "$TMP_CONF" "$OUT_CONF"
  changed=1
else
  rm -f "$TMP_CONF"
fi

# Hot-reload dnsmasq so the new hosts take effect (NO downtime)
# On UniFi, dnsmasq is not managed by dnsmasq.service; use the pid-file/command line instead.
if [ "$changed" -eq 1 ]; then
  # Preferred: UniFi main dnsmasq pid-file
  if [ -r /run/dnsmasq-main.pid ]; then
    DNSMASQ_PID="$(cat /run/dnsmasq-main.pid 2>/dev/null || true)"
    if [ -n "$DNSMASQ_PID" ] && kill -0 "$DNSMASQ_PID" 2>/dev/null; then
      kill -HUP "$DNSMASQ_PID"
      exit 0
    fi
  fi

  # Fallback: HUP the dnsmasq instance that includes our conf-dir
  for pid in $(pgrep -x dnsmasq 2>/dev/null || true); do
    cmd="$(tr '\0' ' ' < /proc/"$pid"/cmdline 2>/dev/null || true)"
    echo "$cmd" | grep -q -- '--conf-dir=/run/dnsmasq.dhcp.conf.d/' || continue
    kill -HUP "$pid"
    exit 0
  done
fi

exit 0
```

---

### 📄 7.5 🔁 Persistence via systemd

Because `/run` is ephemeral, a persistent service is required.

#### 7.5.1 Why a timer (not a loop daemon)

Using a systemd timer provides:

- Clean start/stop control via `systemctl`
- Automatic scheduling and persistence across reboots
- Separation of **logic** (script) from **scheduling** (timer)

The standalone daemon script below is retained as an alternative option, but
the timer is the primary, recommended approach.

#### 7.5.2 Daemon Script

**File:** `/mnt/data/aredn/mesh-dns-daemon.sh`

```sh
#!/bin/sh
set -eu

# Wait a bit for VLAN bridges/routes to be ready
sleep 60

# Loop forever
while true; do
    if /data/aredn/update-mesh-dns.sh; then
        echo "$(date) updated" >> /mnt/data/aredn/mesh-dns.log
    else
        echo "$(date) update failed" >> /mnt/data/aredn/mesh-dns.log
    fi
    sleep 60
done
```

#### 7.5.3 systemd Unit

**File:** `/etc/systemd/system/aredn-mesh-dns.service`

```ini
[Unit]
Description=AREDN Mesh DNS Mirror (dnsmasq addn-hosts)
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/data/aredn/update-mesh-dns.sh
StandardOutput=journal
StandardError=journal
```

**File:** `/etc/systemd/system/aredn-mesh-dns.timer`

```ini
[Unit]
Description=Run AREDN Mesh DNS Mirror periodically

[Timer]
Unit=aredn-mesh-dns.service
OnCalendar=*:0/2
AccuracySec=15s
Persistent=true

[Install]
WantedBy=timers.target
```

### 7.6 📋 Logging

Log file: **File:** `/mnt/data/aredn/mesh-dns.log`

Contents: - Timestamp of each update attempt - Success or failure
indicator

Example:

    Sun Jan 25 20:41:00 UTC 2026 updated
    Sun Jan 25 20:42:00 UTC 2026 updated

Purpose: - Verifies boot persistence - Confirms loop execution - Aids
troubleshooting without syslog

---

## 8 ✅ Validation

```bash
nslookup AL0Y-home.local.mesh 172.21.10.1
nslookup lan.AL0Y-home.local.mesh 172.21.10.1
```

Expected:

    Address: 10.6.229.9

---

## 9 🔄 Full Rebuild / Recreate Checklist

Use this section if you ever want to redo the setup from scratch.

### 9.1 UniFi VLANs (Network Application)

Create the two VLAN-only networks:

- **aredn-wan** — VLAN **40**, subnet **172.21.40.0/29**
- **aredn-lan** — VLAN **41**, subnet **10.6.229.8/29**

Assign the UDR IP on aredn-lan to **10.6.229.10**.

### 9.2 Physical Wiring

- UDR LAN port(s) carry VLAN 40 and VLAN 41 to the switch (tagged).
- AREDN node WAN port connects to the VLAN 40 access/untagged port.
- AREDN node LAN port connects to the VLAN 41 access/untagged port.

### 9.3 AREDN Node IPs

- **WAN**: DHCP from VLAN 40
- **LAN**: Static **10.6.229.9/29** (or ensure it is assigned consistently)

### 9.4 UDR Static Route

Add a route:

- Destination: **10.0.0.0/8**
- Next hop: **10.6.229.9**

### 9.5 UDR Source NAT Rule

Create a rule (LAN/VPN → AREDN):

- Source: Any UniFi LAN/VLAN
- Destination: **10.0.0.0/8**
- Outbound Interface: **aredn-lan**

### 9.6 UDR Firewall Rules

- Allow **Personal/VPN → AREDN**
- Allow **AREDN → Personal/VPN (return traffic)**
- Block **AREDN → Internal (unsolicited)**

### 9.7 SSH Key

- Generate the key on the UDR (see SSH section above)
- Upload the public key to AREDN UI

### 9.8 Place Scripts

- `/data/aredn/update-mesh-dns.sh` (DNS mirror script)
- `/mnt/data/aredn/mesh-dns-daemon.sh` (optional)

### 9.9 Install systemd Units

Create:

- `/etc/systemd/system/aredn-mesh-dns.service`
- `/etc/systemd/system/aredn-mesh-dns.timer`

Then enable:

```bash
systemctl daemon-reload
systemctl enable --now aredn-mesh-dns.timer
```

### 9.10 Validate

```bash
nslookup AL0Y-home.local.mesh 172.21.10.1
nslookup lan.AL0Y-home.local.mesh 172.21.10.1
```

Expected answer: **10.6.229.9**
