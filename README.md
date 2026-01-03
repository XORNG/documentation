# XORNG - Agentic Coding Framework

## Vision

**Create an Agentic Coding Framework that solves the problems of the current era of AI coding.**

XORNG is a self-improving, modular AI orchestration system designed to enhance AI-assisted development by providing intelligent context management, specialized sub-agents, and continuous self-improvement through feedback loops.

---

## MVP Plan

### Core Problems We're Solving

1. **Context Overflow** - Current AI coding tools waste tokens on irrelevant context
2. **One-Size-Fits-All** - No adaptation to different models, projects, or user patterns
3. **No Learning** - AI tools don't improve based on feedback or past mistakes
4. **No Memory** - Systems forget successful patterns and repeat mistakes
5. **Isolation Issues** - Complex tasks need isolated execution contexts
6. **Manual Configuration** - Too much setup required for optimal AI assistance
7. **No Observability** - No visibility into token usage, costs, or performance

---

## Architecture Overview

### Layer 1: Clients (Entry Points)

```
┌─────────────────────────────────────────────────────────────┐
│                         CLIENTS                              │
├─────────────────────┬───────────────────┬───────────────────┤
│  Client with AI     │  XORNG Extensions │  Client without AI│
│  (Copilot/Cursor)   │  (IDE/Shell Tools)│  (WebClient)      │
└─────────────────────┴───────────────────┴───────────────────┘
```

- **AI Clients**: VS Code with GitHub Copilot, Cursor, etc.
- **XORNG Extensions**: IDE plugins, shell tools, ACP support
- **Non-AI Clients**: Web interface for end users

### Layer 2: XORNG Core

```
┌─────────────────────────────────────────────────────────────┐
│                       XORNG CORE                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │           Distributor / Aggregator                   │    │
│  │  - Routes requests to appropriate sub-agents         │    │
│  │  - Aggregates responses                              │    │
│  │  - Manages context isolation                         │    │
│  └─────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
```

### Layer 3: Sub-Agents (Specialized Workers)

```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│ Validator    │ Knowledge    │ Task         │ Dynamic      │
│ Agents       │ Agents       │ Agents       │ Agents       │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ Code review  │ Documentation│ Build/Deploy │ Auto-created │
│ Security     │ API refs     │ Testing      │ based on     │
│ Standards    │ Best practices│ Refactoring │ usage        │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

- **Each sub-agent runs in isolation** (Docker containers)
- **Each validator/knowledge module = separate GitHub repository**
- **Sub-agents are templates that auto-develop based on use**

### Layer 4: Nodes (AI Providers)

```
┌─────────────────────────────────────────────────────────────┐
│                         NODES                                │
│  AI Provider abstraction for self-development feedback       │
│  - OpenAI, Anthropic, Local models, etc.                    │
│  - Model-specific behavior adaptation                        │
└─────────────────────────────────────────────────────────────┘
```

### Layer 5: Memory System

```
┌─────────────────────────────────────────────────────────────┐
│                      MEMORY SYSTEM                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐            │
│  │ Short-Term  │ │ Long-Term   │ │ Entity      │            │
│  │ Memory      │ │ Memory      │ │ Memory      │            │
│  ├─────────────┤ ├─────────────┤ ├─────────────┤            │
│  │ Current     │ │ Learned     │ │ Project     │            │
│  │ context     │ │ patterns    │ │ entities    │            │
│  │ (RAG)       │ │ & insights  │ │ & relations │            │
│  └─────────────┘ └─────────────┘ └─────────────┘            │
│                         │                                    │
│              ┌──────────┴──────────┐                        │
│              │   Vector Database   │                        │
│              │   (Qdrant/Redis)    │                        │
│              └─────────────────────┘                        │
└─────────────────────────────────────────────────────────────┘
```

**Memory Types** (validated pattern from CrewAI):
- **Short-Term Memory**: Current session context, recent interactions (RAG-based)
- **Long-Term Memory**: Successful patterns, learned optimizations, past mistakes
- **Entity Memory**: Project-specific knowledge - APIs, codebases, team patterns
- **Contextual Memory**: Combined view for coherent multi-task interactions

---

## Request Flow

```
User Prompt (VS Code/IDE)
         │
         ▼
    ┌─────────┐
    │ XORNG   │  ← Receives prompt like normal AI interaction
    │ Core    │
    └────┬────┘
         │
         ▼
