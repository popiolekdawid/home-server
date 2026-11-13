# Low-power Ubuntu home server

A recovery-oriented reference architecture for a small Ubuntu Server 24.04
homelab: containerized services on an NVMe system disk, two SATA SSDs in
software RAID1 for user data, tested backups, and optional KVM/libvirt virtual
machines.

This is a guide, not a copy of a production configuration. It intentionally
uses placeholders and generic examples. Adapt paths, disks, IP addresses,
domains, versions, resource limits, and service choices to your environment.
It is not a production enterprise deployment, and RAID1 is not a backup.

## Architecture

```text
                         private LAN / VPN
                                |
                         reverse proxy + TLS
                                |
                 Docker Compose application services
              /srv/app-data (NVMe) + /mnt/data (RAID1)
                                |
       Ubuntu Server 24.04 on NVMe ---- KVM/libvirt VMs
                                |
                 encrypted external/off-site backups
```

- Low-power Intel N-series class host with 16 GB RAM.
- NVMe: Ubuntu, Docker images, and application state.
- Two SATA SSDs: `mdadm` RAID1, mounted at `/mnt/data` for user data.
- Docker Compose: Nextcloud, Immich, Jellyfin, Filebrowser, and Pi-hole.
- Samba: LAN file sharing; Tailscale or another VPN for private administration.
- KVM/libvirt: optional Ubuntu VMs with deliberately bounded resources.
- A reverse proxy terminates TLS for any intentionally published web service.

## Repository map

- [Setup](docs/setup.md) — install the host, assemble storage, and restrict access.
- [Services](docs/services.md) — deploy the sanitized Compose projects.
- [Backup and recovery](docs/backup-and-recovery.md) — protect data and rebuild the host.
- [Operations](docs/operations.md) — health checks, upgrades, and power choices.
- [Compose manifests](examples/compose/) and
  [environment template](examples/.env.example) — sanitized deployment examples.

## Quick-start path

1. Install Ubuntu Server 24.04 on the NVMe and record hardware identifiers.
2. Discover the two data SSDs by model, serial, and size; never assume
   `/dev/sda` and `/dev/sdb`.
3. Assemble or create the RAID1 only after confirming the disks and backups.
4. Harden SSH, enable automatic security updates, and configure a restrictive
   firewall before deploying services.
5. Install Docker from its official repository, create `/srv/app-data`, and
   mount the RAID filesystem at `/mnt/data`.
6. Deploy only the services you need from reviewed Compose files. Keep secrets
   outside Git and use a password manager or secret-management system.
7. Configure database-consistent backups, an encrypted off-site copy, and an
   external-drive copy. Perform a restore test before trusting the setup.
8. Verify RAID health, service health, firewall rules, and backup freshness.

## Scope and safety

The examples are designed to be adapted, not blindly pasted. Review every
firewall rule and storage command. Never commit secrets, private keys,
database files, backup logs, or personal media. Restrict administrative
services to the LAN or VPN, use TLS for external access, and test restores
rather than checking only that a backup command completed.
