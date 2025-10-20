# Media Server Stack Documentation

**Date Created:** October 18, 2025  
**System:** Raspberry Pi 5 (ARM64)  
**User:** media (UID: 1004, GID: 1004)  
**Location:** /home/media  

## Overview

This is a comprehensive media server stack running on a Raspberry Pi 5 with an external HDD for media storage. The setup uses Docker containers managed through Portainer, providing a complete *arr suite for media management, Jellyfin for media serving, and various monitoring and utility tools.

## Hardware Configuration

- **Device:** Raspberry Pi 5 (ARM64)
- **OS:** Linux 6.8.0-1040-raspi
- **External Storage:** HDD mounted at `/mnt/media`
- **Hardware Acceleration:** DRI devices for Jellyfin video transcoding

## Storage Structure

```
/mnt/media/                    # External HDD mount point
├── Movies/                    # Radarr managed movies
├── TV-Shows/                  # Sonarr managed TV shows
└── Downloads/                 # qBittorrent downloads

/home/media/docker/            # Container configurations
├── jellyfin/config/           # Jellyfin configuration
├── qbittorrent/config/        # qBittorrent configuration
├── radarr/config/             # Radarr configuration
├── sonarr/config/             # Sonarr configuration
├── prowlarr/config/           # Prowlarr configuration
├── jellyseerr/config/         # Jellyseerr configuration
├── glance-local/config/       # Local Glance dashboard
├── glance-ts/config/          # Tailscale Glance dashboard
├── [REMOVED] glance/config/   # Main Glance (outdated, removed)
├── recyclarr/config/          # Recyclarr configuration
├── netdata/config/            # Netdata configuration
├── filebrowser/database/      # FileBrowser database
├── filebrowser/settings/      # FileBrowser settings
└── code-server/config/        # Code-server configuration
```

## Network Configuration

- **Local Network:** 192.168.29.160
- **Tailscale Network:** 100.80.146.24
- **Docker Network:** mediacenter_default

## Docker Compose Stacks

### Stack 1: Main Media Services
**Location:** `/var/lib/docker/volumes/portainer_data/_data/compose/1/`

#### Services:
1. **Jellyfin** (Port 8096)
   - Media server with hardware acceleration
   - Privileged container for DRI device access
   - Health checks enabled
   - Groups: 992 (render), 44 (video)

2. **qBittorrent** (Ports 8080, 6881)
   - Torrent client
   - Downloads to `/mnt/media/Downloads`

3. **Sonarr** (Port 8989)
   - TV show management
   - Health checks enabled
   - Manages `/mnt/media/TV-Shows`

4. **Radarr** (Port 7878)
   - Movie management
   - Health checks enabled
   - Manages `/mnt/media/Movies`

5. **Prowlarr** (Port 9696)
   - Indexer management
   - Centralized indexer configuration

6. **Jellyseerr** (Port 5055)
   - Request management for Jellyfin
   - User-friendly request interface

7. **Flaresolverr** (Port 8191)
   - CAPTCHA solving service
   - Supports Prowlarr indexers

8. **Glance Local** (Port 8280 on 192.168.29.160)
   - Local dashboard
   - Service monitoring

9. **Glance Tailscale** (Port 8280 on 100.80.146.24)
   - Remote dashboard via Tailscale
   - Same functionality as local

> **Note:** Main Glance configuration was removed as it was outdated and redundant.

10. **Recyclarr** (No exposed ports)
    - Quality management automation
    - Depends on Radarr and Prowlarr

11. **Watchtower** (No exposed ports)
    - Auto-updates containers
    - Runs daily with cleanup

### Stack 2: System Monitoring & Tools
**Location:** `/var/lib/docker/volumes/portainer_data/_data/compose/2/`

#### Services:
1. **Netdata** (Port 19999)
   - System performance monitoring
   - Real-time metrics

2. **Glances** (Port 61208)
   - System monitoring with web interface
   - Docker container monitoring

3. **FileBrowser** (Port 8085)
   - Web-based file manager
   - Access to media and home directories
   - Admin password: your-admin-password

4. **Code-Server** (Port 8443)
   - VS Code in browser
   - Access to media and home directories
   - Password: your-password

## Environment Variables

### Stack 1 Environment:
```bash
PUID=1004
PGID=1004
TZ=Asia/Kolkata
RADARR_API_KEY=your-radarr-api-key
SONARR_API_KEY=your-sonarr-api-key
PROWLARR_API_KEY=your-prowlarr-api-key
JELLYFIN_PUBLISHED_URL=http://192.168.29.160:8096
TS_HOST=100.80.146.24
```

### Stack 2 Environment:
```bash
CHANGEME_FILEBROWSER_PASSWORD=your-filebrowser-password
CHANGEME_CODE_SERVER_PASSWORD=your-code-server-password
```

## Service URLs

