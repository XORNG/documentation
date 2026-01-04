# XORNG Automation Configuration Guide

Configure XORNG's self-improvement system for automatic codebase enhancements, issue resolution, and continuous improvement.

## Overview

XORNG's automation system enables:

- **Self-Improvement** - Automatic code enhancements based on feedback
- **Issue Resolution** - Auto-process issues from all GitHub repositories
- **Continuous Learning** - Store successful patterns in long-term memory
- **Human-in-the-Loop** - Approval gates for safe deployment
- **VS Code Feedback Loop** - Real-time feedback from extension users
- **Automatic Deployment** - Zero-maintenance server deployment via GitHub Actions

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      XORNG AUTOMATION SYSTEM                            │
│                                                                         │
│  ┌────────────────────────────────────────────────────────────────┐    │
│  │                    WEBHOOK SERVER (VServer)                     │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │    │
│  │  │   GitHub     │  │  VS Code     │  │   Feedback   │          │    │
│  │  │   Webhooks   │  │  Extension   │  │   Service    │          │    │
│  │  │   (all repos)│  │  Endpoints   │  │              │          │    │
│  │  └──────┬───────┘  └──────┬───────┘  └──────────────┘          │    │
│  │         │                 │                                     │    │
│  │         ▼                 ▼                                     │    │
│  │  ┌─────────────────────────────────────────────────────────┐   │    │
│  │  │              Issue Processor (All Issues)               │   │    │
│  │  │  - No label filtering (testing phase)                   │   │    │
│  │  │  - All repositories in organization                     │   │    │
│  │  │  - Auto-discovers new repositories                      │   │    │
│  │  └─────────────────────────────────────────────────────────┘   │    │
│  └────────────────────────────────────────────────────────────────┘    │
│                              │                                          │
│                              ▼                                          │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │   INPUTS     │    │  PROCESSING  │    │   OUTPUTS    │              │
│  ├──────────────┤    ├──────────────┤    ├──────────────┤              │
│  │ All Issues   │───▶│ AI Analysis  │───▶│ Pull Request │              │
│  │ Error Logs   │───▶│ Validation   │───▶│ Config Update│              │
│  │ VS Code Data │───▶│ Human Review │───▶│ New Sub-Agent│              │
│  │ Telemetry    │───▶│ Testing      │───▶│ Documentation│              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
│                              │                                          │
│                              ▼                                          │
│                    ┌──────────────────┐                                │
│                    │  Long-Term Memory │                                │
│                    │  (Pattern Storage)│                                │
│                    └──────────────────┘                                │
└─────────────────────────────────────────────────────────────────────────┘

                     Automatic Deployment via GitHub Actions
                     ┌─────────────────────────────────────┐
                     │ Push to main → Build → Deploy → Run │
                     │ Daily auto-update checks            │
                     │ Zero maintenance required           │
                     └─────────────────────────────────────┘
```

---

## Quick Start

### 1. VServer Setup (One-Time)

Run the setup script on your Debian VServer:

```bash
# Download and run setup script
curl -fsSL https://raw.githubusercontent.com/XORNG/automation/main/scripts/vserver-setup.sh | sudo bash
```

This installs Docker, creates the deploy user, and configures the firewall.
**This is the ONLY manual step on the server** - everything else is automated!

### 2. Configure GitHub Secrets

Add these secrets to your GitHub repository (Settings → Secrets → Actions):

| Secret | Description |
|--------|-------------|
| `VSERVER_HOST` | Your VServer IP or hostname |
| `VSERVER_USER` | `deploy` (created by setup script) |
| `VSERVER_SSH_KEY` | SSH private key for deployment |
| `VSERVER_PORT` | SSH port (default: 22) |
| `GH_AUTOMATION_TOKEN` | GitHub token with `repo` and `admin:org` scopes |
| `WEBHOOK_SECRET` | Random secret for webhook verification |
| `GITHUB_ORG` | Your GitHub organization name (e.g., `XORNG`) |

**For domain + automatic HTTPS (recommended):**

| Secret | Description |
|--------|-------------|
| `AUTOMATION_DOMAIN` | Your domain (e.g., `automation.yourdomain.com`) |
| `ACME_EMAIL` | Email for Let's Encrypt notifications (optional) |

> **Note:** If `AUTOMATION_DOMAIN` is set, GitHub Actions will automatically:
> - Deploy Traefik as a reverse proxy
> - Provision SSL certificates via Let's Encrypt  
> - Configure HTTP → HTTPS redirect
> - Route traffic to the automation server
>
> **You don't need to install or configure anything on the server!**

### 3. Generate SSH Keys

```bash
# On your local machine
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/xorng-deploy

