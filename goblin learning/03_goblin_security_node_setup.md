# 🧌 Pi Cluster - Security Node Setup Guide

## What This Node Does
Turns your Pi 4 into the cluster's security guard and log collector. It watches for intruders, blocks bad actors, and keeps logs from all your other nodes in one place.

**Node Details:**
- **Name**: security
- **Hardware**: Raspberry Pi 4  
- **IP**: YOUR_SECURITY_IP (static)
- **OS**: Ubuntu Server 24.04.3 LTS
- **Purpose**: Firewall, intrusion detection, centralized logging
- **Setup Time**: 60-90 minutes (includes database initialization)
- **Status**: Ready to configure

## 🎯 Quick Start
If you just want it working, jump to the **TL;DR Setup** section at the bottom.

## Why Pi 4 for Security?

**Pi 4 Advantages for Security Work:**
- **Lower power consumption** for always-on monitoring
- **Adequate performance** for security tools and log processing
- **Cost effective** - saves Pi 5s for compute-intensive tasks
- **Stable platform** with proven reliability for security applications
- **Network monitoring** doesn't require cutting-edge performance

**When Pi 5 Might Be Worth It:**
- Very large clusters (20+ nodes)
- Heavy log processing requirements
- Running multiple security tools simultaneously
- Real-time threat analysis needs

## Hardware Requirements

### Essential Components
- **Raspberry Pi 4** (4GB RAM minimum, 8GB recommended)
- **Power Supply**: 5V/3A (15W) USB-C - adequate for Pi 4 operations
- **Storage**: 128GB+ SD card (Class 10/U3)
- **Network**: Gigabit Ethernet for monitoring traffic
- **Cooling**: Passive cooling sufficient for security workloads

### Storage Planning
- **Log Storage**: Plan for growing log files from cluster nodes
- **Database Files**: AIDE integrity database grows with system changes
- **Backup Space**: Configuration backups and security archives
- Consider external storage for long-term log retention (months/years)

## Prerequisites Check
Before starting, make sure you have:
- Pi 4 with Ubuntu Server installed
- Static IP configured (choose an available IP in your network)
- SSH access working (`ssh YOUR_USERNAME@YOUR_SECURITY_IP`)
- System updated (`sudo apt update && sudo apt upgrade`)
- User with sudo privileges

**Test these work before continuing!**

## 🔧 Base System Setup

### Step 1: Create User Account
```bash
# If using fresh Ubuntu Server (default: ubuntu/ubuntu)
sudo adduser your-username
sudo usermod -aG sudo your-username
```

### Step 2: SSH Key Setup
```bash
su - your-username
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Copy/paste your public key from dev machine
chmod 600 ~/.ssh/authorized_keys
```

### Step 3: Set Static IP
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
        - YOUR_SECURITY_IP/24  # e.g., 192.168.1.20/24
      gateway4: YOUR_ROUTER_IP  # e.g., 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

**Apply changes:**
```bash
sudo netplan apply
sudo hostnamectl set-hostname security
echo "127.0.1.1 security" | sudo tee -a /etc/hosts
```

## 🛡️ Security Services Installation

### Install Security Tools
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ufw fail2ban nmap htop iftop aide chkrootkit rkhunter vnstat tcpdump sysstat -y
```

**What this installs:**
- `ufw` - Simple firewall
- `fail2ban` - Blocks brute force attacks
- `aide` - File integrity monitoring
- `nmap` - Network scanning
- Various monitoring tools

## 🔥 Configure UFW Firewall

### Basic Firewall Setup
```bash
# Set default policies (deny incoming, allow outgoing)
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow cluster network communication (adjust for your network)
sudo ufw allow from YOUR_NETWORK_RANGE  # e.g., 192.168.1.0/24

# Allow SSH (critical - don't lock yourself out!)
sudo ufw allow 22

# Enable firewall
sudo ufw enable
```

**Check it works:**
```bash
sudo ufw status verbose
# Should show rules allowing cluster network and SSH
```

**IMPORTANT:** If you lose SSH access, you'll need physical access to fix it!

## 🚫 Configure Fail2Ban (Block Bad Actors)

### Create Fail2Ban Configuration
```bash
sudo nano /etc/fail2ban/jail.local
```

**Add this configuration:**
```ini
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5
destemail = your-email@example.com

