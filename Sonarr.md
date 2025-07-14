---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-14T05:02:16.255Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

# ![Sonarr](/sonarr.png){class="tab-icon"} What is Sonarr?

**Sonarr** is a TV-series PVR for Usenet and BitTorrent users. It monitors RSS feeds for new episodes, grabs, sorts, and renames them, and upgrades quality when better releases appear.

> 📌 Works great with **qBittorrent**, **Prowlarr**, and **Jellyfin / Plex** for fully automated TV downloads.

---

# 1 · Deploy Sonarr

> **Start by setting up your datasets, then choose your install method.**

# tabs {.tabset}

## 📂 Folder Setup

> Create the following datasets in **TrueNAS** or match these paths in **Docker volumes**.

| Dataset               | Mount Path in App | Description             |
| --------------------- | ----------------- | ----------------------- |
| `tank/configs/sonarr` | `/config`         | Stores Sonarr's config  |
| `tank/media`          | `/media`          | Shared media mount      |
| `tank/media/tv`       | `/media/tv`       | Folder for TV downloads |

```text
/mnt/tank/
├── configs/
│   └── sonarr/
└── media/
    └── tv/
```

> 🔒 Set ownership to `apps(568):apps(568)` (default user/group for SCALE apps). This ensures Sonarr can read/write configs and media.

---

## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/sonarr:/config
      - /mnt/tank/media:/media
    ports:
      - 8989:8989 # Use 127.0.0.1:8989 for reverse proxy setups
    restart: unless-stopped
