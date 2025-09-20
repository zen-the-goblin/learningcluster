# Raspberry Pi Boot and SSH Troubleshooting Guide

## Problem: Cannot SSH into Raspberry Pi

When you can't connect to your Pi via SSH, work through these troubleshooting steps systematically:

## Step 1: Verify Basic Network Connectivity

### Check if Pi is on the network
```bash
# From your dev machine, scan the network
nmap -sn 192.168.1.0/24 | grep -E "Nmap scan report|MAC Address"

# Or try pinging the expected IP
ping 192.168.1.10  # Replace with your Pi's IP
```

### Check your router/network admin interface
- Log into your router (usually 192.168.1.1 or 192.168.0.1)
- Look for connected devices
- Check DHCP lease table for new devices
- Note if the Pi appears with a different IP than expected

## Step 2: Physical Connection Verification

### Power Supply Check
```bash
# Signs of inadequate power:
# - Random reboots
# - Boot failures
# - USB devices not working
# - Undervoltage warnings in logs
```

**Power Requirements:**
- Pi 5: 5V/5A (25W) USB-C minimum
- Pi 4: 5V/3A (15W) USB-C minimum
- Check power LED status (should be solid, not flickering)

### Network Cable and Switch/Router Ports
- Try different ethernet cable
- Try different port on switch/router
- Check cable connection at both ends
- Verify link lights on network equipment

### SD Card Issues
- Remove and reseat SD card
- Check for physical damage
- Try SD card in another device to verify it's readable

## Step 3: Monitor Connection via HDMI

### Required Hardware
- Mini-HDMI to HDMI cable (Pi 4) or Micro-HDMI to HDMI cable (Pi 5)
- Monitor or TV with HDMI input
- USB keyboard for interaction

### What to Look For
```bash
# Boot sequence should show:
1. Rainbow screen (indicates Pi is starting)
2. Boot messages scrolling
3. Cloud-init configuration (if using Ubuntu Server)
4. Login prompt

# Error indicators:
- No display output → Power or SD card issue
- Kernel panic messages → Corrupted image or hardware
- Stuck at boot → Configuration or hardware problem
- Login prompt but wrong credentials → User setup issue
```

### Ubuntu Server First Boot
```bash
# Default credentials for fresh Ubuntu Server:
Username: ubuntu
Password: ubuntu

# System will force password change on first login
# After password change, you can:
1. Configure networking
2. Set up SSH keys
3. Create additional users
```

## Step 4: SD Card Reflashing Process

### When to Reflash
- Boot failures or corruption errors
- Unable to access with any credentials
- Pi Imager customization didn't work properly
- File system errors during boot

### Reflashing Steps
```bash
1. Download fresh Raspberry Pi OS or Ubuntu Server image
2. Use Raspberry Pi Imager with these settings:
   - Choose your Pi model
   - Select operating system
   - Configure settings (gear icon):
     * Enable SSH (password or key authentication)
     * Set username and password
     * Configure WiFi (if needed)
     * Set locale settings

3. Flash to SD card
4. Safely eject and insert into Pi
5. Power on and wait for boot completion
```

### Pi Imager Configuration Tips
```bash
# For reliable setup:
- Use password authentication initially (easier troubleshooting)
- Set simple, known password for first boot
- Enable SSH in Services tab
- Don't set complex configurations on first attempt
```

## Step 5: Network Configuration Troubleshooting

### Static IP Issues
If you configured static IP but can't connect:

```bash
# Connect via HDMI and keyboard, then check:
ip addr show                    # See current IP configuration
ping 192.168.1.1               # Test gateway connectivity
systemctl status networking    # Check networking service

# Common static IP problems:
- Wrong subnet (192.168.1.x vs 192.168.0.x)
- Incorrect gateway IP
- DNS server issues
- Netplan configuration syntax errors
```

### DHCP vs Static IP Conflicts
```bash
# If DHCP assigned different IP than expected:
1. Check router DHCP range
2. Verify static IP is outside DHCP range
3. Check for IP address conflicts

# Find Pi's actual IP from HDMI console:
hostname -I                     # Show current IP address
```

## Step 6: SSH Service Troubleshooting

### SSH Not Running
```bash
# Connect via HDMI, then check SSH service:
sudo systemctl status ssh
sudo systemctl start ssh        # Start if stopped
sudo systemctl enable ssh       # Enable at boot

# Check SSH configuration:
sudo nano /etc/ssh/sshd_config
# Verify these settings:
# Port 22
# PermitRootLogin no (or yes for troubleshooting)
# PasswordAuthentication yes (for initial setup)
```

### SSH Key Authentication Issues
```bash
# If password auth works but keys don't:
1. Verify key was copied correctly
2. Check ~/.ssh/authorized_keys permissions (600)
3. Check ~/.ssh directory permissions (700)
4. Verify SSH client is using correct key file

# Test from client:
ssh -v user@pi_ip              # Verbose output for debugging
ssh -i ~/.ssh/specific_key user@pi_ip
```

## Step 7: Systematic Troubleshooting Approach

### Layer 1: Physical
- Power supply adequate
- SD card properly seated
- Network cable connected
- Link lights active on network equipment

### Layer 2: Boot Process
- Pi boots successfully (check via HDMI)
- Operating system loads
- Network services start
- SSH daemon running

### Layer 3: Network
- Pi gets IP address (DHCP or static)
- Can ping gateway
- Can ping from other devices on network
- DNS resolution working

### Layer 4: SSH Access
- SSH service running and enabled
- Correct port (usually 22)
- Authentication method working
- Firewall not blocking connections

## Emergency Recovery Procedures

### Complete Recovery Process
```bash
1. Power down Pi safely
2. Remove SD card
3. Flash fresh image with Pi Imager
4. Use simple configuration:
   - Password authentication only
   - Known simple password
   - SSH enabled
5. Boot with HDMI connected
6. Verify network connectivity at console
7. Test SSH from another machine
8. Reconfigure as needed
```

### Backup Prevention
```bash
# After getting Pi working:
1. Create backup image of working SD card
2. Document working configuration
3. Test backup/restore process
4. Keep known-good image for quick recovery
```

## Common Error Patterns and Solutions

### "Connection Refused"
- SSH service not running
- Wrong port number
- Firewall blocking connections

### "No Route to Host"
- Pi not on network
- Wrong IP address
- Network configuration issue

### "Permission Denied (publickey)"
- SSH key not properly installed
- Wrong key file being used
- Key permissions incorrect

### "Connection Timeout"
- Pi not responding
- Network connectivity issues
- Pi may be powered off

### Boot Loops or Kernel Panics
- Corrupted SD card or image
- Inadequate power supply
- Hardware failure

## Prevention Best Practices

### Initial Setup
- Always use adequate power supply
- Use quality SD cards (Class 10 or better)
- Test basic connectivity before complex configuration
- Document working configurations

### Ongoing Maintenance
- Regular system updates
- Monitor system logs for warnings
- Keep backup images of working systems
- Test SSH access after any network changes

### Hardware Considerations
- Use proper cooling for sustained loads
- Secure all connections
- Protect against power fluctuations
- Use quality network cables

---

**Remember**: Work through steps systematically. Don't skip ahead - physical and basic network connectivity must work before SSH troubleshooting is productive.