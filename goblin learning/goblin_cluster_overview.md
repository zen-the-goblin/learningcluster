# 🧌 Pi Cluster - Complete Infrastructure Overview

## What This Cluster Actually Does
Your Pi cluster is a self-contained development and automation environment that handles code repositories, file storage, security monitoring, home automation, and system backups. Think of it as your personal mini data center that actually works.

## 🏗️ Cluster Architecture

```
Internet Router (YOUR_ROUTER_IP)
├── Development Machine (YOUR_DEV_IP) - Main workstation, monitoring hub
├── Git Node (YOUR_GIT_IP) - Pi 5, Repository Server  
├── Security Node (YOUR_SECURITY_IP) - Pi 4, Firewall & Logging
├── Storage Node (YOUR_STORAGE_IP) - Pi 5, NFS File Server
└── Automation Node (YOUR_AUTOMATION_IP) - Pi 5, Home Assistant
```

### Node Roles and Relationships

**Development Machine (YOUR_DEV_IP)**
- **Purpose**: Main development workstation, monitoring dashboard, backup orchestration
- **Hardware**: Desktop/laptop with Ubuntu (your main machine)
- **Key Services**: Prometheus, Grafana, development tools, backup scripts
- **Role**: Cluster brain that monitors everything else

**Git Node (YOUR_GIT_IP)**
- **Purpose**: Self-hosted Git server alternative
- **Hardware**: Raspberry Pi 5
- **Key Services**: Gitea web interface, SSH git access
- **Role**: Code repository hosting for all projects

**Security Node (YOUR_SECURITY_IP)**  
- **Purpose**: Network security and log aggregation
- **Hardware**: Raspberry Pi 4
- **Key Services**: UFW firewall, Fail2Ban, centralized logging
- **Role**: Cluster security guard and log collector

**Storage Node (YOUR_STORAGE_IP)**
- **Purpose**: Shared file storage and automated backups
- **Hardware**: Raspberry Pi 5  
- **Key Services**: NFS server, automated git backups
- **Role**: Cluster shared storage and backup target

**Automation Node (YOUR_AUTOMATION_IP)**
- **Purpose**: Home automation and camera monitoring
- **Hardware**: Raspberry Pi 5 with touchscreen
- **Key Services**: Home Assistant, camera feeds, touchscreen dashboard
- **Role**: Smart home brain with visual interface

## 🌐 Network Access Points

### Web Interfaces
- **Gitea (Code)**: `http://YOUR_GIT_IP:3000`
- **Home Assistant**: `http://YOUR_AUTOMATION_IP:8123`  
- **Prometheus (Monitoring)**: `http://YOUR_DEV_IP:9090`
- **Grafana (Dashboards)**: `http://YOUR_DEV_IP:3000`

### SSH Access
```bash
ssh your-username@YOUR_GIT_IP      # Git node
ssh your-username@YOUR_SECURITY_IP # Security node  
ssh your-username@YOUR_STORAGE_IP  # Storage node
ssh your-username@YOUR_AUTOMATION_IP # Automation node
ssh your-username@YOUR_DEV_IP      # Development machine
```

### Git Repository Access
```bash
# Clone repos via SSH
git clone git@YOUR_GIT_IP:username/repository.git

# Clone repos via HTTPS
git clone http://YOUR_GIT_IP:3000/username/repository.git
```

### File Storage Access
```bash
# Mount NFS shares from any node
sudo mount -t nfs YOUR_STORAGE_IP:/srv/nfs/shared /mnt/storage/shared
sudo mount -t nfs YOUR_STORAGE_IP:/srv/nfs/backups /mnt/storage/backups
```

## 🔧 Service Port Reference

### Git Node (YOUR_GIT_IP)
- **22**: SSH access
- **3000**: Gitea web interface
- **9100**: Node Exporter metrics

### Security Node (YOUR_SECURITY_IP)  
- **22**: SSH access
- **514**: Syslog server (UDP)
- **9100**: Node Exporter metrics

