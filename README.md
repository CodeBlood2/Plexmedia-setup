# 🍿 Plex Media Server Setup on Raspberry Pi

A step-by-step guide to setting up a **Plex Media Server** on your Raspberry Pi. This includes OS installation, remote access, HDD mounting, Samba file sharing, network configuration, and Plex installation.

---

## 📋 Requirements
- Raspberry Pi 3/4 with Raspberry Pi OS (PiOS)
- microSD card (16GB+ recommended)
- External HDD/SSD or large USB drive for media storage
- Stable Wi-Fi or Ethernet connection
- [Plex Account](https://www.plex.tv)
- Optional: Monitor, keyboard, and mouse for first boot

---

## 🚀 1. Install Raspberry Pi OS (PiOS)

1. Install **Raspberry Pi Imager** (Windows/macOS/Linux).
2. Select:
   - **OS** → Raspberry Pi OS (64-bit)
   - **Storage** → your microSD card
3. Click ⚙️ (Advanced Options) and configure:
   - Hostname (e.g., `raspberrypi.local`)
   - Username & password
   - Wi-Fi, Locale & Timezone (if needed)
   - Enable SSH
4. Write the image → eject SD card → insert into the Pi → power on.

---

## 🔧 2. First Boot & System Updates

Connect via SSH (or attach monitor/keyboard for first boot):

```bash
ssh YOUR_USER@raspberrypi.local
```

Update the system:

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 🖥️ 3. (Optional) Enable VNC for GUI Access

1. Install VNC:
   ```bash
   sudo apt install realvnc-vnc-server realvnc-vnc-viewer
   ```
2. Enable in config:
   ```bash
   sudo raspi-config
   ```
   → **Interface Options → VNC → Enable**
3. Install **VNC Viewer** on your PC and connect with Pi’s IP.

---

## 💾 4. Mount External HDD/SSD

1. Find drive UUID:
   ```bash
   lsblk -f
   ```
   Example output:
   ```
   /dev/sda1: LABEL="TJ_ColdST" UUID="XXXX-XXXX" TYPE="exfat"
   ```
2. Create mount point:
   ```bash
   sudo mkdir -p /media/TJ_ColdST
   ```
3. Edit `/etc/fstab`:
   ```
   UUID=XXXX-XXXX  /media/TJ_ColdST  exfat  defaults,uid=1000,gid=1000,umask=002,nofail,x-systemd.automount  0  0
   ```
4. Apply:
   ```bash
   sudo mount -a
   ```

---

## 📥 5. Install Plex Media Server

1. Add Plex repo:
   ```bash
   curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -
   echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
   sudo apt update
   ```
2. Install Plex:
   ```bash
   sudo apt install plexmediaserver -y
   ```
3. Verify service:
   ```bash
   systemctl status plexmediaserver
   ```

---

## 🌐 6. Access Plex Web Interface

Open a browser and go to:

```
http://<raspberrypi-ip>:32400/web
```

Login with your Plex account.

---

## 📂 7. Add Media Library

1. Open Plex Web → **Libraries → Add Library**
2. Choose type (Movies/TV Shows/Music)
3. Point to the mounted HDD folder

---

## 🔐 8. File Sharing with Samba

To make your HDD accessible across your local network:

1. Install Samba:
   ```bash
   sudo apt install samba samba-common-bin -y
   ```
2. Edit config file:
   ```bash
   sudo nano /etc/samba/smb.conf
   ```
   Add at the bottom:
   ```
   [Shared]
   path = /media/TJ_ColdST
   writeable = yes
   create mask = 0777
   directory mask = 0777
   public = yes
   guest ok = no
   ```
3. Restart Samba:
   ```bash
   sudo systemctl restart smbd
   ```
4. Create a Samba user:
   ```bash
   sudo smbpasswd -a pi
   ```

Now, your drive can be accessed from other devices using:
```
\\raspberrypi\Shared
```

---

## 🌍 9. Network Configuration

### Wi-Fi
If you didn’t set Wi-Fi during flashing:
```bash
sudo raspi-config
```
→ **System Options → Wireless LAN**

### Static IP (recommended for Plex)
Edit DHCP config:
```bash
sudo nano /etc/dhcpcd.conf
```
Add:
```
interface wlan0
static ip_address=192.168.1.50/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```
Restart service:
```bash
sudo systemctl restart dhcpcd
```

---

## 🧪 10. Testing

- Play a video locally in Plex.
- Install Plex mobile app and connect to your server.
- Test remote access (if configured).

---

## ⚡ Extra Tweaks
- Enable **hardware transcoding** (if Pi model supports it)
- Optimize library scanning
- Configure **Remote Access** in Plex settings (may require port-forwarding)

---

## 📝 Troubleshooting

- **Green light stuck on Pi** → check SD card or power supply
- **Drive not visible** → verify UUID in `fstab`
- **Slow transfers** → prefer Ethernet over Wi-Fi, optimize Samba config
- **SSH issues** → clear old host key with:
  ```bash
  ssh-keygen -R raspberrypi.local
  ```

---

## 🎉 Done!

Your Raspberry Pi is now a **home media hub** with Plex! Stream your collection anywhere on your network (and remotely if enabled).



