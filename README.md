# Infrastructure Documentation

Complete documentation for Bazzite system configuration, containerized development environments, and remote access setup.

## 📚 Documentation Index

### Core Guides

- **[DISTROBOX-DEPLOY.md](./DISTROBOX-DEPLOY.md)** - Complete distrobox deployment guide
  - Architecture overview
  - Container specifications
  - Network and security configuration
  - Troubleshooting

- **[SYSTEM-OVERVIEW.md](./SYSTEM-OVERVIEW.md)** - System architecture and services
  - Hardware configuration
  - Network topology (Tailscale VPN)
  - Running services and containers
  - Backup and maintenance procedures

- **[REMOTE-ACCESS.md](./REMOTE-ACCESS.md)** - Remote access setup
  - Tailscale configuration
  - SSH access from anywhere
  - Web interface access (OpenCode, etc.)
  - Mobile access via Claude Projects

### Container-Specific Guides

- **[openclaw-container-guide.md](./openclaw-container-guide.md)** - OpenClaw AI agent setup
- **[opencode-container-guide.md](./opencode-container-guide.md)** - OpenCode development environment (TODO)

### Quick References

- **[QUICK-START.md](./QUICK-START.md)** - Get up and running fast
- **[TROUBLESHOOTING.md](./TROUBLESHOOTING.md)** - Common issues and solutions
- **[COMMANDS-CHEATSHEET.md](./COMMANDS-CHEATSHEET.md)** - Frequently used commands

## 🖥️ System Overview

**Machine**: Bazzite Desktop (Fedora Atomic/Silverblue-based)
**Primary Storage**: NVMe SSD (encrypted)
**Backup Storage**: USB SSD
**Network**: Tailscale VPN (bazzite.tail8be4f7.ts.net)

### Running Containers

| Container | Purpose | Status |
|-----------|---------|--------|
| `opencode-dev` | OpenCode AI development | ✅ Running |
| `openclaw-dev` | OpenClaw AI agent runtime | ✅ Running |
| `fedora-tools` | General development tools | ✅ Running |
| `bazzite-arch` | Arch Linux environment | ✅ Running |
| `cursor-dev` | Cursor IDE environment | 📋 Template |
| `claude-dev` | Claude Code environment | 📋 Template |

## 🌐 Remote Access

### SSH Access (Anywhere)
```bash
ssh yish@bazzite.tail8be4f7.ts.net
```

### Web Interfaces
- OpenCode Frontend: `http://bazzite.tail8be4f7.ts.net:5173`
- OpenCode Backend: `http://bazzite.tail8be4f7.ts.net:5003`

### Mobile Access
Use Claude Projects to store and access documentation from Claude mobile app.

## 🔧 Quick Commands

```bash
# List all containers
distrobox list

# Enter OpenCode container
distrobox enter opencode-dev

# Enter OpenClaw container
distrobox enter openclaw-dev

# Check Tailscale status
tailscale status

# System status
systemctl status tailscaled
```

## 📦 Related Repositories

- [`distrobox-configs`](../distrobox-configs/) - Container configuration scripts
- [`opencode-manager`](../opencode-manager/) - OpenCode Manager project
- [`opencode-antigravity-multi-auth`](../opencode-antigravity-multi-auth/) - OpenCode plugin

## 🎯 Use Cases

This infrastructure supports:

1. **AI Agent Development** - OpenCode and OpenClaw in isolated containers
2. **Remote Development** - Access from anywhere via Tailscale VPN
3. **Mobile Access** - Documentation synced to Claude mobile via Projects
4. **Reproducible Environments** - Git-tracked container configurations
5. **System Immutability** - Bazzite atomic updates, containers for dev tools

## 🔐 Security

- Tailscale VPN for encrypted remote access
- No public ports exposed
- Container isolation via distrobox
- Home directory encryption (LUKS)
- SSH key-based authentication

## 📱 Accessing Documentation on Mobile

### Option 1: Claude Projects (Recommended)

1. **Create a Claude Project** on claude.ai
2. **Upload documentation files** to the project
3. **Access from mobile** - Project syncs to Claude mobile app
4. **Ask questions** - Claude can reference your docs

### Option 2: Git + Mobile Browser

1. **Push repos to GitHub**
2. **View on mobile** via GitHub mobile app or browser
3. **Clone locally** when needed

### Option 3: Syncthing

1. **Set up Syncthing** between Bazzite and mobile
2. **Sync ~/infrastructure-docs**
3. **Read with markdown viewer** on mobile

## 🚀 Getting Started

1. **Read [SYSTEM-OVERVIEW.md](./SYSTEM-OVERVIEW.md)** to understand the architecture
2. **Set up containers** following [DISTROBOX-DEPLOY.md](./DISTROBOX-DEPLOY.md)
3. **Configure remote access** per [REMOTE-ACCESS.md](./REMOTE-ACCESS.md)
4. **Sync to Claude Projects** for mobile access (see below)

## 📝 Maintenance

- **Weekly**: Update containers (`distrobox enter <container> -- sudo dnf update`)
- **Monthly**: System update (`rpm-ostree upgrade --reboot`)
- **Quarterly**: Review and clean old snapshots
- **As needed**: Sync docs to Claude Projects

## 🆘 Support

1. Check [TROUBLESHOOTING.md](./TROUBLESHOOTING.md)
2. Review container logs: `podman logs <container-name>`
3. Test connectivity: `tailscale netcheck`
4. Ask Claude via Projects (reference this documentation)

---

**Last Updated**: 2026-02-10
**System**: Bazzite (Fedora 43)
**Tailscale Network**: tail8be4f7.ts.net
