# Raspberry Pi Cluster Troubleshooting Guide

## Problem Categories Overview

This guide covers troubleshooting for modern Pi cluster deployments including SSH connectivity, Docker services, Home Assistant integration, display configuration, and cluster networking.

## Basic SSH and Network Connectivity

### Cannot SSH into Raspberry Pi

When you can't connect to your Pi via SSH, work through these steps systematically:

#### Step 1: Verify Basic Network Connectivity

**Check if Pi is on the network:**
```bash
# From your dev machine, scan the network
nmap -sn 192.168.1.0/24 | grep -E "Nmap scan report|MAC Address"

# Or try pinging the expected IP
ping [PI_IP_ADDRESS]
```

**Check your router/network admin interface:**
- Log into your router (usually 192.168.1.1 or 192.168.0.1)
- Look for connected devices
- Check DHCP lease table for new devices
- Note if the Pi appears with a different IP than expected

#### Step 2: Physical Connection Verification

**Power Supply Check:**
```bash
# Signs of inadequate power:
# - Random reboots
# - Boot failures
# - USB devices not working
# - Undervoltage warnings in logs
```

**Power Requirements:**
- Pi 5: 5V/5A (27W) USB-C minimum for full load
- Pi 4: 5V/3A (15W) USB-C minimum
- Check power LED status (should be solid, not flickering)

**Network Hardware:**
- Try different ethernet cable
- Try different port on switch/router
- Check cable connection at both ends
- Verify link lights on network equipment

#### Step 3: Monitor Boot Process via HDMI

**Required Hardware:**
- Micro-HDMI to HDMI cable (Pi 5) or Mini-HDMI (Pi 4)
- Monitor or TV with HDMI input
- USB keyboard for interaction

**Boot Sequence Indicators:**
```bash
# Normal boot should show:
1. Rainbow screen (indicates Pi hardware is working)
2. Boot messages scrolling
3. Cloud-init configuration (Ubuntu Server)
4. Login prompt

# Error indicators:
- No display output → Power or SD card issue
- Kernel panic messages → Corrupted image or hardware
- Stuck at boot → Configuration or hardware problem
- Login prompt but wrong credentials → User setup issue
```

#### Step 4: Ubuntu Server First Boot Process

```bash
# Default credentials for fresh Ubuntu Server:
Username: ubuntu
Password: ubuntu

# System will force password change on first login
# After password change:
1. Configure networking
2. Set up SSH keys  
3. Create additional users
4. Install necessary packages
```

## Docker and Container Issues

### Docker Permission Problems

**"Permission denied" when running docker commands:**
```bash
# Check if user is in docker group
groups $USER

# Add user to docker group if missing
sudo usermod -aG docker $USER

# Log out and back in for group changes to take effect
exit
# SSH back in and test
docker --version
docker ps
```

### Container Networking Issues

**Containers can't reach network or each other:**
```bash
# Check Docker daemon status
sudo systemctl status docker

# Inspect Docker networks
docker network ls
docker network inspect bridge

# Check if containers are using host networking
docker ps --format "table {{.Names}}\t{{.Ports}}\t{{.Networks}}"

# Test container connectivity
docker exec -it container_name ping 8.8.8.8
```

**Port conflicts between services:**
```bash
# Check what's using specific ports
sudo netstat -tulpn | grep :8123
sudo lsof -i :9090

# Kill processes using conflicting ports
sudo kill -9 [PID]

# Or change port configuration in docker-compose.yml
```

### Docker Compose Problems

**"docker compose" command not found:**
```bash
# Check Docker Compose installation
docker compose version

# If missing, install Docker Compose plugin
sudo apt update
sudo apt install docker-compose-plugin

# Alternative: use docker-compose (dash version)
docker-compose version
```

**Container won't start or crashes immediately:**
```bash
# Check container logs
docker compose logs service_name
docker logs container_name

# Check resource usage
docker stats

# Verify volume mounts and permissions
ls -la ~/homeassistant/config
```

## Home Assistant Specific Issues

### Container Startup Problems

**Home Assistant container fails to start:**
```bash
# Check logs for specific errors
docker compose logs -f homeassistant

# Common issues and solutions:
# - Port 8123 already in use: Change port or kill conflicting process
# - Permission issues: Check config directory ownership
# - Memory issues: Ensure adequate RAM available

# Verify container configuration
docker compose config
```