### Storage Node (YOUR_STORAGE_IP)
- **22**: SSH access
- **111**: NFS portmapper  
- **2049**: NFS server
- **9100**: Node Exporter metrics

### Automation Node (YOUR_AUTOMATION_IP)
- **22**: SSH access
- **8123**: Home Assistant web interface
- **9100**: Node Exporter metrics

### Development Machine (YOUR_DEV_IP)
- **22**: SSH access
- **3000**: Grafana dashboard
- **9090**: Prometheus monitoring

## 🔄 Data Flow and Backups

### Automated Backup Chain
1. **Git Repositories**: Daily backup from git node to storage node NFS
2. **Home Assistant Config**: Daily backup from automation node to development machine
3. **System Configs**: Daily backup of all node configurations to development machine
4. **Retention**: 30 days for code/configs, 7 days for system backups

### Monitoring Data Flow
1. **Metrics Collection**: All nodes run Node Exporter (port 9100)
2. **Central Monitoring**: Development machine's Prometheus scrapes all nodes every 30 seconds
3. **Visualization**: Grafana on development machine displays cluster health dashboards
4. **Log Aggregation**: Security node collects logs from all other nodes

### File Sharing Flow
1. **Central Storage**: Storage node provides NFS shares to all nodes
2. **Shared Data**: `/srv/nfs/shared` for inter-node file transfer
3. **Backup Storage**: `/srv/nfs/backups` for automated backup storage
4. **Config Sharing**: `/srv/nfs/configs` for shared configuration files

## 🚀 Quick Start Operations

### Start a New Project
```bash
# On development machine or any development environment
ssh your-username@YOUR_GIT_IP
# Create repository via Gitea web interface at YOUR_GIT_IP:3000
# Clone to local development environment
git clone git@YOUR_GIT_IP:username/project.git
```

### Check Cluster Health  
```bash
# Access monitoring dashboard
open http://YOUR_DEV_IP:3000     # Grafana
open http://YOUR_DEV_IP:9090     # Prometheus

# Check individual node status
ssh your-username@YOUR_SECURITY_IP "sudo /usr/local/bin/security-status.sh"
```

### Access Shared Files
```bash
# From any node, mount storage
sudo mount -t nfs YOUR_STORAGE_IP:/srv/nfs/shared /mnt/storage

# Copy files between nodes
cp important-file.txt /mnt/storage/
```

### View Home Automation
```bash
# Access via web interface
open http://YOUR_AUTOMATION_IP:8123

# Or view directly on automation node's touchscreen
```

## 🔍 Inter-Node Dependencies

### Critical Dependencies
- **Storage Node** must be running for automated backups to work
- **Security Node** must be running for centralized logging
- **Development Machine** must be running for cluster monitoring and backup orchestration
- **Git Node** is independent but backs up to Storage Node

### Service Startup Order
1. **Storage Node** (provides NFS for others)
2. **Security Node** (provides logging for others)  
3. **Git Node** (independent, can start anytime)
4. **Automation Node** (independent, can start anytime)
5. **Development Machine** (monitoring, can start anytime after others)

### Network Requirements
- All nodes must be on same subnet (YOUR_NETWORK_RANGE)
- Static IP addresses prevent service discovery issues
- SSH key authentication enables automated operations
- NFS requires ports 111 and 2049 open on storage node

## 🚨 Cluster-Wide Troubleshooting

### "Can't Access Web Interface"
**Check in this order:**
1. Is the node powered on? `ping YOUR_NODE_IP`
2. Is the service running? `ssh your-username@YOUR_NODE_IP "sudo systemctl status service-name"`
3. Is the port open? `nmap YOUR_NODE_IP -p PORT`
4. Check firewall rules on the target node

### "Git Operations Failing"
**Common causes:**
1. SSH key not added to Gitea web interface
2. Git node is down: `ping YOUR_GIT_IP`
3. Gitea service stopped: `ssh your-username@YOUR_GIT_IP "sudo systemctl status gitea"`
4. Repository permissions issue in Gitea web interface