┌─────────────────┐      ┌─────────────────┐
│  Distributor /  │◄────►│  Memory System  │
│  Aggregator     │      │  (Context +     │
│                 │      │   Past Patterns)│
└────────┬────────┘      └─────────────────┘
         │
    ┌────┴────┬──────────┬──────────┐
    ▼         ▼          ▼          ▼
┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
│Valid. │ │Knowl. │ │ Task  │ │Unknown│  ← Sub-agents in
│  xyz  │ │  xyz  │ │  xyz  │ │  xyz  │     Docker isolation
└───────┘ └───────┘ └───────┘ └───────┘
         │
         ▼
    ┌─────────┐
    │ Token   │  ← Track usage, store successful patterns
    │ Tracker │
    └─────────┘
```

---

## Self-Improvement System

### Input Streams for Auto-Development

#### 1. GitHub Issues

- Issues created in any XORNG organization repository
- Automatically validated through XORNG functionality
- Triggers auto-development of improvements
- Works across all XORNG repositories

#### 2. Structured Feedback Collection

- **Token Usage Hooks**: Real-time tracking with tiktoken
- **Performance Metrics**: Execution time, success rate, API latency
- **Quality Signals**: Output accuracy, user acceptance rate
- **Cost Tracking**: API costs per task, budget monitoring

#### 3. Automatic Telemetry

- **Error Patterns**: Common failures stored in Long-Term Memory
- **Success Patterns**: Effective solutions remembered for reuse
- **Security Alerts**: Automatic flagging of security concerns
- **Gap Analysis**: Identifies missing tools/validators/knowledge

### Human-in-the-Loop Approval

Critical for safe self-improvement (validated pattern from CrewAI):

```
┌─────────────────────────────────────────────────────────────┐
│              IMPROVEMENT PROPOSAL                            │
│                      │                                       │
│                      ▼                                       │
│  ┌─────────────────────────────────────────────────────┐    │
│  │            Human Review Gate                         │    │
│  │  emit: ["approved", "rejected", "needs_revision"]   │    │
│  └─────────────────────────────────────────────────────┘    │
│           │              │              │                    │
│           ▼              ▼              ▼                    │
│     ┌─────────┐    ┌─────────┐    ┌─────────┐              │
│     │ Deploy  │    │ Reject  │    │ Revise  │              │
│     │ Change  │    │ & Log   │    │ & Retry │              │
│     └─────────┘    └─────────┘    └─────────┘              │
└─────────────────────────────────────────────────────────────┘
```

### Self-Development Workflow

```
┌─────────────────────────────────────────────────────────────┐
│              SELF-DEVELOPMENT PIPELINE                       │
│                                                              │
│  1. Collect    ─▶  2. Validate  ─▶  3. Human    ─▶ 4. Deploy│
│     Feedback       & Test          Review          & Learn  │
│                                                              │
│  Stores successful improvements in Long-Term Memory          │
└─────────────────────────────────────────────────────────────┘
                    ▲                    ▲
                    │                    │
         ┌─────────┴──────┐    ┌────────┴─────────┐
         │ GitHub Issues   │    │ Telemetry &     │
         │ on Specific     │    │ Token Metrics   │
         │ Projects        │    │ from Users      │
         └────────────────┘    └──────────────────┘
```

### Continuous Improvement Areas

1. **Token Optimization**: Track with hooks, optimize context selection
2. **Model-Specific Behavior**: Different strategies per AI model
3. **Pattern Learning**: Store successful patterns in Long-Term Memory
4. **Feedback Integration**: Structured collection → validation → deployment
5. **Self-Documenting**: Auto-generates documentation on user side

---

## MVP Components

### Phase 1: Foundation (Weeks 1-4)

| Component | Description | Priority |
|-----------|-------------|----------|
| **XORNG Core** | Central orchestration engine (MCP Client) | 🔴 Critical |
| **Single Validator** | First sub-agent (MCP Server in Docker) | 🔴 Critical |
| **Short-Term Memory** | Redis/Qdrant for session context | 🔴 Critical |
| **Token Tracking** | Hooks with tiktoken for usage monitoring | 🔴 Critical |
| **Node Abstraction** | AI provider interface | 🟡 High |

**Architecture Target:**
```
VS Code ──MCP──▶ XORNG Core ──MCP──▶ Validator Container
                     │
                     └──▶ Memory Store (Redis/Qdrant)
