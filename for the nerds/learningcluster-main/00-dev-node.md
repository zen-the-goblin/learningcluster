# Ubuntu LTS Development Environment Setup

**System tested on: Ubuntu 24.04 LTS**

This guide sets up a complete development environment for AI/ML, Discord bots, web development, and general software development. Commands are designed to be copy-pasteable and work reliably.

## Initial System Setup

Update your system and install essential tools:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git vim nano htop tree unzip software-properties-common apt-transport-https ca-certificates gnupg lsb-release build-essential cmake pkg-config
sudo apt install -y ripgrep fd-find bat ncdu glances
```

Install VS Code:
```bash
sudo snap install code --classic
```

## Docker Installation

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
rm get-docker.sh

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Install lazydocker (better Docker management)
curl https://raw.githubusercontent.com/jesseduffield/lazydocker/master/scripts/install_update_linux.sh | bash

echo "Log out and back in for Docker group changes to take effect"
```

## NVIDIA Drivers & CUDA

```bash
# Install NVIDIA drivers
sudo apt install -y ubuntu-drivers-common
sudo ubuntu-drivers autoinstall

# Reboot after driver installation
sudo reboot
```

After reboot, verify drivers work:
```bash
nvidia-smi
```

Install CUDA toolkit:
```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.0-1_all.deb
sudo dpkg -i cuda-keyring_1.0-1_all.deb
sudo apt update
sudo apt install -y cuda-toolkit

# Add CUDA to PATH
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.zshrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.zshrc
source ~/.zshrc
```

Verify CUDA installation:
```bash
nvcc --version
```

## Python Environment with pyenv

Install pyenv dependencies:
```bash
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python3-openssl
```

Install pyenv:
```bash
curl https://pyenv.run | bash

# Add pyenv to shell
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
source ~/.zshrc
```

Install Python versions:
```bash
pyenv install 3.11.7
pyenv install 3.12.1
pyenv global 3.11.7
```

Verify Python installation:
```bash
python --version
pyenv versions
```

Install Poetry for dependency management:
```bash
curl -sSL https://install.python-poetry.org | python3 -
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Verify Poetry works
poetry --version
```

## Node.js Setup

```bash
# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs

# Fix npm global permissions
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc

# Install useful global packages
npm install -g yarn pnpm @nestjs/cli create-react-app nodemon pm2
```

Verify Node.js installation:
```bash
node --version
npm --version
yarn --version
```

## Database Setup

```bash
# PostgreSQL
sudo apt install -y postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Redis
sudo apt install -y redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server

# MongoDB (optional)
curl -fsSL https://pgp.mongodb.com/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt update
sudo apt install -y mongodb-org
```

## Create Your First AI Project

```bash
mkdir ~/ai_projects && cd ~/ai_projects
poetry new ai_template
cd ai_template

# Configure Poetry environment
poetry env use python3.11

# Add AI/ML dependencies
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

# Install pre-commit hooks
poetry run pre-commit install
```

## VS Code Extensions

```bash
cd ~

# Install essential extensions
code --install-extension ms-python.python
code --install-extension ms-python.black-formatter
code --install-extension ms-python.flake8
code --install-extension ms-python.mypy-type-checker
code --install-extension ms-toolsai.jupyter
code --install-extension ms-vscode.cpptools
code --install-extension ms-vscode-remote.remote-ssh
code --install-extension ms-azuretools.vscode-docker
code --install-extension bradlc.vscode-tailwindcss
code --install-extension esbenp.prettier-vscode
code --install-extension GitHub.copilot
```

Alternative IDEs:
```bash
# PyCharm Community Edition
sudo snap install pycharm-community --classic

# Postman for API testing
sudo snap install postman
```

## Enhanced Shell Configuration

Install modern CLI tools:
```bash
# Install Rust (required for zoxide)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source ~/.cargo/env

# Install zoxide (better cd)
cargo install zoxide --locked

# Install Terminator terminal
sudo apt install -y terminator
```

