# LiteLLM Proxy Configuration

Unified API proxy for load balancing, rotation, and fallbacks across multiple LLM providers.

## Overview

LiteLLM acts as a universal proxy that:
- **Rotates** between multiple API keys automatically
- **Load balances** requests across providers
- **Caches** responses to reduce costs
- **Provides fallbacks** when primary providers fail
- **Tracks costs** and usage across all models
- **Exposes OpenAI-compatible** API endpoint

## Container Setup

### Quick Start

```bash
cd ~/distrobox-configs
./manage.sh create litellm
./manage.sh setup litellm
```

### Manual Setup

```bash
distrobox create --name litellm-proxy --image fedora:43 --yes
distrobox enter litellm-proxy
bash ~/distrobox-configs/litellm-setup.sh
```

## Configuration

### Directory Structure

```
~/.litellm/
├── config.yaml          # Main configuration (models, routing, caching)
├── .env                 # API keys (DO NOT COMMIT)
├── .env.template        # Template for API keys
├── .gitignore           # Protects sensitive files
└── README.md            # Usage documentation
```

### API Key Setup

```bash
cd ~/.litellm
cp .env.template .env
vim .env  # Add your actual API keys
```

**Required environment variables:**

```bash
LITELLM_MASTER_KEY=sk-litellm-your-secret-master-key-change-this
LITELLM_UI_USER=admin
LITELLM_UI_PASSWORD=change-this-secure-password

ANTHROPIC_API_KEY_1=sk-ant-api03-your-key-here
OPENAI_API_KEY_1=sk-proj-your-openai-key-here
OPENROUTER_API_KEY=sk-or-v1-your-openrouter-key-here
```

**Optional (for load balancing):**

```bash
ANTHROPIC_API_KEY_2=sk-ant-api03-your-second-key-here
OPENAI_API_KEY_2=sk-proj-your-second-key-here
GROQ_API_KEY=gsk_your-groq-key-here
DEEPSEEK_API_KEY=sk-your-deepseek-key-here
```

## Running the Proxy

### Start Proxy Server

```bash
distrobox enter litellm-proxy
litellm --config ~/.litellm/config.yaml
```

**Default endpoint**: `http://localhost:4000`

### Remote Access (Tailscale)

```bash
litellm --config ~/.litellm/config.yaml --host 0.0.0.0 --port 4000
```

**Remote URL**: `http://bazzite.tail8be4f7.ts.net:4000`

### Background Service

```bash
distrobox enter litellm-proxy -- bash -c \
  'cd ~ && nohup litellm --config ~/.litellm/config.yaml > ~/.litellm/proxy.log 2>&1 &'
```

**Check status:**

```bash
curl http://localhost:4000/health
```

**View logs:**

```bash
tail -f ~/.litellm/proxy.log
```

## Using the Proxy

### OpenAI-Compatible Endpoint

```
http://localhost:4000/v1/chat/completions
```

### Example cURL Request

```bash
curl http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -d '{
    "model": "claude-opus",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### Available Models

| Model Name | Provider | Notes |
|------------|----------|-------|
| `claude-opus` | Anthropic | Rotates between multiple keys |
| `claude-sonnet` | Anthropic | Fast, cheaper |
| `gpt-4` | OpenAI | Rotates between multiple keys |
| `openrouter/claude-opus` | OpenRouter | Access 100+ models |
| `openrouter/gpt-4` | OpenRouter | Alternative routing |
| `groq/llama` | Groq | Very fast inference |
| `deepseek` | DeepSeek | Cheap, good quality |

### Web UI

Access at: `http://localhost:4000/ui`

**Login credentials** (from `.env`):
- Username: `$LITELLM_UI_USER`
- Password: `$LITELLM_UI_PASSWORD`

**Features:**
- View all configured models
- Test models interactively
- Monitor usage and costs
- View logs and metrics

## Load Balancing Strategy

### Simple Shuffle (Default)

Requests to the same model name are randomly distributed across all configured API keys.

**Example:**

```yaml
model_list:
  - model_name: claude-opus
    litellm_params:
      model: claude-opus-4
      api_key: os.environ/ANTHROPIC_API_KEY_1

  - model_name: claude-opus  # Same model name
    litellm_params:
      model: claude-opus-4
      api_key: os.environ/ANTHROPIC_API_KEY_2  # Different key
```

Requests to `claude-opus` shuffle between both keys automatically.

### Fallback Configuration

