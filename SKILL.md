---
name: routeros
description: Use when working with MikroTik RouterOS configuration, CLI commands, REST API, firewall rules, networking setup, VPN, wireless, bridging, routing protocols, MPLS, MLAG, containers, hotspot/captive portal, scripting, or building tools that interact with RouterOS devices. Covers RouterOS v7 CLI syntax, REST API (including auth, ETags, filtering), classic socket API, security hardening, certificates/HTTPS, and best practices.
---

# MikroTik RouterOS

RouterOS is a Linux-based network operating system that powers MikroTik hardware devices and is also available as a virtual machine (Cloud Hosted Router / CHR). RouterOS v7 is the current major version (latest stable branch: v7.21.x), using Linux kernel 5.6.3. It provides routing, firewall, bridging, VPN, wireless, QoS, container workloads, and monitoring capabilities configurable via CLI, WinBox GUI, WebFig, REST API, and a socket-based API.

## When to Use This Skill

**Activate when:**
- Configuring or automating MikroTik RouterOS devices
- Writing CLI commands or scripts for RouterOS
- Building tools that communicate with RouterOS via REST API or classic API
- Designing firewall rules, NAT, routing, VPN, or wireless configurations
- Implementing security hardening on RouterOS devices
- Developing tools or MCP servers that wrap RouterOS functionality

**Do not use when:**
- Working with MikroTik SwOS (separate OS for certain switch models)
- The task is purely about Go language features unrelated to RouterOS

## Documentation Sources

Always consult these authoritative sources for RouterOS information:

> **URL stability warning**: The documentation runs on Atlassian Confluence. URLs embed a numeric page ID (e.g. `/pages/119144601/`) followed by a human-readable slug. The slug is cosmetic — only the ID matters for resolution. MikroTik has reorganized pages in the past, causing IDs to silently resolve to wrong pages or return 404. If a URL below seems to point to the wrong content, look up the current ID via the Confluence child-page API:
> ```
> https://help.mikrotik.com/docs/rest/api/content/328059/child/page?limit=50
> ```
> This returns the live page tree under the RouterOS root (ID `328059`) as JSON, with each entry's `id` and `title`. Use the correct `id` to construct the updated URL as `/spaces/ROS/pages/<id>/<Title+With+Pluses>`.

| Source | URL | Use For |
|---|---|---|
| Official Docs | https://help.mikrotik.com/docs/ | Definitive reference for all features |
| Community Forum | https://forum.mikrotik.com/ | Real-world examples, troubleshooting, edge cases |
| First Time Config | https://help.mikrotik.com/docs/spaces/ROS/pages/328151/First+Time+Configuration | Initial setup guide and best practices |
| Securing Router | https://help.mikrotik.com/docs/spaces/RKB/pages/8323164/Securing+your+router | Security hardening checklist |
| REST API | https://help.mikrotik.com/docs/spaces/ROS/pages/47579162/REST+API | Programmatic access via HTTP/JSON |
| Classic API | https://help.mikrotik.com/docs/spaces/ROS/pages/47579160/API | Socket-based binary protocol |
| Firewall | https://help.mikrotik.com/docs/spaces/ROS/pages/119144601/Firewall+and+Quality+of+Service | Firewall rules, NAT, mangle, QoS |
| Routing | https://help.mikrotik.com/docs/spaces/ROS/pages/328222/Unicast+Routing+Protocols | OSPF, BGP, RIP, static routing |
| VPN | https://help.mikrotik.com/docs/spaces/ROS/pages/119144597/Virtual+Private+Networks | WireGuard, IPsec, L2TP, OpenVPN |
| Scripting | https://help.mikrotik.com/docs/spaces/ROS/pages/328228/Scripting | Built-in scripting language reference |
| Certificates | https://help.mikrotik.com/docs/spaces/ROS/pages/7962650/Certificates | TLS cert management, HTTPS setup |
| Container | https://help.mikrotik.com/docs/spaces/ROS/pages/84901929/Container | OCI container runtime |
| Hotspot | https://help.mikrotik.com/docs/spaces/ROS/pages/1409115/HotSpot | Captive portal / guest access |
| MLAG | https://help.mikrotik.com/docs/spaces/ROS/pages/196345860/MLAG | Multi-chassis link aggregation |
| MPLS | https://help.mikrotik.com/docs/spaces/ROS/pages/328144/MPLS | Label switching and LDP |

## RouterOS CLI Structure

