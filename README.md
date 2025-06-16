# Loki

![Build Status](https://github.com/OnyxJeff/pp3-loki/actions/workflows/build.yml/badge.svg)
![Maintenance](https://img.shields.io/maintenance/yes/2025.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)
![GitHub release](https://img.shields.io/github/v/release/OnyxJeff/pp3-loki)
![Issues](https://img.shields.io/github/issues/OnyxJeff/pp3-loki)

**Loki** is my VPN-protected torrent server built on a Raspberry Pi 4.

## 📁 Repo Structure

```text
loki/
├── .github/workflows/    # CI for YAML validation
├── backups/              # Exported or example snapshot files
├── docker/               # YAML-based -darr stack applications
└── README.md             # You're reading it!
```

---

## 🧰 Services
- **Transmission**: Lightweight and powerful BitTorrent client.
- **MullvadVPN**: Keeps traffic encrypted and anonymous.

### 📦 Docker Compose

```bash
cd docker/transmission-vpn
docker-compose up -d
```

Be sure to configure your Mullvad credentials in the .env file or bind them securely via Docker secrets.

---

## 💾 Backup

```bash
bash backups/backup.sh
```

Backs up Transmission configs and session data.

---

📬 Maintained By
Jeff M. • [@OnyxJeff](https://www.github.com/onyxjeff)