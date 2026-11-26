# Phase 5: Dashboard (Homarr), Monitoring & Security Hardening

In this final phase, you will build your unified home screen (**Homarr**), monitor hard drive health before drives fail (**Scrutiny**), receive instant **Telegram** alerts if any service goes offline (**Uptime Kuma**), and harden your server against unauthorized access.

---

## 1. Deploy Homarr v2 (Unified Home Cloud Dashboard)

1. Generate a 64-character hex encryption key in **CTerm** / SSH:
   ```bash
   openssl rand -hex 32
   ```
2. Create the persistent folder:
   ```bash
   mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/homarr/appdata
   ```
3. In **OMV → Services → Compose → Files → Create (+)**, name the stack **`homarr`**, paste `compose/homarr/docker-compose.yml` (inserting your generated `SECRET_ENCRYPTION_KEY`, `<APPDATA_UUID>`, and `<SERVER_LAN_IP>`), and click **Save → Up**.
4. Open **`http://homecloud.lan:7575`** and create your admin account.

### Recommended Homarr Tweaks
* **Protect Your Board With Admin Login:** Go to **Manage → Settings** and leave **Global home board** and **Global mobile board** blank (`×`) so anonymous visitors cannot view your dashboard without signing in.
* **Same-Tab Navigation on Mobile:** Under **Manage → Apps**, edit each app tile and toggle **`Open in new tab`** **OFF** if you want your phone's Back button to return straight to Homarr.
* **One-Click SSH Tile:** Point your Terminal tile directly to **`https://homecloud.lan`**.

---

## 2. Deploy Scrutiny (Hard Drive & SSD S.M.A.R.T. Health)

Old laptops often have aging hard drives. Scrutiny tracks S.M.A.R.T. attributes, temperatures, and bad sectors so you can replace a failing drive before losing data.

1. Check your exact connected drive device names (`/dev/sda`, `/dev/sdb`, `/dev/sdc`, etc.):
   ```bash
   lsblk -d -o NAME,SIZE,MODEL
   ```
2. Create the config folders:
   ```bash
   mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/scrutiny/{config,influxdb}
   ```
3. Update the `devices:` section in `compose/scrutiny/docker-compose.yml` to match your drives, deploy as **`scrutiny`**, and open **`http://homecloud.lan:8088`**.

---

## 3. Deploy Uptime Kuma + Telegram Downtime Alerts

1. Create the data folder:
   ```bash
   mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/uptime-kuma
   ```
2. Deploy `compose/uptime-kuma/docker-compose.yml` as **`uptime-kuma`** and open **`http://homecloud.lan:3001`**.
3. **Set Up Free Telegram Alerts:**
   * Message **`@BotFather`** on Telegram → send `/newbot` → copy your **Bot Token**.
   * Message **`@userinfobot`** on Telegram → copy your numeric **Chat ID**.
   * In Uptime Kuma, go to **Settings → Notifications → Setup Notification** → select **Telegram**, paste your Bot Token and Chat ID, click **Test**, and enable **Default enabled** so every monitor automatically alerts your phone if a service goes down.

---

## 4. Security Hardening Checklist

Your server is already protected from internet scanners because **zero ports are forwarded on your router** and remote access uses **Tailscale's encrypted WireGuard tunnel**. Complete these final steps to lock down local network access:

1. **Install `fail2ban` (Blocks Brute-Force Login Attempts):**
   ```bash
   sudo apt-get update && sudo apt-get install -y fail2ban
   sudo systemctl enable --now fail2ban
   ```
2. **Enable Automatic Security Updates:**
   * In OMV, go to **System → Update Management → Settings** and keep security updates enabled (or verify via `sudo dpkg-reconfigure -plow unattended-upgrades`).
3. **Use Strong, Unique Passwords:**
   * Ensure OMV (`admin` & `root`), Pi-hole, Homarr, Immich, Jellyfin, FileBrowser, and ConvertX all use strong passwords.
   * Keep **Tailscale Key Expiry** enabled in your Tailscale Admin Console so lost or old phones automatically lose access.

---

## 🛠️ Troubleshooting Common Phase 5 Issues

### Issue 1: All App Tiles in Homarr Show Red Status Dots Even Though Apps Open Fine
* **Cause:** Homarr's background health-check pings `http://homecloud.lan:<PORT>`, which fails if the container cannot resolve `.lan` domains.
* **Fix:** Ensure `extra_hosts: ["homecloud.lan:<SERVER_LAN_IP>"]` is present in `compose/homarr/docker-compose.yml` and `/etc/docker/daemon.json` points to your Pi-hole IP (see Phase 2).

### Issue 2: Scrutiny Fails on First Boot With `s6-rc: fatal: timed out`
* **Cause:** Starting Scrutiny simultaneously with 10+ other containers during boot can cause its embedded InfluxDB service to exceed s6-overlay's startup timeout.
* **Fix:** Simply restart the container once other services have settled: `docker restart scrutiny`.
