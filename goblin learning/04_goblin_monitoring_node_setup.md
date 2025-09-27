# 🧌 Pi Cluster - Monitoring Node Setup Guide

## What This Node Does
Turns your Pi 5 into a cluster monitoring hub that watches all your other nodes, collects metrics, and gives you dashboards to see what's happening. Think mission control for your infrastructure.

**Node Details:**
- **Name**: monitoring
- **Hardware**: Raspberry Pi 5 (8GB RAM recommended)
- **IP**: YOUR_MONITORING_IP (static)
- **OS**: Ubuntu Server 24.04 LTS
- **Purpose**: Prometheus metrics collection + Grafana dashboards
- **Setup Time**: 45-60 minutes
- **Status**: Ready to configure

## 🎯 Quick Start
If you just want it working, jump to the **TL;DR Setup** section at the bottom.

## Why Pi 5 for Monitoring?

**Pi 5 is Necessary Because:**
- **Memory hungry**: Prometheus needs 2-4GB RAM minimum for small clusters
- **Storage intensive**: Metrics databases grow quickly
- **CPU requirements**: Dashboard rendering and metric processing
- **Multiple services**: Running Prometheus + Grafana + Node Exporter simultaneously

**Pi 4 Reality Check:**
- Works for 2-3 node clusters with reduced retention
- Struggles with complex dashboards
- Limited by 4GB RAM maximum
- Slower web interface response

## Hardware Requirements

### Essential Components
- **Raspberry Pi 5** (8GB RAM strongly recommended, 4GB minimum)
- **Power Supply**: 5V/5A (25W) USB-C - monitoring is surprisingly power-hungry
- **Storage**: 128GB+ SD card or NVMe SSD (metrics grow fast)
- **Network**: Gigabit Ethernet for collecting metrics from all nodes
- **Cooling**: Active cooling recommended for continuous operation

### Storage Reality
**Metric Storage Growth:**
- 5 nodes, 15-second scraping: ~1GB/month
- 10 nodes, 15-second scraping: ~2-3GB/month
- Dashboard queries cause temporary spikes

**Performance Comparison:**
- SD Card: Adequate for <5 nodes
- NVMe SSD: Recommended for >5 nodes or complex queries

## Prerequisites Check
Before starting, verify:
- Pi 5 with Ubuntu Server installed
- Static IP configured (choose an IP in your network)
- SSH access working to all nodes you want to monitor
- All target nodes have node_exporter installed (port 9100)

**Test these work before continuing!**

## 🔧 Base System Setup

### Install Essential Tools
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install htop iftop curl wget git -y
```

## 📊 Prometheus Installation

### Download and Install Prometheus
```bash
# Create monitoring directory
mkdir -p ~/cluster-monitoring
cd ~/cluster-monitoring

# Download Prometheus for ARM64
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-arm64.tar.gz
tar xvfz prometheus-2.45.0.linux-arm64.tar.gz

# Install binaries
sudo mv prometheus-2.45.0.linux-arm64/prometheus /usr/local/bin/
sudo mv prometheus-2.45.0.linux-arm64/promtool /usr/local/bin/

# Create user and directories
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus

# Copy default config
sudo cp prometheus-2.45.0.linux-arm64/prometheus.yml /etc/prometheus/
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

### Create Prometheus Service
```bash
sudo nano /etc/systemd/system/prometheus.service
```

**Add this configuration:**
```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --storage.tsdb.retention.time=30d \
    --storage.tsdb.retention.size=10GB \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

## 📈 Grafana Installation

### Install Grafana
```bash
# Add Grafana repository
sudo apt-get install -y software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Install Grafana
sudo apt update
sudo apt install grafana -y
```

## 🔧 Configure Prometheus for Your Cluster

### Edit Prometheus Configuration
```bash
sudo nano /etc/prometheus/prometheus.yml
```

**Replace with your cluster configuration:**
```yaml
global:
  scrape_interval: 30s      # Reduced for Pi performance

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'cluster-nodes'
    static_configs:
      - targets: 
        - 'YOUR_GIT_NODE_IP:9100'
        - 'YOUR_SECURITY_NODE_IP:9100'
        - 'YOUR_STORAGE_NODE_IP:9100'
        - 'YOUR_MONITORING_IP:9100'
        - 'YOUR_AUTOMATION_NODE_IP:9100'
        # Add more nodes as needed
