# Pi Cluster - Monitoring Node Setup Guide

## Node Overview
- **Node Name**: monitoring
- **Hardware**: Raspberry Pi 5 (8GB RAM recommended)
- **IP Address**: Static IP configuration
- **OS**: Ubuntu Server 24.04 LTS
- **Purpose**: Cluster monitoring, metrics collection, visualization
- **Status**: Ready for deployment

## Hardware Specifications
- **CPU**: ARM Cortex-A76 quad-core (Pi 5)
- **RAM**: 8GB recommended (4GB minimum)
- **Storage**: 128GB+ microSD card or NVMe SSD via HAT
- **Network**: Gigabit Ethernet
- **Cooling**: Active cooling recommended for continuous operation

## Prerequisites
- Ubuntu Server 24.04 LTS installed and configured
- Static IP configured on cluster subnet
- SSH access to all cluster nodes
- Network connectivity to all nodes being monitored

## Monitoring Stack Installation

### 1. Install Base Monitoring Tools
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install htop iftop nmon curl wget git -y
```

### 2. Install Prometheus
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

### 3. Create Prometheus Service
```bash
sudo nano /etc/systemd/system/prometheus.service
```

Service configuration:
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
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```

### 4. Install Grafana
```bash
# Add Grafana repository
sudo apt-get install -y software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Install Grafana
sudo apt update
sudo apt install grafana -y
```

### 5. Configure Prometheus for Cluster Monitoring
```bash
sudo nano /etc/prometheus/prometheus.yml
```

Configuration template:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'cluster-nodes'
    static_configs:
      - targets: 
        - 'git-node-ip:9100'
        - 'security-node-ip:9100'
        - 'storage-node-ip:9100'
        - 'monitoring-node-ip:9100'
        - 'homeassist-node-ip:9100'
```

### 6. Install Node Exporter
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

### 7. Start All Services
```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus grafana-server node_exporter
sudo systemctl start prometheus grafana-server node_exporter
```

## Cluster Health Monitoring Scripts

### 1. Cluster Health Check Script
```bash
nano ~/cluster-monitoring/cluster-health.sh
```

Script content:
```bash
#!/bin/bash
NODES=("git-node-ip:git" "security-node-ip:security" "storage-node-ip:storage" "homeassist-node-ip:homeassist")
LOGFILE="/var/log/cluster-health.log"

echo "$(date): Cluster Health Check" >> $LOGFILE

for node in "${NODES[@]}"; do
    IFS=':' read -r ip name <<< "$node"
    if ping -c 1 "$ip" >/dev/null 2>&1; then
        echo "$(date): $name ($ip) - UP" >> $LOGFILE
        # Check SSH access
        if ssh -o ConnectTimeout=5 username@"$ip" 'uptime' >/dev/null 2>&1; then
            echo "$(date): $name SSH - OK" >> $LOGFILE
        else
            echo "$(date): $name SSH - FAILED" >> $LOGFILE
        fi
    else
        echo "$(date): $name ($ip) - DOWN" >> $LOGFILE
    fi
done
```

Make executable and schedule:
```bash
chmod +x ~/cluster-monitoring/cluster-health.sh
crontab -e
```

Add to cron (runs every 5 minutes):
```
*/5 * * * * /home/username/cluster-monitoring/cluster-health.sh
```

### 2. Service Status Monitoring
```bash
nano ~/cluster-monitoring/service-monitor.sh
```

Script content:
```bash
#!/bin/bash
LOGFILE="/var/log/cluster-services.log"

echo "$(date): Service Status Check" >> $LOGFILE

# Check Git server
if curl -s -o /dev/null -w "%{http_code}" http://git-node-ip:3000 | grep -q "200"; then
    echo "$(date): Git Server - UP" >> $LOGFILE
else
    echo "$(date): Git Server - DOWN" >> $LOGFILE
fi

# Check NFS on storage node
if showmount -e storage-node-ip >/dev/null 2>&1; then
    echo "$(date): NFS Storage - UP" >> $LOGFILE
else
    echo "$(date): NFS Storage - DOWN" >> $LOGFILE
