# Pi Cluster - Git Node Setup Guide

## Node Overview
- **Node Name**: git
- **Hardware**: Raspberry Pi 5
- **IP Address**: 192.168.1.10 (static) - adjust for your network
- **OS**: Ubuntu Server 24.04.3 LTS
- **Purpose**: Git repository hosting (Gitea)
- **Estimated Setup Time**: 45-60 minutes
- **Status**: Production Ready

## Why Pi 5 for Git Hosting?
The Git node benefits from Pi 5's improved performance for:
- Faster repository cloning and pushing
- Better web interface responsiveness
- Support for NVMe storage via PCIe HAT
- Adequate resources for multiple concurrent users

## Hardware Requirements

### Minimum Specifications
- **Raspberry Pi 5** (4GB or 8GB RAM recommended)
- **Power Supply**: 5V/5A (25W) USB-C - inadequate power causes boot failures
- **Storage**: 64GB+ SD card OR NVMe SSD via PCIe HAT (recommended)
- **Network**: Gigabit Ethernet connection
- **Cooling**: Active cooling recommended for sustained loads

### Storage Considerations
- **SD Card Only**: 64GB minimum, Class 10 or better
- **NVMe Setup** (Recommended): 64GB+ SD card for boot + 256GB+ NVMe for data
- NVMe provides better performance and reliability for Git operations

## Network Planning

### IP Address Selection
- Choose static IP outside your router's DHCP range
- Common ranges: 192.168.1.x, 192.168.0.x, 10.0.0.x
- Reserve IP in your router to prevent conflicts
- Document IP assignments for cluster expansion

### Port Requirements
- **22**: SSH access
- **3000**: Gitea web interface (customizable)
- Ensure ports don't conflict with existing services

## Prerequisites
- Pi 5 with Ubuntu Server 24.04.3 LTS installed
- Static IP configured
- SSH access configured with key authentication
- System updated: `sudo apt update && sudo apt upgrade`
- Non-root user with sudo privileges

## Gitea Installation

### 1. Create Gitea System User
```bash
sudo adduser --system --shell /bin/bash --gecos 'Git Version Control' --group --disabled-password --home /home/git git
sudo mkdir -p /var/lib/gitea/{custom,data,log}
sudo chown -R git:git /var/lib/gitea/
sudo chmod -R 750 /var/lib/gitea/
```

### 2. Download and Install Gitea Binary
```bash
# Check latest version at gitea.io/downloads
# For Pi 5 (ARM64):
wget -O gitea https://dl.gitea.io/gitea/1.21.3/gitea-1.21.3-linux-arm64.tar.gz
tar -xzf gitea-1.21.3-linux-arm64.tar.gz
chmod +x gitea
sudo mv gitea /usr/local/bin/gitea

# Verify installation
gitea --version
```

### 3. Create Systemd Service
```bash
sudo nano /etc/systemd/system/gitea.service
```

**Service Configuration:**
```ini
[Unit]
Description=Gitea (Git with a cup of tea)
After=syslog.target
After=network.target

[Service]
Type=simple
User=git
Group=git
WorkingDirectory=/var/lib/gitea/
RuntimeDirectory=gitea
ExecStart=/usr/local/bin/gitea web --config /etc/gitea/app.ini
Restart=always
Environment=USER=git HOME=/home/git GITEA_WORK_DIR=/var/lib/gitea

[Install]
WantedBy=multi-user.target
```

### 4. Setup Configuration Directory
```bash
sudo mkdir -p /etc/gitea
sudo chown root:git /etc/gitea
sudo chmod 770 /etc/gitea
```

### 5. Start and Enable Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable gitea
sudo systemctl start gitea
sudo systemctl status gitea
```

## Web Configuration

### Access Initial Setup
Navigate to: `http://your-git-node-ip:3000`

### Database Configuration
**Recommended: SQLite3**
- **Database Type**: SQLite3
- **Database Path**: `/var/lib/gitea/data/gitea.db`
- SQLite is ideal for small to medium repositories
- Consider PostgreSQL for high-traffic or multi-user environments