```

**Example for 192.168.1.x network:**
```yaml
global:
  scrape_interval: 30s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'cluster-nodes'
    static_configs:
      - targets: 
        - '192.168.1.10:9100'  # git node
        - '192.168.1.11:9100'  # security node
        - '192.168.1.12:9100'  # storage node
        - '192.168.1.13:9100'  # automation node
        - '192.168.1.14:9100'  # monitoring node (self)
```

### Install Node Exporter (For This Node)
```bash
cd ~/cluster-monitoring
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-arm64.tar.gz
tar xvfz node_exporter-1.6.1.linux-arm64.tar.gz
sudo mv node_exporter-1.6.1.linux-arm64/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

# Create service
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
```

### Start All Services
```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus grafana-server node_exporter
sudo systemctl start prometheus grafana-server node_exporter

# Check they're running
sudo systemctl status prometheus
sudo systemctl status grafana-server
sudo systemctl status node_exporter
```

## 🔍 Cluster Health Monitoring Scripts

### Create Cluster Health Check Script
```bash
nano ~/cluster-monitoring/cluster-health.sh
```

**Add this script (adjust IPs for your setup):**
```bash
#!/bin/bash
# Replace these with your actual node IPs and names
NODES=(
    "YOUR_GIT_NODE_IP:git"
    "YOUR_SECURITY_NODE_IP:security" 
    "YOUR_STORAGE_NODE_IP:storage"
    "YOUR_AUTOMATION_NODE_IP:automation"
)
LOGFILE="/var/log/cluster-health.log"

echo "$(date): Cluster Health Check" >> $LOGFILE

for node in "${NODES[@]}"; do
    IFS=':' read -r ip name <<< "$node"
    if ping -c 1 "$ip" >/dev/null 2>&1; then
        echo "$(date): $name ($ip) - UP" >> $LOGFILE
        # Check SSH access
        if ssh -o ConnectTimeout=5 your-username@"$ip" 'uptime' >/dev/null 2>&1; then
            echo "$(date): $name SSH - OK" >> $LOGFILE
        else
            echo "$(date): $name SSH - FAILED" >> $LOGFILE
        fi
    else
        echo "$(date): $name ($ip) - DOWN" >> $LOGFILE
    fi
done
```

### Create Service Status Monitor
```bash
nano ~/cluster-monitoring/service-monitor.sh
```

**Add this script:**
```bash
#!/bin/bash
LOGFILE="/var/log/cluster-services.log"

echo "$(date): Service Status Check" >> $LOGFILE

# Check Git server (adjust IP)
if curl -s -o /dev/null -w "%{http_code}" http://YOUR_GIT_NODE_IP:3000 | grep -q "200"; then
    echo "$(date): Git Server - UP" >> $LOGFILE
else
    echo "$(date): Git Server - DOWN" >> $LOGFILE
fi

# Check NFS on storage node (adjust IP)
if showmount -e YOUR_STORAGE_NODE_IP >/dev/null 2>&1; then
    echo "$(date): NFS Storage - UP" >> $LOGFILE
else
    echo "$(date): NFS Storage - DOWN" >> $LOGFILE
fi

# Check Home Assistant (adjust IP)
if curl -s -o /dev/null -w "%{http_code}" http://YOUR_AUTOMATION_NODE_IP:8123 | grep -q "200"; then
    echo "$(date): Home Assistant - UP" >> $LOGFILE
else
    echo "$(date): Home Assistant - DOWN" >> $LOGFILE