fi

# Check Prometheus
if curl -s -o /dev/null -w "%{http_code}" http://localhost:9090 | grep -q "200"; then
    echo "$(date): Prometheus - UP" >> $LOGFILE
else
    echo "$(date): Prometheus - DOWN" >> $LOGFILE
fi

# Check Grafana
if curl -s -o /dev/null -w "%{http_code}" http://localhost:3000 | grep -q "200"; then
    echo "$(date): Grafana - UP" >> $LOGFILE
else
    echo "$(date): Grafana - DOWN" >> $LOGFILE
fi

# Check Home Assistant
if curl -s -o /dev/null -w "%{http_code}" http://homeassist-node-ip:8123 | grep -q "200"; then
    echo "$(date): Home Assistant - UP" >> $LOGFILE
else
    echo "$(date): Home Assistant - DOWN" >> $LOGFILE
fi
```

Make executable and schedule:
```bash
chmod +x ~/cluster-monitoring/service-monitor.sh
crontab -e
```

Add to cron (runs every 10 minutes):
```
*/10 * * * * /home/username/cluster-monitoring/service-monitor.sh
```

## Backup Verification System

### 1. Backup Verification Script
```bash
nano ~/cluster-monitoring/verify-backups.sh
```

Script content:
```bash
#!/bin/bash
STORAGE_NODE="storage-node-ip"
BACKUP_PATH="/srv/nfs/backups"
LOGFILE="/var/log/backup-verification.log"

echo "$(date): Backup Verification Started" >> $LOGFILE

# Mount NFS backup share temporarily
sudo mkdir -p /mnt/backup-check
if sudo mount -t nfs $STORAGE_NODE:$BACKUP_PATH /mnt/backup-check; then
    echo "$(date): Backup share mounted successfully" >> $LOGFILE
    
    # Check Git repository backups
    LATEST_BACKUP=$(ls -t /mnt/backup-check/git-repos/ | head -n1)
    if [ -n "$LATEST_BACKUP" ]; then
        BACKUP_SIZE=$(du -sh "/mnt/backup-check/git-repos/$LATEST_BACKUP" | cut -f1)
        echo "$(date): Latest Git backup: $LATEST_BACKUP (Size: $BACKUP_SIZE)" >> $LOGFILE
    else
        echo "$(date): ERROR: No Git backups found" >> $LOGFILE
    fi
    
    sudo umount /mnt/backup-check
else
    echo "$(date): ERROR: Could not mount backup share" >> $LOGFILE
fi
```

Make executable and schedule:
```bash
chmod +x ~/cluster-monitoring/verify-backups.sh
crontab -e
```

Add to cron (runs daily at 3 AM):
```
0 3 * * * /home/username/cluster-monitoring/verify-backups.sh
```

## Web Interface Access

### 1. Prometheus Web Interface
- **URL**: `http://monitoring-node-ip:9090`
- **Purpose**: Metrics querying, target monitoring, alert management
- **Key Pages**:
  - Status → Targets (view all monitored nodes)
  - Graph (query metrics and create visualizations)
  - Alerts (view active alerts)

