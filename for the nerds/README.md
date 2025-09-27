# Pi Cluster Configuration

Pi cluster infrastructure configuration for learning system administration and infrastructure management.

## Development Environment
* **Development Node**: ✅ Ubuntu LTS workstation with AI/ML, Discord bot, and web development tools

## Pi Cluster Infrastructure
A distributed computing environment for learning system administration and infrastructure management.

## Quick Start
* [Development Node Setup](docs/nodes/00-dev-node.md) - Ubuntu LTS development environment setup
* [Cluster Overview](docs/cluster-overview.md) - Complete architecture and service map
* [Node Setup Guides](docs/nodes/) - Individual node configuration documentation

## Cluster Status
* **Development Node**: ✅ Ubuntu LTS with Python, Node.js, Docker, CUDA, AI/ML tools
* **Git Node (192.168.1.10)**: ✅ Gitea repository hosting
* **Security Node (192.168.1.11)**: ✅ Firewall, monitoring, logging  
* **Storage Node (192.168.1.12)**: ✅ NFS file sharing and backups
* **Beast Monitoring (192.168.1.248)**: ✅ Prometheus, Grafana, cluster health
* **Home Automation (192.168.1.13)**: ✅ Home Assistant, camera monitoring, visual dashboard

## Access Points
* **Development Tools**: Local workstation with Poetry, pyenv, VS Code
* **Gitea**: http://192.168.1.10:3000
* **Prometheus**: http://192.168.1.248:9090  
* **Grafana**: http://192.168.1.248:3000
* **Home Assistant**: http://192.168.1.13:8123

## Node Documentation
* [00-dev-node.md](docs/nodes/00-dev-node.md) - Development workstation setup (Ubuntu LTS)
* [01-git-node.md](docs/nodes/01-git-node.md) - Git repository hosting with Gitea
* [02-storage-node.md](docs/nodes/02-storage-node.md) - NFS file sharing and automated backups
* [03-security-node.md](docs/nodes/03-security-node.md) - Network security, monitoring, centralized logging
* [04-beast-monitoring.md](docs/nodes/04-beast-monitoring.md) - Cluster monitoring with Prometheus and Grafana
* [05-homeautomation-node.md](docs/nodes/05-homeautomation-node.md) - Home Assistant and camera integration

## Infrastructure Services
* **Development Environment**: AI/ML tools, Discord bot development, web development
* **Version Control**: Self-hosted Git with automated backups
* **File Sharing**: Cluster-wide NFS storage with 7-day retention
* **Security**: Firewall, intrusion detection, file integrity monitoring
* **Monitoring**: Real-time metrics collection and visualization
* **Home Automation**: Camera monitoring, system dashboards, visual control panel

## Development Tools Available
* **Python**: pyenv with 3.11.7/3.12.1, Poetry dependency management
* **Node.js**: LTS with npm, yarn, pnpm
* **AI/ML**: PyTorch (CUDA), TensorFlow, Hugging Face transformers
* **Discord**: discord.py, bot development tools
* **Databases**: PostgreSQL, Redis, MongoDB
* **Containers**: Docker, Docker Compose, lazydocker
* **IDE**: VS Code with extensions, PyCharm Community

## Project Templates
* `mkaiproject project_name` - AI/ML project with Poetry
* `mkdiscordbot bot_name` - Discord bot project structure  
* `mkwebproject web_name` - Web development project

## Cheat Sheets Link
[Cheat Sheets](docs/cheat-sheets/) - Command Cheat Sheets for Git, Docker, Linux, and Networking for the Cluster

## Discord Template
[Discord Template](docs/discord-template/) - Discord Template

## Documentation Structure
```
docs/
├── cluster-overview.md
├── nodes/
│   ├── 00-dev-node.md
│   ├── 01-git-node.md
│   ├── 02-storage-node.md
│   ├── 03-security-node.md
│   ├── 04-beast-monitoring.md
│   └── 05-homeautomation-node.md
├── guides/
│   ├── troubleshooting.md
│   └── maintenance.md
├── cheat-sheets/
└── discord-template/
```