### Local Access (192.168.29.160):
- Jellyfin: http://192.168.29.160:8096
- qBittorrent: http://192.168.29.160:8080
- Sonarr: http://192.168.29.160:8989
- Radarr: http://192.168.29.160:7878
- Prowlarr: http://192.168.29.160:9696
- Jellyseerr: http://192.168.29.160:5055
- Glance Local: http://192.168.29.160:8280
- Netdata: http://192.168.29.160:19999
- Glances: http://192.168.29.160:61208
- FileBrowser: http://192.168.29.160:8085
- Code-Server: http://192.168.29.160:8443

### Remote Access (Tailscale - 100.80.146.24):
- Glance Remote: http://100.80.146.24:8280

## Key Features

1. **Hardware Acceleration:** Jellyfin uses DRI devices for video transcoding
2. **Auto-Updates:** Watchtower keeps all containers updated automatically
3. **Health Monitoring:** Built-in health checks for critical services
4. **Remote Access:** Tailscale integration for secure remote management
5. **Quality Management:** Recyclarr automates quality profile management
6. **Request Management:** Jellyseerr provides user-friendly request interface
7. **System Monitoring:** Netdata and Glances for comprehensive monitoring
8. **File Management:** FileBrowser for web-based file operations
9. **Development:** Code-Server for remote development

## Container Management

- **Orchestration:** Portainer (Port 9000)
- **Auto-restart:** All containers set to `unless-stopped`
- **Updates:** Watchtower handles automatic updates
- **Health Checks:** Enabled for Jellyfin, Sonarr, and Radarr

## Security Notes

- API keys are stored in environment files
- Passwords are set for FileBrowser and Code-Server
- Tailscale provides secure remote access
- Containers run with appropriate user permissions (PUID/PGID: 1004)

## Backup Locations

- **Docker Compose Files:** `/var/lib/docker/volumes/portainer_data/_data/compose/`
- **Container Configs:** `/home/media/docker/[service]/config/`
- **Media Data:** `/mnt/media/`
- **Environment Files:** In respective compose directories

## Maintenance

- **Logs:** Available in `/home/media/docker/[service]/config/logs/`
- **Updates:** Handled automatically by Watchtower
- **Monitoring:** Available through Netdata and Glances
- **File Management:** Available through FileBrowser

## Detailed Service Configurations

### Jellyfin (Media Streaming Server)
**Configuration Files:**
- Main config: `/home/media/docker/jellyfin/config/config/system.xml`
- Encoding: `/home/media/docker/jellyfin/config/config/encoding.xml`

**Key Settings:**
- **Hardware Acceleration**: Currently set to "none" but Docker has `/dev/dri` device access configured
- **Published URL**: http://192.168.29.160:8096
- **Transcoding Path**: `/cache/transcodes`
- **Plugins**: MusicBrainz, OMDB, Studio Images, TMDB
- **CORS Hosts**: Configured for cross-origin requests
- **Trickplay**: Enabled for video previews

**Docker Configuration:**
- Privileged mode enabled for hardware access
- Device mapping: `/dev/dri:/dev/dri`
- Environment: `XDG_CACHE_HOME=/cache`
- Volumes: config, cache, and media directories

### qBittorrent (Torrent Client)
**Configuration File:** `/home/media/docker/qbittorrent/config/qBittorrent/qBittorrent.conf`

**Key Settings:**
- **Default Save Path**: `/downloads`
- **Temp Path**: `/downloads/incomplete`
- **Port**: 6881
- **Web UI Address**: `*` (all interfaces)
- **Authentication**: Subnet whitelist configured
- **Categories**: 
  - `radarr`: For movie downloads
  - `tv-sonarr`: For TV show downloads

**Docker Configuration:**
- WebUI Port: 8080
- Download path: `/mnt/media/Downloads`
- Config persistence: `/home/media/docker/qbittorrent/config`

### Sonarr (TV Show Management)
**Configuration File:** `/home/media/docker/sonarr/config/config.xml`

**Key Settings:**
- **API Key**: your-sonarr-api-key
- **Port**: 8989
- **Authentication**: Forms-based, required
- **Log Level**: Debug
- **Update Mechanism**: Docker
- **Branch**: main

**Docker Configuration:**
- TV Shows path: `/mnt/media/TV-Shows`
- Downloads path: `/mnt/media/Downloads`
- Config persistence: `/home/media/docker/sonarr/config`

### Radarr (Movie Management)
**Configuration File:** `/home/media/docker/radarr/config/config.xml`

**Key Settings:**
- **API Key**: your-radarr-api-key
- **Port**: 7878
- **Authentication**: Forms-based, required
- **Log Level**: Debug
- **Update Mechanism**: Docker
- **Branch**: master

**Docker Configuration:**
- Movies path: `/mnt/media/Movies`
- Downloads path: `/mnt/media/Downloads`
- Config persistence: `/home/media/docker/radarr/config`

### Prowlarr (Indexer Management)
**Configuration File:** `/home/media/docker/prowlarr/config/config.xml`

