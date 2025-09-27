# 🧌 Pi Cluster - Complete Git Node Setup Guide

## What This Node Does
Turns your Pi 5 into a professional-grade, self-hosted GitHub alternative using Gitea. All your code repos live here with full web interface, user management, and enterprise features.

**Node Details:**
- **Name**: git-server  
- **Hardware**: Raspberry Pi 5
- **IP**: YOUR_STATIC_IP (you choose this)
- **OS**: Ubuntu Server 24.04.3 LTS
- **Purpose**: Git repository hosting with web interface
- **Setup Time**: 45-60 minutes
- **Status**: Production-ready when complete

## 🎯 Quick Start
If you just want it working, jump to the **TL;DR Setup** section at the bottom.

## 🔧 Why Pi 5 for Git Hosting?

**Performance Benefits:**
- Faster repository cloning and pushing (2-3x improvement over Pi 4)
- Better web interface responsiveness under load
- Support for NVMe storage via PCIe HAT (game changer for performance)
- Can handle 10-20 active developers comfortably
- PCIe bus provides dedicated storage bandwidth

**When Pi 4 Might Be Enough:**
- Solo developer or small team (2-3 people)
- Mostly text-based repositories
- Limited budget constraints
- Basic git operations only

## 🛠️ Hardware Requirements

### Essential Components
- **Raspberry Pi 5** (4GB RAM minimum, 8GB recommended for teams)
- **Power Supply**: 5V/5A (25W) USB-C - cheap chargers cause random failures
- **Storage Options**:
  - **Basic**: 64GB+ SD card (Class 10/U3)
  - **Recommended**: 64GB SD card + 256GB+ NVMe SSD via PCIe HAT
- **Network**: Gigabit Ethernet connection (WiFi works but slower)
- **Cooling**: Active cooling for sustained loads (Pi 5 runs hot)

### Storage Performance Comparison
| Storage Type | Boot Time | Git Clone Speed | Random I/O |
|--------------|-----------|-----------------|------------|
| SD Card Only | ~45s | 15-25 MB/s | Poor |
| SD + NVMe | ~30s | 80-120 MB/s | Excellent |

**Bottom Line:** NVMe is worth the extra cost for anything beyond personal use.

## 🌐 Network Planning

### IP Address Strategy
```bash
# Check your current network
ip route | grep default
# Usually shows 192.168.1.1 or 192.168.0.1

# Find used IPs to avoid conflicts
nmap -sn 192.168.1.0/24
```

**Recommended IP Ranges:**
- **Single Node**: Pick any available IP (e.g., 192.168.1.100)
- **Cluster Planning**: Use sequential IPs (192.168.1.10, .11, .12, etc.)
- **Enterprise**: Dedicate subnet (192.168.2.x for infrastructure)

