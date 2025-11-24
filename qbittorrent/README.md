# qBittorrent

**qBittorrent** is a free and reliable P2P BitTorrent client. It's feature-rich with a built-in search engine, torrent creation tool, bandwidth scheduler, IP filtering, and more. It's one of the most popular torrent clients and is highly recommended for use with the Arr stack.

## Links

- **Official Website**: [https://www.qbittorrent.org/](https://www.qbittorrent.org/)
- **GitHub**: [https://github.com/qbittorrent/qBittorrent](https://github.com/qbittorrent/qBittorrent)
- **Wiki**: [https://github.com/qbittorrent/qBittorrent/wiki](https://github.com/qbittorrent/qBittorrent/wiki)
- **Docker Hub**: [https://hub.docker.com/r/linuxserver/qbittorrent](https://hub.docker.com/r/linuxserver/qbittorrent)

## Configuration

### Environment Variables

Inherited from `service-base.compose.yaml`:

- `PUID`, `PGID`, `TZ`, `UMASK`

Additional variables:

- `WEBUI_PORT=${QBITTORRENT_UI_PORT}` - Web UI port
- `TORRENTING_PORT=${QBITTORRENT_DL_PORT}` - Torrent download port

### Required Environment Variables (in `.env`)

```bash
# Port Configuration
QBITTORRENT_UI_PORT=8080      # Web UI port
QBITTORRENT_DL_PORT=6881      # Torrent traffic port

# Traefik Configuration
QBITTORRENT_REF=qbittorrent   # Traefik router reference
QBITTORRENT_URL=qbit.local    # Traefik hostname

# Path
DOWNLOAD_TORRENT_PATH=/path/to/torrents  # Download directory
```

### Volumes

- `./config:/config` - Configuration and database
- `${DOWNLOAD_TORRENT_PATH}:/downloads` - Download location
- `${ARR_MEDIA_PATH}:/media/arr` - Shared media path

### Ports

- `8080` - Web UI
- `6881` - Torrent traffic (TCP/UDP)

### Advanced Configuration Options

#### Version Tags

LinuxServer.io provides multiple version tags for qBittorrent:

- `latest` - Stable qBittorrent releases (recommended)
- `libtorrentv1` - Static builds using libtorrent v1

To use a specific tag:

```yaml
image: lscr.io/linuxserver/qbittorrent:libtorrentv1
```

#### UMASK Configuration

The `UMASK` variable controls file permissions. Common values:

- `022` - Default, files: 644, dirs: 755 (owner write, all read)
- `002` - Group writable, files: 664, dirs: 775

Set in your `.env` file:

```bash
UMASK=022
```

#### Custom Port Configuration

**WEBUI_PORT**: To change the web UI port, you must update both sides of the port mapping AND set the environment variable:

```yaml
services:
  qbittorrent:
    ports:
      - "8123:8123"
    environment:
      - WEBUI_PORT=8123
```

**TORRENTING_PORT**: To change the BitTorrent connection port:

```yaml
services:
  qbittorrent:
    ports:
      - "6887:6887"
      - "6887:6887/udp"
    environment:
      - TORRENTING_PORT=6887
```

#### Docker Secrets Support

For sensitive environment variables, use Docker secrets with the `FILE__` prefix:

```yaml
environment:
  - FILE__PASSWORD=/run/secrets/qbt_password
secrets:
  - qbt_password
```

#### Architecture Support

The image supports:

- **x86-64** (amd64) - Intel/AMD processors
- **arm64** (aarch64) - Apple Silicon, Raspberry Pi 4/5, ARM servers

#### Docker Mods

Extend container functionality:

```yaml
environment:
  - DOCKER_MODS=linuxserver/mods:universal-package-install
```

Available mods at: [https://mods.linuxserver.io/?mod=qbittorrent](https://mods.linuxserver.io/?mod=qbittorrent)

#### Read-Only and Non-Root Support

This image supports both read-only filesystem and non-root operation. See the [Advanced Security Options](#advanced-security-options) section below.

## Initial Setup

1. **Set environment variables** in `.env`
2. **Start the service**: `docker compose up -d qbittorrent`
3. **Get temporary password**: Check logs for initial password

   ```bash
   docker logs qbittorrent | grep "temporary password"
   ```

4. **Access web UI**: Navigate to `http://localhost:${QBITTORRENT_UI_PORT}`
5. **Login**: Username: `admin`, Password: from logs
6. **Change password**: Tools → Options → Web UI → Authentication

## Configuration Tips

### Web UI Settings

**Tools → Options → Web UI**:

- Change default password
- Enable bypass authentication for localhost (if desired)
- Set language preference
- Configure alternative UI (optional: VueTorrent)

### Downloads

**Tools → Options → Downloads**:

- **Default Save Path**: `/downloads` (already set)
- **Keep incomplete torrents in**: `/downloads/incomplete` (recommended)
- **Auto-delete .torrent files**: Enable
- **Pre-allocate disk space**: Enable for better performance
- **When torrent finishes**: Actions for completed torrents

### Connection

**Tools → Options → Connection**:

- **Port used for incoming connections**: `6881` (default)
- **Use UPnP/NAT-PMP**: Enable if router supports
- **Connections limit**: Set based on your needs (e.g., 500)
- **Upload/Download slots**: Adjust for optimal performance

### Speed

**Tools → Options → Speed**:

- **Global rate limits**: Set if needed
- **Alternative rate limits**: Schedule slow periods
- **Rate limit transport overhead**: Enable

### BitTorrent

**Tools → Options → BitTorrent**:

- **Privacy → Enable DHT**: Yes (for public torrents)
- **Privacy → Enable PEX**: Yes
- **Privacy → Enable Local Peer Discovery**: Yes
- **Privacy → Encryption mode**: Prefer encryption
- **Privacy → Anonymous mode**: Enable if using VPN
- **Torrent Queueing**: Enable with reasonable limits
- **Share ratio limiting**: Set seed ratios (e.g., 2.0)

### Advanced

**Tools → Options → Advanced**:

- **Network Interface**: Select VPN interface if using VPN
- **Optional IP address to bind to**: VPN IP if applicable
- **Validate HTTPS tracker certificates**: Enable
- **Server-side request forgery (SSRF) mitigation**: Enable

## Arr Integration

### Add to Sonarr/Radarr/Lidarr

1. Go to **Settings** → **Download Clients** → **Add** → **qBittorrent**
2. Configure:
   - **Name**: qBittorrent
   - **Host**: `qbittorrent` (Docker service name)
   - **Port**: `8080`
   - **Username**: `admin`
   - **Password**: Your password
   - **Category**: Set specific category (e.g., `tv`, `movies`, `music`)
3. **Test** and **Save**

### Categories

Create categories in qBittorrent:

1. **Options** → **Downloads** → **Default Save Path**
2. Add categories: `tv`, `movies`, `music`, `books`, etc.
3. Set save paths per category (optional)

## VPN Configuration

To route qBittorrent through Gluetun VPN:

### compose.override.yaml

```yaml
services:
  qbittorrent:
    network_mode: "service:gluetun"
    # Remove ports section - they'll be defined in gluetun
```

### Gluetun Configuration

Add qBittorrent ports to Gluetun:

```yaml
services:
  gluetun:
    ports:
      - ${QBITTORRENT_UI_PORT}:8080
      - ${QBITTORRENT_DL_PORT}:6881
      - ${QBITTORRENT_DL_PORT}:6881/udp
```

### Verify VPN

1. Access qBittorrent Web UI
2. Check IP in Settings or use torrent IP checker
3. Ensure IP matches VPN location

## Alternative Web UIs

### VueTorrent

Modern, responsive web UI:

1. Download from [GitHub](https://github.com/WDaan/VueTorrent/releases)
2. Extract to qBittorrent config directory
3. **Options** → **Web UI** → **Use alternative Web UI**
4. Set location to VueTorrent path
5. Refresh browser

## Troubleshooting

### Can't Connect to Web UI

- Check container is running: `docker ps | grep qbittorrent`
- Verify port mapping
- Check firewall rules
- Review container logs

### Forgot Password

Reset admin password:

```bash
docker exec -it qbittorrent bash
rm -f /config/qBittorrent/qBittorrent.conf
exit
docker restart qbittorrent
# Check logs for new temp password
```

### Slow Download Speeds

- Check connection limits
- Verify port forwarding (especially with VPN)
- Increase connections per torrent
- Check ISP throttling
- Verify DHT is enabled

### VPN Kill Switch

If VPN drops, torrents should stop. Verify:

1. Set network interface to VPN in qBittorrent
2. Bind to VPN IP address
3. Test by disconnecting VPN

### Incomplete Downloads

- Check disk space
- Verify permissions
- Review error logs in qBittorrent
- Check torrent health (seeders)

## Advanced Security Options

### Read-Only Operation

For enhanced security, run the container with a read-only filesystem:

```yaml
services:
  qbittorrent:
    read_only: true
    tmpfs:
      - /tmp
      - /var/tmp
```

See [LinuxServer.io Read-Only Docs](https://docs.linuxserver.io/misc/read-only/) for details.

### Non-Root Operation

Run the container as a non-root user:

```yaml
services:
  qbittorrent:
    user: "1000:1000"  # Replace with your UID:GID
```

See [LinuxServer.io Non-Root Docs](https://docs.linuxserver.io/misc/non-root/) for details.

Note: When using `user:`, the PUID/PGID environment variables are ignored.

## Backup

Backup config directory:

```bash
tar -czf qbittorrent-backup-$(date +%Y%m%d).tar.gz services/qbittorrent/config
```

Important files:

- `qBittorrent/qBittorrent.conf` - Main configuration
- `qBittorrent/categories.json` - Categories
- `BT_backup/` - Torrent resume data

## Updates

```bash
docker compose pull qbittorrent
docker compose up -d qbittorrent
```

### Monitoring Updates

Check container version:

```bash
docker inspect -f '{{ index .Config.Labels "build_version" }}' qbittorrent
```

Check image version:

```bash
docker inspect -f '{{ index .Config.Labels "build_version" }}' lscr.io/linuxserver/qbittorrent:latest
```

For automated update notifications, consider using [Diun](https://crazymax.dev/diun/).

## Container Management

### Shell Access

Access the container shell while running:

```bash
docker exec -it qbittorrent /bin/bash
```

### Real-time Logs

Monitor container logs:

```bash
docker logs -f qbittorrent
```

### Rebuild Container

If you need to completely rebuild the container:

```bash
docker compose down qbittorrent
docker compose up -d qbittorrent --force-recreate
```

## Best Practices

1. **Use VPN**: Essential for privacy and security
2. **Set Upload Limits**: Be a good peer but don't saturate
3. **Enable Encryption**: Prefer encryption mode
4. **Use Categories**: Organize downloads by type
5. **Set Ratio Limits**: Seed to reasonable ratios
6. **Regular Backups**: Save your configuration
7. **Monitor Disk Space**: Set up alerts
8. **Keep Updated**: Regular updates for security
9. **Use Strong Password**: Protect web interface
10. **Port Forwarding**: Essential for good speeds (especially with VPN)
