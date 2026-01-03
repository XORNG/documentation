# XORNG Installation Guide

Complete guide for installing and running XORNG locally for development and production use.

## Prerequisites

### Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| **Node.js** | 20.0.0+ | Runtime for XORNG components |
| **Docker** | 24.0.0+ | Container isolation for sub-agents |
| **pnpm** | 8.0.0+ | Package manager (recommended) |
| **Git** | 2.40.0+ | Version control |

### Optional Software

| Software | Purpose |
|----------|---------|
| **Redis** | Memory system caching |
| **Qdrant** | Vector database for semantic search |

## Quick Start

### 1. Clone Repositories

```bash
# Create XORNG directory
mkdir -p ~/xorng && cd ~/xorng

# Clone core repositories
git clone https://github.com/XORNG/core
git clone https://github.com/XORNG/node
git clone https://github.com/XORNG/automation

# Clone template repositories
git clone https://github.com/XORNG/template-base
git clone https://github.com/XORNG/template-validator
git clone https://github.com/XORNG/template-task
git clone https://github.com/XORNG/template-knowledge

# Clone sub-agents
git clone https://github.com/XORNG/validator-code-review
git clone https://github.com/XORNG/validator-security
git clone https://github.com/XORNG/knowledge-documentation
git clone https://github.com/XORNG/knowledge-best-practices
```

### 2. Install Dependencies

```bash
# Install all dependencies using pnpm workspaces
cd ~/xorng
pnpm install

# Or install each package individually
for dir in core node template-* validator-* knowledge-*; do
  cd ~/xorng/$dir && npm install
done
```

### 3. Configure Environment

Create a `.env` file in the root directory:

```bash
# Required: AI Provider Keys (at least one)
OPENAI_API_KEY=sk-your-openai-key
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key

# Optional: For enhanced features
QDRANT_API_KEY=your-qdrant-key
QDRANT_URL=http://localhost:6333
REDIS_URL=redis://localhost:6379

# Optional: For GitHub automation
GITHUB_TOKEN=ghp_your-github-token

# Logging
LOG_LEVEL=info
```

### 4. Build All Packages

```bash
# Build in dependency order
cd ~/xorng/template-base && npm run build
cd ~/xorng/template-validator && npm run build
cd ~/xorng/template-task && npm run build
cd ~/xorng/template-knowledge && npm run build
cd ~/xorng/node && npm run build
cd ~/xorng/core && npm run build

# Build sub-agents
cd ~/xorng/validator-code-review && npm run build
cd ~/xorng/validator-security && npm run build
cd ~/xorng/knowledge-documentation && npm run build
cd ~/xorng/knowledge-best-practices && npm run build

# Build automation
cd ~/xorng/automation && npm run build
```

### 5. Start XORNG

```bash
# Start the core orchestrator
cd ~/xorng/core
npm start

# In another terminal, start automation (optional)
cd ~/xorng/automation
npm start
```

---

## Docker Installation

### Using Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  # Core XORNG Orchestrator
  xorng-core:
    build: ./core
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - REDIS_URL=redis://redis:6379
      - QDRANT_URL=http://qdrant:6333
    depends_on:
      - redis
      - qdrant
    volumes:
      - ./config:/app/config

  # Validator Sub-Agents
  validator-code-review:
    build: ./validator-code-review
    environment:
      - LOG_LEVEL=info

  validator-security:
    build: ./validator-security
    environment:
      - LOG_LEVEL=info

  # Knowledge Sub-Agents
  knowledge-documentation:
    build: ./knowledge-documentation
    environment:
      - DOCS_PATH=/docs
    volumes:
      - ./docs:/docs:ro

  knowledge-best-practices:
    build: ./knowledge-best-practices
    environment:
      - PRACTICES_PATH=/practices
    volumes:
      - ./practices:/practices:ro

  # Supporting Services
  redis:
    image: redis:7-alpine
    volumes:
      - redis-data:/data

  qdrant:
    image: qdrant/qdrant:latest
    volumes:
      - qdrant-data:/qdrant/storage

