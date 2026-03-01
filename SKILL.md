---
name: mikrotik-routeros
description: Use when working with MikroTik RouterOS configuration, CLI commands, REST API, firewall rules, networking setup, VPN, wireless, bridging, routing protocols, or building tools that interact with RouterOS devices. Covers RouterOS v7 CLI syntax, REST API usage, security hardening, and best practices.
---

# MikroTik RouterOS

RouterOS is a Linux-based network operating system that powers MikroTik hardware devices and is also available as a virtual machine (Cloud Hosted Router / CHR). RouterOS v7 is the current major version, using Linux kernel 5.6.3. It provides routing, firewall, bridging, VPN, wireless, QoS, and monitoring capabilities configurable via CLI, WinBox GUI, WebFig, REST API, and a socket-based API.

## When to Use This Skill

**Activate when:**
- Configuring or automating MikroTik RouterOS devices
- Writing CLI commands or scripts for RouterOS
- Building tools that communicate with RouterOS via REST API or classic API
- Designing firewall rules, NAT, routing, VPN, or wireless configurations
- Implementing security hardening on RouterOS devices
- Developing this MCP server that wraps RouterOS functionality

**Do not use when:**
- Working with MikroTik SwOS (separate OS for certain switch models)
- The task is purely about Go language features unrelated to RouterOS

## Documentation Sources

Always consult these authoritative sources for RouterOS information:

| Source | URL | Use For |
|---|---|---|
| Official Docs | https://help.mikrotik.com/docs/ | Definitive reference for all features |
| Community Forum | https://forum.mikrotik.com/ | Real-world examples, troubleshooting, edge cases |
| First Time Config | https://help.mikrotik.com/docs/spaces/ROS/pages/328151/First+Time+Configuration | Initial setup guide and best practices |
| Securing Router | https://help.mikrotik.com/docs/spaces/RKB/pages/8323164/Securing+your+router | Security hardening checklist |
| REST API | https://help.mikrotik.com/docs/spaces/ROS/pages/47579162/REST+API | Programmatic access via HTTP/JSON |
| Classic API | https://help.mikrotik.com/docs/spaces/ROS/pages/47579160/API | Socket-based binary protocol |
| Firewall | https://help.mikrotik.com/docs/spaces/ROS/pages/328168/Firewall+and+Quality+of+Service | Firewall rules, NAT, mangle, QoS |
| Routing | https://help.mikrotik.com/docs/spaces/ROS/pages/328218/Unicast+Routing+Protocols | OSPF, BGP, RIP, static routing |
| VPN | https://help.mikrotik.com/docs/spaces/ROS/pages/328227/Virtual+Private+Networks | WireGuard, IPsec, L2TP, OpenVPN |

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
print count-only      -- just the count
print where disabled=yes  -- filter results
print stats           -- show statistics
```

### Scripting Basics

RouterOS has a built-in scripting language used in `/system/script` and `/system/scheduler`:

```
:local myVar "hello"
:if ($myVar = "hello") do={ :put "world" }
:foreach i in=[/interface find] do={
    :put [/interface get $i name]
}
```

## REST API Reference

The REST API (available since RouterOS v7.1beta4) is a JSON wrapper over the console API. It is the primary programmatic interface for external tools.

### Prerequisites

The `www-ssl` service (HTTPS) must be enabled, or `www` service (HTTP, v7.9+) for testing only.

### Base URL

```
https://<router_ip>/rest
```

### Authentication

HTTP Basic Auth using RouterOS user credentials.

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

Go client library: `github.com/go-routeros/routeros`

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

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328206/Network+Management

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
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328168/Firewall+and+Quality+of+Service

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

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328218/Unicast+Routing+Protocols

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

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328146/Bridging+and+Switching

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

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328227/Virtual+Private+Networks

### Wireless

```
/interface/wifi              -- WiFi interfaces (v7 wifi package)
/interface/wifi/security     -- security profiles
/interface/wifi/channel      -- channel configuration
/interface/wireless          -- legacy wireless interfaces
/interface/wireless/security-profiles -- legacy security profiles
/caps-man                    -- centralized AP management (CAPsMAN)
```

Docs: https://help.mikrotik.com/docs/spaces/ROS/pages/328230/Wireless

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
/ip/service                  -- management services (ssh, winbox, api, www, etc.)
/ip/ssh                      -- SSH server settings
/certificate                 -- TLS/SSL certificate management
/file                        -- file storage
/tool/e-mail                 -- email sending
/tool/fetch                  -- HTTP/FTP fetch utility
/tool/mac-server             -- MAC telnet/winbox server
/ip/neighbor                 -- neighbor discovery
```

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
/system/logging              -- system log
/system/health               -- voltage, temperature, fan speed
/snmp                        -- SNMP agent configuration
```

### Queues & QoS

```
/queue/simple                -- simple queues (per-client bandwidth limits)
/queue/tree                  -- hierarchical queue trees (HTB)
/queue/type                  -- queue disciplines (PCQ, SFQ, CAKE, FIFO)
```

## Security Best Practices

These recommendations come from the official MikroTik security documentation. Apply them when configuring any RouterOS device.

### User Access

- Change the default `admin` username; create a new full-access user with a strong password (12+ characters, mixed case, numbers, symbols)
- Disable or remove the `admin` user after verifying new credentials work
- Restrict user login to specific IP addresses: `/user set myname address=192.168.88.0/24`

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
/export file=config.rsc            -- full export
/export compact file=config.rsc    -- only non-default settings
/ip/firewall export file=fw.rsc   -- export specific section
/import file=config.rsc            -- import configuration
```

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
