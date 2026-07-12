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

## Status

- [x] KVM/libvirt stack installed, `libvirtd` active, user in `libvirt` group
- [x] `lab-isolated` network defined, started, autostart on
- [ ] Server 2025 ISO staged in `/var/lib/libvirt/images/`
- [ ] DC01 VM created and installed
