# ProtonVPN credentials for Gluetun

**Last updated:** 2026-11-18. The old version described a separate `gluetun-proxy` compose file with SOCKS5 on 1080. That setup was never used here, and gluetun has no SOCKS5 server. The old text is in git history.

1. Log in at https://account.protonvpn.com
2. Open **Account → OpenVPN / IKEv2 username**. Copy the username and password. These are NOT your Proton login.
3. In Portainer → Stacks → mediastack → **Environment variables**, set `OPENVPN_USER` and `OPENVPN_PASSWORD`.
4. Update the stack. Check with `docker logs --tail 30 gluetun` (look for a Swiss "Public IP address").

Never put these values in compose files, docs or git.

Setup and addresses: see `PORTAINER_INTEGRATION.md`.
