# Overseerr

**Overseerr** is a request management and media discovery tool built to work with your existing Plex ecosystem. It allows users to request new movies and TV shows, which are automatically sent to Sonarr/Radarr for download.

## Links

- **Official Website**: [https://overseerr.dev/](https://overseerr.dev/)
- **GitHub**: [https://github.com/sct/overseerr](https://github.com/sct/overseerr)
- **Documentation**: [https://docs.overseerr.dev/](https://docs.overseerr.dev/)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/overseerr](https://hub.docker.com/r/linuxserver/overseerr)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
OVERSEERR_PORT=5055           # Web UI port

# Traefik Configuration
OVERSEERR_REF=overseerr       # Traefik router reference
OVERSEERR_URL=overseerr.local # Traefik hostname
```

### Volumes

- `./config:/config` - Overseerr configuration and database

### Ports

- `5055` - Web UI (mapped to `${OVERSEERR_PORT}`)

## Initial Setup

1. **Set environment variables** in `.env`
2. **Ensure Plex is running**
3. **Start the service**: `docker compose --profile plex up -d`
4. **Access web UI**: Navigate to `http://localhost:${OVERSEERR_PORT}`
5. **Complete setup wizard**:
   - Sign in with Plex account
   - Configure Plex server connection
   - Add Radarr for movies
   - Add Sonarr for TV shows
   - Configure user permissions

## Setup Wizard

### Step 1: Plex Authentication

- Sign in with your Plex account
- Overseerr uses Plex authentication
- Admin account automatically created

### Step 2: Plex Server

- Select your Plex server from list
- Or manually enter server details
- **Hostname**: `plex` or `http://plex:32400`
- Sync Plex libraries

### Step 3: Radarr

Configure Radarr for movie requests:

- **Default Server**: Enable
- **4K Server**: If separate 4K instance
- **Server Name**: Radarr
- **Hostname/IP**: `radarr` or `http://radarr:7878`
- **Port**: `7878`
- **API Key**: From Radarr Settings → General
- **Quality Profile**: Select default
- **Root Folder**: `/data`
- **Minimum Availability**: Released/Announced/Cinemas
- **Enable Scan**: Auto-scan for availability
- **Enable Automatic Search**: Immediate search on request
- **Tags**: Optional organization

### Step 4: Sonarr

Configure Sonarr for TV requests:

- **Default Server**: Enable
- **4K Server**: If separate 4K instance
- **Server Name**: Sonarr
- **Hostname/IP**: `sonarr` or `http://sonarr:8989`
- **Port**: `8989`
- **API Key**: From Sonarr Settings → General
- **Quality Profile**: Select default
- **Root Folder**: `/tv`
- **Language Profile**: If configured in Sonarr
- **Tags**: Optional
- **Season Folders**: Enable recommended
- **Enable Scan**: Auto-scan for availability
- **Enable Automatic Search**: Immediate search on request

## User Management

### Plex Authentication

Users sign in with Plex accounts:

- Managed Plex users
- Plex Friends
- Local Plex users

### User Permissions

**Settings → Users**:

Permissions include:

- **Admin**: Full control
- **Manage Requests**: Approve/decline requests
- **Manage Users**: User administration
- **Manage Issues**: Handle reported issues
- **Request Movies**: Can request movies
- **Request TV**: Can request TV shows
- **Request 4K**: Can request 4K content
- **Auto-Approve**: Requests auto-approved
- **Auto-Approve Movies**: Movies auto-approved
- **Auto-Approve TV**: TV shows auto-approved
- **Auto-Approve 4K**: 4K auto-approved

### Request Limits

Set per user or globally:

- **Movie Requests**: Per day/week/month
- **TV Requests**: Per day/week/month
- **Global Defaults**: Apply to all users

## Configuration

### General Settings

**Settings → General**:

- **Application Title**: Custom branding
- **Application URL**: External URL
- **Display Language**: UI language
- **Discover Region**: Filter content by region
- **Discover Language**: Content discovery language
- **Enable CSRF Protection**: Security setting
- **Enable Image Caching**: Performance
- **Display Language**: UI language preferences

### Plex Settings

**Settings → Plex**:

- **Server**: Plex server configuration
- **Libraries**: Select libraries to scan
- **Scan**: Manual or scheduled
- **Webhook**: Plex webhook URL for real-time updates

### Media Management

**Settings → Radarr** and **Settings → Sonarr**:

- Multiple server support
- 4K servers separate from HD
- Quality profiles per server
- Root folders configuration
- Automatic search settings
- Enable/disable scanning

### Notifications

**Settings → Notifications**:

Notification types:

- **Media Requested**: New request submitted
- **Media Auto-Approved**: Request auto-approved
- **Media Approved**: Admin approved request
- **Media Available**: Content ready in Plex
- **Media Failed**: Download failed
- **Media Declined**: Request declined
- **Issue Reported**: User reported issue
- **Issue Resolved**: Issue marked resolved

#### Notification Agents

- **Email**: SMTP configuration required
- **Discord**: Webhook URL
- **Telegram**: Bot token and chat ID
- **Slack**: Incoming webhook URL
- **Pushbullet**: Access token
- **Pushover**: User key and API token
- **Webhook**: Custom webhook endpoints
- **LunaSea**: Mobile app integration
- **Gotify**: Self-hosted notifications

### Public Access

**Settings → General → Public Settings**:

- **Enable Public Access**: Allow requests without login
- **Public Registration**: Users can create accounts

Useful for sharing with friends/family.

## Requesting Content

### Search

Users search for movies/TV shows:

1. Use search bar
2. Browse discovery sections
3. Click title for details
4. Click "Request" button
5. Select options (quality, server)
6. Submit request

### Request Options

- **Server Selection**: Choose Radarr/Sonarr instance
- **Quality Profile**: Select quality
- **Root Folder**: Destination folder
- **Seasons**: TV shows - select seasons to request
- **4K Request**: Toggle 4K separately

### Request Status

Track request progress:

- **Pending**: Awaiting approval
- **Approved**: Sent to Radarr/Sonarr
- **Processing**: Downloading
- **Available**: Ready in Plex
- **Declined**: Request denied

## Discover

Browse content:

- **Movies**: Trending, Popular, Upcoming
- **TV Shows**: Trending, Popular, Airing Today
- **Collections**: Movie collections
- **Genres**: Browse by genre
- **Networks**: Browse by network (TV)
- **Studios**: Browse by studio (Movies)

## Issue Reporting

Users can report problems:

- **Video Issues**: Quality, buffering
- **Audio Issues**: Sync, missing tracks
- **Subtitle Issues**: Missing, wrong language
- **Other**: General issues

Admins notified and can track resolution.

## 4K Support

Separate 4K infrastructure:

### Dual Setup

1. Configure separate Radarr/Sonarr for 4K
2. Check "4K Server" in settings
3. Different quality profiles
4. Different root folders
5. Users can request both HD and 4K

### Request Behavior

- Separate request buttons for HD/4K
- Independent approval for each
- Can limit 4K requests per user

## Plex Integration

### Library Sync

Overseerr syncs with Plex:

- Scans Plex libraries periodically
- Marks available content
- Prevents duplicate requests
- Updates request status

### Plex Webhooks

For real-time updates:

1. **Settings → Plex → Webhook URL**: Copy URL
2. In Plex **Settings → Webhooks**: Add Overseerr URL
3. Real-time availability updates

## Troubleshooting

### Cannot Connect to Plex

- Verify Plex is running
- Check hostname is correct: `plex`
- Ensure Plex authentication worked
- Check network connectivity

### Cannot Connect to Radarr/Sonarr

- Verify services are running
- Use Docker service names
- Check API keys
- Verify network connectivity

### Content Not Showing as Available

- Trigger manual Plex library scan
- Check Overseerr scan settings
- Verify library is selected in settings
- Check Plex library permissions

### Notifications Not Working

- Verify notification agent configuration
- Test notification agent
- Check notification type is enabled
- Review Overseerr logs

## Backup

```bash
tar -czf overseerr-backup-$(date +%Y%m%d).tar.gz services/overseerr/config
```

Important:

- `db/db.sqlite3` - Main database
- Configuration files

## Updates

```bash
docker compose pull overseerr
docker compose --profile plex up -d
```

## Best Practices

1. **Set Request Limits**: Prevent abuse
2. **Configure Auto-Approval**: For trusted users
3. **Enable Plex Webhooks**: Real-time updates
4. **Regular Library Scans**: Keep availability current
5. **4K Separate**: Use separate instances
6. **User Training**: Educate on proper requesting
7. **Monitor Requests**: Review regularly
8. **Configure Notifications**: Stay informed
9. **Issue Tracking**: Address promptly
10. **Backup Regularly**: Database essential

## Mobile Access

- Responsive web interface
- LunaSea app (iOS/Android)
- Mobile browsers work well

## Overseerr vs Jellyseerr

- **Overseerr**: For Plex
- **Jellyseerr**: For Jellyfin
- Nearly identical features
- Choose based on media server