**Key Settings:**
- **API Key**: your-prowlarr-api-key
- **Port**: 9696
- **Authentication**: Forms-based, required
- **Log Level**: Debug
- **Update Mechanism**: Docker
- **Indexer Definitions**: 500+ torrent site definitions configured

**Docker Configuration:**
- Config persistence: `/home/media/docker/prowlarr/config`
- Extensive indexer definitions for global torrent site support

### Jellyseerr (Request Management)
**Configuration File:** `/home/media/docker/jellyseerr/config/settings.json`

**Key Settings:**
- **Media Server Type**: 2 (Jellyfin)
- **Jellyfin Connection**: 
  - Host: `jellyfin`
  - Port: 8096
  - External hostname: http://192.168.29.160:8096
- **Default Permissions**: 32
- **Locale**: English
- **Local Login**: Enabled
- **Media Server Login**: Enabled

**Docker Configuration:**
- Port: 5055
- Config persistence: `/home/media/docker/jellyseerr/config`

### Glance Dashboards
**Configuration Files:**
- Local: `/home/media/docker/glance-local/config/glance.yml`
- Tailscale: `/home/media/docker/glance-ts/config/glance.yml`

**Key Features:**
- **Service Monitoring**: Real-time status of all media services
- **Custom Widgets**: Jellyfin stats, qBittorrent stats, recent releases
- **Dual Configuration**: Separate dashboards for local and Tailscale access
- **Responsive Design**: Custom CSS theming with dark mode
- **Include System**: Modular configuration with separate widget files

**Widget Includes:**
- `jellyfin-stats.yml`: Jellyfin server statistics
- `qbittorrent-stats.yml`: Torrent client statistics
- `radarr-releases.yml`: Recent movie releases
- `sonarr-releases.yml`: Recent TV show releases
- `media-page.yml`: Combined media overview page

### FileBrowser (File Management)
**Configuration File:** `/home/media/docker/filebrowser/settings/settings.json`

**Key Settings:**
- **Port**: 80 (internal)
- **Database**: `/database/filebrowser.db`
- **Root Directory**: `/srv`
- **Log Output**: stdout

**Docker Configuration:**
- External Port: 8085
- Access to: `/mnt/media`, `/home/media`, `/etc`
- Admin credentials configured via environment variables

### Recyclarr (Quality Management)
**Configuration File:** `/home/media/docker/recyclarr/config/config.yml`

**Purpose:**
- Synchronizes quality definitions and custom formats
- Maintains consistent quality profiles across Radarr and Sonarr
- Uses upstream templates for UHD Bluray + WEB profiles
- Automated quality management

### System Monitoring Services

#### Netdata
**Configuration File:** `/home/media/docker/netdata/config/netdata.conf`
- **Port**: 19999
- **Features**: Advanced system metrics, Docker monitoring
- **Access**: Host system resources, Docker socket

#### Glances
**Configuration**: Environment-based
- **Port**: 61208
- **Features**: System overview, Docker container monitoring
- **Options**: Configured via `GLANCES_OPT` environment variable

## API Integration Matrix

All services are interconnected via API keys for seamless automation:

| Service | API Key | Purpose |
|---------|---------|---------|
| Radarr | your-radarr-api-key | Movie management, Prowlarr integration |
| Sonarr | your-sonarr-api-key | TV show management, Prowlarr integration |
| Prowlarr | your-prowlarr-api-key | Indexer management, service coordination |

**Integration Features:**
- Prowlarr manages indexers for both Radarr and Sonarr
- Recyclarr syncs quality profiles using service APIs
- Glance dashboards display real-time service statistics
- Jellyseerr communicates with media management services for requests

## Configuration File Locations

### Primary Configuration Files:
```
/home/media/docker/
├── jellyfin/config/config/
│   ├── system.xml              # Main Jellyfin configuration
│   ├── encoding.xml            # Hardware acceleration settings
│   └── network.xml             # Network configuration
├── qbittorrent/config/qBittorrent/
│   ├── qBittorrent.conf        # Main qBittorrent settings
│   └── categories.json         # Download categories
├── sonarr/config/
│   └── config.xml              # Sonarr configuration
├── radarr/config/
│   └── config.xml              # Radarr configuration
├── prowlarr/config/
│   ├── config.xml              # Prowlarr configuration
│   └── Definitions/            # 500+ indexer definitions
├── jellyseerr/config/
│   └── settings.json           # Jellyseerr configuration
├── glance-local/config/
│   ├── glance.yml              # Local dashboard config
│   └── includes/               # Widget definitions
├── glance-ts/config/
│   └── glance.yml              # Tailscale dashboard config
├── recyclarr/config/
│   ├── config.yml              # Quality sync configuration
│   └── settings.yml            # Recyclarr settings
├── netdata/config/
│   └── netdata.conf            # System monitoring config
└── filebrowser/settings/
    └── settings.json           # File browser settings
```

---

**Last Updated:** October 18, 2025  
**Maintained By:** AI Assistant  
**Status:** Production Ready




