# Deluge

**Deluge** is a lightweight, cross-platform BitTorrent client with extensive plugin support. It features a server-client model, making it ideal for remote management and headless server setups.

## Links

- **Official Website**: [https://deluge-torrent.org/](https://deluge-torrent.org/)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/deluge](https://hub.docker.com/r/linuxserver/deluge)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
DELUGE_UI_PORT=8112           # Web UI port
DELUGE_DL_PORT=6881           # Torrent traffic port

# Traefik Configuration
DELUGE_REF=deluge             # Traefik router reference
DELUGE_URL=deluge.local       # Traefik hostname

# Path
DOWNLOAD_TORRENT_PATH=/path/to/torrents  # Download directory
```

### Volumes

- `./config:/config` - Configuration and state
- `${DOWNLOAD_TORRENT_PATH}:/downloads` - Download location

### Ports

- `8112` - Web UI
- `6881` - Torrent traffic (TCP/UDP)

## Initial Setup

1. **Set environment variables** in `.env`
2. **Start the service**: `docker compose up -d deluge`
3. **Access web UI**: Navigate to `http://localhost:${DELUGE_UI_PORT}`
4. **Default password**: `deluge`
5. **Change password**: Preferences → Interface → Password
6. **Connection Manager**: Connect to local daemon (auto-connects)

## Configuration Tips

### Preferences

Access via top menu **Preferences** icon.

#### Downloads

**Preferences → Downloads**:

- **Download to**: `/downloads/incomplete`
- **Move completed to**: `/downloads/completed`
- **Copy of .torrent files to**: `/config/torrents` (optional)
- **Auto-managed**: Enable for automatic queue management
- **Add torrents in Paused state**: Based on preference

#### Network

**Preferences → Network**:

- **Incoming Ports**: `6881-6881` (or range)
- **Random ports on start**: Disable
- **Outgoing Ports**: Random
- **Network Interface**: (VPN interface if using VPN)
- **TOS**: Leave default
- **Peer TOS**: Leave default

**Encryption**:

- **Inbound**: Forced
- **Outbound**: Forced
- **Level**: Either (RC4 or plaintext)

#### Bandwidth

**Preferences → Bandwidth**:

- **Global Limits**:
  - **Max Download Speed**: 0 (unlimited) or set limit
  - **Max Upload Speed**: Set reasonable limit
  - **Max Connections**: 200
  - **Max Upload Slots**: 4

- **Per-Torrent Limits**:
  - **Max Connections**: 50
  - **Max Upload Slots**: 3
  - **Max Download Speed**: -1 (unlimited)
  - **Max Upload Speed**: -1 (use global)

#### Queue

**Preferences → Queue**:

- **Total Active**: 8
- **Total Active Downloading**: 3
- **Total Active Seeding**: 5
- **Do not count slow torrents**: Enable
- **Share Ratio Limit**: 2.0 (or your preference)
- **Time Ratio**: -1 (disabled)
- **Seed Time Limit**: -1 (unlimited)
- **Remove torrents**: Based on preference

### Plugins

**Preferences → Plugins**:

Enable useful plugins:

- **Label**: Organize torrents with labels
- **AutoAdd**: Watch folder for .torrent files
- **Execute**: Run scripts on events
- **Scheduler**: Schedule speed limits
- **Notifications**: Get alerts

#### Label Plugin

Create labels for categories:

1. Enable Label plugin
2. Right-click in torrent list → Label → Add Label
3. Create: movies, tv, music, etc.
4. Configure auto-label rules (optional)

#### AutoAdd Plugin

**Preferences → AutoAdd**:

- Enable for automatic torrent loading
- Set watch folder
- Configure label/download location per watch folder

#### Execute Plugin

**Preferences → Execute**:

- Run custom scripts on torrent events
- Torrent Complete
- Torrent Added
- Torrent Removed

## Arr Integration

### Add to Sonarr/Radarr/Lidarr

1. **Settings** → **Download Clients** → **Add** → **Deluge**
2. Configure:
   - **Name**: Deluge
   - **Host**: `deluge` (Docker service name)
   - **Port**: `8112`
   - **Password**: Your password
   - **Label**: Set per app (tv, movies, music)
   - **Recent Priority**: Last
   - **Older Priority**: Last
3. **Test** and **Save**

### Labels Setup

Create labels in Deluge for each Arr app:

- tv (for Sonarr)
- movies (for Radarr)
- music (for Lidarr)

Configure move locations per label if desired.

## VPN Configuration

To route Deluge through Gluetun VPN:

### compose.override.yaml

```yaml
services:
  deluge:
    network_mode: "service:gluetun"
```

### Gluetun Configuration

```yaml
services:
  gluetun:
    ports:
      - ${DELUGE_UI_PORT}:8112
      - ${DELUGE_DL_PORT}:6881
      - ${DELUGE_DL_PORT}:6881/udp
```

### Network Interface Binding

In Deluge Preferences → Network:

- Set **Network Interface** to VPN interface (e.g., tun0)
- This ensures Deluge only uses VPN

## Troubleshooting

### Cannot Access Web UI

- Check container is running
- Verify port mapping
- Try default password: `deluge`
- Check firewall rules

### Slow Downloads

- Increase connections limits
- Check bandwidth settings
- Verify port forwarding (especially with VPN)
- Enable DHT and PEX

### Connection Issues

- Check if ports are forwarded
- Verify VPN port forwarding if using VPN
- Test with port check torrent
- Ensure network interface is correct

### Torrents Not Moving

- Check "Move completed to" path
- Verify permissions on destination
- Ensure enough disk space
- Check Deluge logs

### High CPU Usage

- Check number of active torrents
- Reduce connections per torrent
- Disable unnecessary plugins
- Check for disk I/O bottlenecks

## Backup

Backup config directory:

```bash
tar -czf deluge-backup-$(date +%Y%m%d).tar.gz services/deluge/config
```

Important files:

- `core.conf` - Main configuration
- `state/` - Torrent state files
- `label.conf` - Label configuration (if using plugin)

## Updates

```bash
docker compose pull deluge
docker compose up -d deluge
```

## Best Practices

1. **Use VPN**: Essential for torrenting privacy
2. **Set Upload Limits**: Be fair but don't saturate
3. **Use Labels**: Organize torrents
4. **Enable Encryption**: Force encryption
5. **Queue Management**: Auto-managed torrents
6. **Seed Ratios**: Set reasonable ratios
7. **Port Forwarding**: Essential for good speeds
8. **Network Interface**: Bind to VPN when using VPN
9. **Regular Backups**: Save configuration
10. **Monitor Disk Space**: Avoid filling disk

## Deluge vs qBittorrent

### Choose Deluge if

- You prefer plugin-based extensibility
- You want thin client/server architecture
- You use GTK desktop client

### Choose qBittorrent if

- You want built-in search engine
- You prefer modern web UI
- You want sequential downloading
- You need torrent creation tools

## Advanced Features

### Daemon Mode

Deluge runs as daemon (deluged) with web UI:

- Efficient resource usage
- Can connect multiple clients
- Ideal for always-on servers

### Thin Client

Connect desktop Deluge client to server:

1. Install Deluge on desktop
2. Preferences → Interface → Classic Mode
3. Connection Manager → Add server
4. Enter container IP and port 58846

### Custom Scripts

Use Execute plugin for automation:

- Post-processing scripts
- Notifications
- Custom file organization
- Integration with other services

### Web UI Customization

Deluge web UI is functional but basic.
Consider alternatives:

- Flood (modern UI for various clients)
- Remote desktop client for full features
