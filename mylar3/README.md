# Mylar3

**Mylar3** is an automated comic book (cbr/cbz) downloader program for use with NZB and torrents. It's designed to manage your comic book collection, monitor for new issues, and automatically download them when available.

## Links

- **GitHub**: [https://github.com/mylar3/mylar3](https://github.com/mylar3/mylar3)
- **Wiki**: [https://github.com/mylar3/mylar3/wiki](https://github.com/mylar3/mylar3/wiki)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/mylar3](https://hub.docker.com/r/linuxserver/mylar3)

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
MYLAR_3_PORT=8090         # Web UI port

# Traefik Configuration
MYLAR_3_REF=mylar3        # Traefik router reference
MYLAR_3_URL=mylar.local   # Traefik hostname

# Media Paths
MEDIA_COMICS_PATH=/path/to/comics           # Path to your comic library
DOWNLOAD_COMICS_PATH=/path/to/downloads     # Path for comic downloads
```

### Volumes

- `./config:/config` - Mylar3 configuration and database
- `${MEDIA_COMICS_PATH}:/comics` - Comic library location
- `${DOWNLOAD_COMICS_PATH}/comics:/downloads` - Download directory
- `${ARR_MEDIA_PATH}:/media/arr` - Shared media path (from service-base)

### Ports

- `8090` - Web UI (mapped to `${MYLAR_3_PORT}`)

## Initial Setup

1. **Set environment variables** in your `.env` file
2. **Create config directory**: `mkdir -p services/mylar3/config`
3. **Start the service**: `docker compose up -d mylar3`
4. **Access the web UI**: Navigate to `http://localhost:${MYLAR_3_PORT}` or `http://${MYLAR_3_URL}/`
5. **Complete setup**:
   - Run first-time configuration wizard
   - Set comic location: `/comics`
   - Set download directory: `/downloads`
   - Configure Comic Vine API key (required)

## Configuration Tips

### Comic Vine API

Mylar3 requires a Comic Vine API key for metadata:

1. Create account at [Comic Vine](https://comicvine.gamespot.com/)
2. Get API key from your profile
3. Enter in **Settings** → **Web Interface** → **Comic Vine API**

### Search Providers

#### Usenet

Configure in **Settings** → **Search Providers**:

- **Newznab**: Configure manually or via Prowlarr
- **NZBHydra2**: Recommended meta-search
- Enable providers and set priorities

#### Torrents

- Configure torrent providers in **Search Providers**
- Supports torznab feeds (Prowlarr/Jackett)
- Public torrent DHT search available

### Download Clients

#### NZB Clients

Configure in **Settings** → **Download Settings** → **SABnzbd** or **NZBGet**:

- **SABnzbd**: Host: `http://sabnzbd:8080`, API key required
- **NZBGet**: Host: `http://nzbget:6789`, username/password required

#### Torrent Clients

Configure in **Settings** → **Download Settings**:

- **qBittorrent**: Host: `qbittorrent`, Port: `8080`
- **Deluge**: Host: `deluge`, Port: `8112`
- **Transmission**: Host: `transmission`, Port: `9091`

### Quality Settings

Configure in **Settings** → **Quality & Post Processing**:

- **Preferred Quality**: CBZ or CBR
- **Minimum File Size**: Set to avoid bad downloads
- **Allow Packs**: Enable for story arc packs
- **Replace Comics**: Auto-upgrade to better quality

### Post Processing

**Settings** → **Quality & Post Processing**:

- **Enable Post Processing**: Automatic import
- **Folder Monitoring**: Watch download folder
- **Comic Location**: `/comics`
- **Auto-Rename**: Standardize file naming
- **Convert CBR to CBZ**: Optional conversion

### Series Management

Add series in multiple ways:

1. **Search**: Search by series name
2. **Import**: Import existing library
3. **Story Arcs**: Add complete story arcs
4. **Pull List**: Import from weekly pull lists

### Weekly Pull List

Mylar3 can automatically grab new weekly releases:

1. Navigate to **Weekly Pull List**
2. Configure automatic pull list download
3. Select series to auto-download
4. Set schedule for pull list refresh

### Story Arcs

Manage complete story arcs:

1. Go to **Story Arcs**
2. Search for arc by name
3. Add entire arc
4. Monitor for completion

### File Management

**Settings** → **Quality & Post Processing**:

- **Folder Format**: `$Series ($Year)`
- **File Format**: `$Series $Issue ($Year)`
- **Zero-Padding**: Enable for proper sorting
- **Replace Spaces**: Use underscores or keep spaces

## Prowlarr Integration

While Mylar3 doesn't have native Prowlarr app integration:

1. Use Prowlarr's Newznab/Torznab URLs
2. Add as custom indexers in Mylar3
3. Get URLs from Prowlarr indexer pages

## Notifications

Configure in **Settings** → **Notifications**:

- Email
- Telegram
- Discord
- Slack
- Boxcar
- Prowl
- Custom scripts

## Troubleshooting

### Comic Vine API Issues

- Verify API key is correct
- Check API rate limits (not exceeded)
- Try different metadata source if available

### No Search Results

- Check search provider configuration
- Verify indexer connectivity
- Comics can be limited on public indexers
- Try alternative search terms

### Import Issues

- Verify file naming matches expected format
- Check file permissions
- Ensure Comic Vine match is correct
- Manual intervention may be needed

### Missing Issues

- Check if issue exists in Comic Vine database
- Verify issue number format
- Some publishers use different numbering
- Manual search may be necessary

### Post Processing Failures

- Check download path is correct
- Verify folder monitoring is enabled
- Review post-processing logs
- Check file permissions

## Backup

Back up the config directory regularly:

```bash
tar -czf mylar3-backup-$(date +%Y%m%d).tar.gz services/mylar3/config
```

Important files:

- `mylar.db` - Main database
- `config.ini` - Configuration settings

## Updates

Update to the latest version:

```bash
docker compose pull mylar3
docker compose up -d mylar3
```

## Best Practices

1. **Get Comic Vine API Key**: Essential for metadata
2. **Use Weekly Pull List**: Automate new releases
3. **Configure Quality Settings**: Avoid bad downloads
4. **Enable Post Processing**: Automatic import and rename
5. **Monitor Story Arcs**: Track crossovers and events
6. **Organize by Publisher**: Use folder structure
7. **Regular Library Updates**: Keep metadata current
8. **Set File Size Limits**: Filter out incomplete files
9. **Use Multiple Providers**: Improve success rate
10. **Tag Series**: Organize by reading order or events

## Advanced Features

### CV Import

Import reading lists from Comic Vine:

- Add series from CV reading lists
- Bulk import multiple series
- Automatic metadata

### Metadata Management

- **Refresh Series**: Update from Comic Vine
- **Bulk Operations**: Mass update multiple series
- **Manual Search**: Override automatic results

### Reading List Management

- Create custom reading lists
- Export lists
- Share with other Mylar instances
- Integration with comic readers