If a provider fails, LiteLLM automatically falls back to alternatives:

```yaml
router_settings:
  fallbacks: [
    {"gpt-4": ["claude-sonnet", "openrouter/gpt-4"]},
    {"claude-opus": ["gpt-4", "openrouter/claude-opus"]}
  ]
```

**Example flow:**
1. Request to `gpt-4` fails (rate limit)
2. Automatically retries with `claude-sonnet`
3. If that fails, tries `openrouter/gpt-4`

### Retry Logic

```yaml
router_settings:
  num_retries: 3
  timeout: 300
```

Each request retries up to 3 times before failing.

## Caching

### Local Cache (In-Memory)

```yaml
litellm_settings:
  cache: true
  cache_params:
    type: local
    ttl: 3600  # 1 hour
```

**How it works:**
- Identical requests return cached responses
- Saves API costs on repeated queries
- Cache persists for 1 hour (3600 seconds)
- Clears on proxy restart

### Cost Tracking

Every request is logged with:
- Model used
- Tokens consumed (prompt + completion)
- Estimated cost
- Response time

View in Web UI → Usage Stats

## Integration Examples

### OpenCode / OpenClaw

**Base URL:** `http://localhost:4000/v1`
**API Key:** Your `$LITELLM_MASTER_KEY`
**Model:** Any model name from `config.yaml`

### Python

```python
import openai

client = openai.OpenAI(
    base_url="http://localhost:4000/v1",
    api_key="YOUR_LITELLM_MASTER_KEY"
)

response = client.chat.completions.create(
    model="claude-opus",
    messages=[{"role": "user", "content": "Hello!"}]
)

print(response.choices[0].message.content)
```

### JavaScript / Node.js

```javascript
import OpenAI from 'openai';

const client = new OpenAI({
  baseURL: 'http://localhost:4000/v1',
  apiKey: process.env.LITELLM_MASTER_KEY,
});

const response = await client.chat.completions.create({
  model: 'claude-opus',
  messages: [{role: 'user', content: 'Hello!'}],
});

console.log(response.choices[0].message.content);
```

## Advanced Configuration

### Custom Port

```bash
litellm --config ~/.litellm/config.yaml --port 8000
```

### Debug Logging

```bash
litellm --config ~/.litellm/config.yaml --debug
```

### Adding New Providers

**1. Edit `~/.litellm/config.yaml`:**

```yaml
model_list:
  - model_name: my-model
    litellm_params:
      model: provider/model-name
      api_key: os.environ/MY_API_KEY
```

**2. Add key to `~/.litellm/.env`:**

```bash
MY_API_KEY=sk-your-key-here
```

**3. Restart proxy**

## Monitoring

### Health Check

```bash
curl http://localhost:4000/health
```

**Expected response:**

```json
{"status": "healthy"}
```

### View Logs

```bash
tail -f ~/.litellm/proxy.log
```

### Usage Statistics

Visit: `http://localhost:4000/ui` → Usage tab

Shows:
- Total requests
- Tokens consumed
- Cost breakdown by model
- Error rates
- Response times

## Troubleshooting

### Proxy Won't Start

**Check config syntax:**

```bash
cat ~/.litellm/config.yaml
```

**Check environment variables loaded:**

```bash
env | grep API_KEY
```

### API Key Errors

**Test individual provider:**

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY_1" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model":"claude-opus-4",
    "max_tokens":100,
    "messages":[{"role":"user","content":"test"}]
  }'
```

### Connection Refused

**Check if proxy is running:**

```bash
ps aux | grep litellm
```

**Check port is open:**

```bash
netstat -tuln | grep 4000
```

### Rate Limit Errors

LiteLLM will automatically:
1. Retry with exponential backoff (3 times)
2. Rotate to next available API key
3. Fall back to alternative provider (if configured)

If still failing, check:
- API key quotas/limits
- Provider status pages
- Fallback configurations in `config.yaml`

## Security Considerations

### Protection

- ✅ `.env` file is gitignored (never commit API keys)
- ✅ `LITELLM_MASTER_KEY` protects your proxy
- ✅ Bind to `127.0.0.1` for local-only access
- ✅ Use Tailscale for secure remote access

### Best Practices

1. **Keep master key secret** - treat it like a password
2. **Don't expose publicly** - only bind to localhost or Tailscale
3. **Rotate API keys regularly** - update `.env` periodically
4. **Monitor usage** - check Web UI for unusual activity
5. **Use strong passwords** - for Web UI login

### Network Isolation

**Local only:**

```bash
litellm --config ~/.litellm/config.yaml
# Binds to 127.0.0.1 by default
```

**Tailscale VPN only:**

```bash
litellm --config ~/.litellm/config.yaml --host 0.0.0.0 --port 4000
# Access via: http://bazzite.tail8be4f7.ts.net:4000
```

**NEVER expose to public internet** - no firewall rules for port 4000.

## Container Management

### Enter Container

```bash
distrobox enter litellm-proxy
```

**Or using manage script:**

```bash
cd ~/distrobox-configs
./manage.sh enter litellm
```

### Update Container

```bash
./manage.sh update litellm
```

### Stop Container

```bash
./manage.sh stop litellm
```

### Remove Container

```bash
./manage.sh remove litellm
```

## Performance Optimization

### Request Caching

Caching saves costs on repeated queries:

```yaml
litellm_settings:
  cache: true
  cache_params:
    type: local
    ttl: 3600  # Adjust TTL based on use case
