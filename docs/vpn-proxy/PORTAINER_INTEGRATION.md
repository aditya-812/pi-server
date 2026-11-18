# Gluetun (ProtonVPN) for Prowlarr + FlareSolverr

**Last updated:** 2026-11-18 (re-enabled; the previous version of this guide is in git history)

## What it does
Prowlarr and FlareSolverr share gluetun's network (`network_mode: service:gluetun`), so all of their internet traffic leaves through ProtonVPN (Switzerland, OpenVPN UDP). If the VPN drops, gluetun's firewall blocks their traffic rather than leaking it over the home connection. qBittorrent and the other apps are not on the VPN.

**Why:** the ISP blocks several indexer sites. Proxy-only mode doesn't work, because some sites need the VPN *and* FlareSolverr, and Prowlarr applies only one indexer proxy per indexer.

A copy of the live stack is in `docs/portainer/PORTAINER_MEDIA_STACK_CONFIG.yml`.

## Stack pieces (mediastack)
```yaml
  gluetun:
    image: qmcgaw/gluetun:latest
    container_name: gluetun
    cap_add: [NET_ADMIN]
    devices: [/dev/net/tun:/dev/net/tun]
    volumes: [/home/media/docker/gluetun/config:/gluetun]
    environment:
      - VPN_SERVICE_PROVIDER=protonvpn
      - VPN_TYPE=openvpn
      - OPENVPN_USER=${OPENVPN_USER}        # set in Portainer stack env vars
      - OPENVPN_PASSWORD=${OPENVPN_PASSWORD}
      - SERVER_COUNTRIES=Switzerland
      - HTTPPROXY=on                         # :8888, Docker network only, currently unused
      - TZ=${TZ}
    ports: ["9696:9696", "8191:8191"]        # Prowlarr, FlareSolverr
    # no watchtower label on purpose (see Updating)

  prowlarr:     { network_mode: "service:gluetun", depends_on: [gluetun], ... no ports }
  flaresolverr: { network_mode: "service:gluetun", depends_on: [gluetun], ... no ports }
```
Credentials: the ProtonVPN **OpenVPN / IKEv2** username and password (not the Proton login). See `PROTONVPN_SETUP.md`. They live only in Portainer's stack environment variables.

## Addresses (what talks to what)
| From | To | URL |
|---|---|---|
| Prowlarr | FlareSolverr | `http://localhost:8191` (same namespace) |
| Prowlarr | Sonarr / Radarr | `http://sonarr:8989` etc. (container names resolve inside gluetun) |
| Sonarr / Radarr (any *arr app) | Prowlarr | `http://gluetun:9696` (the "Prowlarr Server" field in Prowlarr → Settings → Apps) |
| Glance | Prowlarr / FlareSolverr | `http://gluetun:9696`, `http://gluetun:8191` |
| Browser | Prowlarr UI | `http://192.168.29.160:9696` (unchanged) |

The hostnames `prowlarr` and `flaresolverr` no longer exist on the Docker network.

## Verify
```bash
docker ps --filter name=gluetun                 # (healthy)
docker logs --tail 30 gluetun                   # "Public IP address is ... (Switzerland"
docker exec prowlarr curl -s https://ipinfo.io/country   # CH
docker exec sonarr curl -s http://gluetun:9696/ping      # {"status": "OK"}
```
Then Prowlarr → Indexers → Test All.

## Updating
Watchtower must not recreate gluetun: prowlarr and flaresolverr would be left attached to a network namespace that no longer exists. To update: Portainer → mediastack → Update the stack with "re-pull image" on, then restart `prowlarr` and `flaresolverr` if they lose connectivity.

## Troubleshooting
- **Unhealthy for ~1 min after start:** the first server failed the TLS handshake, and gluetun retries another server after 15s. If it keeps happening, pin servers with `SERVER_HOSTNAMES=` (or switch to WireGuard).
- **AUTH_FAILED:** wrong OpenVPN credentials in the Portainer env vars, or the subscription has lapsed.
- **Subscription lapsed again:** remove the gluetun service, give prowlarr back `ports: ["9696:9696"]` and flaresolverr `ports: ["8191:8191"]` (no `network_mode`). Then in Prowlarr set FlareSolverr to `http://flaresolverr:8191` and the apps' Prowlarr Server to `http://prowlarr:9696`, and point Glance's check-urls back to `prowlarr:`/`flaresolverr:`.
