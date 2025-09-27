# 🧌 Ubuntu LTS Development Environment Setup

## What This Does
Turns your Ubuntu machine into a complete AI/ML development powerhouse. One script run, zero brain cells required for setup decisions.

**System tested on: Ubuntu 24.04 LTS**

## 🎯 Quick Start
If you just want it working without explanation, jump to the **TL;DR Setup** section at the bottom.

## Prerequisites 
- Fresh Ubuntu 24.04 LTS install
- Internet connection
- About 30 minutes (most of it is downloading)
- Patience for the occasional reboot

## 🔧 Essential System Setup

This is the boring but necessary stuff that makes everything else work:

```bash
# Update everything first (this takes forever, start it and go make coffee)
sudo apt update && sudo apt upgrade -y

# Install the stuff that literally everything else depends on
sudo apt install -y curl wget git vim nano htop tree unzip software-properties-common apt-transport-https ca-certificates gnupg lsb-release build-essential cmake pkg-config

# Modern CLI tools that make terminal life bearable
sudo apt install -y ripgrep fd-find bat ncdu glances

# VS Code (the editor that actually works)
sudo snap install code --classic
```

## 🐳 Docker Setup (The Thing That Runs Everything)

```bash
# Install Docker with the official script (lazy but reliable)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
rm get-docker.sh

# Docker Compose (because nobody wants to type docker run commands)
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Lazydocker (Docker GUI for people who hate GUIs)
curl https://raw.githubusercontent.com/jesseduffield/lazydocker/master/scripts/install_update_linux.sh | bash

echo "⚠️  LOG OUT AND BACK IN for Docker to work properly"
```

**Test Docker works:**
```bash
docker run hello-world
# Should print a happy message, not error spam
```

## 🚀 NVIDIA Drivers & CUDA (For the GPU Magic)

### Install Drivers First
```bash
# Let Ubuntu figure out your GPU automatically
sudo apt install -y ubuntu-drivers-common
sudo ubuntu-drivers autoinstall

# This step requires a reboot - no way around it
sudo reboot
```

### After Reboot - Check Drivers Work
```bash
nvidia-smi
# Should show your GPU info, not "command not found"
```

### Install CUDA Toolkit
```bash
# Download the official CUDA keyring
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt update
sudo apt install -y cuda-toolkit

# Add CUDA to your PATH (required for everything to find it)
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc
```

**Test CUDA works:**
```bash
nvcc --version
# Should show CUDA compiler version, not "command not found"
```

## 🐍 Python Environment (The Right Way)

### Install pyenv Dependencies
```bash
# These packages are required for pyenv to compile Python versions
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl
```

### Install pyenv (Python Version Manager)
```bash
# Official pyenv installer
curl https://pyenv.run | bash

# Add pyenv to your shell (critical step people always forget)
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc
```

### Install Python Versions
```bash
# Install the Python versions you actually need
pyenv install 3.11.7  # Stable and well-supported
pyenv install 3.12.1  # Latest features
pyenv global 3.11.7   # Set default to stable version
```

**Test Python works:**
```bash
python --version  # Should show 3.11.7
pyenv versions     # Should list installed versions
```

### Install Poetry (Dependency Management That Doesn't Suck)
```bash
# Official Poetry installer
curl -sSL https://install.python-poetry.org | python3 -
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify it works
poetry --version
```

## 🟢 Node.js Setup

```bash
# Install Node.js LTS (the version that won't break)
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# Fix npm global permissions (prevents sudo requirement)
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Install the global packages you'll actually use
npm install -g yarn pnpm @nestjs/cli create-react-app nodemon pm2
```

**Test Node.js works:**
```bash
node --version  # Should show v18.x or v20.x
npm --version   # Should show npm version
yarn --version  # Should show yarn version
```

## 🗄️ Database Setup

```bash
# PostgreSQL (the database that works)
sudo apt install -y postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Redis (for caching and sessions)
sudo apt install -y redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server

# MongoDB (if you hate yourself and love document databases)
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org
```

## 🤖 Create Your First AI Project

