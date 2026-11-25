# Network Topology

## Subnets

| pfSense interface | Network | Subnet | Purpose |
|---|---|---|---|
| `MGMT` | Management | `10.0.0.0/24` | Infrastructure administration |
| `CYBER` | Cyber Range | `10.10.10.0/24` | Vulnerable targets and web applications |
| `AD` | Active Directory | `10.20.20.0/24` | Windows domain environment |
| `SECURITY` | Security Operations | `10.30.30.0/24` | Wazuh and security monitoring |
| `MALWARE` | Malware Sandbox | `10.40.40.0/24` | Isolated malware analysis |
| `HOST_MGMT` | Host management | `192.168.56.0/24` | Kali-to-pfSense administration |

## Logical diagram

```
                    Kali Linux (VirtualBox host)
                      192.168.56.1 (HOST_MGMT)
                               |
                               v
                       +---------------+
                       |    pfSense    |
                       +---------------+
                        |     |      |
          10.10.10.1 ---+     |      +--- other segments
                |             |            (Mgmt / AD / SecOps / Sandbox)
                v
        Cyber Range 10.10.10.0/24
        |            |              |
 Metasploitable 2  Metasploitable 3  ub-web01
  10.10.10.10       10.10.10.11      10.10.10.20
                                     (Docker: Juice Shop :3000)
```

## Traffic path: Kali to a Cyber Range target

1. Kali sends traffic from the HOST_MGMT network (`192.168.56.0/24`).
2. The packet enters pfSense on the **`HOST_MGMT` interface**.
3. pfSense routes it to the `CYBER` interface (`10.10.10.1`).
4. The target responds via its default gateway, `10.10.10.1`.

Kali needs a static route to the Cyber Range through pfSense:

```bash
sudo ip route add 10.10.10.0/24 via <PFSENSE_HOST_MGMT_IP>
```

## pfSense firewall rules

pfSense evaluates rules on the interface where traffic **enters** the firewall. This matters when deciding where each rule belongs.

### `HOST_MGMT` interface (Kali to Cyber Range)

| Source | Destination | Port | Action | Purpose |
|---|---|---|---|---|
| Kali (`<KALI_HOST_MGMT_IP>`) | `10.10.10.0/24` | Any (or as needed) | Allow | Testing and administration of targets |
| Kali | `10.10.10.20` | TCP 3000 | Allow | Juice Shop direct access |

### `CYBER` interface (traffic from targets)

| Source | Destination | Port | Action | Purpose |
|---|---|---|---|---|
| `CYBER net` | `10.10.10.1` | UDP/TCP 53 | Allow | DNS via pfSense resolver |
| `10.10.10.20` | Any (below the block rules) | TCP 80, 443 | Allow (temporary) | Package and Docker image downloads. Remove or restrict after setup |
| `CYBER net` | `MGMT net` (`10.0.0.0/24`) | Any | Block | Isolation |
| `CYBER net` | `AD net` (`10.20.20.0/24`) | Any | Block | Isolation |
| `CYBER net` | `SECURITY net` (`10.30.30.0/24`) | Any | Block | Isolation (add exceptions only for Wazuh agent ports) |
| `CYBER net` | `MALWARE net` (`10.40.40.0/24`) | Any | Block | Isolation |

Rule ordering matters: place block rules above any broad allow rule.

> Outbound internet access (NAT) for the Cyber Range is needed only during setup (OS updates, Docker). Disable it afterwards.

## DHCP on the `CYBER` interface

DHCP is enabled on `CYBER` with this scope:

| Setting | Value |
|---|---|
| Pool | `10.10.10.100` to `10.10.10.200` |
| Offered DNS | `10.10.10.1` (pfSense Unbound resolver) |
| Gateway | `10.10.10.1` |
| Lease time | Default 7200 s (2 hours), maximum 86400 s (24 hours) |

All static Cyber Range addresses (`.10`, `.11`, `.20`, `.21`, `.22`) sit below the pool, so DHCP never hands them out. See [ip-addressing.md](ip-addressing.md) for the static versus mapped options.
