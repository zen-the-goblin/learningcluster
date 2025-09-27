# 🧌 Pi Cluster SSH Desktop Shortcuts

Desktop shortcuts for quick SSH access to cluster nodes using Terminator terminal. Because clicking icons is faster than typing commands when you're managing multiple nodes.

## What This Does
Creates clickable desktop shortcuts that open SSH connections to your cluster nodes in terminal windows. No more typing `ssh username@192.168.1.whatever` every time you need to check something.

## 🎯 Quick Setup

Replace YOUR_USERNAME with your actual SSH username and YOUR_NODE_IPs with your actual node IP addresses:

```bash
# Create directory for shortcuts
mkdir -p ~/Desktop/cluster-shortcuts

# Git Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/git-node.desktop << 'EOF'
[Desktop Entry]
Name=Git Node SSH
Comment=SSH to Git Node
Exec=terminator -e "ssh YOUR_USERNAME@YOUR_GIT_NODE_IP"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Security Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/security-node.desktop << 'EOF'
[Desktop Entry]
Name=Security Node SSH
Comment=SSH to Security Node
Exec=terminator -e "ssh YOUR_USERNAME@YOUR_SECURITY_NODE_IP"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Storage Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/storage-node.desktop << 'EOF'
[Desktop Entry]
Name=Storage Node SSH
Comment=SSH to Storage Node
Exec=terminator -e "ssh YOUR_USERNAME@YOUR_STORAGE_NODE_IP"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Monitoring Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/monitoring-node.desktop << 'EOF'
[Desktop Entry]
Name=Monitoring Node SSH
Comment=SSH to Monitoring Node
Exec=terminator -e "ssh YOUR_USERNAME@YOUR_MONITORING_NODE_IP"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Automation Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/automation-node.desktop << 'EOF'
[Desktop Entry]
Name=Automation Node SSH
Comment=SSH to Automation Node
Exec=terminator -e "ssh YOUR_USERNAME@YOUR_AUTOMATION_NODE_IP"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# All Cluster Nodes at once (goblin mode)
cat > ~/Desktop/cluster-shortcuts/all-nodes-ssh.desktop << 'EOF'
[Desktop Entry]
Name=All Cluster Nodes
Comment=SSH to All Cluster Nodes (Warning: Opens 5 windows)
Exec=bash -c 'terminator -e "ssh YOUR_USERNAME@YOUR_GIT_NODE_IP" & terminator -e "ssh YOUR_USERNAME@YOUR_SECURITY_NODE_IP" & terminator -e "ssh YOUR_USERNAME@YOUR_STORAGE_NODE_IP" & terminator -e "ssh YOUR_USERNAME@YOUR_MONITORING_NODE_IP" & terminator -e "ssh YOUR_USERNAME@YOUR_AUTOMATION_NODE_IP"'
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Make all shortcuts executable
chmod +x ~/Desktop/cluster-shortcuts/*.desktop

# Copy shortcuts directly to desktop (optional)
cp ~/Desktop/cluster-shortcuts/*.desktop ~/Desktop/
```

## 🌐 Web Interface Shortcuts

Create web shortcuts for cluster services:

```bash
# Git web interface
cat > ~/Desktop/cluster-shortcuts/git-web.desktop << 'EOF'
[Desktop Entry]
Name=Git Web Interface
Comment=Open Gitea Web Interface
Exec=firefox http://YOUR_GIT_NODE_IP:3000
Icon=web-browser
Type=Application
Categories=Network;
EOF

# Monitoring dashboard (Grafana)
cat > ~/Desktop/cluster-shortcuts/grafana-web.desktop << 'EOF'
[Desktop Entry]
Name=Grafana Dashboard
Comment=Open Grafana Monitoring Dashboard
Exec=firefox http://YOUR_MONITORING_NODE_IP:3000
Icon=web-browser
Type=Application
Categories=Network;
EOF

# Prometheus metrics interface
cat > ~/Desktop/cluster-shortcuts/prometheus-web.desktop << 'EOF'
[Desktop Entry]
Name=Prometheus Metrics
Comment=Open Prometheus Metrics Interface
Exec=firefox http://YOUR_MONITORING_NODE_IP:9090
Icon=web-browser
Type=Application
Categories=Network;
EOF

# Home automation dashboard
cat > ~/Desktop/cluster-shortcuts/homeassistant-web.desktop << 'EOF'
[Desktop Entry]
Name=Home Assistant
Comment=Open Home Assistant Dashboard
Exec=firefox http://YOUR_AUTOMATION_NODE_IP:8123
Icon=web-browser
Type=Application
Categories=Network;
EOF

# Make web shortcuts executable
chmod +x ~/Desktop/cluster-shortcuts/*-web.desktop
```

## 📋 Cluster Reference Template

| Node | Example IP | Service | SSH Shortcut | Web Access |
|------|------------|---------|-------------|------------|
| Git Node | 192.168.1.10 | Version Control | `git-node.desktop` | YOUR_GIT_NODE_IP:3000 |
| Security Node | 192.168.1.11 | Firewall/Security | `security-node.desktop` | No web interface |
| Storage Node | 192.168.1.12 | NFS/Backups | `storage-node.desktop` | No web interface |
| Monitoring Node | 192.168.1.13 | Metrics/Dashboards | `monitoring-node.desktop` | Grafana: :3000<br>Prometheus: :9090 |
| Automation Node | 192.168.1.14 | Home Automation | `automation-node.desktop` | Home Assistant: :8123 |

## ⚙️ Customization Examples

