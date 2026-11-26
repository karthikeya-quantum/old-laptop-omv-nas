# 🖥️ Old Laptop to 24/7 Home Cloud (OpenMediaVault + Docker)

Turn an old, unused laptop or PC into a fast, private, self-hosted **Home Cloud & Media Server**—replacing Google Photos, Netflix/Plex, Google Drive, and sketchy online file/PDF converters with your own hardware.

## Architecture Overview
- **Base OS**: OpenMediaVault (Debian-based)
- **Containerization**: Docker + Docker Compose

## Service Matrix

| Service | Description | Port |
| :--- | :--- | :--- |
| **Homarr** | Dashboard | 7575 |
| **Immich** | Photo & video cloud backup | 2283 |
| **Jellyfin** | Media streaming (with Intel QuickSync) | 8096 |
| **FileBrowser** | Local NAS / Cloud storage | 8085 |
| **Stirling-PDF** | PDF operations toolkit | 8089 |
| **ConvertX** | File conversion utility | 3009 |
| **Pi-hole** | Network-wide DNS ad-blocking | 8081 |
| **Tailscale** | Overlay VPN with Split DNS | - |
| **WeTTY** | Web-based SSH terminal | 3011 |
| **Scrutiny** | Hard drive health monitoring | 8088 |
| **Uptime Kuma** | Monitoring & alerts | 3001 |

## Configuration Placeholders
- `<SERVER_LAN_IP>`
- `<SERVER_TAILSCALE_IP>`
- `<APPDATA_UUID>`
- `<MEDIA_UUID>`
