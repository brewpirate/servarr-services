# servarr-services

These are docker containers that are used as base containers for [servarr-homelab-sample](https://github.com/brewpirate/servarr-homelab-sample)

> [!CAUTION]  
> Don't be dumb or greedy, do the things and stay safe. Enjoy the linux ISOs!

> [!WARNING]  
> This was a project to get reacquainted with Docker, I am sure there are many mistakes and will work on correcting them. There may be drastic changes or refactors so make sure to keep your edits in `compose.override.yaml` to avoid conflicts. Please help out by making a PR or opening an issue :)

## Helper Compose Files

### service-base.compose.yaml

This is the base service which mo st services include. This sets the timezone, PUID (Proccess User ID), PGID (Process Group ID), UMASK (Permissions) and a volume `/media/arr`

### nvidia.compose.yaml

Adds NVIDIA GPU support to a container. <https://docs.docker.com/compose/how-tos/gpu-support/>

### vpn.compose.yaml

Adds Gluetun VPN support to a container, make sure to assign the container port to the gluetun service in `compose.override.yaml`

## Services

### Arr

- **[LazyLibrarian](https://lazylibrarian.gitlab.io/)** - Ebook and audiobook manager that integrates with NZB and torrent download clients
  - [GitHub](https://gitlab.com/LazyLibrarian/LazyLibrarian) | [Docker Hub](https://hub.docker.com/r/linuxserver/lazylibrarian)
- **[Lidarr](https://lidarr.audio/)** - Music collection manager for Usenet and BitTorrent users
  - [GitHub](https://github.com/Lidarr/Lidarr) | [Wiki](https://wiki.servarr.com/lidarr) | [Docker Hub](https://hub.docker.com/r/linuxserver/lidarr)
- **[Mylar3](https://github.com/mylar3/mylar3)** - Automated comic book (cbr/cbz) downloader program
  - [GitHub](https://github.com/mylar3/mylar3) | [Wiki](https://github.com/mylar3/mylar3/wiki) | [Docker Hub](https://hub.docker.com/r/linuxserver/mylar3)
- **[Pinchflat](https://github.com/kieraneglin/pinchflat)** - YouTube media manager for downloading and organizing video content
  - [GitHub](https://github.com/kieraneglin/pinchflat) | [Docker Hub](https://hub.docker.com/r/kieraneglin/pinchflat)
- **[Prowlarr](https://prowlarr.com/)** - Indexer manager/proxy for integration with various PVR apps
  - [GitHub](https://github.com/Prowlarr/Prowlarr) | [Wiki](https://wiki.servarr.com/prowlarr) | [Docker Hub](https://hub.docker.com/r/linuxserver/prowlarr)
- **[Radarr](https://radarr.video/)** - Movie collection manager for Usenet and BitTorrent users
  - [GitHub](https://github.com/Radarr/Radarr) | [Wiki](https://wiki.servarr.com/radarr) | [Docker Hub](https://hub.docker.com/r/linuxserver/radarr)
- **[Sonarr](https://sonarr.tv/)** - TV show collection manager for Usenet and BitTorrent users
  - [GitHub](https://github.com/Sonarr/Sonarr) | [Wiki](https://wiki.servarr.com/sonarr) | [Docker Hub](https://hub.docker.com/r/linuxserver/sonarr)

### Download Clients

#### NZB

- **[NZBGet](https://nzbget.com/)** - Efficient Usenet downloader written in C++
  - [GitHub](https://github.com/nzbget/nzbget) | [Docker Hub](https://hub.docker.com/r/linuxserver/nzbget)
- **[SABnzbd](https://sabnzbd.org/)** - Free and easy binary newsreader with extensive automation features
  - [GitHub](https://github.com/sabnzbd/sabnzbd) | [Wiki](https://sabnzbd.org/wiki/) | [Docker Hub](https://hub.docker.com/r/linuxserver/sabnzbd)

#### Torrent

- **[Deluge](https://deluge-torrent.org/)** - Lightweight, cross-platform BitTorrent client with extensive plugin support
  - [Website](https://deluge-torrent.org/) | [Docker Hub](https://hub.docker.com/r/linuxserver/deluge)
- **[qBittorrent](https://www.qbittorrent.org/)** - Free and reliable P2P BitTorrent client with built-in search engine
  - [GitHub](https://github.com/qbittorrent/qBittorrent) | [Wiki](https://github.com/qbittorrent/qBittorrent/wiki) | [Docker Hub](https://hub.docker.com/r/linuxserver/qbittorrent)
- **[Transmission](https://transmissionbt.com/)** - Fast, easy, and free BitTorrent client with minimal resource usage
  - [GitHub](https://github.com/transmission/transmission) | [Docker Hub](https://hub.docker.com/r/linuxserver/transmission)

### Media

- **[Jellyfin](https://jellyfin.org/)** - Free and open-source media server for managing and streaming your media library
  - [GitHub](https://github.com/jellyfin/jellyfin) | [Docs](https://jellyfin.org/docs/) | [Docker Hub](https://hub.docker.com/r/linuxserver/jellyfin)
- **[Jellyseerr](https://github.com/Fallenbagel/jellyseerr)** - Media request manager for Jellyfin (fork of Overseerr)
  - [GitHub](https://github.com/Fallenbagel/jellyseerr) | [Docs](https://docs.jellyseerr.dev/) | [Docker Hub](https://hub.docker.com/r/fallenbagel/jellyseerr)
- **[Kavita](https://www.kavitareader.com/)** - Fast, feature rich, cross-platform reading server for comics, manga, and ebooks
  - [GitHub](https://github.com/Kareadita/Kavita) | [Wiki](https://wiki.kavitareader.com/) | [Docker Hub](https://hub.docker.com/r/jvmilazz0/kavita)
- **[Plex](https://www.plex.tv/)** - Media server for organizing, streaming, and sharing your media collection
  - [Website](https://www.plex.tv/) | [Support](https://support.plex.tv/) | [Docker Hub](https://hub.docker.com/r/linuxserver/plex)
- **[Overseerr](https://overseerr.dev/)** - Media request manager and discovery tool for Plex ecosystem
  - [GitHub](https://github.com/sct/overseerr) | [Docs](https://docs.overseerr.dev/) | [Docker Hub](https://hub.docker.com/r/linuxserver/overseerr)

### Utils

- **[Bazarr](https://www.bazarr.media/)** - Companion application to Sonarr and Radarr for managing and downloading subtitles
  - [GitHub](https://github.com/morpheus65535/bazarr) | [Wiki](https://wiki.bazarr.media/) | [Docker Hub](https://hub.docker.com/r/linuxserver/bazarr)
- **[Beszel](https://github.com/henrygd/beszel)** - Lightweight server monitoring hub with historical data and Docker stats
  - [GitHub](https://github.com/henrygd/beszel) | [Docker Hub](https://hub.docker.com/r/henrygd/beszel)
- **[Checkrr](https://github.com/aetaric/checkrr)** - Validates media files for Radarr, Sonarr, and Lidarr to prevent corruption
  - [GitHub](https://github.com/aetaric/checkrr) | [Docker Hub](https://hub.docker.com/r/aetaric/checkrr)
- **[Dozzle](https://dozzle.dev/)** - Real-time Docker log viewer with a clean, web-based interface
  - [GitHub](https://github.com/amir20/dozzle) | [Docs](https://dozzle.dev/guide) | [Docker Hub](https://hub.docker.com/r/amir20/dozzle)
- **[Gluetun](https://github.com/qdm12/gluetun)** - VPN client in a thin Docker container supporting multiple providers with port forwarding
  - [GitHub](https://github.com/qdm12/gluetun) | [Wiki](https://github.com/qdm12/gluetun-wiki) | [Docker Hub](https://hub.docker.com/r/qmcgaw/gluetun)
- **[Tdarr](https://tdarr.io/)** - Distributed transcode automation tool for optimizing media libraries
  - [GitHub](https://github.com/HaveAGitGat/Tdarr) | [Docs](https://docs.tdarr.io/) | [Docker Hub](https://hub.docker.com/r/haveagitgat/tdarr)
- **[Traefik](https://traefik.io/)** - Modern HTTP reverse proxy and load balancer with automatic SSL/TLS certificates
  - [GitHub](https://github.com/traefik/traefik) | [Docs](https://doc.traefik.io/traefik/) | [Docker Hub](https://hub.docker.com/_/traefik)

### Other

- **[Homepage](https://gethomepage.dev/)** - Highly customizable homepage (or startpage/application dashboard) with Docker and service API integrations
  - [GitHub](https://github.com/gethomepage/homepage) | [Docs](https://gethomepage.dev/latest/) | [Docker Hub](https://hub.docker.com/r/ghcr.io/gethomepage/homepage)

## Notes

- Made a copy of `.env.sample` and `compose.override.yaml` place in your stack repo
- Don't make any changes to the service compose files, make them in `compose.override.yaml`
