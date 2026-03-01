# mikrotik-routeros

An [Agent Skills](https://agentskills.io) skill that gives AI coding agents deep knowledge of MikroTik RouterOS v7 — CLI syntax, REST API, firewall, routing, VPN, wireless, security hardening, and configuration management.

## What it covers

- **CLI structure** — hierarchical menu paths, CRUD operations, print modifiers, scripting basics
- **REST API** — GET/PUT/PATCH/DELETE/POST patterns, filtering, constraints, curl examples
- **Classic API** — socket-based protocol for streaming and monitoring use cases
- **Feature areas** — IP/DHCP, firewall, routing (OSPF/BGP/RIP), bridging, VPN (WireGuard/IPsec/L2TP/OpenVPN), wireless, QoS
- **Security hardening** — user access, service restrictions, firewall input/forward chains, NAT
- **Configuration management** — binary backup, text export/import, safe mode, undo/redo, reset

## Installation

The skill is a single directory containing `SKILL.md`. Place it in the skills directory for your agent tool.

### OpenCode

```bash
mkdir -p ~/.config/opencode/skills
git clone https://github.com/sollie/mikrotik-routeros-skill ~/.config/opencode/skills/mikrotik-routeros
```

### Claude Code

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/sollie/mikrotik-routeros-skill ~/.claude/skills/mikrotik-routeros
```

### Other compatible tools

Any tool listed on [agentskills.io](https://agentskills.io) supports this format. Consult your tool's documentation for the correct skills directory path.

## Usage

Once installed, the skill is loaded automatically when you work on RouterOS-related tasks. You can also invoke it directly:

```
/mikrotik-routeros
```

## Compatibility

Follows the [Agent Skills open standard](https://agentskills.io/specification). Compatible with any agent that implements the standard.
