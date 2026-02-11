# System Overview

Complete architectural overview of the Bazzite development infrastructure.

## 🖥️ Hardware

| Component | Specification |
|-----------|---------------|
| **OS** | Bazzite Desktop (Fedora 43 Atomic) |
| **Kernel** | Linux 6.17.7-ba25.fc43.x86_64 |
| **Primary Storage** | NVMe 476.9GB (encrypted LUKS) |
| **Backup Storage** | USB SSD 953.9GB (btrfs) |
| **Network** | Tailscale VPN (tail8be4f7.ts.net) |

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Bazzite Host System                       │
│                  (Immutable Fedora Atomic)                   │
├─────────────────────────────────────────────────────────────┤
│  Network Layer: Tailscale VPN (100.112.141.3)               │
│  Access: bazzite.tail8be4f7.ts.net                           │
└────────────┬────────────────────────────────────────────────┘
             │
     ┌───────┴───────┐
     │   Distrobox   │
     │  Container    │
     │   Runtime     │
     └───────┬───────┘
             │
    ┌────────┴────────┬────────────┐
    │                 │            │
┌───▼────┐      ┌────▼────┐  ┌───▼────┐    ┌────────┐  ┌──────────┐
│OpenCode│      │OpenClaw │  │LiteLLM │    │ Fedora │  │ Bazzite  │
│  Dev   │      │   Dev   │  │ Proxy  │    │ Tools  │  │   Arch   │
└────────┘      └─────────┘  └────────┘    └────────┘  └──────────┘

All containers share:
- /home/yish (full home directory access)
- Host networking (seamless connectivity)
- Host IPC (process communication)
```

## 📦 Container Ecosystem

### Active Containers

#### 1. opencode-dev
**Purpose**: OpenCode AI agent development environment

```yaml
Base Image: Fedora 43
Status: Running
Tools:
  - Node.js 22.22.0
  - Bun 1.3.9
  - pnpm 10.29.2
  - TypeScript 5.9.3
  - Git 2.53.0
  - GitHub CLI 2.86.0
Ports:
  - 5173 (Vite frontend)
  - 5003 (Backend API)
  - 5551 (OpenCode server)
Workspace: ~/workspace/opencode-projects
Config: ~/opencode-manager
```

#### 2. openclaw-dev
**Purpose**: OpenClaw AI agent runtime

```yaml
Base Image: Fedora 43
Status: Running
Tools:
  - Node.js 22.22.0
  - OpenClaw 2026.2.9
  - Git 2.53.0
Config: ~/.openclaw (shared)
Agents: main (Pi identity)
Model: qwen-portal/coder-model
```

#### 3. litellm-proxy
**Purpose**: Unified LLM API proxy with load balancing

```yaml
Base Image: Fedora 43
Status: Running
Tools:
  - Python 3.14.2
  - LiteLLM 1.81.10
Ports:
  - 4000 (API endpoint + Web UI)
Config: ~/.litellm/config.yaml
Features:
  - API key rotation (Claude, GPT-4, OpenRouter)
  - Request caching (1h TTL)
  - Automatic fallbacks
  - Cost tracking
Endpoint: http://localhost:4000/v1
Web UI: http://localhost:4000/ui
Remote: http://bazzite.tail8be4f7.ts.net:4000
```

#### 4. fedora-tools
**Purpose**: General development utilities

```yaml
Base Image: Fedora 43
Status: Running (2 days uptime)
Tools: Various CLI utilities
```

#### 5. bazzite-arch
**Purpose**: Arch Linux environment for AUR packages

```yaml
Base Image: ghcr.io/ublue-os/bazzite-arch:latest
Status: Running
Tools: pacman, yay (AUR helper)
```

### Template Containers (Not Active)

- **cursor-dev**: Cursor IDE environment (Ubuntu 24.04)
- **claude-dev**: Claude Code environment (Fedora 43)
- **windsurf-dev**: Windsurf AI environment (Fedora 43)

## 🌐 Network Configuration

### Tailscale VPN

```yaml
Network ID: tail8be4f7.ts.net
Owner: Jos-few43@github
DERP Relay: San Francisco (19.8ms)
NAT Traversal: UPnP (working)

Addresses:
  IPv4: 100.112.141.3
  IPv6: fd7a:115c:a1e0::fc01:8d8c
  Public IPv4: 73.222.223.77:43864
  Public IPv6: 2601:642:4f88:1520:d7f8:5583:2883:b848:55924

DNS:
  MagicDNS: Enabled
  Search Domain: tail8be4f7.ts.net
  DNS Server: 100.100.100.100 (Tailscale)
```

### Connected Devices

| Device | Status | Last Seen |
|--------|--------|-----------|
| bazzite | ✅ Online | - |
| iphone-14 | ⚠️ Idle | Active |
| iphone-12 | ❌ Offline | 28 days ago |

### Firewall

- Default: Deny all incoming
- Tailscale: Auto-configured (allow VPN traffic)
- Local services: Accessible via Tailscale only
- No public ports exposed

## 💾 Storage Layout

### Primary (NVMe)

```
nvme0n1                         476.9G
├─nvme0n1p1 (EFI)               600M   /boot/efi
├─nvme0n1p2 (Boot)              2G     /boot
└─nvme0n1p3 (Encrypted)         474.4G
  └─luks-17b3bf76... (LUKS)     474.3G
    └─btrfs                              /var/lib/containers/storage/overlay
                                        /usr/share/sddm/themes
