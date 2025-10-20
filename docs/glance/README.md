# Glance Dashboards Documentation

**Date Created:** October 18, 2025  
**System:** Raspberry Pi 5 Media Server  
**Location:** /home/media/docker/glance*/config/  

## Overview

The media server utilizes multiple Glance dashboard instances to provide comprehensive monitoring and quick access to all services. Glance is a self-hosted dashboard that aggregates service status, statistics, and provides quick navigation to all media server components.

## Dashboard Architecture

### Two Dashboard Configurations:

> **Note:** The main Glance configuration was removed as it was outdated and redundant.

1. **Glance Local** (`/home/media/docker/glance-local/config/glance.yml`)
   - Simplified local network access
   - Modular configuration with include files
   - Focused on local network (192.168.29.160)

2. **Glance Tailscale** (`/home/media/docker/glance-ts/config/glance.yml`)
   - Remote access via Tailscale network
   - Uses `${TS_HOST}` variable for dynamic IP resolution
   - Mirrors local functionality for remote users

## Network Configuration

### Local Network Access
- **IP Address**: 192.168.29.160
- **Port**: 8280
- **URL**: http://192.168.29.160:8280

### Tailscale Network Access
- **IP Address**: 100.80.146.24 (via `${TS_HOST}` variable)
- **Port**: 8280/8281
- **URL**: http://100.80.146.24:8280

## Service Monitoring Matrix

All dashboards monitor these core services:

| Service | Local Port | Tailscale Port | Purpose |
|---------|------------|----------------|---------|
| Jellyfin | 8096 | 8096 | Media streaming server |
| Radarr | 7878 | 7878 | Movie management |
| Sonarr | 8989 | 8989 | TV show management |
| Prowlarr | 9696 | 9696 | Indexer management |
| qBittorrent | 8080 | 8080 | Torrent client |
| FileBrowser | 8085 | 8085 | File management |
| Netdata | 19999 | 19999 | System monitoring |
| Glances | 61208 | 61208 | System overview |
| Code-Server | 8443 | 8443 | Web-based IDE |

## Dashboard Pages Structure

### Main Glance Configuration

#### Local Network Pages:
1. **Home (Local)**
   - Clock widget with 24-hour format
   - Quick search (DuckDuckGo)
   - Bookmark groups for quick navigation
   - System snapshot with HTML widget
   - Service status monitoring
   - API-based queue widgets for Radarr/Sonarr
   - Embedded Netdata and Glances iframes

2. **Media (Local)**
   - Complete media services monitoring
   - Quick links to media applications
   - Download queue status

3. **Monitoring (Local)**
   - System monitoring services
   - Full-page embedded monitoring tools

4. **Tools (Local)**
   - Administrative tools access
   - File management and development tools

5. **Downloads (Local)**
   - Torrent client monitoring
   - Download queue management

#### Tailscale Network Pages:
- Mirror structure of local pages
- All URLs use `${TS_HOST}` variable
- Same functionality for remote access

### Glance Local Configuration

#### Simplified Structure:
1. **Home Page**
   - Clock and search widgets
   - System snapshot
   - All services monitoring in single view
   - Embedded monitoring tools

2. **Media Page** (via include)
   - Modular widget system
   - Separate include files for different components

## Advanced Widget System

### Custom API Widgets

#### 1. Jellyfin Statistics Widget
**File**: `glance-local/config/includes/jellyfin-stats.yml`

**Features:**
- Movie count display
- TV show count display  
- Episode count display
- Song count display
- API-based data retrieval
- Error handling for API failures

**Configuration:**
```yaml
- type: custom-api
  title: "Jellyfin/Emby Stats"
  base-url: http://192.168.29.160:8096
  options:
    url: http://192.168.29.160:8096
    key: "${JELLYFIN_API_KEY}"
```

#### 2. qBittorrent Statistics Widget
**File**: `glance-local/config/includes/qbittorrent-stats.yml`