### General Settings
- **Site Title**: Your Git Server (e.g., "Homelab Git")
- **Repository Root Path**: `/var/lib/gitea/data/gitea-repositories`
- **Git LFS Root Path**: `/var/lib/gitea/data/lfs`
- **Run As Username**: git
- **Server Domain**: your-git-node-ip (or domain name if configured)
- **SSH Server Port**: 22
- **HTTP Port**: 3000
- **Application URL**: `http://your-git-node-ip:3000/`
- **Log Path**: `/var/lib/gitea/log`

### Admin Account Creation
- **Username**: your-admin-username
- **Email**: your-email@domain.com
- **Password**: Strong password for initial access

### Post-Installation Security
Configuration permissions are automatically secured:
```bash
# Verify permissions
ls -la /etc/gitea/
# Should show: drwxr-x--- root git and -rw-r----- root git app.ini
```

## SSH Key Configuration

### 1. Generate SSH Key (on your development machine)
```bash
ssh-keygen -t ed25519 -C "your-email@domain.com"
# Accept default location (~/.ssh/id_ed25519)
# Use a strong passphrase for security
```

### 2. Add Public Key to Gitea
```bash
# Copy public key content
cat ~/.ssh/id_ed25519.pub
```

In Gitea web interface:
1. Click your avatar → Settings
2. SSH / GPG Keys → Add Key
3. Paste public key content
4. Give it a descriptive name (e.g., "Development Machine")
5. Click "Add Key"

### 3. Test SSH Connection
```bash
ssh -T git@your-git-node-ip
# Expected output: "Hi username! You've successfully authenticated..."
```

## First Repository Test

### Create Test Repository
In Gitea web interface:
1. Click "+" → New Repository
2. **Repository Name**: `cluster-infrastructure`
3. **Description**: "Cluster configuration and documentation"
4. **Visibility**: Private (recommended for infrastructure)
5. **Initialize with README**: ✅

### Clone and Test Workflow
```bash
# Clone repository
git clone git@your-git-node-ip:username/cluster-infrastructure.git
cd cluster-infrastructure

# Test commit and push
echo "# Cluster Infrastructure" >> README.md 
echo "Git node: your-git-node-ip - Complete" >> README.md
git add README.md
git commit -m "Document git node completion"
git push origin main
```

## Security Hardening

### SSH Security
```bash
# Edit SSH configuration
sudo nano /etc/ssh/sshd_config

# Recommended settings:
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
MaxAuthTries 3

# Restart SSH service
sudo systemctl restart ssh
```

### Firewall Configuration
```bash
# Install and configure UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 3000/tcp
sudo ufw enable
```

### Gitea Security Settings
In Gitea admin panel:
- Disable user registration (if single-user)
- Enable two-factor authentication
- Set strong password policies
- Configure webhook secrets for integrations

## Backup and Recovery

### Important Paths for Backup
- **Configuration**: `/etc/gitea/app.ini`
- **Data Directory**: `/var/lib/gitea/data/`
- **Repositories**: `/var/lib/gitea/data/gitea-repositories/`
- **Database**: `/var/lib/gitea/data/gitea.db` (SQLite)
- **Logs**: `/var/lib/gitea/log/`

### Simple Backup Script
```bash
#!/bin/bash
BACKUP_DIR="/backup/gitea-$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Stop Gitea service
sudo systemctl stop gitea

# Backup data
sudo cp -r /var/lib/gitea/data "$BACKUP_DIR/"
sudo cp /etc/gitea/app.ini "$BACKUP_DIR/"

# Start Gitea service
sudo systemctl start gitea

echo "Backup completed: $BACKUP_DIR"
```

## Performance Optimization

### NVMe Storage Setup (Optional)
If using Pi 5 with NVMe HAT:
```bash
# Move Gitea data to NVMe
sudo systemctl stop gitea
sudo mkdir -p /mnt/nvme/gitea
sudo mv /var/lib/gitea/data /mnt/nvme/gitea/
sudo ln -s /mnt/nvme/gitea/data /var/lib/gitea/data
sudo chown -R git:git /mnt/nvme/gitea
sudo systemctl start gitea
```

