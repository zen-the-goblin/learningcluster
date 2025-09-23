# Pi Cluster - Home Automation Node Setup Guide

## Node Overview
- **Node Name**: homeassist (automation)
- **Hardware**: Raspberry Pi 5 with 5" display and active cooling
- **IP Address**: 192.168.1.13 (static)
- **OS**: Ubuntu Server 22.04 LTS + minimal desktop
- **Purpose**: Home automation hub, camera monitoring, visual dashboard
- **Status**: ✅ Complete

## Hardware Configuration
- **Pi 5**: 4GB RAM, housed in case with active cooling fan
- **Display**: 5" touchscreen directly mounted to Pi via GPIO
- **Storage**: 128GB SD card (will be cloned to 32GB for production)
- **Camera**: TP-Link Tapo TCW61 (IP: 192.168.1.104)
- **Network**: Ethernet connection with static IP

## Prerequisites
- Existing Pi cluster infrastructure (git, storage, security, monitoring nodes)
- Ubuntu Server 22.04 LTS flashed to SD card
- Network configured with static IP
- SSH access configured

## Installation Steps

### Step 1: Base System Setup
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install curl wget git htop -y
```

### Step 2: Docker Installation
```bash
# Add Docker's official GPG key and repository
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Add user to docker group (requires logout/login to take effect)
sudo usermod -aG docker $USER

# Test Docker installation
sudo docker run hello-world
```

### Step 3: Home Assistant Container Setup
```bash
# Create Home Assistant directory structure
mkdir -p ~/homeassistant/config
cd ~/homeassistant

# Create docker-compose.yml
nano docker-compose.yml
```

**Docker Compose Configuration:**
```yaml
version: '3.8'

services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    volumes:
      - ./config:/config
      - /etc/localtime:/etc/localtime:ro
    restart: unless-stopped
    privileged: true
    network_mode: host
    environment:
      - TZ=America/Chicago
```

```bash
# Start Home Assistant
docker compose up -d

# Verify container is running
docker ps

# Monitor startup logs
docker compose logs -f homeassistant
```

### Step 4: Cluster Monitoring Integration
```bash
# Install Node Exporter for cluster monitoring
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-arm64.tar.gz
tar xvfz node_exporter-1.6.1.linux-arm64.tar.gz
sudo mv node_exporter-1.6.1.linux-arm64/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

# Create systemd service
sudo nano /etc/systemd/system/node_exporter.service
```

**Node Exporter Service Configuration:**
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

```bash
# Enable and start node_exporter
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# Update Prometheus configuration to include this node's metrics endpoint
```

### Step 5: Camera Setup
**Hardware Setup:**
1. Power on TP-Link Tapo TCW61 camera
2. Download "Tapo" mobile app
3. Follow app setup to connect camera to WiFi network
4. Note camera IP address
5. **Critical Step**: Enable third-party access in camera settings:
   - Open Tapo app → Camera settings → Advanced → Camera Account
   - Create third-party account (username/password for local access)
   - Enable third-party compatibility in camera settings

**Home Assistant Integration:**
1. Access Home Assistant web interface at `http://[PI_IP]:8123`
2. Complete initial setup (create account, set location, etc.)
3. Add camera integration:
   - Settings → Devices & Services → Add Integration
   - Search for "Generic Camera" (Tapo integration may have authentication issues)
   - Use RTSP URL: `rtsp://username:password@[CAMERA_IP]:554/live/ch00_0`
   - Configure as TCP transport with basic authentication

### Step 6: GUI Setup for Display
```bash
# Install minimal desktop environment
sudo apt install ubuntu-desktop-minimal firefox -y

# Reboot to start desktop environment
sudo reboot
```

**Configure Kiosk Mode:**
```bash
# Create autostart directory
mkdir -p ~/.config/autostart

# Create Firefox kiosk autostart file
nano ~/.config/autostart/homeassistant-kiosk.desktop
```

**Kiosk Configuration:**
```ini
[Desktop Entry]
Type=Application
Name=Home Assistant Kiosk
Exec=firefox --kiosk http://localhost:8123
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
```

