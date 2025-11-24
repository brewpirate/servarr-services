# Bazarr

**Bazarr** is a companion application to Sonarr and Radarr that manages and downloads subtitles based on your requirements. It uses the video files and your Sonarr/Radarr configurations to search and download subtitles automatically.

## Links

- **Official Website**: [https://www.bazarr.media/](https://www.bazarr.media/)
- **GitHub**: [https://github.com/morpheus65535/bazarr](https://github.com/morpheus65535/bazarr)
- **Wiki**: [https://wiki.bazarr.media/](https://wiki.bazarr.media/)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/bazarr](https://hub.docker.com/r/linuxserver/bazarr)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
BAZARR_PORT=6767              # Web UI port

# Traefik Configuration
BAZARR_REF=bazarr             # Traefik router reference
BAZARR_URL=bazarr.local       # Traefik hostname

# Media Paths
MEDIA_MOVIES_PATH=/path/to/movies  # Movie library
MEDIA_SERIES_PATH=/path/to/tv      # TV library
```

### Volumes

- `./config:/config` - Bazarr configuration and database
- `${MEDIA_MOVIES_PATH}:/movies` - Movie library (same as Radarr)
- `${MEDIA_SERIES_PATH}:/tv` - TV library (same as Sonarr)

### Ports

- `6767` - Web UI (mapped to `${BAZARR_PORT}`)

## Initial Setup

1. **Set environment variables** in `.env`
2. **Ensure Sonarr/Radarr are running and configured**
3. **Start the service**: `docker compose up -d bazarr`
4. **Access web UI**: Navigate to `http://localhost:${BAZARR_PORT}`
5. **Complete setup wizard**:
   - Set authentication (recommended)
   - Configure Sonarr connection
   - Configure Radarr connection
   - Add subtitle providers
   - Set languages

## Sonarr/Radarr Integration

### Connect to Sonarr

**Settings** → **Sonarr**:

1. **Enabled**: Yes
2. **Hostname or IP**: `sonarr` (Docker service name)
3. **Listening Port**: `8989`
4. **Base URL**: Leave empty
5. **SSL**: No (internal network)
6. **API Key**: From Sonarr Settings → General
7. **Download Only Monitored**: Yes (recommended)
8. **Test** and **Save**

### Connect to Radarr

**Settings** → **Radarr**:

1. **Enabled**: Yes
2. **Hostname or IP**: `radarr`
3. **Listening Port**: `7878`
4. **Base URL**: Leave empty
5. **SSL**: No
6. **API Key**: From Radarr Settings → General
7. **Download Only Monitored**: Yes
8. **Test** and **Save**

## Subtitle Providers

### Adding Providers

**Settings** → **Providers**:

Popular providers:

- **OpenSubtitles**: Requires account
- **Subscene**: No account needed
- **Addic7ed**: Requires account
- **Podnapisi**: No account needed
- **TVSubtitles**: No account needed

### Provider Configuration

For each provider:

1. Enable provider
2. Enter credentials (if required)
3. Set priority (order of search)
4. Configure anti-captcha if needed
5. Test and save

### OpenSubtitles Setup

Most reliable provider:

1. Create account at [opensubtitles.com](https://www.opensubtitles.com/)
2. Get API key (VIP account for better limits)
3. Configure in Bazarr
4. Set high priority

## Languages

### Language Configuration

**Settings** → **Languages**:

1. **Languages Filter**: Select languages to download
   - English
   - Spanish
   - French
   - etc.
2. **Default Settings**: Apply to all shows/movies
3. **Languages Profiles**: Create custom profiles
4. **Single Language**: Enable if only want one language

### Language Profiles

Create profiles for different content:

- **Profile 1**: English only
- **Profile 2**: English + Spanish
- **Profile 3**: Multiple languages

Assign profiles per series/movie if needed.

## Subtitle Settings

### General Subtitle Settings

**Settings** → **Subtitles**:

- **Subtitle Folder**: Same folder as media (recommended) or custom
- **Upgrade Previously Downloaded Subtitles**: Enable to get better versions
- **Upgrade Manually Downloaded Subtitles**: Usually disable
- **Days to Upgrade**: How many days to continue looking for better subs
- **Minimal Score**: Minimum match score (85-90 recommended)
- **Perfect Score**: What score is considered perfect (95-100)
- **Use Scene Name**: Try scene release name first
- **Use Original Format**: Try original file format

### Anti-Captcha

Some providers use captchas. Configure anti-captcha service:

- **Anti-Captcha**: Paid service
- **Death by Captcha**: Paid service

Or use providers without captchas.

### Hearing Impaired

- **Use Hearing Impaired**: If needed
- **Don't Use Hearing Impaired**: Exclude if not needed

## Automatic Subtitle Search

Bazarr automatically searches for subtitles:

- When new episode/movie added to Sonarr/Radarr
- On schedule (daily)
- For missing subtitles
- For upgrades (better versions)

### Manual Search

For specific episode/movie:

1. Navigate to series/movie
2. Click subtitle icon
3. **Search** or **Manual Search**
4. Select desired subtitle
5. Download

## Scheduler

**Settings** → **Scheduler**:

Configure automatic tasks:

- **Search for Missing Subtitles**: Daily
- **Upgrade Subtitles**: Daily
- **Update Shows/Movies from Sonarr/Radarr**: Every 6 hours
- **Sync Subtitles**: If using embedded subs

Adjust schedules based on your needs.

## Post-Processing

**Settings** → **Subtitles** → **Post-Processing**:

Options:

- **Encoding**: UTF-8 (recommended)
- **Fix Reverse RTL**: For RTL languages
- **Remove HI Tags**: Clean hearing impaired tags
- **Fix Uppercase**: Convert all-caps to normal case
- **Reverse RTL**: Reverse right-to-left text

## Notifications

**Settings** → **Notifications**:

Get notified on:

- Subtitle downloaded
- Subtitle upgraded
- Search failed

Notification services:

- Email
- Discord
- Telegram
- Slack
- Gotify
- Custom webhooks

## Blacklist

**Settings** → **Blacklist**:

Blacklist poor quality subtitles:

- Automatically blacklist bad subs
- Manual blacklist
- Prevent re-downloading
- Set blacklist duration

## Troubleshooting

### No Subtitles Found

- Check providers are configured correctly
- Verify API keys/credentials
- Try different providers
- Check minimal score isn't too high
- Verify languages are set correctly

### Sonarr/Radarr Connection Failed

- Verify services are running
- Check hostnames (use Docker service names)
- Verify API keys
- Check network connectivity
- Test connection in Bazarr settings

### Subtitles Not Downloading

- Check file permissions
- Verify media paths match Sonarr/Radarr
- Review Bazarr logs
- Check provider status
- Verify minimal score settings

### Provider Rate Limited

- Multiple providers help
- Respect rate limits
- OpenSubtitles VIP has higher limits
- Adjust search frequency

### Path Mapping Issues

Ensure paths match:

- Bazarr `/movies` = Radarr `/data`
- Bazarr `/tv` = Sonarr `/tv`
- Docker service names, not localhost

## Backup

```bash
tar -czf bazarr-backup-$(date +%Y%m%d).tar.gz services/bazarr/config
```

Important files:

- `db/bazarr.db` - Main database
- `config.ini` - Configuration

## Updates

```bash
docker compose pull bazarr
docker compose up -d bazarr
```

## Best Practices

1. **Multiple Providers**: Redundancy improves success rate
2. **OpenSubtitles VIP**: Worth it for better limits
3. **Set Reasonable Scores**: 85-90 minimal score works well
4. **Enable Upgrades**: Better subtitles appear over time
5. **Use Language Profiles**: Flexibility per content
6. **Monitor Providers**: Check provider status regularly
7. **Regular Backups**: Save configuration and database
8. **Blacklist Bad Subs**: Keep quality high
9. **Notifications**: Stay informed
10. **Path Consistency**: Match Sonarr/Radarr exactly

## Advanced Features

### Manual Upload

Upload subtitle files manually:

- Click episode/movie
- Upload subtitle
- Select language
- Bazarr manages it

### Mass Editor

Bulk operations:

- Change language profiles
- Trigger mass subtitle search
- Update multiple items

### History

Track subtitle history:

- View all downloads
- Blacklist from history
- Re-download
- Statistics

### Wanted

View missing subtitles:

- All items without subs
- Search all at once
- Filter by series/movie

## Integration Tips

### With Sonarr/Radarr

- Use same paths
- Monitor same content
- Sync regularly
- Enable notifications in Sonarr/Radarr when Bazarr completes

### With Media Servers

Subtitles automatically available:

- Plex finds them
- Jellyfin finds them
- No additional configuration needed

## Performance

For large libraries:

- Initial scan takes time
- Enable smart scheduling
- Use SSD for database
- Monitor provider limits
