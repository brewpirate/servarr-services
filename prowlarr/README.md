# Prowlarr

**Prowlarr** is an indexer manager/proxy built on the popular Arr stack to integrate with your various PVR apps. It supports management of both torrent trackers and Usenet indexers, eliminating the need to set up each indexer in every Arr application individually.

## Links

- **Official Website**: [https://prowlarr.com/](https://prowlarr.com/)
- **GitHub**: [https://github.com/Prowlarr/Prowlarr](https://github.com/Prowlarr/Prowlarr)
- **Wiki**: [https://wiki.servarr.com/prowlarr](https://wiki.servarr.com/prowlarr)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/prowlarr](https://hub.docker.com/r/linuxserver/prowlarr)

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
PROWLARR_PORT=9696        # Web UI port

# Traefik Configuration
PROWLARR_REF=prowlarr     # Traefik router reference
PROWLARR_URL=prowlarr.local  # Traefik hostname
```

### Volumes

- `./config:/config` - Prowlarr configuration files and database
- `${ARR_MEDIA_PATH}:/media/arr` - Shared media path (from service-base)

### Ports

- `9696` - Web UI (mapped to `${PROWLARR_PORT}`)

### Advanced Configuration Options

#### Version Tags

LinuxServer.io provides multiple version tags for Prowlarr:

- `latest` - Stable releases (recommended)
- `develop` - Development branch (pre-release features)
- `nightly` - Nightly builds (bleeding edge, may be unstable)

To use a specific tag, modify your compose file:

```yaml
image: lscr.io/linuxserver/prowlarr:develop
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
  - FILE__API_KEY=/run/secrets/prowlarr_api_key
secrets:
  - prowlarr_api_key
```

#### Architecture Support

The image automatically detects your architecture:

- **x86-64** (amd64) - Intel/AMD processors
- **arm64** (aarch64) - Apple Silicon, Raspberry Pi 4/5, ARM servers

To pull a specific architecture:

```bash
docker pull lscr.io/linuxserver/prowlarr:amd64-latest
docker pull lscr.io/linuxserver/prowlarr:arm64v8-latest
```

#### Docker Mods

Extend container functionality using LinuxServer.io Docker Mods:

```yaml
environment:
  - DOCKER_MODS=linuxserver/mods:universal-package-install
```

Available mods at: [https://mods.linuxserver.io/?mod=prowlarr](https://mods.linuxserver.io/?mod=prowlarr)

### Traefik Labels

The service is configured with Traefik reverse proxy labels:

- Accessible via `http://${PROWLARR_URL}/`
- Entry point: `web`

### Homepage Integration

Configured to appear on Homepage dashboard:

- Group: Arr
- Shows statistics
- Icon: prowlarr.png
- Description: "The Ultimate Indexer Manager"

## Initial Setup

1. **Set environment variables** in your `.env` file
2. **Create config directory**: `mkdir -p services/prowlarr/config`
3. **Start the service**: `docker compose up -d prowlarr`
4. **Access the web UI**: Navigate to `http://localhost:${PROWLARR_PORT}` or `http://${PROWLARR_URL}/`
5. **Complete setup wizard**:
   - Set authentication (highly recommended)
   - Configure FlareSolverr if needed for Cloudflare-protected indexers

## Configuration Tips

### Adding Indexers

1. Navigate to **Indexers** → **Add Indexer**
2. Search for your preferred indexers
3. Configure each indexer:
   - Enter credentials (username/password or API key)
   - Set categories to sync
   - Configure search capabilities
4. Test the indexer connection

#### Popular Public Indexers

- 1337x
- RARBG replacements
- The Pirate Bay
- EZTV
- Nyaa (for anime)

#### Private Indexers

- Requires registration and often invites
- Better quality and retention
- May require cookies or API keys

### Connecting Applications

Prowlarr can automatically sync indexers to:

- Sonarr
- Radarr
- Lidarr
- Readarr
- Mylar3
- LazyLibrarian

#### Adding an Application

1. Go to **Settings** → **Apps** → **Add Application**
2. Select application type (e.g., Sonarr, Radarr)
3. Configure:
   - **Prowlarr Server**: `http://localhost:9696` (default)
   - **Radarr/Sonarr Server**: Use Docker service name (e.g., `http://radarr:7878`)
   - **API Key**: Get from target app's Settings → General
   - **Sync Level**: Full Sync (recommended)
   - **Sync Categories**: Select relevant categories
4. Test and Save

### Download Clients

Configure download clients in **Settings** → **Download Clients**:

#### Torrent Clients

- **qBittorrent**: Host: `qbittorrent`, Port: `8080`
- **Deluge**: Host: `deluge`, Port: `8112`
- **Transmission**: Host: `transmission`, Port: `9091`

#### Usenet Clients

- **SABnzbd**: Host: `sabnzbd`, Port: `8080`
- **NZBGet**: Host: `nzbget`, Port: `6789`

### FlareSolverr Integration

Some indexers use Cloudflare protection. Use FlareSolverr to bypass:

1. Deploy FlareSolverr container
2. In Prowlarr: **Settings** → **Indexers**
3. Add FlareSolverr:
   - **Host**: `http://flaresolverr:8191`
4. Configure indexers to use FlareSolverr in their settings

### Sync Profiles

Create sync profiles in **Settings** → **Apps** → **Sync Profiles**:

- Standard profile for most apps
- HD profile for HD-only indexers
- Anime profile for anime-specific indexers

### Notifications

Configure notifications in **Settings** → **Connect**:

- Discord
- Telegram
- Email
- Slack
- Pushover
- Webhooks

Get notified about:

- Health issues
- Indexer failures
- Application updates

## Indexer Management

### Categories

Prowlarr uses standardized categories:

- Movies
- TV
- Audio
- Books
- XXX (Adult)

Map indexer-specific categories to these standards.

### Search Capabilities

Each indexer supports different search types:

- **Search**: General search
- **TV Search**: Episode-specific
- **Movie Search**: Movie-specific
- **Audio Search**: Music/audiobook
- **Book Search**: eBook/comic

### Testing Indexers

- Test individual indexers: Click wrench icon → Test
- Bulk test: Select multiple indexers → Mass Editor → Test
- Monitor results in **System** → **Tasks**

## VPN Configuration

If your download clients are behind VPN, Prowlarr usually doesn't need VPN since it only searches, not downloads. However, if desired:

Add to `compose.override.yaml`:

```yaml
services:
  prowlarr:
    extends:
      file: ../vpn.compose.yaml
      service: vpn-service
```

Then in Gluetun config, expose Prowlarr port:

```yaml
services:
  gluetun:
    ports:
      - ${PROWLARR_PORT}:9696
```

## Troubleshooting

### Indexer Connection Issues

- Check indexer status (Indexers page)
- Verify credentials and API keys
- Test indexer connectivity
- Check if indexer is down: [https://downdetector.com/](https://downdetector.com/)
- Try FlareSolverr for Cloudflare-protected sites

### Application Sync Failures

- Verify API keys are correct
- Check application is accessible from Prowlarr container
- Use Docker service names (e.g., `radarr`) not `localhost`
- Ensure applications are on the same Docker network

### No Search Results

- Check indexer categories are properly mapped
- Verify search capabilities are enabled
- Test indexer manually
- Check indexer rate limits

### Database Locked

```bash
docker compose down prowlarr
docker compose up -d prowlarr
```

## Advanced Security Options

### Read-Only Operation

For enhanced security, run the container with a read-only filesystem:

```yaml
services:
  prowlarr:
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
  prowlarr:
    user: "1000:1000"  # Replace with your UID:GID
```

See [LinuxServer.io Non-Root Docs](https://docs.linuxserver.io/misc/non-root/) for details.

Note: When using `user:`, the PUID/PGID environment variables are ignored.

## Backup

Back up the config directory regularly:

```bash
tar -czf prowlarr-backup-$(date +%Y%m%d).tar.gz services/prowlarr/config
```

Important files:

- `prowlarr.db` - Main database with indexers and settings
- `config.xml` - Configuration
- `logs/` - Application logs

## Updates

Update to the latest version:

```bash
docker compose pull prowlarr
docker compose up -d prowlarr
```

### Monitoring Updates

Check container version:

```bash
docker inspect -f '{{ index .Config.Labels "build_version" }}' prowlarr
```

Check image version:

```bash
docker inspect -f '{{ index .Config.Labels "build_version" }}' lscr.io/linuxserver/prowlarr:latest
```

For automated update notifications, consider using [Diun](https://crazymax.dev/diun/).

## Container Management

### Shell Access

Access the container shell while running:

```bash
docker exec -it prowlarr /bin/bash
```

### Real-time Logs

Monitor container logs:

```bash
docker logs -f prowlarr
```

### Rebuild Container

If you need to completely rebuild the container:

```bash
docker compose down prowlarr
docker compose up -d prowlarr --force-recreate
```

Prowlarr auto-updates applications when configured:

- Settings → Apps → Each App → Sync Level → Full Sync

## Best Practices

1. **Start with Prowlarr**: Configure indexers here first, then connect apps
2. **Use Multiple Indexers**: Redundancy improves success rates
3. **Mix Public and Private**: Balance quality with accessibility
4. **Enable Authentication**: Protect your Prowlarr instance
5. **Monitor Health**: Check System → Status regularly
6. **Regular Backups**: Indexer configurations take time to set up
7. **Use Tags**: Organize indexers and apps with tags
8. **Rate Limiting**: Respect indexer rate limits to avoid bans