volumes:
  redis-data:
  qdrant-data:
```

Run with Docker Compose:

```bash
docker-compose up -d
```

### Building Individual Images

```bash
# Build sub-agent images
docker build -t xorng/validator-code-review ./validator-code-review
docker build -t xorng/validator-security ./validator-security
docker build -t xorng/knowledge-documentation ./knowledge-documentation
docker build -t xorng/knowledge-best-practices ./knowledge-best-practices

# Run a validator
docker run -d \
  --name code-review \
  --memory="512m" \
  --cpus="0.5" \
  xorng/validator-code-review
```

---

## Configuration Files

### xorng.yaml (Project Configuration)

Create `xorng.yaml` in your project root:

```yaml
# XORNG Project Configuration
version: "1.0"

# Memory Configuration
memory:
  type: redis  # redis | memory | qdrant
  url: ${REDIS_URL}
  ttl: 3600    # seconds

# Token Budget
tokens:
  daily_limit: 100000
  warning_threshold: 80000
  cost_tracking: true

# Enabled Validators
validators:
  - name: code-review
    enabled: true
    severity_threshold: warning
  - name: security
    enabled: true
    severity_threshold: info

# Enabled Knowledge Sources
knowledge:
  - name: documentation
    enabled: true
    sources:
      - type: local
        path: ./docs
      - type: git
        url: https://github.com/your-org/docs.git
  - name: best-practices
    enabled: true
    sources:
      - type: local
        path: ./practices

# Self-Improvement Settings
self_improvement:
  enabled: true
  human_approval: true
  auto_deploy: false
  github_repo: your-org/xorng-config

# Notification Settings
notifications:
  tool_suggestions: true
  frequency: once_per_tool
```

### Node Configuration (node.yaml)

See [Node Configuration Guide](./NODE_CONFIGURATION.md) for detailed AI provider setup.

---

## Development Setup

### Running in Development Mode

```bash
# Watch mode for core
cd ~/xorng/core
npm run dev

# Watch mode for sub-agents
cd ~/xorng/validator-code-review
npm run dev
```

### Testing

```bash
# Run tests for all packages
cd ~/xorng
pnpm test

# Run tests for specific package
cd ~/xorng/core
npm test
```

### Linking Local Packages

For development, link packages locally:

```bash
# Link template-base
cd ~/xorng/template-base
npm link

# Use in other packages
cd ~/xorng/core
npm link @xorng/template-base
```

---

## VS Code Extension Setup

### Install Extension

```bash
# From VS Code marketplace (when available)
code --install-extension xorng.xorng-assistant

# Or install from VSIX
code --install-extension ./extensions/xorng-assistant-0.1.0.vsix
```

### Configure Extension

Add to VS Code settings (`.vscode/settings.json`):

```json
{
  "xorng.serverUrl": "http://localhost:3000",
  "xorng.autoStart": true,
  "xorng.logLevel": "info",
  "xorng.enableTelemetry": true
}
```

---

## Troubleshooting

### Common Issues

#### Docker Permission Denied

```bash
# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

#### Node Version Mismatch

```bash
# Use nvm to manage Node versions
nvm install 20
nvm use 20
```

#### Port Already in Use

```bash
# Find and kill process on port
lsof -ti:3000 | xargs kill -9
```

#### Memory Issues with Sub-Agents

```bash
# Increase Docker memory limit
docker run --memory="1g" xorng/validator-code-review
```

### Logs

```bash
# View core logs
tail -f ~/xorng/core/logs/xorng.log

# View Docker container logs
docker logs -f xorng-core

# Enable debug logging
LOG_LEVEL=debug npm start
```

---

## Next Steps

- [Node Configuration](./NODE_CONFIGURATION.md) - Configure AI providers
- [Automation Guide](./AUTOMATION.md) - Set up self-improvement
- [Creating Sub-Agents](./CREATING_SUBAGENTS.md) - Build custom sub-agents
- [API Reference](./API_REFERENCE.md) - Core API documentation
