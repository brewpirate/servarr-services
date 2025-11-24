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

- LazyLibrarian
- Lidarr
- Mylar3
- Pinchflat
- Prowlarr
- Radarr
- Sonarr

### Download Clients

#### NZB

- nzbget
- SABnzbd

#### Torrent

- deluge
- qbittorrent
- transmission

### Media

- Jellyfin
- Jellyseerr
- Kavita
- Plex
- Overseerr

### Utils

- Bazarr
- Beszel
- Checkrr
- Dozzle
- Gluetun
- Tdarr
- Traefik

### Other

- Homepage

## Notes

- Made a copy of `.env.sample` and `compose.override.yaml` place in your stack repo
- Don't make any changes to the service compose files, make them in `compose.override.yaml`
