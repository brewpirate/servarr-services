# NZBGet

**NZBGet** is an efficient Usenet downloader written in C++ and designed with performance in mind to achieve maximum download speed by using very little system resources. It's a lightweight alternative to SABnzbd with powerful features.

## Links

- **Official Website**: [https://nzbget.com/](https://nzbget.com/)
- **GitHub**: [https://github.com/nzbget/nzbget](https://github.com/nzbget/nzbget)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/nzbget](https://hub.docker.com/r/linuxserver/nzbget)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
NZBGET_PORT=6789              # Web UI port

# Traefik Configuration
NZBGET_REF=nzbget             # Traefik router reference
NZBGET_URL=nzbget.local       # Traefik hostname

# Path
DOWNLOAD_NZB_PATH=/path/to/usenet  # Download directory
```

### Volumes

- `./config:/config` - Configuration and database
- `${DOWNLOAD_NZB_PATH}:/downloads` - Download location

### Ports

- `6789` - Web UI (mapped to `${NZBGET_PORT}`)

## Initial Setup

1. **Set environment variables** in `.env`
2. **Start the service**: `docker compose up -d nzbget`
3. **Access web UI**: Navigate to `http://localhost:${NZBGET_PORT}`
4. **Default credentials**:
   - Username: `nzbget`
   - Password: `tegbzn6789`
5. **Change password**: Settings → Security
6. **Add Usenet server**: Settings → News-Servers

## Configuration Tips

### News Servers

**Settings → News-Servers**:

Add your Usenet provider(s):

- **Active**: Yes
- **Name**: Provider name
- **Level**: 0 (primary), 1 (backup)
- **Host**: news.provider.com
- **Port**: 563 (SSL) or 119
- **Username/Password**: From provider
- **Connections**: Based on provider limits
- **Encryption**: yes (TLS/SSL)
- **Cipher**: Empty (auto)

#### Server Priority

Use levels for fallback:

- **Level 0**: Primary unlimited server
- **Level 1**: Backup/block account servers
- NZBGet tries lower levels first

### Paths

**Settings → Paths**:

Default paths (already configured):

- **MainDir**: `/downloads`
- **DestDir**: `${MainDir}/completed`
- **InterDir**: `${MainDir}/intermediate`
- **NzbDir**: `${MainDir}/nzb`
- **QueueDir**: `${MainDir}/queue`
- **TempDir**: `${MainDir}/tmp`

### Categories

**Settings → Categories**:

Create categories for media types:

- **Name**: movies, tv, music, books
- **DestDir**: Optional custom destination
- **Unpack**: Yes
- **Extensions**: Optional file filters

Default categories included:

- Movies
- TV
- Music
- Books

### Download Queue

**Settings → Download Queue**:

- **WriteBuffer**: 1024 (KB) - Larger = faster, more RAM
- **ArticleCache**: 0 (auto) - Disk cache for articles
- **DirectWrite**: no - Buffered writes (faster)
- **CrcCheck**: yes - Verify downloads
- **FileNaming**: article - or nzb for original names
- **DupeCheck**: yes - Avoid duplicates

### Connection

**Settings → Connection**:

- **DailyQuota**: 0 (unlimited) - Set if provider has limits
- **DownloadRate**: 0 (unlimited) - Limit speed if needed
- **ArticleTimeout**: 60 seconds
- **UrlTimeout**: 60 seconds
- **RemoteTimeout**: 60 seconds
- **WriteBuffer**: 1024 KB
- **UMask**: 000 - File permissions

### Logging

**Settings → Logging**:

- **WriteLog**: append - Log to file
- **RotateLog**: 3 - Keep 3 old logs
- **ErrorTarget**: both - Console and log
- **WarningTarget**: both
- **InfoTarget**: both
- **DetailTarget**: log - Only to file
- **DebugTarget**: none - Unless troubleshooting

### Security

**Settings → Security**:

- **ControlIP**: 0.0.0.0 - Allow all (Docker network)
- **ControlUsername**: Change from default
- **ControlPassword**: Change from default
- **RestrictedUsername**: Optional read-only user
- **AddUsername**: Optional add-only user
- **SecureControl**: yes - SSL for web interface
- **SecureCert**: Optional custom certificate
- **AuthorizedIP**: Optional IP whitelist

### Extension Scripts

**Settings → Extension Scripts**:

Scripts for post-processing:

- **ScriptDir**: `/config/scripts`
- **Extensions**: Comma-separated script list
- Scripts run after download completion
- Can perform cleanup, notification, etc.

#### Popular Scripts

- **nzbToMedia**: For Arr integration
- **VideoSort**: Organize video files
- **FailureLink**: Handle failed downloads

## Arr Integration

### Add to Sonarr/Radarr/Lidarr

1. **Settings** → **Download Clients** → **Add** → **NZBGet**
2. Configure:
   - **Name**: NZBGet
   - **Host**: `nzbget` (Docker service name)
   - **Port**: `6789`
   - **Username**: Your username
   - **Password**: Your password
   - **Category**: Set per app (tv, movies, music)
   - **Recent Priority**: 0 (normal)
   - **Older Priority**: 0 (normal)
   - **Add Paused**: No
3. **Test** and **Save**

### nzbToMedia Script

Enhanced post-processing for Arr apps:

1. Download from [GitHub](https://github.com/clinton-hall/nzbToMedia)
2. Extract to `/config/scripts/nzbToMedia`
3. Configure `autoProcessMedia.cfg`
4. Set as post-processing script in NZBGet

## Performance Tuning

### Speed Optimization

For maximum performance:

**Download Queue**:

- **WriteBuffer**: 1024 or higher
- **ArticleCache**: 200-500 MB
- **DirectWrite**: no (buffered is faster)

**News-Servers**:

- **Connections**: Max allowed by provider (20-50)
- Multiple servers in parallel

**System**:

- **ParCheck**: auto - Automatic multicore
- **ParThreads**: 0 (auto-detect cores)

### Resource Usage

NZBGet is very efficient:

- Low CPU usage
- Minimal RAM (adjust buffers)
- Fast disk I/O with good settings

Monitor in **Status** tab.

## Troubleshooting

### Slow Downloads

- Increase server connections
- Check WriteBuffer setting
- Verify no bandwidth limits
- Ensure SSL isn't bottleneck (use fast cipher)
- Check multiple servers are active

### Incomplete Downloads

- Check server retention
- Add backup server
- Verify server credentials
- Check server status/downtime

### PAR2 Verification Fails

- Missing repair blocks
- Download different release
- Check disk space
- Verify file integrity

### Post-Processing Fails

- Check script permissions
- Review script logs
- Verify paths are correct
- Test script manually

### High CPU Usage

- ParCheck/Repair in progress (normal)
- Reduce ParThreads if needed
- Check for runaway scripts

## Backup

Backup config directory:

```bash
tar -czf nzbget-backup-$(date +%Y%m%d).tar.gz services/nzbget/config
```

Important files:

- `nzbget.conf` - Main configuration
- `scripts/` - Post-processing scripts

## Updates

```bash
docker compose pull nzbget
docker compose up -d nzbget
```

## Best Practices

1. **Use SSL**: Always enable encryption
2. **Change Default Password**: Security first
3. **Multiple Servers**: Primary + backup
4. **Tune WriteBuffer**: Balance speed/memory
5. **Enable ParCheck**: Verify downloads
6. **Categories**: Organize by type
7. **Monitor Performance**: Check Status tab
8. **Extension Scripts**: Automate post-processing
9. **Set DailyQuota**: If provider has limits
10. **Regular Backups**: Save configuration

## Advantages Over SABnzbd

- **Performance**: Faster, lower resource usage
- **Efficiency**: Written in C++
- **Features**: Powerful automation
- **Stability**: Very stable and reliable
- **Speed**: Generally faster downloads
- **Resources**: Less RAM/CPU needed

## When to Choose NZBGet

- Limited system resources
- Maximum performance needed
- Technical users comfortable with config
- Script-based automation preferred