**Features:**
- Real-time download/upload speeds
- Active torrent counts (seeding/leeching)
- Detailed torrent progress bars
- ETA calculations
- Collapsible torrent lists
- Multiple view modes (basic/detailed)
- Speed unit conversion (KiB/s, MiB/s, MB/s)

**Configuration Options:**
- `view`: "basic" or "detailed"
- `mode`: "default" or "upload"
- Cache: 10 seconds for real-time updates

#### 3. Radarr Releases Widget
**File**: `glance-local/config/includes/radarr-releases.yml`

**Features:**
- Recent movie releases tracking
- Upcoming movie calendar
- Missing movies display
- Configurable time intervals
- Movie poster integration
- Progress tracking
- API key integration

**Configuration Options:**
- `service`: radarr
- `type`: recent, upcoming, or missing
- `size`: small, medium, large, or huge
- `collapse-after`: Number of items before collapse
- `interval`: Days to look back/forward

#### 4. Sonarr Releases Widget
**File**: `glance-local/config/includes/sonarr-releases.yml`

**Features:**
- TV show episode tracking
- Season management
- Air date monitoring
- Episode progress tracking
- Series poster integration

### Queue Monitoring Widgets

#### Radarr Queue Widget
- Real-time download queue monitoring
- API-based data retrieval
- Error handling and status codes
- Separate configurations for local and Tailscale

#### Sonarr Queue Widget
- TV show download queue tracking
- Episode-specific monitoring
- Progress and ETA display

## Theme and Styling

### Custom CSS Theme
**File**: `glance/assets/user.css`

**Color Scheme:**
```css
:root {
  --bg: #0f1115;        /* Dark background */
  --card: #111418;       /* Card background */
  --muted: #9aa7b2;      /* Muted text */
  --accent: #7cc0ff;     /* Accent color (blue) */
  --text: #e6eef6;       /* Primary text */
}
```

**Styling Features:**
- Dark theme optimized for media server use
- Gradient card backgrounds
- Rounded corners (12px border-radius)
- Box shadows for depth
- Consistent accent color throughout
- High contrast for readability

## Configuration Management

### Environment Variables

#### API Keys Integration:
- `${RADARR_API_KEY}`: your-radarr-api-key-here
- `${SONARR_API_KEY}`: your-sonarr-api-key-here
- `${PROWLARR_API_KEY}`: your-prowlarr-api-key-here
- `${TS_HOST}`: 100.80.146.24

#### Network Variables:
- Local network: 192.168.29.160
- Tailscale network: ${TS_HOST}

### Modular Configuration System

#### Include Files Structure:
```
glance-local/config/includes/
├── jellyfin-stats.yml      # Jellyfin media statistics
├── qbittorrent-stats.yml   # Torrent client statistics  
├── radarr-releases.yml     # Movie releases tracking
├── sonarr-releases.yml     # TV show releases tracking
└── media-page.yml          # Combined media page layout
```

#### Benefits:
- Maintainable configuration
- Reusable components
- Easy customization
- Organized by functionality

## Docker Integration

### Container Configuration

#### Main Glance Container:
```yaml
glance:
  image: glanceapp/glance
  container_name: glance
  ports:
    - "8280:8080"
  volumes:
    - /home/media/docker/glance/config:/app/glance.yml
    - /home/media/docker/glance/assets:/app/assets
  environment:
    - TZ=Asia/Kolkata
```

#### Network Binding:
- **Glance Local**: Binds to 192.168.29.160:8280
- **Glance Tailscale**: Binds to 100.80.146.24:8280

## API Integration Details

### Service APIs Used:

#### Jellyfin API:
- **Endpoint**: `/emby/Items/Counts`
- **Purpose**: Media library statistics
- **Authentication**: API key-based
- **Data**: Movie, TV show, episode, and song counts

#### qBittorrent API:
- **Endpoints**: 
  - `/api/v2/transfer/info` (transfer statistics)
  - `/api/v2/torrents/info` (torrent details)
