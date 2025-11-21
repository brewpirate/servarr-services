# servarr-services

These are docker containers that are used as base containers for [servarr-homelab-sample](https://github.com/brewpirate/servarr-homelab-sample)

## Helper Compose Files

### service-base.compose.yaml

This is the base service which mo st services include. This sets the timezone, PUID (Proccess User ID), PGID (Process Group ID), UMASK (Permissions) and a volume `/media/arr`

### nvidia.compose.yaml

Adds NVIDIA GPU support to a container. <https://docs.docker.com/compose/how-tos/gpu-support/>

### vpn.compose.yaml

Adds Gluetun VPN support to a container, make sure to assign the container port to the gluetun service in `compose.override.yaml`

## Notes

- Made a copy of `.env.sample` and `compose.override.yaml` place in your stack repo
- Don't make any changes to the service compose files, make them in `compose.override.yaml`
- Enjoy all your newly downloaded linux ISOs!
