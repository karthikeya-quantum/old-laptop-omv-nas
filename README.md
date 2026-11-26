# 🖥️ Old Laptop to 24/7 Home Cloud (OpenMediaVault + Docker)

Turn an old, unused laptop or PC into a fast, private, self-hosted **Home Cloud & Media Server**—replacing Google Photos, Netflix/Plex, Google Drive, and sketchy online file/PDF converters with your own hardware.

This repository is a **battle-tested, step-by-step blueprint** built from real-world deployment and troubleshooting. Every guide includes **copy-paste commands**, **expected terminal outputs**, and **exact fixes for common errors** so anyone can build this from scratch—even with zero prior Linux or server experience.

---

## 🗺️ Step-by-Step Build Phases

Follow these guides strictly in order:

| Phase | Guide | What You Will Set Up |
| :--- | :--- | :--- |
| **Phase 1** | **[Base OS, 24/7 Laptop Tuning & Storage](docs/01-base-os-and-hardware-tuning.md)** | Install OpenMediaVault, disable lid-close sleep, enable **Battery Conservation Mode**, mount drives by UUID, and set up Docker Compose |
| **Phase 2** | **[Networking, Pi-hole & Tailscale Split DNS](docs/02-networking-pihole-and-tailscale.md)** | Static IP, network-wide ad blocking, custom `.lan` URLs, global Docker DNS fix, and remote access without port forwarding |
| **Phase 3** | **[Media & Personal Cloud Apps](docs/03-media-and-personal-cloud.md)** | **Immich** (photo backup), **Jellyfin** (with Intel QuickSync GPU transcoding), **FileBrowser**, and Windows/Mac SMB shares |
| **Phase 4** | **[Daily Utilities & One-Click Web Terminal](docs/04-daily-utilities-and-web-terminal.md)** | **Stirling-PDF**, **ConvertX** (1,000+ format converter), and **WeTTY** one-click browser SSH login |
| **Phase 5** | **[Dashboard, Monitoring & Security](docs/05-dashboard-monitoring-and-security.md)** | **Homarr** dashboard, **Scrutiny** drive S.M.A.R.T. health, **Uptime Kuma** Telegram alerts, and **Fail2Ban** hardening |

---

## ✨ Full Service & Port Map

| Category | Service | Replaces | Port | Local / Remote URL |
| :--- | :--- | :--- | :--- | :--- |
| **Dashboard** | **Homarr** | Browser Bookmarks | `7575` | `http://homecloud.lan:7575` |
| **Photos & Videos** | **Immich** | Google Photos / iCloud | `2283` | `http://homecloud.lan:2283` |
| **Movies & TV** | **Jellyfin** | Netflix / Plex | `8096` | `http://homecloud.lan:8096` |
| **Cloud Files** | **FileBrowser & SMB** | Google Drive / Local NAS | `8085` | `http://homecloud.lan:8085` |
| **PDF Toolkit** | **Stirling-PDF** | iLovePDF / Smallpdf | `8089` | `http://homecloud.lan:8089` |
| **File Converter** | **ConvertX** | CloudConvert (1,000+ formats) | `3009` | `http://homecloud.lan:3009` |
| **Ad-Blocker & DNS** | **Pi-hole** | Network-wide Ad & Tracker Blocking | `8081` | `http://homecloud.lan` |
| **Remote Access** | **Tailscale** | Port Forwarding / Paid VPN | — | Works anywhere via `.lan` Split DNS |
| **Web SSH Terminal**| **WeTTY** | Desktop SSH Client | `3011` | `https://homecloud.lan` |
| **Drive Health** | **Scrutiny** | Manual S.M.A.R.T. Checks | `8088` | `http://homecloud.lan:8088` |
| **Uptime & Alerts** | **Uptime Kuma** | Paid Monitoring + Telegram Alerts | `3001` | `http://homecloud.lan:3001` |

---

## 🛠️ Why an Old Laptop Makes a Great Home Server

* **Built-In UPS:** The laptop's battery keeps your server running safely through short power cuts without corrupting databases.
* **Ultra-Low Power Draw:** Mobile CPUs (like Intel Core i5-6200U) idle at **8W–15W**, costing pennies a day to run 24/7.
* **Hardware Video Transcoding:** Built-in Intel QuickSync (QSV) handles smooth 1080p/4K video streaming in Jellyfin.

---

## 🔒 Configuration Placeholders

All `compose/*/docker-compose.yml` templates use standardized placeholders—replace these with your own values before deploying:

* `<SERVER_LAN_IP>` — Your server's static local IP (e.g., `192.168.0.113`)
* `<SERVER_TAILSCALE_IP>` — Your server's Tailscale IP (e.g., `100.x.x.x`)
* `<APPDATA_UUID>` — Mount UUID of your fast SSD for Docker configurations
* `<MEDIA_UUID>` — Mount UUID of your storage drive for photos, movies, and files