### 2. Grafana Dashboard
- **URL**: `http://monitoring-node-ip:3000`
- **Default Login**: admin/admin (change on first login)
- **Purpose**: Advanced dashboards, alerting, visualization
- **Setup**: Add Prometheus as data source (http://localhost:9090)

## Log Management

### 1. Log Locations
- Cluster health: `/var/log/cluster-health.log`
- Service status: `/var/log/cluster-services.log`
- Backup verification: `/var/log/backup-verification.log`
- Prometheus logs: `journalctl -u prometheus`
- Grafana logs: `journalctl -u grafana-server`

### 2. Log Rotation
```bash
sudo nano /etc/logrotate.d/cluster-monitoring
```

Configuration:
```
/var/log/cluster-health.log /var/log/cluster-services.log /var/log/backup-verification.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 644 username username
}
```

## Performance Considerations for Pi Hardware

### Resource Management
- **Memory Usage**: Prometheus can be memory-intensive; 8GB Pi 5 recommended
- **Storage**: Use high-quality SD card or NVMe SSD for better I/O performance
- **Retention**: Configure shorter retention periods if storage is limited
- **Scrape Intervals**: Adjust intervals based on cluster size and Pi performance

### Optimization Settings
```bash
# Add to Prometheus configuration for Pi optimization
sudo nano /etc/prometheus/prometheus.yml
```

Add these settings:
```yaml
global:
  scrape_interval: 30s      # Longer interval for Pi
  evaluation_interval: 30s  # Reduce evaluation frequency

# Add storage retention settings to service file
sudo nano /etc/systemd/system/prometheus.service
```

Modify ExecStart line:
```
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --storage.tsdb.retention.time=30d \
    --storage.tsdb.retention.size=10GB \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries
```

## Maintenance Tasks

### Daily Monitoring
- Check Prometheus targets: All nodes should show "UP"
- Review cluster health logs: `tail -f /var/log/cluster-health.log`
- Verify service status: `tail -f /var/log/cluster-services.log`
- Monitor Pi temperature: `vcgencmd measure_temp`

### Weekly Tasks
- Review backup verification logs
- Check system resource usage in Grafana
- Update monitoring dashboards as needed
- Verify SD card health: `sudo dmesg | grep mmc`

### Monthly Tasks
- System updates: `sudo apt update && sudo apt upgrade`
- Review and clean old logs
- Check disk space: `df -h`
- Verify all monitoring scripts are functioning

## Performance Monitoring

### Key Metrics for Pi Clusters
- **CPU Usage**: Monitor ARM CPU performance across nodes
- **Memory Usage**: Critical on Pi hardware with limited RAM
- **SD Card I/O**: Watch for performance degradation
- **Temperature**: Monitor Pi thermal throttling
- **Network Traffic**: Ensure monitoring doesn't overwhelm network

### Pi-Specific Alert Thresholds
- CPU usage > 70% for 5 minutes (lower threshold for Pi)
- Memory usage > 85% for 5 minutes (Pi memory constraints)
- Temperature > 70°C (thermal throttling protection)
- Disk usage > 80% (SD card space management)
- Service down for > 2 minutes

## Troubleshooting

### Pi-Specific Issues
- **High temperature**: Ensure adequate cooling, check `vcgencmd measure_temp`
- **SD card corruption**: Monitor `dmesg` for I/O errors, consider NVMe upgrade
- **Memory pressure**: Reduce retention periods, increase swap if needed
- **Network issues**: Check ethernet connection, verify static IP configuration

### Common Issues
- **Prometheus targets down**: Check node_exporter services on monitored nodes
- **Grafana connection issues**: Verify Prometheus data source configuration
- **High resource usage**: Adjust scrape intervals and retention settings
- **Log file growth**: Ensure log rotation is properly configured

### Emergency Procedures
- Stop monitoring services: `sudo systemctl stop prometheus grafana-server`
- Restart individual services: `sudo systemctl restart <service>`
- Check service logs: `journalctl -u <service> -f`
- Monitor system resources: `htop`, `iostat`, `free -h`

## Hardware Recommendations

### Optimal Pi Configuration
- **Model**: Raspberry Pi 5 (8GB RAM)
- **Storage**: NVMe SSD via official HAT (preferred) or high-quality SD card
- **Cooling**: Active cooling solution (fan + heatsink)
- **Power**: Official 27W USB-C power supply
- **Case**: Ventilated case with GPIO access

### Storage Options
- **SD Card**: SanDisk Extreme Pro (minimum Class 10, A2 rating)
- **NVMe**: Official Pi 5 NVMe HAT with quality SSD
- **External**: USB 3.0 SSD for additional storage if needed

---
**Node Status**: Ready for deployment
**Hardware**: Raspberry Pi 5 (8GB recommended)
**Services**: Prometheus, Grafana, Node Exporter, Cluster Health Monitoring
**Resource Usage**: ~2-3GB RAM, moderate CPU impact
**Storage Requirements**: 32GB minimum, 128GB+ recommended