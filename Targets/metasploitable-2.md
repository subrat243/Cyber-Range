# Metasploitable 2

Intentionally vulnerable Linux (Ubuntu 8.04-based) virtual machine from Rapid7, used for practicing network enumeration, service exploitation and Metasploit usage.

| Item | Value |
|---|---|
| IP address | `10.10.10.10/24` |
| Gateway | `10.10.10.1` |
| Default login | `msfadmin` / `msfadmin` |
| Network | VirtualBox Internal Network (Cyber Range) |

> This VM is extremely vulnerable. Keep it on the isolated Internal Network only.

## 1. Download

Download the Metasploitable 2 archive from the official Rapid7 / SourceForge distribution and extract it. It contains a pre-built disk image (`Metasploitable.vmdk`).

Store the VMDK in your VirtualBox VM folder, for example `~/VirtualBox VMs/Metasploitable2/`.

## 2. Create the VM in VirtualBox

1. **Machine > New**
2. Name: `metasploitable2`
3. Type: **Linux**, Version: **Ubuntu (64-bit)** (32-bit also works)
4. Memory: **512 MB** (1024 MB if available)
5. Hard disk: choose **Use an existing virtual hard disk file** and select `Metasploitable.vmdk`
6. Finish creating the VM.

## 3. Configure networking

Open **Settings > Network > Adapter 1**:

- Attached to: **Internal Network**
- Name: `<CYBER_RANGE_NETWORK_NAME>` (must match the pfSense Cyber Range adapter)
- Promiscuous Mode: Deny
- Cable Connected: Enabled

Disable any other adapters.

## 4. First boot and login

Start the VM and log in:

```
login: msfadmin
password: msfadmin
```

## 5. Set a static IP

Edit the network configuration:

```bash
sudo nano /etc/network/interfaces
```

Replace the `eth0` section with:

```
auto eth0
iface eth0 inet static
address 10.10.10.10
netmask 255.255.255.0
gateway 10.10.10.1
```

Restart networking:

```bash
sudo /etc/init.d/networking restart
```

## 6. Verify

On the VM:

```bash
ifconfig eth0
route -n
ping -c 4 10.10.10.1
```

From Kali:

```bash
ping -c 4 10.10.10.10
nmap -sV 10.10.10.10
```

Expected: many open services (FTP, SSH, Telnet, SMTP, HTTP, SMB, MySQL, PostgreSQL, VNC and others).

## 7. Snapshot

Power off or pause the VM, then take a snapshot named `ms2-clean-baseline` so the target can be restored after exploitation exercises.

## Notes

- Isolation: pfSense blocks this subnet from reaching Management, Active Directory, Security Operations and Malware Sandbox networks.
- If Kali cannot reach the VM, check the static route on Kali and the pfSense rules on the HOST_MGMT interface (see [network-topology.md](../Architecture/network-topology.md)).