```bash
# Create projects directory and first AI project
mkdir ~/ai_projects && cd ~/ai_projects
poetry new ai_template
cd ai_template

# Use Python 3.11 for this project
poetry env use python3.11

# Add ALL the AI/ML packages you'll need
poetry add numpy pandas matplotlib seaborn plotly
poetry add jupyter jupyterlab ipykernel
poetry add scikit-learn
poetry add torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121
poetry add tensorflow[and-cuda]
poetry add transformers datasets accelerate tokenizers
poetry add opencv-python pillow
poetry add requests beautifulsoup4 httpx aiohttp
poetry add fastapi uvicorn sqlalchemy alembic
poetry add pytest black flake8 mypy ruff pre-commit
poetry add python-dotenv pydantic

# Set up code quality tools
poetry run pre-commit install
```

**This takes forever to download. Go touch grass.**

## 📝 VS Code Extensions (The Ones That Matter)

```bash
# Essential Python development
code --install-extension ms-python.python
code --install-extension ms-python.black-formatter
code --install-extension ms-python.flake8
code --install-extension ms-python.mypy-type-checker
code --install-extension ms-toolsai.jupyter

# Docker and remote development
code --install-extension ms-azuretools.vscode-docker
code --install-extension ms-vscode-remote.remote-ssh

# Web development
code --install-extension bradlc.vscode-tailwindcss
code --install-extension esbenp.prettier-vscode

# C++ (for when Python isn't fast enough)
code --install-extension ms-vscode.cpptools

# AI pair programming (requires GitHub Copilot subscription)
code --install-extension GitHub.copilot
```

**Alternative IDEs:**
```bash
# PyCharm Community (if you prefer heavyweight IDEs)
sudo snap install pycharm-community --classic

# Postman (for API testing and debugging)
sudo snap install postman
```

## 🐚 Enhanced Shell Configuration

### Install Modern CLI Tools
```bash
# Install Rust (required for some modern CLI tools)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source ~/.cargo/env

# Install zoxide (smart cd replacement)
cargo install zoxide --locked

# Better terminal emulator
sudo apt install -y terminator
```

### Add Shell Enhancements
```bash
# Better history settings
echo 'export HISTSIZE=10000' >> ~/.bashrc
echo 'export SAVEHIST=20000' >> ~/.bashrc
echo 'export HISTCONTROL=ignoredups' >> ~/.bashrc

# Useful aliases
echo '' >> ~/.bashrc
echo '# Useful aliases' >> ~/.bashrc
echo 'alias ls="ls --color=auto"' >> ~/.bashrc
echo 'alias ll="ls -alF"' >> ~/.bashrc
echo 'alias la="ls -A"' >> ~/.bashrc
echo 'alias cat="batcat"' >> ~/.bashrc
echo 'alias find="fdfind"' >> ~/.bashrc
echo 'alias top="htop"' >> ~/.bashrc
echo 'alias gpu="nvidia-smi"' >> ~/.bashrc
echo 'alias py="python"' >> ~/.bashrc

# Smart directory navigation
echo '' >> ~/.bashrc
echo '# Zoxide integration (smart cd)' >> ~/.bashrc
echo 'eval "$(zoxide init bash)"' >> ~/.bashrc
```

### Project Creation Functions (The Goblin Magic)

```bash
# AI project creator
echo 'mkaiproject() {
    mkdir -p "$1"/{data,notebooks,src,tests,models,config}
    cd "$1"
    poetry init --no-interaction --name "$1" --dependency numpy --dependency pandas
    mkdir -p src/"$1"
    touch src/"$1"/__init__.py
    touch README.md .env.example
    echo -e "*.pyc\n__pycache__/\n.env\ndata/raw/\nmodels/*.pkl\n.jupyter/\n.ipynb_checkpoints/\ndist/\n*.egg-info/" > .gitignore
    git init
    echo "🤖 AI project \"$1\" created with Poetry!"
}' >> ~/.bashrc

# Discord bot creator
echo 'mkdiscordbot() {
    mkdir -p "$1"/{cogs,utils,data}
    cd "$1"
    poetry init --no-interaction --name "$1" --dependency discord.py
    touch main.py config.py .env.example
    echo -e "TOKEN=your_bot_token_here\nGUILD_ID=your_guild_id_here" > .env.example
    echo -e "*.pyc\n__pycache__/\n.env\n*.log" > .gitignore
    git init
    echo "🤖 Discord bot project \"$1\" created!"
}' >> ~/.bashrc

# Web project creator
echo 'mkwebproject() {
    mkdir -p "$1"/{frontend,backend,database}
    cd "$1"
    echo "🌐 Web project \"$1\" created!"
}' >> ~/.bashrc

# Smart virtual environment activator
echo 'activate() {
    if [ -f "pyproject.toml" ]; then
        poetry shell
    elif [ -d ".venv" ]; then
        source .venv/bin/activate
    elif [ -d "venv" ]; then
        source venv/bin/activate
    else
        echo "❌ No virtual environment found"
    fi
}' >> ~/.bashrc

# Reload shell to apply changes
source ~/.bashrc
```

