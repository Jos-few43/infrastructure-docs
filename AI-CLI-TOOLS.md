# AI CLI Tools Container

Isolated container for AI-powered command-line coding assistants including Qwen Code and Google Gemini CLI.

## Overview

This container houses multiple AI CLI tools in an isolated environment, keeping the host system clean while providing full access to your home directory for configuration and project access.

### Installed Tools

| Tool | Version | Provider | Purpose |
|------|---------|----------|---------|
| **Qwen Code** | 0.10.0 | Alibaba Cloud | AI coding assistant with interactive REPL |
| **Gemini CLI** | 0.28.0 | Google | Gemini-powered coding assistant with MCP support |

## Container Setup

### Quick Start

```bash
cd ~/distrobox-configs
./manage.sh create ai-cli-tools
./manage.sh setup ai-cli-tools
```

### Manual Setup

```bash
distrobox create --name ai-cli-tools-dev --image fedora:43 --yes
distrobox enter ai-cli-tools-dev
bash ~/distrobox-configs/ai-cli-tools-setup.sh
```

## Using the Tools

### Entering the Container

```bash
distrobox enter ai-cli-tools-dev
```

**Or create a shell alias:**

```bash
echo "alias aic='distrobox enter ai-cli-tools-dev'" >> ~/.bashrc
source ~/.bashrc
aic  # Enter container
```

### Qwen Code Usage

**Interactive mode:**

```bash
qwen
```

**One-shot query:**

```bash
qwen "explain this function"
qwen "write a function to parse JSON"
```

**With file context:**

```bash
qwen "refactor this code" < script.js
```

**Interactive with initial prompt:**

```bash
qwen -i "help me debug this code"
```

**Manage MCP servers:**

```bash
qwen mcp
```

**Manage extensions:**

```bash
qwen extensions list
qwen extensions install <extension-name>
```

### Gemini CLI Usage

**Interactive mode:**

```bash
gemini
```

**One-shot query:**

```bash
gemini -p "write a bash script to backup files"
gemini -p "optimize this SQL query"
```

**Interactive with initial prompt:**

```bash
gemini -i "help me implement authentication"
```

**Specify model:**

```bash
gemini -m gemini-2.0-flash-exp "generate unit tests"
```

**Manage MCP servers:**

```bash
gemini mcp
```

**Manage skills:**

```bash
gemini skills list
gemini skills add
```

**Manage hooks:**

```bash
gemini hooks list
```

**Debug mode:**

```bash
gemini -d  # Open debug console with F12
```

## Configuration

### Qwen Code Configuration

**Location:** `~/.config/qwen-code/`

**Settings file:** `~/.config/qwen-code/settings.json`

**Example configuration:**

```json
{
  "model": "qwen-coder",
  "apiKey": "your-api-key",
  "telemetry": {
    "enabled": false
  }
}
```

### Gemini CLI Configuration

**Location:** `~/.config/gemini-cli/`

**Settings file:** `~/.config/gemini-cli/settings.json`

**API Key setup:**

```bash
export GEMINI_API_KEY="your-google-ai-api-key"
```

**Or configure in settings:**

```json
{
  "apiKey": "your-google-ai-api-key",
  "model": "gemini-2.0-flash-exp"
}
```

## Running from Host (Without Entering Container)

### Direct Execution

```bash
distrobox enter ai-cli-tools-dev -- qwen "your query"
distrobox enter ai-cli-tools-dev -- gemini -p "your prompt"
```

### Shell Aliases for Convenience

Add to `~/.bashrc`:

```bash
# AI CLI Tools aliases
alias qwen='distrobox enter ai-cli-tools-dev -- qwen'
alias gemini='distrobox enter ai-cli-tools-dev -- gemini'
```

Then use directly from host:

```bash
qwen "explain this code"
gemini -p "write unit tests"
```

## MCP Server Integration

Both tools support **Model Context Protocol (MCP)** servers for extended capabilities.

### Managing MCP Servers

**Qwen Code:**

```bash
qwen mcp
# Follow interactive prompts to add/remove MCP servers
```

**Gemini CLI:**

```bash
gemini mcp
# Configure MCP servers for enhanced context
```

### Example MCP Servers

- **Filesystem** - Read/write local files
- **GitHub** - Access repositories, issues, PRs
- **Database** - Query databases directly
- **Web Search** - Search the web for context

## Migration from Host Installation

If you previously installed these tools globally on the host, follow these steps to migrate to the containerized setup:

### 1. Verify Container Installation

```bash
distrobox enter ai-cli-tools-dev -- qwen --version
distrobox enter ai-cli-tools-dev -- gemini --version
```

### 2. Backup Host Configurations

```bash
# Backup Qwen config
cp -r ~/.config/qwen-code ~/.config/qwen-code.backup

# Backup Gemini config
cp -r ~/.config/gemini-cli ~/.config/gemini-cli.backup
```

### 3. Uninstall from Host

```bash
# Remove global npm packages
npm uninstall -g @qwen-code/qwen-code
npm uninstall -g @google/gemini-cli

# Verify removal
which qwen    # Should show nothing
which gemini  # Should show nothing
```