### "NFS Mounts Failing"
**Troubleshooting steps:**
1. Storage node running? `ping YOUR_STORAGE_IP`
2. NFS service active? `ssh your-username@YOUR_STORAGE_IP "sudo systemctl status nfs-kernel-server"`
3. Exports available? `showmount -e YOUR_STORAGE_IP`
4. Firewall blocking? Check UFW rules on storage node

### "Monitoring Data Missing"
**Check chain:**
1. Node Exporter running? `curl http://YOUR_NODE_IP:9100/metrics`
2. Prometheus scraping? Check targets at `http://YOUR_DEV_IP:9090/targets`
3. Grafana connected? Check data sources in Grafana
4. Network connectivity between development machine and target node

### "Backup Script Failures"
**Common issues:**
1. SSH keys not working between development machine and target nodes
2. Target directories don't exist or have wrong permissions
3. Disk space full on backup destination
4. Services stopped on source nodes

## 📊 Daily Health Checks

### Quick Status Commands
```bash
# Check all nodes are reachable
for ip in YOUR_GIT_IP YOUR_SECURITY_IP YOUR_STORAGE_IP YOUR_AUTOMATION_IP; do 
    echo "$ip: $(ping -c1 $ip >/dev/null && echo "UP" || echo "DOWN")"
done

# Check key services
curl -s http://YOUR_GIT_IP:3000 >/dev/null && echo "Git: UP" || echo "Git: DOWN"
curl -s http://YOUR_AUTOMATION_IP:8123 >/dev/null && echo "Home Assistant: UP" || echo "Home Assistant: DOWN"  
curl -s http://YOUR_DEV_IP:9090 >/dev/null && echo "Prometheus: UP" || echo "Prometheus: DOWN"

# Check backup status
ssh your-username@YOUR_DEV_IP "tail -5 /var/log/cluster-backup.log"
```

### Weekly Maintenance
```bash
# Update all nodes
for ip in YOUR_GIT_IP YOUR_SECURITY_IP YOUR_STORAGE_IP YOUR_AUTOMATION_IP; do 
    ssh your-username@$ip "sudo apt update && sudo apt upgrade -y"
done

# Check disk usage on all nodes  
for ip in YOUR_GIT_IP YOUR_SECURITY_IP YOUR_STORAGE_IP YOUR_AUTOMATION_IP; do 
    echo "=== $ip ==="
    ssh your-username@$ip "df -h"
done

# Review security logs
ssh your-username@YOUR_SECURITY_IP "sudo fail2ban-client status sshd"
```

## 🔧 Expansion and Scaling

### Adding New Nodes
1. **Configure static IP** in YOUR_NETWORK_RANGE
2. **Install Node Exporter** for monitoring integration
3. **Add SSH keys** for development machine backup access
4. **Configure centralized logging** to security node
5. **Update Prometheus** configuration on development machine
6. **Mount NFS shares** from storage node

### Adding New Services
1. **Check port conflicts** with existing services
2. **Configure firewall rules** if needed
3. **Add monitoring** via Node Exporter or custom metrics
4. **Set up backup strategy** if service has persistent data
5. **Document access methods** and integration points

### Hardware Upgrades
1. **Plan for Pi 5 thermal management** (active cooling required)
2. **Storage expansion** via USB 3.0 drives on storage node
3. **Network upgrades** to Gigabit switches for better NFS performance
4. **UPS backup power** for cluster reliability

## 🧌 Goblin Operational Notes

### Things That Actually Break
- **Pi 5 overheating** causing random shutdowns (get better cooling)
- **SD card corruption** from power failures (get UPS)
- **SSH key rot** when rebuilding nodes (keep backup of keys)
- **IP address changes** breaking everything (use static IPs religiously)
- **Firewall rules** blocking legitimate traffic (document exceptions)

