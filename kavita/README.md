# Kavita

**Kavita** is a fast, feature-rich, cross-platform reading server for all your comics, manga, and ebooks. It supports a wide range of formats and provides a beautiful, responsive web interface for reading on any device.

## Links

- **Official Website**: [https://www.kavitareader.com/](https://www.kavitareader.com/)
- **GitHub**: [https://github.com/Kareadita/Kavita](https://github.com/Kareadita/Kavita)
- **Wiki**: [https://wiki.kavitareader.com/](https://wiki.kavitareader.com/)
- **Docker Hub**: [https://hub.docker.com/r/jvmilazz0/kavita](https://hub.docker.com/r/jvmilazz0/kavita)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
KAVITA_PORT=5000              # Web UI port

# Media Paths
MEDIA_BASE_PATH=/path/to/media      # Base media path
MEDIA_EBOOKS_PATH=/path/to/books    # Books library
MEDIA_MANGA_PATH=/path/to/manga     # Manga library
MEDIA_COMICS_PATH=/path/to/comics   # Comics library
```

### Volumes

- `./config:/config` - Kavita configuration and database
- `${MEDIA_BASE_PATH}:/data` - Base media location
- `${MEDIA_EBOOKS_PATH}:/books` - Books
- `${MEDIA_MANGA_PATH}:/manga` - Manga
- `${MEDIA_COMICS_PATH}:/comics` - Comics

### Ports

- `5000` - Web UI (mapped to `${KAVITA_PORT}`)

## Supported Formats

### Comics

- CBZ, CBR, CB7, CBT
- ZIP, RAR, 7ZIP, TAR

### Manga

- Same as comics
- Supports right-to-left reading

### eBooks

- EPUB
- PDF (basic support)

### Archives

- ZIP, RAR, 7ZIP, TAR containing images

## Initial Setup

1. **Set environment variables** in `.env`
2. **Start the service**: `docker compose up -d kavita`
3. **Access web UI**: Navigate to `http://localhost:${KAVITA_PORT}`
4. **Create admin account**:
   - First user is automatically admin
   - Set username and password
5. **Add libraries**:
   - Navigate to Admin Dashboard
   - Add libraries for different media types

## Library Setup

### Adding Libraries

**Admin Dashboard** → **Libraries** → **Add Library**:

1. **Name**: Library name (e.g., "Comics", "Manga", "Books")
2. **Type**: Select type (Comic, Manga, Book)
3. **Folders**: Add folders to scan
   - `/books` for books
   - `/manga` for manga
   - `/comics` for comics
4. **Scan on Startup**: Enable
5. **Save**

### Library Types

- **Comic**: Left-to-right reading
- **Manga**: Right-to-left reading
- **Book**: eBook format (EPUB)

### File Organization

Recommended structure:

```
/books
  /Author Name
    /Series Name
      /Volume 1
        Book files
        
/comics
  /Publisher
    /Series Name
      /Volume 01
        Issue files
        
/manga
  /Series Name
    /Volume 01
      Chapter files
```

Kavita is flexible with naming but consistent organization helps.

## User Management

### Adding Users

**Admin Dashboard** → **Users** → **Invite User**:

1. **Username**: Set username
2. **Email**: Optional
3. **Role**: Admin or normal user
4. **Libraries**: Select accessible libraries
5. **Age Restriction**: Set if needed
6. **Save** and share credentials

### Roles

- **Admin**: Full access, can manage server
- **User**: Read access, personal settings

### Age Restrictions

Set per user to restrict mature content.

## Reading Experience

### Web Reader

Built-in web reader features:

- **Reading Modes**: Single page, double page, continuous scroll
- **Fit to**: Width, height, original
- **Reading Direction**: LTR (left-to-right) or RTL (right-to-left)
- **Background Color**: Black, white, custom
- **Margins**: Adjust page margins
- **Fullscreen**: Immersive reading
- **Page Splitting**: For double-page spreads

### Reading Progress

- Automatic progress tracking
- Resume where you left off
- Sync across devices
- Reading history

### Collections

Create custom collections:

- Group series together
- Organize by theme, genre, reading order
- Share collections with users

## Metadata

### Automatic Metadata

Kavita extracts metadata from:

- ComicInfo.xml (in CBZ files)
- Filenames
- Folder structure
- EPUB metadata

### Manual Metadata

Edit series/volume metadata:

- Cover images
- Summary
- Genres
- Tags
- Publication date
- Age rating

### Cover Images

Kavita extracts covers automatically:

- First page of comic/manga
- EPUB cover
- Or use custom uploaded cover

## Advanced Features

### OPDS

OPDS feed for mobile readers:

- **OPDS URL**: `http://your-server:5000/api/opds/`
- Use with compatible readers:
  - Chunky Reader (iOS)
  - Tachiyomi (Android)
  - KOReader
  - FBReader

### API

Full REST API available:

- Custom integrations
- Automation
- Third-party tools

### External Readers

Send to external readers:

- Kindle (via Send to Kindle)
- Email delivery
- Download for offline reading

### Reading Lists

Smart and manual reading lists:

- CBL file support
- Custom created lists
- Automatic population
- Share lists

### Statistics

Track reading stats:

- Pages read
- Time spent reading
- Completion percentages
- Reading streaks

## Mobile Apps

### Progressive Web App (PWA)

Install as app on mobile:

1. Access Kavita in browser
2. "Add to Home Screen"
3. Behaves like native app

### Third-Party Apps

Compatible with OPDS:

- **Chunky**: iOS comic reader
- **Tachiyomi**: Android manga reader
- **Moon+ Reader**: Android eBook reader
- **KOReader**: Multi-platform reader

## Settings

### Server Settings

**Admin Dashboard** → **Settings**:

- **Port**: Server port (5000 default)
- **Base URL**: If behind reverse proxy
- **Cache Directory**: Image cache location
- **Bookmarks Directory**: Reading progress storage
- **Task Frequency**: Scan schedules
- **API Key**: For external integrations

### Email Settings

For password reset and invitations:

- SMTP server configuration
- From address
- Authentication

### Security

- Force HTTPS (if configured)
- Authentication required
- Session timeout
- Password requirements

## Troubleshooting

### Library Not Scanning

- Check folder permissions
- Verify paths are correct
- Manual scan from library settings
- Check logs for errors

### Images Not Loading

- Check image cache directory
- Clear cache and rescan
- Verify file formats are supported
- Check disk space

### Cannot Login

- Check username/password
- Clear browser cache
- Check Kavita is running
- Review authentication logs

### Reading Progress Not Syncing

- Check user is logged in
- Verify database is writable
- Check network connectivity
- Review sync logs

## Backup

```bash
tar -czf kavita-backup-$(date +%Y%m%d).tar.gz services/kavita/config
```

Important files:

- `kavita.db` - Main database (reading progress, users, metadata)
- `config/appsettings.json` - Configuration
- `cache/` - Cover cache (regeneratable)

## Updates

```bash
docker compose pull kavita
docker compose up -d kavita
```

Kavita has auto-update notifications in admin dashboard.

## Best Practices

1. **Organize Files**: Consistent folder structure
2. **Include Metadata**: ComicInfo.xml files improve experience
3. **Regular Backups**: Database contains all progress
4. **User Permissions**: Set appropriate library access
5. **OPDS for Mobile**: Best mobile reading experience
6. **Age Ratings**: Set for appropriate content filtering
7. **Collections**: Organize reading orders
8. **Monitor Disk Space**: Image cache can grow
9. **Update Regularly**: New features frequently added
10. **Use Tags**: Organize and filter content

## Integration with Arr Apps

### Readarr

While not directly integrated, organize:

1. Readarr downloads to designated folder
2. Kavita scans that folder
3. New books automatically appear
4. Metadata from Readarr assists Kavita

### Mylar3

Similar to Readarr:

1. Mylar3 downloads comics
2. Kavita scans comic folder
3. Automatic library updates

## Performance

Kavita is very fast:

- Efficient image caching
- Lazy loading
- Optimized database
- Responsive on low-power devices

For large libraries (10k+ files):

- Initial scan takes time
- Subsequent scans are incremental
- Image cache improves performance
- SSD recommended for database

## Community

- Active Discord community
- Feature requests on GitHub
- Regular updates and improvements
- Responsive developer
