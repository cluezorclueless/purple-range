# Build Log 03 — vmbr1 isolation validation

**Date:** 2026-09-03
**Node:** attack (7060) — VM 100 (attacker-01) on vmbr1

## Goal
Verify that vmbr1 (the isolated lab bridge, see ADR 0001) actually
seals VMs off from the home LAN and internet — before trusting it
with a deliberately vulnerable target.

## Method
Built a minimal Debian 13 VM (attacker-01) with its only NIC on
vmbr1. Two failures during install were the first evidence:
- DHCP autoconfiguration failed (no DHCP server on the isolated net)
- Network mirror unreachable (installed CD-only)

Both are expected: nothing on vmbr1 routes off-box.

Assigned static 10.10.10.50/24, gateway 10.10.10.1 (the host).
Then tested reachability in three directions.

## Results
| Target | Meaning | Result |
|-------------------|----------------|----------|
| 10.10.10.1 | Proxmox host | REPLIED |
| 8.8.8.8 | Internet | FAILED |
| 192.168.12.1 | Home LAN | FAILED |

reply / fail / fail = isolation holds. The VM can talk to the host on
the isolated bridge but has no path to the internet or the home
network.

(Screenshot: pending upload)
## Consequence
An isolated attacker box can't `apt install` tools while sealed.
That controlled outbound access is the job of the pfSense firewall
(future work) — one leg on vmbr0, one on vmbr1, policing every
crossing. Isolation first, controlled egress second.
