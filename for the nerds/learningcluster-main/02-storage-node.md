# Pi Cluster - Storage Node Setup Guide

## Node Overview
- **Node Name**: storage
- **Hardware**: Raspberry Pi 5
- **IP Address**: 192.168.1.12 (static) - adjust for your network
- **OS**: Ubuntu Server 24.04.3 LTS
- **Purpose**: Network file storage, sharing, and automated backups
- **Estimated Setup Time**: 30-45 minutes
- **Status**: Production Ready

## Why Pi 5 for Network Storage?
The storage node benefits from Pi 5's improved performance for:
- Faster network file transfers
- Better handling of concurrent NFS connections
- Support for external storage via USB 3.0
- Adequate RAM for file caching and NFS operations

## Hardware Requirements

### Minimum Specifications
- **Raspberry Pi 5** (4GB RAM minimum, 8GB recommended for heavy usage)
- **Power Supply**: 5V/5A (25W) USB-C - critical for stable operation
- **Storage**: 128GB+ SD card (Class 10 or better)
- **Network**: Gigabit Ethernet connection for optimal performance
- **Optional**: External USB 3.0 storage for expanded capacity

### Storage Planning
- **SD Card**: Houses OS and NFS configuration
- **External Storage**: Recommended for large file sharing and backups
- Consider storage growth - plan for 2-3x current data needs
- USB 3.0 external drives provide better performance than USB 2.0

## Network Considerations

### NFS Network Requirements
- **Bandwidth**: Gigabit recommended for multiple concurrent users
- **Latency**: Low-latency network for responsive file operations
- **Security**: NFS operates within trusted network boundaries
- **Firewall**: Configure appropriate rules for cluster access

### Port Requirements
- **22**: SSH administration
- **2049**: NFS server
- **111**: RPC portmapper
- **Various**: Dynamic RPC ports (configured in firewall section)

## Prerequisites
- Pi 5 with Ubuntu Server 24.04.3 LTS installed
- Static IP address configured for reliable NFS access
- SSH access configured with key authentication
- System updated: `sudo apt update && sudo apt upgrade -y`
- Non-root user with sudo privileges

## Base System Setup

### 1. Initial User Configuration
```bash
# If using fresh Ubuntu Server installation:
# Default login: ubuntu/ubuntu (will prompt for password change)

# Create administrative user
sudo adduser your-username
sudo usermod -aG sudo your-username
```

### 2. SSH Key Configuration
```bash
# Switch to your user account
su - your-username

# Create SSH directory
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add your public key
nano ~/.ssh/authorized_keys
# Paste content from: cat ~/.ssh/id_ed25519.pub (from your dev machine)
chmod 600 ~/.ssh/authorized_keys
```

### 3. Configure Static IP
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

Replace contents with:
```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 192.168.1.12/24  # Adjust for your network
      gateway4: 192.168.1.1  # Your router IP
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Apply network configuration:
```bash
sudo netplan apply
```

### 4. Set Hostname
```bash
sudo hostnamectl set-hostname storage
sudo nano /etc/hosts
```

Add line:
```
127.0.1.1 storage
```

### 5. System Updates
```bash
sudo apt update && sudo apt upgrade -y
```

## NFS Server Installation and Configuration

### 1. Install NFS Server
```bash
sudo apt install nfs-kernel-server nfs-common -y
```

### 2. Create NFS Directory Structure
```bash
# Create NFS export directories
sudo mkdir -p /srv/nfs/shared
sudo mkdir -p /srv/nfs/backups
sudo mkdir -p /srv/nfs/configs

# Set proper ownership and permissions
sudo chown -R nobody:nogroup /srv/nfs/
sudo chmod 755 /srv/nfs/shared /srv/nfs/backups /srv/nfs/configs
```

**Directory Purpose:**
- **shared**: General file sharing between cluster nodes
- **backups**: Automated backups from cluster services
- **configs**: Shared configuration files and scripts

### 3. Configure NFS Exports
```bash
sudo nano /etc/exports
```

Add export configurations:
```
# Cluster network access - adjust IP range for your network
/srv/nfs/shared    192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/backups   192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
/srv/nfs/configs   192.168.1.0/24(rw,sync,no_subtree_check,no_root_squash)
```

**Export Options Explained:**
- **rw**: Read-write access
- **sync**: Synchronous writes (safer but slower)
- **no_subtree_check**: Improves performance
- **no_root_squash**: Allows root access (use carefully)

### 4. Apply NFS Configuration
```bash
# Export the file systems
sudo exportfs -a

# Start and enable NFS services
sudo systemctl restart nfs-kernel-server
sudo systemctl enable nfs-kernel-server

# Verify service status
sudo systemctl status nfs-kernel-server
```

### 5. Verify NFS Exports
```bash
# Check active exports
sudo exportfs -v

# Test local NFS functionality
sudo showmount -e localhost
```

## Automated Backup System

### 1. Create Backup Directory Structure
```bash
sudo mkdir -p /srv/nfs/backups/{git-repos,configs,logs}
sudo chown -R nobody:nogroup /srv/nfs/backups/
```

### 2. Git Repository Backup Script
```bash
sudo nano /usr/local/bin/backup-git-repos.sh
```

Add backup script:
```bash
#!/bin/bash
# Git repository backup script for cluster storage node

