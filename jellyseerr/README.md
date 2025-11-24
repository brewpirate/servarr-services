# Jellyseerr

**Jellyseerr** is a fork of Overseerr built specifically for Jellyfin media servers. It's a request management and media discovery tool that allows users to request new content, which is then automatically sent to Sonarr/Radarr for download.

## Links

- **GitHub**: [https://github.com/Fallenbagel/jellyseerr](https://github.com/Fallenbagel/jellyseerr)
- **Documentation**: [https://docs.jellyseerr.dev/](https://docs.jellyseerr.dev/)
- **Docker Hub**: [https://hub.docker.com/r/fallenbagel/jellyseerr](https://hub.docker.com/r/fallenbagel/jellyseerr)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
JELLYSEERR_PORT=5055          # Web UI port

# Traefik Configuration
JELLYSEERR_REF=jellyseerr     # Traefik router reference
JELLYSEERR_URL=jellyseerr.local  # Traefik hostname
```

### Volumes

- `./config:/app/config` - Jellyseerr configuration and database

### Ports

- `5055` - Web UI (mapped to `${JELLYSEERR_PORT}`)

## Initial Setup

1. **Set environment variables** in `.env`
2. **Ensure Jellyfin is running**
3. **Start the service**: `docker compose --profile jellyfin up -d`
4. **Access web UI**: Navigate to `http://localhost:${JELLYSEERR_PORT}`
5. **Complete setup wizard**:
   - Sign in with Jellyfin account
   - Configure Jellyfin server connection
   - Add Radarr for movies
   - Add Sonarr for TV shows
   - Configure user permissions

## Setup Wizard

### Step 1: Jellyfin Server

- **Server Name**: Your Jellyfin server name
- **Hostname/IP**: `jellyfin` (Docker service name) or `http://jellyfin:8096`
- **Port**: `8096`
- **Use SSL**: If applicable
- **API Key**: Get from Jellyfin Dashboard → API Keys
- **Test Connection**: Verify connectivity

### Step 2: Radarr

- **Server Name**: Radarr
- **Hostname/IP**: `radarr` or `http://radarr:7878`
- **Port**: `7878`
- **API Key**: From Radarr Settings → General
- **Quality Profile**: Select default
- **Root Folder**: `/data` or your configured path
- **Minimum Availability**: Released or Announced
- **Tags**: Optional
- **External URL**: For links back to Radarr
- **Test Connection**

### Step 3: Sonarr

- **Server Name**: Sonarr
- **Hostname/IP**: `sonarr` or `http://sonarr:8989`
- **Port**: `8989`
- **API Key**: From Sonarr Settings → General
- **Quality Profile**: Select default
- **Root Folder**: `/tv` or your configured path
- **Language Profile**: Optional
- **Tags**: Optional
- **Season Folders**: Enable
- **Anime/Non-Anime**: Configure per preference
- **External URL**: For links back to Sonarr
- **Test Connection**

## User Management

### Jellyfin Authentication

Jellyseerr uses Jellyfin for authentication:

- Users sign in with Jellyfin credentials
- Permissions managed in Jellyseerr
- Admin users can manage settings

### User Roles

**Settings → Users**:

- **Admin**: Full access to all settings
- **User**: Can request media
- **Request Limits**: Set per user or globally

### Request Limits

Configure per user:

- **Movie requests per day/week/month**
- **TV show requests per day/week/month**
- **Unlimited requests**: For trusted users

## Configuration

### General Settings

**Settings → General**:

- **Application Title**: Custom title
- **Application URL**: External URL for notifications
- **Display Language**: UI language
- **Discover Region**: Content discovery region
- **Original Language**: Prefer original language content

### Media Management

**Settings → Radarr/Sonarr**:

- Add multiple Radarr/Sonarr instances
- Set default servers for requests
- Configure quality profiles per server
- Set root folders
- 4K servers separate from 1080p

### Notifications

**Settings → Notifications**:

Configure notifications for:

- **Request Status**: Approved, declined, available
- **Issue Reported**: User reports problem
- **Media Added**: New media available
- **Media Failed**: Download failed

#### Notification Agents

- **Email**: SMTP configuration
- **Discord**: Webhook URL
- **Telegram**: Bot token and chat ID
- **Slack**: Webhook URL
- **Webhook**: Custom webhooks
- **Pushover**: For mobile notifications
- **LunaSea**: Mobile app integration

### Auto-Approval

**Settings → Radarr/Sonarr → Request Options**:

- **Enable auto-approval**: Requests auto-approved
- **Auto-approve users**: Select users for auto-approval
- **Auto-approve 4K requests**: Separate 4K auto-approval

## Request Management

### Requesting Content

Users can request via:

1. **Search**: Find content via TMDb
2. **Discover**: Browse popular/trending/upcoming
3. **Request Button**: Click to request
4. **Options**: Select server, quality profile
5. **Submit**: Request sent to admins

### Managing Requests

**Requests Page**:

- View all pending requests
- Approve or decline
- Add to Sonarr/Radarr automatically
- Track request status
- View request history

### Request Status

- **Pending**: Awaiting approval
- **Approved**: Approved, sent to Sonarr/Radarr
- **Processing**: Being downloaded
- **Available**: Ready to watch in Jellyfin
- **Declined**: Request declined

## Issues

Users can report issues with media:

- **Video issues**: Quality, codec problems
- **Audio issues**: Missing audio, sync issues
- **Subtitle issues**: Missing or incorrect
- **Other**: Custom issue

Issues appear in dashboard for admin action.

## Discover

### Trending Content

- Movies/TV trending today or this week
- Upcoming releases
- Popular content
- Top rated

### Collections

Browse by:

- Genres
- Studios
- Networks
- Keywords

### Watchlist Integration

Sync with TMDb watchlist (if configured).

## 4K Configuration

Support separate 4K and 1080p instances:

### Dual Radarr Setup

1. Add 4K Radarr instance
2. Enable "4K Server" checkbox
3. Configure 4K quality profile
4. Set 4K root folder

Users can request both HD and 4K versions.

## Jellyfin Integration

### Availability Sync

Jellyseerr scans Jellyfin library:

- Marks available media
- Prevents duplicate requests
- Shows status in search results

### Sync Frequency

**Settings → Jellyfin**:

- Manual sync
- Automatic scanning schedule
- Runs every few hours by default

## Troubleshooting

### Cannot Connect to Jellyfin

- Verify Jellyfin is running
- Check hostname/IP is correct
- Use Docker service name: `jellyfin`
- Verify API key is valid
- Check network connectivity

### Cannot Connect to Radarr/Sonarr

- Verify services are running
- Use Docker service names
- Check API keys are correct
- Verify network connectivity
- Check external URLs if accessing from different networks

### Requests Not Appearing in Radarr/Sonarr

- Check quality profiles exist
- Verify root folders are correct
- Review Radarr/Sonarr logs
- Check API connectivity
- Ensure auto-approval is configured

### Users Cannot Log In

- Verify Jellyfin authentication is working
- Check user exists in Jellyfin
- Clear browser cache
- Check Jellyseerr logs

## Backup

Backup config directory:

```bash
tar -czf jellyseerr-backup-$(date +%Y%m%d).tar.gz services/jellyseerr/config
```

Important files:

- `db/db.sqlite3` - Main database
- `settings.json` - Configuration

## Updates

```bash
docker compose pull jellyseerr
docker compose --profile jellyfin up -d
```

## Best Practices

1. **Set Request Limits**: Prevent abuse
2. **Configure Auto-Approval**: For trusted users
3. **Enable Notifications**: Stay informed
4. **Regular Syncs**: Keep availability updated
5. **4K Separate**: Use separate instances for 4K
6. **User Training**: Educate users on requesting
7. **Monitor Requests**: Review regularly
8. **Issue Tracking**: Address reported issues promptly
9. **Quality Profiles**: Configure appropriately
10. **Backup Database**: Regular backups essential

## Integration Best Practices

### With Radarr/Sonarr

- Use meaningful quality profile names
- Configure multiple root folders for organization
- Use tags for organization
- Set minimum availability appropriately

### With Jellyfin

- Regular library scans
- Ensure proper naming in Jellyfin
- Configure metadata correctly

## Mobile Apps

Access Jellyseerr via:

- Mobile web browser
- LunaSea app (iOS/Android)
- Custom mobile apps (if available)

Responsive design works well on mobile.
