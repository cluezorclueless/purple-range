# Build Log 02 — Tailscale remote access: the Crostini userspace trap

**Date:** 2026-09-03
**Node:** penguin (Chromebook / Crostini) → attack (7060)

## Goal
Reach the Proxmox host (attack) over Tailscale from the Chromebook,
so the lab is reachable off-LAN (campus, hotspot).

## Symptom
- `tailscale ping 100.84.45.34` → pong every time, 6-7ms
- `ssh root@100.84.45.34` → hangs forever at "Connecting... port 22"
- Plain `ping` to the tailnet IP → 100% loss at every packet size

## Investigation
- Ruled out the box: LAN ssh (192.168.12.200) works instantly,
sshd listening on :22, netcheck clean (UDP true, easy NAT, DERP reachable).
- Ruled out MTU: 500-byte pings failed too, not just large ones.
- `ip route get 100.84.45.34` → routed via **eth0**, not a tunnel interface.
- `ip addr show tailscale0` → **Device does not exist.**

## Root cause
Tailscale inside Crostini runs in **userspace-networking mode** — no
`tailscale0` kernel interface. `tailscale ping` uses Tailscale's own
userspace proxy so it works, but real ICMP/TCP has no tunnel route and
gets dumped out eth0, where the 100.x address goes nowhere. Crostini
can't give the container a TUN device, so Tailscale falls back to
userspace automatically. No command inside the container can fix this.

## Resolution
Disabled the container Tailscale (`tailscale down`, `systemctl disable
--now tailscaled`) and installed Tailscale at the **ChromeOS level**
(Android app, Play Store), signed in as the same account. That creates a
real system tunnel the container inherits. `ssh root@100.84.45.34` then
connected.

## Lessons
- `tailscale ping` proves the control plane, NOT that real traffic routes.
- `ip route get <ip>` and `ip addr show tailscale0` are the two commands
that pinpoint a missing-tunnel-interface problem in one shot.
- Near-end vs far-end: the box was blamed for hours; the crippled end was
always the client.
