# Pi Cluster - Security Node Setup Guide

## Node Overview
- **Node Name**: security
- **Hardware**: Raspberry Pi 4
- **IP Address**: 192.168.1.11 (static) - adjust for your network
- **OS**: Ubuntu Server 24.04.3 LTS
- **Purpose**: Network security, intrusion detection, and centralized logging
- **Estimated Setup Time**: 60-90 minutes (includes AIDE initialization)
- **Status**: Production Ready

## Why Pi 4 for Security?
The security node is well-suited for Pi 4 because:
- **Lower power consumption** for always-on monitoring
- **Adequate performance** for security tools and log processing
- **Cost effective** - saves Pi 5s for compute-intensive tasks
- **Stable platform** with proven reliability for security applications
- **Network monitoring** doesn't require cutting-edge performance

## Hardware Requirements

### Minimum Specifications
- **Raspberry Pi 4** (4GB RAM minimum, 8GB recommended)
- **Power Supply**: 5V/3A (15W) USB-C - adequate for Pi 4 operations
- **Storage**: 128GB+ SD card (Class 10 or better)
- **Network**: Gigabit Ethernet for monitoring traffic
- **Cooling**: Passive cooling sufficient for security workloads

### Storage Considerations
- **Log Storage**: Plan for growing log files from cluster nodes
- **Database Files**: AIDE integrity database grows with system changes
- **Backup Space**: Configuration backups and security archives
- Consider external storage for long-term log retention

## Network Security Role

### Security Responsibilities
- **Firewall Management**: Central policy enforcement
- **Intrusion Detection**: Failed login monitoring and prevention
- **Log Aggregation**: Centralized logging from all cluster nodes
- **Network Monitoring**: Regular scans for unauthorized devices
- **File Integrity**: System change detection and alerting

### Port Requirements
- **22**: SSH administration
- **514**: Syslog (UDP) for centralized logging
- **9100**: Node Exporter metrics (if using Prometheus)

## Prerequisites
- Pi 4 with Ubuntu Server 24.04.3 LTS installed
- Static IP address configured for reliable log collection
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

### 3. Static IP Configuration
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
        - 192.168.1.11/24  # Adjust for your network
      gateway4: 192.168.1.1  # Your router IP
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

Apply configuration:
```bash
sudo netplan apply
sudo hostnamectl set-hostname security
sudo nano /etc/hosts
```

Add hostname:
```
127.0.1.1 security
```

## Security Services Installation

### 1. Install Security Tools
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ufw fail2ban nmap htop iftop netstat-nat aide chkrootkit rkhunter postfix -y
```

**Tool Overview:**
- **UFW**: Uncomplicated Firewall for access control
- **Fail2Ban**: Intrusion prevention system
- **AIDE**: Advanced Intrusion Detection Environment
- **nmap**: Network discovery and security auditing
- **chkrootkit/rkhunter**: Rootkit detection tools

### 2. Configure UFW Firewall
```bash
# Set secure defaults
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow cluster network communication
sudo ufw allow from 192.168.1.0/24  # Adjust for your network
sudo ufw allow 22/tcp

# Enable firewall
sudo ufw enable
sudo ufw status verbose
```

### 3. Configure Fail2Ban Intrusion Prevention
```bash
sudo nano /etc/fail2ban/jail.local
```

Add protection configuration:
```ini
[DEFAULT]
# Ban duration (1 hour)
bantime = 3600
# Time window for violations (10 minutes)  
findtime = 600
# Maximum violations before ban
maxretry = 5
# Email for notifications (optional)
destemail = your-email@domain.com