**Configuration directory issues:**
```bash
# Check permissions on Home Assistant config
ls -la ~/homeassistant/
sudo chown -R $USER:$USER ~/homeassistant/config/

# Verify volume mounting
docker inspect homeassistant | grep -A 10 "Mounts"
```

### Camera Integration Failures

**Cannot connect to TP-Link Tapo cameras:**
```bash
# Common camera connection issues:

1. Third-party access not enabled in Tapo app
   - Open Tapo app → Camera → Advanced → Camera Account
   - Create local account for Home Assistant
   - Enable third-party compatibility

2. Wrong RTSP URL format
   - Try: rtsp://username:password@camera_ip:554/live/ch00_0
   - Try: rtsp://username:password@camera_ip:554/stream1
   - Try: rtsp://username:password@camera_ip/stream1

3. Network connectivity
   - Test: ping camera_ip
   - Verify camera is on same subnet as Pi

4. Authentication timeout/blocking
   - Wait 30 minutes for camera security timeout
   - Reset camera if persistent issues
```

**Camera authentication errors:**
```bash
# Debug camera connectivity
# Test RTSP stream manually:
ffmpeg -i "rtsp://username:password@camera_ip:554/live/ch00_0" -t 5 test.mp4

# Check camera web interface (if available)
curl -I http://camera_ip
```

### Dashboard Creation Issues

**"No views in dashboard" error:**
```bash
# Correct dashboard creation workflow:
1. Create new dashboard from scratch (not using templates)
2. Add areas for organization (Camera Feeds, System Info, etc.)
3. Add entities to areas rather than individual cards
4. Use area-based organization for simpler management

# If dashboard is broken:
1. Delete problematic dashboard
2. Start fresh with blank dashboard
3. Create areas first, then populate with entities
```

**System Monitor integration not appearing:**
```bash
# Add System Monitor through UI, not YAML
# Settings → Devices & Services → Add Integration
# Search for "System Monitor"

# If sensors don't appear in dashboard:
1. Check Developer Tools → States
2. Look for entities starting with "sensor."
3. Manually add entities by typing exact names in dashboard
```

## Display and GUI Configuration Issues

### Desktop Environment Problems

**Ubuntu desktop won't start after installation:**
```bash
# Check if desktop packages installed correctly
dpkg -l | grep ubuntu-desktop-minimal

# Verify display manager status
sudo systemctl status gdm3

# Force start desktop
sudo systemctl start gdm3
sudo systemctl enable gdm3

# Check for display driver issues
dmesg | grep -i display
```

**Display resolution issues:**
```bash
# Check current resolution
xrandr

# For Pi-specific display issues:
# Edit boot configuration
sudo nano /boot/firmware/config.txt

# Add or modify display settings:
# hdmi_force_hotplug=1
# hdmi_group=2
# hdmi_mode=82  # 1920x1080 60Hz
# hdmi_drive=2
```

### Kiosk Mode Configuration

**Firefox kiosk mode not starting automatically:**
```bash
# Check autostart file exists and is correct
ls -la ~/.config/autostart/
cat ~/.config/autostart/homeassistant-kiosk.desktop

# Verify autostart file permissions
chmod +x ~/.config/autostart/homeassistant-kiosk.desktop

# Test manual kiosk launch
firefox --kiosk http://localhost:8123

# Check for conflicting desktop sessions
ps aux | grep firefox
```

**Display timeout or screen blanking:**
```bash
# Disable screen blanking for kiosk mode
# Add to autostart or .bashrc:
xset s off
xset -dpms
xset s noblank

# For persistent setting, edit:
sudo nano /etc/xdg/autostart/disable-screensaver.desktop
```

## Cluster Integration Issues

### Node Exporter Connectivity

**Prometheus can't scrape metrics from nodes:**
```bash
# Check node_exporter status on target node
sudo systemctl status node_exporter

# Test metrics endpoint manually
curl http://localhost:9100/metrics

# From monitoring node, test connectivity
curl http://target_node_ip:9100/metrics

# Check firewall rules
sudo ufw status
```

**Prometheus target discovery issues:**
```bash
# Verify Prometheus configuration
sudo nano /etc/prometheus/prometheus.yml

# Check Prometheus targets status
# Access: http://prometheus_ip:9090
# Go to Status → Targets

# Restart Prometheus after config changes
sudo systemctl restart prometheus
```

