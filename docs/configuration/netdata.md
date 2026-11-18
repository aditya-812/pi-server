# Netdata

## Per-container bandwidth: `netreport`

**Added:** 2026-11-17 · **Documented:** 2026-11-18

### What it is
`/usr/local/bin/netreport` (owned by root, needs `jq`) asks the Netdata API on `localhost:19999` how much each container downloaded and uploaded. It also reports the Pi's network interfaces (`net.wlan0`, `net.eth0`). Output is sorted by download.

```bash
netreport            # last 24 hours
netreport today      # since 00:00
netreport week       # since Monday 00:00
netreport month      # since the 1st (calendar month)
netreport 720        # last N hours, e.g. for an ISP billing cycle
```
Columns: `DOWN_GB`, `UP_GB`, and `DATA_HRS` (hours in the window that actually have data).
**Method (fixed 2026-11-18):** Netdata returns 15-min average rates; each point is multiplied by its length (900 s) and the results are added up. Points with no data count as 0.
The original version asked for one average over the whole window (`points=1`). That undercounted about 5x: the Pi total for 24h came out as 1.00 GB vs 4.83 GB. The old copy is `/usr/local/bin/netreport.bak-20261118`.

### What it depends on
Netdata runs in the Portainer stack **monitoring** (Stacks → monitoring → Editor). Edit it there, not with CLI compose. The live container has:
- `network_mode: host` and `pid: host`
- `/proc` → `/host/proc`, `/sys` → `/host/sys`, and the Docker socket
- the volume `monitoring_netdata_cache` → `/var/cache/netdata`, which holds the metric history. It lives under the Docker data-root on the HDD (`/mnt/media/docker-root`).

Per-container charts are named `cgroup_<name>.net_eth0`.
The live stack is copied in `docs/portainer/PORTAINER_MONITORING_STACK_CONFIG.yml` (updated 2026-11-18).

### Limits
- **History starts 2026-11-17**, when Netdata was redeployed. Longer windows only cover what exists; check `DATA_HRS`.
- **Uplink is Ethernet since 2026-11-18 ~21:20.** `net.eth0` carries the traffic, and `net.wlan0` is a backup link (before that date, everything went over Wi-Fi). The Pi total is both added together: netreport prints both rows, and the widget adds them (`aggregation=sum`).
- **Host-network containers don't get a row.** This includes Netdata and Glances. Their traffic shows only in the Pi total (`net.eth0` + `net.wlan0`).
- **VPN group = one row.** gluetun, prowlarr and flaresolverr share one network namespace (since 2026-11-18). Netdata reports that namespace under a single chart, currently `flaresolverr`, so that row is all three. The gluetun and prowlarr rows only hold traffic from before 2026-11-18 19:20 and get no new data.
- **Jellyfin's row includes LAN streaming**, not just internet traffic.
- **Partly stopped containers:** since the fix, time with no data counts as 0, so they are no longer overcounted.
- **Accuracy:** the result can differ by about 2% depending on where the window starts (15-min point alignment). Netdata's hourly points lose partial hours (they gave 4.6 vs 5.14 GB), so the script avoids them.
- **`month` means since the 1st.** For the ISP billing cycle, pass hours instead.

### Verified
2026-11-18: the script is installed and executable, and `jq` is present. `netreport today` returned rows for 19 containers plus `net.wlan0` / `net.eth0`, with DATA_HRS 20.4.

### Undo
To restore the old script: `sudo cp -p /usr/local/bin/netreport.bak-20261118 /usr/local/bin/netreport`. To remove it: `sudo rm /usr/local/bin/netreport`. The source of the fixed version is `/home/media/scripts/netreport.new`.

## Glance widget: Bandwidth tabs (24 hours / 7 days / 30 days)

**Added:** 2026-11-18 · on the Network page of both dashboards · file `docker/glance-{local,ts}/config/includes/bandwidth.yml`

- A Glance `group` widget with 3 tabs: rolling 24 hours, 7 days and 30 days. Each tab shows the Pi total (`net.wlan0`) plus each container's ↓/↑ GB, sorted by download. The top 15 get their own rows and the rest are merged into "others". Cache: 15 min (24h tab), 1 h (7d and 30d tabs).
- Same method as netreport (netreport uses the v1 API per chart; the widget uses the v2 API grouped by container): 15-min points (96 / 672 / 2880), each × `view.update_every` / 8e6, added up. One template is shared by all three tabs through a YAML anchor (`&bwtemplate`).
- The 7d and 30d tabs only add up the history that exists. Until about 2026-12-17 the 30d tab is "since 2026-11-17".
- **Verified 2026-11-18:** the 24h Pi total of 4.80 GB matches v1 (1440 and 96 points). The kernel counter shows 5.65 GB since boot, which is consistent. jdownloader2 shows 3.50 GB, matching Glances' own counter. All 3 tabs render on both dashboards with no widget errors, in about 1 s.
- **Undo:** use git history for the earlier single-table 24h version. To remove the widget completely, delete the `- $include: includes/bandwidth.yml` line from both `glance.yml` files.