fi

# Check local services
if curl -s -o /dev/null -w "%{http_code}" http://localhost:9090 | grep -q "200"; then
    echo "$(date): Prometheus - UP" >> $LOGFILE
else
    echo "$(date): Prometheus - DOWN" >> $LOGFILE
fi

if curl -s -o /dev/null -w "%{http_code}" http://localhost:3000 | grep -q "200"; then
    echo "$(date): Grafana - UP" >> $LOGFILE
else
    echo "$(date): Grafana - DOWN" >> $LOGFILE
fi
```

### Make Scripts Executable and Schedule
```bash
chmod +x ~/cluster-monitoring/cluster-health.sh
chmod +x ~/cluster-monitoring/service-monitor.sh

# Schedule in crontab
crontab -e
```

**Add these lines:**
```bash
# Cluster health check every 5 minutes
*/5 * * * * /home/your-username/cluster-monitoring/cluster-health.sh

# Service status check every 10 minutes  
*/10 * * * * /home/your-username/cluster-monitoring/service-monitor.sh
```

## 🌐 Web Interface Access

### Prometheus Web Interface
- **URL**: `http://YOUR_MONITORING_IP:9090`
- **Purpose**: Metrics querying, target monitoring
- **Key Pages**:
  - Status → Targets (view all monitored nodes)
  - Graph (query metrics and create visualizations)

### Grafana Dashboard
- **URL**: `http://YOUR_MONITORING_IP:3000`
- **Default Login**: admin/admin (change on first login)
- **Setup Steps**:
  1. Login and change default password
  2. Add data source: Configuration → Data Sources → Prometheus
  3. URL: `http://localhost:9090`
  4. Save & Test

## 📊 Essential Dashboards for Pi Clusters

### Import Pre-built Dashboard
1. Go to Grafana → + → Import
2. Use Dashboard ID: `1860` (Node Exporter Full)
3. Select Prometheus data source
4. Import

### Key Metrics to Monitor
- **CPU Usage**: Watch for thermal throttling on Pis
- **Memory Usage**: Critical on limited Pi RAM
- **Disk I/O**: Monitor SD card performance
- **Temperature**: Prevent Pi overheating
- **Network Traffic**: Ensure monitoring doesn't overwhelm network

## 🚨 When Things Go Wrong

### Prometheus Not Starting
```bash
# Check service status
sudo systemctl status prometheus

# Check logs
sudo journalctl -u prometheus -f

# Common issues:
# - Config file syntax error: sudo promtool check config /etc/prometheus/prometheus.yml
# - Permission issues: sudo chown -R prometheus:prometheus /var/lib/prometheus
# - Port conflicts: sudo ss -tlnp | grep 9090
```

### Grafana Connection Issues
```bash
# Check Grafana status
sudo systemctl status grafana-server

# Check if it's listening
sudo ss -tlnp | grep 3000

# Reset admin password if needed
sudo grafana-cli admin reset-admin-password newpassword
```

### Node Targets Down in Prometheus
```bash
# Check if node_exporter is running on target nodes
ssh your-username@TARGET_NODE_IP "sudo systemctl status node_exporter"

# Check if port 9100 is accessible
telnet TARGET_NODE_IP 9100

# Check firewall on target nodes
ssh your-username@TARGET_NODE_IP "sudo ufw status"
```

### High Memory Usage
```bash
# Check Prometheus memory usage
sudo systemctl status prometheus
htop

# Reduce retention if needed
sudo nano /etc/systemd/system/prometheus.service
# Modify: --storage.tsdb.retention.time=15d
sudo systemctl daemon-reload
sudo systemctl restart prometheus
```

### Performance Issues
```bash
# Monitor Pi temperature
vcgencmd measure_temp

# Check SD card health
sudo dmesg | grep mmc

# Monitor system resources
htop
iostat 1

# Increase scrape interval if needed
sudo nano /etc/prometheus/prometheus.yml
# Change: scrape_interval: 60s
```