```

### Backup (USB SSD)

```
sda                             953.9G  /run/media/yish/bazzite_bazzite
├─sda1 (EFI)                    600M
├─sda2 (Boot)                   2G
└─sda3 (btrfs)                  951.3G  (Previous system backup)
```

## 🔐 Security

### Encryption

- ✅ NVMe storage: LUKS encrypted
- ✅ Home directory: Encrypted at rest
- ✅ Tailscale: WireGuard encryption (ChaCha20-Poly1305)

### Authentication

- ✅ SSH: Key-based authentication
- ✅ Tailscale: OAuth via Jos-few43@github
- ✅ System: Local user password

### Access Control

- No public ports exposed
- All remote access via Tailscale VPN
- Container isolation via distrobox
- SELinux: Disabled for containers (label=disable)

## 🛠️ Services

### System Services

```bash
tailscaled    # Tailscale VPN daemon (active, 2 days uptime)
sshd          # SSH server (active, enabled)
podman        # Container runtime
systemd       # Init system
```

### Container Services

Running inside containers:
- OpenCode: Development servers (when started)
- OpenClaw: AI agent runtime (on-demand)

## 📊 Resource Usage

Current snapshot:

```yaml
Memory:
  Tailscale: 38.3M (peak: 70M)
  Containers: Shared from host

Storage:
  /home/yish: 82G used (on NVMe)
  Container images: ~10GB
  OpenClaw config: ~5.3MB
  OpenCode workspace: Variable
```

## 🔄 Update Strategy

### Host System (Bazzite)

```bash
# Atomic updates (immutable OS)
rpm-ostree upgrade --reboot

# Check current deployment
rpm-ostree status
```

### Containers

```bash
# Update individual container
distrobox enter opencode-dev -- sudo dnf update -y

# Update all containers
for container in $(distrobox list --no-color | tail -n +2 | awk '{print $2}'); do
  distrobox enter "$container" -- sudo dnf update -y
done
```

### Applications

- Flatpak apps: `flatpak update`
- npm packages: `npm update -g` (in containers)
- System-wide: Bazzite handles via rpm-ostree

## 📁 Important Directories

### On Host

```
/var/home/yish/
├── distrobox-configs/          # Container configuration repo (git)
├── infrastructure-docs/        # This documentation repo (git)
├── opencode-manager/           # OpenCode Manager project
├── opencode-antigravity-multi-auth/  # OpenCode plugin
├── .openclaw/                  # OpenClaw config (shared to containers)
├── .opencode/                  # OpenCode config
├── .litellm/                   # LiteLLM config (shared to containers)
│   ├── config.yaml             # Proxy configuration
│   ├── .env                    # API keys (gitignored)
│   └── README.md               # Usage guide
├── workspace/                  # Project workspaces
│   ├── opencode-projects/
│   └── openclaw-projects/
└── Projects/                   # User projects
```

### In Containers

Containers see the same `/home/yish` as the host (bind-mounted).

## 🎯 Use Cases

This infrastructure supports:

1. **AI-Assisted Development**
   - OpenCode for code generation
   - OpenClaw for agent workflows
   - Isolated environments per tool

2. **Remote Development**
   - SSH from anywhere via Tailscale
   - Web UI access for OpenCode
   - Mobile access to documentation

3. **Reproducible Environments**
   - Git-tracked container configs
   - Declarative setup scripts
   - Version-controlled documentation

4. **System Stability**
   - Atomic OS updates (rollback if needed)
   - Container isolation (no host pollution)
   - Encrypted storage

## 📈 Performance

- **DERP Latency**: 19.8ms to San Francisco relay
- **Container Overhead**: Minimal (shared kernel)
- **Network**: Direct peer-to-peer when possible via Tailscale
- **Storage**: NVMe for fast I/O

## 🔍 Monitoring

### Check System Health

```bash
# Tailscale connectivity
tailscale status
tailscale netcheck

# Container status
distrobox list
podman ps -a

# System resources
htop
df -h
```

### Logs

```bash
# System logs
journalctl -xe

# Tailscale logs
journalctl -u tailscaled

# Container logs
podman logs opencode-dev
podman logs openclaw-dev
```

## 🚨 Disaster Recovery

### Backup Strategy

1. **Container configs**: Git repos (push to GitHub)
2. **OpenClaw config**: `~/.openclaw` (backup to USB/cloud)
3. **Projects**: Regular git commits + push
4. **System**: USB SSD has previous system snapshot

### Recovery Procedure

If system failure:

1. **Boot from USB** (previous Bazzite install available)
2. **Restore from GitHub**:
   ```bash
   git clone https://github.com/yishua/distrobox-configs
   git clone https://github.com/yishua/infrastructure-docs
   ```
3. **Recreate containers**: Run setup scripts
4. **Restore OpenClaw**: Copy `~/.openclaw` from backup
5. **Reconnect Tailscale**: `tailscale up`

## 📝 Maintenance Schedule

- **Daily**: Git commits for active projects
- **Weekly**: Container updates
- **Monthly**: System update (rpm-ostree upgrade)
- **Quarterly**: Review and cleanup old container snapshots
- **Annually**: Full backup verification

---

**System Status**: ✅ All systems operational
**Last Updated**: 2026-02-10
**Uptime**: Tailscale 2 days, containers stable
