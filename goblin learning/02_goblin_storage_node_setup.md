# 🧌 Pi Cluster - Storage Node Setup Guide

## What This Node Does
Turns your Pi 5 into a network storage server that all your other cluster nodes can access. Think of it as your cluster's shared hard drive that automatically backs up important stuff.

**Node Details:**
- **Name**: storage
- **Hardware**: Raspberry Pi 5  
- **IP**: YOUR_STORAGE_IP (static)
- **OS**: Ubuntu Server 24.04.3 LTS
- **Purpose**: NFS file sharing + automated backups
- **Setup Time**: 30-45 minutes
- **Status**: Ready to configure

## 🎯 Quick Start
If you just want it working, jump to the **TL;DR Setup** section at the bottom.

## Hardware Requirements

### Essential Components
- **Raspberry Pi 5** (4GB RAM minimum, 8GB recommended for heavy usage)
- **Power Supply**: 5V/5A (25W) USB-C - cheap chargers cause weird issues
- **Storage**: 128GB+ SD card (Class 10/U3)
- **Network**: Gigabit Ethernet connection preferred over WiFi
- **Optional**: External USB 3.0 storage for expanded capacity

### Storage Planning Strategy
- **SD Card**: Houses OS and NFS configuration (~10GB used)
- **External Storage**: Recommended for large file sharing and backups
- **Growth Planning**: Plan for 2-3x your current data needs
- **Performance**: USB 3.0 external drives beat USB 2.0 significantly

**Storage Performance Reality:**
- SD Card only: 20-40 MB/s sustained transfer
- SD + USB 3.0: 80-120 MB/s sustained transfer
- Multiple concurrent users: External storage becomes mandatory

## Prerequisites Check
Before starting, make sure you have:
- Pi 5 with Ubuntu Server installed
- Static IP configured (choose an available IP in your network)
- SSH access working (`ssh YOUR_USERNAME@YOUR_STORAGE_IP`)
- System updated (`sudo apt update && sudo apt upgrade`)
- User with sudo privileges

**Test these work before continuing!**

## 🔧 Base System Setup

### Step 1: Create User Account
```bash
# If using fresh Ubuntu Server (default login: ubuntu/ubuntu)
sudo adduser your-username
sudo usermod -aG sudo your-username
```

### Step 2: SSH Key Setup (So No Passwords)
```bash
# Switch to your user account
su - your-username

# Create SSH directory
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add your public key from dev machine
nano ~/.ssh/authorized_keys
# Copy/paste from: cat ~/.ssh/id_ed25519.pub on your dev machine
chmod 600 ~/.ssh/authorized_keys
```

**Test it works:**
```bash
# From dev machine
ssh your-username@YOUR_STORAGE_IP
# Should connect without password
```

### Step 3: Set Static IP Address
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

**Replace everything with:**
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - YOUR_STORAGE_IP/24  # e.g., 192.168.1.20/24
      gateway4: YOUR_ROUTER_IP  # e.g., 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

**Apply the changes:**
```bash
sudo netplan apply
```

**Test it worked:**
```bash
ip addr show eth0
# Should show YOUR_STORAGE_IP/24
```

### Step 4: Set Hostname
```bash
sudo hostnamectl set-hostname storage
sudo nano /etc/hosts
```

**Add this line:**
```
127.0.1.1 storage
```

### Step 5: Update Everything
```bash
sudo apt update && sudo apt upgrade -y
```

**This takes forever. Go make coffee.**

## 📁 NFS Server Setup (The File Sharing Part)

### Install NFS Server
```bash
sudo apt install nfs-kernel-server nfs-common -y
```

### Create Directory Structure
```bash
# Create the directories that other nodes will access
sudo mkdir -p /srv/nfs/shared    # General file sharing
sudo mkdir -p /srv/nfs/backups   # Automated backups
sudo mkdir -p /srv/nfs/configs   # Shared config files

# Set proper ownership (nobody:nogroup is correct for NFS)
sudo chown -R nobody:nogroup /srv/nfs/
sudo chmod 755 /srv/nfs/shared /srv/nfs/backups /srv/nfs/configs
```

**What this does:** Creates shared folders that other cluster nodes can mount