### For 192.168.1.x Network
```bash
# Replace YOUR_GIT_NODE_IP with 192.168.1.10
# Replace YOUR_SECURITY_NODE_IP with 192.168.1.11
# Replace YOUR_STORAGE_NODE_IP with 192.168.1.12
# etc.
```

### For 192.168.0.x Network  
```bash
# Replace YOUR_GIT_NODE_IP with 192.168.0.10
# Replace YOUR_SECURITY_NODE_IP with 192.168.0.11
# Replace YOUR_STORAGE_NODE_IP with 192.168.0.12
# etc.
```

### For Different Username
```bash
# Replace YOUR_USERNAME with your actual SSH username
# Example: zen, pi, admin, your-name, etc.
```

## 🔑 SSH Key Setup (Highly Recommended)

For passwordless SSH access, set up SSH keys:

```bash
# Generate SSH key if you don't have one
ssh-keygen -t ed25519 -C "your_email@domain.com"

# Copy SSH key to all cluster nodes
ssh-copy-id YOUR_USERNAME@YOUR_GIT_NODE_IP
ssh-copy-id YOUR_USERNAME@YOUR_SECURITY_NODE_IP
ssh-copy-id YOUR_USERNAME@YOUR_STORAGE_NODE_IP
ssh-copy-id YOUR_USERNAME@YOUR_MONITORING_NODE_IP
ssh-copy-id YOUR_USERNAME@YOUR_AUTOMATION_NODE_IP
```

**Without SSH keys, you'll have to type passwords every time you click a shortcut.**

## 📦 Prerequisites

**Required software:**
```bash
# Install Terminator terminal
sudo apt install terminator

# Install Firefox (usually pre-installed)
sudo apt install firefox

# SSH client is usually pre-installed on Linux
```

**Network requirements:**
- All cluster nodes must be accessible on your network
- SSH service running on all nodes (port 22)
- Web services running on designated ports

## 🚨 When Things Go Wrong

### Shortcuts Don't Work
**"Nothing happens when I click":**
```bash
# Check if Terminator is installed
terminator --version

# Check file permissions
ls -la ~/Desktop/cluster-shortcuts/

# Make executable if needed
chmod +x ~/Desktop/cluster-shortcuts/*.desktop
```

### SSH Connection Refused
**"Connection refused" errors:**
```bash
# Test if node is reachable
ping YOUR_NODE_IP

# Test SSH manually
ssh YOUR_USERNAME@YOUR_NODE_IP

# Check SSH service on target node
ssh YOUR_USERNAME@YOUR_NODE_IP 'systemctl status ssh'
```

### Web Interfaces Don't Load
**"This site can't be reached":**
1. Check if services are running on target nodes
2. Verify correct IP addresses and ports
3. Check firewall settings on target nodes
4. Try accessing from terminal first: `curl http://YOUR_NODE_IP:PORT`

### Too Many Windows Opening
**"All nodes shortcut opened 20 windows":**
- This happens if you click it multiple times
- Close extra windows manually
- Wait a few seconds between clicks
- Consider using individual node shortcuts instead

## 🔧 Terminal and Browser Alternatives

### Different Terminals
Replace `terminator` with:
```bash
# GNOME Terminal
Exec=gnome-terminal -- ssh YOUR_USERNAME@YOUR_NODE_IP

# Konsole (KDE)
Exec=konsole -e ssh YOUR_USERNAME@YOUR_NODE_IP

# XFCE Terminal
Exec=xfce4-terminal -e "ssh YOUR_USERNAME@YOUR_NODE_IP"
```

### Different Browsers
Replace `firefox` with:
```bash
# Google Chrome
Exec=google-chrome http://YOUR_NODE_IP:PORT

# Chromium
Exec=chromium-browser http://YOUR_NODE_IP:PORT

# Microsoft Edge
Exec=microsoft-edge http://YOUR_NODE_IP:PORT
```

## 🔒 Security Notes

**SSH Security:**
- Use SSH keys instead of passwords
- Consider changing default SSH port (22) if needed
- Keep SSH client software updated

**Network Security:**
- These shortcuts work only on your local network
- Firewall rules should allow SSH and web traffic
- Don't expose cluster services to the internet without proper security

## 🧌 Goblin Notes

- **SSH keys are mandatory** - typing passwords every time defeats the purpose
- **Terminator splits better** - use it over default terminal for cluster work
- **All nodes shortcut is chaos** - only use when you need to check everything at once
- **Replace YOUR_* placeholders** - the shortcuts won't work with placeholder text
- **Test manually first** - make sure SSH works before creating shortcuts
- **Don't click all-nodes repeatedly** - you'll end up with 47 terminal windows
- **Bookmark web interfaces** - browser bookmarks might be easier than desktop shortcuts
- **Check your network range** - 192.168.1.x is common but not universal

## Usage Tips

**Daily Operations:**
1. **Individual Node Access**: Double-click node shortcut to open SSH connection
2. **System Monitoring**: Use web shortcuts to check dashboards
3. **Cluster-wide Commands**: Use all-nodes shortcut when updating all systems

**Workflow Examples:**
- Check cluster health: Click monitoring web shortcut
- Update all nodes: Click all-nodes SSH shortcut, run updates
- Debug service: Click individual node SSH shortcut
- View repositories: Click git web shortcut

**Power User Tips:**
- Put shortcuts in a dedicated folder for organization
- Create additional shortcuts for common commands
- Use SSH config file for even faster connections
- Consider tmux/screen for persistent sessions

---

**Setup Time**: 5 minutes  
**Convenience Level**: High  
**Goblin Approval**: ✅ Practical and time-saving