[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 7200
```

**What this does:**
- Bans IP addresses for 1-2 hours after failed login attempts
- Monitors SSH login attempts
- Aggressive settings for SSH (3 strikes and you're out)

### Start Fail2Ban
```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban

# Check it's working
sudo fail2ban-client status
sudo fail2ban-client status sshd
```

## 📊 File Integrity Monitoring (AIDE)

### Initialize AIDE Database
```bash
# This takes 10-15 minutes - go make coffee
echo "Initializing AIDE database - this will take 10-15 minutes..."
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

**What this does:** Creates a baseline of all system files to detect unauthorized changes

### Enable System Monitoring
```bash
sudo systemctl enable vnstat sysstat
sudo systemctl start vnstat sysstat
```

## 📝 Centralized Logging Setup

### Configure Rsyslog Server
```bash
sudo nano /etc/rsyslog.conf
```

**Find and uncomment these lines:**
```
module(load="imudp")
input(type="imudp" port="514")
```

**Create log directory:**
```bash
sudo mkdir -p /var/log/cluster
sudo chown syslog:adm /var/log/cluster
sudo systemctl restart rsyslog

# Allow log traffic through firewall
sudo ufw allow 514/udp
```

### Configure Other Nodes to Send Logs Here
**On each other cluster node:**
```bash
ssh your-username@OTHER_NODE_IP
echo "*.* @@YOUR_SECURITY_IP:514" | sudo tee -a /etc/rsyslog.conf
sudo systemctl restart rsyslog
```

**Test centralized logging works:**
```bash
# Watch for logs from other nodes
sudo tail -f /var/log/syslog | grep -E "(git|storage|other-node-names)"
```

## 🔍 Monitoring Scripts

### Network Scan Script
```bash
sudo nano /usr/local/bin/cluster-scan.sh
```

**Add this script:**
```bash
#!/bin/bash
NETWORK="YOUR_NETWORK_RANGE"  # e.g., 192.168.1.0/24
LOGFILE="/var/log/cluster/network-scan.log"

echo "$(date): Starting cluster network scan" >> $LOGFILE
nmap -sn $NETWORK >> $LOGFILE 2>&1
echo "$(date): Scan completed" >> $LOGFILE
echo "----------------------------------------" >> $LOGFILE
```

**Make executable and schedule:**
```bash
sudo chmod +x /usr/local/bin/cluster-scan.sh

# Schedule to run every 6 hours
sudo crontab -e
# Add this line:
# 0 */6 * * * /usr/local/bin/cluster-scan.sh
```

### Security Status Script
```bash
sudo nano /usr/local/bin/security-status.sh
```

**Add this script:**
```bash
#!/bin/bash
echo "=== CLUSTER SECURITY STATUS ==="
echo "Date: $(date)"
echo ""

echo "=== UFW Status ==="
sudo ufw status

echo ""
echo "=== Fail2Ban Status ==="
sudo fail2ban-client status

echo ""
echo "=== Active Network Connections ==="
ss -tuln

echo ""
echo "=== System Load ==="
uptime

echo ""
echo "=== Disk Usage ==="
df -h

echo ""
echo "=== Memory Usage ==="
free -h
```

**Make executable:**
```bash
sudo chmod +x /usr/local/bin/security-status.sh
```

## 📈 Node Exporter (For Monitoring)

### Install Node Exporter
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-arm64.tar.gz
tar xvfz node_exporter-1.6.1.linux-arm64.tar.gz
sudo mv node_exporter-1.6.1.linux-arm64/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
rm -rf node_exporter-1.6.1.linux-arm64*
```

### Create Systemd Service
```bash
sudo nano /etc/systemd/system/node_exporter.service
```

**Add this configuration:**
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

**Start the service:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# Allow monitoring traffic
sudo ufw allow 9100
```

**Test it works:**
```bash
curl http://localhost:9100/metrics | head
# Should show lots of system metrics
```

## 🧪 Testing and Verification

### Test Security Services
```bash
# Check firewall status
sudo ufw status verbose

# Check Fail2Ban status
sudo fail2ban-client status sshd

# Run security status script
sudo /usr/local/bin/security-status.sh

# Test network scan
sudo /usr/local/bin/cluster-scan.sh
cat /var/log/cluster/network-scan.log
```

### Verify Centralized Logging
```bash
# Check if remote logs are being received
sudo tail -f /var/log/syslog | grep -E "(OTHER_NODE_NAMES)"

# Generate test log from another node
ssh your-username@OTHER_NODE_IP "logger 'Test log from other node'"
# Should appear in security node logs
```

## 🚨 When Things Go Wrong

### Locked Out of SSH
**If UFW blocks you:**
- Physical access required to fix
- Boot to recovery mode
- Run `sudo ufw disable` 
- Fix rules and re-enable

**Prevention:** Always test rules before enabling UFW

### Fail2Ban Blocking Legitimate Users
**Check who's banned:**
```bash
sudo fail2ban-client status sshd
```

**Unban an IP:**
```bash
sudo fail2ban-client set sshd unbanip YOUR_IP_ADDRESS
```

**Adjust settings in `/etc/fail2ban/jail.local` if too aggressive**

### Centralized Logging Not Working
**Check rsyslog configuration:**
```bash
sudo systemctl status rsyslog
sudo netstat -ulnp | grep 514
```

**Check firewall:**
```bash
sudo ufw status | grep 514
```

**Test connectivity from other nodes:**
```bash
ssh your-username@OTHER_NODE_IP "nc -u YOUR_SECURITY_IP 514"
```

### AIDE Database Issues
**Reinitialize if corrupted:**
```bash
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

**Update after legitimate system changes:**
```bash
sudo aide --update
```

### Node Exporter Not Accessible
**Check service status:**
```bash
sudo systemctl status node_exporter
```

**Check firewall:**
```bash
sudo ufw status | grep 9100
```

## 📊 Daily Monitoring Tasks

### Check Security Status
```bash
# Run security status report
sudo /usr/local/bin/security-status.sh

# Check for banned IPs
sudo fail2ban-client status sshd

# Review auth logs for suspicious activity
sudo tail -20 /var/log/auth.log
```

### Weekly Tasks
```bash
# Review network scan results
cat /var/log/cluster/network-scan.log

# Update AIDE database after system changes
sudo aide --update

# Check for rootkits
sudo rkhunter --check --skip-keypress
```

## 🎯 TL;DR Setup (Just Make It Work)

**Copy/paste these commands in order:**

```bash
# 1. User setup and static IP
sudo adduser your-username && sudo usermod -aG sudo your-username
su - your-username
mkdir -p ~/.ssh && chmod 700 ~/.ssh
# Add SSH key to ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

sudo tee /etc/netplan/50-cloud-init.yaml > /dev/null <<EOF
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - YOUR_SECURITY_IP/24
      gateway4: YOUR_ROUTER_IP
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
EOF

sudo netplan apply
sudo hostnamectl set-hostname security
echo "127.0.1.1 security" | sudo tee -a /etc/hosts
```

**Install security tools:**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ufw fail2ban nmap htop iftop aide chkrootkit rkhunter vnstat tcpdump sysstat -y
```

**Configure firewall:**
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from YOUR_NETWORK_RANGE
sudo ufw allow 22
sudo ufw allow 514/udp
sudo ufw allow 9100
sudo ufw enable
```

**Configure Fail2Ban:**
```bash
sudo tee /etc/fail2ban/jail.local > /dev/null <<EOF
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 7200
EOF

sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```

**Set up centralized logging:**
```bash
sudo sed -i 's/#module(load="imudp")/module(load="imudp")/' /etc/rsyslog.conf
sudo sed -i 's/#input(type="imudp" port="514")/input(type="imudp" port="514")/' /etc/rsyslog.conf
sudo mkdir -p /var/log/cluster
sudo chown syslog:adm /var/log/cluster
sudo systemctl restart rsyslog
```

**Install Node Exporter:**
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-arm64.tar.gz
tar xvfz node_exporter-1.6.1.linux-arm64.tar.gz
sudo mv node_exporter-1.6.1.linux-arm64/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
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
EOF

sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

**Initialize AIDE and start monitoring:**
```bash
sudo aideinit
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
sudo systemctl enable vnstat sysstat
sudo systemctl start vnstat sysstat
```

**Test everything:**
```bash
sudo ufw status
sudo fail2ban-client status
sudo systemctl status node_exporter
curl http://localhost:9100/metrics | head
```

---

## 🧌 Goblin Notes
- **Don't enable UFW until rules are set** - you'll lock yourself out
- **Fail2Ban can block legitimate users** - monitor the ban list
- **AIDE initialization takes forever** - this is normal
- **Centralized logging requires UDP port 514** - check firewall rules
- **False positives in security scans are normal** - focus on patterns
- **Keep security tools updated** - they're only as good as their databases
- **Physical access is your last resort** - don't rely on remote-only access
- **Replace YOUR_SECURITY_IP with your actual IP** - usually something like 192.168.1.xxx
- **Replace YOUR_NETWORK_RANGE with your network** - usually 192.168.1.0/24 or 192.168.0.0/24

## Network Planning Examples
- **Home network 192.168.1.x**: Use `192.168.1.0/24` in firewall rules
- **Home network 192.168.0.x**: Use `192.168.0.0/24` in firewall rules
- **Business network 10.0.x.x**: Use `10.0.0.0/16` in firewall rules
- **Check your network**: `ip route | grep default` shows your router IP

## Important Log Locations
- **System logs**: `/var/log/syslog`
- **Authentication**: `/var/log/auth.log`
- **Fail2Ban**: `/var/log/fail2ban.log`
- **UFW**: `/var/log/ufw.log`
- **Cluster logs**: `/var/log/cluster/`

---
**Node Status**: Ready for Production  
**Setup Time**: 60-90 minutes  
**Services**: UFW, Fail2Ban, AIDE, Centralized Logging, Node Exporter  
**Monitoring**: `http://YOUR_SECURITY_IP:9100/metrics`