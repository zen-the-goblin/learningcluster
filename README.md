# Raspberry Pi Cluster Setup Guide

A comprehensive guide for building a production-ready Raspberry Pi cluster with Git hosting, network storage, security monitoring, and centralized observability.

## Overview

This repository contains step-by-step instructions for building a distributed computing environment using Raspberry Pi boards. The cluster provides essential infrastructure services including version control, file sharing, security monitoring, and system observability.

## Architecture

The cluster consists of specialized nodes, each serving a specific role:

| Node | Hardware | Purpose | Key Services |
|------|----------|---------|--------------|
| **Git** | Pi 5 | Source code hosting | Gitea, SSH Git access |
| **Storage** | Pi 5 | Network file sharing | NFS server, automated backups |
| **Security** | Pi 4 | Security monitoring | UFW, Fail2Ban, centralized logging |
| **Monitoring** | Laptop/Server | Cluster oversight | Prometheus, Grafana |

## Features

### Infrastructure Services
- **Git Repository Hosting** - Self-hosted Gitea with web interface and SSH access
- **Network File Sharing** - NFS-based shared storage accessible across all nodes
- **Security Monitoring** - Intrusion detection, firewall management, and log aggregation
- **System Monitoring** - Real-time metrics collection and visualization
- **Automated Backups** - Scheduled backups of critical data and configurations

### Learning Outcomes
- Linux server administration and systemd service management
- Network services configuration (NFS, SSH, HTTP)
- Security hardening and monitoring implementation
- Infrastructure as code and documentation practices
- Distributed systems concepts and troubleshooting

## Quick Start

### Prerequisites
- 3-4 Raspberry Pi boards (Pi 4 or Pi 5 recommended)
- MicroSD cards (64GB+ recommended)
- Network switch or router with available ports
- Development machine for administration

### Hardware Requirements
- **Pi 5 nodes**: 25W USB-C power supplies
- **Pi 4 nodes**: 15W USB-C power supplies
- **Network**: Gigabit Ethernet recommended
- **Storage**: Class 10+ SD cards, optional external storage

### Setup Order
Follow the node setup guides in this recommended order:

1. **[Git Node](docs/nodes/git-node-setup.md)** - Establish version control foundation
2. **[Storage Node](docs/nodes/storage-node-setup.md)** - Enable shared storage and backups
3. **[Security Node](docs/nodes/security-node-setup.md)** - Implement monitoring and protection
4. **[Monitoring Setup](docs/nodes/monitoring-setup.md)** - Add cluster observability

## Documentation Structure

```
docs/
├── nodes/
│   ├── git-node-setup.md         # Gitea installation and configuration
│   ├── storage-node-setup.md     # NFS server and backup automation
│   ├── security-node-setup.md    # Security monitoring and logging
│   └── monitoring-setup.md       # Prometheus and Grafana deployment
├── guides/
│   ├── troubleshooting.md         # Common issues and solutions
│   ├── networking-ssh.md          # Network configuration and SSH setup
│   └── maintenance.md             # Ongoing maintenance procedures
└── cheat-sheets/
    ├── docker-commands.md         # Docker reference
    ├── git-commands.md            # Git workflow reference
    ├── linux-admin.md             # Linux administration commands
    └── networking.md              # Network troubleshooting
```

## Network Configuration

### Default IP Scheme
- **Gateway**: 192.168.1.1
- **Git Node**: 192.168.1.10
- **Security Node**: 192.168.1.11
- **Storage Node**: 192.168.1.12
- **Monitoring**: 192.168.1.248

*Adjust IP addresses to match your network configuration*

### Service Ports
- **SSH**: 22 (all nodes)
- **Gitea**: 3000 (git node)
- **NFS**: 2049 (storage node)
- **Syslog**: 514 (security node)
- **Prometheus**: 9090 (monitoring)
- **Grafana**: 3000 (monitoring)
- **Node Exporter**: 9100 (all nodes)

## Security Considerations

### Authentication
- SSH key-based authentication required
- No password authentication on cluster nodes
- Firewall rules restrict access to cluster network

