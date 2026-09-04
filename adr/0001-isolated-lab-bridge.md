# ADR 0001 — Isolated lab bridge (vmbr1)

**Date:** 2026-09-03
**Status:** Accepted

## Context
The range needs a network where the attacker VM and a deliberately
vulnerable AD forest can talk to each other and NOTHING else — no home
LAN, no internet. Running vulnerable/exploited machines on the home
network (192.168.12.0/24, via vmbr0) would expose real devices to lab
malware and attack traffic.

## Decision
Add a second bridge on the 7060, **vmbr1**, configured as isolated:
- `bridge-ports none` — not attached to any physical NIC, so traffic
cannot leave the host.
- No `gateway` line — nothing on the bridge has a route off-box.
- Host address `10.10.10.1/24` — separate RFC1918 range from the home
LAN, so there is no overlap or accidental routing.

vmbr0 (management, 192.168.12.200, bridged to nic0) is left untouched.

## Consequences
- Lab VMs attached only to vmbr1 are fully sealed from the home network
and internet by default.
- To give lab VMs controlled outbound access later, a firewall VM
(pfSense) will straddle both bridges — one leg on vmbr0, one on vmbr1
— so every crossing packet is explicitly policed.
- The 10.10.10.1 host address lets the Proxmox host reach lab VMs
directly (e.g. for a pfSense mgmt interface) without opening a path
off-box.

## Alternatives considered
- VLAN tagging on vmbr0: rejected — relies on switch config and still
shares the physical path; weaker isolation than no-physical-port.
- Single flat network: rejected — puts vulnerable machines next to real
devices.
