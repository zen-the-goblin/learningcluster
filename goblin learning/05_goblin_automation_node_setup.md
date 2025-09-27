# 🧌 Pi Cluster - Home Automation Node Setup Guide

## What This Node Does
Turns your Pi 5 into a Home Assistant hub with camera monitoring and a touchscreen dashboard. Think smart home brain that actually works.

**Node Details:**
- **Name**: automation (homeassist)
- **Hardware**: Raspberry Pi 5 + 5" touchscreen + active cooling
- **IP**: YOUR_AUTOMATION_IP (static)
- **OS**: Ubuntu Server 22.04 LTS + minimal desktop
- **Purpose**: Home automation, camera feeds, visual dashboard
- **Setup Time**: 60-90 minutes
- **Status**: Ready to configure

## 🎯 Quick Start
If you just want it working, jump to the **TL;DR Setup** section at the bottom.

## Hardware Requirements
- **Pi 5**: 4GB RAM minimum (gets hot under load)
- **Display**: 5" touchscreen mounted via GPIO
- **Cooling**: Active cooling fan (mandatory for continuous operation)
- **Storage**: 128GB SD card (will clone to smaller for production)
- **Network**: Ethernet connection preferred over WiFi
- **Camera**: IP camera compatible with RTSP (optional but recommended)

## Prerequisites Check
- Other cluster nodes running (git, storage, security, monitoring)
- Ubuntu Server 22.04 LTS on SD card
- Static IP configured (YOUR_AUTOMATION_IP)
- SSH access working
- Camera hardware ready to configure (if using)

**Test SSH before starting!**

## 🔧 Base System Setup

### Install Essential Tools
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget git htop -y
```

**This takes forever on Pi 5. Go make coffee.**

## 🐳 Docker Installation (The Right Way)

### Install Docker
```bash
# Add Docker's official repository
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

# Add user to docker group
sudo usermod -aG docker $USER
```

**IMPORTANT: You must logout and login for docker group to take effect!**

**Test Docker works:**
```bash
# After logout/login
docker run hello-world
# Should show success message, not permission errors
```

## 🏠 Home Assistant Setup

### Create Directory Structure
```bash
mkdir -p ~/homeassistant/config
cd ~/homeassistant
```

### Create Docker Compose File
```bash
nano docker-compose.yml
```

**Copy/paste this exact configuration:**
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
      - TZ=YOUR_TIMEZONE  # e.g., America/New_York, Europe/London
```

**What this does:**
- Uses official Home Assistant container
- Mounts config directory for persistence
- Privileged mode for hardware access
- Host networking for device discovery

### Start Home Assistant
```bash
# Start the container
docker compose up -d

# Check it's running
docker ps
# Should show homeassistant container running

# Watch startup logs (takes 2-3 minutes first time)
docker compose logs -f homeassistant
```

**Access Home Assistant:**
Open browser: `http://YOUR_AUTOMATION_IP:8123`

**If this doesn't load:**
- Wait longer (first startup is slow)
- Check docker logs: `docker compose logs homeassistant`
- Verify container is running: `docker ps`

## 📊 Cluster Monitoring Integration

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

### Start Node Exporter
```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

# Test it works
curl http://localhost:9100/metrics | head
```

## 📹 Camera Setup (The Tricky Part)

### Hardware Setup (Example with TP-Link Tapo Camera)
1. Power on your IP camera
2. Download manufacturer's mobile app
3. Connect camera to your WiFi network via app
4. Note camera IP address (check your router)

### Enable Third-Party Access (CRITICAL STEP)
**For TP-Link Tapo cameras:**
1. Camera settings → Advanced → Camera Account
2. Create third-party account (username/password for local access)
3. Enable third-party compatibility in camera settings

**For other brands:**
- Look for "ONVIF", "RTSP", or "third-party" settings
- Enable local access with username/password
- Note the RTSP stream URL format

**Without this step, Home Assistant integration will fail!**

