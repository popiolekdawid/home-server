# Backup and recovery

RAID1 keeps the data available when one SSD fails. It does not recover deleted
files, corrupted files, ransomware damage, or a lost host. Keep one encrypted
off-site copy and one independently controlled local copy.

## Back up

Protect these separately:

- `/mnt/data` user files, photos, documents, and irreplaceable media;
- `/srv/app-data` service configuration and state;
- database-consistent dumps for Immich, Paperless, and other databases;
- Nextcloud's documented AIO backup;
- `/etc/fstab`, `/etc/mdadm/mdadm.conf`, firewall rules, and Compose files;
- libvirt definitions and VM disks that matter.

Do not back up Docker's `overlay2` directory, caches, transcodes, or generated
downloads unless they are meaningful to you. Keep secrets in a password
manager or a separate encrypted store.

Example dry run:

```bash
rclone sync --dry-run /mnt/data backup:example-bucket/data
```

The repository's example environment uses `backup:example-bucket` only as a
placeholder. Configure the remote locally.

## Restore test

At least once, restore:

1. One ordinary file.
2. One database into an isolated location.
3. One complete service configuration.

Record how long it took and what was missing. A backup job that exits
successfully is not proof that the data can be restored.

## Rebuild after OS-disk loss

1. Install Ubuntu on the replacement NVMe.
2. Install `mdadm`, inspect both SSDs by serial number, and assemble the
   existing array with `sudo mdadm --assemble --scan`.
3. Mount the confirmed filesystem UUID at `/mnt/data`.
4. Reinstall Docker and recreate `/srv/app-data`.
5. Restore configuration, database dumps, and secrets from their separate
   stores.
6. Start services one at a time with `docker compose config` and
   `docker compose up -d`.
7. Test SMB, web access, photo metadata, and a backup restore.

Useful checks:

```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
findmnt /mnt/data
systemctl --failed
```
