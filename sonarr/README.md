# Sonarr

**Sonarr** is a PVR (Personal Video Recorder) for Usenet and BitTorrent users. It monitors multiple RSS feeds for new episodes of your favorite TV shows and automatically downloads, sorts, and renames them. It can also be configured to automatically upgrade the quality of files already downloaded when a better quality format becomes available.

## Links

- **Official Website**: [https://sonarr.tv/](https://sonarr.tv/)
- **GitHub**: [https://github.com/Sonarr/Sonarr](https://github.com/Sonarr/Sonarr)
- **Wiki**: [https://wiki.servarr.com/sonarr](https://wiki.servarr.com/sonarr)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/sonarr](https://hub.docker.com/r/linuxserver/sonarr)

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
SONARR_PORT=8989          # Web UI port

# Traefik Configuration
SONARR_REF=sonarr         # Traefik router reference
SONARR_URL=sonarr.local   # Traefik hostname

# Media Path
MEDIA_SERIES_PATH=/path/to/tv/shows  # Path to your TV show library
```

### Volumes

- `./config:/config` - Sonarr configuration files and database
- `${MEDIA_SERIES_PATH}:/tv` - TV show library location
- `${ARR_MEDIA_PATH}:/media/arr` - Shared media path (from service-base)

### Ports

- `8989` - Web UI (mapped to `${SONARR_PORT}`)

### Advanced Configuration Options

#### Version Tags

LinuxServer.io provides multiple version tags for Sonarr:

- `latest` - Stable releases (recommended)
- `develop` - Development branch (pre-release features)

To use a specific tag, modify your compose file:

```yaml
image: lscr.io/linuxserver/sonarr:develop
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
  - FILE__API_KEY=/run/secrets/sonarr_api_key
secrets:
  - sonarr_api_key
```

#### Architecture Support

The image automatically detects your architecture:

- **x86-64** (amd64) - Intel/AMD processors
- **arm64** (aarch64) - Apple Silicon, Raspberry Pi 4/5, ARM servers

To pull a specific architecture:

```bash
docker pull lscr.io/linuxserver/sonarr:amd64-latest
docker pull lscr.io/linuxserver/sonarr:arm64v8-latest
```

#### Docker Mods

Extend container functionality using LinuxServer.io Docker Mods:

```yaml
environment:
  - DOCKER_MODS=linuxserver/mods:universal-package-install
```

Available mods at: [https://mods.linuxserver.io/?mod=sonarr](https://mods.linuxserver.io/?mod=sonarr)

### Traefik Labels

The service is configured with Traefik reverse proxy labels:

- Accessible via `http://${SONARR_URL}/`
- Entry point: `web`

### Homepage Integration

Configured to appear on Homepage dashboard:

- Group: Arr
- Shows statistics
- Icon: sonarr.png

## Initial Setup

1. **Set environment variables** in your `.env` file
2. **Create config directory**: `mkdir -p services/sonarr/config`
3. **Start the service**: `docker compose up -d sonarr`
4. **Access the web UI**: Navigate to `http://localhost:${SONARR_PORT}` or `http://${SONARR_URL}/`
5. **Complete setup wizard**:
   - Set authentication if desired
   - Configure media management settings
   - Add root folder: `/tv`
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
2. Add Sonarr application
3. Use internal Docker hostname: `http://sonarr:8989`
4. Generate and enter API key from Sonarr Settings → General

### Media Management

Recommended settings (Settings → Media Management):

- Enable "Rename Episodes"
- Set naming format for better organization
- Enable "Unmonitor Deleted Episodes"
- Configure file permissions to match your UMASK

### Quality Profiles

Configure in Settings → Profiles:

- Create custom quality profiles based on your preferences
- Set quality cutoff limits
- Enable/disable quality upgrades

### Root Folders

- Default TV folder: `/tv` (mapped to `${MEDIA_SERIES_PATH}`)
- Shared media: `/media/arr` for cross-service access

## VPN Configuration

To route Sonarr through VPN (Gluetun), add to `compose.override.yaml`:

```yaml
services:
  sonarr:
    extends:
      file: ../vpn.compose.yaml
      service: vpn-service
```

Then in Gluetun config, expose Sonarr port:

```yaml
services:
  gluetun:
    ports:
      - ${SONARR_PORT}:8989
```

## Troubleshooting

### Permission Issues

Ensure PUID/PGID match your host user:

```bash
id $USER  # Check your UID and GID
```

Update `.env` file with correct values.

### Database Locked

If database is locked, stop the container and check for multiple instances:

```bash
docker compose down sonarr
docker compose up -d sonarr
```

### Import Issues

- Verify path mappings match between Sonarr and download clients
- Ensure download client remote path mapping is configured if needed
- Check file permissions on mounted volumes

## Advanced Security Options

### Read-Only Operation

For enhanced security, run the container with a read-only filesystem:

```yaml
services:
  sonarr:
    read_only: true
    tmpfs:
      - /tmp
      - /var/tmp
```

See [LinuxServer.io Read-Only Docs](https://docs.linuxserver.io/misc/read-only/) for details.

### Non-Root Operation

Run the container as a non-root user:

```yaml
services:
  sonarr:
    user: "1000:1000"  # Replace with your UID:GID
```

See [LinuxServer.io Non-Root Docs](https://docs.linuxserver.io/misc/non-root/) for details.

Note: When using `user:`, the PUID/PGID environment variables are ignored.

## Backup

Back up the config directory regularly:

```bash
tar -czf sonarr-backup-$(date +%Y%m%d).tar.gz services/sonarr/config
```

## Updates

Update to the latest version:

```bash
docker compose pull sonarr
docker compose up -d sonarr
```

### Monitoring Updates

Check container version:

```bash
docker inspect -f '{{ index .Config.Labels "build_version" }}' sonarr
```

Check image version:

```bash
docker inspect -f '{{ index .Config.Labels "build_version" }}' lscr.io/linuxserver/sonarr:latest
```

For automated update notifications, consider using [Diun](https://crazymax.dev/diun/).

## Container Management

### Shell Access

Access the container shell while running:

```bash
docker exec -it sonarr /bin/bash
```

### Real-time Logs

Monitor container logs:

```bash
docker logs -f sonarr
```

### Rebuild Container

If you need to completely rebuild the container:

```bash
docker compose down sonarr
docker compose up -d sonarr --force-recreate
```

Sonarr will automatically update its internal database schema if needed.
