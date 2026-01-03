# XORNG VS Code Extension Installation Guide

Complete guide for installing, configuring, and using the XORNG VS Code Extension for AI-powered code assistance.

## Overview

The XORNG VS Code Extension integrates AI orchestration directly into your development environment, allowing you to leverage GitHub Copilot, Claude, OpenAI, Anthropic, and local models through a unified chat interface.

### Core Features

The extension provides a complete AI orchestration experience:

| Feature | Status | Description |
|---------|--------|-------------|
| **Auto-Setup** | ✅ Working | Automatically clones and builds XORNG components on first run |
| **Auto-Update** | ✅ Working | Pulls latest changes on extension restart |
| **Multi-Provider Support** | ✅ Working | Switch between Copilot, OpenAI, Anthropic, Ollama |
| **Chat Participant** | ✅ Working | `@xorng` in VS Code Chat |
| **Slash Commands** | ✅ Working | `/review`, `/security`, `/explain`, `/refactor`, `/config` |
| **Specialized Prompts** | ✅ Working | Context-aware system prompts per command |
| **Model Selection** | ✅ Working | Choose model family (GPT-4o, Claude, etc.) |
| **Conversation Context** | ✅ Working | Maintains chat history within session |

### With XORNG Core Running

Full orchestration features when Core is running (auto-started by default):

| Feature | Status | Description |
|---------|--------|-------------|
| **Local Core Process** | ✅ Working | Core runs as a local child process via IPC |
| **Copilot Model Sharing** | ✅ Working | Core uses Copilot models via extension proxy |
| **Sub-Agent Orchestration** | ✅ Working | Intelligent routing to specialized sub-agents |
| **Auto-Discovery** | ✅ Working | Automatic sub-agent detection from Core |
| **Memory System** | ✅ Working | Short-term and long-term memory |
| **Streaming Responses** | ✅ Working | Real-time streaming from Core via IPC |

> **Note:** The extension automatically sets up XORNG Core on first run. No manual installation required!

## Prerequisites

### Required

| Requirement | Version | Notes |
|-------------|---------|-------|
| **VS Code** | 1.95.0+ | Required for Chat Participant API |
| **Node.js** | 18.0.0+ | For building from source |
| **npm** | 9.0.0+ | Package manager |

### Recommended

| Requirement | Purpose |
|-------------|---------|
| **GitHub Copilot** | Best experience with Copilot provider |
| **GitHub Copilot Chat** | Required for `@xorng` chat participant |

> **Note:** The extension works without GitHub Copilot by using native providers (OpenAI, Anthropic, or local models), but the chat participant experience is optimized for Copilot.

---

## Installation Methods

### Method 1: VS Code Marketplace (Recommended)

The easiest way to install XORNG:

1. Open VS Code
2. Press `Ctrl+Shift+X` (Windows/Linux) or `Cmd+Shift+X` (macOS)
3. Search for **"XORNG"**
4. Click **Install**

Or install via command line:

```bash
code --install-extension xorng.xorng-vscode
```

### Method 2: Install from VSIX File

For offline installation or testing pre-release versions:

#### Option A: Via VS Code UI

