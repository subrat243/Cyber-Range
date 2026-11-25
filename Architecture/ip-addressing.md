# IP Addressing

The Cyber Range uses the pfSense interface `CYBER` (`10.10.10.0/24`). pfSense is the gateway and DNS resolver at `10.10.10.1`.

## Cyber Range (`CYBER` interface, `10.10.10.0/24`)

| Device | Hostname | IP address | Notes |
|---|---|---|---|
| pfSense (`CYBER` interface) | pfSense | `10.10.10.1` | Default gateway |
| Metasploitable 2 | `metasploitable` | `10.10.10.10` | Static |
| Metasploitable 3 | `<METASPLOITABLE3_HOSTNAME>` | `10.10.10.11` | Static |
| Ubuntu web server | `ub-web01` | `10.10.10.20` | Static. Docker host for Juice Shop |
| Future: DVWA | - | `10.10.10.21` | Reserved |
| Future: WebGoat | - | `10.10.10.22` | Reserved |

## DHCP pool and static addresses

| Range | Use |
|---|---|
| `10.10.10.1` | pfSense (`CYBER` interface) |
| `10.10.10.2` to `10.10.10.99` | Static and reserved infrastructure addresses |
| `10.10.10.100` to `10.10.10.200` | DHCP pool |
| `10.10.10.201` to `10.10.10.254` | Unused |

All Cyber Range static addresses are below the DHCP pool, so they never conflict with dynamically assigned leases.

## Two ways to assign a fixed address

Use **one** method per VM, never both.

| Method | How | Best for |
|---|---|---|
| A. Static IP on the VM | Set the address in the guest OS (netplan on Ubuntu, `/etc/network/interfaces` on Metasploitable 2) | VMs that must work even if pfSense DHCP is down |
| B. DHCP static mapping | Keep the VM on DHCP and bind its MAC to the address in **Services > DHCP Server > CYBER > Static Mappings** | Central management of addresses in pfSense |

Example mapping for the web server:

| Hostname | MAC Address | IP Address | Subnet | Description |
|---|---|---|---|---|
| `ub-web01` | `<VM_MAC_ADDRESS>` | `10.10.10.20` | Cyber Range | Juice Shop host (later SafeLine) |

Find the VM's MAC address in VirtualBox under **Settings > Network > Adapter 1 > Advanced**. The pool is at `10.10.10.100` to `10.10.10.200`; mapped addresses outside it are allowed in pfSense.

## VirtualBox network settings

| VM | Adapter 1 mode | Network name |
|---|---|---|
| Metasploitable 2 | Internal Network | `<CYBER_RANGE_NETWORK_NAME>` |
| Metasploitable 3 | Internal Network | `<CYBER_RANGE_NETWORK_NAME>` |
| ub-web01 | Internal Network | `<CYBER_RANGE_NETWORK_NAME>` |

The network name must exactly match the one used by the pfSense Cyber Range adapter (names are case-sensitive).
