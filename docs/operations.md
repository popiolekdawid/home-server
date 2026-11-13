# Operations

These are the few checks worth running regularly. They focus on failures that
can silently reduce recoverability: a degraded array, a full filesystem, a
failed service, or an untested backup.

## Health check

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
df -h / /mnt/data
docker ps
sudo systemctl --failed
sudo journalctl -p warning -b --no-pager
```

For disk health:

```bash
sudo smartctl -H /dev/disk/by-id/<data-disk>
```

## Updates

Review Compose changes before applying them:

```bash
docker compose pull
docker compose config
docker compose up -d
docker compose ps
```

Keep image versions pinned in the public examples and update them deliberately.
Do not upgrade database images without checking the application's migration
notes and backup first.

## Low-power choices

- Leave the NVMe and data SSDs available for normal service use; aggressive
  standby settings can cause latency and unnecessary device wake-ups.
- Schedule photo indexing, media scans, backups, and VM workloads instead of
  running all of them continuously.
- Stop unused VMs and remove containers that are no longer part of the design.

After a change:

```bash
uptime
df -h / /mnt/data
powerprofilesctl get 2>/dev/null || true
```
