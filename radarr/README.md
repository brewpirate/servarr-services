# Radarr

**Radarr** is a movie collection manager for Usenet and BitTorrent users. It monitors multiple RSS feeds for new movies and interfaces with clients and indexers to grab, sort, and rename them. It can also automatically upgrade the quality of existing files when a better quality format becomes available.

## Links

- **Official Website**: [https://radarr.video/](https://radarr.video/)
- **GitHub**: [https://github.com/Radarr/Radarr](https://github.com/Radarr/Radarr)
- **Wiki**: [https://wiki.servarr.com/radarr](https://wiki.servarr.com/radarr)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/radarr](https://hub.docker.com/r/linuxserver/radarr)

## Configuration

### Environment Variables

The following environment variables are inherited from `service-base.compose.yaml`:

- `PUID`: Process User ID (default from `.env`)
- `PGID`: Process Group ID (default from `.env`)
- `TZ`: Timezone (default from `.env`)
- `UMASK`: File permission mask (default from `.env`)

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
RADARR_PORT=7878          # Web UI port

# Traefik Configuration
RADARR_REF=radarr         # Traefik router reference
RADARR_URL=radarr.local   # Traefik hostname

# Media Path
MEDIA_MOVIES_PATH=/path/to/movies  # Path to your movie library
```

### Volumes

- `./config:/config` - Radarr configuration files and database
- `${MEDIA_MOVIES_PATH}:/data` - Movie library location
- `${ARR_MEDIA_PATH}:/media/arr` - Shared media path (from service-base)

### Ports

- `7878` - Web UI (mapped to `${RADARR_PORT}`)

### Advanced Configuration Options

#### Version Tags

LinuxServer.io provides multiple version tags for Radarr:

- `latest` - Stable releases (recommended)
- `develop` - Development branch (pre-release features)
- `nightly` - Nightly builds (bleeding edge, may be unstable)

To use a specific tag, modify your compose file:

```yaml
image: lscr.io/linuxserver/radarr:develop
```

#### UMASK Configuration

The `UMASK` variable controls file permissions for created files. It subtracts from default permissions (not chmod).

Common values:

- `022` - Default, files: 644, dirs: 755 (owner write, all read)
- `002` - Group writable, files: 664, dirs: 775
- `000` - All writable, files: 666, dirs: 777

Set in your `.env` file:

```bash
UMASK=022
```

#### Docker Secrets Support

For sensitive environment variables, use Docker secrets with the `FILE__` prefix:

```yaml
environment:
  - FILE__API_KEY=/run/secrets/radarr_api_key
secrets:
  - radarr_api_key
```

#### Architecture Support

The image automatically detects your architecture:

- **x86-64** (amd64) - Intel/AMD processors
- **arm64** (aarch64) - Apple Silicon, Raspberry Pi 4/5, ARM servers

To pull a specific architecture:

```bash
docker pull lscr.io/linuxserver/radarr:amd64-latest
docker pull lscr.io/linuxserver/radarr:arm64v8-latest
```

#### Docker Mods

Extend container functionality using LinuxServer.io Docker Mods:

```yaml
environment:
  - DOCKER_MODS=linuxserver/mods:radarr-striptracks
```

Available mods at: [https://mods.linuxserver.io/?mod=radarr](https://mods.linuxserver.io/?mod=radarr)

### Traefik Labels

The service is configured with Traefik reverse proxy labels:

- Accessible via `http://${RADARR_URL}/`
- Entry point: `web`

### Homepage Integration

Configured to appear on Homepage dashboard:

- Group: Arr
- Shows statistics
- Icon: radarr.png

## Initial Setup

1. **Set environment variables** in your `.env` file
2. **Create config directory**: `mkdir -p services/radarr/config`
3. **Start the service**: `docker compose up -d radarr`
4. **Access the web UI**: Navigate to `http://localhost:${RADARR_PORT}` or `http://${RADARR_URL}/`
5. **Complete setup wizard**:
   - Set authentication if desired
   - Configure media management settings
   - Add root folder: `/data`
   - Connect to Prowlarr for indexers
   - Add download clients (qBittorrent, Deluge, SABnzbd, etc.)

## Configuration Tips

### Download Clients

Add download clients in Settings → Download Clients:

- **qBittorrent**: Host: `qbittorrent`, Port: `8080` (or your configured port)
- **Deluge**: Host: `deluge`, Port: `8112`
- **SABnzbd**: Host: `sabnzbd`, Port: `8080`

### Indexers via Prowlarr

1. In Prowlarr, go to Settings → Apps
2. Add Radarr application
3. Use internal Docker hostname: `http://radarr:7878`
4. Generate and enter API key from Radarr Settings → General

### Media Management

Recommended settings (Settings → Media Management):

- Enable "Rename Movies"
- Set naming format with quality and release info
- Enable "Replace Illegal Characters"
- Configure "Minimum Free Space" (e.g., 100 MB)
- Set "Propers and Repacks" to prefer proper releases

### Quality Profiles

Configure in Settings → Profiles → Quality Profiles:

- Create profiles for different quality preferences (1080p, 4K, etc.)
- Set upgrade cutoffs
- Define quality order and size limits

### Custom Formats

Use Custom Formats (Settings → Custom Formats) to:

- Prefer or avoid specific release groups
- Prioritize HDR content
- Control audio formats (Atmos, DTS-HD, etc.)
- Manage release sources (BluRay, WEB-DL, etc.)

### Root Folders

- Default movie folder: `/data` (mapped to `${MEDIA_MOVIES_PATH}`)
- Shared media: `/media/arr` for cross-service access

## Lists Integration

Radarr supports automatic imports from:

- **TMDb Lists**: Import from TMDb popular, trending, etc.
- **Trakt Lists**: Sync with your Trakt watchlists
- **IMDb Lists**: Import from IMDb lists
- **Plex Watchlist**: Import from Plex watchlist
- **Radarr Lists**: Share lists between Radarr instances

Configure in Settings → Lists.

## VPN Configuration

To route Radarr through VPN (Gluetun), add to `compose.override.yaml`:

```yaml
services:
  radarr:
    extends:
      file: ../vpn.compose.yaml
      service: vpn-service
```

Then in Gluetun config, expose Radarr port:

```yaml
services:
  gluetun:
    ports:
      - ${RADARR_PORT}:7878
```

## Troubleshooting

### Permission Issues

Ensure PUID/PGID match your host user:

```bash
id $USER  # Check your UID and GID
```

Update `.env` file with correct values.

### Failed Downloads

- Check download client configuration
- Verify path mappings match between Radarr and download clients
- Check indexer status in System → Status
- Review activity history for error messages

### Missing Movies After Import

- Verify movie files meet minimum quality requirements
- Check Media Management settings for file naming
- Review logs at System → Logs

### Database Issues

If experiencing database corruption:

```bash
docker compose down radarr
# Restore from backup or let Radarr rebuild
docker compose up -d radarr
```

## Advanced Security Options

### Read-Only Operation

For enhanced security, run the container with a read-only filesystem. This prevents any modifications to the container's filesystem while allowing writes to mounted volumes.

Add to your compose file:

```yaml
services:
  radarr:
    read_only: true
    tmpfs:
      - /tmp
      - /var/tmp
```

See [LinuxServer.io Read-Only Docs](https://docs.linuxserver.io/misc/read-only/) for details.

### Non-Root Operation

Run the container as a non-root user for additional security:

```yaml
services:
  radarr:
    user: "1000:1000"  # Replace with your UID:GID
```

See [LinuxServer.io Non-Root Docs](https://docs.linuxserver.io/misc/non-root/) for details.

Note: When using `user:`, the PUID/PGID environment variables are ignored.

## Backup

Back up the config directory regularly:

```bash
tar -czf radarr-backup-$(date +%Y%m%d).tar.gz services/radarr/config
```

The config directory contains:

- `radarr.db` - Main database
- `config.xml` - Configuration settings
- `logs/` - Application logs

## Updates

Update to the latest version:

```bash
docker compose pull radarr
docker compose up -d radarr
```

Radarr will automatically update its internal database schema if needed.

### Monitoring Updates

Check container version:

```bash
docker inspect -f '{{ index .Config.Labels "build_version" }}' radarr
```

Check image version:

```bash
docker inspect -f '{{ index .Config.Labels "build_version" }}' lscr.io/linuxserver/radarr:latest
```

For automated update notifications, consider using [Diun](https://crazymax.dev/diun/).

## Container Management

### Shell Access

Access the container shell while running:

```bash
docker exec -it radarr /bin/bash
```

### Real-time Logs

Monitor container logs:

```bash
docker logs -f radarr
```

### Rebuild Container

If you need to completely rebuild the container:

```bash
docker compose down radarr
docker compose up -d radarr --force-recreate
```

## Integration with Other Services

### Overseerr/Jellyseerr

Configure Radarr in Overseerr/Jellyseerr:

- Server: `http://radarr:7878`
- API Key: From Radarr Settings → General
- Quality Profile: Select default profile
- Root Folder: `/data`

### Plex/Jellyfin

Radarr can update your media server library:

- Settings → Connect → Add Connection
- Select Plex or Jellyfin
- Enter server details and authentication