### 4. Set Up Aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
# AI CLI Tools (containerized)
alias qwen='distrobox enter ai-cli-tools-dev -- qwen'
alias gemini='distrobox enter ai-cli-tools-dev -- gemini'
```

Reload shell:

```bash
source ~/.bashrc  # or source ~/.zshrc
```

### 5. Test Aliases

```bash
qwen --version     # Should show 0.10.0
gemini --version   # Should show 0.28.0
```

**Note:** Configurations in `~/.config/` are automatically shared with the container, so your settings persist without migration.

## Troubleshooting

### Tool Not Found

**Problem:** `command not found: qwen` or `command not found: gemini`

**Solution:**

1. Enter container first:
   ```bash
   distrobox enter ai-cli-tools-dev
   qwen --version
   ```

2. Or use full command:
   ```bash
   distrobox enter ai-cli-tools-dev -- qwen --version
   ```

3. Or set up aliases (see above)

### Configuration Not Loading

**Problem:** Settings or API keys not found

**Solution:**

Container shares `~/.config/` from host. Verify files exist:

```bash
ls -la ~/.config/qwen-code/
ls -la ~/.config/gemini-cli/
```

### Node.js Version Issues

**Problem:** "Node.js version mismatch"

**Solution:**

Container uses Node.js 22. Update tools:

```bash
distrobox enter ai-cli-tools-dev
npm update -g @qwen-code/qwen-code @google/gemini-cli
```

### Container Won't Start

**Problem:** Container fails to start

**Solution:**

1. Check container status:
   ```bash
   distrobox list
   ```

2. Recreate container:
   ```bash
   distrobox rm -f ai-cli-tools-dev
   cd ~/distrobox-configs
   ./manage.sh create ai-cli-tools
   ./manage.sh setup ai-cli-tools
   ```

## Advanced Usage

### Using with LiteLLM Proxy

If you have LiteLLM proxy running, configure tools to use it:

**Qwen Code:**

```json
{
  "apiBase": "http://localhost:4000/v1",
  "apiKey": "your-litellm-master-key"
}
```

**Gemini CLI:**

```json
{
  "apiBase": "http://localhost:4000/v1",
  "apiKey": "your-litellm-master-key",
  "model": "gemini-pro"
}
```

### Batch Processing

**Process multiple files:**

```bash
for file in *.js; do
  qwen "document this code" < "$file" > "${file%.js}-docs.md"
done
```

### Pipeline Integration

**Use in scripts:**

```bash
#!/bin/bash
CODE=$(cat my-code.js)
REVIEW=$(echo "$CODE" | qwen -p "review this code for bugs")
echo "$REVIEW" > code-review.md
```

### CI/CD Integration

**Example GitHub Actions:**

```yaml
- name: AI Code Review
  run: |
    distrobox enter ai-cli-tools-dev -- \
      qwen -p "review this PR for security issues" < changed-files.txt
```

## Container Management

### Update Tools

```bash
distrobox enter ai-cli-tools-dev
npm update -g @qwen-code/qwen-code @google/gemini-cli
```

### Update System Packages

```bash
cd ~/distrobox-configs
./manage.sh update ai-cli-tools
```

### Stop Container

```bash
./manage.sh stop ai-cli-tools
```

### Remove Container

```bash
./manage.sh remove ai-cli-tools
```

### Recreate Container

```bash
distrobox rm -f ai-cli-tools-dev
./manage.sh create ai-cli-tools
./manage.sh setup ai-cli-tools
```

## Security Considerations

### API Keys

- Store API keys in `~/.config/<tool>/settings.json`
- Never commit API keys to git
- Use environment variables for CI/CD:
  ```bash
  export QWEN_API_KEY="..."
  export GEMINI_API_KEY="..."
  ```

### Container Isolation

- Container runs with user permissions (not root)
- Shares `~/.config/` for settings persistence
- Full access to home directory (for project work)
- Isolated from host npm global packages

### Network Access

- Both tools require internet for AI API calls
- Configure proxy if needed:
  ```bash
  export HTTP_PROXY="http://proxy.example.com:8080"
  export HTTPS_PROXY="http://proxy.example.com:8080"
  ```

## Useful Commands

```bash
# Enter container
distrobox enter ai-cli-tools-dev

# Run qwen interactively
qwen

# Run gemini with specific model
gemini -m gemini-2.0-flash-exp

# One-shot code generation
qwen "write a function to sort an array"

# Debug mode
gemini -d

# List installed extensions
qwen extensions list
gemini skills list

# Update tools
npm update -g @qwen-code/qwen-code @google/gemini-cli

# Check versions
qwen --version
gemini --version

# Exit container
exit
```

## Comparison: Qwen Code vs Gemini CLI

| Feature | Qwen Code | Gemini CLI |
|---------|-----------|------------|
| **Provider** | Alibaba Cloud | Google AI |
| **Primary Model** | Qwen Coder | Gemini 2.0 Flash |
| **MCP Support** | ✅ Yes | ✅ Yes |
| **Extensions** | ✅ Yes | ✅ Skills system |
| **Hooks** | ❌ No | ✅ Yes |
| **Debug Mode** | ✅ Yes | ✅ Yes (F12) |
| **Sandbox** | ❌ No | ✅ Yes |
| **Interactive REPL** | ✅ Yes | ✅ Yes |
| **One-shot Mode** | ✅ Yes | ✅ Yes (-p flag) |

## Performance

- **Cold start:** ~2 seconds (container already running)
- **Query latency:** Depends on API provider
- **Container overhead:** Minimal (shared kernel)
- **Disk usage:** ~500MB (Node.js + tools)

## References

- [Qwen Code Documentation](https://qwen.readthedocs.io/)
- [Gemini CLI Documentation](https://ai.google.dev/gemini-api/docs)
- [MCP Protocol](https://modelcontextprotocol.io/)
- [Distrobox Documentation](https://distrobox.it/)

---

**Status**: ✅ Active
**Container**: `ai-cli-tools-dev` (Fedora 43)
**Tools**: Qwen Code 0.10.0, Gemini CLI 0.28.0
**Node.js**: 22.22.0
**Configuration**: `~/.config/qwen-code/`, `~/.config/gemini-cli/`