Add shell enhancements and project helper functions:
```bash
# Add basic aliases and configuration
echo 'export HISTSIZE=10000' >> ~/.zshrc
echo 'export SAVEHIST=20000' >> ~/.zshrc
echo 'setopt HIST_IGNORE_DUPS' >> ~/.zshrc
echo 'setopt HIST_IGNORE_ALL_DUPS' >> ~/.zshrc
echo 'setopt SHARE_HISTORY' >> ~/.zshrc
echo '' >> ~/.zshrc
echo '# Aliases' >> ~/.zshrc
echo 'alias ls="ls --color=auto"' >> ~/.zshrc
echo 'alias ll="ls -alF"' >> ~/.zshrc
echo 'alias la="ls -A"' >> ~/.zshrc
echo 'alias cat="batcat"' >> ~/.zshrc
echo 'alias find="fdfind"' >> ~/.zshrc
echo 'alias top="htop"' >> ~/.zshrc
echo 'alias gpu="nvidia-smi"' >> ~/.zshrc
echo 'alias py="python"' >> ~/.zshrc
echo '' >> ~/.zshrc
echo '# Zoxide integration' >> ~/.zshrc
echo 'eval "$(zoxide init zsh)"' >> ~/.zshrc

# Add project creation functions one by one
echo 'mkaiproject() {
    mkdir -p "$1"/{data,notebooks,src,tests,models,config}
    cd "$1"
    poetry init --no-interaction --name "$1" --dependency numpy --dependency pandas
    mkdir -p src/"$1"
    touch src/"$1"/__init__.py
    touch README.md .env.example
    echo -e "*.pyc\n__pycache__/\n.env\ndata/raw/\nmodels/*.pkl\n.jupyter/\n.ipynb_checkpoints/\ndist/\n*.egg-info/" > .gitignore
    git init
    echo "AI project '\''$1'\'' created with Poetry!"
}' >> ~/.zshrc

echo 'mkdiscordbot() {
    mkdir -p "$1"/{cogs,utils,data}
    cd "$1"
    poetry init --no-interaction --name "$1" --dependency discord.py
    touch main.py config.py .env.example
    echo -e "TOKEN=your_bot_token_here\nGUILD_ID=your_guild_id_here" > .env.example
    echo -e "*.pyc\n__pycache__/\n.env\n*.log" > .gitignore
    git init
    echo "Discord bot project '\''$1'\'' created!"
}' >> ~/.zshrc

echo 'mkwebproject() {
    mkdir -p "$1"/{frontend,backend,database}
    cd "$1"
    echo "Web project '\''$1'\'' created!"
}' >> ~/.zshrc

echo 'activate() {
    if [ -f "pyproject.toml" ]; then
        poetry shell
    elif [ -d ".venv" ]; then
        source .venv/bin/activate
    elif [ -d "venv" ]; then
        source venv/bin/activate
    else
        echo "No virtual environment found"
    fi
}' >> ~/.zshrc

# Reload shell configuration
source ~/.zshrc
```

## Verification Script

Create a script to verify everything works:
```bash
cat > ~/verify_setup.py << 'EOF'
#!/usr/bin/env python3
import sys
import subprocess
import importlib

print(f"Python version: {sys.version}")
print("-" * 60)

# Check Python packages
packages = [
    'torch', 'tensorflow', 'transformers', 'numpy', 'pandas', 
    'matplotlib', 'cv2', 'PIL', 'fastapi', 'sqlalchemy'
]

for package in packages:
    try:
        if package == 'cv2':
            import cv2
            print(f"OpenCV: {cv2.__version__}")
        elif package == 'PIL':
            from PIL import Image
            print(f"Pillow: {Image.__version__}")
        else:
            mod = importlib.import_module(package)
            version = getattr(mod, '__version__', 'installed')
            print(f"{package}: {version}")
    except ImportError:
        print(f"{package}: Not installed")

print("\n" + "-" * 60)

# PyTorch CUDA check
try:
    import torch
    print(f"CUDA available: {torch.cuda.is_available()}")
    if torch.cuda.is_available():
        print(f"CUDA version: {torch.version.cuda}")
        print(f"GPU device: {torch.cuda.get_device_name(0)}")
        print(f"VRAM: {torch.cuda.get_device_properties(0).total_memory / 1024**3:.1f} GB")
        
        # Quick GPU test
        x = torch.randn(1000, 1000).cuda()
        y = torch.mm(x, x)
        print("GPU tensor operations: ✓")
except:
    print("PyTorch CUDA: Not available")

print("\n" + "-" * 60)

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
            print(f"{name}: {version}")
        else:
            print(f"{name}: Error")
    except (FileNotFoundError, subprocess.TimeoutExpired):
        print(f"{name}: Not installed")

print("\nSetup verification complete!")
EOF

chmod +x ~/verify_setup.py
```

## Project Structure Templates

**AI/ML Project:**
```
your_ai_project/
├── data/                 # Raw and processed datasets
├── notebooks/            # Jupyter notebooks
├── src/                 # Source code
├── tests/               # Unit tests
├── models/              # Trained models
├── config/              # Configuration files
├── pyproject.toml       # Dependencies
└── README.md
```

**Discord Bot:**
```
your_discord_bot/
├── cogs/                # Bot commands
├── utils/               # Helper functions
├── data/                # Bot data
├── main.py             # Bot entry point
├── config.py           # Configuration
├── .env.example        # Environment template
└── pyproject.toml
```

## Quick Start Commands

```bash
# Create an AI project
mkaiproject my_ai_project
cd my_ai_project
activate

# Create a Discord bot
mkdiscordbot my_bot
cd my_bot
activate

# Start Jupyter Lab
cd ~/ai_projects/ai_template
activate
jupyter lab

# Verify everything works
python ~/verify_setup.py

# Test Docker
docker run hello-world
```

## Troubleshooting

**NVIDIA issues:**
- Driver problems: `sudo ubuntu-drivers list` then install specific version
- CUDA not found: Check PATH includes `/usr/local/cuda/bin`

**Python/Poetry issues:**
- Poetry not found: `source ~/.zshrc` or add to PATH manually
- Wrong Python version: Use `pyenv global 3.11.7`

**Shell issues:**
- Functions not available: Run `source ~/.zshrc`
- Mixed shell configs: If seeing `command not found: shopt`, clean up `.bashrc`

**Docker issues:**
- Permission denied: Ensure user in docker group, logout/login required
- Commands not found: Logout and login after installation

**Node.js issues:**
- Global package permission errors: Set up `~/.npm-global` directory
- npm not found: Add `~/.npm-global/bin` to PATH

**Development tools:**
- Rust/Cargo not found: Run `source ~/.cargo/env`
- VS Code extensions fail: Install VS Code via snap first