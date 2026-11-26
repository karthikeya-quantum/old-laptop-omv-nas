# Phase 3: Media & Personal Cloud (Immich, Jellyfin & FileBrowser)

In this phase, you will deploy your three core personal cloud services:
* **Immich** (`:2283`) — Automatic mobile photo and video backup (Google Photos replacement)
* **Jellyfin** (`:8096`) — Home theater streaming for Smart TVs, phones, and laptops with Intel QuickSync GPU transcoding
* **FileBrowser & SMB** (`:8085`) — Web file manager and native Windows/Mac network drive access

---

## 1. Deploy Immich (Photo & Video Backup)

1. Create the database, cache, and upload directories:
   ```bash
   mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/immich/{postgres,model-cache}
   mkdir -p /srv/dev-disk-by-uuid-<MEDIA_UUID>/Media/Photos
   ```
2. In **OMV → Services → Compose → Files → Create (+)**, name the stack **`immich`** and paste `compose/immich/docker-compose.yml` (replacing `<MEDIA_UUID>`, `<APPDATA_UUID>`, and `<YOUR_IMMICH_DB_PASSWORD>`).
3. Click **Save**, select **`immich`**, and click **Up** (play icon).
4. Open **`http://homecloud.lan:2283`**, create your admin account, and install the **Immich** app on your Android/iOS phone using `http://homecloud.lan:2283` as the server URL.

### Recommended Immich Storage Template
In Immich **Administration → Settings → Storage Template**, enable the template and set it to:
`{{y}}/{{MM}}/{{filename}}`
This organizes uploaded photos cleanly by **Year / Month** on your drive so they remain easy to browse even without Immich.

---

## 2. Deploy Jellyfin (Movies & TV Streaming)

Notice `compose/jellyfin/docker-compose.yml` uses **`network_mode: host`** and passes **`/dev/dri:/dev/dri`**. Host networking allows Smart TVs on your Wi-Fi to auto-discover Jellyfin automatically.

1. Create the config and cache folders:
   ```bash
   mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/jellyfin/{config,cache}
   ```
2. In **OMV → Services → Compose → Files → Create (+)**, name the stack **`jellyfin`**, paste `compose/jellyfin/docker-compose.yml`, and click **Save → Up**.
3. Open **`http://homecloud.lan:8096`**, complete the setup wizard, and add `/media/Movies` and `/media/TV` as your libraries.

### Enable Intel QuickSync (QSV) Hardware Acceleration
1. Open Jellyfin **Dashboard → Playback → Transcoding**.
2. Set **Hardware acceleration** to **`Intel QuickSync (QSV)`** (or **`Video Acceleration API (VAAPI)`** with `/dev/dri/renderD128`).
3. **Important for 6th-Gen Intel CPUs (e.g., i5-6200U / Skylake):**
   * Check **`H264`**, **`MPEG2`**, and **`VC1`**.
   * **Leave `HEVC` and `HEVC 10bit` UNCHECKED** (6th-gen Intel iGPUs lack full 10-bit HEVC hardware decoding and can fail playback if checked; Smart TVs Direct Play HEVC natively).
4. Check **Enable hardware encoding** and click **Save**.

---

## 3. Deploy FileBrowser & Native SMB Network Shares

### Web FileBrowser (`:8085`)
1. Pre-create the empty database and config files (prevents Docker from creating them as directories):
   ```bash
   mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/filebrowser
   touch /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/filebrowser/filebrowser.db
   echo '{"port":80,"baseURL":"","address":"","log":"stdout","database":"/database/filebrowser.db","root":"/srv"}' > /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/filebrowser/settings.json
   ```
2. Deploy `compose/filebrowser/docker-compose.yml` in OMV Compose as **`filebrowser`**.
3. Open **`http://homecloud.lan:8085`** (default login: `admin` / `admin` — change immediately under **Settings → Profile**).

### Windows / Mac SMB Network Drive
1. In OMV, go to **Storage → Shared Folders → Create (+)** and share your `Media` folder.
2. Go to **Services → SMB/CIFS → Settings** → check **Enabled** → **Save**.
3. Go to **Services → SMB/CIFS → Shares → Create (+)** → select the `Media` shared folder → **Save → Apply**.
4. On Windows, open File Explorer and type **`\\<SERVER_LAN_IP>`** (or **`\\homecloud.lan`**) to map it as a network drive.

---

## 🛠️ Troubleshooting Common Phase 3 Issues

### Issue 1: Jellyfin Smart TV App Says `"Unable to Connect"` or `"Server Mismatch"`
* **Cause 1 (Bind Address):** Jellyfin sometimes defaults its bind address to IPv6 `::`, ignoring IPv4 connections from Tailscale or secondary interfaces.
  * **Fix:** Open Jellyfin **Dashboard → Networking** → clear `::` from **Bind to local network address** (set to **`0.0.0.0`** or leave blank) and ensure **Base URL** is completely blank.
* **Cause 2 (Stale Browser/App Cache):** Reinstalling Jellyfin generates a new Server ID, causing browsers or Android apps to fail with `"Server Unavailable"` or `"Server Mismatch"`.
  * **Fix:** On Android, clear **Storage & cache** in App Info. In desktop Chrome, press `F12` → **Console** → run `localStorage.clear(); sessionStorage.clear(); location.reload();`.

### Issue 2: FileBrowser Fails to Start With `is a directory` Error
* **Cause:** If `filebrowser.db` or `settings.json` does not exist on the host before starting the container, Docker creates empty folders with those names instead of files.
* **Fix:** Remove the folders (`rm -rf .../filebrowser.db .../settings.json`), run the `touch` and `echo` commands in Section 3, and restart the container.