## 🔍 Verification Script

Create a script to test everything works:

```bash
cat > ~/verify_setup.py << 'EOF'
#!/usr/bin/env python3
import sys
import subprocess
import importlib

print(f"🐍 Python version: {sys.version}")
print("=" * 60)

# Check Python packages
packages = [
    'torch', 'tensorflow', 'transformers', 'numpy', 'pandas', 
    'matplotlib', 'cv2', 'PIL', 'fastapi', 'sqlalchemy'
]

for package in packages:
    try:
        if package == 'cv2':
            import cv2
            print(f"✅ OpenCV: {cv2.__version__}")
        elif package == 'PIL':
            from PIL import Image
            print(f"✅ Pillow: {Image.__version__}")
        else:
            mod = importlib.import_module(package)
            version = getattr(mod, '__version__', 'installed')
            print(f"✅ {package}: {version}")
    except ImportError:
        print(f"❌ {package}: Not installed")

print("\n" + "=" * 60)

# PyTorch CUDA check
try:
    import torch
    print(f"🔥 CUDA available: {torch.cuda.is_available()}")
    if torch.cuda.is_available():
        print(f"🔥 CUDA version: {torch.version.cuda}")
        print(f"🎮 GPU device: {torch.cuda.get_device_name(0)}")
        print(f"💾 VRAM: {torch.cuda.get_device_properties(0).total_memory / 1024**3:.1f} GB")
        
        # Quick GPU test
        x = torch.randn(1000, 1000).cuda()
        y = torch.mm(x, x)
        print("✅ GPU tensor operations: Working!")
except:
    print("❌ PyTorch CUDA: Not available")

print("\n" + "=" * 60)

# System tools
tools = [
    ('docker', ['docker', '--version']),
    ('docker-compose', ['docker-compose', '--version']),
    ('node', ['node', '--version']),
    ('npm', ['npm', '--version']),
    ('poetry', ['poetry', '--version']),
    ('pyenv', ['pyenv', '--version']),
    ('nvcc', ['nvcc', '--version'])
]

for name, cmd in tools:
    try:
        result = subprocess.run(cmd, capture_output=True, text=True, timeout=5)
        if result.returncode == 0:
            version = result.stdout.strip().split('\n')[0]
            print(f"✅ {name}: {version}")
        else:
            print(f"❌ {name}: Error")
    except (FileNotFoundError, subprocess.TimeoutExpired):
        print(f"❌ {name}: Not installed")

print("\n🎉 Setup verification complete!")
EOF

chmod +x ~/verify_setup.py
```

## 📁 Project Structure Templates

**AI/ML Project (what mkaiproject creates):**
```
your_ai_project/
├── data/                 # Raw and processed datasets
├── notebooks/            # Jupyter notebooks for exploration
├── src/                  # Actual Python modules
├── tests/                # Unit tests (write them!)
├── models/               # Trained model files
├── config/               # Configuration files
├── pyproject.toml        # Dependencies and project info
└── README.md             # Documentation
```

**Discord Bot (what mkdiscordbot creates):**
```
your_discord_bot/
├── cogs/                 # Bot command modules
├── utils/                # Helper functions
├── data/                 # Bot persistent data
├── main.py               # Bot entry point
├── config.py             # Configuration handling
├── .env.example          # Environment variables template
└── pyproject.toml        # Dependencies
```

## 🚀 Quick Start Commands

```bash
# Create and start an AI project
mkaiproject my_ai_project
cd my_ai_project
activate

# Create a Discord bot
mkdiscordbot my_bot
cd my_bot
activate

# Start Jupyter Lab for data science work
cd ~/ai_projects/ai_template
activate
jupyter lab

# Verify everything works
python ~/verify_setup.py

# Test Docker
docker run hello-world

# Check GPU status
nvidia-smi
```

