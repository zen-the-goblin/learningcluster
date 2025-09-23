# Pi Cluster - Complete Architecture Overview

## Cluster Summary
A distributed computing environment consisting of Raspberry Pi nodes and monitoring infrastructure, designed for learning system administration, network services, and infrastructure management. The cluster provides Git hosting, network storage, security monitoring, cluster-wide observability, and home automation.

## Infrastructure Overview

### Network Architecture
- **Network**: Private subnet (192.168.x.x/24)
- **Gateway**: Router/gateway device
- **DNS**: Public DNS servers (8.8.8.8, 1.1.1.1)
- **Communication**: SSH (port 22), HTTP/HTTPS (various ports)

### Node Configuration Matrix

| Node | Hardware | Purpose | Primary Services | Status |
|------|----------|---------|------------------|--------|
| git | Pi 5 | Version Control | Git server, repository hosting | ✅ Complete |
| security | Pi 4 | Security & Monitoring | Firewall, logging, intrusion detection | ✅ Complete |
| storage | Pi 5 | Network Storage | NFS file sharing, automated backups | ✅ Complete |
| monitoring | Laptop/Pi | Cluster Monitoring | Prometheus, Grafana, metrics collection | ✅ Complete |
| homeassist | Pi 5 | Home Automation | Home Assistant, camera monitoring | ✅ Complete |

### Additional Hardware Considerations
- **Expansion Capacity**: Additional Pi nodes can be added
- **Total Scalability**: 8+ nodes when fully deployed
- **Display Integration**: Optional touchscreen displays for monitoring

## Detailed Node Specifications

### Git Node
- **Hardware**: Raspberry Pi 5 in case with active cooling
- **Storage**: SD card + optional NVMe SSD via PCIe HAT
- **Services**: Git server (port 3000), Node Exporter (port 9100)
- **Purpose**: Central source code repository hosting
- **Key Features**:
  - Web-based Git interface
  - SSH Git access
  - Repository backup integration
  - SQLite database backend

### Security Node
- **Hardware**: Raspberry Pi 4
- **Storage**: SD card with adequate capacity
- **Services**: UFW firewall, Fail2Ban, AIDE, rsyslog server, Node Exporter
- **Purpose**: Network security, intrusion detection, centralized logging
- **Key Features**:
  - Network traffic monitoring
  - Failed login protection
  - File integrity monitoring
  - Centralized log collection from all nodes
  - Network scanning and alerts

### Storage Node
- **Hardware**: Raspberry Pi 5
- **Storage**: SD card with network storage capability
- **Services**: NFS server, automated backup system, Node Exporter
- **Purpose**: Network file sharing and backup storage
- **Key Features**:
  - NFS shares for cluster-wide file access
  - Automated repository backups
  - Shared configuration storage
  - Cross-platform file sharing capability

### Monitoring Node
- **Hardware**: Laptop or high-spec Pi with adequate RAM
- **Storage**: SSD + external storage array
- **Services**: Prometheus, Grafana, optional media server, Node Exporter
- **Purpose**: Cluster monitoring, metrics collection, visualization
- **Key Features**:
  - Real-time cluster health monitoring
  - Metrics visualization and dashboards
  - Service status tracking
  - Backup verification
  - Optional media server functionality

### Home Automation Node
- **Hardware**: Raspberry Pi 5 with display and active cooling
- **Storage**: SD card (production deployments may use smaller capacity)
- **Services**: Home Assistant, camera monitoring, visual dashboard, Node Exporter
- **Purpose**: Home automation hub, security camera management, visual monitoring station
- **Key Features**:
  - Docker-based Home Assistant deployment
  - IP camera integration (TP-Link Tapo compatibility)
  - Real-time cluster monitoring dashboard
  - Kiosk mode display for 24/7 monitoring
  - System health monitoring (CPU, memory, temperature)

## Service Interconnections

