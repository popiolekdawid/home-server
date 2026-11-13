# Services

The repository includes sanitized Compose manifests under
[`../examples/compose/`](../examples/compose/). They show the actual service
layout without copying credentials, private addresses, or local filesystem
names.

## Directory layout

```text
/srv/app-data/
├── immich/
├── jellyfin/
├── nextcloud/
├── paperless/
├── pihole/
└── ...

/mnt/data/
├── documents/
├── media/
└── photos/
```

Use `/srv/app-data` for configuration, databases, and caches. Use `/mnt/data`
for photos, documents, and media. Keep each Compose project in its own
directory.

## Before starting a project

1. Copy the relevant Compose file.
2. Create a local `.env` from [`../examples/.env.example`](../examples/.env.example).
3. Replace every `<PINNED_VERSION>` and `CHANGE_ME...` value.
4. Create the host directories used by its bind mounts.
5. Read the upstream install and upgrade notes, especially for Nextcloud AIO,
   Immich, and Pi-hole.

Example:

```bash
cd /srv/app-data/<project>
docker compose config
docker compose pull
docker compose up -d
docker compose ps
docker compose logs --tail=100
```

`docker compose config` should succeed before containers are started. Do not
copy `.env` files into this repository.

## Service notes

- **Samba:** share selected directories under `/mnt/data`; validate with
  `sudo testparm` before restarting `smbd`.
- **Filebrowser:** useful for occasional browser access; keep it private.
- **Nextcloud:** use its documented AIO workflow and back up its database and
  data through that workflow.
- **Immich:** back up both the photo library and its database; the database
  contains albums and metadata.
- **Jellyfin:** keep config on the NVMe and media on `/mnt/data`.
- **Paperless:** keep the consume/export directories on application storage
  and documents on `/mnt/data`.
- **Pi-hole:** bind DNS only to the intended LAN/VPN address.
- **Beszel:** monitoring requires a separately generated agent token and key.
- **Pinchflat:** treat downloaded media as replaceable unless it is part of
  your backup set.
- **KVM/libvirt:** install separately when needed; leave RAM for Docker and
  databases.

Verify a running deployment:

```bash
docker compose ps
docker compose logs --tail=50
sudo ss -tulpn
```