### Home Assistant Camera Integration
1. Access Home Assistant: `http://YOUR_AUTOMATION_IP:8123`
2. Complete initial setup (create account, set timezone, etc.)
3. Settings → Devices & Services → Add Integration
4. Search for "Generic Camera" (more reliable than brand-specific)
5. Use RTSP URL: `rtsp://username:password@CAMERA_IP:554/stream_path`
6. Configure as TCP transport

**Common RTSP URL formats:**
- **TP-Link Tapo**: `rtsp://username:password@CAMERA_IP:554/live/ch00_0`
- **Generic**: `rtsp://username:password@CAMERA_IP:554/stream1`

**Test the camera feed works before continuing.**

## 🖥️ GUI Setup for Touchscreen

### Install Desktop Environment
```bash
# Install minimal desktop
sudo apt install ubuntu-desktop-minimal firefox -y

# Reboot to start desktop
sudo reboot
```

**After reboot, desktop should appear on touchscreen.**

### Configure Kiosk Mode
```bash
# Create autostart directory
mkdir -p ~/.config/autostart

# Create Firefox kiosk launcher
nano ~/.config/autostart/homeassistant-kiosk.desktop
```

**Add this configuration:**
```ini
[Desktop Entry]
Type=Application
Name=Home Assistant Kiosk
Exec=firefox --kiosk http://localhost:8123
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
```

**After reboot, Firefox should auto-open Home Assistant in fullscreen.**

## 🏠 Home Assistant Configuration

### Add System Monitoring
1. Settings → Devices & Services → Add Integration
2. Search for "System Monitor"
3. Enable sensors: CPU usage, memory, disk usage, temperature

### Add Cluster Node Monitoring
1. Settings → Devices & Services → Add Integration  
2. Search for "Ping"
3. Add ping sensors for each cluster node:
   - YOUR_GIT_NODE_IP (git)
   - YOUR_SECURITY_NODE_IP (security)
   - YOUR_STORAGE_NODE_IP (storage)
   - YOUR_MONITORING_NODE_IP (monitoring)

### Create Dashboard
1. Click + tab → Add Dashboard → New dashboard from scratch
2. Title: "Cluster Monitor"
3. Choose "Overview" template
4. Create areas for organization:
   - Camera Feeds
   - System Info  
   - Cluster Status
5. Add entities to areas (not individual cards)

**Area-based organization is much easier than individual cards.**

## 🚨 When Things Go Wrong

### Docker Issues
**"Permission denied" errors:**
- Check user is in docker group: `groups $USER`
- Must logout and login after adding to group
- Test with: `docker run hello-world`

**Container won't start:**
```bash
# Check logs
docker compose logs homeassistant

# Common issues:
# - Port 8123 already in use
# - Insufficient permissions
# - SD card corruption
```

### Camera Integration Problems
**Authentication failures:**
- Create third-party account in camera app FIRST
- Enable third-party compatibility in camera settings
- Wait 30 minutes for security timeout if blocked

**RTSP stream issues:**
- Test URL manually: `ffplay rtsp://user:pass@CAMERA_IP:554/stream_path`
- Try different stream paths: `/stream1`, `/stream2`, `/live/ch00_0`
- Use Generic Camera instead of brand-specific integrations

**Connection timeouts:**
- Check camera IP address: `ping CAMERA_IP`
- Verify camera is on same network
- Check router settings for device isolation

### Home Assistant Dashboard Issues
**"No views in dashboard":**
- Create blank dashboard first
- Add areas before entities
- Use area organization instead of individual cards

**Entities not appearing:**
- Check integration is working: Settings → Devices & Services
- Reload integrations if needed
- Some sensors take time to populate

**System sensors missing:**
- Add System Monitor integration through UI, not YAML
- Restart Home Assistant after adding: Developer Tools → Restart

### Display and GUI Issues
**Desktop won't start:**
- Check installation completed: `dpkg -l | grep ubuntu-desktop`
- Verify display is connected properly
- Check for graphics driver issues in logs

**Kiosk mode not working:**
- Check autostart file exists: `ls ~/.config/autostart/`
- Verify Firefox is installed: `firefox --version`
- Test manually: `firefox --kiosk http://localhost:8123`