**Directory Purposes:**
- **shared**: General file exchange between nodes
- **backups**: Automated backups from cluster services  
- **configs**: Shared configuration files and scripts

### Configure NFS Exports
```bash
sudo nano /etc/exports
```

**Add these lines (adjust IP range for your network):**
```
/srv/nfs/shared    YOUR_NETWORK_RANGE(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/backups   YOUR_NETWORK_RANGE(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/configs   YOUR_NETWORK_RANGE(rw,sync,no_subtree_check,no_root_squash)
```

**Example for 192.168.1.x network:**
```
/srv/nfs/shared    192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/backups   192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/configs   192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
```

**What this means:** Only your local network can access these shares with full permissions

### Start NFS Service
```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
sudo systemctl enable nfs-kernel-server

# Check it's running
sudo systemctl status nfs-kernel-server
```

**Should show "active (running)" in green.**

### Verify NFS Works
```bash
# Check what's being exported
sudo exportfs -v

# Test local NFS functionality
sudo showmount -e localhost
# Should list your three shares
```

## 🔄 Automated Backup System

### Create Backup Directory Structure
```bash
sudo mkdir -p /srv/nfs/backups/{git-repos,configs,logs}
sudo chown -R nobody:nogroup /srv/nfs/backups/
```

### Create Git Backup Script (Example)
```bash
sudo nano /usr/local/bin/backup-git-repos.sh
```

**Copy/paste this script (adjust IPs for your setup):**
```bash
#!/bin/bash
# Git repository backup script for cluster storage node

BACKUP_DIR="/srv/nfs/backups/git-repos"
GIT_NODE_IP="YOUR_GIT_NODE_IP"  # e.g., 192.168.1.10
SOURCE_DIR="/var/lib/gitea/data/gitea-repositories"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Create timestamped backup using rsync
echo "$(date): Starting Git repository backup"
if rsync -av --delete-after "your-username@${GIT_NODE_IP}:${SOURCE_DIR}/" "$BACKUP_DIR/repos_$DATE/"; then
    echo "$(date): Backup completed successfully: $DATE"
    
    # Clean up old backups
    find "$BACKUP_DIR" -name "repos_*" -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;
    echo "$(date): Cleaned up backups older than $RETENTION_DAYS days"
else
    echo "$(date): Backup failed" >&2
    exit 1
fi
```

### Make Script Executable
```bash
sudo chmod +x /usr/local/bin/backup-git-repos.sh
```

### Test the Backup Script
```bash
sudo /usr/local/bin/backup-git-repos.sh
ls -la /srv/nfs/backups/git-repos/
```

**Should create a timestamped backup directory.**

### Schedule Daily Backups
```bash
sudo crontab -e
```

**Add this line for daily 2 AM backups:**
```
0 2 * * * /usr/local/bin/backup-git-repos.sh >> /var/log/git-backup.log 2>&1
```

## 🔗 Client Setup (How Other Nodes Access Storage)

### On Any Other Cluster Node
```bash
# Install NFS client tools
sudo apt install nfs-common -y

# Create mount points
sudo mkdir -p /mnt/storage/{shared,backups,configs}

# Test mounting manually
sudo mount -t nfs YOUR_STORAGE_IP:/srv/nfs/shared /mnt/storage/shared

# Test it works
echo "test from $(hostname)" | sudo tee /mnt/storage/shared/test.txt
ls -la /mnt/storage/shared/

# Unmount for now
sudo umount /mnt/storage/shared
```

### Make Mounts Permanent
```bash
# Add to /etc/fstab for automatic mounting at boot
echo "YOUR_STORAGE_IP:/srv/nfs/shared /mnt/storage/shared nfs defaults,_netdev 0 0" | sudo tee -a /etc/fstab
echo "YOUR_STORAGE_IP:/srv/nfs/backups /mnt/storage/backups nfs defaults,_netdev 0 0" | sudo tee -a /etc/fstab
echo "YOUR_STORAGE_IP:/srv/nfs/configs /mnt/storage/configs nfs defaults,_netdev 0 0" | sudo tee -a /etc/fstab

# Mount everything
sudo mount -a
```

**Mount Options Explained:**
- **defaults**: Standard mount options
- **_netdev**: Wait for network before mounting (prevents boot hangs)

