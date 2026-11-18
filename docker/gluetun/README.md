# Gluetun VPN for Prowlarr + FlareSolverr

This directory contains the gluetun VPN configuration for your media stack. Gluetun provides a VPN proxy that Prowlarr can use to route traffic through ProtonVPN.

## Quick Start

Since you're using Portainer, see the detailed guide in:

📖 **[Portainer Integration Guide](../docs/vpn-proxy/PORTAINER_INTEGRATION.md)**

## What's in This Directory

- **config/** - Gluetun configuration directory (created when container runs)

## Documentation

All documentation is in `/home/media/docs/vpn-proxy/`:

- [PORTAINER_INTEGRATION.md](../docs/vpn-proxy/PORTAINER_INTEGRATION.md) - Complete step-by-step guide for Portainer
- [PROTONVPN_SETUP.md](../docs/vpn-proxy/PROTONVPN_SETUP.md) - How to get your ProtonVPN credentials

## Basic Commands

```bash
# View logs
docker logs -f gluetun

# Check VPN IP
docker exec gluetun curl -s ifconfig.me

# Restart
docker restart gluetun
```

## Status

Active since 2026-11-18. Prowlarr and FlareSolverr run inside gluetun's network (see PORTAINER_INTEGRATION.md).
Do not add a watchtower label to gluetun.