RouterOS uses a hierarchical command structure organized into menu paths. In v7, paths use `/` as separator.

### Menu Path Syntax

```
/ip/address
/ip/firewall/filter
/interface/bridge
/system/identity
/routing/ospf/instance
```

### CRUD Operations

Every menu level supports a standard set of commands:

| Command | Description | Example |
|---|---|---|
| `add` | Create a new item | `/ip/address add address=192.168.1.1/24 interface=ether2` |
| `set` | Modify existing item | `/ip/address set 0 address=192.168.1.2/24` |
| `remove` | Delete an item | `/ip/address remove 0` |
| `print` | List items | `/ip/address print` |
| `enable` | Enable a disabled item | `/interface enable ether3` |
| `disable` | Disable an item | `/interface disable ether3` |
| `move` | Reorder items (firewall) | `/ip/firewall/filter move 3 destination=1` |
| `export` | Export config as text | `/ip/firewall export` |
| `find` | Find items matching criteria | `/interface find type=ether` |

### Item References

Items can be referenced by:
- **Number** (positional index): `set 0 disabled=yes`
- **Internal ID**: `set *1A disabled=yes`
- **Name** (for named items): `set ether1 disabled=yes`
- **Find expression**: `set [find name=ether1] disabled=yes`

### Print Modifiers

```
print detail          -- show all properties
print brief           -- show summary
print value-list      -- machine-parseable output
print as-value        -- returns an array of records usable in scripts (e.g. [/ip/address get 0 as-value])
print count-only      -- just the count
print where disabled=yes  -- filter results
print stats           -- show statistics
```

### Scripting Basics

RouterOS has a built-in scripting language used in `/system/script` and `/system/scheduler`:

```
# Declare a local variable, then assign/reassign with :set
:local myVar "hello"
:if ($myVar = "hello") do={
    :set myVar "world"
    :put $myVar
}
:foreach i in=[/interface find] do={
    :put [/interface get $i name]
}
```

> **`:local` vs `:set`**: `:local` declares a variable (must come first, at the top of scope). `:set` assigns or updates its value. Using `:set` on an undeclared variable at script scope is an error; always declare with `:local` first.

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328228/Scripting

### Scripting — Error Handling

Use `:do`/`:on-error` to catch failures without aborting the script:

```
:do {
    /ip/address add address=192.168.1.1/24 interface=ether2
} on-error={
    :put "Failed to add address"
}
```

### Scripting — Dynamic Evaluation

`[:parse]` compiles a string into executable code at runtime, enabling dynamic command generation:

```
:local cmd "/ip/address print"
[:parse $cmd]
```

### Scripting — Environment Variables & Script Arguments

Scripts receive arguments via the `$1`, `$2`, ... positional parameters when called via `/system/script/run` with arguments, or from the scheduler with parameters set in the `parameters` field.

Access global environment variables with `:global`:

```
:global myGlobal
:set myGlobal "shared-value"
```

Read a global set by another script or the scheduler:

```
:global counter
:if ([:typeof $counter] = "nothing") do={ :set counter 0 }
:set counter ($counter + 1)
:put "Run count: $counter"
```

Pass parameters from CLI:

```
/system/script/run scriptName parameters="arg1 arg2"
```

Inside the script, read them as `$1` and `$2`. For named parameters, use global variables as a convention.

### Scripting — Common Built-ins

| Function | Description |
|---|---|
| `:put <value>` | Print to console / log |
| `:log <topic> <message>` | Write to system log (topics: `info`, `warning`, `error`, `debug`) |
| `[:len $var]` | Length of string or array |
| `[:tostr $val]` | Convert value to string |
| `[:tonum $val]` | Convert string to number |
| `[:typeof $var]` | Type of value (`str`, `num`, `bool`, `array`, `nothing`) |
| `[:time { ... }]` | Measure execution time of a block |
| `[:find $str "sub"]` | Find substring position |
| `[:pick $str 0 3]` | Substring extraction |
| `/delay 1s` | Pause execution |

## REST API Reference

The REST API (available since RouterOS v7.1beta4) is a JSON wrapper over the console API. It is the primary programmatic interface for external tools.

### Prerequisites

The `www-ssl` service (HTTPS) must be enabled, or `www` service (HTTP, v7.9+) for testing only.

To generate a self-signed certificate and enable HTTPS (required for production REST API use):

