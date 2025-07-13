---
title: SABnzbd
description: A guide to deploying SABnzbd via TrueNAS or docker
published: true
date: 2025-07-13T22:48:15.195Z
tags: 
editor: markdown
dateCreated: 2025-06-30T22:21:23.261Z
---

# ![SABnzbd](/sabnzbd.png){class="tab-icon"} What is SABnzbd?

**SABnzbd** is a free, open‑source Usenet downloader.  It automatically **grabs → verifies → repairs → extracts → renames → sorts** NZB files, then hands off the finished job to tools such as **Sonarr / Radarr / Lidarr** for seamless media management.


<details class="quickstart" open>
<summary><strong>🚀 Quick‑Start Checklist</strong></summary>

1. **Deploy container** (Docker Compose *or* TrueNAS App)
2. **Create** `/media/downloads` with **complete** + **incomplete** sub‑folders
3. **Point SABnzbd**: *Temporary* → `/media/downloads/incomplete`, *Completed* → `/media/downloads/complete`
4. **Add Usenet provider** credentials + **News Server**
5. *(Optional)* Route SABnzbd & qBittorrent **through VPN** using the *Hotio + VPN* stack

</details>


# 1 · Deploy SABnzbd
# {.tabset}
## <img src="/docker.png" class="tab-icon"> Docker Compose

```yaml
services:
  sabnzbd:
    image: lscr.io/linuxserver/sabnzbd:latest
    container_name: sabnzbd
    environment:
      - PUID=568
      - PGID=568
      - TZ=America/New_York
    volumes:
      - /mnt/tank/configs/sabnzbd:/config
      - /mnt/tank/media:/media
    ports:
      - 8080:8080
    restart: unless-stopped
```

### Permissions & Folder Structure {.is-success}

* **PUID / PGID** – match your media‑owner account (TrueNAS default **568:568**).
* **Volumes** – configs at `/mnt/tank/configs/sabnzbd`, media at `/mnt/tank/media`.
  📌 Follow the [Folder‑Structure](/Folder-Structure) guide.


## <img src="/docker.png" class="tab-icon"> Hotio + VPN (WireGuard)

*A single VPN container protects SABnzbd & qBittorrent.*

```yaml
services:
  sabnzbd:
    container_name: sabnzbd
    image: ghcr.io/hotio/sabnzbd
    ports:
      - 8080:8080
    environment:
      - PUID=568
      - PGID=568
      - UMASK=002
      - TZ=Europe/Amsterdam
      - WEBUI_PORTS=8080/tcp,8080/udp
      - VPN_ENABLED=true #
      - VPN_CONF=wg0 #
      - VPN_PROVIDER=generic #
      - VPN_LAN_NETWORK=192.168.1.0/24 #
      - VPN_LAN_LEAK_ENABLED=false #
      - VPN_EXPOSE_PORTS_ON_LAN #
      - VPN_AUTO_PORT_FORWARD=false #
      - VPN_AUTO_PORT_FORWARD_TO_PORTS= #
      - VPN_FIREWALL_TYPE=auto #
      - VPN_HEALTHCHECK_ENABLED=false #
      - VPN_NAMESERVERS= #
      - PRIVOXY_ENABLED=false #
      - UNBOUND_ENABLED=false #
      - UNBOUND_NAMESERVERS #
    cap_add:
      - NET_ADMIN
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1 #
      - net.ipv6.conf.all.disable_ipv6=1 #
    volumes:
      - /mnt/tank/configs/sabnzbd:/config
      - /mnt/tank/media:/media
```

> **wg0.conf required** – drop your WireGuard file into `config/wireguard/wg0.conf` before first launch. {.is-warning}


## <img src="/truenas.png" class="tab-icon"> TrueNAS Community Edition

| Step  | Action                                                                          |
| ----- | -------------------- |
| **1** | **Apps → Discover Apps → SABnzbd → Install**    |
| **2** | **SABnzbd Config Storage → Host Path** → `/mnt/tank/configs/sabnzbd`  |
| **3** | **Additional Storage → Host Path** → mount dataset `/mnt/tank/media` ➜ `/media` |
| **4** | Click **Save → Deploy**      |


# 2 · First‑Run Configuration

## 2.1 Folder Paths  <span class="chip">Mandatory</span>

Inside the web‑UI (`Settings → Folders`):

| Purpose             | Path                          |
| ------------------- | ----------------------------- |
| Temporary Downloads | `/media/downloads/incomplete` |
| Completed Downloads | `/media/downloads/complete`   |

## 2.2 Usenet Server  <span class="chip">Provider Login</span>

1. **Settings → Servers → +**
2. Enter your provider’s host, port, SSL, username & password.
3. **Test Server** → green ✔️ → **Save**.

## 2.3 Categories (for Sonarr/Radarr)

Create two categories:

| Category        | Directory       | Script |
| --------------- | --------------- | ------ |
| `tv-sonarr`     | (leave default) | none   |
| `movies-radarr` | (leave default) | none   |

Sonarr/Radarr will assign these per download and sort post‑process.

# 3 · Advanced Tweaks *(optional)*

> **Show Advanced** in the UI to reveal orange fields. {.is-warning}

| Setting             | Recommended     | Why                      |
| ------------------- | --------------- | ------------------------ |
| **Direct Unpack**   | `True`          | Speeds up large releases |
| **Rating**          | Block below 50% | Filters junk uploads     |
| **Maximum Retries** | `3`             | Avoid infinite loops     |
| **Permissions**     | `chmod 770`     | Matches media-share mask |

### Post‑Processing Cleanup (blocked extensions)

Paste this list into **Settings → Switches → Cleanup list** to delete unsafe executables after download:

```
exe, bat, cmd, com, scr, pif, hta, vbs, js, jar, wsf, ps1, msi, msp, cpl, ad, apk, dll, bin, gadget, vb, vbe, ws, wsc, wsh, lnk, iso, img, dmg, zipx, psm1, psd1, psc1, sh, rb, perl, py, pyd, url
```


# 4 · Troubleshooting

<details><summary><strong>SABnzbd won’t start (port in use)</strong></summary>
Another service (often Hotio/qBittorrent) is already bound to 8080. Change Web UI port in your compose file or TrueNAS form.
</details>

<details>
  <summary><strong>Downloads stay in “Failed” state</strong></summary>

* **Missing par2 binaries** — enable *Repair* in **Settings → Switches**
* **News‑server article age too low** — switch to a provider with **> 3000 days** retention.

</details>

<details><summary><strong>Permission denied writing to /media</strong></summary>
Ensure host path uses the same PUID/PGID as Sonarr/Radarr (TrueNAS: 568:568) or run `chown -R 568:568 /mnt/tank/media`.
</details>

## ✏️ Editors & Contributors

> **Special thanks to the following members for reviewing and polishing this guide**
>
> * Deepblue
> * Scar13t
> * TechFahter
> * Tom Tech

Feel free to open a pull‑request or ping us on Discord if you spot an inaccuracy!

# <img src="/patreon-light.png" class="tab-icon"> Video Guide

*Coming soon – follow our [Patreon](https://www.patreon.com/serversathome) for a walk‑through!*