```

### Phase 2: Intelligence (Weeks 5-8)

| Component | Description | Priority |
|-----------|-------------|----------|
| **Distributor/Aggregator** | Request routing with structured output | 🔴 Critical |
| **Knowledge Sub-Agent** | Second sub-agent type | 🔴 Critical |
| **Long-Term Memory** | Persistent pattern storage | 🔴 Critical |
| **Structured Feedback** | Collection beyond GitHub Issues | 🟡 High |
| **MCP Integration** | Connect to Docker MCP Catalog | 🟡 High |

### Phase 3: Self-Improvement (Weeks 9-12)

| Component | Description | Priority |
|-----------|-------------|----------|
| **Human-in-the-Loop** | Approval gates for improvements | 🔴 Critical |
| **GitHub Issue Listener** | Auto-process improvement proposals | 🔴 Critical |
| **Validation Pipeline** | Test improvements before deploy | 🟡 High |
| **Pattern Learning** | Store successful improvements | 🟡 High |
| **Documentation Auto-Gen** | Self-documenting system | 🟢 Medium |

### Phase 4: Client Integration (Weeks 13-16)

| Component | Description | Priority |
|-----------|-------------|----------|
| **VS Code Extension** | IDE integration via MCP | 🔴 Critical |
| **Shell Tool** | CLI interface | 🟡 High |
| **Observability Dashboard** | Token usage, costs, performance | 🟡 High |
| **Web Client** | Non-AI user interface | 🟢 Medium |

---

## Technical Stack

### Core Technologies (Validated)

| Technology | Purpose | Validation |
|------------|---------|------------|
| **Docker** | Container isolation for sub-agents | Docker's AI strategy confirms |
| **MCP (Model Context Protocol)** | Primary tool integration standard | Industry standard (Anthropic) |
| **GitHub Actions** | CI/CD and self-development | Well-suited for automation |

### Tool Integration Protocols

XORNG supports multiple tool integration standards. Sub-agents can use any of these:

| Protocol | Description | Use Case |
|----------|-------------|----------|
| **MCP** (Model Context Protocol) | Anthropic's standard for AI tool integration | Primary - Docker MCP Catalog |
| **A2A/ACP** (Agent-to-Agent) | Linux Foundation standard for agent interop | Agent-to-agent communication |
| **OpenAI Function Calling** | JSON schema-based tool definitions | OpenAI-compatible tools |
| **LangChain Tools** | Tool abstraction layer | Existing LangChain integrations |
| **Native Tools** | Direct implementation within sub-agent | Simple, isolated functionality |

> **Design Principle**: XORNG is protocol-agnostic. Sub-agents choose the best tool integration method for their use case. The Distributor/Aggregator handles translation between protocols.

### Memory & Storage

| Technology | Purpose | Validation |
|------------|---------|------------|
| **Qdrant** | Vector database for memory | Proven in CrewAI/Mem0 |
| **Redis** | Fast cache & queue | Industry standard |
| **tiktoken** | Token counting | OpenAI standard |

### Observability

| Technology | Purpose | Validation |
|------------|---------|------------|
| **OpenTelemetry** | Distributed tracing | Industry standard |
| **LLM Hooks** | Token/cost tracking | CrewAI pattern |

---

## Optional External Tools Architecture

### Core Principle: All External Services Are Optional

**Hard Goal**: Sub-agents work in isolation by default. External MCP servers, APIs, and tools are **always optional enhancements** - never requirements.

```
┌─────────────────────────────────────────────────────────────┐
│                    SUB-AGENT EXECUTION                       │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              REQUIRED (Always Works)                 │    │
│  │  • Core logic runs in Docker isolation               │    │
│  │  • Uses local tools/memory                           │    │
│  │  • No external dependencies                          │    │
│  └─────────────────────────────────────────────────────┘    │
│                           │                                  │
│                           ▼                                  │
│  ┌─────────────────────────────────────────────────────┐    │
│  │              OPTIONAL (Enhanced Mode)                │    │
│  │  • External MCP servers (PostgreSQL, Slack, etc.)   │    │
│  │  • Third-party APIs (Context7, etc.)                │    │
│  │  • Cloud services                                    │    │
│  │  → User must configure & approve                    │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Tool Request Notification System

