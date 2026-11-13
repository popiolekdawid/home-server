# Setup

This is the one-time setup path for a small Ubuntu Server 24.04 host:

```text
NVMe                         SATA SSDs
Ubuntu + Docker + /srv       mdadm RAID1 mounted at /mnt/data
```

Keep application state on the NVMe and large files on `/mnt/data`. The
separation makes an OS reinstall straightforward without mixing it with user
data.

## 1. Install the base system

Install Ubuntu Server 24.04 in UEFI mode on the NVMe. Create your own
administrator account and install OpenSSH. Then run:

```bash
sudo apt update
sudo apt full-upgrade -y
sudo apt install -y mdadm smartmontools ufw unattended-upgrades \
  curl ca-certificates git openssh-server
sudo systemctl enable --now ssh
sudo mkdir -p /srv/app-data /mnt/data
```

Use your own hostname, LAN address, and username. They are intentionally not
included in this repository.

## 2. Identify the data disks

Do this before touching RAID:

```bash
sudo lsblk -e7 -o NAME,PATH,SIZE,MODEL,SERIAL,FSTYPE,UUID,MOUNTPOINTS
sudo udevadm info --query=all --name=/dev/disk/by-id/<data-disk>
```

Identify both SSDs by model and serial number. Never assume that `/dev/sda`
and `/dev/sdb` refer to the same disks after a reboot.

For an existing array:

```bash
sudo mdadm --assemble --scan
cat /proc/mdstat
sudo mdadm --detail /dev/md0
sudo mdadm --detail --scan | sudo tee /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

Mount it using its confirmed filesystem UUID:

```text
UUID=<DATA_FILESYSTEM_UUID> /mnt/data ext4 defaults,noatime 0 2
```

```bash
sudo mount /mnt/data
findmnt /mnt/data
```

Do not use `mdadm --create`, `mkfs`, or `--zero-superblock` during recovery
unless you have explicitly confirmed that the selected disks contain no data.

## 3. Install Docker

Use Docker's official Ubuntu repository:

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg |
  sudo tee /etc/apt/keyrings/docker.asc >/dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" |
  sudo tee /etc/apt/sources.list.d/docker.list >/dev/null
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```

Create one directory per project under `/srv/app-data`. Copy only the
sanitized Compose files from [`../examples/compose/`](../examples/compose/);
create the real `.env` files locally.

## 4. Restrict access

The safe default is LAN/VPN administration and no direct internet exposure:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.1.0/24 to any port 22 proto tcp
sudo ufw enable
sudo ufw status verbose
```

Change the example subnet to yours. Add SMB, DNS, or reverse-proxy rules only
when you need them. Use Tailscale, WireGuard, or an equivalent private network
for remote administration.

## Setup check

```bash
hostnamectl
findmnt /mnt/data
docker compose version
sudo ufw status verbose
```

If any check fails, fix the base system before deploying applications.