BACKUP_DIR="/srv/nfs/backups/git-repos"
GIT_NODE_IP="192.168.1.10"  # Adjust for your git node IP
SOURCE_DIR="/var/lib/gitea/data/gitea-repositories"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Create timestamped backup using rsync
echo "$(date): Starting Git repository backup"
if rsync -av --delete-after "root@${GIT_NODE_IP}:${SOURCE_DIR}/" "$BACKUP_DIR/repos_$DATE/"; then
    echo "$(date): Backup completed successfully: $DATE"
    
    # Clean up old backups
    find "$BACKUP_DIR" -name "repos_*" -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;
    echo "$(date): Cleaned up backups older than $RETENTION_DAYS days"
else
    echo "$(date): Backup failed" >&2
    exit 1
fi
```

### 3. Make Script Executable and Test
```bash
sudo chmod +x /usr/local/bin/backup-git-repos.sh

# Test backup script
sudo /usr/local/bin/backup-git-repos.sh

# Verify backup creation
ls -la /srv/nfs/backups/git-repos/
```

### 4. Schedule Automated Backups
```bash
sudo crontab -e
```

Add backup schedule:
```
# Daily Git repository backup at 2:00 AM
0 2 * * * /usr/local/bin/backup-git-repos.sh >> /var/log/git-backup.log 2>&1

# Weekly backup verification at 3:00 AM on Sundays
0 3 * * 0 find /srv/nfs/backups/git-repos -name "repos_*" -type d | wc -l >> /var/log/backup-count.log
```

## Client Configuration

### Setting Up NFS Client Access
**On any cluster node or client machine:**

```bash
# Install NFS client utilities
sudo apt install nfs-common -y

# Create mount points
sudo mkdir -p /mnt/storage/{shared,backups,configs}

# Test manual mounting
sudo mount -t nfs storage-node-ip:/srv/nfs/shared /mnt/storage/shared

# Verify mount
df -h | grep nfs
ls -la /mnt/storage/shared
```

### Permanent NFS Mounts
```bash
# Add to /etc/fstab for automatic mounting
sudo nano /etc/fstab
```

Add NFS mount entries:
```
# NFS mounts for cluster storage
192.168.1.12:/srv/nfs/shared /mnt/storage/shared nfs defaults,_netdev 0 0
192.168.1.12:/srv/nfs/backups /mnt/storage/backups nfs defaults,_netdev 0 0
192.168.1.12:/srv/nfs/configs /mnt/storage/configs nfs defaults,_netdev 0 0
```

**Mount Options:**
- **defaults**: Standard mount options
- **_netdev**: Wait for network before mounting

```bash
# Test fstab entries
sudo mount -a

# Verify all mounts
mount | grep nfs
```

## Security Configuration

### Firewall Setup
```bash
# Install and configure UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH
sudo ufw allow 22/tcp

# Allow NFS traffic from cluster network (adjust IP range)
sudo ufw allow from 192.168.1.0/24 to any port nfs
sudo ufw allow from 192.168.1.0/24 to any port 2049
sudo ufw allow from 192.168.1.0/24 to any port 111

# Enable firewall
sudo ufw enable
sudo ufw status verbose
```

### NFS Security Considerations
```bash
# Restrict NFS to specific network only
# Never expose NFS to public internet
# Consider using NFSv4 with Kerberos for enhanced security

# Monitor NFS access
sudo tail -f /var/log/syslog | grep nfs
```

## Testing and Verification

### Comprehensive NFS Testing
```bash
# From client machine, test all operations:

# Mount test
sudo mount -t nfs storage-node-ip:/srv/nfs/shared /mnt/test

# Write test
echo "test from $(hostname) at $(date)" | sudo tee /mnt/test/write-test.txt

# Read test
cat /mnt/test/write-test.txt

# Permission test
sudo touch /mnt/test/permission-test
ls -la /mnt/test/

# Unmount test
sudo umount /mnt/test
```

### Service Verification
```bash
# Check NFS server status
sudo systemctl status nfs-kernel-server

# View active exports
sudo exportfs -v

# Check NFS statistics
nfsstat -s

# Monitor active connections
sudo ss -an | grep :2049