# Copy public key to VServer
ssh-copy-id -i ~/.ssh/xorng-deploy.pub deploy@YOUR_VSERVER_IP

# Add private key content to GitHub secret VSERVER_SSH_KEY
cat ~/.ssh/xorng-deploy
```

### 4. Point DNS (if using domain)

Point your domain's A record to your VServer IP:
```
automation.yourdomain.com  →  A  →  YOUR_VSERVER_IP
```

### 5. Deploy

Push to the `main` branch of the automation repository - deployment happens automatically!

```bash
git push origin main
# GitHub Actions will build, push, and deploy automatically
```

---

## Web Service Endpoints

### Health Check

```bash
GET /health
# Returns: { "status": "healthy", "timestamp": "...", "uptime": 123 }
```

### Status

```bash
GET /status
# Returns detailed server status including version and memory usage
```

### GitHub Webhook

```bash
POST /webhook/github
# Receives all GitHub webhook events
# Automatically processes issues and PRs from ALL repositories
```

### VS Code Extension Feedback

```bash
# Submit feedback
POST /api/feedback
Content-Type: application/json

{
  "type": "improvement-accepted",
  "extensionVersion": "1.0.0",
  "data": {
    "message": "AI suggestion was helpful",
    "rating": 5,
    "file": "src/index.ts"
  }
}

# Submit telemetry
POST /api/telemetry
Content-Type: application/json

{
  "metrics": [...],
  "events": [...],
  "extensionVersion": "1.0.0"
}

# Get pending tasks (for VS Code extension)
GET /api/pending-tasks?workspaceId=xxx&capabilities=review,security

# Submit task result
POST /api/task/:taskId/result
Content-Type: application/json

{
  "status": "completed",
  "result": { ... }
}
```

---

## Key Features

### Automatic Repository Discovery

**No manual repository configuration required!**

The server automatically:
- Discovers all repositories in your GitHub organization
- Registers webhooks for new repositories
- Processes issues from any repository

```typescript
// Internally uses GitHubOrgService
const repos = await githubOrgService.listRepositories();
// Returns ALL repos in the organization
```

### Process All Issues (Testing Phase)

**No label filtering - all issues are processed!**

During the testing phase, all issues trigger automation:

```typescript
// IssueProcessor processes ALL issues
// No label checks - will add filtering later when ready
await issueProcessor.processIssueEvent({
  action: 'opened',
  issue: { ... },  // Any issue, any labels
  repository: { ... }
});
```

Future filtering can be added when needed:

```yaml
# Future: automation.yaml configuration
filtering:
  enabled: false  # Currently disabled for testing
  labels:
    include: ['auto-improvement', 'bug']
    exclude: ['wontfix', 'duplicate']
```

### VS Code Extension Feedback Loop

The server provides endpoints for the VS Code extension to:
- Submit user feedback on AI suggestions
- Report errors and issues
- Receive pending tasks for processing
- Submit task results

```typescript
// VS Code extension can submit feedback
await fetch('https://xorng.yourdomain.com/api/feedback', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    type: 'improvement-accepted',
    data: { rating: 5, message: 'Great suggestion!' }
  })
});
```

### Automatic Deployment

**Zero maintenance deployment via GitHub Actions!**

- **On Push**: Any push to `main` triggers automatic deployment
- **Daily Updates**: Automatic update checks run daily at 3 AM UTC
- **Health Monitoring**: Deployment verifies server health before completing
- **Rollback Ready**: Previous images are retained for quick rollback

---

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `GITHUB_TOKEN` | Yes | - | GitHub token with repo/org permissions |
| `GITHUB_ORG` | No | `XORNG` | GitHub organization name |
| `WEBHOOK_SECRET` | No | - | Secret for webhook signature verification |
| `WEBHOOK_URL` | No | - | Public URL for webhook callbacks |
| `PORT` | No | `3000` | Server port |
| `HOST` | No | `0.0.0.0` | Server host |
| `LOG_LEVEL` | No | `info` | Log level (debug, info, warn, error) |

---

## Local Development

### Running Locally

```bash
cd automation

