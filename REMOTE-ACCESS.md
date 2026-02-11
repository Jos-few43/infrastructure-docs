# Remote Access Guide

Complete guide for accessing your Bazzite infrastructure from anywhere, including mobile devices.

## 🌐 Tailscale VPN

Your infrastructure is accessible remotely via Tailscale VPN.

### Connection Details

```
Tailscale IPv4:   100.112.141.3
Tailscale IPv6:   fd7a:115c:a1e0::fc01:8d8c
MagicDNS Name:    bazzite.tail8be4f7.ts.net
Network:          Jos-few43@github
```

### Connected Devices

- **bazzite** (this machine) - Online
- **iphone-14** - Idle
- **iphone-12** - Offline (28 days)

## 💻 SSH Access

### From Desktop/Laptop

```bash
# Using MagicDNS (recommended)
ssh yish@bazzite.tail8be4f7.ts.net

# Using IP address
ssh yish@100.112.141.3

# Using Tailscale SSH (zero-config)
tailscale ssh bazzite
```

### From Mobile (iPhone)

**Option 1: Termius App**
1. Install Termius from App Store
2. Add new host:
   - Host: `bazzite.tail8be4f7.ts.net`
   - User: `yish`
   - Port: `22`
3. Connect

**Option 2: Blink Shell**
1. Install Blink Shell from App Store
2. Add SSH config:
   ```
   Host bazzite
       HostName bazzite.tail8be4f7.ts.net
       User yish
   ```
3. Connect: `ssh bazzite`

## 🌍 Web Access

### Development Servers

When dev servers are running:

| Service | URL | Port |
|---------|-----|------|
| OpenCode Frontend | http://bazzite.tail8be4f7.ts.net:5173 | 5173 |
| OpenCode Backend | http://bazzite.tail8be4f7.ts.net:5003 | 5003 |
| OpenCode Server | http://bazzite.tail8be4f7.ts.net:5551 | 5551 |

### Start Dev Servers Remotely

```bash
# SSH in
ssh yish@bazzite.tail8be4f7.ts.net

# Enter container
distrobox enter opencode-dev

# Start servers
cd ~/opencode-manager
pnpm dev
```

Then access from any browser on your Tailscale network:
```
http://bazzite.tail8be4f7.ts.net:5173
```

## 📱 Mobile Access via Claude

### Method 1: Claude Projects (Best for Mobile)

**Setup:**

1. **Go to claude.ai** on desktop
2. **Create a new Project**:
   - Name: "Bazzite Infrastructure"
   - Description: "My development infrastructure documentation and configuration"

3. **Upload documentation** to the project:
   - `README.md`
   - `DISTROBOX-DEPLOY.md`
   - `REMOTE-ACCESS.md` (this file)
   - `SYSTEM-OVERVIEW.md`
   - `openclaw-container-guide.md`
   - Any other relevant docs

4. **Add custom instructions** to the project:
   ```
   This project contains my Bazzite system infrastructure documentation.

   Key details:
   - System: Bazzite (Fedora atomic)
   - Tailscale: bazzite.tail8be4f7.ts.net (100.112.141.3)
   - Containers: opencode-dev, openclaw-dev
   - User: yish

   When I ask questions, reference these docs and provide specific
   commands for my setup.
   ```

**Usage on Mobile:**

1. **Open Claude mobile app**
2. **Select "Bazzite Infrastructure" project**
3. **Ask questions** like:
   - "How do I SSH into my Bazzite machine?"
   - "What's the command to enter the OpenCode container?"
   - "Show me the OpenClaw setup process"
   - "How do I check Tailscale status?"

4. **Claude will reference your uploaded docs** and provide contextual answers!

**Benefits:**
- ✅ Docs accessible offline (cached in app)
- ✅ Ask questions in natural language
- ✅ Get commands specific to your setup
- ✅ No need to remember exact paths/commands
- ✅ Syncs between mobile and web

### Method 2: GitHub + Mobile Browser

**Setup:**

```bash
# Create GitHub repos (if not already done)
cd ~/distrobox-configs
gh repo create distrobox-configs --public --source=. --push

cd ~/infrastructure-docs
gh repo create infrastructure-docs --public --source=. --push
```

**Usage:**
1. Visit GitHub.com on mobile browser
2. Navigate to your repos
3. Read markdown files directly on GitHub

### Method 3: Syncthing (Advanced)

**Setup Syncthing between Bazzite and iPhone:**

1. **Install Syncthing on Bazzite:**
   ```bash
   flatpak install flathub me.kozec.syncthingtk
   ```

2. **Install Möbius Sync on iPhone** (App Store)