```
# Generate a self-signed CA and server certificate
/certificate add name=ca-cert common-name=ca key-usage=key-cert-sign,crl-sign
/certificate sign ca-cert
/certificate add name=router-cert common-name=10.0.0.1 \
    subject-alt-name=IP:10.0.0.1
/certificate sign router-cert ca=ca-cert

# Bind the certificate to www-ssl and enable it
/ip/service set www-ssl certificate=router-cert disabled=no

# Optionally disable plain HTTP
/ip/service set www disabled=yes
```

> **REST API clients**: Pass `-k` (curl) or disable certificate verification in your HTTP client when using a self-signed cert. For production, use a cert signed by a trusted CA (e.g. Let's Encrypt via the ACME package).

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/7962650/Certificates

### Base URL

```
https://<router_ip>/rest
```

### Authentication

HTTP Basic Auth using RouterOS user credentials.

For session-based auth (avoids sending credentials on every request), use the login endpoint:

```bash
# Obtain session cookie
curl -k -c cookies.txt -X POST \
  https://10.0.0.1/rest/login \
  -H "content-type: application/json" \
  -d '{"username":"admin","password":"secret"}'

# Use session cookie on subsequent requests
curl -k -b cookies.txt https://10.0.0.1/rest/ip/address
```

The session cookie is named `_api_session`. Logout to invalidate it:

```bash
curl -k -b cookies.txt -X GET https://10.0.0.1/rest/logout
```

### HTTP Methods

| HTTP Method | RouterOS Command | Description | URL Pattern |
|---|---|---|---|
| `GET` | `print` | List/read records | `GET /rest/ip/address` |
| `PUT` | `add` | Create a new record | `PUT /rest/ip/address` |
| `PATCH` | `set` | Update a record | `PATCH /rest/ip/address/*1` |
| `DELETE` | `remove` | Delete a record | `DELETE /rest/ip/address/*1` |
| `POST` | any command | Universal command execution | `POST /rest/ip/firewall/filter/move` |

### GET Examples

```bash
# List all IP addresses
curl -k -u admin:password https://10.0.0.1/rest/ip/address

# Get a single record by internal ID
curl -k -u admin:password https://10.0.0.1/rest/ip/address/*1

# Get interface by name
curl -k -u admin:password https://10.0.0.1/rest/interface/ether1

# Filter results
curl -k -u admin:password "https://10.0.0.1/rest/ip/address?network=192.168.88.0&dynamic=false"

# Select specific properties
curl -k -u admin:password "https://10.0.0.1/rest/ip/address?.proplist=address,interface,disabled"
```

### PUT Example (Create)

```bash
curl -k -u admin:password -X PUT \
  https://10.0.0.1/rest/ip/address \
  -H "content-type: application/json" \
  -d '{"address":"192.168.100.1/24","interface":"ether3"}'
```

### PATCH Example (Update)

```bash
curl -k -u admin:password -X PATCH \
  https://10.0.0.1/rest/ip/address/*3 \
  -H "content-type: application/json" \
  -d '{"comment":"management network"}'
```

### DELETE Example

```bash
curl -k -u admin:password -X DELETE \
  https://10.0.0.1/rest/ip/address/*3
```

### POST Example (Universal Command)

```bash
# Ping a host (must use count to avoid timeout)
curl -k -u admin:password -X POST \
  https://10.0.0.1/rest/ping \
  -H "content-type: application/json" \
  -d '{"address":"8.8.8.8","count":"4"}'

# Run a script
curl -k -u admin:password -X POST \
  https://10.0.0.1/rest/system/script/run \
  -H "content-type: application/json" \
  -d '{".id":"*1"}'

# Export configuration
curl -k -u admin:password -X POST \
  https://10.0.0.1/rest/export \
  -H "content-type: application/json" \
  -d '{}'
```

### POST Filtering with .query

```bash
# Get ethernet and VLAN interfaces using query stack
curl -k -u admin:password -X POST \
  https://10.0.0.1/rest/interface/print \
  -H "content-type: application/json" \
  -d '{".query":["type=ether","type=vlan","#|"]}'
```

### Key Constraints

- **Timeout**: Commands that run indefinitely timeout after 60 seconds. Always use limiting parameters (e.g., `"count":"4"` for ping, `"duration":"2s"` for bandwidth-test, `"once":""` for monitor commands).
- **No streaming**: The REST API cannot run continuous commands like `monitor`. Use `"once":""` to get a single snapshot.
- **String values**: All JSON response values are encoded as strings, even numbers and booleans.
- **Single creation**: Only one resource can be created per PUT request.
- **Error format**: `{"error": <http_code>, "message": "<description>", "detail": "<optional>"}`

### Optimistic Concurrency (ETag / If-Match, v7.7+)

GET requests return an `ETag` header containing a hash of the resource state. Use it with `If-Match` on PATCH/DELETE to prevent overwriting concurrent changes:

```bash
# Fetch resource and capture ETag
ETAG=$(curl -k -u admin:password -I https://10.0.0.1/rest/ip/address/*3 \
  | grep -i etag | awk '{print $2}' | tr -d '\r')

# Update only if resource hasn't changed
curl -k -u admin:password -X PATCH \
  https://10.0.0.1/rest/ip/address/*3 \
  -H "content-type: application/json" \
  -H "If-Match: $ETAG" \
  -d '{"comment":"updated"}'
```

If the resource was modified since the ETag was issued, the server returns `412 Precondition Failed`.

## Classic API (Socket-Based)

The classic API is a binary TCP protocol for persistent, streaming connections. It predates the REST API.

| Property | Value |
|---|---|
| Plain port | TCP 8728 |
| SSL port | TCP 8729 |
| Authentication | Custom login sentence (plain text over TCP; use SSL) |
| Data format | Binary length-prefixed words |
| Streaming | Supported via `listen` command |
| Concurrent commands | Supported via `.tag` mechanism |

The classic API is useful when:
- Real-time monitoring/streaming is needed (the REST API cannot do this)
- Persistent connections with event-based updates are required
- Working with RouterOS versions before v7.1

**Login sequence** (MD5 challenge-response):
1. Connect to TCP 8728 (plain) or 8729 (SSL)
2. Send a `/login` sentence — router replies with a `=ret=<challenge>` word
3. Hash the password: `MD5(null_byte + password + unhex(challenge))`, encode as lowercase hex
4. Send a second `/login` sentence with `=name=<user>` and `=response=00<hash>`
5. Router replies with an empty `!done` sentence on success

Most users should use an existing library rather than implementing this directly.

Go client library: `github.com/go-routeros/routeros/v3`

## RouterOS Feature Areas

### IP Addressing & DHCP

```
/ip/address                  -- manage IP addresses on interfaces
/ip/dhcp-client              -- DHCP client configuration
/ip/dhcp-server              -- DHCP server, leases, networks, options
/ip/pool                     -- IP address pools
/ip/dns                      -- DNS settings and static entries
/ipv6/address                -- IPv6 addressing
/ipv6/dhcp-client            -- DHCPv6 client
/ipv6/dhcp-server            -- DHCPv6 server
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/119144613/Network+Management

### Firewall

```
/ip/firewall/filter          -- packet filter rules (input, forward, output chains)
/ip/firewall/nat             -- source NAT (masquerade) and destination NAT (port forwarding)
/ip/firewall/mangle          -- packet marking for QoS and policy routing
/ip/firewall/raw             -- bypass connection tracking (DDoS protection)
/ip/firewall/address-list    -- dynamic and static address lists
/ip/firewall/connection      -- active connection tracking table
/ipv6/firewall/filter        -- IPv6 filter rules
/ipv6/firewall/nat           -- IPv6 NAT
/ipv6/firewall/raw           -- IPv6 bypass connection tracking
```

**Dynamic address lists** allow rules to automatically add IPs to a named list, enabling port-knock sequences and brute-force protection:

```
# SSH brute-force protection — block IPs that exceed 10 connections in 1 minute
/ip/firewall/filter
add chain=input protocol=tcp dst-port=22 connection-state=new \
    src-address-list=ssh-blacklist action=drop
add chain=input protocol=tcp dst-port=22 connection-state=new \
    src-address-list=ssh-stage3 action=add-src-to-address-list \
    address-list=ssh-blacklist address-list-timeout=1d
add chain=input protocol=tcp dst-port=22 connection-state=new \
    src-address-list=ssh-stage2 action=add-src-to-address-list \
    address-list=ssh-stage3 address-list-timeout=1m
add chain=input protocol=tcp dst-port=22 connection-state=new \
    src-address-list=ssh-stage1 action=add-src-to-address-list \
    address-list=ssh-stage2 address-list-timeout=1m
add chain=input protocol=tcp dst-port=22 connection-state=new \
    action=add-src-to-address-list address-list=ssh-stage1 \
    address-list-timeout=1m
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/119144601/Firewall+and+Quality+of+Service

### Routing

```
/ip/route                    -- static routes and routing table
/routing/ospf                -- OSPF instances, areas, interfaces
/routing/bgp                 -- BGP connections, templates, sessions
/routing/rip                 -- RIP configuration
/routing/filter              -- route filters and selection rules
/routing/table               -- routing tables for VRF / policy routing
/routing/bfd                 -- Bidirectional Forwarding Detection
/routing/id                  -- router ID configuration
/ip/vrf                      -- Virtual Routing and Forwarding
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328222/Unicast+Routing+Protocols

### Bridging & Switching

```
/interface/bridge            -- bridge interfaces
/interface/bridge/port       -- bridge port membership
/interface/bridge/vlan       -- bridge VLAN table (802.1Q)
/interface/vlan              -- VLAN interfaces
/interface/vxlan             -- VXLAN interfaces
/interface/bonding           -- link aggregation (LACP)
/interface/ethernet          -- ethernet interface settings
/interface/ethernet/switch   -- switch chip configuration
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328068/Bridging+and+Switching

### MLAG (Multi-chassis Link Aggregation, v7.3+)

MLAG allows two RouterOS devices to present a single logical bonding interface to downstream switches, providing active-active redundancy without STP blocking.

```
/interface/bridge/mlag       -- MLAG peer and port configuration
```

Key concepts:
- Both peers share the same bridge and VLAN configuration
- Peers communicate via a dedicated inter-chassis link (ICL) — typically a dedicated VLAN on the bond
- Configure the same `mlag-id` on matching ports on both peers

```
# On both peers — create the ICL VLAN
/interface/vlan add name=icl-vlan vlan-id=4000 interface=ether10

# On both peers — add ICL port to bridge
/interface/bridge/port add interface=icl-vlan bridge=bridge1

# On peer A
/interface/bridge/mlag set bridge=bridge1 peer-port=icl-vlan

# On both peers — assign MLAG IDs to the bond ports facing downstream
/interface/bridge/port set [find interface=bond1] mlag-id=10
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/196345860/MLAG

### MPLS

RouterOS supports MPLS (label switching) and LDP for label distribution.

```
/mpls                        -- global MPLS settings
/mpls/ldp                    -- LDP (Label Distribution Protocol) configuration
/mpls/ldp/interface          -- LDP-enabled interfaces
/mpls/forwarding-table       -- label forwarding table (read-only)
/mpls/local-label            -- manually assigned local labels
/mpls/traffic-eng            -- RSVP-TE / traffic engineering tunnels
/interface/mpls              -- MPLS-enabled interfaces
/ip/route                    -- use routing-mark for MPLS-based policy routing
```

Basic LDP setup:

```
/mpls/ldp set enabled=yes transport-address=10.0.0.1 lsr-id=10.0.0.1
/mpls/ldp/interface add interface=ether1
/mpls/ldp/interface add interface=ether2
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328144/MPLS

### VPN

```
/interface/wireguard         -- WireGuard interfaces and peers
/ip/ipsec                    -- IPsec policies, peers, proposals
/interface/l2tp-server       -- L2TP server
/interface/l2tp-client       -- L2TP client
/interface/ovpn-server       -- OpenVPN server
/interface/ovpn-client       -- OpenVPN client
/interface/sstp-server       -- SSTP server
/interface/sstp-client       -- SSTP client
/interface/pptp-server       -- PPTP server (legacy, insecure)
/interface/eoip              -- EoIP tunnels
/interface/gre               -- GRE tunnels
/interface/ipip              -- IPIP tunnels
/zerotier                    -- ZeroTier virtual networking
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/119144597/Virtual+Private+Networks

### Wireless

```
/interface/wifi              -- WiFi interfaces (v7 wifi package, replaces /interface/wireless on new hardware)
/interface/wifi/security     -- security profiles (WPA2/WPA3)
/interface/wifi/channel      -- channel configuration
/interface/wifi/datapath     -- datapath (bridge/routing integration)
/interface/wifi/configuration -- reusable AP/station configuration profiles
/interface/wifi/cap          -- CAPsMAN v2 Controlled AP (new wifi package, v7.13+)
/interface/wireless          -- legacy wireless interfaces (older hardware / ath9k / ath10k)
/interface/wireless/security-profiles -- legacy security profiles
/caps-man                    -- legacy CAPsMAN controller (for /interface/wireless APs)
```

> **v7 wifi package note**: The `wifi` package (new) and the `wireless` package (legacy) are mutually exclusive on a given device. Hardware using ath9k/ath10k chips uses `wireless`; newer hardware (IPQ-40xx and later) uses `wifi`. CAPsMAN v2 (`/interface/wifi/cap` on APs, `/wifi/capsman` on the controller) is the new unified management plane.

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/1409138/Wireless

### System & Management

```
/system/identity             -- device hostname
/system/resource             -- CPU, memory, uptime, version info
/system/routerboard          -- hardware info, firmware version
/system/package              -- installed packages
/system/package/update       -- check for and install updates
/system/clock                -- date/time settings
/system/ntp/client           -- NTP client
/system/scheduler            -- scheduled tasks
/system/script               -- stored scripts
/system/logging              -- log configuration and actions
/system/history              -- undo/redo history
/system/backup               -- binary backup save/load
/user                        -- user accounts and groups
/user/group                  -- permission groups (full, read, write, or custom policy bitmask)
/user/aaa                    -- AAA settings: RADIUS authentication, accounting
/radius                      -- RADIUS client configuration (server address, secret, service)
/interface/list              -- named interface lists used in firewall, discovery, MAC server rules
/ip/service                  -- management services (ssh, winbox, api, www, etc.)
/ip/ssh                      -- SSH server settings
/certificate                 -- TLS/SSL certificate management
/file                        -- file storage
/tool/e-mail                 -- email sending
/tool/fetch                  -- HTTP/FTP fetch utility
/tool/mac-server             -- MAC telnet/winbox server
/ip/neighbor                 -- neighbor discovery
/tool/romon                  -- RoMON (Router Management Overlay Network) for remote device access
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328059/RouterOS

### Diagnostics & Monitoring

```
/ping                        -- ICMP ping
/tool/traceroute             -- traceroute
/tool/torch                  -- real-time traffic monitor per interface
/tool/packet-sniffer         -- packet capture
/tool/bandwidth-test         -- throughput testing
/tool/speed-test             -- speed test
/tool/netwatch               -- host reachability monitor
/tool/ip-scan                -- network scanner
/ip/traffic-flow             -- NetFlow/IPFIX export
/system/health               -- voltage, temperature, fan speed
/snmp                        -- SNMP agent configuration
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328236/Tools

### Queues & QoS

```
/queue/simple                -- simple queues (per-client bandwidth limits)
/queue/tree                  -- hierarchical queue trees (HTB)
/queue/type                  -- queue disciplines (PCQ, SFQ, CAKE, FIFO)
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328088/Queues

### Container (Docker-compatible, v7.4+)

RouterOS supports running OCI-compatible containers via the `container` package. Requires a device with enough RAM (≥ 256 MB free) and the `container` package installed.

```
/container                   -- container instances
/container/config            -- global container settings (registry URL, RAM limit, etc.)
/container/mounts            -- bind-mount paths from RouterOS filesystem into containers
/container/envs              -- environment variable sets for containers
```

**Workflow:**

```
# 1. Enable container mode and set registry (requires reboot after package install)
/system/device-mode/update container=yes
/container/config/set registry-url=https://registry-1.docker.io ram-high=512M

# 2. Create a veth pair for the container's network
/interface/veth add name=veth1 address=172.17.0.2/24 gateway=172.17.0.1

# 3. Add the veth to a bridge (or use as standalone)
/interface/bridge/port add interface=veth1 bridge=bridge1

# 4. Pull and create the container
/container add remote-image=alpine:latest interface=veth1 \
    root-dir=disk1/containers/alpine logging=yes

# 5. Start / stop
/container start 0
/container stop 0

# 6. Check status
/container print
```

> **Constraints**: Containers run in a read-only root by default; use `/container/mounts` for persistent storage. No GPU passthrough. ARM and x86 images only — match the router's architecture. The `container` package must be installed separately and device-mode must be enabled, which requires a reboot.

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/84901929/Container

### Hotspot (Captive Portal)

RouterOS Hotspot provides a captive portal for authenticating users before granting internet access. Commonly deployed in guest Wi-Fi and hotel/venue networks.

```
/ip/hotspot                  -- hotspot server instances
/ip/hotspot/user             -- local hotspot user database
/ip/hotspot/user/profile     -- user profiles (rate limits, session time, data quota)
/ip/hotspot/host             -- active hotspot hosts (MAC → IP bindings)
/ip/hotspot/active           -- currently authenticated sessions
/ip/hotspot/ip-binding       -- bypass or block specific IPs/MACs
/ip/hotspot/walled-garden    -- URLs accessible without authentication
/ip/hotspot/walled-garden/ip -- IP ranges accessible without authentication
/ip/hotspot/profile          -- server profiles (RADIUS, login page, DNS)
/ip/hotspot/cookie           -- remembered login cookies
```

Quick setup (wizard):

```
/ip/hotspot/setup
```

Manual setup:

```
# Create hotspot server on the access interface
/ip/hotspot add name=hotspot1 interface=bridge-guests \
    address-pool=hs-pool idle-timeout=30m keepalive-timeout=none

# Create an address pool for hotspot clients
/ip/pool add name=hs-pool ranges=192.168.100.2-192.168.100.254

# Add a local user
/ip/hotspot/user add name=guest password=guest123 profile=default

# Set rate limit on the default profile
/ip/hotspot/user/profile set default rate-limit=10M/10M
```

> **RADIUS integration**: Set `use-radius=yes` on the hotspot server profile and configure `/radius` to authenticate users against an external RADIUS server instead of the local database.

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/1409115/HotSpot

## Security Best Practices

These recommendations come from the official MikroTik security documentation. Apply them when configuring any RouterOS device.

### User Access

- Change the default `admin` username; create a new full-access user with a strong password (12+ characters, mixed case, numbers, symbols)
- Disable or remove the `admin` user after verifying new credentials work
- Restrict user login to specific IP addresses: `/user set myname address=192.168.88.0/24`
- **For API/automation access, always use a dedicated restricted user** — never use a full-privilege account in scripts or external tools:

```
# Create a read-only group for monitoring/automation
/user/group add name=api-read policy=read,api,!write,!policy,!test,!password,!reboot,!sniff,!sensitive

# Create the API user, restricted to the management subnet
/user add name=api-user group=api-read password=<strong-password> \
    address=192.168.88.0/24
```

If the automation needs to make changes, scope the policy bitmask to only the required permissions (e.g. add `write` but not `policy` or `password`).

### Services

- Disable unused services: `/ip/service disable telnet,ftp,www,api`
- Change default SSH port: `/ip/service set ssh port=2200`
- Restrict service access by IP: `/ip/service set winbox address=192.168.88.0/24`
- Enable strong SSH crypto: `/ip/ssh set strong-crypto=yes`

### MAC Access & Discovery

- Restrict MAC server to LAN interfaces only (create an interface list, assign bridge to it, apply to MAC server and MAC WinBox)
- Disable neighbor discovery on WAN interfaces: `/ip/neighbor/discovery-settings set discover-interface-list=LAN`

### Firewall Input Chain

Protect the router itself with input chain rules:

```
/ip/firewall/filter
add chain=input action=accept connection-state=established,related,untracked
add chain=input action=drop connection-state=invalid
add chain=input in-interface=ether1 action=accept protocol=icmp
add chain=input in-interface=ether1 action=drop
```

### Firewall Forward Chain

Protect LAN clients:

```
/ip/firewall/filter
add chain=forward action=fasttrack-connection connection-state=established,related
add chain=forward action=accept connection-state=established,related
add chain=forward action=drop connection-state=invalid
add chain=forward action=drop connection-state=new connection-nat-state=!dstnat in-interface=ether1
```

> **Fasttrack warning**: `fasttrack-connection` offloads established/related flows to the fast path, bypassing subsequent firewall rules, mangle rules, and queues. If QoS (queue trees, mangle marking) is not working after adding fasttrack, this is the cause. Either remove the fasttrack rule or mark the traffic you want to shape *before* it hits fasttrack using a connection mark in the mangle table.

### NAT (Masquerade)

```
/ip/firewall/nat add chain=srcnat out-interface=ether1 action=masquerade
```

### Disable Unnecessary Services

```
/tool/bandwidth-server set enabled=no
/ip/dns set allow-remote-requests=no
/ip/proxy set enabled=no
/ip/socks set enabled=no
/ip/upnp set enabled=no
/ip/cloud set ddns-enabled=no update-time=no
```

> **v7.17+ note**: In RouterOS v7.17 and later the default for `ddns-enabled` changed. If Back To Home was enabled, disable it first, then use `ddns-enabled=auto` instead of `ddns-enabled=no`.

### Remote Access

- Never open management ports directly to the internet
- Use VPN (WireGuard or IPsec) for remote management
- If remote access is required, use firewall rules to restrict source IPs

### Keep Updated

- Regularly check for RouterOS updates: `/system/package/update check-for-updates`
- Upgrade when security patches are available: `/system/package/update install`

## Configuration Management

### Binary Backup

Full device state backup (not human-readable, includes passwords):

```
/system/backup save name=mybackup
/system/backup load name=mybackup.backup
```

### Text Export/Import

Human-readable configuration export (does not include passwords or certificates):

```
/export file=config.rsc            -- full export (includes all defaults — noisy for diffing)
/export compact file=config.rsc    -- only non-default settings (recommended for automation and diffs)
/ip/firewall export file=fw.rsc   -- export specific section
/import file=config.rsc            -- import configuration
```

> **Prefer `export compact`** for version control, diffing, and automation. The full export includes every default value, producing large files where meaningful changes are hard to spot.

### Undo/Redo

RouterOS tracks all changes in `/system/history`:

```
/system/history print
/undo
/redo
```

### Safe Mode

Enter safe mode from CLI by pressing `Ctrl+X`. All changes are automatically reverted if the session disconnects unexpectedly. Exit safe mode with `Ctrl+X` again to commit changes.

### Reset

```
/system/reset-configuration                           -- reset to default config
/system/reset-configuration no-defaults=yes            -- reset to blank (no config)
/system/reset-configuration skip-backup=yes            -- reset without saving backup
/system/reset-configuration run-after-reset=flash/config.rsc  -- apply script after reset
```

## Common Patterns Quick Reference

### Basic Home Router Setup

```
/interface/bridge add name=bridge1
/interface/bridge/port add interface=ether2 bridge=bridge1
/ip/address add address=192.168.88.1/24 interface=bridge1
/ip/dhcp-client add disabled=no interface=ether1
/ip/dhcp-server/setup                    -- interactive DHCP setup
/ip/firewall/nat add chain=srcnat out-interface=ether1 action=masquerade
```

### WireGuard VPN

```
/interface/wireguard add name=wg0 listen-port=13231
/interface/wireguard/peers add interface=wg0 public-key="<peer_pubkey>" \
    allowed-address=10.0.0.2/32 endpoint-address=<peer_ip> endpoint-port=13231
/ip/address add address=10.0.0.1/24 interface=wg0

# Allow WireGuard UDP on the input chain (replace ether1 with your WAN interface)
/ip/firewall/filter add chain=input protocol=udp dst-port=13231 \
    in-interface=ether1 action=accept place-before=0

# Route traffic destined for the peer's LAN via the tunnel
/ip/route add dst-address=10.0.0.2/32 gateway=wg0
```

### Port Forwarding

```
/ip/firewall/nat add chain=dstnat protocol=tcp dst-port=8080 \
    in-interface=ether1 action=dst-nat to-addresses=192.168.88.100 to-ports=80
```

### Static Route

```
/ip/route add dst-address=10.10.0.0/16 gateway=192.168.88.254
```

### VLAN Configuration

```
/interface/vlan add name=vlan100 vlan-id=100 interface=ether1
/ip/address add address=10.100.0.1/24 interface=vlan100
```

### IPsec Site-to-Site VPN

```
# Proposal (encryption settings)
/ip/ipsec/proposal add name=default auth-algorithms=sha256 enc-algorithms=aes-256-cbc pfs-group=modp2048

# Peer (remote gateway)
/ip/ipsec/peer add name=site-b address=203.0.113.2/32 exchange-mode=ike2

# Identity (authentication — pre-shared key)
/ip/ipsec/identity add peer=site-b auth-method=pre-shared-key secret=s3cr3t

# Policy (traffic to protect)
/ip/ipsec/policy add src-address=192.168.1.0/24 dst-address=192.168.2.0/24 \
    peer=site-b tunnel=yes action=encrypt proposal=default

# Exclude IPsec traffic from masquerade (add before srcnat masquerade rule)
/ip/firewall/nat add chain=srcnat src-address=192.168.1.0/24 \
    dst-address=192.168.2.0/24 action=accept place-before=0
```

### OSPF (Single Area)

```
# Create router ID
/routing/id add id=10.0.0.1 name=main

# Create OSPF instance
/routing/ospf/instance add name=default version=2 router-id=main

# Create backbone area
/routing/ospf/area add name=backbone area-id=0.0.0.0 instance=default

# Add interfaces to OSPF
/routing/ospf/interface-template add interfaces=ether1 area=backbone
/routing/ospf/interface-template add interfaces=ether2 area=backbone

# Verify neighbors
/routing/ospf/neighbor print
```