```

> **Behind a reverse‑proxy?** Expose port **8989** on `127.0.0.1` and route through Nginx Proxy Manager or Cloudflare Tunnel.

---

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

> Use the official TrueNAS Sonarr app with custom host paths.

| Step | Action                                                         |
| ---- | -------------------------------------------------------------- |
| 1    | Apps → Discover Apps → Sonarr → Install                        |
| 2    | Set **Port** to 8989                                           |
| 3    | Sonarr Config Storage → Host Path → `/mnt/tank/configs/sonarr` |
| 4    | Additional Storage → Host Path → `/mnt/tank/media` → `/media`  |
| 5    | Click **Save** → **Deploy**                                    |

---

## <img src="/nginx-proxy-manager.png" class="tab-icon"> NGINX Reverse Proxy

> Configure a reverse proxy (subdirectory or subdomain). Prefer a GUI? See [NGINX Proxy Manager](/nginx) or [Cloudflare Tunnel](/CloudflareTunnels).

### Subdirectory `/sonarr`

```nginx
location ^~ /sonarr {
  proxy_pass http://127.0.0.1:8989;
  proxy_set_header Host $host;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_upgrade;
  proxy_set_header Connection $http_connection;
}
```

### Subdomain `sonarr.yourdomain.tld`

```nginx
server {
  listen 80;
  server_name sonarr.yourdomain.tld;

  location / {
    proxy_pass http://127.0.0.1:8989;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $http_connection;
  }
}
```

---

# 2 · First‑Run Configuration

> **Set folders, connect a downloader, and add indexers. Done in \~5 minutes.**

# tabs {.tabset}

## 📺 Series

<details open><summary><strong>🔍 Series Overview</strong></summary>

* **Library view**: See every show you’re tracking; click one for details or bulk actions.
* **Search bar**: Instantly find a specific series in large libraries.
* **Filters**: Narrow by release date, episode count, disk size, tags, or custom fields.

**Add New Series**

1. Click **Add New**
2. Fill in:

   * **Root Folder** (where files live)
   * **Monitor** (All, Future, Missing, Existing, First Season, Latest Season, None)
   * **Quality Profile** (e.g. 1080p)
   * **Series Type** (standard vs. anime search rules)
   * **Season Folder** (on/off)
   * **Tags** (for release/delay/indexer profiles or organization)
3. Hit **Start Search** to grab missing or cutoff-unmet episodes.

**Library Import**

* Pull in an existing, well-organized media folder.
* Not for download or unstructured folders—must be one series per folder, correctly named.
* Ensure the Sonarr user has R/W on that path, and that downloads and media live in separate folders.

**Mass Editor**

* Select multiple series → bulk-edit monitor settings, profiles, tags, etc.

**Season Pass**

* See at a glance how many seasons you have and which episodes are still missing.

</details>

## 📁 Library

<details open><summary><strong>⚙️ Media Management</strong></summary>

* ✅ Enable **Rename Episodes**
* Include quality and release group in naming
* Enable **Use Hard Links instead of Copy** for performance
* ➕ Add `.srt` to **Import Extra Files**

</details>

<details><summary><strong>📁 Root Folders</strong></summary>

* Add `/media/tv` as a monitored root folder
* Ensure Sonarr has read/write access

</details>

## 🔍 Indexers

<details open><summary><strong>📡 Configure via Prowlarr</strong></summary>

1. Install **[Prowlarr](/Prowlarr)**
2. In Sonarr: **Settings → Apps → +** and connect Prowlarr
3. Add indexers (Jackett, Torznab, etc.) within Prowlarr
4. **Test → Save** — Sonarr will import these indexers automatically

</details>

## 📥 Download Clients

<details open><summary><strong>qBittorrent Setup</strong></summary>

| Field            | Example        |
| ---------------- | -------------- |
| Host             | `10.251.0.244` |
| Port             | `10095`        |
| Username         | `admin`        |
| Password         | ••••••••       |
| Category         | `tv-sonarr`    |
| Remove Completed | ✅              |

> Remote downloader? Use **Path Translation** to map `/downloads` inside qBittorrent to `/media` inside Sonarr.

</details>

## 🎯 Profiles

<details open><summary><strong>Quality Profiles</strong></summary>

* Create or customize profiles with desired sources
* Remove defaults you don't use
* Set **Cutoff** to stop upgrades when target quality is reached

</details>

## 🛠️ Automation

<details open><summary><strong>Recyclarr Setup (Optional)</strong></summary>

* Sync TRaSH naming templates, profiles, and quality settings
* Useful for multiple libraries (4K vs 1080p)
* Place Recyclarr config in Sonarr's `/config` or a separate folder

</details>

---

# 3 · FAQ

<details><summary><strong>Why didn’t Sonarr grab an episode?</strong></summary>

Check RSS sync timing, run a manual search, and ensure indexers are active.

</details>

<details><summary><strong>What do status colors mean?</strong></summary>

* **Blue**: Download in progress
* **Yellow**: Warning (e.g. missing file)
* **Red**: Error

</details>

<details><summary><strong>How to back up and restore?</strong></summary>

* **Backup**: System → Backup → Download .zip
* **Restore**: System → Backup → Restore → Upload .zip

</details>

---

# 4 · Advanced Tweaks *(optional)*

> For power users tuning profiles, metadata, or running multiple instances.

<details><summary><strong>🧩 Media Management Presets</strong></summary>

| Field           | Recommendation                         |
| --------------- | -------------------------------------- |
| Rename Episodes | True                                   |
| Episode Format  | `{Series TitleYear} [imdbid-{ImdbId}]` |
| Use Hard Links  | True                                   |

</details>

<details><summary><strong>🧬 Running Multiple Instances</strong></summary>

> Want to manage **both 1080p and 4K** libraries separately? Sonarr supports running multiple instances.

**Requirements:**

* Each instance needs its own /config folder
* Different **external port** per instance (e.g. 8989, 7879, etc.)
* Unique **root folders**, **download categories**, and **app names**

**Docker:**

```yaml
  sonarr-4k:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr-4k
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/sonarr4k:/config
      - /mnt/tank/media-4k:/media
    ports:
      - 7879:8989
    restart: unless-stopped
```

> You can even use one Sonarr to sync to the other via **Lists → Import → Sonarr**.

</details>

---

# 5 · Troubleshooting

> **Start with the Health tab** to spot missing paths or failed downloads.

<details><summary><strong>Permission Denied</strong></summary>

```bash
chmod -R 770 /mnt/tank/media/tv
chown -R 568:568 /mnt/tank/media/tv
```

</details>

<details><summary><strong>Downloads not importing</strong></summary>

Verify path translation and permissions between Sonarr and your client.

</details>

---

## ✏️ Editors & Contributors

* **Scar13t** — Page Layout & Design

> Want to help? Open a PR or join us on Discord!

---

# 📀 6 · Video Guide

[![Watch on Patreon](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)

[⇗ Back to top](#what-is-sonarr){.back-top}
