# 🧌 Pi Cluster Infrastructure - Goblin Edition

## What This Actually Is
A distributed computing playground for learning system administration without the corporate buzzword nonsense. Real services, real learning, zero fluff.

**Current Status: Actually working and useful**

## 🎯 Quick Access (The Stuff You Actually Use)

**Development Box**: Your Ubuntu workstation (where the magic happens)
**Git Repos**: http://192.168.1.10:3000 (Gitea - like GitHub but yours)
**Cluster Health**: http://192.168.1.248:3000 (Grafana - pretty graphs of everything)
**Home Control**: http://192.168.1.13:8123 (Home Assistant - because why not)

## 🏗️ The Cluster Layout (What Actually Runs Where)

### Development Node (Your Main Machine)
**What it does**: Everything development-related so you don't have to SSH into tiny Pi's to code
**Tools installed**: Python (3.11.7/3.12.1), Node.js LTS, Docker, CUDA if you have a GPU, all the AI/ML stuff
**Why it exists**: Because developing on a Pi is like coding on a calculator

### Git Node (192.168.1.10)
**What it does**: Hosts your code repositories with Gitea
**Why not GitHub**: Because you own your data and it's faster on your local network
**Backup status**: Actually backs up (unlike that one time you lost everything)

### Security Node (192.168.1.11) 
**What it does**: Firewall, monitoring, logging - the paranoid guardian of your network
**Features**: Intrusion detection that actually works, file integrity monitoring, centralized logs
**Why it matters**: Because "security through obscurity" isn't security

### Storage Node (192.168.1.12)
**What it does**: NFS file sharing across the cluster with automated backups
**Retention**: 7 days (adjust if you're more/less paranoid)
**Access**: Every node can read/write shared storage seamlessly

### Beast Monitoring (192.168.1.248)
**What it does**: Prometheus + Grafana monitoring everything that has an IP
**Why "Beast"**: Because it's probably the most powerful Pi in your cluster
**Monitors**: CPU, memory, disk, network, services, your sanity levels

### Home Automation (192.168.1.13)
**What it does**: Home Assistant + camera monitoring + visual dashboards
**Features**: Camera feeds, system status, probably controls your lights
**Why included**: Because if you're running a cluster, might as well automate everything

## 🚀 Project Templates (The Goblin Magic)

These commands create complete project structures so you can start coding instead of making directories:

```bash
# AI/ML project with all the dependencies
mkaiproject my_ai_experiment
# Creates: data/, notebooks/, src/, tests/, models/, config/
# Includes: numpy, pandas, torch, tensorflow, transformers, jupyter

# Discord bot that actually works
mkdiscordbot chaos_bot
# Creates: cogs/, utils/, data/, main.py, config.py
# Includes: discord.py, proper structure, .env template

# Web project structure
mkwebproject my_webapp
# Creates: frontend/, backend/, database/
# Because separating concerns matters
```

## 📚 Documentation That Actually Helps

### Node Setup Guides
- **[Development Setup](docs/nodes/00-dev-node.md)** - Turn Ubuntu into a development powerhouse
- **[Git Node](docs/nodes/01-git-node.md)** - Self-hosted repos with Gitea
- **[Storage Node](docs/nodes/02-storage-node.md)** - NFS that doesn't make you cry
- **[Security Node](docs/nodes/03-security-node.md)** - Network security that works
- **[Monitoring Node](docs/nodes/04-beast-monitoring.md)** - Prometheus/Grafana setup
- **[Home Assistant](docs/nodes/05-homeautomation-node.md)** - Automate all the things

### Actually Useful References
- **[Cheat Sheets](docs/cheat-sheets/)** - Commands you'll actually use for Git, Docker, Linux, networking
- **[Discord Template](docs/discord-template/)** - Bot structure that doesn't suck
- **[Cluster Overview](docs/cluster-overview.md)** - How everything connects together

## 🔧 Development Environment Status

**Languages Ready**: Python (multiple versions via pyenv), Node.js LTS, C++ build tools
**Databases**: PostgreSQL, Redis, MongoDB (all running and configured)
**Containers**: Docker, Docker Compose, lazydocker for the GUI-preferring
**AI/ML**: PyTorch with CUDA, TensorFlow, Hugging Face transformers, Jupyter
**Discord**: discord.py, proper bot dev environment
**IDE**: VS Code with all the extensions, PyCharm Community as backup

## 🎮 Getting Started (Actually Getting Work Done)

**Set up development environment:**
```bash
# Follow the goblin setup guide first
cat docs/nodes/00-dev-node.md
# Then verify everything works
python ~/verify_setup.py
```

**Create your first project:**
```bash
# AI project
mkaiproject my_first_ai_thing
cd my_first_ai_thing && activate
jupyter lab  # Start coding

# Discord bot
mkdiscordbot my_bot
cd my_bot && activate
# Edit main.py, add your token, go

# Check cluster status
firefox http://192.168.1.248:3000  # Grafana dashboard
```

**Access your services:**
- Push code to: http://192.168.1.10:3000
- Monitor everything: http://192.168.1.248:3000  
- Control your house: http://192.168.1.13:8123

## 🚨 When Things Break (Because They Will)

**Most common issues and actual solutions:**

**"Command not found" errors:**
```bash
source ~/.bashrc  # Fixes 90% of problems
```

**Docker permission errors:**
```bash
# You forgot to logout/login after adding yourself to docker group
# No really, logout and login. GUI logout, not just closing terminal.
```

**Python environment confusion:**
```bash
pyenv versions      # See what's installed
pyenv global 3.11.7 # Set the version you want
python --version    # Verify it worked
```

**Cluster services down:**
- Check Grafana dashboard first (http://192.168.1.248:3000)
- SSH to the relevant node and restart services
- If monitoring is down, you're troubleshooting blind - fix that first

**Development environment broken:**
```bash
# Nuclear option - re-run the setup
cd docs/nodes && cat 00-dev-node.md
# Or just the verification
python ~/verify_setup.py
```

## 🧌 Goblin Philosophy

**This setup prioritizes:**
- **Actually working** over theoretical perfection
- **Learning by doing** over reading documentation
- **Real projects** over toy examples  
- **Your time** over "best practices" that waste hours
- **Practical skills** over resume buzzwords

**What this isn't:**
- Enterprise-grade (though surprisingly robust)
- Optimized for production (but good enough)
- Following every best practice (following the ones that matter)
- Certified by anyone (certified by actually working)

## 📁 Project Structure (Where Everything Lives)

```
pi-cluster/
├── docs/nodes/           # Setup guides for each Pi
├── docs/guides/          # Troubleshooting and maintenance
├── docs/cheat-sheets/    # Commands you'll actually use
├── docs/discord-template/ # Bot development template
├── README.md            # This file (the one you're reading)
└── scripts/             # Automation scripts (coming soon)
```

**Development projects live in:**
- `~/ai_projects/` - AI/ML experiments
- `~/discord_bots/` - Bot development  
- `~/web_projects/` - Web development
- `/mnt/cluster_storage/` - Shared files across all nodes

---

**TL;DR**: Development box + 5 Pi nodes running Git, storage, security, monitoring, and home automation. Everything actually works, documentation doesn't lie, setup scripts don't break. Perfect for learning system administration without the corporate nonsense.