## 🔥 Firewall Configuration

```bash
# Allow NFS traffic from local network only
sudo ufw allow from YOUR_NETWORK_RANGE to any port nfs
sudo ufw allow from YOUR_NETWORK_RANGE to any port 2049
sudo ufw allow from YOUR_NETWORK_RANGE to any port 111

# Allow SSH
sudo ufw allow 22

# Enable firewall
sudo ufw enable
```

**Example for 192.168.1.x network:**
```bash
sudo ufw allow from 192.168.1.0/24 to any port nfs
sudo ufw allow from 192.168.1.0/24 to any port 2049
sudo ufw allow from 192.168.1.0/24 to any port 111
sudo ufw allow 22
sudo ufw enable
```

## 🧪 Testing and Verification

### Test NFS From Another Node
```bash
# From any client machine
sudo mkdir -p /mnt/test
sudo mount -t nfs YOUR_STORAGE_IP:/srv/nfs/shared /mnt/test
echo "NFS test from $(hostname)" | sudo tee /mnt/test/nfs-test.txt
ls -la /mnt/test/
sudo umount /mnt/test
```

**If this fails, NFS isn't working properly.**

### Check NFS Server Status
```bash
# Verify NFS server is running
sudo systemctl status nfs-kernel-server

# View active exports
sudo exportfs -v

# Check what's available to mount
sudo showmount -e localhost

# Monitor NFS activity
nfsstat -s
```

## 📊 Monitoring and Maintenance

### Check Storage Usage
```bash
# Overall disk usage
df -h

# NFS directory usage
du -sh /srv/nfs/*

# Backup directory sizes
du -sh /srv/nfs/backups/git-repos/repos_*
```

### Monitor NFS Activity
```bash
# View NFS statistics
nfsstat -s

# Check active connections
sudo ss -an | grep :2049

# List available backups
ls -la /srv/nfs/backups/git-repos/
```

### Performance Monitoring
```bash
# Network activity
sudo iftop -i eth0

# Monitor concurrent connections
sudo netstat -an | grep :2049 | wc -l

# Check system load during file transfers
htop
```

## 🚨 When Things Go Wrong

### NFS Mount Fails
**"Permission denied":**
- Check `/etc/exports` syntax
- Reload exports: `sudo exportfs -ra`
- Verify client IP is in allowed range

**"Connection timed out":**
- Check firewall rules: `sudo ufw status`
- Verify NFS service: `sudo systemctl status nfs-kernel-server`
- Test network connectivity: `ping YOUR_STORAGE_IP`

**"Mount hangs forever":**
- Kill mount process: `sudo umount -f /mount/point`
- Check network connectivity
- Restart NFS server: `sudo systemctl restart nfs-kernel-server`

### Service Issues
**NFS server won't start:**
```bash
# Check logs
sudo journalctl -u nfs-kernel-server --no-pager

# Common fix - reload exports
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server
```

**Backup script fails:**
```bash
# Check if source directory exists
ssh your-username@YOUR_GIT_NODE_IP "ls -la /var/lib/gitea/data/"

# Check permissions
ls -la /srv/nfs/backups/

# Run manually to see errors
sudo /usr/local/bin/backup-git-repos.sh
```

### Performance Issues
**Slow file transfers:**
- Check network connection: `iperf3` between nodes
- Monitor CPU/memory: `htop`
- Consider network buffer tuning (advanced)

**High CPU usage:**
- Check for runaway backup processes: `ps aux | grep rsync`
- Monitor NFS activity: `nfsstat -s`

### "Stale File Handle" Errors
```bash
# Unmount and remount affected shares
sudo umount /mnt/storage/shared
sudo mount -t nfs YOUR_STORAGE_IP:/srv/nfs/shared /mnt/storage/shared

# Or remount all NFS shares
sudo umount -a -t nfs
sudo mount -a
```

## ⚡ Performance Optimization (Optional)

