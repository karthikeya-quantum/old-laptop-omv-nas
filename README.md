# Old Laptop Home Cloud (OpenMediaVault NAS)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OpenMediaVault](https://img.shields.io/badge/OpenMediaVault-7.x-blue.svg)](https://www.openmediavault.org/)
[![Debian](https://img.shields.io/badge/OS-Debian%2012%20(Bookworm)-red.svg)](https://www.debian.org/)

A beginner-friendly, step-by-step guide and automation repository to transform an old, unused laptop or PC into a low-power, 24/7 **Network Attached Storage (NAS)** and personal home cloud server using **OpenMediaVault (OMV)**.

> **Why use an old laptop?** Laptops consume minimal electricity (typically 10W–25W at idle), include a built-in screen and keyboard for local troubleshooting, and feature an integrated battery that acts as a built-in Uninterruptible Power Supply (UPS) during short power outages.

---

## 🗺️ Master Setup Roadmap

Follow these phases in exact order. Each guide includes **copy-paste commands**, **expected screen outputs**, and **fixes for common errors**:

1. **[Phase 1: BIOS & Minimal Debian 12 Installation](docs/01-bios-and-debian.md)** — Prepare your USB installer, configure UEFI/BIOS, and install a clean command-line Debian base.
2. **[Phase 2: OpenMediaVault 7 Installation & Network Setup](docs/02-omv-install.md)** — Install OMV via script, log into the web dashboard, and lock in a Static IP address. *(Coming Next)*
3. **[Phase 3: Laptop Power, Lid-Close & Screen Optimization](docs/03-laptop-power.md)** — Prevent sleep when closing the lid, turn off the screen backlight, and optimize battery health. *(Coming Next)*
4. **[Phase 4: Storage Drives, Shared Folders & SMB File Sharing](docs/04-storage-shares.md)** — Format/mount external drives, create user accounts, and access files from Windows, Mac, iOS, and Android. *(Coming Next)*
5. **[Phase 5: Self-Hosted Apps with Docker (Immich, Jellyfin, etc.)](docs/05-docker-apps.md)** — Install OMV-Extras and Docker Compose to run your own private Google Photos or Netflix alternative. *(Coming Next)*
6. **[Phase 6: Master Troubleshooting & Error Reference](docs/06-troubleshooting.md)** — Quick solutions for permission errors, disconnected USB drives, and network drops. *(Coming Next)*

---

## 💻 Hardware Checklist

Before starting, make sure you have the following:

| Component | Minimum Requirement | Recommended |
| :--- | :--- | :--- |
| **Laptop / PC** | Any 64-bit x86 laptop/PC (made after ~2012) | Intel Core i3/i5 or AMD Ryzen/A-Series |
| **RAM** | 2 GB | 4 GB – 8 GB+ (essential if running Docker apps) |
| **OS Boot Drive** | 16 GB internal HDD/SSD | 64 GB+ internal SSD *(Note: OMV reserves the entire OS drive for itself)* |
| **Storage Drive(s)** | 1x External USB HDD/SSD or secondary internal drive | USB 3.0 External HDD/SSD (1 TB+) for your actual files |
| **Network** | Wi-Fi (works for setup, not ideal for daily use) | **Wired Ethernet Cable** connected directly to your router |
| **USB Installer** | 1x 8 GB USB Flash Drive | Any 8 GB+ USB 3.0 stick (will be erased during setup) |

> ⚠️ **CRITICAL BATTERY SAFETY WARNING:** Inspect your laptop battery before leaving it plugged in 24/7. If the battery is swollen, bulging, or dead, **physically remove it** and run the laptop directly off the wall power adapter to prevent overheating or fire hazards.

---

## ✨ What You Will Build
* **Private Local Cloud:** Store and stream photos, videos, and backups across all devices on your home Wi-Fi with zero monthly subscription fees.
* **Headless Operation:** Runs quietly in a corner with the laptop lid closed and screen turned off.
* **Expandable App Server:** Capable of running Docker containers for personal photo backup (Immich), media streaming (Jellyfin), or ad-blocking (Pi-hole).

---

## 📄 License
This project is open-source under the [MIT License](LICENSE).