3. **Configure folder sync:**
   - Add `~/infrastructure-docs` folder
   - Connect devices via Tailscale IPs

4. **Access on iPhone:**
   - Docs sync to local storage
   - Read with any markdown viewer app

## 🎯 Common Remote Workflows

### Workflow 1: Quick SSH Check

```bash
# From iPhone (Termius)
ssh yish@bazzite.tail8be4f7.ts.net
distrobox list
tailscale status
exit
```

### Workflow 2: Start Dev Servers

```bash
# From any device
ssh yish@bazzite.tail8be4f7.ts.net
distrobox enter opencode-dev
cd ~/opencode-manager
pnpm dev
# Keep terminal open, access http://bazzite.tail8be4f7.ts.net:5173
```

### Workflow 3: Run OpenClaw

```bash
# SSH in
ssh yish@bazzite.tail8be4f7.ts.net

# Enter container
distrobox enter openclaw-dev

# Run OpenClaw
openclaw
```

### Workflow 4: Ask Claude for Help

1. Open Claude mobile app
2. Go to "Bazzite Infrastructure" project
3. Ask: "I want to check if my containers are running, what command do I use?"
4. Claude references your docs and responds: `distrobox list`

## 🔐 Security Best Practices

### SSH Keys

**Add your mobile SSH key to Bazzite:**

```bash
# On Bazzite
cat >> ~/.ssh/authorized_keys << EOF
<paste your mobile device's public key>
EOF
```

### Tailscale Access Control

Your Tailscale network is private to:
- Jos-few43@github account
- Only your authorized devices

No public internet exposure!

### Port Security

- ✅ All services accessible only via Tailscale
- ✅ No ports exposed to public internet
- ✅ Firewall configured for localhost/Tailscale only

## 📊 Connection Testing

### Test from Remote Device

```bash
# Test Tailscale connectivity
tailscale ping bazzite

# Test SSH
ssh -v yish@bazzite.tail8be4f7.ts.net echo "Connected!"

# Test web access (if servers running)
curl -I http://bazzite.tail8be4f7.ts.net:5173
```

### Troubleshooting Connection Issues

**Can't connect:**

1. **Check Tailscale on both devices:**
   ```bash
   tailscale status
   ```

2. **Verify Bazzite is online:**
   ```bash
   # From another device on network
   ping bazzite.tail8be4f7.ts.net
   ```

3. **Check SSH service:**
   ```bash
   # On Bazzite
   systemctl status sshd
   ```

4. **Test from Tailscale IP:**
   ```bash
   ssh yish@100.112.141.3
   ```

## 🚀 Advanced: VS Code Remote SSH

### Setup

1. **Install "Remote - SSH" extension** in VS Code

2. **Add to SSH config** (~/.ssh/config on your laptop):
   ```
   Host bazzite
       HostName bazzite.tail8be4f7.ts.net
       User yish

   Host bazzite-opencode
       HostName bazzite.tail8be4f7.ts.net
       User yish
       RemoteCommand distrobox enter opencode-dev
       RequestTTY yes
   ```

3. **Connect:**
   - Cmd/Ctrl+Shift+P → "Remote-SSH: Connect to Host"
   - Select "bazzite"
   - Opens remote VS Code session

4. **Open workspace:**
   - File → Open Folder → `/home/yish/opencode-manager`

## 📱 Quick Mobile Reference Card

Save this to your mobile notes app:

```
BAZZITE QUICK ACCESS

SSH: ssh yish@bazzite.tail8be4f7.ts.net
Web: http://bazzite.tail8be4f7.ts.net:5173

CONTAINERS:
- OpenCode: distrobox enter opencode-dev
- OpenClaw: distrobox enter openclaw-dev

QUICK COMMANDS:
- List containers: distrobox list
- OpenClaw: openclaw
- OpenCode dev: cd ~/opencode-manager && pnpm dev
- Tailscale status: tailscale status

CLAUDE PROJECT: "Bazzite Infrastructure"
```

## 🎓 Tips

1. **Use Claude Projects** for quick command lookup on mobile
2. **Add SSH keys** to avoid password prompts
3. **Use MagicDNS names** instead of IPs (more reliable)
4. **Keep Termius** installed for emergency SSH access
5. **Bookmark** `http://bazzite.tail8be4f7.ts.net:5173` in mobile Safari
6. **Upload new docs** to Claude Project when you create them

---

**Your infrastructure is now accessible from anywhere!** 🌍

Whether you're at a coffee shop, traveling, or just on your couch with your iPhone, you can access your development environment, check container status, and get help from Claude using your own documentation.
