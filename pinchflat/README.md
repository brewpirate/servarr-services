# Pinchflat

**Pinchflat** is a YouTube media manager that helps you download and organize video content from YouTube channels, playlists, and subscriptions. It's designed to work like the Arr stack but specifically for YouTube content, making it easy to build and maintain a local YouTube media library.

## Links

- **GitHub**: [https://github.com/kieraneglin/pinchflat](https://github.com/kieraneglin/pinchflat)
- **Docker Hub**: [https://hub.docker.com/r/kieraneglin/pinchflat](https://hub.docker.com/r/kieraneglin/pinchflat)

## Configuration

### Environment Variables

The following environment variables are inherited from `service-base.compose.yaml`:

- `PUID`: Process User ID (default from `.env`)
- `PGID`: Process Group ID (default from `.env`)
- `TZ`: Timezone (default from `.env`)
- `UMASK`: File permission mask (default from `.env`)

Additional environment variables:

- `BASIC_AUTH_USERNAME=${PINCHFLAT_AUTH_USER}` - Basic auth username (optional)
- `BASIC_AUTH_PASSWORD=${PINCHFLAT_AUTH_PASS}` - Basic auth password (optional)

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
PINCHFLAT_PORT=8945       # Web UI port

# Traefik Configuration
PINCHFLAT_REF=pinchflat   # Traefik router reference
PINCHFLAT_URL=pinchflat.local  # Traefik hostname

# Authentication (optional)
PINCHFLAT_AUTH_USER=admin      # Basic auth username
PINCHFLAT_AUTH_PASS=password   # Basic auth password

# Media Path
DOWNLOAD_YOUTUBE_PATH=/path/to/youtube  # Path for YouTube downloads
```

### Volumes

- `./config:/config` - Pinchflat configuration and database
- `${DOWNLOAD_YOUTUBE_PATH}:/downloads` - YouTube download location
- `${ARR_MEDIA_PATH}:/media/arr` - Shared media path (from service-base)

### Ports

- `8945` - Web UI (mapped to `${PINCHFLAT_PORT}`)

## Initial Setup

1. **Set environment variables** in your `.env` file
2. **Create config directory**: `mkdir -p services/pinchflat/config`
3. **Start the service**: `docker compose up -d pinchflat`
4. **Access the web UI**: Navigate to `http://localhost:${PINCHFLAT_PORT}` or `http://${PINCHFLAT_URL}/`
5. **Begin adding sources**:
   - Add YouTube channels
   - Add playlists
   - Configure download settings

## Configuration Tips

### Adding Sources

Pinchflat supports multiple source types:

#### YouTube Channels

1. Navigate to **Sources** → **Add Source**
2. Enter channel URL (e.g., `https://www.youtube.com/@channelname`)
3. Configure download settings
4. Set schedule for checking new videos

#### Playlists

1. Add via **Sources** → **Add Source**
2. Enter playlist URL
3. Choose to monitor for new additions
4. Configure quality preferences

#### Individual Videos

1. Add via **Media** → **Add Media**
2. Enter video URL
3. Download immediately or schedule

### Download Settings

Configure per source or globally:

#### Quality Settings

- **Video Quality**: 1080p, 720p, 4K, etc.
- **Format**: MP4, MKV, WebM
- **Audio Quality**: Best, 320k, 128k
- **Prefer Format**: Choose preferred codecs (H.264, VP9, AV1)

#### Content Filters

- **Date Range**: Only download videos within date range
- **Duration**: Filter by video length (e.g., exclude shorts)
- **Title Filter**: Include/exclude by title keywords
- **Upload Date**: Only new videos or include history

#### Subtitles

- **Download Subtitles**: Enable/disable
- **Languages**: Select preferred languages
- **Auto-generated**: Include auto-generated subtitles
- **Embed**: Embed in video file or separate file

### Organization

#### File Naming

Configure naming templates:

- **Channel Videos**: `{channel}/{title} [{id}].{ext}`
- **Playlist Videos**: `{playlist}/{upload_date} - {title}.{ext}`
- **Custom Templates**: Use yt-dlp template variables

#### Folder Structure

Options:

- Organize by channel
- Organize by playlist
- Flat structure
- Custom organization

### Scheduling

Set automatic check schedules:

- **Check Interval**: How often to check for new videos
- **Download Immediately**: Auto-download new content
- **Queue Management**: Limit concurrent downloads
- **Retry Failed**: Retry failed downloads

### Metadata

Download and manage metadata:

- **Thumbnail**: Download video thumbnails
- **Description**: Save video descriptions
- **Comments**: Optionally save comments (resource-intensive)
- **NFO Files**: Create .nfo files for media servers

## Media Server Integration

### Jellyfin/Plex Integration

Organize downloads for media server compatibility:

1. Set output format to MP4 or MKV
2. Enable metadata download
3. Use consistent folder structure
4. Enable thumbnail download

#### Recommended Settings

- **Format**: MP4 (best compatibility)
- **Naming**: Include upload date for sorting
- **Metadata**: Enable NFO file creation
- **Thumbnails**: Download for poster art

### Library Structure

Suggested organization:

```
/youtube
  ├── Channels
  │   ├── Channel 1
  │   │   ├── video1.mp4
  │   │   ├── video1.jpg
  │   │   └── video1.nfo
  │   └── Channel 2
  └── Playlists
      └── Playlist Name
```

## Advanced Features

### SponsorBlock Integration

Skip sponsored segments:

- Enable SponsorBlock
- Configure segment types to skip (sponsor, intro, outro)
- Automatic segment removal or chapter markers

### Archive Mode

Keep track of all videos without downloading:

- Monitor channels without downloading everything
- Selective manual downloads
- Historical tracking

### Batch Operations

- Bulk add channels/playlists
- Mass update settings
- Bulk refresh metadata

### Download Limits

Control resource usage:

- **Concurrent Downloads**: Limit simultaneous downloads
- **Bandwidth Limiting**: Set download speed limits
- **Storage Quotas**: Set maximum storage per source
- **Age Limits**: Auto-delete old videos

## Notifications

Configure notifications for:

- New video downloads
- Failed downloads
- Storage warnings
- Update availability

Supported platforms:

- Email
- Discord
- Telegram
- Custom webhooks

## Troubleshooting

### Download Failures

- Check internet connectivity
- Verify YouTube URL is valid
- Check yt-dlp is updated (Pinchflat updates automatically)
- Review error logs
- Some videos may be geo-restricted

### Quality Not Available

- Requested quality may not exist for video
- Try lower quality setting
- Check format availability
- Some live streams have limited options

### Storage Issues

- Check available disk space
- Configure storage quotas
- Enable auto-cleanup of old videos
- Review download history for large files

### Slow Downloads

- Check bandwidth limits
- Reduce concurrent downloads
- Check network performance
- YouTube may be throttling

## Backup

Back up the config directory regularly:

```bash
tar -czf pinchflat-backup-$(date +%Y%m%d).tar.gz services/pinchflat/config
```

Important files:

- Database with channel/playlist configurations
- Download history
- Settings and preferences

## Updates

Pinchflat automatically updates yt-dlp. To update Pinchflat:

```bash
docker compose pull pinchflat
docker compose up -d pinchflat
```

## Best Practices

1. **Start with Channels**: Monitor favorite channels automatically
2. **Use Quality Profiles**: Set reasonable quality to save space
3. **Enable Metadata**: Better media server integration
4. **Set Schedules**: Regular checks for new content
5. **Use Filters**: Avoid downloading unwanted content (shorts, streams)
6. **Storage Management**: Enable auto-cleanup or set quotas
7. **Organize by Channel**: Easier to browse and manage
8. **Download Subtitles**: Accessibility and searchability
9. **SponsorBlock**: Save time by skipping sponsor segments
10. **Regular Monitoring**: Check for failed downloads

## Use Cases

### Educational Content

- Archive educational channels
- Build subject-specific libraries
- Offline access to tutorials

### Entertainment Archive

- Save favorite creators' content
- Build personal video collections
- Preserve content that may be deleted

### News and Commentary

- Archive news channels
- Track specific topics
- Build research libraries

### Music Videos

- Download music video playlists
- Organize by artist or genre
- High-quality audio extraction