### Network Security
- UFW firewall on all nodes with restrictive defaults
- Fail2Ban intrusion prevention on security node
- Centralized logging for security event correlation
- Regular security scanning and monitoring

### Data Protection
- Automated daily backups of critical services
- File integrity monitoring with AIDE
- Configuration version control
- Secure inter-node communication

## Troubleshooting

### Boot and Connection Issues
Common problems when setting up new nodes:

1. **Power Supply Problems**
   - Pi 5 requires 25W USB-C supply
   - Undervoltage causes boot failures and instability

2. **Network Configuration**
   - Verify static IP settings in netplan
   - Check network cable and switch connections
   - Confirm IP address doesn't conflict with DHCP

3. **SSH Access Problems**
   - Use HDMI connection for initial troubleshooting
   - Verify SSH service is running and enabled
   - Check firewall rules and key authentication

See [troubleshooting guide](docs/guides/troubleshooting.md) for detailed solutions.

## Maintenance

### Regular Tasks
- **Daily**: Monitor cluster health dashboards
- **Weekly**: Review security logs and backup verification
- **Monthly**: System updates and security patches
- **Quarterly**: Full backup testing and documentation updates

### Monitoring
Access cluster monitoring at:
- **Prometheus**: `http://monitoring-ip:9090`
- **Grafana**: `http://monitoring-ip:3000`

## Contributing

### Adding Nodes
The cluster architecture supports easy expansion:

1. Follow base system setup from any node guide
2. Configure static IP in appropriate range
3. Add to monitoring configuration
4. Update documentation

### Improvements
- Submit issues for problems or enhancement requests
- Provide detailed information about your hardware setup
- Include relevant log outputs and error messages
- Test proposed changes in isolated environment

## Hardware Tested

### Verified Configurations
- **Raspberry Pi 5** (4GB/8GB) - Recommended for compute nodes
- **Raspberry Pi 4** (4GB/8GB) - Suitable for monitoring and security
- **Ubuntu Server 24.04.3 LTS** - Primary OS for all testing

### Storage Options
- **SD Cards**: SanDisk Extreme, Samsung EVO Select (64GB+)
- **NVMe**: Pi 5 with PCIe HAT for improved performance
- **External**: USB 3.0 drives for expanded storage capacity

## Cost Estimation

### Minimum Viable Cluster (3 nodes)
- **Pi Boards**: $300-400 (Pi 4 and Pi 5 mix)
- **Power Supplies**: $60-80 (quality USB-C adapters)
- **Storage**: $60-90 (quality SD cards)
- **Networking**: $30-50 (cables, switch if needed)
- **Total**: ~$450-620

### Recommended Configuration (4 nodes + monitoring)
- **Total**: ~$800-1200 (including external storage and quality components)

## Performance Expectations

### Resource Usage
- **Git Node**: 200-500MB RAM, minimal CPU except during operations
- **Storage Node**: 500MB-1GB RAM, network I/O dependent
- **Security Node**: 1-2GB RAM, periodic CPU spikes during scans
- **Monitoring**: 2-4GB RAM, scales with retention and metrics

### Network Requirements
- **Bandwidth**: 100Mbps minimum, Gigabit recommended
- **Latency**: Low-latency network for responsive file operations
- **Reliability**: Stable connections required for clustering services

## License

This project is released under the MIT License. See [LICENSE](LICENSE) for details.

## Acknowledgments

This cluster setup guide is based on hands-on experience building and maintaining Raspberry Pi infrastructure. The documentation captures real-world deployment challenges and solutions.

## Support

### Getting Help
- Review the troubleshooting guide for common issues
- Check existing GitHub issues for similar problems
- Provide detailed system information when reporting issues

### Community
- Share your cluster configurations and improvements
- Document additional use cases and applications
- Help others troubleshoot deployment issues

---

**Status**: Production Ready  
**Last Updated**: September 2025  
**Tested Platform**: Ubuntu Server 24.04.3 LTS on Raspberry Pi 4/5