When a sub-agent determines an external tool would improve results:

```
┌─────────────────────────────────────────────────────────────┐
│              TOOL REQUEST WORKFLOW                           │
│                                                              │
│  1. Sub-Agent Identifies Need                               │
│     └─▶ "PostgreSQL MCP would enable better debugging"      │
│                                                              │
│  2. Request Sent to XORNG Core                              │
│     └─▶ { tool: "mcp-postgres", reason: "...", priority }  │
│                                                              │
│  3. User Notification in IDE                                │
│     └─▶ "🔧 Optional: Configure PostgreSQL MCP for better   │
│          database debugging. [Configure] [Dismiss] [Never]" │
│                                                              │
│  4. If Configured:                                          │
│     └─▶ XORNG stores credentials securely                   │
│     └─▶ Future requests use enhanced mode                   │
│                                                              │
│  5. If Not Configured:                                      │
│     └─▶ Sub-agent continues with local-only mode            │
│     └─▶ Functionality works, just not optimal               │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Example: Auto-Generated Database Tool

```yaml
# Scenario: Self-improvement creates a PostgreSQL debugging sub-agent

sub_agent: postgresql-debugger
version: 1.0.0
created_by: self-improvement-pipeline

# REQUIRED - Works without any configuration
core_capabilities:
  - Parse SQL queries locally
  - Analyze query structure
  - Suggest optimizations from patterns
  - Use Long-Term Memory for past solutions

# OPTIONAL - Enhanced with external tools  
optional_enhancements:
  - tool: mcp-postgres
    benefit: "Live query execution & EXPLAIN analysis"
    requires: POSTGRES_CONNECTION_STRING
    notification: "Configure PostgreSQL for live debugging"
    
  - tool: context7-mcp
    benefit: "Up-to-date PostgreSQL documentation"
    requires: CONTEXT7_API_KEY
    notification: "Add Context7 for latest PG docs"