### Port Planning
- **22**: SSH access (standard)
- **3000**: Gitea web interface (customizable to 80/443 if desired)
- **9418**: Git protocol (optional, for git:// URLs)

## ✅ Prerequisites Check

Before starting, verify:
```bash
# System is updated
sudo apt update && sudo apt upgrade -y

# Static IP is configured and working
ip addr show
ping YOUR_ROUTER_IP

# SSH key authentication works
ssh-copy-id YOUR_USERNAME@YOUR_PI_IP

# User has sudo privileges
sudo whoami
```

## 🔧 Gitea Installation

### Step 1: Create Git System User
```bash
# Create dedicated service user
sudo adduser --system --shell /bin/bash --gecos 'Git Version Control' --group --disabled-password --home /home/git git

# Create directory structure
sudo mkdir -p /var/lib/gitea/{custom,data,log}
sudo chown -R git:git /var/lib/gitea/
sudo chmod -R 750 /var/lib/gitea/
```

**What this does:** Creates an isolated user account that runs Gitea for security

### Step 2: Download Gitea Binary
```bash
# Check latest version at gitea.io/downloads
# For Pi 5 (ARM64):
wget -O gitea https://dl.gitea.io/gitea/1.21.3/gitea-1.21.3-linux-arm64
chmod +x gitea
sudo mv gitea /usr/local/bin/gitea

# Verify installation
gitea --version
```

**Should show version info, not "command not found"**

### Step 3: Create Systemd Service
```bash
sudo nano /etc/systemd/system/gitea.service
```

**Copy/paste this exact configuration:**
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

### Step 4: Setup Configuration Directory
```bash
sudo mkdir -p /etc/gitea
sudo chown root:git /etc/gitea
sudo chmod 770 /etc/gitea
```

**Permission explanation:** Only root can write, git group can read, others blocked

### Step 5: Start and Test Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable gitea
sudo systemctl start gitea

# Verify it's running
sudo systemctl status gitea
```

**Should show "active (running)" in green**

## 🌐 Web Configuration

### Access Setup Interface
Open browser: `http://YOUR_PI_IP:3000`

**If this doesn't load:**
- Service not running: `sudo systemctl status gitea`
- Firewall blocking: `sudo ufw status`
- Wrong IP: `ip addr show`

### Database Configuration
**Recommended: SQLite3**
- **Database Type**: SQLite3
- **Database Path**: `/var/lib/gitea/data/gitea.db`

**When to use PostgreSQL:**
- 20+ active users
- Multiple organizations
- Heavy CI/CD integration
- Enterprise features needed

### General Settings
- **Site Title**: `Your Organization Git` (whatever makes sense)
- **Repository Root**: `/var/lib/gitea/data/gitea-repositories`
- **Git LFS Root**: `/var/lib/gitea/data/lfs`
- **Run As Username**: `git`
- **Server Domain**: `YOUR_PI_IP` (or domain name if you have one)
- **SSH Port**: `22`
- **HTTP Port**: `3000`
- **Application URL**: `http://YOUR_PI_IP:3000/`

### Admin Account Setup
- **Username**: `admin` (or your preferred admin username)
- **Email**: `admin@your-domain.com`
- **Password**: Use a strong password

**Write these credentials down!**

### Verify Installation
```bash
# Check permissions were secured automatically
ls -la /etc/gitea/
# Should show: drwxr-x--- root git and -rw-r----- root git app.ini
```

## 🔑 SSH Key Configuration

### Generate SSH Key (Development Machine)
```bash
# Create new key specifically for git operations
ssh-keygen -t ed25519 -C "your-email@domain.com"
# Save to default location: ~/.ssh/id_ed25519
# Use a passphrase for security
```

### Add Key to Gitea
```bash
# Copy public key
cat ~/.ssh/id_ed25519.pub
```

**In Gitea web interface:**
1. Avatar → Settings → SSH / GPG Keys
2. Add Key → Paste public key content
3. Title: "Development Machine" (or descriptive name)
4. Add Key

### Test SSH Connection
```bash
ssh -T git@YOUR_PI_IP
# Expected: "Hi YOUR_USERNAME! You've successfully authenticated..."
```

**If this fails:** SSH key isn't properly configured in Gitea

## 🧪 Repository Testing

### Create Test Repository
**In Gitea web interface:**
1. "+" → New Repository
2. **Name**: `infrastructure-test`
3. **Description**: "Testing git server functionality"
4. **Visibility**: Private (recommended)
5. **Initialize with README**: ✅

### Test Git Operations
```bash
# Clone repository
git clone git@YOUR_PI_IP:YOUR_USERNAME/infrastructure-test.git
cd infrastructure-test

# Test workflow
echo "# Infrastructure Test" >> README.md 
echo "Git server: YOUR_PI_IP - ✅ Operational" >> README.md
git add README.md
git commit -m "Test git server functionality"
git push origin main
```

**Verify in web interface that changes appear**

## 🔒 Security Hardening

### SSH Hardening
```bash
sudo nano /etc/ssh/sshd_config
```

**Recommended settings:**
```bash
# Disable root login
PermitRootLogin no

# Key-only authentication
PasswordAuthentication no
PubkeyAuthentication yes

# Connection limits
MaxAuthTries 3
MaxStartups 3

# Restart SSH
sudo systemctl restart ssh
```

### Firewall Configuration
```bash
# Install and configure UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow required services
sudo ufw allow 22/tcp comment 'SSH'
sudo ufw allow 3000/tcp comment 'Gitea'

# Enable firewall
sudo ufw enable
sudo ufw status verbose
```

### Gitea Security Settings
**In Gitea admin panel:**
- Disable public registration (unless needed)
- Enable two-factor authentication
- Set password complexity requirements
- Configure rate limiting for API calls

## 💾 Backup Strategy

### Critical Data Locations
- **Configuration**: `/etc/gitea/app.ini`
- **All Repositories**: `/var/lib/gitea/data/gitea-repositories/`
- **Database**: `/var/lib/gitea/data/gitea.db`
- **User Data**: `/var/lib/gitea/data/`
- **Logs**: `/var/lib/gitea/log/`

### Simple Backup Script
```bash
sudo nano /usr/local/bin/backup-gitea.sh
```

**Backup script:**
```bash
#!/bin/bash
BACKUP_DIR="/backup/gitea-$(date +%Y%m%d_%H%M%S)"
mkdir -p "$BACKUP_DIR"

echo "Starting Gitea backup..."

# Stop Gitea service
sudo systemctl stop gitea

# Backup data
sudo cp -r /var/lib/gitea/data "$BACKUP_DIR/"
sudo cp /etc/gitea/app.ini "$BACKUP_DIR/"

# Start Gitea service
sudo systemctl start gitea

echo "Backup completed: $BACKUP_DIR"
```

**Make executable and schedule:**
```bash
sudo chmod +x /usr/local/bin/backup-gitea.sh

# Add to crontab for weekly backups
sudo crontab -e
# Add: 0 2 * * 0 /usr/local/bin/backup-gitea.sh
```

## ⚡ Performance Optimization

### NVMe Storage Setup (Pi 5 + PCIe HAT)
```bash
# Check if NVMe is detected
lsblk

# Format NVMe drive
sudo mkfs.ext4 /dev/nvme0n1

# Mount NVMe drive
sudo mkdir /mnt/nvme
sudo mount /dev/nvme0n1 /mnt/nvme

# Move Gitea data to NVMe
sudo systemctl stop gitea
sudo mkdir -p /mnt/nvme/gitea
sudo mv /var/lib/gitea/data /mnt/nvme/gitea/
sudo ln -s /mnt/nvme/gitea/data /var/lib/gitea/data
sudo chown -R git:git /mnt/nvme/gitea

# Add to fstab for permanent mount
echo "/dev/nvme0n1 /mnt/nvme ext4 defaults 0 0" | sudo tee -a /etc/fstab

sudo systemctl start gitea
```

### Memory Optimization
```bash
# Add swap for stability (if not present)
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### Monitor Performance
```bash
# Real-time monitoring
htop

# Storage usage
df -h

# Gitea-specific monitoring
sudo systemctl status gitea
sudo journalctl -u gitea --since "1 hour ago"
```

## 🔧 Service Management

### Daily Operations
```bash
# Check service status
sudo systemctl status gitea

# View real-time logs
sudo journalctl -u gitea -f

# Restart service (for updates/config changes)
sudo systemctl restart gitea

# Stop/start for maintenance
sudo systemctl stop gitea
sudo systemctl start gitea
```

### Log Management
```bash
# View Gitea application logs
tail -f /var/lib/gitea/log/gitea.log

# Configure log rotation
sudo nano /etc/logrotate.d/gitea
```

**Log rotation config:**
```bash
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

## 🔗 Integration Possibilities

### CI/CD Webhook Setup
```bash
# Example webhook for automated deployments
# Repository Settings → Webhooks → Add Webhook
# URL: http://your-ci-server:port/webhook
# Secret: generate secure token
# Events: Push, Pull Request
```

### External Authentication
```bash
# LDAP/Active Directory integration
# Admin Panel → Authentication Sources → Add New Source
# Type: LDAP (via BindDN)
# Configure your LDAP server details
```

### Repository Mirroring
```bash
# Mirror external repositories
# Repository Settings → Mirror Settings
# Add GitHub/GitLab repository with access token
# Configure sync schedule
```

## 🚨 Troubleshooting Guide

### Service Won't Start
```bash
# Check detailed service status
sudo systemctl status gitea -l

# View recent logs
sudo journalctl -u gitea -n 50

# Common causes and fixes:
# 1. Port 3000 in use: netstat -tlnp | grep 3000
# 2. Permission issues: sudo chown -R git:git /var/lib/gitea/
# 3. Config errors: check /etc/gitea/app.ini syntax
```

### SSH Authentication Problems
```bash
# Debug SSH connection
ssh -vT git@YOUR_PI_IP

# Check SSH key in Gitea
# Settings → SSH / GPG Keys (verify key is listed)

# Verify SSH service
sudo systemctl status ssh

# Check auth logs
sudo tail -f /var/log/auth.log
```

### Web Interface Issues
```bash
# Check if Gitea is listening
sudo ss -tlnp | grep 3000

# Verify firewall settings
sudo ufw status numbered

# Test local connection
curl http://localhost:3000

# Check for errors in logs
sudo journalctl -u gitea -f
```

### Repository Access Problems
```bash
# Check repository permissions
sudo ls -la /var/lib/gitea/data/gitea-repositories/

# Fix ownership if needed
sudo chown -R git:git /var/lib/gitea/

# Verify git user can access repositories
sudo -u git ls /var/lib/gitea/data/gitea-repositories/
```

### Performance Issues
```bash
# Monitor system resources
htop
iotop
df -h

# Check for storage bottlenecks
iostat 1

# Review Gitea configuration
# Increase cache sizes in app.ini if needed
```

## 📈 Resource Usage Expectations

### Typical Resource Consumption
- **RAM**: 200-500MB (varies with repository size and concurrent users)
- **CPU**: <5% during normal operations, spikes during git operations
- **Storage**: 50MB base + repository data
- **Network**: Minimal except during clone/push operations

### Scaling Indicators
**Consider upgrades when:**
- RAM usage consistently >80%
- Clone/push operations slow noticeably
- Web interface becomes unresponsive
- Storage space <20% free

**Scaling Options:**
- Upgrade to 8GB Pi 5
- Move to NVMe storage
- Consider dedicated server for >50 users

## 🎯 TL;DR Setup (Copy/Paste Installation)

**System preparation:**
```bash
sudo apt update && sudo apt upgrade -y
```

**Create git user and directories:**
```bash
sudo adduser --system --shell /bin/bash --gecos 'Git Version Control' --group --disabled-password --home /home/git git
sudo mkdir -p /var/lib/gitea/{custom,data,log}
sudo chown -R git:git /var/lib/gitea/
sudo chmod -R 750 /var/lib/gitea/
sudo mkdir -p /etc/gitea
sudo chown root:git /etc/gitea
sudo chmod 770 /etc/gitea
```

**Download and install Gitea:**
```bash
wget -O gitea https://dl.gitea.io/gitea/1.21.3/gitea-1.21.3-linux-arm64
chmod +x gitea && sudo mv gitea /usr/local/bin/gitea
```

**Create systemd service:**
```bash
sudo tee /etc/systemd/system/gitea.service > /dev/null <<EOF
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
EOF
```

**Start services:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable gitea
sudo systemctl start gitea
sudo systemctl status gitea
```

**Configure firewall:**
```bash
sudo ufw allow 22/tcp
sudo ufw allow 3000/tcp
sudo ufw enable
```

**Complete setup:**
1. Open `http://YOUR_PI_IP:3000`
2. Choose SQLite3 database
3. Configure paths as shown in guide
4. Create admin account
5. Add SSH keys for passwordless git access

---

## 🧌 Goblin Notes
- **Pi 5 power supply matters** - cheap chargers cause random crashes
- **NVMe makes a huge difference** - worth the extra cost for active use
- **SQLite is fine for most use cases** - don't overthink database choice
- **Backup your data regularly** - SD cards fail, repositories are precious
- **Static IP is mandatory** - dynamic IPs break git remotes
- **SSH keys are non-negotiable** - password git auth is painful
- **Monitor storage space** - full disks corrupt git repositories
- **Keep Gitea updated** - security patches matter for internet-facing services

## Next Steps
- Set up automated backups to external storage
- Configure SSL/TLS if exposing to internet
- Add monitoring for server health
- Consider additional cluster nodes (storage, CI/CD, monitoring)
- Document your git workflow and branching strategy

---
**Node Status**: Production Ready  
**Estimated Resource Usage**: 200-500MB RAM, <5% CPU  
**Maintenance Schedule**: Monthly updates, weekly backup verification  
**Access**: `http://YOUR_PI_IP:3000` (web), `git@YOUR_PI_IP:username/repo.git` (SSH)