```

**When to use:**
- Development/testing (same prompts repeated)
- Documentation generation (similar queries)
- Batch processing with duplicates

**When to disable:**
- Real-time chat (responses should be unique)
- Production workloads (fresh responses required)

### Rate Limiting

Prevent provider quota exhaustion:

```yaml
router_settings:
  rpm: 500   # Requests per minute
  tpm: 90000 # Tokens per minute
```

### Load Distribution

Multiple API keys = higher throughput:
- 1 key = 500 RPM (example)
- 2 keys = 1000 RPM (doubled)
- 3 keys = 1500 RPM (tripled)

## Useful Commands

```bash
# Create container
./manage.sh create litellm

# Run setup
./manage.sh setup litellm

# Enter container
./manage.sh enter litellm

# Start proxy (inside container)
litellm --config ~/.litellm/config.yaml

# Start proxy with debug
litellm --config ~/.litellm/config.yaml --debug

# Start proxy in background
nohup litellm --config ~/.litellm/config.yaml > ~/.litellm/proxy.log 2>&1 &

# Check health
curl http://localhost:4000/health

# Test model
curl http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -d '{"model":"claude-opus","messages":[{"role":"user","content":"test"}]}'

# View logs
tail -f ~/.litellm/proxy.log

# Kill proxy
pkill -f litellm
```

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Client Application                        │
│           (OpenCode, OpenClaw, custom scripts)               │
└─────────────────────┬───────────────────────────────────────┘
                      │ HTTP Request
                      │ POST /v1/chat/completions
                      │ Authorization: Bearer LITELLM_MASTER_KEY
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                  LiteLLM Proxy (localhost:4000)              │
│                                                               │
│  ┌─────────────┐   ┌──────────────┐   ┌─────────────┐      │
│  │   Router    │──▶│    Cache     │   │  Fallback   │      │
│  │ (Shuffle)   │   │  (1h TTL)    │   │   Logic     │      │
│  └─────────────┘   └──────────────┘   └─────────────┘      │
│         │                                                     │
│         ▼                                                     │
│  ┌──────────────────────────────────────────────────┐       │
│  │           API Key Rotation Pool                   │       │
│  │  [KEY_1] [KEY_2] [KEY_3] ... [KEY_N]             │       │
│  └──────────────────────────────────────────────────┘       │
└──────────┬────────────┬────────────┬───────────┬────────────┘
           │            │            │           │
           ▼            ▼            ▼           ▼
      ┌─────────┐  ┌─────────┐  ┌──────────┐  ┌─────────┐
      │Anthropic│  │ OpenAI  │  │OpenRouter│  │  Groq   │
      │  API    │  │   API   │  │   API    │  │   API   │
      └─────────┘  └─────────┘  └──────────┘  └─────────┘
```

## References

- [LiteLLM Documentation](https://docs.litellm.ai/)
- [Supported Providers](https://docs.litellm.ai/docs/providers)
- [Load Balancing Guide](https://docs.litellm.ai/docs/routing)
- [Caching Guide](https://docs.litellm.ai/docs/caching)
- [Proxy Server Docs](https://docs.litellm.ai/docs/proxy/quick_start)

---

**Status**: ✅ Configured and ready
**Container**: `litellm-proxy` (Fedora 43)
**Endpoint**: `http://localhost:4000/v1`
**Remote**: `http://bazzite.tail8be4f7.ts.net:4000`
**Web UI**: `http://localhost:4000/ui`
