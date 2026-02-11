# OpenClaw Container Guide

OpenClaw is now running in an isolated distrobox container for better dependency management and reproducibility.

## 🚀 Quick Start

### Launch OpenClaw

```bash
# Quick alias (recommended)
claw

# Or use the launcher script
./launch-openclaw.sh

# Or enter the container first
claw-enter
openclaw
```

### Common Commands

```bash
# Run agent
claw agent

# Manage agents
claw-agents

# Run with specific options
claw --help
```

## 📂 Configuration

**Location**: `~/.openclaw` (shared between host and container)

- **Agents**: `~/.openclaw/agents/`
- **Workspace**: `~/.openclaw/workspace/`
- **Credentials**: `~/.openclaw/credentials/`
- **Config**: `~/.openclaw/openclaw.json`

## 🔧 Container Details

| Property | Value |
|----------|-------|
| **Container Name** | `openclaw-dev` |
| **Base Image** | Fedora 43 |
| **OpenClaw Version** | 2026.2.9 |
| **Node.js Version** | 22.22.0 |
| **Configuration** | Shared from `~/.openclaw` |

## 🌐 Remote Access

Since the container runs on your Bazzite machine, you can access it remotely via Tailscale:

```bash
# SSH into Bazzite
ssh yish@bazzite.tail8be4f7.ts.net

# Enter OpenClaw container
claw-enter

# Run OpenClaw
openclaw
```

## 🛠️ Management

```bash
# Enter container shell
claw-enter

# Stop container
distrobox stop openclaw-dev

# Restart container
distrobox stop openclaw-dev
distrobox enter openclaw-dev

# Remove container (keeps ~/.openclaw data)
distrobox rm openclaw-dev

# Recreate container
distrobox create --name openclaw-dev --image fedora:43
distrobox enter openclaw-dev
bash ~/distrobox-configs/openclaw-setup.sh
```

## 📋 Available Aliases

```bash
claw-enter          # Enter OpenClaw container
claw                # Run openclaw command
claw-agent          # Run openclaw agent
claw-agents         # Manage agents
```

## 🔍 Troubleshooting

### Check container status
```bash
distrobox list | grep openclaw
```

### Verify OpenClaw installation
```bash
claw --version
```

### View configuration
```bash
cat ~/.openclaw/openclaw.json | jq .
```

### Check agent list
```bash
ls -la ~/.openclaw/agents/
```

## 🎯 Advantages of Containerization

1. **Isolation**: OpenClaw dependencies don't conflict with other tools
2. **Reproducibility**: Container can be recreated from scratch anytime
3. **Version Control**: Run different OpenClaw versions in different containers
4. **Clean System**: No global npm packages cluttering the host
5. **Shared Config**: Same `~/.openclaw` accessible from host and container

## 🔒 Security Note

The container shares your home directory (`/home/yish`) which includes:
- OpenClaw credentials (`~/.openclaw/credentials/`)
- SSH keys (`~/.ssh/`)
- Git configuration (`~/.gitconfig`)

This is intentional for convenience, but be aware that the container has full access to these files.