### Data Flow Architecture
```
┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│ Monitoring  │    │   Security   │    │    Git      │
│ Prometheus  │◄───┤   Logging    │◄───┤  Repository │
│   Grafana   │    │   rsyslog    │    │   Server    │
└─────────────┘    └──────────────┘    └─────────────┘
       ▲                   ▲                   │
       │                   │                   ▼
       │            ┌──────────────┐    ┌─────────────┐
       │            │   Storage    │◄───┤   Backups   │
       │            │     NFS      │    │  Automated  │
       │            └──────────────┘    └─────────────┘
       │                   ▲                    
       │                   │                    
       └───────────────────┼────────────────────┘
                           │
                    ┌──────────────┐
                    │ Home Assist  │
                    │   Cameras    │◄─── IP Cameras
                    │ Automation   │     (Various IPs)
                    └──────────────┘
```

### Network Services Map
- **Git Repository**: Git server web interface
- **File Sharing**: NFS services on various ports
- **Monitoring**: Prometheus metrics, Grafana dashboards
- **Metrics Collection**: Node Exporter on all nodes (port 9100)
- **Centralized Logging**: rsyslog UDP collection
- **Home Automation**: Home Assistant dashboard
- **Camera Streams**: Various RTSP/HTTP endpoints from IP cameras

## Security Architecture

### Network Security
- **Firewall**: UFW with default deny policies
- **Access Control**: SSH key-based authentication cluster-wide
- **Intrusion Detection**: Fail2Ban monitoring failed authentication attempts
- **File Integrity**: AIDE monitoring system file changes
- **Network Monitoring**: Regular network scans for unauthorized devices

### Authentication System
- **Primary User**: Standard user account with sudo privileges
- **Service Users**: Dedicated system users for each service
- **SSH Access**: Public key authentication only
- **Admin Contact**: Configured email for alerts

### Data Protection
- **Backup Strategy**: Automated daily backups between nodes
- **Retention Policy**: Configurable backup retention with automated cleanup
- **File Integrity**: AIDE monitoring for unauthorized changes
- **Access Logging**: Centralized authentication logs

## Operational Procedures

### Daily Operations
1. **Health Monitoring**: Automated cluster health checks every 5 minutes
2. **Service Monitoring**: Service status verification every 10 minutes
3. **Backup Verification**: Daily backup integrity checks
4. **Security Scanning**: Network scans every 6 hours
5. **Log Aggregation**: Continuous centralized logging

### Maintenance Schedule
- **Weekly**: Review monitoring dashboards, check system resources
- **Monthly**: System updates, security patch deployment
- **Quarterly**: Full backup testing, disaster recovery validation

### Emergency Procedures
1. **Node Failure**: Health monitoring detects outages, triggers alerts
2. **Service Outage**: Monitoring alerts trigger investigation procedures
3. **Security Incident**: Automated blocking, manual investigation
4. **Data Recovery**: Restore from automated backups

## Current Service Ports

### Standard Port Allocation
- **SSH**: Port 22 on all nodes
- **Git Server**: Port 3000
- **NFS**: Port 2049 and related services
- **Prometheus**: Port 9090
- **Grafana**: Port 3000 (monitoring node)
- **Home Assistant**: Port 8123
- **Node Exporter**: Port 9100 (all nodes)
- **Centralized Logging**: Port 514 UDP

## Performance Characteristics

### Resource Utilization
- **CPU Load**: Typically <20% across cluster under normal operations
- **Memory Usage**: 
  - Pi nodes: ~1-2GB usage (4-8GB total available depending on model)
  - Monitoring node: ~3-4GB usage (varies by hardware)
- **Network Traffic**: Minimal overhead from monitoring (< 1Mbps)
- **Storage Growth**: ~100MB per day from logs and backups

### Scalability Considerations
- **Horizontal Scaling**: Ready for additional Pi nodes
- **Service Expansion**: Network and monitoring infrastructure supports growth
- **Performance Headroom**: Significant unused capacity across all nodes

## Development Environment Integration

### Git Workflow
1. **Development**: Code on local development machine
2. **Version Control**: Push to cluster git server
3. **Backup**: Automated daily backup to storage node
4. **Monitoring**: Track repository activity via Prometheus

### SSH Configuration Template
```
Host git-node
    HostName [GIT_NODE_IP]
    User [USERNAME]

Host security-node
    HostName [SECURITY_NODE_IP]
    User [USERNAME]

Host storage-node
    HostName [STORAGE_NODE_IP]
    User [USERNAME]

Host monitoring-node
    HostName [MONITORING_NODE_IP]
    User [USERNAME]

Host homeassist-node
    HostName [HOMEASSIST_NODE_IP]
    User [USERNAME]
```

