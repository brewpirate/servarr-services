# Plex

**Plex** organizes your video, music, and photo collections and streams them to all of your devices. It's one of the most popular media server solutions with native apps for virtually every platform.

## Links

- **Official Website**: [https://www.plex.tv/](https://www.plex.tv/)
- **Support**: [https://support.plex.tv/](https://support.plex.tv/)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/plex](https://hub.docker.com/r/linuxserver/plex)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

Additional:

- `VERSION=docker` - Use Docker version

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
PLEX_PORT=32400               # Plex port

# Traefik Configuration
PLEX_REF=plex                 # Traefik router reference
PLEX_URL=plex.local           # Traefik hostname

# Media Paths
MEDIA_MOVIES_PATH=/path/to/movies  # Movie library
MEDIA_SERIES_PATH=/path/to/tv      # TV library
```

### Volumes

- `./config:/config` - Plex configuration and metadata
- `${MEDIA_MOVIES_PATH}:/movies` - Movie library
- `${MEDIA_SERIES_PATH}:/tv` - TV library

### Ports

- `32400` - Plex web interface and API

## Initial Setup

1. **Set environment variables** in `.env`
2. **Start the service**: `docker compose --profile plex up -d`
3. **Access web UI**: Navigate to `http://localhost:${PLEX_PORT}/web`
4. **Sign in** with Plex account (create one if needed)
5. **Complete setup wizard**:
   - Name your server
   - Add media libraries
   - Allow remote access (optional)

## Claim Server

To link with your Plex account, you may need to claim the server:

Visit `http://localhost:32400/web` immediately after first start, sign in, and server will be claimed automatically.

Alternatively, use claim token in environment variables.

## Library Setup

### Adding Libraries

1. **Settings** → **Manage** → **Libraries** → **Add Library**
2. Select library type (Movies, TV Shows, Music, Photos)
3. Add folders:
   - Movies: `/movies`
   - TV Shows: `/tv`
4. Configure advanced settings (agents, scanners)
5. Add library

### Library Settings

- **Scanner**: Plex Movie/TV Series scanner (default)
- **Agent**: TheMovieDB or TheTVDB
- **Enable video preview thumbnails**: Optional (resource-intensive)
- **Include in dashboard**: Show in home
- **Scan my library automatically**: Enable
- **Run a partial scan when changes are detected**: Enable

### Metadata Agents

**Settings** → **Agents**:

Select metadata sources:

- **TheMovieDB** (recommended for movies)
- **TheTVDB** (recommended for TV)
- **Local Media Assets**: Include local metadata
- Order affects priority

## Transcoding

### Hardware Acceleration

Requires Plex Pass subscription.

For NVIDIA GPU, add to `compose.override.yaml`:

```yaml
services:
  plex:
    extends:
      file: ../nvidia.compose.yaml
      service: nvidia-service
```

### Transcoding Settings

**Settings** → **Transcoder**:

- **Transcoder quality**: Automatic (recommended)
- **Transcoder temporary directory**: Default `/config/Library/Application Support/Plex Media Server/Cache/Transcode`
- **Transcode throttle buffer**: 60 seconds
- **Background transcoding x264 preset**: Fast
- **Use hardware acceleration when available**: Enable (Plex Pass)
- **Enable HDR tone mapping**: If needed (Plex Pass)

## User Management

### Plex Home

**Settings** → **Users & Sharing** → **Plex Home**:

- Add managed users
- Kids accounts with restrictions
- Separate watch history per user

### Friends

**Settings** → **Users & Sharing** → **Friends**:

- Share libraries with other Plex users
- Select libraries to share
- Set restrictions per user
- Control sync, camera upload, etc.

### Authentication

- Users need Plex accounts
- Friends sign in with their accounts
- Managed users selected at sign-in

## Remote Access

### Enable Remote Access

**Settings** → **Remote Access**:

1. **Enable Remote Access**
2. **Manually specify public port**: Optional
3. **Test Remote Access**
4. If successful, fully accessible remotely

### Requirements

- Port forwarding on router (32400)
- Or use Plex Relay (slower, but works anywhere)
- Plex account required

## Settings Optimization

### Network

**Settings** → **Network**:

- **Enable server support for IPv6**: If applicable
- **Secure connections**: Preferred or Required
- **Treat WAN IP As LAN Bandwidth**: If accessing locally via public IP
- **List of IP addresses and networks that are allowed without auth**: Trusted networks

### DLNA

**Settings** → **DLNA**:

- **Enable the DLNA server**: For DLNA devices
- DLNA server name

## Plugins

**Settings** → **Plugins**:

Plex has limited plugin support now. Most functionality built-in.

Popular plugins:

- **Trakt Scrobbler**: Sync with Trakt
- **Sub-Zero**: Enhanced subtitle support
- **Webhooks**: For automation

## Arr Integration

### Connect from Radarr/Sonarr

1. **Settings** → **Connect** → **Add** → **Plex Media Server**
2. Configure:
   - **Host**: `http://plex:32400`
   - **Port**: `32400`
   - **Auth Token**: Get from Plex (see below)
3. **Update Library**: On Download, On Upgrade, On Rename
4. Test and Save

### Get Plex Auth Token

Visit: `http://localhost:32400/web` → Settings → Account → XML

Or use X-Plex-Token from browser developer tools.

## Mobile Apps

Official Plex apps:

- **iOS**: App Store
- **Android**: Google Play
- **Android TV**: Google Play
- **Apple TV**: App Store
- **Roku**: Roku Channel Store
- **Fire TV**: Amazon App Store
- **Smart TVs**: Built-in app on Samsung, LG, Vizio, etc.
- **Game Consoles**: PlayStation, Xbox

All require Plex account to sign in.

## Troubleshooting

### Cannot Access Web Interface

- Verify container is running
- Check port mapping
- Try `http://localhost:32400/web`
- Check firewall rules

### Remote Access Not Working

- Check port forwarding (32400)
- Verify external IP is correct
- Test with Plex Relay as fallback
- Check router UPnP settings

### Transcoding Not Working

- Check transcoder settings
- Verify enough disk space for transcoding
- Check permissions on transcode directory
- Review logs for errors

### Library Not Updating

- Trigger manual scan
- Check "Scan my library automatically" is enabled
- Verify file permissions
- Check file naming matches Plex guidelines

### Buffering Issues

- Check network speed
- Reduce transcoding quality
- Enable hardware acceleration
- Check client device capabilities
- Consider local playback vs remote

## Backup

```bash
tar -czf plex-backup-$(date +%Y%m%d).tar.gz services/plex/config
```

Important (but large):

- `Library/Application Support/Plex Media Server/` - Main data
- Configuration files
- Metadata and posters

**Note**: Plex config can be very large due to metadata.

## Updates

```bash
docker compose pull plex
docker compose --profile plex up -d
```

Plex auto-updates are available with Plex Pass.

## Best Practices

1. **Plex Pass**: Recommended for hardware transcoding
2. **Proper Naming**: Follow Plex naming conventions
3. **Hardware Acceleration**: Essential for multiple streams
4. **Regular Backups**: Metadata takes time to build
5. **Monitor Bandwidth**: Remote streaming uses upload bandwidth
6. **Secure Server**: Use authentication
7. **Library Organization**: Separate movies/TV/music
8. **Match Agent**: Use correct agents for media type
9. **Update Regularly**: Keep Plex current
10. **Monitor Logs**: Check for issues

## Plex Pass Benefits

Subscription features:

- **Hardware transcoding**: GPU acceleration
- **Mobile sync**: Download for offline
- **Live TV & DVR**: With TV tuner
- **Music videos**: Enhanced music features
- **Premium music features**: Sonic analysis
- **Parental controls**: Enhanced restrictions
- **Early access**: New features first

## File Naming

Follow Plex guidelines:

### Movies

```
/movies
  /Movie Name (Year)
    Movie Name (Year).mkv
```

### TV Shows

```
/tv
  /Show Name (Year)
    /Season 01
      Show Name - S01E01 - Episode Name.mkv
```

## Advanced Features

### Live TV & DVR

With Plex Pass and TV tuner:

- Watch live TV
- Record shows (DVR)
- Guide data
- Skip commercials (experimental)

### Tautulli Integration

Monitor Plex usage with Tautulli:

- User statistics
- Library stats
- Notification system
- History tracking

Third-party tool, requires separate installation.

## Alternatives

If Plex doesn't fit:

- **Jellyfin**: Free, open-source alternative
- **Emby**: Similar to Plex, some features paid
- **Kodi**: Media center, not server
