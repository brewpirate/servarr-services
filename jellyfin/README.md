# Jellyfin

**Jellyfin** is a free and open-source media server that lets you manage and stream your media collection. It's a community-driven alternative to proprietary media servers, with no premium features, licenses, or tracking.

## Links

- **Official Website**: [https://jellyfin.org/](https://jellyfin.org/)
- **GitHub**: [https://github.com/jellyfin/jellyfin](https://github.com/jellyfin/jellyfin)
- **Documentation**: [https://jellyfin.org/docs/](https://jellyfin.org/docs/)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/jellyfin](https://hub.docker.com/r/linuxserver/jellyfin)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

Additional:

- `VERSION=docker` - Use Docker version

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
JELLYFIN_PORT=8096            # Web UI/API port

# Traefik Configuration
JELLYFIN_REF=jellyfin         # Traefik router reference
JELLYFIN_URL=jellyfin.local   # Traefik hostname

# Media Paths
MEDIA_MOVIES_PATH=/path/to/movies  # Movie library
MEDIA_SERIES_PATH=/path/to/tv      # TV show library
```

### Volumes

- `./config:/config` - Jellyfin configuration and data
- `${MEDIA_MOVIES_PATH}:/Films` - Movie library
- `${MEDIA_SERIES_PATH}:/TVShows` - TV library

### Ports

- `8096` - Web UI/API
- `1900/udp` - Client Discovery (DLNA)
- `7359/udp` - Service Discovery

## Initial Setup

1. **Set environment variables** in `.env`
2. **Start the service**: `docker compose --profile jellyfin up -d`
3. **Access web UI**: Navigate to `http://localhost:${JELLYFIN_PORT}`
4. **Complete setup wizard**:
   - Select language
   - Create admin account
   - Add media libraries
   - Configure metadata providers
   - Set up remote access (optional)

## Library Configuration

### Adding Libraries

1. Navigate to **Dashboard** → **Libraries**
2. Click **Add Media Library**
3. Select library type (Movies, Shows, Music, etc.)
4. Add folders:
   - Movies: `/Films`
   - TV Shows: `/TVShows`
5. Configure metadata settings
6. Save

### Library Settings

- **Real Time Monitoring**: Enable for automatic updates
- **Metadata Downloaders**: TMDb, TVDb, etc.
- **Image Fetchers**: Configure poster/fanart sources
- **Chapter Image Extraction**: Optional

## Transcoding

### Hardware Acceleration

For NVIDIA GPU support, add to `compose.override.yaml`:

```yaml
services:
  jellyfin:
    extends:
      file: ../nvidia.compose.yaml
      service: nvidia-service
```

### Playback Settings

**Dashboard** → **Playback**:

- **Transcoding Thread Count**: Auto or set manually
- **Hardware Acceleration**: NVIDIA NVENC, Intel QuickSync, AMD AMF, etc.
- **Enable hardware encoding**: For transcoding
- **Allow encoding in HEVC format**: For supported clients
- **Transcoding Temporary Path**: Default `/config/transcodes`

### Transcoding Quality

- **H264 Encoding CRF**: 23 (lower = better quality)
- **HEVC Encoding CRF**: 28
- **Prefer fMP4-HLS Container**: Enable
- **Throttle Transcodes**: Enable to reduce load

## User Management

### Adding Users

**Dashboard** → **Users** → **Add User**:

- Set username and password
- Configure library access per user
- Set parental controls
- Enable/disable features

### User Permissions

- **Allow remote access**: Enable for internet streaming
- **Allow media deletion**: Admin feature
- **Enable Live TV**: If using Live TV
- **Restrict access**: Limit to specific libraries

## Plugins

**Dashboard** → **Plugins** → **Catalog**:

### Recommended Plugins

- **TMDb**: Enhanced movie metadata
- **TVDb**: TV show metadata
- **Trakt**: Scrobbling and sync
- **Kodi Sync Queue**: Kodi integration
- **Playback Reporting**: Usage statistics
- **Intro Skipper**: Skip TV show intros
- **Fanart**: Additional artwork
- **Skip Intro**: Detect and skip intros automatically

### Installing Plugins

1. Browse catalog
2. Click Install
3. Restart Jellyfin
4. Configure in plugin settings

## Integration with Arr Apps

### Sonarr/Radarr

Configure Jellyfin as connection:

1. **Settings** → **Connect** → **Add** → **Jellyfin/Emby**
2. Configure:
   - **Host**: `http://jellyfin:8096`
   - **API Key**: From Jellyfin Dashboard → API Keys
3. Enable notifications:
   - On Download
   - On Upgrade
   - On Rename
4. Test and Save

Jellyfin will update library when new media arrives.

## Clients

### Official Clients

- **Web**: Built-in browser client
- **Android/iOS**: Mobile apps
- **Android TV**: TV app
- **Roku**: Roku channel
- **Apple TV**: Native app
- **Fire TV**: Amazon app
- **Desktop**: Windows/Mac/Linux apps

### Third-Party Clients

- **Jellyfin Media Player**: Desktop player
- **Gelli**: Android music player
- **Finamp**: Android/iOS music player
- **Infuse**: iOS/Apple TV (premium features)
- **Kodi**: Via Jellyfin addon

## Remote Access

### Secure Remote Access

**Dashboard** → **Networking**:

1. **Enable automatic port mapping**: For UPnP
2. **Public HTTPS port**: 8920 (if using SSL)
3. **Known proxies**: Add reverse proxy IPs

### Using Reverse Proxy

Configure via Traefik (already set up):

- Access via `http://${JELLYFIN_URL}`
- For SSL, configure Traefik with certificates

### Mobile Apps

Mobile apps will automatically discover server on local network.

For remote:

- Enter server URL manually
- Or use Jellyfin Connect (requires account)

## Troubleshooting

### Playback Issues

- Check transcoding logs
- Verify client codec support
- Test direct play vs transcode
- Check network bandwidth

### Library Not Updating

- Enable real-time monitoring
- Manually scan library
- Check file permissions
- Verify paths are correct

### Transcoding Fails

- Check hardware acceleration settings
- Verify GPU drivers (if using HW accel)
- Check transcoding logs
- Test with software transcoding

### Slow Performance

- Enable hardware acceleration
- Reduce transcoding quality
- Check server resources
- Optimize library database

### Permission Issues

- Verify PUID/PGID settings
- Check file ownership
- Ensure read access to media

## Backup

Backup config directory:

```bash
tar -czf jellyfin-backup-$(date +%Y%m%d).tar.gz services/jellyfin/config
```

Important:

- `data/jellyfin.db` - Main database
- `data/library.db` - Library database
- Configuration files
- User data

## Updates

```bash
docker compose pull jellyfin
docker compose --profile jellyfin up -d
```

Jellyfin checks for updates automatically.

## Best Practices

1. **Hardware Acceleration**: Essential for transcoding
2. **Organize Media**: Use proper folder structure
3. **Metadata**: Let Jellyfin fetch automatically
4. **Regular Backups**: Database corruption can happen
5. **User Management**: Set up users with appropriate permissions
6. **Plugins**: Only install needed plugins
7. **Scheduled Tasks**: Configure library scans during off-hours
8. **Monitor Logs**: Check for issues regularly
9. **Secure Access**: Use strong passwords
10. **Update Regularly**: Keep Jellyfin current

## Comparison with Plex

### Jellyfin Advantages

- Completely free and open-source
- No telemetry or tracking
- No account required
- No premium features
- Community-driven
- More customizable

### Jellyfin Considerations

- Smaller ecosystem
- Fewer official clients
- Less polished UI (subjective)
- Manual setup for some features

## Advanced Features

### Live TV & DVR

With TV tuner:

- Add TV tuner device
- Configure EPG guide
- Schedule recordings
- Watch/record live TV

### Sync Play

Watch together feature:

- Synchronized playback
- Group watching
- Chat integration

### DLNA

Stream to DLNA devices:

- Smart TVs
- Game consoles
- Media players

### API

Full REST API:

- Custom integrations
- Third-party apps
- Automation scripts