### Step 7: Home Assistant Configuration

**Add System Monitoring:**
1. Settings → Devices & Services → Add Integration
2. Search for "System Monitor"
3. Enable sensors: CPU usage, memory, disk usage, temperature

**Add Cluster Monitoring:**
1. Settings → Devices & Services → Add Integration
2. Search for "Ping" 
3. Add ping sensors for each cluster node

**Create Dashboard:**
1. Click + tab → Add Dashboard → New dashboard from scratch
2. Title: "Cluster Monitor"
3. Choose "Overview" template
4. Create areas for organization:
   - Camera Feeds
   - System Info
   - Cluster Status
5. Add entities to appropriate areas rather than individual cards

## Troubleshooting Guide

### Docker Issues
- **Permission denied**: Ensure user is in docker group and log out/in
- **Container won't start**: Check logs with `docker compose logs homeassistant`
- **Port conflicts**: Ensure port 8123 is available

### Camera Integration Problems
- **Authentication failures**: Create third-party account in Tapo app settings
- **Connection timeouts**: Enable third-party compatibility in camera settings
- **RTSP stream issues**: Try different stream paths or use Generic Camera integration
- **Device blocked errors**: Wait 30 minutes for security timeout to clear

### Home Assistant Dashboard Issues
- **"No views in dashboard"**: Create blank dashboard first, then add areas
- **Entities not appearing**: Use area-based organization instead of individual cards
- **System sensors missing**: Add System Monitor integration through UI, not YAML

### Display and GUI Issues
- **Desktop won't start**: Verify Ubuntu desktop installation completed
- **Kiosk mode not working**: Check autostart file permissions and Firefox installation
- **Display resolution problems**: Configure display settings after hardware changes

## Lessons Learned and Improvements

### What Worked Well
- Docker approach for Home Assistant provides isolation and easy management
- Generic Camera integration more reliable than brand-specific integrations
- Area-based dashboard organization simpler than individual card management
- Node exporter integration seamless with existing cluster monitoring

### Pain Points and Solutions
- **Camera authentication**: Always set up third-party access in camera app first
- **Dashboard creation**: Start with blank dashboard and use area organization
- **Integration conflicts**: Use UI-based integrations instead of YAML when possible
- **Display setup**: Install GUI after core functionality is working

### Recommended Changes for Next Time
1. **Set up camera third-party access before Home Assistant integration**
2. **Use Generic Camera integration initially, try brand-specific later**
3. **Create dashboard areas first, then add entities**
4. **Install GUI components after verifying headless functionality**
5. **Test RTSP URLs manually before adding to Home Assistant**

## Network Integration

### Cluster Connectivity
- **Monitoring**: Integrated with existing Prometheus monitoring
- **Backup**: Configuration can be backed up to network storage
- **Logging**: Can be added to centralized logging infrastructure

### Port Usage
- 8123: Home Assistant web interface
- 9100: Node Exporter metrics
- 22: SSH access

## Security Considerations
- Home Assistant runs in privileged container (required for hardware access)
- Camera uses local authentication credentials
- Display auto-starts in kiosk mode (no login required)
- Network access should be limited to trusted subnets

## Expansion Possibilities
- **Additional cameras**: Repeat camera setup process for other TP-Link devices
- **Motion detection**: Home Assistant built-in motion detection and alerts
- **Automation rules**: Trigger actions based on camera events or system status
- **Recording**: Local or network-based video recording
- **Integration**: MQTT, Zigbee, Z-Wave device support

## File Locations
- **Home Assistant config**: `~/homeassistant/config/`
- **Docker compose**: `~/homeassistant/docker-compose.yml`
- **System logs**: `journalctl -u docker`
- **HA logs**: `docker compose logs homeassistant`

---
**Node Status**: ✅ Production Ready  
**Services**: Home Assistant, Node Exporter, Camera Integration, GUI Dashboard  
**Next Steps**: Add additional cameras, expand automation rules