## Future Expansion Plans

### Available Expansion Options
- Additional Raspberry Pi nodes for specialized services
- Sufficient network capacity for additional services
- Monitoring infrastructure ready for expansion

### Potential Services
- **Container Orchestration**: Kubernetes cluster or Docker Swarm
- **Database Services**: PostgreSQL cluster, Redis cache
- **CI/CD Pipeline**: GitLab Runner, Jenkins, or similar
- **Development Environment**: Code-server, development containers
- **Additional Security Cameras**: Expand to full multi-camera setup
- **Smart Home Devices**: Lights, switches, sensors via Zigbee/Z-Wave integration

### Infrastructure Enhancements
- **Load Balancing**: HAProxy or similar for high availability
- **Network Segmentation**: VLANs for security isolation
- **External Connectivity**: VPN server for remote access
- **Backup Redundancy**: Off-site backup strategy

## Learning Outcomes

### System Administration Skills
- Linux server deployment and configuration
- Network service setup and management
- Security hardening and monitoring
- Backup and disaster recovery procedures

### Infrastructure as Code
- Version-controlled configuration management
- Automated deployment procedures
- Monitoring and alerting systems
- Documentation and knowledge management

### Technology Stack Proficiency
- **Operating Systems**: Ubuntu Server, systemd service management
- **Networking**: Static IP configuration, NFS, SSH, firewall management
- **Monitoring**: Prometheus, Grafana, system metrics
- **Security**: UFW, Fail2Ban, AIDE, centralized logging
- **Storage**: NFS, automated backup systems
- **Version Control**: Self-hosted Git solutions
- **Containerization**: Docker and Docker Compose
- **Home Automation**: Home Assistant, IP camera integration

## Troubleshooting Guide

### Common Issues and Solutions

#### Network Connectivity
- **Symptom**: Node unreachable via SSH
- **Diagnosis**: `ping <node-ip>`, check network cables, verify IP configuration
- **Resolution**: Restart networking, check static IP settings

#### Service Failures
- **Symptom**: Service not responding
- **Diagnosis**: `systemctl status <service>`, check logs with `journalctl -u <service>`
- **Resolution**: Restart service, check configuration, verify dependencies

#### Storage Issues
- **Symptom**: NFS mounts failing
- **Diagnosis**: Check NFS server status, verify exports, test network connectivity
- **Resolution**: Restart NFS services, remount shares, check firewall rules

#### Monitoring Gaps
- **Symptom**: Missing metrics in monitoring
- **Diagnosis**: Check node_exporter status, verify network connectivity, review config
- **Resolution**: Restart exporters, update monitoring targets, check firewall rules

### Log Locations
- **System Logs**: `/var/log/syslog` on each node
- **Service Logs**: `journalctl -u <service-name>`
- **Cluster Health**: Custom monitoring logs
- **Security Events**: `/var/log/auth.log` and centralized logging
- **Application Logs**: Service-specific locations

## Documentation Structure

### Repository Organization
```
cluster-config/
├── docs/
│   ├── cluster-overview.md (this document)
│   ├── nodes/
│   │   ├── 01-git-node.md
│   │   ├── 02-storage-node.md
│   │   ├── 03-security-node.md
│   │   ├── 04-monitoring-node.md
│   │   └── 05-homeautomation-node.md
│   └── guides/
│       ├── troubleshooting.md
│       └── maintenance.md
├── configs/
│   ├── prometheus/
│   ├── grafana/
│   └── scripts/
└── README.md
```

### Version Control
- **Repository**: Self-hosted Git repository
- **Backup**: Automated daily backups to storage node
- **Access**: SSH key authentication
- **Workflow**: Document updates committed as cluster evolves

---

**Cluster Status**: ✅ Production Ready  
**Total Nodes**: 5 active nodes with expansion capability  
**Services**: Git hosting, NFS storage, security monitoring, cluster observability, home automation  
**Next Phase**: Additional camera deployment, smart home device integration