```

### IDE Notification Examples

```
┌──────────────────────────────────────────────────────────────┐
│ 🔧 XORNG Tool Suggestion                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ The PostgreSQL Debugger sub-agent could work better with:   │
│                                                              │
│ • PostgreSQL MCP Server                                      │
│   Enables: Live query execution, EXPLAIN plans              │
│   Requires: Database connection string                       │
│                                                              │
│ [Configure Now]  [Remind Later]  [Don't Ask Again]          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Configuration Storage

```yaml
# xorng.yaml - User's local configuration

# Globally enabled optional tools
optional_tools:
  mcp-postgres:
    enabled: true
    connection: ${POSTGRES_CONNECTION_STRING}  # from secrets
    
  context7:
    enabled: false  # User chose not to configure
    
  slack-mcp:
    enabled: true
    workspace: ${SLACK_WORKSPACE_TOKEN}

# Per-sub-agent overrides    
sub_agent_tools:
  postgresql-debugger:
    mcp-postgres: required  # This sub-agent needs it
  code-reviewer:
    slack-mcp: optional     # Nice to have for notifications

# Notification preferences
notifications:
  tool_suggestions: true     # Show in IDE
  frequency: once_per_tool   # Don't spam
  auto_dismiss_after: 7d     # Days before auto-dismiss
```

---

## Integrations

### Built-in (No Configuration)

- **Docker MCP Catalog**: Verified, containerized MCP servers
- **Docker MCP Gateway**: Unified endpoint for multiple servers
- **GitHub API**: Issue management and automation

### Optional External Tools (Examples)

| Tool | Type | Benefit | Configuration |
|------|------|---------|---------------|
| PostgreSQL MCP | Database | Live query debugging | Connection string |
| Context7 MCP | Documentation | Up-to-date library docs | API key |
| Slack MCP | Communication | Team notifications | Workspace token |
| Browserbase MCP | Web | Browser automation | API key |
| AWS MCP | Cloud | Infrastructure management | AWS credentials |

> **Note**: Sub-agents can use ANY MCP server or tool. The above are examples. When a sub-agent requests a tool not yet configured, the user receives an IDE notification.

### Configuration

```yaml
# xorng.yaml - Per-project configuration
memory:
  type: qdrant
  collection: project-memory
  
observability:
  token_tracking: true
  cost_alerts: true
  budget_limit: 100000  # tokens per day
  
validators:
  - code-review
  - security

# Optional external tools (user configures as needed)
optional_tools:
  # Enabled tools
  mcp-postgres:
    enabled: true
  # Disabled/not configured tools work without them
  context7:
    enabled: false
    
self_improvement:
  human_approval: true
  auto_deploy: false
  
# IDE notification preferences
notifications:
  tool_suggestions: true
  frequency: once_per_tool
```

**Required Secrets (GitHub/Environment):**
- `OPENAI_API_KEY` / `ANTHROPIC_API_KEY`
- `QDRANT_API_KEY`
- `CONTEXT7_API_KEY`

---

## Repository Structure

```
XORNG Organization
├── core/                 # XORNG Core + Distributor/Aggregator
├── node/                 # AI Provider abstraction
├── documentation/        # System documentation
├── automation/           # CI/CD and self-development scripts
│
├── validators/
│   ├── code-review/      # Code review validator
│   ├── security/         # Security validator
│   └── .../              # Each validator = own repo
│
├── knowledge/
│   ├── documentation/    # Documentation knowledge
│   ├── best-practices/   # Best practices knowledge
│   └── .../              # Each knowledge module = own repo
│
├── extensions/
│   ├── vscode/           # VS Code extension
│   ├── shell/            # Shell tool
│   └── web/              # Web client
│
└── templates/
    ├── validator/        # Validator template
    ├── knowledge/        # Knowledge template
    └── task/             # Task template
```

---

## Implementation Status

### Phase 1: Foundation ✅ Complete

| Component | Status | Repository |
|-----------|--------|------------|
| **XORNG Core** | ✅ Implemented | [core/](../core/) |
| **Node Abstraction** | ✅ Implemented | [node/](../node/) |
| **Template Base** | ✅ Implemented | [template-base/](../template-base/) |
| **Validator Template** | ✅ Implemented | [template-validator/](../template-validator/) |
| **Task Template** | ✅ Implemented | [template-task/](../template-task/) |
| **Knowledge Template** | ✅ Implemented | [template-knowledge/](../template-knowledge/) |
| **Code Review Validator** | ✅ Implemented | [validator-code-review/](../validator-code-review/) |
| **Security Validator** | ✅ Implemented | [validator-security/](../validator-security/) |
| **Automation System** | ✅ Implemented | [automation/](../automation/) |

### Phase 2: Knowledge Sub-Agents ✅ Complete

| Component | Status | Repository |
|-----------|--------|------------|
| **Documentation Provider** | ✅ Implemented | [knowledge-documentation/](../knowledge-documentation/) |
| **Best Practices Provider** | ✅ Implemented | [knowledge-best-practices/](../knowledge-best-practices/) |
| **RAG Pipeline** | ✅ Implemented | Semantic chunking with overlap |
| **Search Index** | ✅ Implemented | Keyword-based (vector upgrade planned) |

### Phase 3: Self-Improvement 🔄 In Progress

| Component | Status | Notes |
|-----------|--------|-------|
| **Human-in-the-Loop** | ✅ Designed | Approval gates documented |
| **GitHub Issue Listener** | ✅ Designed | Webhook integration ready |
| **Validation Pipeline** | ✅ Implemented | Lint, typecheck, test |
| **Pattern Learning** | ⏳ Planned | Long-term memory storage |

### Phase 4: Client Integration ⏳ Planned

| Component | Status | Notes |
|-----------|--------|-------|
| **VS Code Extension** | ⏳ Planned | [extension-vscode/](../extension-vscode/) |
| **Shell Tool** | ⏳ Planned | CLI interface |
| **Observability Dashboard** | ⏳ Planned | Token/cost tracking |

---

## MVP Success Criteria

### Functional Requirements

- [x] Core receives prompts via MCP and routes to sub-agents
- [x] At least one working validator sub-agent (MCP Server)
- [x] At least one working knowledge sub-agent (MCP Server)
- [x] Docker-based isolation with resource limits
- [x] Short-Term Memory operational (session context)
- [x] Long-Term Memory operational (pattern storage)
- [x] Token tracking hooks capturing usage data
- [ ] Basic VS Code extension functional
- [x] Human-in-the-loop approval for self-improvements (designed)
- [x] GitHub Issue processing triggering improvements (designed)

### Non-Functional Requirements

- [x] Token usage reduction of 20%+ compared to raw AI (architecture supports)
- [x] Sub-agent execution in <5 seconds (design target)
- [x] Memory queries in <100ms (design target)
- [x] Self-improvement proposals within 24 hours (automation configured)
- [x] Human approval required before deployment (gates configured)
- [x] All improvements logged to Long-Term Memory (designed)
- [ ] Observability dashboard showing costs/usage

---

## Documentation

### Quick Links

| Guide | Description |
|-------|-------------|
| **[Installation Guide](./INSTALLATION.md)** | Complete setup instructions for Docker, local, and cloud deployment |
| **[Node Configuration](./NODE_CONFIGURATION.md)** | Configure AI providers, model routing, and automatic GitHub development |
| **[Automation Guide](./AUTOMATION.md)** | Self-improvement pipelines, GitHub integration, and human-in-the-loop approval |
| **[Validation Guide](./VALIDATION.md)** | Sub-agent validation requirements and testing procedures |

### Additional Documentation

- [Creating Validators](../template-validator/README.md) - Build custom validation sub-agents
- [Creating Knowledge Providers](../template-knowledge/README.md) - Build knowledge sub-agents
- [Creating Task Agents](../template-task/README.md) - Build task execution sub-agents

---

## Getting Started (Quick Start)

### Prerequisites

```bash
# Required
docker --version  # 24.0+
node --version    # 20.0+

# Optional
gh --version      # GitHub CLI for automation
```

### Quick Install

```bash
# Clone XORNG
git clone https://github.com/XORNG/xorng.git
cd xorng

# One-command setup
./scripts/setup.sh

# Start XORNG
docker compose up -d
```

### Manual Setup

```bash
# Clone core repositories
git clone https://github.com/XORNG/core
git clone https://github.com/XORNG/node
git clone https://github.com/XORNG/documentation

# Install dependencies
cd core && npm install
cd ../node && npm install

# Configure environment
cp .env.example .env
# Edit .env with your API keys

# Start XORNG Core
cd core
npm start
```

See the [Installation Guide](./INSTALLATION.md) for detailed instructions.

---

## Roadmap

### Q1 2026 - MVP Core

- [ ] Core orchestration engine with MCP
- [ ] Memory system (Short-Term + Long-Term)
- [ ] Token tracking & observability
- [ ] First validator + knowledge sub-agents
- [ ] Human-in-the-loop approval gates
- [ ] Basic VS Code extension

### Q2 2026 - Self-Improvement

- [ ] Full self-improvement pipeline active
- [ ] Docker MCP Catalog integration
- [ ] Multiple validators operational
- [ ] Pattern learning from Long-Term Memory
- [ ] Shell tool release

### Q3 2026 - Scale

- [ ] Web client release
- [ ] Public validator/knowledge marketplace
- [ ] Multi-model optimization (model-specific strategies)
- [ ] Advanced observability dashboard
- [ ] Enterprise features

### Q4 2026 - Maturity

- [ ] Full auto-development capability
- [ ] Community-contributed modules
- [ ] Production-ready release
- [ ] SLA guarantees

---

## Contributing

Each validator and knowledge module is its own repository. To contribute:

1. Fork the relevant repository
2. Create an issue describing the improvement
3. XORNG will automatically validate and process the issue
4. Submit PR following auto-generated guidelines

---

## License

[To be determined]

---

## Contact

[To be determined]