### Resource Monitoring
```bash
# Monitor Gitea resource usage
sudo systemctl status gitea
sudo journalctl -u gitea --since "1 hour ago"

# System resource monitoring
htop
df -h
```

## Service Management

### Status and Control Commands
```bash
# Service status
sudo systemctl status gitea

# View logs
sudo journalctl -u gitea -f

# Service control
sudo systemctl restart gitea
sudo systemctl stop gitea
sudo systemctl start gitea

# Check if service enabled
sudo systemctl is-enabled gitea
```

### Log Management
```bash
# Gitea application logs
tail -f /var/lib/gitea/log/gitea.log

# System service logs
sudo journalctl -u gitea -f

# Log rotation (add to /etc/logrotate.d/gitea)
/var/lib/gitea/log/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 644 git git
}
```

## Integration with External Services

### GitHub Mirroring (Optional)
Configure repository mirroring in Gitea:
1. Repository Settings → Mirror Settings
2. Add GitHub repository URL with personal access token
3. Configure sync schedule

### Webhook Integration
Set up webhooks for CI/CD integration:
1. Repository Settings → Webhooks
2. Add webhook URL and secret
3. Configure trigger events

## Troubleshooting

### Common Issues

#### Service Won't Start
```bash
# Check service status and logs
sudo systemctl status gitea
sudo journalctl -u gitea -n 50

# Common causes:
# - Incorrect permissions on /var/lib/gitea/ or /etc/gitea/
# - Port 3000 already in use
# - Database connectivity issues
```

#### SSH Authentication Fails
```bash
# Debug SSH connection
ssh -vT git@your-git-node-ip

# Common causes:
# - SSH key not added to Gitea
# - Incorrect SSH key permissions
# - SSH service not running
# - Firewall blocking port 22
```

#### Web Interface Not Accessible
```bash
# Check if Gitea is listening
sudo ss -tlnp | grep 3000

# Check firewall
sudo ufw status

# Check service logs
sudo journalctl -u gitea -f
```

#### Repository Access Issues
```bash
# Check repository permissions
sudo ls -la /var/lib/gitea/data/gitea-repositories/

# Verify git user ownership
sudo chown -R git:git /var/lib/gitea/
```

### Log Locations
- **Gitea Application**: `/var/lib/gitea/log/gitea.log`
- **System Service**: `sudo journalctl -u gitea`
- **SSH**: `/var/log/auth.log`
- **System**: `/var/log/syslog`

### Recovery Procedures
```bash
# Reset Gitea to defaults
sudo systemctl stop gitea
sudo rm -rf /var/lib/gitea/data
sudo rm /etc/gitea/app.ini
# Reinstall and reconfigure

# Restore from backup
sudo systemctl stop gitea
sudo cp -r /backup/gitea-date/data /var/lib/gitea/
sudo cp /backup/gitea-date/app.ini /etc/gitea/
sudo chown -R git:git /var/lib/gitea/
sudo systemctl start gitea
```

## Resource Usage Expectations

### Typical Resource Consumption
- **RAM**: 200-500MB (varies with repository size and activity)
- **CPU**: Low usage except during git operations
- **Storage**: ~50MB for Gitea + repository data
- **Network**: Minimal except during clone/push operations

### Scaling Considerations
- Single Pi 5 can handle 10-20 active developers
- Repository size limited by available storage
- Consider database upgrade for >100 repositories
- Monitor performance with system tools

## Next Steps
- Set up additional cluster nodes (storage, monitoring, security)
- Configure automated backups to network storage
- Implement monitoring for Git server health
- Consider setting up local DNS for easier access
- Document your cluster architecture

## Integration Notes
This Git node serves as the foundation for cluster infrastructure management:
- Store configuration files for other cluster nodes
- Version control infrastructure-as-code
- Maintain documentation and runbooks
- Coordinate deployments across cluster

---
**Node Status**: Production Ready  
**Estimated Resource Usage**: 200-500MB RAM, <5% CPU  
**Maintenance**: Monthly updates, weekly backup verification  
**Next Recommended Node**: Network storage for backup integration
