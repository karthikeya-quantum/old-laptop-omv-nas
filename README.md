# Old Laptop Home Cloud (OpenMediaVault NAS)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenMediaVault](https://img.shields.io/badge/OpenMediaVault-7.x-blue.svg)](https://www.openmediavault.org/)
[![Debian](https://img.shields.io/badge/OS-Debian%2012%20(Bookworm)-red.svg)](https://www.debian.org/)

A step-by-step guide and configuration repository to turn an old, unused laptop into a low-power, reliable **Network Attached Storage (NAS)** and personal home cloud server using **OpenMediaVault (OMV)**.

> **Why use an old laptop?** Laptops consume very little power (typically 10W–25W at idle), include a built-in screen and keyboard for easy troubleshooting, and have an integrated battery that acts as a built-in Uninterruptible Power Supply (UPS) during short power outages.

---

## Table of Contents
- [Features](#-features)
- [Hardware Requirements](#-hardware-requirements)
- [Phase 1: Hardware & BIOS Preparation](#-phase-1-hardware--bios-preparation)
- [Phase 2: Installing Debian & OpenMediaVault](#-phase-2-installing-debian--openmediavault)
- [Phase 3: Laptop-Specific Power Configuration](#-phase-3-laptop-specific-power-configuration)
- [Phase 4: Storage & Shared Folders Setup](#-phase-4-storage--shared-folders-setup)
- [Phase 5: Optional Docker Containers](#-phase-5-optional-docker-containers)
- [Troubleshooting & FAQ](#-troubleshooting--faq)
- [License](#-license)

---

## ✨ Features
- **Local Network File Sharing:** Seamless access from Windows, macOS, Linux, iOS, and Android via **SMB/CIFS**.
- **Low Power Consumption:** Optimized idle power draw with lid-closed headless operation.
- **Automated Backups:** Scheduled `rsync` jobs or Time Machine support.
- **Expandable via Docker:** Ready for self-hosted apps like Immich (Google Photos alternative), Jellyfin (media server), or Nextcloud.

---

## 💻 Hardware Requirements
- **Laptop:** Any 64-bit x86 laptop made in the last 10–12 years.
- **CPU:** Dual-core processor or better (Intel Core i3/i5 or AMD equivalent).
- **RAM:** Minimum **2 GB** (4 GB+ recommended if running Docker containers).
- **Boot Drive:** Internal SSD or HDD (minimum 16 GB; OMV cannot share the OS partition for user data without plugins).
- **Storage Drive(s):** External USB 3.0 HDD/SSD or secondary internal drive for bulk data storage.
- **Network:** **Ethernet cable (Recommended)** for stable, high-speed file transfers. *(Wi-Fi works for setup, but wired is strongly advised for NAS workloads).*
- **Installation Media:** 1x USB flash drive (8 GB or larger).

> ⚠️ **Battery Safety Warning:** Inspect your laptop battery before running it 24/7. If the battery is swollen or severely degraded, remove it completely and run the laptop directly off the AC power adapter to prevent fire hazards.

---

## 🛠 Phase 1: Hardware & BIOS Preparation

1. **Create Bootable Media:**
   - Download the **Debian 12 (Bookworm) Netinst ISO** (64-bit) or the official **OpenMediaVault ISO**.
   - Flash the ISO onto your USB stick using [Rufus](https://rufus.ie/) (Windows) or [BalenaEtcher](https://etcher.balena.io/) (Windows/macOS/Linux).
2. **Configure Laptop BIOS/UEFI:**
   - Plug in the USB drive, power on the laptop, and enter BIOS (usually `F2`, `F10`, `F12`, or `Del`).
   - Enable **Intel VT-x / AMD-V** (Virtualization Technology) if you plan to run Docker.
   - Set **AC Power Recovery** (if available) to *Power On* so the NAS restarts automatically after a blackout.
   - Set the USB drive as the first boot priority, save, and exit.

---

## 💿 Phase 2: Installing Debian & OpenMediaVault

*(Note: Installing minimal Debian first is recommended on laptops because it handles laptop Wi-Fi/Ethernet drivers and disk partitioning better than the standalone OMV ISO).*

1. **Install Debian (No Desktop Environment):**
   - Boot from the USB drive and select **Install** (text mode) or **Graphical Install**.
   - Follow the prompts for language, location, and hostname (e.g., `homelab-nas`).
   - Leave the **root password** blank to automatically grant `sudo` privileges to your primary user.
   - At the **Software Selection** screen, **UNCHECK** *Debian desktop environment* and *GNOME*, and **CHECK** only:
     - `SSH server`
     - `standard system utilities`
2. **Install OpenMediaVault via Script:**
   - Reboot into your new Debian terminal and log in.
   - Run the official OMV installation script:
     ```bash
     sudo apt update && sudo apt upgrade -y
     sudo apt install -y curl wget gnupg
     wget -O - [https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install](https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install) | sudo bash
     ```
   - Wait 10–15 minutes for the installation to finish and reboot automatically.

---

## 🔋 Phase 3: Laptop-Specific Power Configuration

By default, closing a laptop lid puts the system to sleep, disconnecting your NAS. Configure `systemd` to ignore the lid switch:

1. Connect to your laptop via SSH (`ssh username@<laptop-ip>`) or use the local keyboard.
2. Edit the `logind.conf` file:
   ```bash
   sudo nano /etc/systemd/logind.conf