## 📝 Log Management

### Important Log Locations
- **Cluster health**: `/var/log/cluster-health.log`
- **Service status**: `/var/log/cluster-services.log`
- **Prometheus**: `sudo journalctl -u prometheus`
- **Grafana**: `sudo journalctl -u grafana-server`

### Set Up Log Rotation
```bash
sudo nano /etc/logrotate.d/cluster-monitoring
```

**Add this configuration:**
```bash
/var/log/cluster-health.log /var/log/cluster-services.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 644 your-username your-username
}
```

## 🎯 TL;DR Setup (Just Make It Work)

**Install monitoring stack:**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install htop iftop curl wget git software-properties-common -y

# Create monitoring directory
mkdir -p ~/cluster-monitoring && cd ~/cluster-monitoring
```

**Install Prometheus:**
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-arm64.tar.gz
tar xvfz prometheus-2.45.0.linux-arm64.tar.gz
sudo mv prometheus-2.45.0.linux-arm64/prometheus /usr/local/bin/
sudo mv prometheus-2.45.0.linux-arm64/promtool /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false prometheus
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
sudo cp prometheus-2.45.0.linux-arm64/prometheus.yml /etc/prometheus/
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

**Install Grafana:**
```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
sudo apt update && sudo apt install grafana -y
```

**Install Node Exporter:**
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-arm64.tar.gz
tar xvfz node_exporter-1.6.1.linux-arm64.tar.gz
sudo mv node_exporter-1.6.1.linux-arm64/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

**Create services:**
```bash
# Prometheus service
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus --config.file /etc/prometheus/prometheus.yml --storage.tsdb.path /var/lib/prometheus/ --storage.tsdb.retention.time=30d --storage.tsdb.retention.size=10GB

[Install]
WantedBy=multi-user.target
EOF

# Node Exporter service  
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
```

**Configure Prometheus targets:**
```bash
sudo tee /etc/prometheus/prometheus.yml > /dev/null <<EOF
global:
  scrape_interval: 30s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'cluster-nodes'
    static_configs:
      - targets: 
        - 'YOUR_NODE_1_IP:9100'
        - 'YOUR_NODE_2_IP:9100'
        - 'YOUR_NODE_3_IP:9100'
        # Add your actual node IPs here
EOF
```

**Start everything:**
```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus grafana-server node_exporter
sudo systemctl start prometheus grafana-server node_exporter
```

**Access interfaces:**
- Prometheus: `http://YOUR_MONITORING_IP:9090`
- Grafana: `http://YOUR_MONITORING_IP:3000` (admin/admin)

---

## 🧌 Goblin Notes
- **Pi 5 with 8GB RAM is not optional** - monitoring is memory-hungry
- **Temperature monitoring is critical** - Pis throttle when hot
- **Start with longer scrape intervals** - you can always decrease later
- **SD card performance matters** - metrics databases are I/O intensive
- **Don't over-monitor** - too many metrics slow everything down
- **Test your dashboards** - pretty charts are useless if they're wrong
- **Replace YOUR_NODE_IP with actual IPs** - the scripts won't work with placeholders
- **Monitor the monitor** - if this node goes down, you're blind

## Performance Tips for Pi Clusters
- **Scrape interval 30s+** - shorter intervals overwhelm Pis
- **Retention 15-30 days** - longer periods fill storage quickly
- **Limit dashboard complexity** - complex queries slow response
- **Use active cooling** - sustained load generates heat
- **Monitor disk usage** - metrics grow faster than you expect

---
**Node Status**: Ready for Production  
**Setup Time**: 45-60 minutes  
**Services**: Prometheus, Grafana, Node Exporter, Health Monitoring  
**Access**: `http://YOUR_MONITORING_IP:3000` (Grafana), `http://YOUR_MONITORING_IP:9090` (Prometheus)