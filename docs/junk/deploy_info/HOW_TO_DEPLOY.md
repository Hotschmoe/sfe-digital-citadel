# Family Infrastructure Deployment Guide

This guide provides comprehensive instructions for deploying and maintaining the family infrastructure services.

## Table of Contents
1. [Repository Structure](#repository-structure)
2. [Prerequisites](#prerequisites)
3. [Initial Setup](#initial-setup)
4. [Deployment](#deployment)
5. [Backup Strategy](#backup-strategy)
6. [Maintenance](#maintenance)
7. [Recovery Procedures](#recovery-procedures)
8. [Security Considerations](#security-considerations)

## Repository Structure

```plaintext
infrastructure/
├── deploy/
│   ├── docker-compose.yml
│   ├── docker-compose.critical.yml    # Critical services
│   ├── docker-compose.additional.yml  # Additional services
│   ├── .env.example
│   ├── SERVICES.md                    # Service documentation
│   └── HOW_TO_DEPLOY.md              # This file
├── config/
│   ├── traefik/
│   │   ├── config.yml
│   │   └── README.md
│   ├── nextcloud/
│   │   ├── config.php.example
│   │   └── README.md
│   └── README.md
├── scripts/
│   ├── backup/
│   │   ├── backup-configs.sh
│   │   └── backup-volumes.sh
│   ├── setup/
│   │   ├── initial-setup.sh
│   │   └── generate-secrets.sh
│   └── maintenance/
│       ├── update-services.sh
│       └── health-check.sh
└── docs/
    ├── SETUP.md
    ├── BACKUP.md
    ├── RECOVERY.md
    └── MAINTENANCE.md
```

## Prerequisites

### Hardware Requirements
- CPU: 32 cores recommended (16 cores minimum)
- RAM: 128GB recommended (64GB minimum)
- Storage:
  - System: 256GB SSD/NVMe
  - Data: Multiple TB NAS/Storage for media and documents
  - Backup: Equal or larger than data storage

### Software Requirements
```bash
# Required software
docker
docker-compose
git
rclone (for remote backups)
fail2ban
ufw (or other firewall)
```

### Network Requirements
- Static IP or DDNS
- Open ports:
  - 80/443 (HTTP/HTTPS)
  - 51820 (WireGuard VPN)
  - 25565 (Minecraft)
  - Additional ports as needed

## Initial Setup

### 1. System Preparation
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y \
    docker.io \
    docker-compose \
    git \
    rclone \
    fail2ban \
    ufw

# Add user to docker group
sudo usermod -aG docker $USER

# Configure firewall
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 51820/udp
sudo ufw enable
```

### 2. Clone Repository
```bash
# Clone the repository
git clone https://github.com/your/infrastructure.git
cd infrastructure

# Create production directories
sudo mkdir -p /opt/infrastructure/{backups,secrets,data}
```

### 3. Environment Setup
```bash
# Copy example environment file
cp deploy/.env.example deploy/.env

# Generate secrets
./scripts/setup/generate-secrets.sh

# Configure environment variables
nano deploy/.env
```

### 4. Storage Setup
```bash
# Mount NAS storage
echo "//nas.local/data /opt/infrastructure/data/nas cifs credentials=/etc/nas-credentials,_netdev 0 0" | sudo tee -a /etc/fstab

# Create required directories
mkdir -p /opt/infrastructure/data/{media,documents,photos,backups}
```

## Deployment

### 1. Critical Services First
```bash
# Deploy critical infrastructure
cd deploy
docker-compose -f docker-compose.critical.yml up -d

# Verify critical services
docker-compose -f docker-compose.critical.yml ps
```

### 2. Additional Services
```bash
# Deploy additional services
docker-compose -f docker-compose.additional.yml up -d

# Verify all services
docker-compose ps
```

### 3. Post-Deployment Verification
```bash
# Check service health
./scripts/maintenance/health-check.sh

# Verify backups
./scripts/backup/test-backup.sh
```

## Backup Strategy

### 1. Local Backups
```bash
# Directory structure
/opt/infrastructure/backups/
├── configs/          # Service configurations
├── databases/        # Database dumps
├── volumes/         # Docker volume backups
└── secrets/         # Encryption keys and credentials
```

### 2. Backup Schedule
```bash
# Add to crontab
0 */6 * * * /opt/infrastructure/scripts/backup/backup-databases.sh
0 0 * * * /opt/infrastructure/scripts/backup/backup-volumes.sh
0 */12 * * * /opt/infrastructure/scripts/backup/remote-sync.sh
```

### 3. Remote Backups
```bash
# Configure rclone
rclone config

# Test remote backup
rclone sync /opt/infrastructure/backups remote:infrastructure-backups --dry-run
```

## Maintenance

### 1. Regular Updates
```bash
# Update services (weekly)
0 3 * * 0 /opt/infrastructure/scripts/maintenance/update-services.sh

# Update system (monthly)
0 4 1 * * /opt/infrastructure/scripts/maintenance/update-system.sh
```

### 2. Health Monitoring
```bash
# Monitor service health
*/15 * * * * /opt/infrastructure/scripts/maintenance/health-check.sh

# Monitor disk space
0 * * * * /opt/infrastructure/scripts/maintenance/disk-check.sh
```

### 3. Log Management
```bash
# Rotate logs
/etc/logrotate.d/infrastructure
{
    rotate 7
    daily
    compress
    missingok
    notifempty
}
```

## Recovery Procedures

### 1. Service Recovery
```bash
# Restart failed service
docker-compose -f docker-compose.yml up -d <service_name>

# Check service logs
docker-compose logs -f <service_name>
```

### 2. Full System Recovery
```bash
# Clone repository
git clone https://github.com/your/infrastructure.git /opt/infrastructure

# Restore environment
cp /path/to/backup/.env /opt/infrastructure/deploy/.env
cp -r /path/to/backup/secrets/* /opt/infrastructure/secrets/

# Restore volumes
./scripts/recovery/restore-volumes.sh

# Start services
cd /opt/infrastructure/deploy
docker-compose up -d
```

## Security Considerations

### 1. Access Control
- Use strong passwords
- Enable 2FA where possible
- Use SSH keys instead of passwords
- Implement proper firewall rules

### 2. Data Protection
- Encrypt sensitive data at rest
- Use SSL/TLS for all services
- Regular security updates
- Monitor access logs

### 3. Network Security
```bash
# Basic hardening
# /etc/sysctl.conf
net.ipv4.conf.all.rp_filter=1
net.ipv4.conf.all.accept_redirects=0
net.ipv4.icmp_echo_ignore_broadcasts=1
```

### 4. Monitoring
- Set up alerts for:
  - Failed login attempts
  - Disk space warnings
  - Service downtime
  - Backup failures

## Additional Notes

### Resource Management
- Monitor resource usage
- Set appropriate limits in docker-compose
- Plan for scaling
- Regular performance optimization

### Documentation
- Keep configuration changes documented
- Update this guide as needed
- Maintain service-specific documentation
- Document all custom modifications

### Troubleshooting
- Check logs first: `docker-compose logs -f`
- Verify configurations
- Check resource usage
- Validate network connectivity
- Review recent changes

### Future Considerations
- Plan for hardware upgrades
- Consider high availability
- Evaluate new services
- Regular security audits

## Contact & Support
- Create issues in GitHub repository
- Document problems and solutions
- Share improvements with the community

Remember to regularly test backups and recovery procedures to ensure they work when needed. 