### Things That Look Broken But Aren't
- **Permission errors** in backup logs for system files (normal)
- **NFS "nobody:nogroup" ownership** (correct for NFS)
- **Node Exporter "failed to read" errors** (expected for some system files)
- **Home Assistant first startup** taking 3+ minutes (normal)
- **Fail2Ban blocking your own IP** after typos (check ban list)

### Maintenance Reality
- **Check backups weekly** - they fail silently
- **Monitor temperatures** - Pis overheat easily
- **Update gradually** - don't update all nodes at once
- **Keep documentation current** - you'll forget how things work
- **Test restore procedures** - backups are useless if you can't restore

### When Everything Goes Wrong
1. **Development machine is your lifeline** - keep it simple and stable
2. **Physical access to security node** may be needed for firewall issues
3. **Storage node failure** breaks backups but not services
4. **Start with network connectivity** - 90% of issues are network-related
5. **Rebuild from documentation** rather than trying to fix corrupted nodes

## 📋 Quick Reference Template

### SSH Configuration Template
Replace YOUR_* placeholders with your actual IPs:

```
Host git
    HostName YOUR_GIT_IP
    User your-username

Host security
    HostName YOUR_SECURITY_IP
    User your-username

Host storage
    HostName YOUR_STORAGE_IP
    User your-username

Host automation
    HostName YOUR_AUTOMATION_IP
    User your-username

Host dev
    HostName YOUR_DEV_IP
    User your-username
```

### Network Planning Template
Choose IPs that work for your network:

```
Router: YOUR_ROUTER_IP (usually 192.168.1.1 or 192.168.0.1)
Network Range: YOUR_NETWORK_RANGE (usually 192.168.1.0/24 or 192.168.0.0/24)

Git Node: YOUR_GIT_IP (e.g., 192.168.1.10)
Security Node: YOUR_SECURITY_IP (e.g., 192.168.1.11)
Storage Node: YOUR_STORAGE_IP (e.g., 192.168.1.12)
Automation Node: YOUR_AUTOMATION_IP (e.g., 192.168.1.13)
Development Machine: YOUR_DEV_IP (e.g., 192.168.1.100)
```

### Service Access Quick Reference
Replace YOUR_* with your actual IPs:

**Cluster Status Dashboard**: `http://YOUR_DEV_IP:3000`  
**Code Repositories**: `http://YOUR_GIT_IP:3000`  
**Home Automation**: `http://YOUR_AUTOMATION_IP:8123`  
**Backup Logs**: `/var/log/cluster-backup.log` on development machine  
**Emergency SSH**: `ssh your-username@YOUR_SECURITY_IP` (security node)

## 📁 Documentation Structure Template

### Repository Organization
```
cluster-config/
├── docs/
│   ├── cluster-overview.md (this document)
│   ├── nodes/
│   │   ├── 01-git-node.md
│   │   ├── 02-storage-node.md
│   │   ├── 03-security-node.md
│   │   ├── 04-monitoring-node.md
│   │   └── 05-automation-node.md
│   └── guides/
│       ├── troubleshooting.md
│       └── maintenance.md
├── configs/
│   ├── prometheus/
│   ├── grafana/
│   └── scripts/
└── README.md
```

---

## Network Planning Examples

**Common Home Networks:**
- **192.168.1.x network**: Router at 192.168.1.1, use 192.168.1.0/24 in configs
- **192.168.0.x network**: Router at 192.168.0.1, use 192.168.0.0/24 in configs
- **10.0.x.x network**: Router at 10.0.0.1, use 10.0.0.0/16 in configs

**Finding Your Network:**
```bash
# Check your router IP
ip route | grep default

# Scan your network for used IPs
nmap -sn 192.168.1.0/24  # Adjust for your network
```

**Total Node Count**: 5 (Development Machine + 4 Pis)  
**Total Services**: 12+ (Gitea, Home Assistant, Prometheus, Grafana, NFS, etc.)  
**Network Requirements**: All nodes on same subnet with static IPs  
**Backup Retention**: 30 days (critical), 7 days (system configs)