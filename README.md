# 🐧 Linux Lab Machine — Swapping Kali Linux for Ubuntu
### Building a Dedicated Physical Linux Environment for IT Infrastructure Learning

![Ubuntu](https://img.shields.io/badge/Ubuntu-26.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![SSH](https://img.shields.io/badge/SSH-Enabled-4A90D9?style=for-the-badge&logo=openssh&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-2ea44f?style=for-the-badge)
![Hardware](https://img.shields.io/badge/Hardware-Physical_Machine-6C757D?style=for-the-badge)

---

## Overview

This project documents the migration of a dedicated physical lab machine, an **ASUS X550ZA laptop** from Kali Linux to **Ubuntu Desktop 26.04 LTS (Resolute Raccoon)**. The goal: establish a real, bare-metal Linux environment that I can SSH into remotely from my primary workstation and use as a daily driver for building IT infrastructure skills.

This is **not a virtual machine**. The deliberate choice to work on physical hardware exposes real-world challenges that VMs never will — BIOS boot configuration, driver compatibility, network interface management, and hardware-level troubleshooting. Those are exactly the skills that matter in data center and IT infrastructure roles.

> **Why Ubuntu over Kali?** Kali is a penetration testing tool, not a daily driver. For building foundational Linux skills that apply directly to help desk, SOC Analyst, and data center technician roles, Ubuntu is the industry standard. It's what actually runs on enterprise servers.

---

## Hardware

| Component | Specification |
|---|---|
| **Model** | ASUS X550ZA (2015) |
| **CPU** | AMD A8-7100 (4-core, Kaveri mobile APU) |
| **RAM** | 8GB DDR3 |
| **Storage** | 256GB Fanxiang S101Q SATA SSD *(upgraded from original 1TB HDD)* |
| **Wired NIC** | Realtek Gigabit Ethernet (`enp4s0`) |
| **Wireless NIC** | MediaTek MT7630E 802.11b/g/n |
| **Known Issue** | Trackpad non-functional — USB mouse required for direct interaction |
| **Previous OS** | Kali Linux |
| **New OS** | Ubuntu Desktop 26.04 LTS |

> The original 1TB HDD was replaced with a SATA SSD as part of a prior hardware repurposing project. The old HDD was repurposed as external backup storage for a Jellyfin Media Server via USB enclosure — documented separately in this portfolio.

---

## Lab Environment

This machine is part of a broader home lab ecosystem:

| Machine | Role | OS |
|---|---|---|
| **SYKES-DESKTOP** | Primary workstation — all lab operations | Windows 11 |
| **SYKESHOMESERVER** | Docker, VMs, Pi-hole, Jellyfin, Pelican Panel | Ubuntu Server |
| **sykes-ubuntu** *(this machine)* | Dedicated Linux daily driver & learning environment | Ubuntu Desktop 26.04 LTS |
| **HP Victus** | Secondary workstation / portable | Windows 11 |

---

## Why Ubuntu 26.04 LTS?

Ubuntu 26.04 LTS *"Resolute Raccoon"* was released April 23, 2026. As an LTS release it receives 5 years of security updates (through April 2031). Key reasons for selecting it:

- **Enterprise dominance** — Ubuntu is the most widely deployed Linux distribution in federal IT and cloud environments
- **Direct skill transfer** — APT package management and `systemctl` service management apply immediately to the Ubuntu Server already running in my home lab
- **AMD hardware support** — Kernel 7.0 ships with improved AMD CPU and GPU driver support, directly relevant to the A8-7100 platform
- **Career alignment** — Targets the exact environments I'll encounter in federal IT contracting roles (help desk, SOC Analyst Tier 1, data center technician)

---

## Installation

### Flash Media Preparation

Initial attempts using **Balena Etcher** failed — the X550ZA's BIOS did not recognize the Etcher-created USB as a valid UEFI boot device. Switched to **Rufus 4.15** with the following settings:

| Setting | Value |
|---|---|
| Partition Scheme | GPT |
| Target System | UEFI (non-CSM) |
| File System | FAT32 |
| Write Mode | ISO Image mode (Recommended) |

![Rufus Settings](screenshots/01_Rufus_Settings.png)
*Rufus 4.15 — GPT / UEFI (non-CSM) configuration*

### BIOS Configuration

On the ASUS X550ZA — press **F2** on power-on to enter BIOS, **ESC** for one-time boot menu:

- **Fast Boot** → Disabled
- **Secure Boot** → Disabled
- **Boot Mode** → UEFI
- **Boot Order** → USB drive first

### Ubuntu Installer

| Screen | Selection |
|---|---|
| Installation Type | **Erase disk and install Ubuntu** *(wiped Kali completely)* |
| Encryption | No encryption |
| Timezone | America/New_York |
| Username | `mrbryansykes` |
| Hostname | `sykes-ubuntu` |
| Login | Password required |

![Ubuntu Installer](screenshots/03_Ubuntu_Installer_Language_Screen.jpg)
*Ubuntu 26.04 installer running on the ASUS X550ZA*

![Timezone](screenshots/04_Ubuntu_Installer_Timezone.jpg)
*Timezone set to America/New_York — Eastern*

---

## Post-Installation Configuration

### 1. Restore Sudo Access

After first boot, the user account lacked sudo privileges — and the root account is locked by default in Ubuntu 26.04. Fixed via Recovery Mode:

```bash
# Accessed via: GRUB → Advanced options for Ubuntu → Recovery mode → root shell
usermod -aG sudo mrbryansykes
passwd mrbryansykes

# Verified after reboot:
sudo whoami
# Output: root
```

### 2. Network Connectivity

Wi-Fi drivers were unavailable at install time (MediaTek MT7630E not included in the live environment). Connected via wired ethernet through a Meshforce mesh node LAN port.

> **Note:** Initial connection attempt used the WAN/uplink port — returned a `169.254.x.x` APIPA self-assigned IP with no internet. Switching to the correct LAN port resolved this immediately.

![Ping Success](screenshots/05_Ping_Success_Network_Confirmed.jpg)
*Consistent ping responses to 8.8.8.8 confirming ethernet connectivity*

### 3. System Update

```bash
sudo apt update && sudo apt upgrade -y
```

**317 packages upgraded** — including 179 LTS security updates (1,209MB). The MediaTek MT7630E wireless drivers installed automatically as part of this upgrade. Wi-Fi was available immediately after — no manual driver installation required.

### 4. OpenSSH Server

```bash
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh     # confirmed: active (running)
ip a                          # local IP: 192.168.1.224
```

### 5. SSH Remote Access Test

Tested from primary Windows desktop workstation via PowerShell. Initial connection returned a host key mismatch — the previous Kali install had registered a different key for the same IP:

```powershell
# Clear stale known_hosts entry
ssh-keygen -R 192.168.1.224

# Connect
ssh mrbryansykes@192.168.1.224
```

![SSH Warning](screenshots/07_SSH_Host_Key_Mismatch_Warning.png)
*Host key mismatch warning — resolved by clearing the stale known_hosts entry*

![SSH Success](screenshots/08_SSH_Session_Success.png)
*Successful SSH session — PowerShell title bar confirms `mrbryansykes@sykes-ubuntu`*

**The machine is now fully administrable over the network without direct physical interaction.**

---

## Troubleshooting Log

Every issue encountered and resolved during this build. These aren't failures — they're the job.

| # | Issue | Root Cause | Resolution |
|---|---|---|---|
| 1 | System hung on ASUS logo when booting from USB | Balena Etcher failed to write a valid UEFI bootloader for X550ZA | Switched to Rufus 4.15 with GPT / UEFI (non-CSM) settings |
| 2 | USB drive became write-protected after Etcher flash | Linux partition layout unrecognized by Windows — flagged as write-protected | Recovered via Windows Diskpart (`attributes disk clear readonly` → `clean` → `convert mbr` → `format`) |
| 3 | "No volume detected" error during Diskpart format | No valid partition table present after write-protect clear | Run `clean` before `create partition primary` |
| 4 | "Selected disk is not a fixed MBR disk" error | Etcher had written GPT — `active` command requires MBR | Run `convert mbr` after `clean`, before `create partition primary` |
| 5 | Windows format prompts after Rufus flash | Windows cannot read Linux filesystems — flags them as inaccessible | Dismissed prompts without formatting — flash was successful, proceeded to boot |
| 6 | Wi-Fi not detected during installation | MT7630E driver not included in live installer environment | Installed offline, connected via ethernet post-install, drivers pulled via `apt upgrade` |
| 7 | Sudo not available after first boot | User account created without administrator privileges; root locked by default | Used Ubuntu Recovery Mode → root shell → `usermod -aG sudo mrbryansykes` |
| 8 | Ethernet showing `169.254.x.x` APIPA IP | Cable connected to WAN/uplink port on Meshforce node — not a LAN port | Moved cable to LAN port — valid DHCP lease obtained immediately |
| 9 | SSH host key mismatch warning | Kali Linux previously registered a different SSH host key for same IP | `ssh-keygen -R 192.168.1.224` cleared stale entry — new connection accepted |
| 10 | Forgot sudo/login password — locked out of machine | Automatic login bypassed password on boot; SHIFT key trick failed on SSD due to fast boot | Booted from original Ubuntu USB in recovery mode → root shell → `passwd mrbryansykes` → reset successfully |

### USB Recovery Procedure (Diskpart)

```cmd
diskpart
list disk
select disk #          ← verify by size before proceeding
attributes disk clear readonly
clean                  ← WARNING: destroys all data on selected disk
convert mbr
create partition primary
active
format fs=fat32 quick
assign
exit
```

---

## End State

```
mrbryansykes@sykes-ubuntu:~$
```

- ✅ Kali Linux fully wiped
- ✅ Ubuntu Desktop 26.04 LTS installed on bare metal
- ✅ System fully updated — 317 packages, 179 LTS security patches
- ✅ Wi-Fi drivers installed and operational (MediaTek MT7630E)
- ✅ OpenSSH Server installed, enabled, and running
- ✅ SSH remote access confirmed from primary Windows desktop workstation
- ✅ Machine administrable entirely over the network — no direct physical interaction required
- ✅ SSH key-based authentication implemented (ED25519)
- ✅ Password-based SSH login disabled — key only

---

## Security Hardening

### SSH Key-Based Authentication

Password-based SSH login has been replaced with ED25519 key-based authentication — a significantly more secure access method that eliminates brute force risk entirely.

**On the Windows desktop — generate the key pair:**
```powershell
ssh-keygen -t ed25519 -C "sykes-ubuntu"
```

**Copy the public key to Ubuntu** (`ssh-copy-id` is not available natively in Windows PowerShell — use this instead):
```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh mrbryansykes@192.168.1.224 "cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

**On Ubuntu — disable password authentication entirely:**
```bash
sudo nano /etc/ssh/sshd_config
```
Change:
```
#PasswordAuthentication yes
```
To:
```
PasswordAuthentication no
```
Then restart SSH:
```bash
sudo systemctl restart ssh
```

**Verify key-only access:**
```powershell
ssh -o PasswordAuthentication=no mrbryansykes@192.168.1.224
```

Result: Connected without password prompt — key authentication confirmed, password login disabled.

> **Key backup reminder:** The private key lives at `C:\Users\<username>\.ssh\id_ed25519`. Back it up to a secure location. Losing it means losing SSH access to the machine.

---

## Learning Progression

This machine is Phase 1 of a structured Linux learning path:

```
Phase 1 (Current) → Ubuntu Desktop 26.04 LTS
                     CLI fundamentals, APT, systemctl, SSH, file permissions, networking

Phase 2           → Ubuntu Server (bare metal)
                     Headless administration, cron, UFW, LVM, log management

Phase 3           → Rocky Linux (RHEL-equivalent)
                     DNF/YUM package management, enterprise Linux for federal environments
```

---

## Next Steps

- [ ] Set DHCP reservation in router for `sykes-ubuntu` MAC address — lock IP permanently
- [ ] Configure UFW firewall — restrict to SSH (port 22) only
- [x] Implement SSH key-based authentication — disable password login
- [ ] Build CLI proficiency: `apt`, `systemctl`, `chmod`/`chown`, `grep`/`awk`/`sed`, `cron`, `ufw`, `htop`
- [ ] Transition to Ubuntu Server on bare metal (Phase 2)
- [ ] Begin Rocky Linux familiarization (Phase 3)

---

## Related Projects

- [🏠 Home Lab Server — SYKESHOMESERVER](#)
- [🪟 Active Directory Home Lab — Phase 1](#)
- [🔒 Security Onion Home Lab](#)
- [☁️ AWS GuardDuty Threat Detection](#)

---

## Documentation

Full project documentation (PDF) is available in this repository — covering hardware specs, installation walkthrough, post-install configuration, and the complete troubleshooting log with root cause analysis for every issue encountered.

---

*Bryan Sykes | Home Lab Portfolio | August 2026*  
*[![LinkedIn](https://img.shields.io/badge/LinkedIn-SecuredByBryan-0A66C2?style=flat&logo=linkedin)](https://linkedin.com) [![GitHub](https://img.shields.io/badge/GitHub-MrBSykes-181717?style=flat&logo=github)](https://github.com/MrBSykes) [![X](https://img.shields.io/badge/X-@SecuredByBryan-000000?style=flat&logo=x)](https://x.com/SecuredByBryan)*
  


