# Metasploitable 3

Rapid7's intentionally vulnerable multi-service target, built for more realistic exploitation, post-exploitation and privilege escalation practice than Metasploitable 2.

| Item | Value |
|---|---|
| IP address | `10.10.10.11/24` |
| Gateway | `10.10.10.1` |
| Variant used | `<WINDOWS_2008_R2 or UBUNTU_14.04>` |
| Default login | `vagrant` / `vagrant` |
| Network | VirtualBox Internal Network (Cyber Range) |

> Adjust this document to match the variant and build method you actually used.

## 1. Prerequisites (on Kali)

- VirtualBox
- Vagrant
- Packer (only if building from source)
- Git
- Roughly 60 GB free disk space for a build, and 2 to 4 GB RAM available for the VM

```bash
vagrant --version
VBoxManage --version
```

## 2. Obtain the VM

### Option A: Pre-built Vagrant box

```bash
mkdir -p ~/labs/metasploitable3 && cd ~/labs/metasploitable3

# Windows Server 2008 R2 variant
vagrant init rapid7/metasploitable3-win2k8

# or Ubuntu 14.04 variant
# vagrant init rapid7/metasploitable3-ub1404

vagrant up
```

### Option B: Build from source

```bash
git clone https://github.com/rapid7/metasploitable3.git
cd metasploitable3

# Windows variant
packer build --only=virtualbox-iso ./packer/templates/windows_2008_r2.json
vagrant box add windows_2008_r2_virtualbox.box --name=metasploitable3-win2k8
vagrant up
```

Builds take a long time and need a stable internet connection. Check the repository README for the current build commands, as paths and template names change between versions.

## 3. Move the VM to the Cyber Range network

Vagrant creates the VM with a NAT adapter for its own management. For the Cyber Range, switch it to the Internal Network.

1. Shut the VM down: `vagrant halt`
2. In VirtualBox, open the VM **Settings > Network**.
3. Set **Adapter 1**:
   - Attached to: **Internal Network**
   - Name: `<CYBER_RANGE_NETWORK_NAME>`
4. Disable any extra adapters (such as a host-only adapter) you do not need.
5. Start the VM from the VirtualBox GUI from now on. Vagrant commands will no longer reach it once NAT is removed.

## 4. Set a static IP

### Windows variant

Run in an elevated command prompt (find the real adapter name with `netsh interface show interface`):

```cmd
netsh interface ip set address name="<ADAPTER_NAME>" static 10.10.10.11 255.255.255.0 10.10.10.1
netsh interface ip set dns name="<ADAPTER_NAME>" static 10.10.10.1
```

### Ubuntu 14.04 variant

```bash
sudo nano /etc/network/interfaces
```

```
auto eth0
iface eth0 inet static
address 10.10.10.11
netmask 255.255.255.0
gateway 10.10.10.1
```

```bash
sudo ifdown eth0 && sudo ifup eth0
```

## 5. Verify

From the VM:

```
ping 10.10.10.1
```

From Kali:

```bash
ping -c 4 10.10.10.11
nmap -sV -Pn 10.10.10.11
```

Expected: a wide range of vulnerable services (web servers, SMB, database engines, remote administration services and others, depending on the variant). Use `-Pn` if the Windows firewall blocks ping.

## 6. Snapshot

Take a snapshot named `ms3-clean-baseline` before any exploitation exercises.

## Notes

- Isolation and routing are identical to the other Cyber Range targets (see [network-topology.md](../Architecture/network-topology.md)).
- Do not give this VM unrestricted internet access.
