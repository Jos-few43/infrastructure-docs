# Distrobox Deployment Instructions for Coding Agents

Reproducible container configurations for isolated, properly-configured development environments on Bazzite (immutable Fedora atomic).

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [OpenCode Agent Container](#opencode-agent-container)
- [Common Base Configuration](#common-base-configuration)
- [Container Management](#container-management)
- [Troubleshooting](#troubleshooting)

## Architecture Overview

Each coding agent runs in an isolated distrobox container with:

- **Isolation**: Separate container per agent to prevent dependency conflicts
- **Permissions**: Host home directory mounted for seamless file access
- **Networking**: Full network access for git operations, package management, API calls
- **Reproducibility**: Declarative configuration for consistent deployments
- **Shared Resources**: SSH keys, git config, and credentials shared from host

### Container Naming Convention

`<agent-name>-dev` — e.g., `opencode-dev`, `cursor-dev`, `windsurf-dev`

### Base Images

- **Fedora 43**: Default for most agents (registry.fedoraproject.org/fedora:43)
- **Arch-based**: For AUR package requirements (ghcr.io/ublue-os/bazzite-arch:latest)
- **Ubuntu-based**: For agents requiring Debian/Ubuntu packages (docker.io/library/ubuntu:24.04)

## OpenCode Agent Container

### Quick Deploy

```bash
# Create container
distrobox create \
  --name opencode-dev \
  --image registry.fedoraproject.org/fedora:43 \
  --home /var/home/yish \
  --volume /var/home/yish:/var/home/yish:rw \
  --volume /run/host/etc/localtime:/etc/localtime:ro \
  --additional-flags "--network host" \
  --additional-flags "--security-opt label=disable" \
  --additional-flags "--ipc host" \
  --init

# Enter container
distrobox enter opencode-dev

# Run initial setup (inside container)
bash /var/home/yish/distrobox-configs/opencode-setup.sh
```

### Initial Setup Script

Create `/var/home/yish/distrobox-configs/opencode-setup.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "==> OpenCode Development Container Setup"

# Update system
echo "==> Updating system packages..."
sudo dnf update -y

# Install core dependencies
echo "==> Installing core dependencies..."
sudo dnf install -y \
  git \
  curl \
  wget \
  unzip \
  gcc \
  g++ \
  make \
  openssl-devel \
  libsqlite3x-devel \
  ca-certificates \
  gnupg \
  lsb-release

# Install Node.js 22 LTS
echo "==> Installing Node.js 22..."
curl -fsSL https://rpm.nodesource.com/setup_22.x | sudo bash -
sudo dnf install -y nodejs

# Install Bun
echo "==> Installing Bun..."
if ! command -v bun &> /dev/null; then
  curl -fsSL https://bun.sh/install | bash
  export BUN_INSTALL="$HOME/.bun"
  export PATH="$BUN_INSTALL/bin:$PATH"
fi

# Install pnpm
echo "==> Installing pnpm..."
npm install -g pnpm

# Install global TypeScript tools
echo "==> Installing TypeScript toolchain..."
npm install -g typescript tsx @types/node

# Install Docker CLI (for docker-based workflows)
echo "==> Installing Docker CLI..."
sudo dnf install -y docker-ce-cli || {
  sudo dnf config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
  sudo dnf install -y docker-ce-cli
}

# Configure Git (inherit from host)
echo "==> Configuring Git..."
if [ -f ~/.gitconfig ]; then
  echo "Git config already exists"
else
  git config --global user.name "$(git config --global user.name || echo 'YiSHuA')"
  git config --global user.email "$(git config --global user.email || echo 'yishua@example.com')"
  git config --global init.defaultBranch main
  git config --global pull.rebase false
fi

# Set up SSH agent forwarding
echo "==> Configuring SSH..."
if [ ! -S "$SSH_AUTH_SOCK" ]; then
  eval "$(ssh-agent -s)"
fi

# Install OpenCode CLI (if not present)
echo "==> Checking OpenCode installation..."
if [ ! -d ~/.opencode ]; then
  echo "OpenCode not found in ~/.opencode - install manually if needed"
else
  echo "OpenCode found at ~/.opencode"
fi

# Create workspace directory
mkdir -p ~/workspace
mkdir -p ~/workspace/opencode-projects

# Set default shell to bash if needed
if [ "$SHELL" != "/bin/bash" ] && [ -f /bin/bash ]; then
  echo "==> Setting default shell to bash..."
  sudo chsh -s /bin/bash $USER
fi

echo "==> Setup complete! Reload shell or run: source ~/.bashrc"
```

### Container Specifications

| Setting | Value | Purpose |
|---------|-------|---------|
| **Image** | `fedora:43` | Latest stable Fedora for Node.js ecosystem |
| **Home Mount** | `/var/home/yish:/var/home/yish:rw` | Full home directory access |
| **Network** | `--network host` | Direct host networking for APIs, git, npm |
| **IPC** | `--ipc host` | Shared IPC namespace for process communication |
| **Security** | `--security-opt label=disable` | Disable SELinux labels for seamless file access |
| **Init** | `--init` | Proper PID 1 process for signal handling |

### Environment Variables

Add to `~/.bashrc` inside container:

```bash
# OpenCode Development Environment
export OPENCODE_HOME="$HOME/.opencode"
export PATH="$OPENCODE_HOME/bin:$PATH"

# Bun
export BUN_INSTALL="$HOME/.bun"
export PATH="$BUN_INSTALL/bin:$PATH"

# pnpm
export PNPM_HOME="$HOME/.local/share/pnpm"
export PATH="$PNPM_HOME:$PATH"

# Node.js
export NODE_OPTIONS="--max-old-space-size=4096"

# Editor
export EDITOR="vim"

# Container identifier
export DISTROBOX_CONTAINER="opencode-dev"
```

### Verification Checklist

Run these commands inside the container to verify setup:

```bash
# Check tools
node --version        # Should be v22.x.x
bun --version         # Should be 1.x.x
pnpm --version        # Should be 9.x.x
git --version         # Should be 2.x.x
docker --version      # Should be 27.x.x

# Check OpenCode
ls -la ~/.opencode    # Should show bin/, plugins/, node_modules/
~/.opencode/bin/opencode --version

# Check file permissions
touch ~/test-write && rm ~/test-write  # Should succeed

# Check network
curl -I https://registry.npmjs.org  # Should return 200 OK

# Check git
git config --global user.name  # Should show configured name
ssh -T git@github.com 2>&1 | grep -i "success\|authenticated"  # Test SSH
```

## Common Base Configuration

### Universal Dependencies

All coding agent containers should have:

```bash
# Core build tools
gcc g++ make cmake autoconf automake libtool pkg-config

# Version control
git gh

# Network tools
curl wget aria2 rsync

# Archive tools
unzip zip tar gzip bzip2 xz

# Editors (fallback)
vim nano

# Process management
htop btop tmux

# SSL/TLS
ca-certificates openssl

# Development libraries
libsqlite3x-devel openssl-devel zlib-devel
```

### Shared Volume Mounts

Mount these consistently across all agent containers:

```bash
# Home directory (full access)
--volume /var/home/yish:/var/home/yish:rw

# System time (read-only)
--volume /run/host/etc/localtime:/etc/localtime:ro

# SSH agent socket (if running)
--volume $SSH_AUTH_SOCK:$SSH_AUTH_SOCK:ro

# Docker socket (if needed)
--volume /run/host/run/docker.sock:/var/run/docker.sock:rw
```

### Networking Best Practices

```bash
# Use host networking for simplicity
--additional-flags "--network host"

# OR use bridge with port mapping for isolation
--additional-flags "--network bridge"
--additional-flags "-p 5173:5173"   # Vite dev server
--additional-flags "-p 5003:5003"   # Backend API
--additional-flags "-p 5551:5551"   # OpenCode server
```

### Security Settings

```bash
# Disable SELinux labels for home directory access (Bazzite/Fedora atomic)
--additional-flags "--security-opt label=disable"

# Drop unnecessary capabilities
--additional-flags "--cap-drop ALL"
--additional-flags "--cap-add CHOWN,DAC_OVERRIDE,SETUID,SETGID,NET_BIND_SERVICE"

# Read-only root filesystem (advanced)
# --additional-flags "--read-only"
# --additional-flags "--tmpfs /tmp:rw,noexec,nosuid,size=2g"
```

## Container Management

### Start/Stop Containers

```bash
# Start container
distrobox enter opencode-dev

# Stop container (exit shell)
exit

# Stop all containers
distrobox stop --all

# Remove container
distrobox rm opencode-dev

# Recreate from scratch
distrobox rm -f opencode-dev && \
  distrobox create --name opencode-dev --image fedora:43 [...]
```

### Backup Container Configuration

```bash
# Export container
distrobox-export --container opencode-dev --export-path ~/exports/

# Save container state
podman commit opencode-dev opencode-dev:snapshot-$(date +%Y%m%d)

# List snapshots
podman images | grep opencode-dev
```

### Update Container

```bash
# Inside container
sudo dnf update -y
pnpm update -g
bun upgrade

# Update Node.js
curl -fsSL https://rpm.nodesource.com/setup_22.x | sudo bash -
sudo dnf update -y nodejs
```

### Container Health Check

Create `/var/home/yish/distrobox-configs/opencode-healthcheck.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "==> OpenCode Container Health Check"

# Check if container is running
if ! distrobox list | grep -q "opencode-dev.*Up"; then
  echo "ERROR: opencode-dev container is not running"
  exit 1
fi

# Enter container and run checks
distrobox enter opencode-dev -- bash -c '
  set -e
  echo "  [✓] Container is running"

  command -v node &>/dev/null && echo "  [✓] Node.js installed" || echo "  [✗] Node.js missing"
  command -v bun &>/dev/null && echo "  [✓] Bun installed" || echo "  [✗] Bun missing"
  command -v pnpm &>/dev/null && echo "  [✓] pnpm installed" || echo "  [✗] pnpm missing"
  command -v git &>/dev/null && echo "  [✓] Git installed" || echo "  [✗] Git missing"

  [ -d ~/.opencode ] && echo "  [✓] OpenCode directory exists" || echo "  [✗] OpenCode missing"
  [ -w ~ ] && echo "  [✓] Home directory is writable" || echo "  [✗] Home not writable"

  curl -s -o /dev/null -w "%{http_code}" https://registry.npmjs.org | grep -q 200 && \
    echo "  [✓] Network connectivity OK" || echo "  [✗] Network issues"
'

echo "==> Health check complete"
```

## Troubleshooting

### Permission Issues

```bash
# Fix SELinux labels (if disabled labels don't work)
sudo chcon -R -t container_file_t /var/home/yish

# Fix ownership inside container
sudo chown -R $USER:$USER /var/home/yish/.opencode
```

### Networking Issues

```bash
# Test DNS
nslookup npmjs.org

# Test registry access
curl -v https://registry.npmjs.org

# Check firewall (on host)
sudo firewall-cmd --list-all

# Allow distrobox through firewall
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### Storage Issues

```bash
# Check disk usage
df -h ~

# Clean npm cache
pnpm store prune
npm cache clean --force

# Clean container overlays (on host)
podman system prune -af
```

### Container Won't Start

```bash
# Check container logs
podman logs opencode-dev

# Inspect container
podman inspect opencode-dev

# Force recreate
distrobox rm -f opencode-dev
distrobox create --name opencode-dev [...]
```

### SSH Agent Issues

```bash
# Start SSH agent (on host)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Export SSH_AUTH_SOCK (on host)
echo "export SSH_AUTH_SOCK=$SSH_AUTH_SOCK" >> ~/.bashrc

# Test inside container
distrobox enter opencode-dev -- ssh -T git@github.com
```

## Additional Agent Configurations

### Cursor IDE Container (Placeholder)

```bash
distrobox create \
  --name cursor-dev \
  --image ubuntu:24.04 \
  --home /var/home/yish \
  --volume /var/home/yish:/var/home/yish:rw \
  --additional-flags "--network host" \
  --init
```

### Windsurf Container (Placeholder)

```bash
distrobox create \
  --name windsurf-dev \
  --image fedora:43 \
  --home /var/home/yish \
  --volume /var/home/yish:/var/home/yish:rw \
  --additional-flags "--network host" \
  --init
```

### Claude Code Container (Placeholder)

```bash
distrobox create \
  --name claude-dev \
  --image fedora:43 \
  --home /var/home/yish \
  --volume /var/home/yish:/var/home/yish:rw \
  --additional-flags "--network host" \
  --init
```

## References

- [Distrobox Documentation](https://distrobox.it/)
- [Podman Security](https://docs.podman.io/en/latest/markdown/podman-run.1.html#security-opt-option)
- [Bazzite Documentation](https://universal-blue.org/images/bazzite/)
- [OpenCode Manager README](/var/home/yish/opencode-manager/README.md)
- [CLAUDE.md Project Instructions](/var/home/yish/CLAUDE.md)