### Network Buffer Tuning
```bash
# Add to /etc/sysctl.conf for better network performance
echo 'net.core.rmem_default = 262144' | sudo tee -a /etc/sysctl.conf
echo 'net.core.rmem_max = 16777216' | sudo tee -a /etc/sysctl.conf
echo 'net.core.wmem_default = 262144' | sudo tee -a /etc/sysctl.conf
echo 'net.core.wmem_max = 16777216' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

**Only do this if you're having performance problems.**

### NFS Performance Tuning
```bash
# Edit NFS configuration for better performance
sudo nano /etc/default/nfs-kernel-server
```

**Add these lines:**
```
# Increase NFS thread count for better concurrency
RPCNFSDCOUNT=16

# Enable NFS over TCP (more reliable)
RPCNFSDOPTS="--tcp"
```

**Restart NFS:**
```bash
sudo systemctl restart nfs-kernel-server
```

## 🎯 TL;DR Setup (Just Make It Work)

**Copy/paste these commands in order:**

```bash
# 1. Create user and SSH setup
sudo adduser your-username && sudo usermod -aG sudo your-username
su - your-username
mkdir -p ~/.ssh && chmod 700 ~/.ssh
# Add your SSH public key to ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

**Set static IP:**
```bash
sudo tee /etc/netplan/50-cloud-init.yaml > /dev/null <<EOF
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - YOUR_STORAGE_IP/24
      gateway4: YOUR_ROUTER_IP
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
EOF
sudo netplan apply
```

**Set hostname and update:**
```bash
sudo hostnamectl set-hostname storage
echo "127.0.1.1 storage" | sudo tee -a /etc/hosts
sudo apt update && sudo apt upgrade -y
```

**Install and configure NFS:**
```bash
sudo apt install nfs-kernel-server nfs-common -y
sudo mkdir -p /srv/nfs/{shared,backups,configs}
sudo chown -R nobody:nogroup /srv/nfs/
sudo chmod 755 /srv/nfs/shared /srv/nfs/backups /srv/nfs/configs

sudo tee /etc/exports > /dev/null <<EOF
/srv/nfs/shared    YOUR_NETWORK_RANGE(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/backups   YOUR_NETWORK_RANGE(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/configs   YOUR_NETWORK_RANGE(rw,sync,no_subtree_check,no_root_squash)
EOF

sudo exportfs -a
sudo systemctl restart nfs-kernel-server
sudo systemctl enable nfs-kernel-server
```

**Configure firewall:**
```bash
sudo ufw allow from YOUR_NETWORK_RANGE to any port nfs
sudo ufw allow from YOUR_NETWORK_RANGE to any port 2049
sudo ufw allow from YOUR_NETWORK_RANGE to any port 111
sudo ufw allow 22
sudo ufw enable
```

**Test it works:**
```bash
sudo systemctl status nfs-kernel-server
sudo showmount -e localhost
```

---

## 🧌 Goblin Notes
- **Nobody:nogroup ownership is correct for NFS** - don't try to "fix" it
- **25W power supply is mandatory for Pi 5** - cheap chargers cause weird issues
- **NFS v4 is automatic in modern Ubuntu** - don't force v3 unless you have problems  
- **Backup script needs source directory to exist** - set up other nodes first
- **Firewall rules matter** - NFS won't work if blocked
- **Mount hangs usually mean firewall/network issues** - not NFS config problems
- **Test manually before making permanent mounts** - saves debugging pain later
- **YOUR_NETWORK_RANGE example: 192.168.1.0/24** - adjust for your actual network
- **External USB 3.0 storage makes a huge difference** - worth the investment for active use

## Network Planning Examples
- **Home network 192.168.1.x**: Use `192.168.1.0/24` in exports
- **Home network 192.168.0.x**: Use `192.168.0.0/24` in exports  
- **Business network 10.0.x.x**: Use `10.0.0.0/16` in exports
- **Check your network**: `ip route | grep default` shows your router IP

## Integration Notes
- **Git Node**: Daily automated repository backups
- **Other Nodes**: Shared config and data storage
- **Development**: Can mount for backup verification
- **Future Nodes**: Ready to share datasets, models, etc.

---
**Node Status**: Ready for Production  
**Setup Time**: 30-45 minutes  
**Services**: NFS Server, Automated Backups  
**Access**: `sudo mount -t nfs YOUR_STORAGE_IP:/srv/nfs/shared /mnt/storage`