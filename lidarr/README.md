# Lidarr

**Lidarr** is a music collection manager for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new albums from your favorite artists and will interface with clients and indexers to grab, sort, and rename them. It can also be configured to automatically upgrade the quality of existing files in the library when a better quality format becomes available.

## Links

- **Official Website**: [https://lidarr.audio/](https://lidarr.audio/)
- **GitHub**: [https://github.com/Lidarr/Lidarr](https://github.com/Lidarr/Lidarr)
- **Wiki**: [https://wiki.servarr.com/lidarr](https://wiki.servarr.com/lidarr)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/lidarr](https://hub.docker.com/r/linuxserver/lidarr)

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
LIDARR_PORT=8686          # Web UI port

# Traefik Configuration
LIDARR_REF=lidarr         # Traefik router reference
LIDARR_URL=lidarr.local   # Traefik hostname

# Media Paths
MEDIA_MUSIC_PATH=/path/to/music      # Path to your music library
DOWNLOAD_MUSIC_PATH=/path/to/downloads  # Path for music downloads
```

### Volumes

- `./config:/config` - Lidarr configuration files and database
- `${MEDIA_MUSIC_PATH}:/music` - Music library location
- `${DOWNLOAD_MUSIC_PATH}:/downloads` - Download directory
- `${ARR_MEDIA_PATH}:/media/arr` - Shared media path (from service-base)

### Ports

- `8686` - Web UI (mapped to `${LIDARR_PORT}`)

## Initial Setup

1. **Set environment variables** in your `.env` file
2. **Create config directory**: `mkdir -p services/lidarr/config`
3. **Start the service**: `docker compose up -d lidarr`
4. **Access the web UI**: Navigate to `http://localhost:${LIDARR_PORT}` or `http://${LIDARR_URL}/`
5. **Complete setup**:
   - Set authentication
   - Add root folder: `/music`
   - Connect to Prowlarr for indexers
   - Add download clients
   - Configure metadata profiles

## Configuration Tips

### Metadata Profiles

Lidarr uses metadata profiles to determine which releases to grab:

1. Navigate to **Settings** → **Profiles** → **Metadata Profiles**
2. Default profile includes all release types
3. Create custom profiles to:
   - Prefer studio albums over singles
   - Include/exclude live albums, remixes, EPs
   - Filter by secondary types

### Quality Profiles

Configure in **Settings** → **Profiles** → **Quality Profiles**:

- Set preferred quality (FLAC, MP3 320, MP3 V0, etc.)
- Define upgrade path (e.g., MP3 → FLAC)
- Set cutoff quality

### Release Profiles

Use release profiles to fine-tune releases:

- Prefer or ignore specific release groups
- Prefer specific formats (CD, WEB, Vinyl)
- Ignore releases with unwanted tags

### Download Clients

Add download clients in **Settings** → **Download Clients**:

- **qBittorrent**: Host: `qbittorrent`, Port: `8080`
- **Deluge**: Host: `deluge`, Port: `8112`
- **SABnzbd**: Host: `sabnzbd`, Port: `8080`

Configure remote path mapping if download client paths differ from Lidarr.

### Indexers via Prowlarr

1. In Prowlarr, go to **Settings** → **Apps**
2. Add Lidarr application
3. Use Docker hostname: `http://lidarr:8686`
4. Enter API key from Lidarr **Settings** → **General**

### Media Management

Recommended settings (**Settings** → **Media Management**):

- **Rename Tracks**: Enable for consistent naming
- **Replace Illegal Characters**: Enable
- **Standard Track Format**: `{Artist Name}/{Album Title}/{track:00} - {Track Title}`
- **Artist Folder Format**: `{Artist Name}`
- **Album Folder Format**: `{Album Title} ({Release Year})`
- **Import Extra Files**: Enable for lyrics, artwork, etc.

### Music Tagging

Lidarr can tag music files with metadata:

- Enable **Write Metadata to Audio Files**
- Configure tags to write (MusicBrainz IDs recommended)
- Use **Scrub Audio Tags** to clean existing tags

### Lists

Import artists automatically from:

- **Spotify**: Import playlists and saved artists
- **Last.fm**: Import user libraries
- **MusicBrainz**: Import from collections
- **Lidarr Lists**: Share between instances

Configure in **Settings** → **Import Lists**.

## Integration with Music Services

### MusicBrainz

Lidarr uses MusicBrainz as its primary metadata source:

- Automatic metadata retrieval
- Manual search available
- Consider running local MusicBrainz mirror for faster searches

### Spotify Integration

Link Spotify account for:

- Importing artists and playlists
- Syncing new releases
- Better matching

## Troubleshooting

### No Results When Searching

- Check indexer configuration in Prowlarr
- Verify indexers support music/audio category
- Test indexers manually
- Music on Usenet can be limited; private trackers work better

### Import Failures

- Check path mappings between Lidarr and download clients
- Verify file permissions
- Ensure files meet quality requirements
- Check logs for specific errors

### Artist/Album Not Found

- Search MusicBrainz directly first
- Use MusicBrainz ID for exact matches
- Some obscure artists may not be in database
- Manual import may be necessary

### Track Numbering Issues

- Verify correct album edition in MusicBrainz
- Check disc numbering in multi-disc albums
- Use manual import to correct track order

## Backup

Back up the config directory regularly:

```bash
tar -czf lidarr-backup-$(date +%Y%m%d).tar.gz services/lidarr/config
```

Important files:

- `lidarr.db` - Main database
- `config.xml` - Configuration settings

## Updates

Update to the latest version:

```bash
docker compose pull lidarr
docker compose up -d lidarr
```

## Best Practices

1. **Use Metadata Profiles**: Filter unwanted release types early
2. **Quality Over Quantity**: Set reasonable quality cutoffs
3. **Tag Your Music**: Enable metadata writing for better organization
4. **Use Lists**: Automate discovery of new releases
5. **Private Trackers**: Public indexers have limited music content
6. **Monitor Settings**: Balance between monitored and unmonitored albums
7. **Path Consistency**: Ensure consistent naming across your setup
