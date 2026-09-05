# 03 — attacker-01 VM Networking & Exegol Install

**Node:** attack (Proxmox VE 9.2.2)
**VM:** 100 / attacker-01 (Debian 13 "Trixie")
**Goal:** Give the attacker VM internet to install Exegol, while keeping it on the isolated lab bridge for engagements.

## Objective

attacker-01 was built on the isolated lab bridge only (vmbr1 / 10.10.10.0/24) — correct for detonation isolation, but it had no internet, so it couldn't pull Exegol. Added a second NIC on the LAN bridge (vmbr0 / 192.168.12.0/24) to install over the internet, keeping the isolated NIC for lab traffic.

- ens18 -> vmbr1 -> 10.10.10.50 (isolated, no gateway out, by design)
- ens19 -> vmbr0 -> 192.168.12.x (LAN + internet, for tooling)

## Step 1 — Add the second NIC (from host)

qm set 100 --net1 virtio,bridge=vmbr0
qm start 100

## Step 2 — Gotcha: no dhclient on Debian 13

ens19 was present but DOWN with no IP. dhclient is not shipped on minimal Debian 13.

ip link set ens19 up
dhcpcd -4 ens19 # -4 forces IPv4; without it only IPv6 came up

Result: ens19 got 192.168.12.101/24, UP.

## Step 3 — Core bug: dual default routes

Had an IPv4 address but ping 8.8.8.8 = 100% loss. ip route showed:

default via 10.10.10.1 dev ens18 onlink # WRONG: isolated NIC
default via 192.168.12.1 dev ens19 proto dhcp metric 1003

Root cause: DHCP on the isolated bridge handed out a default route via 10.10.10.1 at metric 0, beating the real gateway on ens19. All internet traffic went out the dead isolated NIC.

Fix:

ip route del default via 10.10.10.1 dev ens18
ping -c 3 8.8.8.8 # 0% loss

## Step 4 — Gotcha: apt only had the install CD as a source

apt install curl failed with "no installation candidate" — the netinst ISO was the only apt source. Fixed by pointing apt at the internet (deb.debian.org trixie main/updates/security), then:

apt update && apt install -y curl git pipx python3-pip

Result: 111 MB fetched at 14 MB/s.

## Step 5 — Install Exegol

curl -fsSL https://get.docker.com | sh
pipx install exegol
pipx ensurepath
exec bash
exegol install

## Known issues / TODO

- [ ] Route fix is manual and non-persistent. Isolated bridge (vmbr1) should NOT advertise a default gateway via DHCP. Fix at the DHCP config for 10.10.10.0/24.
- [ ] apt cdrom source disabled by hand. Bake proper deb.debian.org sources into the VM template.
- [ ] Detach the install ISO (ide2) from VM 100.
- [ ] No serial getty / guest agent in VM. Enable both so console never depends on the web UI.

## Lessons

- ICMP working != TCP working. Ping succeeded while all TCP hung — pointed at routing, not a dead host.
- Check ip route before assuming DNS/firewall. 100% loss looked like a firewall; it was a metric-0 route out the wrong NIC.
- Minimal Debian 13 is genuinely minimal — no dhclient, apt points only at install media.
