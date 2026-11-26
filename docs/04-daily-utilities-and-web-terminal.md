# Phase 4: Daily Utilities & One-Click Web SSH Terminal

In this phase, you will replace ad-filled online tools with private, self-hosted utilities running locally on your server, plus a **one-click browser SSH terminal** accessible right from your dashboard on any laptop or phone:
* **Stirling-PDF** (`:8089`) — Merge, split, compress, OCR, sign, and convert PDFs locally
* **ConvertX** (`:3009`) — Convert 1,000+ image, video, audio, document, and ebook formats locally
* **WeTTY Web SSH Terminal** (`:3011`) — Full-screen browser terminal with direct `root` password login

---

## 1. Deploy Stirling-PDF (All-in-One Local PDF Toolkit)

1. Create the persistent directories on your AppData SSD:
   ```bash
   mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/stirling-pdf/{trainingData,configs}
   ```
2. In **OMV → Services → Compose → Files → Create (+)**, name the stack **`stirling-pdf`** and paste `compose/stirling-pdf/docker-compose.yml` (replacing `<APPDATA_UUID>`).
3. Click **Save**, select **`stirling-pdf`**, and click **Up** (play icon).
4. Open **`http://homecloud.lan:8089`** — all PDF tools load immediately with zero login friction on your home network (`DOCKER_ENABLE_SECURITY=false`). *(Keep port `8089` strictly LAN/Tailscale-only; never expose it to the public internet.)*

---

## 2. Deploy ConvertX (1,000+ Format Local File Converter)

1. Create the data directory:
   ```bash
   mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/convertx
   ```
2. In **OMV → Services → Compose → Files → Create (+)**, name the stack **`convertx`** and paste `compose/convertx/docker-compose.yml` (replacing `<APPDATA_UUID>` and `<CHANGE_TO_A_LONG_RANDOM_SECRET>`).
3. Click **Save**, select **`convertx`**, and click **Up** (play icon).
4. Open **`http://homecloud.lan:3009`** and register your admin account on the first screen (public registration automatically locks after the first account is created).

---

## 3. Set Up a One-Click Browser SSH Terminal (`openmediavault-wetty`)

Instead of opening a desktop terminal and typing `ssh root@<SERVER_LAN_IP>` every time, OpenMediaVault's official **`openmediavault-wetty`** plugin provides a full web terminal you can launch in one click from your dashboard.

### Step A: Create a Self-Signed SSL Certificate in OMV
`openmediavault-wetty` runs behind a built-in Caddy HTTPS proxy (`3011 -> 8443`) and requires an SSL certificate before its container will stay running:
1. Open **OMV → System → Certificates → SSL → Create (+)**.
2. Fill in **Common Name** (`homecloud.lan`), select your **Country**, and click **Create** → **Apply**.

### Step B: Enable SSH & Install `openmediavault-wetty`
1. Go to **OMV → Services → SSH**, ensure **Enabled** and **Permit root login** are checked, and click **Save → Apply**.
2. Go to **OMV → System → Plugins**, search for **`wetty`**, select **`openmediavault-wetty`**, and click **Install**.
3. Go to **OMV → Services → WeTTY**:
   * Check **Enabled**.
   * Set **Port** to **`3011`**.
   * Select your **`homecloud.lan`** SSL certificate under **Certificate**.
   * Click **Save** → **Apply**.

### Step C: Fix the WeTTY v3 `ALLOWEDORIGINS` Requirement
Newer `wettyoss/wetty` images refuse to start (`502 Bad Gateway`) unless `ALLOWEDORIGINS` is explicitly passed. Run this one-time patch in **CTerm** / SSH:

```bash
sudo omv-salt deploy run wetty
sudo sed -i 's|docker.io/wettyoss/wetty|-e ALLOWEDORIGINS=https://homecloud.lan:3011,https://<SERVER_LAN_IP>:3011,https://<SERVER_TAILSCALE_IP>:3011 docker.io/wettyoss/wetty|g' /etc/systemd/system/container-wetty-app.service
sudo systemctl daemon-reload
sudo systemctl restart pod-wetty.service
```

### Step D: Direct One-Click Login URL (Skips Username Prompt)
Use this exact URL in your browser or dashboard tile so WeTTY skips `Enter your username:` and jumps straight to the `root` password prompt:
**`https://homecloud.lan`**
*(Accept the self-signed certificate warning once via **Advanced → Proceed**.)*

---

## 🛠️ Troubleshooting Common Phase 4 Issues

### Issue 1: ConvertX Logs You Out Immediately or Fails Over `http://homecloud.lan:3009`
* **Cause:** By default, ConvertX sets `Secure` cookies that browsers ignore over plain `http://`.
* **Fix:** Ensure `HTTP_ALLOWED=true` is present under `environment:` in `compose/convertx/docker-compose.yml`.

### Issue 2: WeTTY Shows `502 Bad Gateway` After Installing the Plugin
* **Cause:** `wetty-app` exited immediately because `ALLOWEDORIGINS` was missing from OMV's auto-generated systemd unit.
* **Fix:** Verify `sudo podman ps -a` shows both `wetty-proxy` and `wetty-app` as `Up`. If `wetty-app` is missing, re-run the `sed` command in Step C (note: if you ever click Save/Apply on OMV's **Services → WeTTY** page later, re-run that `sed` command since OMV regenerates the service file).
