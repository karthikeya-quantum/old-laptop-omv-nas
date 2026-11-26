# Phase 2: Networking, Split DNS (Pi-hole) & Remote Access (Tailscale)

In this phase, you will lock in a static local IP address, deploy **Pi-hole** to block ads across every device on your Wi-Fi, create clean custom `.lan` URLs (like `http://homecloud.lan:7575`), fix Docker container DNS resolution permanently, and install **Tailscale** so the exact same `.lan` links work securely from anywhere in the world—with **zero router port forwarding**.

---

## 1. Set a Static LAN IP Address

1. Log into your home Wi-Fi router's admin page (usually `http://192.168.0.1` or `http://192.168.1.1`).
2. Find **DHCP Reservation / Address Reservation / Static IP Binding**.
3. Bind your server's Ethernet MAC address to a fixed IP (referred to as `<SERVER_LAN_IP>` throughout this guide, e.g., `192.168.0.113`).

---

## 2. Disable `systemd-resolved` Stub Listener (Frees Port `53` for Pi-hole)

Linux reserves port `53` for its local stub resolver by default, which causes Pi-hole to fail on startup with `address already in use`.

Run this in **OMV → Services → CTerm** (or SSH):

```bash
sudo mkdir -p /etc/systemd/resolved.conf.d
sudo tee /etc/systemd/resolved.conf.d/pihole.conf > /dev/null << 'EOF'
[Resolve]
DNS=1.1.1.1 8.8.8.8
DNSStubListener=no
EOF
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
sudo systemctl restart systemd-resolved
```

### Expected Output
Verify nothing is listening on port `53`:
```bash
sudo ss -tulpn | grep :53
# Expected: empty output (port 53 is now free for Pi-hole)
```

---

## 3. Deploy Pi-hole in OpenMediaVault

1. Create the configuration directory:
   ```bash
   mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata/pihole/etc-pihole
   ```
2. In **OMV → Services → Compose → Files → Create (+)**, name the stack **`pihole`** and paste `compose/pihole/docker-compose.yml` (replacing `<YOUR_PIHOLE_ADMIN_PASSWORD>` and `<APPDATA_UUID>`).
3. Click **Save**, select **`pihole`**, and click **Up** (play icon).

### Configure Custom `.lan` Local DNS & Router DHCP
1. Open Pi-hole at **`http://<SERVER_LAN_IP>:8081/admin`** and log in with your password.
2. Go to **Settings → Local DNS Records** and add:
   * **Domain:** `homecloud.lan` → **IP Address:** `<SERVER_LAN_IP>`
   * *(Later, once Tailscale is installed, also add a second record: `homecloud.lan` → `<SERVER_TAILSCALE_IP>` so Split DNS works seamlessly.)*
3. Log into your Wi-Fi router's **DHCP Settings** and set **Primary DNS** to `<SERVER_LAN_IP>` (and **Secondary DNS** to `1.1.1.1` as a fallback). Reconnect Wi-Fi on your phone/laptop to pick up the new DNS.

---

## 4. Fix Docker Container DNS Globally (`ENOTFOUND homecloud.lan`)

By default, Docker containers bypass your router's DHCP DNS and query external servers (`8.8.8.8`), causing dashboards and uptime monitors inside Docker to fail when pinging `homecloud.lan`.

Configure Docker's daemon globally to query your Pi-hole first:

```bash
sudo tee /etc/docker/daemon.json > /dev/null << 'EOF'
{
  "dns": ["<SERVER_LAN_IP>", "1.1.1.1"]
}
EOF
sudo systemctl restart docker
```

---

## 5. Install Tailscale & Enable Split DNS (Remote Access Without Port Forwarding)

1. Install Tailscale directly on the OpenMediaVault host:
   ```bash
   curl -fsSL https://tailscale.com | sh
   sudo tailscale up
   ```
2. Open the authentication URL printed in the terminal, sign in, and note your server's Tailscale IP (`<SERVER_TAILSCALE_IP>`, e.g., `100.x.x.x`) via:
   ```bash
   tailscale ip -4
   ```
3. In Pi-hole (**Settings → Local DNS Records**), add a second entry for `homecloud.lan` pointing to `<SERVER_TAILSCALE_IP>`.
4. In the **Tailscale Admin Console** (`https://tailscale.com`):
   * Click **Add nameserver → Custom**.
   * Enter `<SERVER_TAILSCALE_IP>` as the nameserver.
   * Enable **Restrict to domain (Split DNS)** and enter **`lan`**.
   * Click **Save**.

Now, whether you are at home on Wi-Fi or miles away on mobile data with Tailscale enabled, **`http://homecloud.lan:<PORT>`** resolves automatically to your server!

---

## 🛠️ Troubleshooting Common Phase 2 Issues

### Issue 1: Pi-hole Fails to Start (`Bind for 0.0.0.0:53 failed: port is already allocated`)
* **Cause:** `systemd-resolved` is still holding port `53`.
* **Fix:** Re-run the `DNSStubListener=no` steps in Section 2 and check `sudo ss -tulpn | grep :53` before clicking **Up** in OMV Compose.

### Issue 2: `homecloud.lan` Works on Laptop/Phone, But Docker Containers Say `ENOTFOUND`
* **Cause:** Containers do not inherit router DHCP DNS unless configured in `/etc/docker/daemon.json`.
* **Fix:** Ensure `/etc/docker/daemon.json` contains `"dns": ["<SERVER_LAN_IP>", "1.1.1.1"]` and run `sudo systemctl restart docker`.

### Issue 3: Smart TV Apps Cannot Resolve `homecloud.lan`
* **Cause:** Many Smart TV operating systems hardcode `8.8.8.8` and ignore router DHCP DNS settings.
* **Fix:** Always enter the direct IP URL (`http://<SERVER_LAN_IP>:8096`) inside Smart TV apps, while using `http://homecloud.lan:<PORT>` on phones, tablets, and laptops.
