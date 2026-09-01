# purple-range-meaning
node purple team range - Proxmox, AD , forest , detection stack	

# purple-range

A three-node purple team range I'm building on used office hardware. Red side, blue side, and the network between them, all of it stood up by hand.

I'm not running someone else's lab script. Every node gets built, broken, and documented, because if I can't build it I don't understand it yet.

## Why

Most home labs are a checklist. Install Kali, run the tool, screenshot the flag. That teaches you the tool, not the system underneath it.

I want the other thing. Write the tooling, watch what it does on the wire, then sit on the blue side and try to catch it. Both halves of the same problem.

## Architecture

| Node | Hostname | Role |
|---|---|---|
| 1 | `attack.home.arpa` | Attacker infrastructure, custom tooling, pfSense |
| 2 | `forest.home.arpa` | Windows AD forest — the target |
| 3 | `blue.home.arpa` | Detection stack and log pipeline |

Two bridges:

- `vmbr0` — management, sits on the home LAN
- `vmbr1` — isolated lab network, no gateway, no physical port. Nothing in the range talks to the house.

## Hardware

Dell OptiPlex 7060 MT per node. Proxmox VE 9.2, ext4 root. Cheap, quiet, and if I destroy one it costs me an afternoon.

Node 1 is up. Nodes 2 and 3 are next.

## No Kali

Deliberate. Kali hands you 600 tools and no reason to understand any of them. I'm running minimal Debian and writing what I need.

Slower. That's the point.
