# Transmission

**Transmission** is a fast, easy, and free BitTorrent client with minimal resource usage. Known for its simplicity and efficiency, it's ideal for users who want a straightforward torrent client without excessive features.

## Links

- **Official Website**: [https://transmissionbt.com/](https://transmissionbt.com/)
- **GitHub**: [https://github.com/transmission/transmission](https://github.com/transmission/transmission)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/transmission](https://hub.docker.com/r/linuxserver/transmission)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
TRANSMISSION_UI_PORT=9091     # Web UI port
TRANSMISSION_DL_PORT=51413    # Torrent traffic port

# Traefik Configuration
TRANSMISSION_REF=transmission # Traefik router reference
TRANSMISSION_URL=transmission.local  # Traefik hostname

# Path
DOWNLOAD_TORRENT_PATH=/path/to/torrents  # Download directory
```

### Volumes

- `./config:/config` - Configuration
- `${DOWNLOAD_TORRENT_PATH}:/downloads` - Download location

### Ports

- `9091` - Web UI
- `51413` - Torrent traffic (TCP/UDP)

## Initial Setup

1. **Set environment variables** in `.env`
2. **Start the service**: `docker compose up -d transmission`
3. **Access web UI**: Navigate to `http://localhost:${TRANSMISSION_UI_PORT}`
4. **No default password**: Open by default (configure authentication)

## Configuration

Configuration is done via `settings.json` file or web UI.

### Important: Stop Before Manual Edits

If editing `settings.json` manually:

```bash
docker compose stop transmission
# Edit services/transmission/config/settings.json
docker compose start transmission
```

Transmission overwrites settings.json on exit.

### Key Settings

Edit `/config/settings.json`:

```json
{
    "download-dir": "/downloads/completed",
    "incomplete-dir": "/downloads/incomplete",
    "incomplete-dir-enabled": true,
    
    "rpc-authentication-required": true,
    "rpc-username": "admin",
    "rpc-password": "password",
    "rpc-whitelist-enabled": false,
    
    "peer-port": 51413,
    "peer-port-random-on-start": false,
    "port-forwarding-enabled": true,
    
    "speed-limit-down-enabled": false,
    "speed-limit-up-enabled": true,
    "speed-limit-up": 100,
    
    "ratio-limit-enabled": true,
    "ratio-limit": 2.0,
    "idle-seeding-limit-enabled": false,
    
    "encryption": 2,
    "dht-enabled": true,
    "pex-enabled": true,
    "lpd-enabled": true,
    
    "download-queue-enabled": true,
    "download-queue-size": 5,
    "seed-queue-enabled": true,
    "seed-queue-size": 10
}
```

### Authentication

Enable authentication:

```json
{
    "rpc-authentication-required": true,
    "rpc-username": "your-username",
    "rpc-password": "your-password",
    "rpc-whitelist-enabled": false
}
```

**Note**: Password is automatically hashed on restart.

### Network Settings

```json
{
    "peer-port": 51413,
    "peer-port-random-on-start": false,
    "port-forwarding-enabled": true,
    "utp-enabled": true,
    "peer-limit-global": 200,
    "peer-limit-per-torrent": 50
}
```

### Speed Limits

```json
{
    "speed-limit-down-enabled": false,
    "speed-limit-down": 100,
    "speed-limit-up-enabled": true,
    "speed-limit-up": 100,
    "upload-slots-per-torrent": 4
}
```

### Encryption

```json
{
    "encryption": 2
}
```

Values:

- `0`: Prefer unencrypted
- `1`: Prefer encrypted
- `2`: Require encrypted

### Queue Management

```json
{
    "download-queue-enabled": true,
    "download-queue-size": 5,
    "seed-queue-enabled": true,
    "seed-queue-size": 10,
    "queue-stalled-enabled": true,
    "queue-stalled-minutes": 30
}
```

### Seeding Limits

```json
{
    "ratio-limit-enabled": true,
    "ratio-limit": 2.0,
    "idle-seeding-limit-enabled": true,
    "idle-seeding-limit": 30
}
```

## Arr Integration

### Add to Sonarr/Radarr/Lidarr

1. **Settings** → **Download Clients** → **Add** → **Transmission**
2. Configure:
   - **Name**: Transmission
   - **Host**: `transmission` (Docker service name)
   - **Port**: `9091`
   - **Username**: Your username
   - **Password**: Your password
   - **Category**: Set per app
   - **Recent Priority**: Normal
   - **Older Priority**: Normal
3. **Test** and **Save**

### Note on Categories

Transmission doesn't have native category support like qBittorrent. Use labels or separate download paths per Arr app.

## VPN Configuration

To route Transmission through Gluetun VPN:

### compose.override.yaml

```yaml
services:
  transmission:
    network_mode: "service:gluetun"
```

### Gluetun Configuration

```yaml
services:
  gluetun:
    ports:
      - ${TRANSMISSION_UI_PORT}:9091
      - ${TRANSMISSION_DL_PORT}:51413
      - ${TRANSMISSION_DL_PORT}:51413/udp
```

## Alternative Web UIs

### Transmission Web Control

Enhanced web interface:

1. Download from [GitHub](https://github.com/ronggang/transmission-web-control)
2. Install via provided script or manually
3. Replace default web UI files
4. Refresh browser

### Combustion

Another alternative UI with better features than default.

## Remote Control

### transmission-remote

Command-line tool for remote control:

```bash
transmission-remote HOST:PORT -n USERNAME:PASSWORD [options]
```

Examples:

```bash
# List all torrents
transmission-remote transmission:9091 -n admin:password -l

# Add torrent
transmission-remote transmission:9091 -n admin:password -a "magnet:?..."

# Remove torrent
transmission-remote transmission:9091 -n admin:password -t 1 -r
```

## Troubleshooting

### Cannot Access Web UI

- Check container is running
- Verify port mapping
- Check `rpc-whitelist-enabled` is false or includes your IP
- Verify authentication settings

### Slow Downloads

- Check speed limits
- Verify port forwarding
- Ensure peer limits aren't too low
- Enable DHT, PEX, and LPD

### Settings Not Saving

- Stop Transmission before editing settings.json
- Transmission overwrites file on exit
- Check JSON syntax is valid
- Check file permissions

### Port Not Open

- Verify port forwarding in router
- Check VPN port forwarding if using VPN
- Use online port checker to verify
- Ensure `peer-port-random-on-start` is false

### High CPU/Memory Usage

- Reduce peer limits
- Lower queue sizes
- Check for disk I/O issues
- Transmission is generally very efficient

## Backup

Backup config directory:

```bash
tar -czf transmission-backup-$(date +%Y%m%d).tar.gz services/transmission/config
```

Important files:

- `settings.json` - Main configuration
- `resume/` - Torrent resume data
- `torrents/` - Torrent files

## Updates

```bash
docker compose pull transmission
docker compose up -d transmission
```

## Best Practices

1. **Use VPN**: Essential for privacy
2. **Enable Authentication**: Secure web interface
3. **Set Upload Limits**: Fair sharing without saturation
4. **Enable Encryption**: Require encrypted connections
5. **Port Forwarding**: Critical for performance
6. **Set Ratio Limits**: Automatic seeding management
7. **Use Incomplete Dir**: Separate complete/incomplete
8. **Regular Backups**: Save configuration
9. **Monitor Disk Space**: Avoid filling disk
10. **Keep Updated**: Security and bug fixes

## Transmission vs Others

### Advantages

- **Simplicity**: Clean, minimal interface
- **Efficiency**: Very low resource usage
- **Stability**: Extremely stable and reliable
- **Cross-platform**: Native clients for all platforms

### Disadvantages

- **Features**: Fewer features than qBittorrent/Deluge
- **No Categories**: Limited organization
- **Basic Web UI**: Functional but simple
- **No Search**: No built-in search engine

### Best For

- Users who want simplicity
- Low-resource systems
- Set-and-forget setups
- Those who don't need advanced features

## Advanced Configuration

### Blocklists

Block malicious peers:

```json
{
    "blocklist-enabled": true,
    "blocklist-url": "http://list.iblocklist.com/?list=..."
}
```

### Scripts

Run scripts on torrent completion:

```json
{
    "script-torrent-done-enabled": true,
    "script-torrent-done-filename": "/config/scripts/done.sh"
}
```

### Scheduling

Use `alt-speed` for scheduling:

```json
{
    "alt-speed-enabled": false,
    "alt-speed-up": 50,
    "alt-speed-down": 50,
    "alt-speed-time-enabled": true,
    "alt-speed-time-begin": 540,
    "alt-speed-time-end": 1020
}
```

Times in minutes from midnight.

## Environment Variables

LinuxServer image supports environment variables:

- `USER`: Set RPC username
- `PASS`: Set RPC password
- `PEERPORT`: Set peer port

These override settings.json values.
