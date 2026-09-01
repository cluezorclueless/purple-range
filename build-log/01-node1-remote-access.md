# Node 1 Remote Access via Tailscale
**Date:** 2026-09-01

## Goal
Reach node 1 from outside the LAN so range work isn't limited to
hours I'm physically at home. Long-running attack chains and AD
enumeration need to be startable from campus and checkable later,
which means shell access from anywhere without exposing SSH to
the internet.


## Symptom

Tailscale install script failed on attack.home.arpa:

curl: (6) Could not resolve host: tailscale.com

Presented as DNS. It wasn't.

## Investigation

1. `cat /etc/resolv.conf` → `nameserver 192.168.12.1`. Correct. DNS config ruled out.
2. `ping -c 3 1.1.1.1` → `Destination Host Unreachable` from 192.168.12.200. The local box generated that, not a remote host. Packets never left.
3. `ip r` → routes correct, both flagged `linkdown`.
4. `ip a` → `NO-CARRIER` on nic0 and vmbr0. Ethernet cable was loose.
5. Reseated cable, `systemctl restart networking`. `linkdown` cleared, interfaces `UP` and `LOWER_UP`.
6. `ping -c 3 192.168.12.1` → still 100% loss, but silent now. No local error. Packets leaving, nothing returning.
7. `ip neigh` → gateway MAC resolved (`f8:3e:b0:48:69:48`), IPv6 entry flagged `router`, same MAC. Layer 2 fine.
8. Flushed ARP, re-pinged. Still silent.
9. Hypothesized an ISP outage. Wrong — no outage. The box has no WiFi card, which is a separate issue.
10. `dhclient -v vmbr0` → `DHCPOFFER of 192.168.12.227 from 192.168.12.1`, `DHCPACK`, bound. Router alive, subnet correct, leasing normally.
11. Re-ran the install script. Downloaded clean, apt repos all reachable, Tailscale installed.

## Root cause

Two separate things, one real:

**Loose Ethernet cable.** Actual fault. Fixed by reseating.

**ICMP-silent gateway.** Not a fault. The router doesn't answer pings on its LAN interface — common on consumer hardware. It made a working network read as dead.

## Resolution

Cable reseated, networking restarted, Tailscale installed and authenticated. Node reachable from outside the LAN.

Node is currently on DHCP lease .227 rather than static .200. To be reconciled — either restore static config or convert to a DHCP reservation.


## Lessons
**Dont Trust Cables.**
**DHCP.**
