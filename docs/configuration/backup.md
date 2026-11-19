# Backup & Recovery

**Added:** 2026-11-18 · **Tool:** [restic](https://restic.net) (encrypted, deduplicated, versioned)

## What is backed up
| Item | Why |
|---|---|
| App configs and databases (`/home/media/docker`) | Libraries, indexers, quality profiles, dashboards |
| Portainer data volume (stack files and their environment variables) | Stack definitions **including secrets**, which is why the backup is encrypted |
| Network and boot config (`/etc/netplan`, `/etc/systemd/resolved.conf.d`, `/etc/docker/daemon.json`, `/etc/fstab`, the docker.service drop-in) | Ethernet, DNS, Docker data-root, HDD mount |
| `/usr/local/bin/netreport`, the backup script and its timer | Restores the tooling |
| Consistent DB copies (`/mnt/media/backups/staging`) | `sqlite3 .backup` of every live SQLite DB, plus a `pg_dump` of Jellystat |

**Not backed up:** the media library (~1.2 TB), logs, poster/metadata caches (regenerated), and Docker images (re-pulled).
The FileBrowser DB is BoltDB, not SQLite, so it is only in the snapshot as a raw file (it rarely changes).

## Where it lives
1. **On the Pi's HDD** (automatic): `/mnt/media/backups/restic`. This protects against the SD card failing.
   - Runs nightly at 04:00 via `pi-backup.timer` (it catches up after downtime).
   - Keeps the **last 7 nights**; dedup keeps that at about the size of one copy.
   - Each run spot-checks 5% of the data.
2. **On the admin's Mac** (manual, on demand): `~/Backups/pi-restic`. This protects against the HDD failing or the Pi being lost.
   - Run `backup/pull-pi-backup.sh` on the Mac. It pulls `/mnt/media/backups/restic-mirror` (a media-readable copy of the encrypted repo) over the LAN, or over Tailscale when away.
   - It never deletes files on the Mac side, so older data survives pruning on the Pi.

**Restic password:** stored on the Pi in `/root/.restic-pass` and in the admin's password manager. **Without it, no copy can be restored.**

## Check it
```bash
systemctl list-timers pi-backup.timer                      # next run
sudo journalctl -u pi-backup -o cat | grep -E "WARN|ERROR|OK 20"
sudo env RESTIC_PASSWORD_FILE=/root/.restic-pass restic -r /mnt/media/backups/restic snapshots
```

## Restore
**One file or app** (Pi and HDD fine):
```bash
R="sudo env RESTIC_PASSWORD_FILE=/root/.restic-pass restic -r /mnt/media/backups/restic"
$R restore latest --target /tmp/restore --include /home/media/docker/sonarr/config
# Stop the app's container, then copy back from /tmp/restore/home/media/docker/...
# For SQLite DBs, use the consistent copy:
#   /tmp/restore/mnt/media/backups/staging/sqlite/home/media/docker/<app>/.../<name>.db
```
**SD card died** (HDD fine):
1. Flash Ubuntu 24.04, create user `media` (UID/GID 1004), then install Docker, restic and Tailscale.
2. Restore everything to `/tmp/restore` with the command above, without `--include`.
3. Copy back `/etc/netplan`, `/etc/docker/daemon.json`, `/etc/fstab`, `/etc/systemd/resolved.conf.d` and the docker.service drop-in. Then copy back `/home/media/docker`.
4. Start Docker. It reads its data-root from the HDD, so the images, volumes and Portainer stacks are already there.

**HDD died or Pi lost:** copy `~/Backups/pi-restic` from the Mac to the new disk, then follow the steps above with `-r <that path>`. To read the backup on the Mac itself: `brew install restic`, then `restic -r ~/Backups/pi-restic snapshots`.

**Verified 2026-11-19:**
- First backup OK: 746 MB, 1 snapshot. Mac pull OK: 19 s over LAN.
- Restore test: Sonarr `config.xml` is byte-identical to the live file, and the restored `sonarr.db` passes `PRAGMA integrity_check` = ok.

**Undo:** `sudo systemctl disable --now pi-backup.timer`, then delete `/mnt/media/backups`.