# Install dependencies
npm install

# Set environment variables
export GITHUB_TOKEN=ghp_your_token
export GITHUB_ORG=XORNG
export WEBHOOK_SECRET=your_secret

# Start development server
npm run dev:server
```

### Testing Webhooks Locally

Use [ngrok](https://ngrok.com/) to expose local server:

```bash
# Terminal 1: Start server
npm run dev:server

# Terminal 2: Start ngrok tunnel
ngrok http 3000

# Configure GitHub webhook to point to ngrok URL
# https://abc123.ngrok.io/webhook/github
```

---

## Deployment Options

### Option 1: GitHub Actions (Recommended)

Fully automated deployment - just push to main!

```yaml
# .github/workflows/deploy-automation.yml
# Already configured - see automation/.github/workflows/
```

### Option 2: Manual Docker

```bash
# Build image
docker build -t xorng-automation ./automation

# Run container
docker run -d \
  --name xorng-automation \
  --restart unless-stopped \
  -p 3000:3000 \
  -e GITHUB_TOKEN="your_token" \
  -e GITHUB_ORG="XORNG" \
  -e WEBHOOK_SECRET="your_secret" \
  xorng-automation
```

### Option 3: Docker Compose

```yaml
# docker-compose.yml
version: '3.8'
services:
  xorng-automation:
    build: ./automation
    ports:
      - "3000:3000"
    environment:
      - GITHUB_TOKEN=${GITHUB_TOKEN}
      - GITHUB_ORG=${GITHUB_ORG}
      - WEBHOOK_SECRET=${WEBHOOK_SECRET}
    restart: unless-stopped
```

---

## Domain & HTTPS (Automatic via Docker Compose)

When you configure the `AUTOMATION_DOMAIN` secret, the deployment uses Docker Compose profiles to automatically include Traefik:

### How it works

The `docker-compose.yml` includes Traefik as an optional service using Docker Compose profiles:

```yaml
services:
  traefik:
    image: traefik:v3.2
    profiles:
      - with-domain  # Only starts when this profile is active
    command:
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge=true"
      # ... automatic SSL configuration
    
  automation-server:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.automation.rule=Host(`${AUTOMATION_DOMAIN}`)"
      - "traefik.http.routers.automation.tls.certresolver=letsencrypt"
```

**Deployment modes:**

| `AUTOMATION_DOMAIN` | Command | Result |
|---------------------|---------|--------|
| Set | `docker compose --profile with-domain up -d` | Traefik + SSL |
| Not set | `docker compose up -d automation-server redis` | Direct port 3001 |

### Traffic flow with domain

```
Internet → :443 → Traefik → automation-server:3000
              ↑
        Let's Encrypt
        (automatic SSL)
```

### Without a domain

If `AUTOMATION_DOMAIN` is not set:
- Port 3001 is exposed directly
- No HTTPS (use a separate proxy if needed)
- Set `WEBHOOK_URL` manually in secrets

---

## Monitoring

### View Logs

```bash
# SSH into VServer
ssh deploy@YOUR_VSERVER_IP
cd /opt/xorng

# View all logs
docker compose logs -f

# View specific service
docker compose logs -f automation-server

# View last 100 lines
docker compose logs --tail 100 automation-server
```

### Check Status

```bash
# All services
docker compose ps

# Health check (HTTPS with domain)
curl https://automation.yourdomain.com/health

# Health check (direct access)
curl http://YOUR_VSERVER_IP:3001/health
```

---

## Troubleshooting

### Server not starting

```bash
# Check Docker status
docker ps -a

# Check container logs
docker logs xorng-automation