## 🚨 When Things Go Wrong

### NVIDIA Issues
**Driver problems:**
```bash
# List available drivers
sudo ubuntu-drivers list
# Install specific version if autoinstall failed
sudo apt install nvidia-driver-XXX
```

**CUDA not found:**
- Check PATH includes `/usr/local/cuda/bin`
- Run `source ~/.bashrc` to reload environment
- Verify with `echo $PATH`

### Python/Poetry Issues
**Poetry not found:**
```bash
# Reload shell configuration
source ~/.bashrc
# Or manually add to PATH
export PATH="$HOME/.local/bin:$PATH"
```

**Wrong Python version:**
```bash
# Check what pyenv has installed
pyenv versions
# Set global version
pyenv global 3.11.7
# Verify
python --version
```

### Shell Configuration Issues
**Functions not available:**
```bash
# Reload configuration
source ~/.bashrc
# Check if functions exist
type mkaiproject
```

**Mixed shell configs:**
- This guide uses bash (Ubuntu default)
- If using zsh, replace `~/.bashrc` with `~/.zshrc` in commands
- Check current shell: `echo $SHELL`

### Docker Issues
**Permission denied:**
```bash
# Ensure user is in docker group
groups $USER
# If docker group missing, add it
sudo usermod -aG docker $USER
# MUST logout and login for changes to take effect
```

**Commands not found:**
- Logout and login after Docker installation
- Check `docker --version` works

### Node.js Issues
**Global package permission errors:**
```bash
# Verify npm prefix is set correctly
npm config get prefix
# Should show /home/YOUR_USERNAME/.npm-global
```

**npm not found:**
```bash
# Add npm global directory to PATH
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Development Tools Issues
**Rust/Cargo not found:**
```bash
# Reload Rust environment
source ~/.cargo/env
# Verify installation
cargo --version
```

**VS Code extensions fail:**
- Install VS Code via snap first: `sudo snap install code --classic`
- Restart VS Code after extension installation

## 🎯 TL;DR Setup (Just Make It Work)

**Copy/paste these command blocks in order. Reboot when told to.**

```bash
# 1. System essentials
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git vim nano htop tree unzip software-properties-common apt-transport-https ca-certificates gnupg lsb-release build-essential cmake pkg-config ripgrep fd-find bat ncdu glances
sudo snap install code --classic
```

```bash
# 2. Docker
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh
sudo usermod -aG docker $USER && rm get-docker.sh
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
curl https://raw.githubusercontent.com/jesseduffield/lazydocker/master/scripts/install_update_linux.sh | bash
```

```bash
# 3. NVIDIA drivers
sudo apt install -y ubuntu-drivers-common
sudo ubuntu-drivers autoinstall
# REBOOT NOW: sudo reboot
```

**After reboot:**
```bash
# 4. CUDA
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb && sudo apt update && sudo apt install -y cuda-toolkit
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
```

```bash
# 5. Python setup
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl
curl https://pyenv.run | bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc
pyenv install 3.11.7 && pyenv global 3.11.7
curl -sSL https://install.python-poetry.org | python3 -
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
```

```bash
# 6. Node.js
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
mkdir ~/.npm-global && npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
npm install -g yarn pnpm @nestjs/cli create-react-app nodemon pm2
```

```bash
# 7. Databases
sudo apt install -y postgresql postgresql-contrib redis-server
sudo systemctl start postgresql redis-server
sudo systemctl enable postgresql redis-server
```

```bash
# 8. Shell enhancements
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source ~/.cargo/env && cargo install zoxide --locked
# Copy the shell configuration from the main guide above
```

**Test everything:**
```bash
source ~/.bashrc
python ~/verify_setup.py
docker run hello-world
nvidia-smi
```

---

## 🧌 Goblin Notes
- **Reboot when told to** - don't skip this step thinking you're clever
- **Source ~/.bashrc frequently** - most "command not found" errors are solved this way
- **Permission errors usually mean logout/login required** - especially for Docker
- **If verification script shows red X's, something broke** - fix before proceeding
- **Don't optimize this setup until it's working** - resist the goblin urge to tinker
- **Keep the verification script** - run it after system updates to catch breakage early
- **This guide uses bash** - adapt paths if you're using zsh or other shells
- **CUDA installation is optional** - skip if you don't have NVIDIA GPU