- **Purpose**: Real-time torrent monitoring
- **Authentication**: Session-based
- **Data**: Download/upload speeds, torrent status, progress

#### Radarr API:
- **Endpoints**:
  - `/api/v3/queue` (download queue)
  - `/api/v3/history/since` (recent activity)
  - `/api/v3/calendar` (upcoming releases)
- **Purpose**: Movie management monitoring
- **Authentication**: X-Api-Key header
- **Data**: Queue status, download progress, release dates

#### Sonarr API:
- **Endpoints**:
  - `/api/v3/queue` (download queue)
  - `/api/v3/history/since` (recent activity)
  - `/api/v3/calendar` (upcoming episodes)
- **Purpose**: TV show management monitoring
- **Authentication**: X-Api-Key header
- **Data**: Episode queue, download status, air dates

## Widget Types and Features

### Standard Widgets:
1. **Clock**: 24-hour format display
2. **Search**: Integrated search engines (Google, DuckDuckGo)
3. **Bookmarks**: Organized link groups with color coding
4. **Monitor**: Service status monitoring with health checks
5. **HTML**: Custom HTML content for system information
6. **Iframe**: Embedded external services

### Custom API Widgets:
1. **Media Statistics**: Library counts and metrics
2. **Download Queues**: Real-time download monitoring
3. **Release Tracking**: Upcoming and recent media releases
4. **System Metrics**: Performance and resource monitoring

## Responsive Design Features

### Layout System:
- **Column-based layout**: small, full-width columns
- **Collapsible containers**: Expandable content sections
- **Responsive iframes**: Embedded monitoring tools
- **Flexible widgets**: Adaptive sizing based on content

### Mobile Optimization:
- Touch-friendly interface
- Responsive breakpoints
- Optimized widget sizing
- Readable typography

## Maintenance and Updates

### Configuration Reload:
```bash
# Restart Glance container to reload configuration
docker restart glance
docker restart glance-local
docker restart glance-ts
```

### Log Monitoring:
```bash
# View Glance logs
docker logs glance
docker logs glance-local  
docker logs glance-ts
```

### Configuration Validation:
- YAML syntax validation
- API endpoint testing
- Service connectivity verification
- Widget functionality testing

## Troubleshooting

### Common Issues:

#### 1. API Widget Failures:
- **Cause**: Invalid API keys or unreachable services
- **Solution**: Verify API keys in environment variables
- **Check**: Service availability and network connectivity

#### 2. Iframe Loading Issues:
- **Cause**: CORS restrictions or service unavailability
- **Solution**: Configure service CORS settings
- **Alternative**: Use direct links instead of embedded iframes

#### 3. Theme Not Loading:
- **Cause**: CSS file path issues or container volume mounting
- **Solution**: Verify volume mounts and file permissions
- **Check**: CSS file accessibility within container

#### 4. Environment Variable Resolution:
- **Cause**: Variables not properly passed to container
- **Solution**: Verify Docker Compose environment configuration
- **Check**: Variable substitution in rendered configuration

## Security Considerations

### API Key Management:
- API keys stored in environment variables
- No hardcoded credentials in configuration files
- Secure key rotation procedures

### Network Security:
- Local network access restrictions
- Tailscale encrypted tunnel for remote access
- Service-specific authentication requirements

### Container Security:
- Non-root container execution
- Minimal required permissions
- Regular image updates

## Performance Optimization

### Caching Strategy:
- API widget caching (10s-30m depending on data volatility)
- Static asset caching
- Efficient polling intervals

### Resource Management:
- Lightweight container footprint
- Optimized API request frequency
- Efficient widget rendering

---

**Last Updated:** October 18, 2025  
**Maintained By:** AI Assistant  
**Status:** Production Ready

This documentation provides comprehensive coverage of the Glance dashboard system, enabling effective monitoring and management of the entire media server infrastructure through intuitive web interfaces.
