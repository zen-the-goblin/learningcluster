# Pi Cluster SSH Desktop Shortcuts

Desktop shortcuts for quick SSH access to cluster nodes using Terminator terminal.

## Setup Commands

Create desktop shortcuts directory and generate all SSH shortcuts:

```bash
# Create directory for shortcuts
mkdir -p ~/Desktop/cluster-shortcuts

# Git Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/git-node.desktop << 'EOF'
[Desktop Entry]
Name=Git Node SSH
Comment=SSH to Git Node
Exec=terminator -e "ssh user@192.168.1.10"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Security Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/security-node.desktop << 'EOF'
[Desktop Entry]
Name=Security Node SSH
Comment=SSH to Security Node
Exec=terminator -e "ssh user@192.168.1.11"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Storage Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/storage-node.desktop << 'EOF'
[Desktop Entry]
Name=Storage Node SSH
Comment=SSH to Storage Node
Exec=terminator -e "ssh user@192.168.1.12"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Monitoring Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/monitoring-node.desktop << 'EOF'
[Desktop Entry]
Name=Monitoring Node SSH
Comment=SSH to Monitoring Node
Exec=terminator -e "ssh user@192.168.1.13"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Home Automation Node SSH shortcut
cat > ~/Desktop/cluster-shortcuts/home-automation.desktop << 'EOF'
[Desktop Entry]
Name=Home Automation SSH
Comment=SSH to Home Automation Node
Exec=terminator -e "ssh user@192.168.1.14"
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# All Cluster Nodes at once
cat > ~/Desktop/cluster-shortcuts/all-nodes-ssh.desktop << 'EOF'
[Desktop Entry]
Name=All Cluster Nodes
Comment=SSH to All Cluster Nodes
Exec=bash -c 'terminator -e "ssh user@192.168.1.10" & terminator -e "ssh user@192.168.1.11" & terminator -e "ssh user@192.168.1.12" & terminator -e "ssh user@192.168.1.13" & terminator -e "ssh user@192.168.1.14"'
Icon=utilities-terminal
Type=Application
Categories=Network;
EOF

# Make all shortcuts executable
chmod +x ~/Desktop/cluster-shortcuts/*.desktop

# Copy shortcuts directly to desktop (optional)
cp ~/Desktop/cluster-shortcuts/*.desktop ~/Desktop/
```

## Web Interface Shortcuts

Create web interface shortcuts for cluster services:

```bash
# Git web interface
cat > ~/Desktop/cluster-shortcuts/git-web.desktop << 'EOF'
[Desktop Entry]
Name=Git Web Interface
Comment=Open Git Web Interface
Exec=firefox http://192.168.1.10:3000
Icon=web-browser
Type=Application
Categories=Network;
EOF

# Monitoring dashboard
cat > ~/Desktop/cluster-shortcuts/monitoring-web.desktop << 'EOF'
[Desktop Entry]
Name=Monitoring Dashboard
Comment=Open Monitoring Dashboard
Exec=firefox http://192.168.1.13:3000
Icon=web-browser
Type=Application
Categories=Network;
EOF

# Metrics interface
cat > ~/Desktop/cluster-shortcuts/metrics-web.desktop << 'EOF'
[Desktop Entry]
Name=Metrics Interface
Comment=Open Metrics Interface
Exec=firefox http://192.168.1.13:9090
Icon=web-browser
Type=Application
Categories=Network;
EOF

# Home automation dashboard
cat > ~/Desktop/cluster-shortcuts/homeautomation-web.desktop << 'EOF'
[Desktop Entry]
Name=Home Automation
Comment=Open Home Automation Dashboard
Exec=firefox http://192.168.1.14:8123
Icon=web-browser
Type=Application
Categories=Network;
EOF

# Make web shortcuts executable
chmod +x ~/Desktop/cluster-shortcuts/*-web.desktop
```

## Cluster Node Template

| Node | IP Address | Service | SSH Shortcut | Web Access |
|------|------------|---------|-------------|------------|
| Git Node | 192.168.1.10 | Version Control | `git-node.desktop` | http://192.168.1.10:3000 |
| Security Node | 192.168.1.11 | Firewall/Security | `security-node.desktop` | - |
| Storage Node | 192.168.1.12 | NFS/Backups | `storage-node.desktop` | - |
| Monitoring Node | 192.168.1.13 | Metrics/Dashboards | `monitoring-node.desktop` | Dashboard: :3000<br>Metrics: :9090 |
| Home Automation | 192.168.1.14 | IoT/Automation | `home-automation.desktop` | http://192.168.1.14:8123 |

## Customization

**Update IP addresses:** Replace `192.168.1.x` with your network's IP range

**Update usernames:** Replace `user@` with your SSH username

**Different services:** Modify web ports and services to match your cluster setup

## Usage

1. **Individual Node Access**: Double-click any node shortcut to open SSH connection in Terminator
2. **All Nodes**: Use `all-nodes-ssh.desktop` to open connections to all nodes simultaneously
3. **Web Interfaces**: Use web shortcuts to open service dashboards in your browser

## SSH Key Setup (Recommended)

For passwordless SSH access, generate and copy SSH keys:

```bash
# Generate SSH key if you don't have one
ssh-keygen -t ed25519 -C "your_email@domain.com"

# Copy SSH key to all cluster nodes
ssh-copy-id user@192.168.1.10
ssh-copy-id user@192.168.1.11
ssh-copy-id user@192.168.1.12
ssh-copy-id user@192.168.1.13
ssh-copy-id user@192.168.1.14
```

## Prerequisites

**Required software:**
- Terminator terminal: `sudo apt install terminator`
- SSH client (usually pre-installed)
- Firefox or preferred browser

**Network requirements:**
- All cluster nodes must be accessible on your network
- SSH service running on all nodes
- Web services running on designated ports

## Troubleshooting

**Shortcuts don't work:**
- Verify Terminator is installed: `terminator --version`
- Check file permissions: `ls -la ~/Desktop/cluster-shortcuts/`
- Make executable: `chmod +x ~/Desktop/cluster-shortcuts/*.desktop`

**SSH connection refused:**
- Verify node is running: `ping 192.168.1.10`
- Check SSH service: `ssh user@192.168.1.10 'systemctl status ssh'`
- Verify correct username and IP address

**Web interfaces not accessible:**
- Check services are running on target nodes
- Verify firewall settings allow connections on service ports
- Confirm correct ports and URLs

## Terminal and Browser Options

**Alternative terminals:** Replace `terminator` with:
- `gnome-terminal -- ssh user@192.168.1.10`
- `konsole -e ssh user@192.168.1.10`
- `xfce4-terminal -e "ssh user@192.168.1.10"`

**Alternative browsers:** Replace `firefox` with:
- `google-chrome`
- `chromium-browser`
- `microsoft-edge`

## Security Considerations

- Use SSH keys instead of passwords for authentication
- Consider changing default SSH port (22) for additional security
- Implement proper firewall rules on cluster nodes
- Regularly update SSH client and server software
- Use strong passphrases for SSH keys

## License

MIT License - adapt and modify for your cluster setup.