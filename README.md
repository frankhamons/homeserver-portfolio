# Frank Hamons — HomeServer Project

> **Live portfolio:** [frankhamons.github.io/homeserver-portfolio](https://frankhamons.github.io/homeserver-portfolio)

A self-hosted home server built from scratch on a repurposed consumer laptop — custom dashboard, AI voice assistant, photo/media library, document management, printer/scanner integration, and real security hardening. Every line of the dashboard was hand-written, no framework, no template.

---

## Hardware

| Component | Spec |
|-----------|------|
| Machine | ASUS Vivobook X1605ZA |
| CPU | Intel i7-1255U (10-core) |
| GPU | Intel Iris Xe (Quick Sync hardware transcoding) |
| RAM | 16GB DDR4 |
| System Drive | 512GB NVMe |
| Expansion Drive | 4TB Seagate (exFAT, udev automount) |
| OS | Ubuntu 24.04 Server |
| Remote Access | Tailscale VPN |

---

## What It Does

### 🏠 HANK Dashboard
Custom web dashboard (Node.js/Express backend, vanilla HTML/CSS/JS frontend) served over HTTPS via Tailscale certificate. Runs as a systemd service. Features:

- Live CPU/RAM/temperature monitoring with ring gauges
- Storage usage across all drives
- Real-time network activity graph
- Connected devices list (arp-scan with editable names)
- Services status panel
- Security monitor (live failed SSH attempt tracking from auth.log)
- Weather (Marysville, CA)
- Persistent music player bar

### 🎙️ HANK AI Assistant
Voice + chat assistant powered by Groq's Llama 3.3 70B. Streams responses sentence-by-sentence for low latency. Voice synthesis via self-hosted Edge-TTS container (ChristopherNeural voice) with an indexed audio-slot queue guaranteeing in-order playback. Microphone input works on mobile browsers via Web Speech API over HTTPS.

### 📸 Photos (Immich)
Immich with separate libraries per family member — Frank (108 photos) and Janet (5,522 photos) each have their own admin account so collections stay independent. Auto-backup from phone camera rolls. Photos browsable directly in the dashboard.

### 📺 Movies & TV (Plex)
Plex Media Server in Docker with Intel Iris Xe Quick Sync hardware transcoding (verified via active session logs, not just settings). Accessible from anywhere including external sharing.

### 🎵 Music (Navidrome + Custom Player)
710-song library with a hand-built dashboard music player: shuffle/repeat, scrubbing, volume, genre filtering, sorted newest-first. Navidrome also available for external music apps.

### 📄 Documents
Full file manager for `/mnt/expansion/Documents`: breadcrumb navigation, thumbnail previews (images and PDF first-page via Ghostscript/ImageMagick), inline image preview modal, PDF opens in new tab, move (with navigable folder browser modal), delete, new folder creation.

### 🖨️ Print / Scan
Full integration with Epson WF-2950 over WiFi:
- **Flatbed scanning** via SANE's eSCL backend with resolution (150/300/600 DPI) and color mode (Color/Grayscale/B&W) controls
- **Phone camera scanning** via native camera capture (full-resolution)
- **Direct printing** via CUPS, with Word→PDF auto-conversion
- **Live ink level monitoring** (Black/Cyan/Magenta/Yellow)
- Recent scans listed with View button

### 📁 Files
General file browser rooted at `/mnt/expansion` — covers all top-level folders (Music, Photos, Movies, TV Shows, Documents, SOFTWARE, etc.). Upload, move, delete, download, thumbnail previews for images and PDFs, new folder creation.

### 🔒 Security & Monitoring
- **UFW firewall** — default deny incoming; only necessary ports open, internal services scoped to LAN + Tailscale only
- **Fail2Ban** — sshd jail, 3 attempts → 24-hour ban, fires GeoIP lookup + ntfy push notification on every ban event
- **SSH key authentication** — ED25519 keys, password login disabled
- **Tailscale VPN** — dashboard and internal services not publicly exposed at all
- **ntfy alerts** — instant phone push notifications for CPU temperature tiers (85°C/90°C/95°C), RAM >90%, disk >90%, container downtime, and Fail2Ban bans
- **Security panel** — live failed SSH attempt tracking from auth.log

---

## Tech Stack

| Category | Technology |
|----------|-----------|
| OS | Ubuntu 24.04 Server |
| Backend | Node.js + Express |
| Frontend | Vanilla HTML/CSS/JS (no framework) |
| Containers | Docker + docker-compose |
| Photos | Immich |
| Media | Plex (hardware transcoding via Intel Quick Sync) |
| Music | Navidrome |
| AI Assistant | Groq API (Llama 3.3 70B) |
| Voice Synthesis | Edge-TTS (self-hosted FastAPI container) |
| Remote Access | Tailscale |
| Firewall | UFW |
| Intrusion Detection | Fail2Ban |
| Push Alerts | ntfy (self-hosted) |
| Printing | CUPS + IPP |
| Scanning | SANE + eSCL |
| PDF Thumbnails | ImageMagick + Ghostscript |
| Network Scanner | arp-scan |
| Service Manager | systemd |

---

## Docker Services

| Service | Port | Purpose |
|---------|------|---------|
| immich-server | 2283 | Photo library |
| immich-machine-learning | — | AI features |
| immich-redis | — | Internal cache |
| immich-postgres | — | Database |
| navidrome | 4533 | Music streaming |
| plex | 32400 | Media streaming |
| stirling-pdf | 3004 | PDF tools |
| hank-voice | 5002 | Edge-TTS voice synthesis |
| ntfy | 8090 | Push notifications |

---

## Disaster Recovery

Full versioned backup via tar.gz covering:
- `/opt/homeserver/` — all service configs, dashboard code (server.js, index.html, .env), docker-compose.yml
- `/etc/systemd/system/homeserver-dashboard.service`
- `/etc/sudoers.d/arp-scan`

**Restore:** `sudo tar -xzf homeserver-backup-*.tar.gz -C /` → `cd /opt/homeserver && docker compose up -d` → `sudo systemctl daemon-reload && sudo systemctl enable --now homeserver-dashboard`

---

## Project Notes

- Built after a critical data-loss incident (Python `UnicodeEncodeError` truncated `index.html` to zero bytes mid-write) — rebuilt from scratch and implemented the backup system as a direct result
- Dashboard frontend is hand-written vanilla JS/HTML/CSS — no React, no Vue, no build step
- Hardware transcoding verified live via Plex transcoder session logs, not just settings
- Fail2Ban was already installed; this project added the GeoIP + ntfy alert action on ban

---

## Contact

**Frank Hamons** — Marysville, California  
✉ Hamons8@hotmail.com