### NFS Mount Failures

**Cannot mount NFS shares between nodes:**
```bash
# On NFS server, check exports
sudo exportfs -v
sudo systemctl status nfs-kernel-server

# On client, test NFS connectivity
showmount -e nfs_server_ip

# Manual mount test
sudo mount -t nfs nfs_server_ip:/srv/nfs/shared /mnt/test

# Check for network/firewall issues
sudo ufw allow from cluster_subnet to any port nfs
```

**NFS performance issues:**
```bash
# Check NFS mount options
mount | grep nfs

# Optimize NFS mount options in /etc/fstab
nfs_server:/path /mount/point nfs defaults,rsize=8192,wsize=8192,timeo=14,intr 0 0
```

## Pi 5 Specific Issues

### Power and Performance

**Pi 5 power delivery problems:**
```bash
# Check for undervoltage warnings
dmesg | grep -i voltage
vcgencmd get_throttled

# Monitor power status
vcgencmd measure_volts core
vcgencmd measure_temp

# Ensure adequate power supply (5V/5A minimum)
# Use official Pi 5 power supply for reliability
```

### NVMe HAT Integration

**NVMe SSD not detected:**
```bash
# Check if NVMe is detected
lsblk
dmesg | grep nvme

# Verify PCIe configuration
lspci | grep -i nvme

# Enable PCIe in boot config if needed
sudo nano /boot/firmware/config.txt
# Add: dtparam=pciex1

# Check NVMe health
sudo smartctl -a /dev/nvme0n1
```

### Cooling and Thermal Management

**Pi 5 overheating issues:**
```bash
# Monitor temperature continuously
watch vcgencmd measure_temp

# Check thermal throttling
vcgencmd get_throttled
# 0x0 = no issues
# 0x50000 = currently throttled

# Verify cooling solution:
# - Active cooling fan operational
# - Thermal pads properly installed
# - Case ventilation adequate
```

## Advanced Troubleshooting Techniques

### System Resource Analysis

**High CPU or memory usage:**
```bash
# Monitor system resources
htop
iotop
iostat 5

# Check Docker container resources
docker stats

# Analyze system load
uptime
cat /proc/loadavg

# Check for memory leaks
sudo dmesg | grep -i "out of memory"
```

### Network Diagnostics

**Comprehensive network testing:**
```bash
# Test basic connectivity
ping -c 4 gateway_ip
ping -c 4 8.8.8.8

# DNS resolution test
nslookup google.com
dig google.com

# Port connectivity testing
telnet target_ip 22
nc -zv target_ip 22

# Network interface analysis
ip addr show
ip route show
```

### Log Analysis

**Systematic log investigation:**
```bash
# System logs
sudo journalctl -f
sudo journalctl -u service_name -f

# Container logs
docker logs container_name --follow
docker compose logs -f

# System messages
tail -f /var/log/syslog
dmesg | tail -20

# Authentication logs
sudo tail -f /var/log/auth.log
```

## Emergency Recovery Procedures

### Complete System Recovery

**When all else fails:**
```bash
1. Power down Pi safely
2. Remove SD card and backup if possible
3. Flash fresh OS image with Pi Imager
4. Use minimal configuration:
   - Enable SSH with password authentication
   - Set known username/password
   - Configure basic network settings
5. Boot with HDMI connected to monitor progress
6. Verify basic connectivity before advanced configuration
7. Restore from backups incrementally
```

### Backup and Prevention

**Create recovery images:**
```bash
# Create backup of working SD card
sudo dd if=/dev/sdX of=pi_backup.img bs=4M status=progress

# Compress backup image
gzip pi_backup.img

# Restore from backup
sudo dd if=pi_backup.img.gz of=/dev/sdX bs=4M status=progress
```

## Troubleshooting Workflow

### Systematic Approach

**Layer-by-layer diagnosis:**
1. **Physical**: Power, cables, hardware connections
2. **Boot**: OS loads, services start, login available
3. **Network**: IP assignment, connectivity, DNS resolution
4. **Services**: Docker running, containers healthy, ports accessible
5. **Application**: Home Assistant, cameras, integrations working

**Documentation practices:**
- Record working configurations
- Document changes that break functionality
- Keep command history of troubleshooting steps
- Maintain backup images of stable systems

---

**Key Principle**: Work systematically through layers. Don't skip steps - each layer must function before the next can be properly diagnosed.
