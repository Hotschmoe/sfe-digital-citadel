# Backup Plan for Mini-PC Docker Server

This backup strategy ensures data resilience, offsite redundancy, and rapid recovery for a bare-metal Linux server (Intel i9-12900H/HK, 64 GB RAM, 2x 2 TB NVMe) running 30-50 Docker containers, including Nextcloud AIO and Plex/Jellyfin. Media/shares are on a Synology NAS.

## 1. Mirrored NVMe (RAID 1)

- **Purpose**: Protects against single NVMe failure, ensuring uptime for OS, Docker, and container data (e.g., Nextcloud DB, Plex metadata).
- **Setup**: 2x 2 TB NVMe in RAID 1 via `mdadm` for 2 TB usable.
- **Command**:
  ```bash
  sudo mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/nvme0n1 /dev/nvme1n1
  sudo mkfs.ext4 /dev/md0
  sudo mount /dev/md0 /mnt/nvme
  ```
- **Why**: Hardware redundancy minimizes downtime. 2 TB covers OS (20 GB), Docker (100 GB), and container volumes (~100s GB).
- **Maintenance**: Check array status with `cat /proc/mdstat`.

## 2. Rclone for Offsite Container Data

- **Purpose**: Backs up critical container data (e.g., Nextcloud DB, Plex configs) to the cloud for disaster recovery (e.g., fire, theft).
- **Setup**: Sync `/mnt/nvme/docker-volumes` to Backblaze B2 (or similar) weekly; daily for Nextcloud if data changes often.
- **Commands**:
  ```bash
  # Configure
  rclone config  # set up remote:backup/docker, encrypt with crypt
  
  # Weekly sync
  rclone sync /mnt/nvme/docker-volumes remote:backup/docker --progress
  
  # Daily Nextcloud
  rclone sync /mnt/nvme/docker-volumes/nextcloud remote:backup/nextcloud --progress
  
  # Cron (daily at 2 AM)
  0 2 * * * /usr/bin/rclone sync /mnt/nvme/docker-volumes/nextcloud remote:backup/nextcloud
  ```
- **Why**: Offsite ensures data survival beyond local hardware. 100s GB is manageable for cloud storage ($5-10/month on B2).
- **How**: Encrypts sensitive data; restores via `rclone copy` if needed.

## 3. GitHub for Docker-Compose

- **Purpose**: Stores docker-compose.yml for quick redeployment on new hardware, minimizing setup time.
- **Setup**: Private GitHub repo for docker-compose.yml. Local .env file stored on NVMe, backed up via rclone.
- **Commands**:
  ```bash
  # Push
  git add docker-compose.yml
  git commit -m "Update"
  git push origin main
  
  # Pull on new system
  git clone <repo-url> /opt/docker
  cd /opt/docker
  docker-compose up -d
  ```
- **Why**: Separates config from data, enabling recovery in ~1 hour with .env and volumes restored.
- **How**: .env stays secure locally; GitHub is version-controlled and accessible.

## 4. Full System Rsync to NAS

- **Purpose**: Optional full-system snapshot to Synology NAS for faster bare-metal recovery without reinstalling OS.
- **Setup**: Weekly rsync of entire NVMe (excluding volatile dirs) over 2.5GbE (~200 MB/s, ~2-3 hours for 2 TB).
- **Command**:
  ```bash
  # Full system backup
  rsync -aAX --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*"} / nas:/backups/full-system
  
  # Cron (weekly, Sunday 3 AM)
  0 3 * * 0 /usr/bin/rsync -aAX --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*"} / nas:/backups/full-system
  ```
- **Why**: Restores OS + Docker in one step if both NVMes fail. Complements rclone for local redundancy.
- **How**: NAS acts as a hot spare; restore by reversing rsync or booting from NAS if supported.

## Importance

- **Resilience**: Mirroring + NAS rsync guards against hardware failure.
- **Redundancy**: Rclone ensures offsite survival; GitHub enables config portability.
- **Recovery**: Full system or container-only restore options balance speed (1-2 hours) and granularity.

## Notes

- Test recovery annually (e.g., spin up a VM, restore from rclone + GitHub).
- Monitor NVMe health (`smartctl -a /dev/nvme0n1`).
- Adjust rclone frequency if Nextcloud data churn increases.