# Check mounted file systems
sudo showmount -e localhost
```

## Monitoring and Maintenance

### System Monitoring
```bash
# Disk usage monitoring
df -h
du -sh /srv/nfs/*

# Network activity
sudo iftop -i eth0
sudo nethogs

# NFS activity
nfsstat -c  # Client statistics
nfsstat -s  # Server statistics
```

### Backup Verification
```bash
# List available backups
ls -la /srv/nfs/backups/git-repos/

# Check backup sizes and verify integrity
du -sh /srv/nfs/backups/git-repos/repos_*

# Test backup restoration
sudo cp -r /srv/nfs/backups/git-repos/repos_latest/test-repo /tmp/restore-test
```

### Log Management
```bash
# Create log rotation for backup logs
sudo nano /etc/logrotate.d/cluster-backups
```

Add log rotation configuration:
```
/var/log/git-backup.log /var/log/backup-count.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 644 root root
}
```

## Performance Optimization

### Network Tuning
```bash
# Optimize network buffers for NFS performance
sudo nano /etc/sysctl.conf
```

Add network optimizations:
```
# Network buffer optimization for NFS
net.core.rmem_default = 262144
net.core.rmem_max = 16777216
net.core.wmem_default = 262144
net.core.wmem_max = 16777216

# TCP optimization
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_rmem = 4096 16384 16777216
net.ipv4.tcp_wmem = 4096 16384 16777216
```

Apply settings:
```bash
sudo sysctl -p
```

### NFS Performance Tuning
```bash
# Edit NFS configuration for better performance
sudo nano /etc/default/nfs-kernel-server
```

Add performance settings:
```
# Increase NFS thread count for better concurrency
RPCNFSDCOUNT=16

# Enable NFS over TCP (more reliable)
RPCNFSDOPTS="--tcp"
```

Restart NFS service:
```bash
sudo systemctl restart nfs-kernel-server
```

## Troubleshooting

### Common NFS Issues

#### Permission Denied Errors
```bash
# Check export permissions
sudo exportfs -v

# Verify directory ownership
ls -la /srv/nfs/

# Reload exports
sudo exportfs -ra

# Check client mount options
mount | grep nfs
```

#### Mount Hangs or Timeouts
```bash
# Check network connectivity
ping storage-node-ip

# Verify NFS service is running
sudo systemctl status nfs-kernel-server

# Check firewall rules
sudo ufw status

# Test with different mount options
sudo mount -t nfs -o tcp,hard,intr storage-node-ip:/srv/nfs/shared /mnt/test
```

#### Stale File Handle Errors
```bash
# Unmount and remount affected shares
sudo umount /mnt/storage/shared
sudo mount -t nfs storage-node-ip:/srv/nfs/shared /mnt/storage/shared

# Or remount all NFS shares
sudo umount -a -t nfs
sudo mount -a
```

### Service Management
```bash
# Restart NFS server
sudo systemctl restart nfs-kernel-server

# Reload exports without restart
sudo exportfs -ra

# Check NFS server logs
sudo journalctl -u nfs-kernel-server -f

# Monitor NFS kernel threads
ps aux | grep nfsd
```

### Log Analysis
```bash
# System logs
sudo tail -f /var/log/syslog | grep -i nfs

# NFS-specific logs
sudo journalctl -u nfs-kernel-server -n 50

# Network connectivity logs
sudo tail -f /var/log/auth.log
```

## Backup and Recovery

### Storage Node Backup
```bash
# Backup NFS configuration
sudo cp /etc/exports /backup/exports.backup
sudo cp -r /etc/default/nfs* /backup/

# Backup data directories (if local storage used)
sudo tar -czf /backup/nfs-data-$(date +%Y%m%d).tar.gz /srv/nfs/
```

### Disaster Recovery
```bash
# Restore NFS configuration
sudo cp /backup/exports.backup /etc/exports
sudo exportfs -ra
sudo systemctl restart nfs-kernel-server

# Restore data from backup
sudo tar -xzf /backup/nfs-data-date.tar.gz -C /
sudo chown -R nobody:nogroup /srv/nfs/
```

## Integration with Cluster Services

### Directory Structure Overview
```
/srv/nfs/
├── shared/              # Inter-node file sharing
│   ├── scripts/         # Cluster management scripts
│   ├── configs/         # Shared configuration files
│   └── temp/           # Temporary file exchange
├── backups/            # Service backups
│   ├── git-repos/      # Git repository backups
│   ├── configs/        # Configuration backups
│   └── logs/          # Log file backups
└── configs/           # Persistent configurations
    ├── monitoring/     # Monitoring configurations
    ├── security/      # Security policies
    └── deployment/    # Deployment scripts
```

### Integration Points
- **Git Node**: Automated repository backups
- **Security Node**: Log storage and security configuration sharing
- **Monitoring Node**: Configuration backup and log aggregation
- **Development**: Shared development environments and datasets

## Resource Usage and Scaling

### Expected Resource Consumption
- **RAM**: 500MB-1GB (depends on concurrent connections and file caching)
- **CPU**: Low usage except during backup operations
- **Network**: Scales with file transfer volume
- **Storage**: Grows with backup retention and shared files

### Scaling Considerations
- **Concurrent Users**: Pi 5 handles 5-10 concurrent NFS users well
- **Storage Growth**: Plan for 3-6 months of backup retention
- **Network Bandwidth**: Monitor for bottlenecks during peak usage
- **External Storage**: Consider USB 3.0 drives for capacity expansion

## Next Steps
- Configure client nodes to use NFS storage
- Set up monitoring for storage usage and NFS performance
- Implement additional backup strategies for critical data
- Consider high availability with multiple storage nodes
- Document storage policies and access procedures

---
**Node Status**: Production Ready  
**Estimated Resource Usage**: 500MB-1GB RAM, minimal CPU  
**Maintenance**: Weekly backup verification, monthly system updates  
**Next Recommended Node**: Security monitoring for network protection
