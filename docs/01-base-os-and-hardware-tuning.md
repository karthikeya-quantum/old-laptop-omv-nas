# Phase 1: Base OS, 24/7 Laptop Tuning & Storage Setup

In this phase, you will install **OpenMediaVault (OMV)** on your old laptop or PC, prevent the laptop from sleeping when the lid is closed, protect the battery from swelling while plugged in 24/7, mount your storage drives, and enable Docker.

---

## 1. Install OpenMediaVault (OMV)

1. Download the official **OpenMediaVault ISO** from [openmediavault.org](https://openmediavault.org).
2. Flash the ISO onto a USB pendrive using **Rufus** (Windows) or **BalenaEtcher** (Windows/Mac/Linux).
3. Plug an **Ethernet cable** from your Wi-Fi router directly into the laptop (required during installation).
4. Boot the laptop from the USB drive (`F12` / `F2` / `Del` at startup), follow the installer prompts, set a strong **root password**, and unplug the USB drive when prompted to reboot.

### Expected Output After Reboot
Once booted, the laptop screen will display your server's local IP address (for example: `http://192.168.0.113`).
* Open that IP in your browser on another computer.
* **Default OMV Web Login:**
  * **Username:** `admin`
  * **Password:** `openmediavault` *(change this immediately under **Account → Change Password**)*

---

## 2. Prevent Laptop Sleep When Closing the Lid

By default, Linux puts a laptop to sleep the moment you close the lid—which will disconnect your entire home cloud.

Open **Services → CTerm** (or SSH into `root@<SERVER_LAN_IP>`) and run:

```bash
sudo sed -i 's/^#\?HandleLidSwitch=.*/HandleLidSwitch=ignore/' /etc/systemd/logind.conf
sudo sed -i 's/^#\?HandleLidSwitchExternalPower=.*/HandleLidSwitchExternalPower=ignore/' /etc/systemd/logind.conf
sudo sed -i 's/^#\?HandleLidSwitchDocked=.*/HandleLidSwitchDocked=ignore/' /etc/systemd/logind.conf
sudo systemctl restart systemd-logind
```

### Verification
Close the laptop lid physically and ping `<SERVER_LAN_IP>` from another device—it should continue responding with zero interruption.

---

## 3. Enable Battery Conservation Mode (Prevents Battery Swelling)

Keeping a laptop plugged into AC power 24/7 at 100% charge degrades and swells lithium-ion batteries over time. Enabling **Conservation Mode / Charge Thresholds** caps charging around **60%–80%** while running directly on AC power—leaving the battery healthy as a built-in UPS during power cuts.

### For Lenovo Laptops (`ideapad_acpi` / `thinkpad_acpi`)
Run this in CTerm / SSH:

```bash
# Check if Lenovo conservation mode is supported
ls /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode 2>/dev/null || ls /sys/class/power_supply/BAT*/charge_control_end_threshold 2>/dev/null
```

If `conservation_mode` exists, enable it permanently across reboots using a systemd service:

```bash
sudo tee /etc/systemd/system/battery-conservation.service > /dev/null << 'EOF'
[Unit]
Description=Enable Lenovo Battery Conservation Mode
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo 1 > /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable --now battery-conservation.service
```

### Expected Output
```bash
cat /sys/bus/platform/drivers/ideapad_acpi/VPC2004:00/conservation_mode
# Output: 1
```
*(For Dell, ASUS, or HP laptops, check `/sys/class/power_supply/BAT0/charge_control_end_threshold` or set the battery charge limit to 80% inside the laptop's BIOS.)*

---

## 4. Install OMV-Extras, Docker & Compose Plugin

1. In the OMV Web UI, go to **System → Update Management → Updates** and install all available system updates first.
2. Open **CTerm** (or SSH) and install **OMV-Extras**:
   ```bash
   wget -O - https://github.com | bash
   ```
3. Refresh your OMV browser tab (`Ctrl + F5`).
4. Go to **System → Plugins**, search for and install:
   * **`openmediavault-compose`** (manages all Docker containers cleanly from the OMV GUI)
   * **`openmediavault-cterm`** (adds a built-in web terminal inside OMV)
5. Go to **Services → Compose → Settings**:
   * Set **Docker storage** to a dedicated folder on your fast AppData SSD (e.g., `/srv/dev-disk-by-uuid-<APPDATA_UUID>/docker`) so container images never fill up your OS drive.
   * Click **Save** → **Apply** → **Reinstall Docker** (if Docker is not yet running).

---

## 5. Format & Mount Your Storage Drives

1. Go to **Storage → Disks** to verify all connected internal and USB SSDs/HDDs appear.
2. *(Only for brand-new or empty drives)* Go to **Storage → File Systems → Create (+)** → choose **EXT4** → select the drive and format it.
3. Go to **Storage → File Systems**, select each drive, and click **Mount** (play icon) → click the yellow **Apply** banner.
4. Copy the **Mount Point** paths (for example: `/srv/dev-disk-by-uuid-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`)—you will use these exact UUID paths for `<APPDATA_UUID>` and `<MEDIA_UUID>` in all upcoming phases.

### Create Standard Base Folders
Run this in CTerm (replace `<APPDATA_UUID>` and `<MEDIA_UUID>` with your actual drive UUIDs):

```bash
mkdir -p /srv/dev-disk-by-uuid-<APPDATA_UUID>/appdata
mkdir -p /srv/dev-disk-by-uuid-<MEDIA_UUID>/Media/{Movies,TV,Photos,Backups,PersonalFiles}
```

---

## 🛠️ Troubleshooting Common Phase 1 Issues

### Issue 1: USB Drive Disconnects or Fails to Mount on Boot
* **Cause:** Some USB 3.0 enclosures use UAS (USB Attached SCSI) drivers that drop connections under Linux, or USB drives spin up slower than systemd's boot check.
* **Fix:** Ensure external drives are plugged into a powered USB 3.0 port and mounted via OMV's GUI (`Storage → File Systems`), which automatically adds `nofail` to `/etc/fstab` so the server still boots cleanly even if a drive is unplugged.

### Issue 2: `Docker` Fails to Start After Reboot Because Storage Drive Mounted Late
* **Cause:** Docker starts before external USB drives finish mounting.
* **Fix:** Add a systemd drop-in override so Docker waits for local file systems:
  ```bash
  sudo mkdir -p /etc/systemd/system/docker.service.d
  sudo tee /etc/systemd/system/docker.service.d/wait-for-mounts.conf > /dev/null << 'EOF'
  [Unit]
  After=local-fs.target
  Requires=local-fs.target
  EOF
  sudo systemctl daemon-reload
  ```