[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 7200  # 2 hours for SSH violations
```

Start and enable Fail2Ban:
```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
sudo fail2ban-client status
```

### 4. Configure AIDE File Integrity Monitoring
```bash
# Initialize AIDE database (takes 10-15 minutes on Pi 4)
echo "Initializing AIDE database - this will take 10-15 minutes..."
sudo aideinit

# Move database to active location
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Verify AIDE installation
sudo aide --check
```

### 5. Install Network Monitoring Tools
```bash
sudo apt install vnstat tcpdump wireshark-common sysstat iotop nethogs -y

# Enable system monitoring services
sudo systemctl enable vnstat sysstat
sudo systemctl start vnstat sysstat
```

## Centralized Logging Setup

### 1. Configure Rsyslog Server
```bash
sudo nano /etc/rsyslog.conf
```

Enable remote log reception by uncommenting:
```
# Provides UDP syslog reception
module(load="imudp")
input(type="imudp" port="514")
```

### 2. Create Cluster Log Directory
```bash
sudo mkdir -p /var/log/cluster
sudo chown syslog:adm /var/log/cluster

# Create log separation by source
sudo mkdir -p /var/log/cluster/{git,storage,monitoring}
```

### 3. Configure Log Separation
```bash
sudo nano /etc/rsyslog.d/cluster.conf
```

Add log routing rules:
```
# Separate logs by source hostname
:hostname, isequal, "git" /var/log/cluster/git/git.log
:hostname, isequal, "storage" /var/log/cluster/storage/storage.log
:hostname, isequal, "monitoring" /var/log/cluster/monitoring/monitoring.log

# Stop processing these messages locally
& stop
```

### 4. Restart and Configure Firewall
```bash
sudo systemctl restart rsyslog
sudo ufw allow 514/udp
```

### 5. Configure Remote Logging on Client Nodes
**On each cluster node, add remote logging:**

```bash
# Example for git node - repeat for each node
ssh your-username@git-node-ip
echo "*.* @@security-node-ip:514" | sudo tee -a /etc/rsyslog.conf
sudo systemctl restart rsyslog
exit
```

## Security Monitoring Scripts

### 1. Network Security Scanning
```bash
sudo nano /usr/local/bin/cluster-scan.sh
```

Create network monitoring script:
```bash
#!/bin/bash
# Cluster network security scanner

NETWORK="192.168.1.0/24"  # Adjust for your network
LOGFILE="/var/log/cluster/network-scan.log"
EMAIL="your-email@domain.com"  # Optional email alerts

echo "$(date): Starting cluster network scan" >> "$LOGFILE"

# Perform network discovery scan
nmap_output=$(nmap -sn "$NETWORK" 2>&1)
echo "$nmap_output" >> "$LOGFILE"

# Count active devices
device_count=$(echo "$nmap_output" | grep -c "Nmap scan report")
echo "$(date): Found $device_count active devices" >> "$LOGFILE"

# Alert on unexpected device count (adjust threshold as needed)
if [ "$device_count" -gt 10 ]; then
    echo "$(date): WARNING: High device count detected ($device_count)" >> "$LOGFILE"
    # Optional: Send email alert
    # echo "High device count: $device_count" | mail -s "Network Alert" "$EMAIL"
fi

echo "$(date): Scan completed" >> "$LOGFILE"
```

Make executable and schedule:
```bash
sudo chmod +x /usr/local/bin/cluster-scan.sh

# Add to crontab
sudo crontab -e
```

Add scheduled scan (every 6 hours):
```
0 */6 * * * /usr/local/bin/cluster-scan.sh
```

### 2. Security Status Dashboard
```bash
sudo nano /usr/local/bin/security-status.sh
```

Create comprehensive status script:
```bash
#!/bin/bash
# Cluster security status dashboard

echo "=== CLUSTER SECURITY STATUS ==="
echo "Generated: $(date)"
echo "Host: $(hostname)"
echo ""

echo "=== FIREWALL STATUS ==="
sudo ufw status verbose
echo ""

echo "=== FAIL2BAN STATUS ==="
sudo fail2ban-client status
echo ""

echo "=== ACTIVE NETWORK CONNECTIONS ==="
ss -tuln | head -20
echo ""

echo "=== SYSTEM RESOURCES ==="
echo "Load Average: $(uptime | awk -F'load average:' '{print $2}')"
echo "Memory Usage:"
free -h | grep -E '^Mem|^Swap'
echo "Disk Usage:"
df -h / | tail -1
echo ""

echo "=== RECENT AUTHENTICATION ATTEMPTS ==="
echo "Last 10 SSH login attempts:"
grep "ssh" /var/log/auth.log | tail -10 | awk '{print $1, $2, $3, $9, $11}'
echo ""

echo "=== FAIL2BAN RECENT BANS ==="
sudo fail2ban-client status sshd
```

Make executable:
```bash
sudo chmod +x /usr/local/bin/security-status.sh

# Test the script
sudo /usr/local/bin/security-status.sh
```

### 3. Email Alert System (Optional)
```bash
sudo nano /usr/local/bin/cluster-alert.sh
```

Create alerting script:
```bash
#!/bin/bash
# Cluster alerting system

ALERT_EMAIL="your-email@domain.com"
SUBJECT="Cluster Security Alert: $1"
MESSAGE="$2"
HOSTNAME=$(hostname)

# Send email alert
{
    echo "Alert from: $HOSTNAME"
    echo "Time: $(date)"
    echo "Alert Type: $1"
    echo ""
    echo "Details:"
    echo "$MESSAGE"
} | mail -s "$SUBJECT" "$ALERT_EMAIL"

# Log alert locally
echo "$(date): Alert sent - $SUBJECT" >> /var/log/cluster/alerts.log
```

Make executable:
```bash
sudo chmod +x /usr/local/bin/cluster-alert.sh
```

## Node Exporter for Prometheus Monitoring

### 1. Download and Install Node Exporter
```bash
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-arm64.tar.gz
tar xvfz node_exporter-1.6.1.linux-arm64.tar.gz
sudo mv node_exporter-1.6.1.linux-arm64/node_exporter /usr/local/bin/

# Create service user
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

### 2. Create Systemd Service
```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Add service configuration:
```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

Enable and start service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

## Testing and Verification

### 1. Security Services Testing
```bash
# Test UFW firewall
sudo ufw status verbose

# Test Fail2Ban
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Test security status script
sudo /usr/local/bin/security-status.sh

# Test network scanning
sudo /usr/local/bin/cluster-scan.sh
cat /var/log/cluster/network-scan.log
```

### 2. Centralized Logging Verification
```bash
# Check if remote logs are being received
sudo tail -f /var/log/syslog | grep -E "(git|storage)"

# Check cluster-specific logs
ls -la /var/log/cluster/

# Test log separation
sudo tail -f /var/log/cluster/git/git.log
```

### 3. Node Exporter Testing
```bash
# Test local metrics endpoint
curl http://localhost:9100/metrics | head -20

# Check if port is listening
sudo ss -tlnp | grep 9100
```

## Maintenance Procedures

### Daily Monitoring Tasks
```bash
# Review security status
sudo /usr/local/bin/security-status.sh

# Check Fail2Ban activity
sudo fail2ban-client status sshd

# Monitor system resources
htop
df -h
```

### Weekly Maintenance
```bash
# Review network scan logs
cat /var/log/cluster/network-scan.log | grep "$(date +%Y-%m)"

# Update AIDE database after authorized changes
sudo aide --update
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Run security scans
sudo rkhunter --check --skip-keypress
sudo chkrootkit
```

### Monthly Tasks
```bash
# System updates
sudo apt update && sudo apt upgrade -y

# Review all centralized logs for anomalies
sudo find /var/log/cluster -name "*.log" -mtime -30 -exec grep -l "ERROR\|FAIL\|WARN" {} \;

# Backup security configurations
sudo tar -czf /backup/security-config-$(date +%Y%m%d).tar.gz /etc/fail2ban/ /etc/ufw/ /etc/aide/ /usr/local/bin/cluster-*
```

## Log Management and Rotation

### Configure Log Rotation
```bash
sudo nano /etc/logrotate.d/cluster-security
```

Add log rotation configuration:
```
/var/log/cluster/*.log /var/log/cluster/*/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 644 syslog adm
}

/var/log/cluster/network-scan.log {
    weekly
    rotate 12
    compress
    delaycompress
    missingok
    notifempty
    create 644 root root
}
```

## Performance Optimization

### System Tuning for Security Workloads
```bash
# Optimize for log processing
sudo nano /etc/sysctl.conf
```

Add optimizations:
```
# Increase inotify limits for file monitoring
fs.inotify.max_user_watches = 524288
fs.inotify.max_user_instances = 512

# Network optimizations for log collection
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
```

Apply settings:
```bash
sudo sysctl -p
```

## Troubleshooting

### Common Security Node Issues

#### UFW Blocking Legitimate Traffic
```bash
# Check current rules
sudo ufw status numbered

# Add specific rule for cluster communication
sudo ufw insert 1 allow from 192.168.1.0/24

# Review UFW logs
sudo tail -f /var/log/ufw.log
```

#### Fail2Ban False Positives
```bash
# Check banned IPs
sudo fail2ban-client status sshd

# Unban specific IP
sudo fail2ban-client set sshd unbanip 192.168.1.xxx

# Review fail2ban logs
sudo tail -f /var/log/fail2ban.log

# Adjust thresholds in /etc/fail2ban/jail.local
```

#### AIDE Database Errors
```bash
# Reinitialize AIDE database
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Check AIDE configuration
sudo aide --config-check
```

#### Centralized Logging Issues
```bash
# Check rsyslog configuration
sudo nginx -t  # Test config syntax
sudo systemctl status rsyslog

# Verify firewall allows syslog
sudo ufw status | grep 514

# Test manual log entry
logger -n security-node-ip "Test log message"
```

### Service Management
```bash
# Restart security services
sudo systemctl restart fail2ban
sudo systemctl restart rsyslog
sudo systemctl restart ufw

# Check service logs
sudo journalctl -u fail2ban -f
sudo journalctl -u rsyslog -f

# Monitor system resources
htop
iotop
nethogs
```

## Security Hardening

### SSH Security Enhancement
```bash
sudo nano /etc/ssh/sshd_config
```

Recommended SSH hardening:
```
# Disable root login
PermitRootLogin no

# Key-only authentication
PasswordAuthentication no
PubkeyAuthentication yes

# Limit login attempts
MaxAuthTries 3
MaxSessions 2

# Disable empty passwords
PermitEmptyPasswords no
```

Restart SSH:
```bash
sudo systemctl restart ssh
```

### Additional Security Measures
```bash
# Enable automatic security updates
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades

# Set up intrusion detection alerts
sudo nano /etc/fail2ban/action.d/mail-whois.conf
# Configure email notifications for bans
```

## Integration with Cluster Services

### Security Node Responsibilities
- **Git Node Protection**: Monitor repository access, backup security logs
- **Storage Node Security**: Monitor NFS access, protect shared storage
- **Monitoring Integration**: Provide security metrics to cluster monitoring
- **Backup Security**: Secure configuration backups and log archives

### Directory Structure
```
/var/log/cluster/
├── git/            # Git node logs
├── storage/        # Storage node logs  
├── monitoring/     # Monitoring node logs
├── network-scan.log # Network security scans
└── alerts.log      # Security alert history
```

## Resource Usage and Scaling

### Expected Resource Consumption
- **RAM**: 1-2GB (depends on log volume and retention)
- **CPU**: Low usage except during scans and log processing
- **Storage**: Grows with log retention and security databases
- **Network**: Moderate for log collection and scanning

### Scaling Considerations
- **Log Volume**: Monitor disk usage as cluster grows
- **Scan Frequency**: Adjust based on security requirements
- **Alert Sensitivity**: Tune thresholds to reduce false positives
- **Backup Retention**: Balance security needs with storage capacity

## Disaster Recovery

### Backup Critical Security Data
```bash
# Create security backup script
sudo nano /usr/local/bin/backup-security.sh
```

Add backup script:
```bash
#!/bin/bash
BACKUP_DIR="/backup/security-$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Backup configurations
cp -r /etc/fail2ban/ "$BACKUP_DIR/"
cp -r /etc/ufw/ "$BACKUP_DIR/"
cp -r /etc/aide/ "$BACKUP_DIR/"
cp /etc/rsyslog.conf "$BACKUP_DIR/"

# Backup AIDE database
cp /var/lib/aide/aide.db "$BACKUP_DIR/"

# Backup custom scripts
cp /usr/local/bin/cluster-* "$BACKUP_DIR/"

echo "Security backup completed: $BACKUP_DIR"
```

## Next Steps
- Configure client nodes to send logs to security node
- Set up email alerts for critical security events
- Integrate with cluster monitoring for security dashboards
- Document incident response procedures
- Plan for security audit and compliance requirements

---
**Node Status**: Production Ready  
**Estimated Resource Usage**: 1-2GB RAM, low CPU except during scans  
**Maintenance**: Daily log review, weekly database updates, monthly security scans  
**Next Recommended Node**: Monitoring system for centralized cluster oversight