# Restart container
docker restart xorng-automation
```

### Webhooks not received

1. Check webhook secret matches
2. Verify webhook URL is accessible
3. Check GitHub webhook delivery logs
4. Ensure firewall allows port 3000

### Deployment fails

1. Check GitHub Actions logs
2. Verify SSH key is correct
3. Ensure Docker is running on VServer
4. Check available disk space

---

## Best Practices

### Security

1. Always use HTTPS in production (Caddy handles this automatically)
2. Use strong webhook secrets
3. Rotate GitHub tokens periodically
4. Keep the VServer updated

### Maintenance

1. Monitor disk space for Docker images
2. Review logs periodically
3. Update dependencies regularly
4. Test deployments in staging first

---

## Next Steps

- [Installation Guide](./INSTALLATION.md) - Complete setup
- [VS Code Extension](./VSCODE_EXTENSION.md) - Configure the extension
- [Node Configuration](./NODE_CONFIGURATION.md) - Configure AI providers
    steps:
      - name: analyze_errors
        action: analyze_error_pattern
        inputs:
          - error_logs
          - stack_traces
        outputs:
          - root_cause
          - fix_suggestion
          
      - name: implement_fix
        action: generate_code
        inputs:
          - fix_suggestion
        outputs:
          - code_changes
          
      - name: validate
        action: run_validation
        
      - name: submit
        action: create_pull_request
        config:
          labels:
            - bug-fix
            - auto-generated
```

### Scheduled Optimization

```yaml
# automation.yaml
workflows:
  scheduled_optimization:
    enabled: true
    
    schedule:
      # Run every Sunday at 2 AM
      cron: "0 2 * * 0"
      
    steps:
      - name: analyze_metrics
        action: analyze_performance
        inputs:
          - token_usage_weekly
          - error_rates_weekly
          - response_times_weekly
        outputs:
          - optimization_opportunities
          
      - name: prioritize
        action: rank_improvements
        inputs:
          - optimization_opportunities
        outputs:
          - prioritized_improvements
          
      - name: implement_top_3
        action: batch_implement
        inputs:
          - prioritized_improvements[:3]
        outputs:
          - pull_requests
```

---

## Human-in-the-Loop Approval

### Approval Gates

```yaml
# automation.yaml
approval:
  # Always require human approval
  required: true
  
  # Who can approve
  approvers:
    # GitHub teams
    teams:
      - XORNG/maintainers
      - XORNG/reviewers
      
    # Individual users
    users:
      - lead-developer
      
  # Approval requirements
  requirements:
    # Minimum approvals needed
    min_approvals: 1
    
    # Require approval from code owners
    require_codeowner: true
    
    # Auto-approve low-risk changes
    auto_approve:
      enabled: true
      conditions:
        - change_type: documentation
          confidence: 95
        - change_type: style
          confidence: 90
          
  # Timeout settings
  timeout:
    # Wait max 7 days for approval
    days: 7
    
    # Action on timeout
    on_timeout: close  # close | remind | escalate
```

### Notification Configuration

```yaml
# automation.yaml
notifications:
  # Notify on PR creation
  on_pr_created:
    enabled: true
    channels:
      - type: github
        mention_reviewers: true
      - type: slack
        channel: "#xorng-automation"
        
  # Notify on approval needed
  on_approval_needed:
    enabled: true
    reminder_after_hours: 24
    
  # Notify on merge
  on_merged:
    enabled: true
    channels:
      - type: slack
        channel: "#xorng-releases"
```

---

## Memory & Learning

### Long-Term Memory Storage

```yaml
# automation.yaml
memory:
  # Store successful improvements
  long_term:
    enabled: true
    
    storage:
      type: qdrant
      url: ${QDRANT_URL}
      collection: xorng-improvements
      
    # What to store
    store:
      - successful_fixes      # Bug fixes that worked
      - optimization_patterns # Effective optimizations
      - error_patterns        # Known error → fix mappings
      - code_patterns         # Reusable code patterns
      
    # Retention
    retention:
      successful_fixes: forever
      failed_attempts: 90_days
```

### Pattern Learning

```yaml
# automation.yaml
learning:
  enabled: true
  
  # Learn from successful improvements
  success_learning:
    # Extract patterns from merged PRs
    on_pr_merged:
      - extract_code_pattern
      - extract_fix_pattern
      - update_confidence_scores
      
  # Learn from failures
  failure_learning:
    # Learn from rejected PRs
    on_pr_rejected:
      - analyze_rejection_reason
      - update_model_prompts
      - adjust_confidence_thresholds
      
  # Apply learning
  apply:
    # Use learned patterns for future improvements
    use_similar_patterns: true
    
    # Boost confidence for known patterns
    confidence_boost: 0.1
```

---

## GitHub Actions Integration

### Workflow File

Create `.github/workflows/xorng-automation.yml`:

```yaml
name: XORNG Automation

on:
  issues:
    types: [opened, labeled]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM
  workflow_dispatch:
    inputs:
      action:
        description: 'Action to perform'
        required: true
        default: 'analyze'
        type: choice
        options:
          - analyze
          - optimize
          - full-improvement

jobs:
  process-issue:
    if: github.event_name == 'issues'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install XORNG Automation
        run: |
          cd automation
          npm ci
          
      - name: Process Issue
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          cd automation
          npm run process-issue -- \
            --issue-number=${{ github.event.issue.number }} \
            --repo=${{ github.repository }}
            
  scheduled-optimization:
    if: github.event_name == 'schedule'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install XORNG Automation
        run: |
          cd automation
          npm ci
          
      - name: Run Optimization
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
        run: |
          cd automation
          npm run optimize -- --mode=scheduled
```

### Required Secrets

Configure in GitHub repository settings:

| Secret | Description |
|--------|-------------|
| `ANTHROPIC_API_KEY` | API key for AI provider |
| `OPENAI_API_KEY` | Backup AI provider (optional) |
| `QDRANT_API_KEY` | Vector database (optional) |
| `SLACK_WEBHOOK` | Notifications (optional) |

---

## CLI Commands

### Process Issues

```bash
# Process a specific issue
xorng-auto process-issue --issue-number=42 --repo=XORNG/core

# Process all pending issues
xorng-auto process-issues --repo=XORNG/core --label=auto-improvement
```

### Run Optimization

```bash
# Analyze and suggest optimizations
xorng-auto optimize --mode=analyze

# Generate and submit improvements
xorng-auto optimize --mode=full --auto-submit

# Dry run (no changes)
xorng-auto optimize --mode=full --dry-run
```

### Manage Memory

```bash
# List stored patterns
xorng-auto memory list --type=successful_fixes

# Export patterns
xorng-auto memory export --output=patterns.json

# Import patterns
xorng-auto memory import --input=patterns.json
```

### View Status

```bash
# View automation status
xorng-auto status

# View pending improvements
xorng-auto status --pending

# View recent activity
xorng-auto status --recent=7d
```

---

## Full Configuration Example

```yaml
# automation.yaml - Complete Example
version: "1.0"

enabled: true

github:
  token: ${GITHUB_TOKEN}
  organization: XORNG
  repositories:
    - name: core
      enabled: true
    - name: node
      enabled: true
    - name: validator-code-review
      enabled: true
    - name: validator-security
      enabled: true
    - name: knowledge-documentation
      enabled: true
    - name: knowledge-best-practices
      enabled: true
  trigger_labels:
    - auto-improvement
    - bug
    - enhancement

provider:
  name: anthropic
  model: claude-sonnet-4-20250514
  fallback:
    - name: openai
      model: gpt-4o

workflows:
  issue_processing:
    enabled: true
    confidence_threshold: 80
    
  error_improvement:
    enabled: true
    error_threshold: 5
    
  scheduled_optimization:
    enabled: true
    schedule:
      cron: "0 2 * * 0"

approval:
  required: true
  min_approvals: 1
  auto_approve:
    enabled: true
    conditions:
      - change_type: documentation
        confidence: 95

memory:
  long_term:
    enabled: true
    storage:
      type: qdrant
      url: ${QDRANT_URL}

notifications:
  on_pr_created:
    enabled: true
    channels:
      - type: github
        mention_reviewers: true

telemetry:
  enabled: true
  storage:
    type: redis
    url: ${REDIS_URL}
```

---

## Best Practices

### Safety First

1. **Always require human approval** for non-trivial changes
2. **Set confidence thresholds** high (>80%) for auto-actions
3. **Use dry-run mode** when testing workflows
4. **Monitor closely** for the first few weeks

### Gradual Rollout

1. Start with **documentation-only** improvements
2. Add **bug fixes** after confidence builds
3. Enable **enhancements** last
4. Increase **auto-approve** gradually

### Monitoring

1. **Review all auto-generated PRs** initially
2. **Track success/rejection rates**
3. **Adjust thresholds** based on data
4. **Store and analyze** failure patterns

---

## Next Steps

- [Node Configuration](./NODE_CONFIGURATION.md) - Configure AI providers
- [Installation Guide](./INSTALLATION.md) - Complete setup
- [Creating Sub-Agents](./CREATING_SUBAGENTS.md) - Build custom sub-agents