1. Download the `.vsix` file from [Releases](https://github.com/XORNG/extension-vscode/releases)
2. Open VS Code
3. Press `Ctrl+Shift+X` to open Extensions
4. Click the `...` menu (Views and More Actions)
5. Select **Install from VSIX...**
6. Navigate to and select the downloaded `.vsix` file
7. Reload VS Code when prompted

#### Option B: Via Command Line

```bash
# Download the latest release
curl -LO https://github.com/XORNG/extension-vscode/releases/latest/download/xorng-vscode-0.1.0.vsix

# Install
code --install-extension xorng-vscode-0.1.0.vsix

# For VS Code Insiders
code-insiders --install-extension xorng-vscode-0.1.0.vsix
```

### Method 3: Build from Source

For developers or those who want the latest features:

#### Step 1: Clone the Repository

```bash
git clone https://github.com/XORNG/extension-vscode.git
cd extension-vscode
```

#### Step 2: Install Dependencies

```bash
npm install
```

#### Step 3: Build the Extension

```bash
npm run build
```

#### Step 4: Package as VSIX

```bash
# Install vsce globally if not already installed
npm install -g @vscode/vsce

# Package the extension
npm run package
# or
vsce package

# This creates: xorng-vscode-0.1.0.vsix
```

#### Step 5: Install the VSIX

```bash
code --install-extension xorng-vscode-0.1.0.vsix
```

### Method 4: Development Mode (F5)

For extension development and testing:

1. Open the `extension-vscode` folder in VS Code
2. Press `F5` to launch Extension Development Host
3. A new VS Code window opens with the extension loaded
4. Test the extension in this window

---

## Post-Installation Setup

### Step 1: Verify Installation

1. Open VS Code
2. Check the Extensions sidebar - XORNG should appear as installed
3. Look for the XORNG status bar item (bottom-right)
4. Open Command Palette (`Ctrl+Shift+P`) and type "XORNG" - commands should appear

### Step 2: Configure AI Provider

Choose your preferred AI provider:

#### Option A: GitHub Copilot (Recommended)

No additional configuration needed if you have GitHub Copilot:

1. Click the XORNG status bar item
2. Select **GitHub Copilot**
3. The extension will use your Copilot subscription

#### Option B: Native Providers

For OpenAI, Anthropic, or local models:

1. Open VS Code Settings (`Ctrl+,`)
2. Search for "xorng"
3. Configure the following:

```json
{
  "xorng.provider": "native",
  "xorng.native.provider": "openai",  // or "anthropic" or "local"
  "xorng.native.apiKey": "your-api-key-here"
}
```

### Step 3: Enable Chat Participant

To use `@xorng` in VS Code Chat:

1. Open VS Code Chat (`Ctrl+Shift+I` or click Chat icon)
2. Type `@xorng` followed by your message
3. If prompted, allow the XORNG participant

---

## Configuration Reference

### Settings Overview

Access settings via `File > Preferences > Settings` or `Ctrl+,`, then search for "xorng".

### Provider Settings

| Setting | Default | Options | Description |
|---------|---------|---------|-------------|
| `xorng.provider` | `copilot` | `copilot`, `native`, `claude`, `cursor`, `codex` | Primary AI provider |
| `xorng.copilot.modelFamily` | `gpt-4o` | `gpt-4o`, `gpt-4o-mini`, `o1`, `o1-mini`, `claude-3.5-sonnet` | Model for Copilot |
| `xorng.native.provider` | `openai` | `openai`, `anthropic`, `local` | Native provider type |
| `xorng.native.apiKey` | `""` | String | API key for native provider |

### Memory Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `xorng.memory.enabled` | `true` | Enable in-session conversation memory |
| `xorng.memory.shortTermTTL` | `3600` | Memory TTL in seconds (future use with Core) |

### Other Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `xorng.telemetry.enabled` | `true` | Enable usage telemetry |
| `xorng.logging.level` | `info` | Log level: debug, info, warn, error |

### XORNG Core Settings

Configure XORNG Core behavior:

| Setting | Default | Description |
|---------|---------|-------------|
| `xorng.autoSetup` | `true` | Automatically setup XORNG components on first run |
| `xorng.core.autoStart` | `true` | Automatically start Core after setup |

### Example settings.json

```json
{
  // Use GitHub Copilot with GPT-4o
  "xorng.provider": "copilot",
  "xorng.copilot.modelFamily": "gpt-4o",
  
  // Or use native OpenAI
  // "xorng.provider": "native",
  // "xorng.native.provider": "openai",
  // "xorng.native.apiKey": "sk-your-key",
  
  // Enable memory
  "xorng.memory.enabled": true,
  
  // Auto-setup (enabled by default)
  "xorng.autoSetup": true,
  
  // Auto-start Core (enabled by default)
  "xorng.core.autoStart": true,
  
  // Debug logging
  "xorng.logging.level": "debug"
}
```

---

## Usage Guide

### Chat Participant (@xorng)

Use `@xorng` in VS Code Chat for AI assistance:

```
@xorng Review this code for issues
@xorng What does this function do?
@xorng How can I improve this code?
```

### Slash Commands

| Command | Usage | Description |
|---------|-------|-------------|
| `/review` | `@xorng /review` | Code review for quality and best practices |
| `/security` | `@xorng /security` | Security vulnerability analysis |
| `/explain` | `@xorng /explain` | Code explanation and documentation |
| `/refactor` | `@xorng /refactor` | Code improvement suggestions |
| `/config` | `@xorng /config` | Configuration help |

### VS Code Commands

Access via Command Palette (`Ctrl+Shift+P`):

| Command | Description |
|---------|-------------|
| `XORNG: Select AI Provider` | Choose between providers |
| `XORNG: Toggle Between Copilot and Native Mode` | Quick provider switch |
| `XORNG: Show Status` | View current configuration |
| `XORNG: Clear Conversation Memory` | Reset in-session chat history |
| `XORNG: Start Core` | Start the XORNG Core process |
| `XORNG: Stop Core` | Stop the XORNG Core process |
| `XORNG: Restart Core` | Restart the XORNG Core process |
| `XORNG: Setup Components` | Manually run XORNG setup |
| `XORNG: Update Components` | Pull latest changes from git |
| `XORNG: Show Available Sub-Agents` | View sub-agents (from Core if running) |
| `XORNG: Show Core Logs` | View Core output logs |

### Status Bar

The XORNG status bar item (bottom-right) shows:
- Current provider icon
- Provider name (Copilot/Native/Claude/etc.)
- Running indicator ($(vm-running)) when Core is running
- Click to switch providers

---

## Provider-Specific Setup

### GitHub Copilot

**Prerequisites:**
- Active GitHub Copilot subscription
- GitHub Copilot extension installed in VS Code

**Configuration:**
```json
{
  "xorng.provider": "copilot",
  "xorng.copilot.modelFamily": "gpt-4o"
}
```

**Available Models:**
- `gpt-4o` - Best balance of speed and quality
- `gpt-4o-mini` - Faster, lower cost
- `o1` - Advanced reasoning
- `o1-mini` - Faster reasoning
- `claude-3.5-sonnet` - Claude via Copilot

### OpenAI (Native)

**Prerequisites:**
- OpenAI API key from [platform.openai.com](https://platform.openai.com)

**Configuration:**
```json
{
  "xorng.provider": "native",
  "xorng.native.provider": "openai",
  "xorng.native.apiKey": "sk-your-openai-api-key"
}
```

### Anthropic Claude (Native)

**Prerequisites:**
- Anthropic API key from [console.anthropic.com](https://console.anthropic.com)

**Configuration:**
```json
{
  "xorng.provider": "native",
  "xorng.native.provider": "anthropic",
  "xorng.native.apiKey": "sk-ant-your-anthropic-api-key"
}
```

### Local Models (Ollama)

**Prerequisites:**
- [Ollama](https://ollama.ai) installed and running
- At least one model pulled (e.g., `ollama pull codellama`)

**Configuration:**
```json
{
  "xorng.provider": "native",
  "xorng.native.provider": "local"
}
```

**Starting Ollama:**
```bash
# Start Ollama server (default: http://localhost:11434)
ollama serve

# Pull a code-focused model
ollama pull codellama
# or
ollama pull deepseek-coder
```

---

## XORNG Core Integration

### Overview

XORNG Core runs as a **local child process** managed by the extension. This architecture provides:
- **Zero Configuration**: Core is automatically set up and started
- **Shared Copilot Access**: Core uses Copilot models via IPC proxy
- **Process Isolation**: Core runs in a separate Node.js process
- **Automatic Updates**: Git pull on extension restart keeps components current

### How It Works

```
┌─────────────────────────────────────────────────────────────┐
│                    VS Code Extension                         │
│  ┌────────────────────────────────────────────────────┐     │
│  │              LocalOrchestrator                      │     │
│  │  - Spawns Core via fork() with IPC                 │     │
│  │  - Proxies LLM requests to vscode.lm API           │     │
│  │  - Manages Core lifecycle                          │     │
│  └─────────────────────┬──────────────────────────────┘     │
│                        │ IPC Channel                         │
└────────────────────────┼────────────────────────────────────┘
                         │
┌────────────────────────┼────────────────────────────────────┐
│                        ▼                                     │
│                   XORNG Core (Child Process)                 │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  - Receives requests via IPC                         │    │
│  │  - Routes to sub-agents                              │    │
│  │  - Requests LLM via IPC (uses extension's Copilot)   │    │
│  │  - Manages memory system                             │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Auto-Setup Process

On first activation, the extension automatically:

1. **Creates storage directory** in VS Code's globalStorageUri
2. **Clones repositories**:
   - `core` - Main orchestration engine
   - `validator-code-review` - Code review sub-agent
   - `validator-security` - Security analysis sub-agent
   - `knowledge-best-practices` - Best practices knowledge
   - `knowledge-documentation` - Documentation knowledge
3. **Installs dependencies** (`npm install` for each)
4. **Builds components** (`npm run build` for each)
5. **Starts Core** automatically (if `xorng.core.autoStart` is true)

### Manual Setup

If auto-setup is disabled or you need to re-run setup:

1. Open Command Palette (`Ctrl+Shift+P`)
2. Run `XORNG: Setup Components`
3. Wait for setup to complete

### Starting/Stopping Core

#### Automatic (Default)

Core starts automatically on extension activation if:
- Setup has been completed
- `xorng.core.autoStart` is `true` (default)

#### Manual Control

| Command | Description |
|---------|-------------|
| `XORNG: Start Core` | Start the Core process |
| `XORNG: Stop Core` | Stop the Core process gracefully |
| `XORNG: Restart Core` | Stop and restart Core |

### Updating Components

The extension automatically pulls latest changes from git on restart. To manually update:

1. Run `XORNG: Update Components`
2. Core will be stopped, repositories updated, rebuilt, and Core restarted

### Core Status

The status bar shows Core status:
- **No indicator**: Core is not running
- **$(vm-running) icon**: Core is running

### Viewing Core Logs

1. Run `XORNG: Show Core Logs`
2. The XORNG Core output channel opens with logs

### Disabling Core

To use the extension without Core:

```json
{
  "xorng.autoSetup": false,
  "xorng.core.autoStart": false
}
```

The extension will work in standalone mode using only the AI provider directly.

---

## Troubleshooting

### Extension Not Appearing

**Problem:** Extension doesn't show in Extensions list after installation.

**Solutions:**
1. Restart VS Code completely
2. Check VS Code version: `Help > About` (requires 1.95.0+)
3. Reinstall the extension

### Chat Participant Not Working

**Problem:** `@xorng` doesn't respond in chat.

**Solutions:**
1. Ensure GitHub Copilot Chat is installed
2. Check the Output panel (`View > Output`) for errors
3. Run `Developer: Reload Window` from Command Palette
4. Verify the extension is enabled in Extensions sidebar

### Copilot Provider Errors

**Problem:** "No models available" or similar errors.

**Solutions:**
1. Verify GitHub Copilot subscription is active
2. Sign out and sign back into GitHub
3. Try a different model family in settings
4. Check Copilot status at [githubstatus.com](https://githubstatus.com)

### Native Provider API Errors

**Problem:** API key errors or rate limiting.

**Solutions:**
1. Verify API key is correct and has sufficient credits
2. Check API key permissions (needs chat/completions access)
3. Wait if rate limited, or upgrade API plan
4. Try a different model

### Local Model Connection Issues

**Problem:** Cannot connect to Ollama.

**Solutions:**
1. Verify Ollama is running: `curl http://localhost:11434/api/tags`
2. Check firewall settings
3. Ensure at least one model is installed: `ollama list`
4. Restart Ollama: `ollama serve`

### Extension Crashes or Freezes

**Problem:** VS Code becomes unresponsive.

**Solutions:**
1. Check memory usage in Task Manager
2. Disable and re-enable the extension
3. Clear memory cache: `XORNG: Clear Memory Cache`
4. Check logs: `Help > Toggle Developer Tools > Console`

### XORNG Core Issues

**Problem:** Core fails to start.

**Solutions:**
1. Check Core logs: Run `XORNG: Show Core Logs`
2. Verify setup completed: Run `XORNG: Setup Components`
3. Check Node.js is installed: `node --version` (requires 18+)
4. Ensure npm dependencies installed properly

**Problem:** "Core path not set" error.

**Solutions:**
1. Run `XORNG: Setup Components` to install Core
2. Verify auto-setup is enabled: `"xorng.autoSetup": true`
3. Check VS Code has write access to globalStorageUri

**Problem:** Core exits unexpectedly.

**Solutions:**
1. View logs: `XORNG: Show Core Logs`
2. Restart Core: `XORNG: Restart Core`
3. Re-run setup: `XORNG: Setup Components`
4. Check for conflicting processes

**Problem:** Setup fails during git clone.

**Solutions:**
1. Verify internet connectivity
2. Check git is installed: `git --version`
3. Ensure GitHub is accessible
4. Try running setup again

**Problem:** Setup fails during npm install/build.

**Solutions:**
1. Check Node.js version: `node --version` (requires 18+)
2. Clear npm cache: `npm cache clean --force`
3. Delete and re-run setup
4. Check disk space

### Viewing Logs

```bash
# VS Code Developer Tools Console
Help > Toggle Developer Tools > Console tab

# Output Panel
View > Output > Select "XORNG" or "XORNG Core" from dropdown

# Core Logs via Command
Run "XORNG: Show Core Logs" from Command Palette
```

---

## Updating the Extension

### From Marketplace

1. VS Code auto-updates extensions by default
2. Or manually: Extensions sidebar > Click update icon on XORNG

### From Source

```bash
cd extension-vscode
git pull
npm install
npm run build
npm run package
code --install-extension xorng-vscode-*.vsix --force
```

---

## Uninstalling

### Via VS Code

1. Open Extensions (`Ctrl+Shift+X`)
2. Find XORNG
3. Click **Uninstall**

### Via Command Line

```bash
code --uninstall-extension xorng.xorng-vscode
```

### Clean Up Settings

Remove XORNG settings from your settings.json if desired.

---

## Keyboard Shortcuts

Default shortcuts (customizable in Keyboard Shortcuts):

| Action | Windows/Linux | macOS |
|--------|---------------|-------|
| Open Chat | `Ctrl+Shift+I` | `Cmd+Shift+I` |
| Command Palette | `Ctrl+Shift+P` | `Cmd+Shift+P` |
| Extensions | `Ctrl+Shift+X` | `Cmd+Shift+X` |
| Settings | `Ctrl+,` | `Cmd+,` |

---

## Security Considerations

### API Key Storage

- API keys configured in VS Code settings are stored in your local settings file
- For enhanced security, use environment variables:

```bash
# Set in your shell profile (.bashrc, .zshrc, etc.)
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
```

### Data Privacy

- XORNG respects VS Code's telemetry settings
- Code sent to AI providers follows their respective privacy policies
- Local models (Ollama) keep all data on your machine

---

## Next Steps

- [Main Installation Guide](./INSTALLATION.md) - Full XORNG setup
- [Node Configuration](./NODE_CONFIGURATION.md) - Advanced AI provider setup
- [Automation Guide](./AUTOMATION.md) - Self-improvement features
- [Extension README](https://github.com/XORNG/extension-vscode) - Source and contributing

---

## Support

- **Issues:** [GitHub Issues](https://github.com/XORNG/extension-vscode/issues)
- **Discussions:** [GitHub Discussions](https://github.com/XORNG/extension-vscode/discussions)
- **Documentation:** [XORNG Docs](https://github.com/XORNG/documentation)
