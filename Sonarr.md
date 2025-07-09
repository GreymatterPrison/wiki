---
title: Sonarr
description: A guide to installing Sonarr in TrueNAS Scale as well as docker via compose
published: true
date: 2025-07-09T11:02:27.943Z
tags: 
editor: markdown
dateCreated: 2024-02-23T13:32:51.765Z
---

## Card‑Grid Landing (DEMO)

> *Drop this **above** the “Deploy Sonarr” section to test. Uses CSS class `card-grid` already defined on category page.*

<div class="card-grid" style="grid-template-columns:repeat(auto-fill,minmax(18rem,1fr));gap:1rem">
  <div class="card" style="padding:1rem;border-left:4px solid #42a5f5;background:#263238;border-radius:8px">
    <h3><img src="/docker.png" class="tab-icon"> Docker Compose</h3>
    <p>Copy‑paste the ready‑made <code>docker‑compose.yml</code> and run <code>up -d</code>.</p>
    <a href="#docker-compose" class="button">View YAML</a>
  </div>
  <div class="card" style="padding:1rem;border-left:4px solid #29b6f6;background:#263238;border-radius:8px">
    <h3><img src="/truenas.png" class="tab-icon"> TrueNAS SCALE</h3>
    <p>Use the Community Apps catalog to deploy Sonarr in a couple of clicks.</p>
    <a href="#truenas-community-edition" class="button">Open Steps</a>
  </div>
  <div class="card" style="padding:1rem;border-left:4px solid #26a69a;background:#263238;border-radius:8px">
    <h3>🎥 Video Walkthrough</h3>
    <p>Prefer watching? Follow the full Sonarr setup on YouTube.</p>
    <a href="#video-guide" class="button">Watch Now</a>
  </div>
</div>

<!-- Add minimal CSS to global stylesheet if `button` not defined -->

<style>
.button{display:inline-block;padding:.4rem .9rem;background:#42a5f5;color:#fff;border-radius:6px;text-decoration:none;font-weight:500}
.button:hover{opacity:.85}
</style>




# ![Sonarr](/sonarr.png){class="tab-icon"} What is Sonarr?

Sonarr is a PVR for Usenet and BitTorrent users. It monitors RSS feeds for new episodes, grabs, sorts and renames them, then upgrades quality when a better release appears.

---

<details class="quickstart" open>
<summary><strong>🚀 Quick-Start Checklist</strong></summary>

1. **Deploy container** (Docker Compose *or* TrueNAS chart).
2. **Create** `/media/tv` **root folder** in Sonarr.
3. **Add qBittorrent** as Download Client.
4. *(Optional)* Import Recyclarr profiles & advanced cleanup.

</details>

---

# 1 · Deploy Sonarr

# tabs {.tabset}

## <img src="/docker.png" class="tab-icon"> Docker Compose

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
      - 8989:8989
    restart: unless-stopped
```

### Permissions & Folder Structure {.is-success}

* **PUID / PGID** – media-owner UID/GID (TrueNAS SCALE default **568:568**).
* **Volumes** – configs at `/mnt/tank/configs/sonarr`, media at `/mnt/tank/media`.
  📌 See the [Folder-Structure](/Folder-Structure) guide.

---

## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

| Step  | Action                                                                          |
| ----- | ------------------------------------------------------------------------------- |
| **1** | **Apps → Discover Apps → Sonarr → Install**                                     |
| **2** | **Port Number → 8989**                                                          |
| **3** | **Sonarr Config Storage → Host Path** → `/mnt/tank/configs/sonarr`              |
| **4** | **Additional Storage → Host Path** → mount dataset `/mnt/tank/media` ➜ `/media` |
| **5** | Click **Save → Deploy**                                                         |

---

# 2 · First-Run Configuration

## 2.1 Root Folder

1. **Settings → Media Management → Add Root Folder**
2. Choose **/media/tv**.

## 2.2 Download Client

1. **Settings → Download Client → ➕ → qBittorrent**
2. Fill the form:

| Field                   | Example        |
| ----------------------- | -------------- |
| Host                    | `10.251.0.244` |
| Port                    | `10095`        |
| Username                | `admin`        |
| Password                | ••••••••       |
| Category                | `tv-sonarr`    |
| Recent / Older Priority | **Last**       |
| Remove Completed        | ✅              |

> **Tip:** A dedicated category (e.g. `tv-sonarr`) keeps Sonarr torrents separate from others.

---

# 3 · Advanced Tweaks *(optional)*

> **Warning** – For Recyclarr users. Enable **Show Advanced** first. {.is-warning}

### Media-Management Presets

| Field                | Recommended                            |
| -------------------- | -------------------------------------- |
| Rename Episodes      | `True`                                 |
| Episode Formats      | *TRaSH template strings*               |
| Series Folder Format | `{Series TitleYear} [imdbid-{ImdbId}]` |
| Propers & Repacks    | `Do Not Prefer`                        |
| Set Permissions      | `True` *(chmod 777)*                   |

### Profiles & Quality

Delete default profiles → keep Recyclarr-generated profiles → set Jellyseerr default.

### Metadata & Backups

Enable **Kodi/Emby** metadata. Backups: `/media`, **Interval = 1 day**, **Retention = 7**.

---

# 4 · Troubleshooting

<details><summary><strong>Sonarr cannot see media files</strong></summary>

```bash
ls -lah /mnt/tank/media/tv
chown -R 568:568 /mnt/tank/media/tv
```

</details>

<details><summary><strong>Permission denied</strong></summary>

```bash
chmod -R 770 /mnt/tank/media/tv
```

</details>

<details><summary><strong>Downloads stay in qBittorrent</strong></summary>

* Verify **Download Client Path Mapping** matches container paths.
* Confirm Sonarr can access the completed-downloads directory.

</details>

---

# Video Guide

[![Promo](/2025-03-24-advanced-media-management-with-s-promo-card.png)](https://www.patreon.com/posts/advanced-media-124639393)

[⇧ Back to top](#what-is-sonarr){.back-top}
