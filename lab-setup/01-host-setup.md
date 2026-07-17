# 01 — Host setup (Fedora KVM/libvirt)

Hypervisor for the lab is **native KVM/libvirt** on a Fedora host, driven by
`virt-manager`. This doc records the reproducible host-side prep done before any
VM exists: virtualization stack + an isolated lab network the DC and attacker
VMs attach to.

## Host

- Fedora (host), `qemu-kvm` + `virt-manager` from the distro repos.
- Hardware virtualization must be enabled in firmware (`/dev/kvm` present).

Verify:

```bash
ls -l /dev/kvm                       # exists => VT-x/AMD-V on
lscpu | grep -iE 'vmx|svm'           # cpu flag present
```

## Install the virtualization stack

```bash
sudo dnf install -y libvirt virt-install virt-manager qemu-kvm
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt "$USER"     # log out/in after, or: newgrp libvirt
```

Verify:

```bash
systemctl is-active libvirtd         # active
id | tr ',' '\n' | grep libvirt      # user is in the libvirt group
```

## Isolated lab network

The lab must not touch the internet — an isolated network keeps the DC from
silently pulling Windows Updates that would patch out the BadSuccessor
primitive, and contains the attack traffic.

`networks/lab-isolated.xml`:

- **isolated** (no `<forward>` element => no NAT, no route off-host)
- subnet **10.10.10.0/24**, gateway 10.10.10.1
- **no DHCP** — every VM gets a static IP (the DC requires one anyway)

Define, start, autostart:

```bash
virsh -c qemu:///system net-define networks/lab-isolated.xml
virsh -c qemu:///system net-start   lab-isolated
virsh -c qemu:///system net-autostart lab-isolated
virsh -c qemu:///system net-list --all
```

## Planned static addressing

| Host      | Role                     | IP           |
|-----------|--------------------------|--------------|
| gateway   | libvirt bridge (virtual) | 10.10.10.1   |
| DC01      | Server 2025 DC / DNS     | 10.10.10.10  |
| attacker  | client / tooling VM      | 10.10.10.20  |

DC01 points DNS at itself (10.10.10.10). Client points DNS at the DC.

## Create the DC VM

Stage the Server 2025 evaluation ISO in the pool directory
(`/var/lib/libvirt/images/`), then create the VM:

```bash
virt-install --connect qemu:///system \
  --name DC01 \
  --memory 4096 --vcpus 2 --cpu host \
  --disk path=/var/lib/libvirt/images/DC01.qcow2,size=60,bus=sata,format=qcow2 \
  --cdrom /var/lib/libvirt/images/<server-2025-eval>.iso \
  --network network=lab-isolated,model=e1000e \
  --os-variant win2k25 \
  --graphics spice --video qxl \
  --noautoconsole
```

Notes on the deliberate choices:

- **`bus=sata`** for the disk and **`model=e1000e`** for the NIC so the Windows
  installer sees both with its built-in drivers — no VirtIO driver injection
  needed. (VirtIO is faster; swap it in later if you want the perf.)
- **`--os-variant win2k25`** sets Server 2025 defaults (requires a recent
  `osinfo-db`; verify with `osinfo-query os | grep win2k25`).
- **`format=qcow2`** — thin/growable disk, not a full 60 GB up front.
- Network is the **isolated** `lab-isolated`, so the VM has no path to the
  internet and cannot pull Windows Updates.

Open the console (`virt-manager` -> DC01) to run Windows Setup. Pick
**Standard Evaluation (Desktop Experience)** — the GUI edition needed for
ADUC / `dsacls`. Set the built-in **Administrator** password at first boot.

## Base OS config (before promoting the DC)

Run in an elevated PowerShell on DC01. Adapter is `Ethernet` by default
(confirm with `Get-NetAdapter`).

Static IP, with DNS pointed at itself (a DC must resolve its own domain):

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.10.10 -PrefixLength 24 -DefaultGateway 10.10.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 10.10.10.10
```

Disable Windows Update to pin the patch baseline (the isolated network already
blocks it; this makes it explicit so the BadSuccessor primitive can't be
patched out mid-project):

```powershell
Stop-Service wuauserv
Set-Service wuauserv -StartupType Disabled
```

Rename the host and reboot:

```powershell
Rename-Computer -NewName "DC01" -Restart
```

After reboot, record the exact build number (part of the pinned baseline):

```powershell
hostname
Get-ComputerInfo | Select-Object OsBuildNumber, WindowsProductName, OsHardwareAbstractionLayer
```

Then take a clean pre-domain snapshot from the host:

```bash
virsh -c qemu:///system snapshot-create-as DC01 01-clean-os "clean OS, pre-domain"
```

## Status

- [x] KVM/libvirt stack installed, `libvirtd` active, user in `libvirt` group
- [x] `lab-isolated` network defined, started, autostart on
- [x] Server 2025 ISO staged in `/var/lib/libvirt/images/`
- [x] DC01 VM created; Windows Server 2025 Desktop Experience installed, at desktop
- [ ] Static IP 10.10.10.10, DNS-to-self, host renamed DC01
- [ ] Windows Update disabled, build number recorded
- [ ] Clean snapshot `01-clean-os` taken
- [ ] DC promotion (forest `dmsalab.local`) — next doc
