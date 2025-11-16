# pp3-loki

![Build Status](https://github.com/OnyxJeff/pp3-loki/actions/workflows/build.yml/badge.svg)
![Maintenance](https://img.shields.io/maintenance/yes/2025.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![GitHub release](https://img.shields.io/github/v/release/OnyxJeff/pp3-loki)
![Issues](https://img.shields.io/github/issues/OnyxJeff/pp3-loki)

**Loki** is the internal playground server for my homelab, hosted on a Raspberry Pi 4.

## 📁 Repo Structure

```text
pp3-loki/
├── .github/workflows/      # CI for YAML validation
├── backup_logs/            # Oldest logs from update script
├── dockprom/               # Docker container(s) for Prometheus Node Exporter for RPi
├── images/                 # Images for README files
├── logs/                   # Most recent update script logs
├── scripts/                # Auto-Updater script for RPi (can be associated with cronjob)
├── U6143_ssd1306/          # Python, C code, and script for UCTronics display screen
└── README.md               # You're reading it!
```

---

## 🧰 Services
- **Transmission**: Fast, lightweight BitTorrent client that handles downloads without getting in your way.

---

## 🖥️ Installing U6143_ssd1306 Display

- Preparation

  - Install GIT (app used to download this repo onto your device)
  ```bash
  sudo apt install git -y
  ```

  - Download repo
  ```bash
  cd
  git clone https://github.com/OnyxJeff/pp3-loki.git
  ```

```bash
sudo raspi-config
```
Choose Interface Options Enable i2c

- Run setup_display_service.sh script
```bash
cd ~/pp3-loki/U6143_ssd1306
chmod +x setup_display_service.sh
sudo ./setup_display_service.sh
```

- Custom display temperature type
  - Open the U6143_ssd1306/C/ssd1306_i2c.h file. You can modify the value of the TEMPERATURE_TYPE variable to change the type of temperature displayed. (The default is Fahrenheit)
  ![Select Temperature](images/select_temperature.jpg)

- Custom display IPADDRESS_TYPE type
  - Open the U6143_ssd1306/C/ssd1306_i2c.h file. You can modify the value of the IPADDRESS_TYPE variable to change the type of IP displayed. (The default is ETH0)
  ![Select IP](images/select_ip.jpg)

- Custom display information
  - Open the U6143_ssd1306/C/ssd1306_i2c.h file. You can modify the value of the IP_SWITCH variable to determine whether to display the IP address or custom information. (The custom IP address is displayed by default)
  ![Custom Display](images/custom_display.jpg)

---

## ⚠️ Updating the OS

- Update and Upgrade the System via script:
```bash
cd ~/pp3-loki/scripts
chmod +x apt-get-autoupdater.sh
sudo ./apt-get-autoupdater.sh
```

- Start CronJob (optional but recommended if doing headless/always on installation)
```bash
sudo crontab -e
```

  - add the following to the bottom of the document:
  ```bash
  # OS-Auto-Updater
    00 01 * * 0 bash $HOME/pp3-loki/scripts/apt-get-autoupdater.sh
      # execute automatic update script and log every sunday at 01:00 am
    50 00 1 * * /bin/bash -c 'cp $HOME/pp3-loki/logs/apt-get-autoupdater.log $HOME/pp3-loki/backup_logs/apt-get-autoupdater-$(date +\%Y\%m\%d).log'
      # saves monthly version of "apt-get-autoupdater.log" on the 1st of every month at 00:50 am
    51 00 1 * * rm -f $HOME/pp3-loki/logs/apt-get-autoupdater.log
      # deletes old weekly log on the 1st of every month at 00:51 am
  ```

## 📦 Installing Docker Compose

- Install Docker
```bash
cd
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

- Add User to Docker Group
```bash
sudo usermod -aG docker $USER
```

> [!IMPORTANT]
> After running this command you will need to log out and log back in (or I recommend just rebooting) for the changes to take effect.

- Install Docker Compose:
```bash
sudo apt install docker-compose-plugin
```

- Verify Installation:
```bash
docker run hello-world
docker compose version
```

---

### 📝 Installing your first container(s)

- Installing Dockprom (Prometheus Exporter)
```bash
cd ~/pp3-loki/dockprom
docker compose up -d
```

---

### 🗂️ Setting up an NFS shared folder

- Installing NFS client
```bash
sudo apt install nfs-common
```
- Creating a Mount Point
```bash
sudo mkdir -p /media/nfs_share
```

- Creating .mount file
```bash
sudo nano /etc/systemd/system/media-nfs_share.mount
```

- Paste the following into file:
```bash
[Unit]
Description=NFS Mount for Transmission
After=network.target

[Mount]
What="<NAS IP Address>":/"<server>"/nfs_share
Where=/media/nfs_share
Type=nfs
Options=_netdev,auto,hard,intr,noatime

[Install]
WantedBy=multi-user.target
```

- Enable and start the .mount unit
```bash
sudo systemctl daemon-reload
sudo systemctl enable media-nfs_share.mount
sudo systemctl start media-nfs_share.mount
```

- Check if the NFS share is mounted successfully
```bash
mount | grep nfs_share
```

### 🧲 Installing and Setting up Transmission

- Run the command below to install *transmission* to the Raspberry Pi.
```bash
sudo apt install transmission-daemon
```

- Stop the service temporarily by running the command below.
```bash
sudo systemctl stop transmission-daemon
```

- Create two different folders in our newly created share
```bash
sudo mkdir -p /media/nfs_share/torrent-inprogress
sudo mkdir -p /media/nfs_share/torrent-complete
```

- Give your User Access to the folders
```bash
sudo chown -R $USER:$USER /media/nfs_share/torrent-inprogress
sudo chown -R $USER:$USER /media/nfs_share/torrent-complete
```

- Edit Transmission's config file
```bash
sudo nano /etc/transmission-daemon/settings.json
```

  - modify the following configuration options:
  ```bash
  "incomplete-dir": "/media/nfs_share/torrent-inprogress",
  ```
  ```bash
  "download-dir": "/media/nfs_share/torrent-complete",
  ```
  ```bash
  "incomplete-dir-enabled": true,
  ```
  ```bash
  "rpc-password": "Your_Password",
  ```
  ```bash
  "rpc-username": "Your_Username",
  ```
  > [!NOTE]
  > This IP should be that of your home network
  ```bash
  "rpc-whitelist": "192.168.*.*",
  ```
  
  - Once you have finished editing the file you can save it by pressing ```CTRL``` + ```X```, then ```Y```, followed by the ```ENTER``` key.

- edit the Transmission daemon startup script so that it uses your user instead of the default “debian-transmission” user.
```bash
sudo nano /etc/init.d/transmission-daemon
```

  - edit the “USER=” line, so that the Transmission daemon will be run by your user and not the “debian-transmission” user that is setup by default.
  ```bash
  USER=<YOURUSERNAME>
  ```

- change the user from “debian-transmission” to your username in the service file
```bash
sudo nano /etc/systemd/system/multi-user.target.wants/transmission-daemon.service
```

  - change the “User=” line so that it points to your user instead.
  ```bash
  User=<YOURUSERNAME>
  ```

- reload all service configuration files by running the following command.
```bash
sudo systemctl daemon-reload
```

- take ownership of the “/etc/transmission-daemon” folder.
```bash
sudo chown -R $USER:$USER /etc/transmission-daemon
```

- create a directory where the transmission-daemon will access the “setting.json” file.
```bash
sudo mkdir -p /home/$USER/.config/transmission-daemon/
sudo ln -s /etc/transmission-daemon/settings.json /home/$USER/.config/transmission-daemon/
sudo chown -R $USER:$USER /home/$USER/.config/transmission-daemon/
```

- Start the Transmission daemon service
```bash
sudo systemctl start transmission-daemon
```

- Reboot to enable settings
```bash
sudo shutdown now -r
```

- Check out Transmission's web interface by going to the Raspberry Pi’s IP address followed by the port ```:9091```

- Replace “<IPADDRESS>” in the URL below with your Pi’s local IP address to go to Transmission’s web interface.
```bash
http://<IPADDRESS>:9091
```

---

## Acknowledgements

This project uses or is inspired by the following repositories:

- [U6143_ssd1306](https://github.com/UCTRONICS/U6143_ssd1306) – Provides the C display code used in the systemd service setup.
- [Dockprom](https://github.com/stefanprodan/dockprom) – Used for Docker-based Prometheus monitoring and metrics collection.
- [Transmission](https://pimylifeup.com/raspberry-pi-transmission/) - Initial instructions on installing Transmission-Daemon on a Raspberry Pi using a network share as it's download folders.

---

📬 Maintained By
Jeff M. • [@OnyxJeff](https://www.github.com/onyxjeff)