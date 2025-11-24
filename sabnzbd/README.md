# SABnzbd

**SABnzbd** is a free and easy binary newsreader (Usenet client) with extensive automation features. It downloads files from Usenet servers based on NZB files and is the most popular choice for Usenet downloading with the Arr stack.

## Links

- **Official Website**: [https://sabnzbd.org/](https://sabnzbd.org/)
- **GitHub**: [https://github.com/sabnzbd/sabnzbd](https://github.com/sabnzbd/sabnzbd)
- **Wiki**: [https://sabnzbd.org/wiki/](https://sabnzbd.org/wiki/)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/sabnzbd](https://hub.docker.com/r/linuxserver/sabnzbd)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
SABNZBD_UI_PORT=8080          # Web UI port

# Traefik Configuration
SABNZBD_REF=sabnzbd           # Traefik router reference
SABNZBD_URL=sabnzbd.local     # Traefik hostname

# Paths
DOWNLOAD_NZB_PATH=/path/to/usenet  # Download directory
```

### Volumes

- `./config:/config` - Configuration and database
- `${DOWNLOAD_NZB_PATH}/complete:/downloads` - Completed downloads
- `${DOWNLOAD_NZB_PATH}/incomplete:/incomplete-downloads` - In-progress downloads
- `${ARR_MEDIA_PATH}:/media/arr` - Shared media path

### Ports

- `8080` - Web UI (mapped to `${SABNZBD_UI_PORT}`)

## Initial Setup

1. **Set environment variables** in `.env`
2. **Start the service**: `docker compose up -d sabnzbd`
3. **Access web UI**: Navigate to `http://localhost:${SABNZBD_UI_PORT}`
4. **Complete setup wizard**:
   - Select language
   - Add Usenet server(s)
   - Create API key (auto-generated)
   - Set up categories

## Configuration Tips

### Usenet Servers

**Config → Servers**:

Add your Usenet provider(s):

- **Host**: news.provider.com
- **Port**: 563 (SSL) or 119 (non-SSL)
- **Username/Password**: From provider
- **Connections**: Based on provider limits (e.g., 20-50)
- **SSL**: Enable for security
- **Priority**: Set if using multiple providers
- **Retention**: Days of retention (optional)

#### Popular Usenet Providers

- Newshosting
- Eweka
- UsenetServer
- Frugal Usenet
- Tweaknews

#### Multiple Servers

Use primary + backup/block accounts:

1. **Primary**: Unlimited subscription (Priority 0)
2. **Backup**: Block account for missing articles (Priority 1)

### Categories

**Config → Categories**:

Create categories for different media types:

```
movies:      /downloads/movies
tv:          /downloads/tv
music:       /downloads/music
books:       /downloads/books
```

Set per-category:

- **Folder/Path**: Download location
- **Priority**: Download priority
- **Processing**: Post-processing script
- **Indexer Categories**: Map to Newznab categories

### Folders

**Config → Folders**:

- **Temporary Download Folder**: `/incomplete-downloads` (default)
- **Completed Download Folder**: `/downloads` (default)
- **Watched Folder**: For .nzb files to auto-import
- **Script Folder**: For post-processing scripts

### Switches

**Config → Switches**:

Recommended settings:

- **Check before download**: Verify NZB completeness
- **Abort jobs that cannot be completed**: Yes
- **Pause download on post-processing**: If system resources limited
- **Nice parameters**: Lower priority for system resources
- **Auto-resume**: After network/power interruption
- **Pre-queue script**: Advanced filtering

### Special

**Config → Special**:

- **Download_free**: Minimum free space required
- **Direct Unpack**: Faster but uses more disk space
- **Unwanted Extensions**: Filter file types
- **Action when encrypted RAR**: Abort/Pause/Ignore
- **Cleanup List**: Files to auto-delete

## Arr Integration

### Add to Sonarr/Radarr/Lidarr

1. **Settings** → **Download Clients** → **Add** → **SABnzbd**
2. Configure:
   - **Name**: SABnzbd
   - **Host**: `sabnzbd` (Docker service name)
   - **Port**: `8080`
   - **API Key**: From SABnzbd Config → General
   - **Category**: Set per app (`tv`, `movies`, `music`)
   - **Recent Priority**: Normal
   - **Older Priority**: Normal
3. **Test** and **Save**

### Remote Path Mapping

Usually not needed with Docker, but if required:

- **Host**: `sabnzbd`
- **Remote Path**: `/downloads`
- **Local Path**: Your actual download path

## Post-Processing

### Sorting

**Config → Sorting**:

Enable TV/Movie sorting:

- **Sort TV**: Organize TV episodes
- **Sort Movies**: Organize movie files
- **Sort Music**: Organize music files
- **Sort Date**: Organize by date

### Scripts

Place scripts in configured script folder:

1. Create script directory
2. Add custom scripts
3. Select in category settings
4. Scripts receive download info as parameters

#### Common Scripts

- **Clean**: Remove samples, NFO files
- **Notify**: Send completion notifications
- **Convert**: Media conversion

## VPN Configuration

SABnzbd doesn't usually need VPN (Usenet uses SSL), but if desired:

### compose.override.yaml

```yaml
services:
  sabnzbd:
    network_mode: "service:gluetun"
```

### Gluetun ports

```yaml
services:
  gluetun:
    ports:
      - ${SABNZBD_UI_PORT}:8080
```

## Monitoring

### Queue

Monitor active downloads:

- Speed
- Time remaining
- Article completion
- Repair/unpack status

### History

Review completed/failed downloads:

- Success rate
- Failed articles
- Repair results
- Post-processing status

### Status

Check system health:

- Disk space
- Download speed
- Server status
- Queue size

## Troubleshooting

### Incomplete Downloads

- **Cause**: Missing articles on server
- **Solution**:
  - Add backup Usenet provider
  - Check provider retention
  - Download may be too old

### Password Protected RARs

- **Detection**: SABnzbd shows "Encrypted"
- **Action**: Configure in Special settings
- **Solution**: Set to Abort if unwanted

### Slow Downloads

- Check connection count to server
- Verify SSL overhead isn't limiting
- Check bandwidth limits
- Ensure server supports your speeds
- Try different server (if multiple configured)

### Verification/Repair Failed

- Missing par2 files
- Download from different indexer
- File may be damaged on server

### Disk Space Issues

- Set minimum free space in Special
- Clean up completed downloads
- Check for failed downloads taking space

## Backup

Backup config directory:

```bash
tar -czf sabnzbd-backup-$(date +%Y%m%d).tar.gz services/sabnzbd/config
```

Important files:

- `sabnzbd.ini` - Main configuration
- API key
- Server credentials

## Updates

```bash
docker compose pull sabnzbd
docker compose up -d sabnzbd
```

SABnzbd will auto-update its internal components.

## Best Practices

1. **Use SSL**: Always enable SSL for servers
2. **Multiple Providers**: Primary + backup/block account
3. **Categories**: Organize by media type
4. **Set Free Space**: Prevent disk full errors
5. **Enable Pre-check**: Verify before downloading
6. **Abort Encrypted**: Avoid password-protected content
7. **Monitor History**: Check completion rates
8. **Keep API Secret**: Don't expose publicly
9. **Regular Backups**: Save configuration
10. **Update Regularly**: Keep SABnzbd current

## Advanced Features

### RSS Feeds

Monitor RSS feeds for automatic downloads:

- Add feed URL
- Set filters
- Define category
- Schedule checks

### Pre-queue Scripts

Filter downloads before adding to queue:

- Check file names
- Verify size
- Custom logic
- Accept/Reject/Fail

### Notifications

**Config → Notifications**:

- Email
- Prowl
- Pushover
- Pushbullet
- Telegram
- Discord
- Custom scripts

### Quota Management

Limit downloads per time period:

- Set quota in GB
- Define time period
- Auto-pause when reached
- Reset schedule
