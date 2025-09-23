# Pi Cluster Configuration

Raspberry Pi cluster infrastructure configuration for learning system administration and infrastructure management.

## Overview
A distributed computing environment consisting of Raspberry Pi nodes designed for learning system administration, network services, and infrastructure management. The cluster provides Git hosting, network storage, security monitoring, cluster-wide observability, and home automation.

## Quick Start
* [Cluster Overview](docs/cluster-overview.md) - Complete architecture and service map
* [Node Setup Guides](docs/nodes/) - Individual node configuration documentation

## Cluster Architecture
* **Git Node**: Self-hosted Git repository with Gitea
* **Security Node**: Firewall, monitoring, centralized logging  
* **Storage Node**: NFS file sharing and automated backups
* **Monitoring Node**: Prometheus, Grafana, cluster health tracking
* **Home Automation Node**: Home Assistant, camera monitoring, visual dashboard

## Infrastructure Services
* **Version Control**: Self-hosted Git with automated backups
* **File Sharing**: Cluster-wide NFS storage with retention policies
* **Security**: Firewall, intrusion detection, file integrity monitoring
* **Monitoring**: Real-time metrics collection and visualization
* **Home Automation**: Camera monitoring, system dashboards, visual control panel

## Node Documentation
* [01-git-node.md](docs/nodes/01-git-node.md) - Git repository hosting with Gitea
* [02-storage-node.md](docs/nodes/02-storage-node.md) - NFS file sharing and automated backups
* [03-security-node.md](docs/nodes/03-security-node.md) - Network security, monitoring, centralized logging
* [04-monitoring-node.md](docs/nodes/04-monitoring-node.md) - Cluster monitoring with Prometheus and Grafana
* [05-homeautomation-node.md](docs/nodes/05-homeautomation-node.md) - Home Assistant and camera integration

## Technology Stack
* **Operating System**: Ubuntu Server 22.04/24.04 LTS
* **Containerization**: Docker and Docker Compose
* **Monitoring**: Prometheus, Grafana, Node Exporter
* **Security**: UFW firewall, Fail2Ban, AIDE file integrity
* **Storage**: NFS network file system
* **Version Control**: Gitea self-hosted Git
* **Home Automation**: Home Assistant with camera integration

## Hardware Requirements
* Multiple Raspberry Pi 4/5 boards
* MicroSD cards (32GB+ recommended)
* Network switch and ethernet cables
* Optional: External storage for NFS shares
* Optional: Dedicated display for monitoring dashboard

## Learning Outcomes
This cluster setup provides hands-on experience with:
* Linux server administration and configuration
* Network service deployment and management
* Security hardening and monitoring practices
* Infrastructure monitoring and alerting
* Container orchestration with Docker
* Network file systems and backup strategies
* Home automation and IoT device integration

## Getting Started
1. Review the [cluster overview](docs/cluster-overview.md) for architecture understanding
2. Follow individual node setup guides in order
3. Configure monitoring and security according to documentation
4. Expand with additional services as needed

## Contributing
This is a learning project documenting a home lab cluster setup. Feel free to adapt the configurations for your own environment.

## Documentation Structure
```
docs/
├── cluster-overview.md
├── nodes/
│   ├── 01-git-node.md
│   ├── 02-storage-node.md
│   ├── 03-security-node.md
│   ├── 04-monitoring-node.md
│   └── 05-homeautomation-node.md
└── guides/
    ├── troubleshooting.md
    └── maintenance.md
```
