# Home Lab: Building an Ubuntu Server VM with KVM

**Date:** July 2026
**Host machine:** Linux Mint desktop
**Goal:** Build a virtual server lab for hands-on Linux practice (A+ / IT fundamentals study)

---

## Summary

Built a virtualized Ubuntu Server 24.04 lab on my Linux Mint machine using KVM/virt-manager, connected to it remotely over SSH, patched it, and completed my first command-line practice session.

---

## Setup Steps

### 1. Chose KVM over VirtualBox

VirtualBox failed to build its kernel modules on my system (DKMS/kernel incompatibility), so I removed it and switched to KVM, which is built into the Linux kernel — no third-party modules to break.

Installed the virtualization stack:

```bash
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
sudo usermod -aG libvirt,kvm $USER
```

Logged out and back in for the group change to take effect.

### 2. Verified the hypervisor

```bash
virt-host-validate
```

All QEMU checks passed (hardware virtualization, /dev/kvm present and accessible). The LXC failures in the output are for container virtualization, which I'm not using — safe to ignore.

### 3. Downloaded the Ubuntu Server ISO

Browser download didn't work, so I grabbed it from the terminal instead:

```bash
cd ~/Downloads
wget https://releases.ubuntu.com/24.04/ubuntu-24.04.3-live-server-amd64.iso
```

### 4. Fixed a permissions error

**Problem:** VM creation failed with `Could not open '...iso': Permission denied`.

**Cause:** The VM process runs as the `libvirt-qemu` system user, which can't read files inside my home folder.

**Fix:** Moved the ISO into libvirt's own storage directory:

```bash
sudo mv ~/Downloads/ubuntu-24.04.3-live-server-amd64.iso /var/lib/libvirt/images/
```

Deleted the failed VM in virt-manager and recreated it — worked on the second try.

### 5. Installed Ubuntu Server 24.04

VM specs: 2 GB RAM, 2 CPUs, 20 GB virtual disk.

Installer choices:
- Ubuntu Server (full, not minimized)
- Use entire disk with default LVM layout (ubuntu-vg / ubuntu-lv)
- **Installed OpenSSH server** so I can manage the VM remotely
- No extra server snaps

### 6. Connected over SSH

Found the VM's IP with `ip a` (the `inet 192.168.122.x` line under `enp1s0` — leave off the `/24` suffix), then from the Mint terminal:

```bash
ssh <username>@192.168.122.x
```

### 7. Patched the server

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

56 updates on the fresh install. Reconnected after reboot to verify — banner now shows 0 updates.

---

## Troubleshooting Log

| Problem | Cause | Fix |
|---|---|---|
| VirtualBox wouldn't install | Kernel module build failure (DKMS) | Switched to KVM |
| ISO "Permission denied" at VM creation | libvirt-qemu user can't read my home folder | Moved ISO to /var/lib/libvirt/images/ |
| `ssh: Could not resolve hostname 192.168.122.xx` | Typed the placeholder instead of the real IP | Used actual IP from `ip a` |
| `ssh: No route to host` | Tried to connect while VM was still booting | Checked VM was running in virt-manager, waited, retried |
| SSH failed with `/24` in the address | `/24` is the network mask shown by `ip a`, not part of the IP | Dropped the `/24` |

**Lesson learned:** before troubleshooting a connection, check whether the server is even running. "Is it plugged in?" applies to VMs too.

---

## First Commands Session

Practiced the basic navigation and file commands over SSH:

| Command | What it does |
|---|---|
| `pwd` | Print working directory — where am I? |
| `ls` | List the contents of the current folder |
| `cd <folder>` | Move into a folder (`cd` alone returns home) |
| `mkdir <name>` | Make a new directory |
| `nano <file>` | Simple terminal text editor (Ctrl+O save, Ctrl+X exit) |
| `cat <file>` | Print a file's contents to the screen |

Full cycle completed: made a `notes` folder, created `first-note.txt` in nano, and read it back with `cat`.

**Tip picked up along the way:** the prompt itself shows your location — `~/notes$` means you're in the notes folder under your home directory. An empty result from `ls` isn't an error; the folder is just empty. Silence usually means success in Linux.

---

## Next Steps

- File operations: `cp`, `mv`, `rm`
- System health: `df -h`, `free -h`, `top`
- Service management: `systemctl status ssh`
