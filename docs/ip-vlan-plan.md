# Homelab Network Architecture

| Network | Linux Bridge | Subnet | Gateway | Notes |
|---|---|---|---|---|
| WAN / Default | `virbr0` | `192.168.122.0/24` | `192.168.122.1` | NAT network; connected to OPNsense WAN |
| VLAN 10 – Users | `virbr1` | `10.10.10.0/24` | `10.10.10.1` | DHCP range: `10.10.10.100–10.10.10.200` |
| VLAN 20 – Servers | `virbr2` | `10.10.20.0/24` | `10.10.20.1` | Static IP addressing only |
| VLAN 30 – Management | `virbr3` | `10.10.30.0/24` | `10.10.30.1` | Mint host address: `10.10.30.2` |
| VLAN 40 – DMZ | `virbr4` | `10.10.40.0/24` | `10.10.40.1` | Web server planned for later deployment |

## Network Overview

- **Firewall / Router:** OPNsense
- **WAN:** Connected to the libvirt default NAT network.
- **Internal networks:** Four isolated virtual networks connected to separate OPNsense interfaces.
- **Management access:** Linux Mint host connects to the Management network through `virbr3` at `10.10.30.2`.
- **DHCP:** Planned for the Users network only; Servers and Management use static addressing.
- **DMZ:** Reserved for publicly exposed lab services, such as a web server.