**Touchscreen not responsive:**
- Check GPIO connections
- Verify display drivers loaded
- May need display-specific configuration

## 📊 System Monitoring

### Check Container Status
```bash
# List running containers
docker ps

# Check Home Assistant logs
docker compose logs -f homeassistant

# Check system resources
htop
```

### Monitor Performance
```bash
# Check temperature (Pi 5 runs hot)
vcgencmd measure_temp

# Check memory usage
free -h

# Check disk space
df -h
```

### Backup Configuration
```bash
# Backup Home Assistant config
tar -czf ha-backup-$(date +%Y%m%d).tar.gz ~/homeassistant/config

# Copy to storage node (if NFS mounted)
cp ha-backup-*.tar.gz /mnt/storage/backups/
```

## 🔧 Performance Tips

### Pi 5 Specific Optimizations
```bash
# Increase GPU memory split for display
echo "gpu_mem=128" | sudo tee -a /boot/firmware/config.txt

# Enable hardware acceleration
echo "dtoverlay=vc4-kms-v3d" | sudo tee -a /boot/firmware/config.txt

# Reboot after changes
sudo reboot
```

### Container Optimizations
```bash
# Limit Home Assistant memory usage (add to docker-compose.yml)
# deploy:
#   resources:
#     limits:
#       memory: 1G
```

## 🎯 TL;DR Setup (Just Make It Work)

**Copy/paste these commands in order:**

```bash
# 1. System update and Docker
sudo apt update && sudo apt upgrade -y
sudo apt install curl wget git htop ca-certificates curl gnupg -y

# 2. Docker installation
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker $USER

# LOGOUT AND LOGIN HERE
```

**After logout/login:**
```bash
# 3. Home Assistant setup
mkdir -p ~/homeassistant/config
cd ~/homeassistant

tee docker-compose.yml > /dev/null <<EOF
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
      - TZ=America/New_York
EOF

docker compose up -d
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

**Install GUI (optional):**
```bash
sudo apt install ubuntu-desktop-minimal firefox -y
mkdir -p ~/.config/autostart

tee ~/.config/autostart/homeassistant-kiosk.desktop > /dev/null <<EOF
[Desktop Entry]
Type=Application
Name=Home Assistant Kiosk
Exec=firefox --kiosk http://localhost:8123
Hidden=false
NoDisplay=false
X-GNOME-Autostart-enabled=true
EOF

sudo reboot
```

**Access Home Assistant:** `http://YOUR_AUTOMATION_IP:8123`

---

## 🧌 Goblin Notes
- **Pi 5 gets HOT** - active cooling is mandatory, not optional
- **Docker group requires logout/login** - don't skip this step
- **Camera third-party access is mandatory** - set up in app first
- **First Home Assistant startup takes 2-3 minutes** - be patient
- **Generic Camera integration is more reliable** - don't fight with brand-specific ones
- **Area-based dashboards are easier** - resist the urge to manually place every card
- **Desktop installation takes forever** - seriously, go do something else
- **Kiosk mode might not start immediately** - give it a reboot or two
- **Replace YOUR_AUTOMATION_IP with your actual IP** - usually something like 192.168.1.xxx
- **Change TZ to your timezone** - America/New_York, Europe/London, etc.

## Camera Setup Reality
- **Most IP cameras support RTSP** - check manufacturer documentation
- **Third-party access varies by brand** - look for ONVIF or RTSP settings
- **Test RTSP URLs with VLC or ffplay** - verify before adding to Home Assistant
- **Generic Camera integration works with most brands** - start here
- **Camera authentication timeouts are normal** - manufacturers add security delays

## Integration Notes
- **Cluster monitoring**: Connects to monitoring node's Prometheus
- **Backup**: Config can backup to storage node
- **Security**: Can send logs to security node
- **Display**: Auto-starts in kiosk mode for dashboard viewing

---
**Node Status**: Ready for Production  
**Services**: Home Assistant, Node Exporter, Camera Integration, Kiosk Dashboard  
**Access**: `http://YOUR_AUTOMATION_IP:8123`