# LazyLibrarian

**LazyLibrarian** is an automated eBook and audiobook manager for Usenet and BitTorrent. It's similar to Sonarr/Radarr but for books. It finds NZB and torrent downloads, manages your library, and can integrate with Calibre for eBook management.

## Links

- **Official Website**: [https://lazylibrarian.gitlab.io/](https://lazylibrarian.gitlab.io/)
- **GitLab**: [https://gitlab.com/LazyLibrarian/LazyLibrarian](https://gitlab.com/LazyLibrarian/LazyLibrarian)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/lazylibrarian](https://hub.docker.com/r/linuxserver/lazylibrarian)

## Configuration

### Environment Variables

The following environment variables are inherited from `service-base.compose.yaml`:

- `PUID`: Process User ID (default from `.env`)
- `PGID`: Process Group ID (default from `.env`)
- `TZ`: Timezone (default from `.env`)
- `UMASK`: File permission mask (default from `.env`)

Additional environment variables:

- `DOCKER_MODS=linuxserver/mods:universal-calibre|linuxserver/mods:lazylibrarian-ffmpeg` - Adds Calibre and FFmpeg support

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
LAZYLIB_PORT=5299         # Web UI port

# Traefik Configuration
LAZYLIB_REF=lazylibrarian # Traefik router reference
LAZYLIB_URL=lazylib.local # Traefik hostname

# Media Paths
MEDIA_EBOOKS_PATH=/path/to/books     # Path to your book library
DOWNLOAD_BOOKS_PATH=/path/to/downloads  # Path for book downloads
```

### Volumes

- `./config:/config` - LazyLibrarian configuration and database
- `${DOWNLOAD_BOOKS_PATH}:/downloads` - Download directory
- `${MEDIA_EBOOKS_PATH}:/books` - Book library location
- `${ARR_MEDIA_PATH}:/media/arr` - Shared media path (from service-base)

### Ports

- `5299` - Web UI (mapped to `${LAZYLIB_PORT}`)

### Docker Mods

This container includes two important mods:

1. **Calibre**: For eBook conversion and management
2. **FFmpeg**: For audiobook processing

## Initial Setup

1. **Set environment variables** in your `.env` file
2. **Create config directory**: `mkdir -p services/lazylibrarian/config`
3. **Start the service**: `docker compose up -d lazylibrarian`
4. **Access the web UI**: Navigate to `http://localhost:${LAZYLIB_PORT}` or `http://${LAZYLIB_URL}/`
5. **Complete setup**:
   - Run configuration wizard
   - Set download directory: `/downloads`
   - Set book directory: `/books`
   - Configure search providers

## Configuration Tips

### Search Providers

LazyLibrarian supports multiple search providers:

#### Usenet (NZB)

- **NZBHydra2**: Recommended meta-search
- **Prowlarr**: If configured for books
- Configure in **Config** → **Search Providers** → **Usenet**

#### Torrent

- Configure torrent providers in **Config** → **Search Providers** → **Torrent**
- Supports DHT and torznab feeds
- Can use Prowlarr/Jackett

#### Direct Download

- **Library Genesis (LibGen)**: Free eBook source
- **Google Books**: For book information
- **GoodReads**: For book metadata and wishlists

### Download Clients

#### NZB Clients

Configure in **Config** → **Downloaders** → **NZB Downloaders**:

- **SABnzbd**: Host: `sabnzbd`, Port: `8080`
- **NZBGet**: Host: `nzbget`, Port: `6789`

#### Torrent Clients

Configure in **Config** → **Downloaders** → **Torrent Downloaders**:

- **qBittorrent**: Host: `qbittorrent`, Port: `8080`
- **Deluge**: Host: `deluge`, Port: `8112`
- **Transmission**: Host: `transmission`, Port: `9091`

### Book Processing

Configure in **Config** → **Processing**:

- **eBook Format Preferences**: EPUB, MOBI, PDF, etc.
- **Audiobook Format Preferences**: M4B, MP3
- **Automatic Conversion**: Use Calibre for format conversion
- **Move or Copy**: Choose import behavior

### Author Management

- **Add Authors**: Search by name
- **Wanted Status**: Set books as wanted/skipped
- **Author Information**: Pulls from GoodReads, LibraryThing
- **Automatic Discovery**: Monitor new releases from authors

### Calibre Integration

With the Calibre mod installed:

1. Configure Calibre library path in **Config** → **Processing**
2. Enable **Use Calibre** option
3. Set preferred eBook formats for conversion
4. Configure automatic metadata updates

### Notifications

Configure in **Config** → **Notifications**:

- Email
- Telegram
- Discord
- Slack
- Custom scripts

### GoodReads Integration

1. Get GoodReads API key (Developer Key)
2. Configure in **Config** → **API Keys**
3. Import wishlists and reading lists
4. Automatic author discovery from shelves

## Advanced Features

### Magazine Management

LazyLibrarian can also manage magazines:

- Add magazines by name
- Download latest issues automatically
- Configure in **Magazines** tab

### Audiobook Support

- Searches for audiobooks separately
- Supports M4B and MP3 formats
- Uses FFmpeg for processing
- Configure preferred narrators

### eBook Conversion

Using the built-in Calibre mod:

- Convert between formats (EPUB ↔ MOBI, etc.)
- Set output format preferences
- Automatic conversion on import

### Import Existing Library

1. Navigate to **Manage** → **Import Alternative**
2. Select directory to import
3. Match books with online metadata
4. Process and organize

## Troubleshooting

### No Search Results

- Check search provider configuration
- Verify API keys are correct
- Test providers individually
- Book searches can be limited on public indexers

### Import Failures

- Check file permissions
- Verify path mappings
- Ensure book files are valid
- Check logs for specific errors

### Calibre Conversion Issues

- Verify Calibre mod is loaded
- Check conversion settings
- Test with single file first
- Review conversion logs

### Author Not Found

- Try alternative spellings
- Use GoodReads or Google Books directly
- Manual author entry available
- Check API rate limits

## Backup

Back up the config directory regularly:

```bash
tar -czf lazylibrarian-backup-$(date +%Y%m%d).tar.gz services/lazylibrarian/config
```

Important files:

- `lazylibrarian.db` - Main database
- `config.ini` - Configuration settings

## Updates

Update to the latest version:

```bash
docker compose pull lazylibrarian
docker compose up -d lazylibrarian
```

## Best Practices

1. **Use Multiple Search Providers**: Increase success rate
2. **Configure Calibre**: Essential for format management
3. **Set Up GoodReads**: Better metadata and discovery
4. **Monitor Favorite Authors**: Automatic new release detection
5. **Use Preferred Formats**: Set EPUB or MOBI based on your readers
6. **Regular Database Maintenance**: Clean up old wanted books
7. **LibGen as Fallback**: Great free source for many books
8. **Path Consistency**: Keep book organization consistent
