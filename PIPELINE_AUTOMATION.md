# Pipeline Automation - Self-Healing CI/CD

This document describes the automated pipeline fixing and human-in-the-loop merge approval system implemented in the XORNG automation server.

## Overview

The Pipeline Automation system provides:

1. **Automatic Pipeline Failure Detection** - Monitors CI/CD pipeline status via GitHub webhooks across ALL organization repositories
2. **Multi-Repository Scanning** - Periodic scans ensure no failing PRs are missed (backup for webhooks)
3. **AI-Powered Failure Analysis** - Uses AI to analyze failures and suggest fixes
4. **Self-Healing Auto-Fix** - Automatically applies fixes for common failure types
5. **Human-in-the-Loop Merge Approval** - Requires human approval before merging

## Architecture

The architecture is **centralized and server-based**, monitoring ALL repositories in the organization from a single automation server:

```
┌─────────────────────────────────────────────────────────────────┐
│                     GitHub Organization                          │
│            (All repositories across the org)                    │
└─────────────────────────────────────────────────────────────────┘
                              │
            ┌─────────────────┴─────────────────┐
            ▼                                   ▼
┌──────────────────────────┐      ┌──────────────────────────────┐
│     GitHub Webhooks      │      │     MultiRepoScanner         │
│ (Real-time notifications)│      │   (Periodic backup scan)     │
│                          │      │   - Scans every 5 minutes    │
│ Events:                  │      │   - Catches missed webhooks  │
│ - check_run              │      │   - Startup initialization   │
│ - check_suite            │      │                              │
│ - workflow_run           │      │                              │
│ - pull_request           │      │                              │
│ - issue_comment          │      │                              │
└──────────────────────────┘      └──────────────────────────────┘
            │                                   │
            └─────────────────┬─────────────────┘
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Automation Server                            │
│              (Centralized for entire organization)              │
├─────────────────────────────────────────────────────────────────┤
│                     Webhook Server                               │
│        (Signature verification, event routing)                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  PipelineAutomation                              │
│  (Event coordination, fix tracking, human intervention)         │
│  - Repository locking (one PR at a time per repo)              │
│  - Fix attempt tracking                                         │
│  - Learning service integration                                 │
└─────────────────────────────────────────────────────────────────┘
            │                                   │
            ▼                                   ▼
┌──────────────────────────┐      ┌──────────────────────────────┐
│    PipelineMonitor       │      │   MergeApprovalService       │
│  - Status monitoring     │      │  - Command processing        │
│  - Failure analysis      │      │  - Approval workflow         │
│  - Auto-fix application  │      │  - Merge execution           │
│  - AI integration        │      │                              │
└──────────────────────────┘      └──────────────────────────────┘
            │
            ▼
┌──────────────────────────┐
│    GitHubOrgService      │
│  - Repository discovery  │
│  - Webhook registration  │
│  - New repo detection    │
└──────────────────────────┘
```

## Why Server-Based (Not GitHub Actions)?

The system uses a **centralized server** instead of GitHub Actions for several important reasons:

1. **Organization-Wide Coverage**: GitHub Actions `workflow_run` events are repository-scoped - they can only see workflow runs in their own repository. A centralized server can monitor ALL repositories.

2. **Unified Webhook Reception**: All repositories send webhooks to one server, enabling consistent monitoring and coordination.

3. **AI/Learning Integration**: The automation server integrates with AI services and learning systems that would be complex to replicate in each repository.

4. **State Management**: Repository locking, queue management, and fix attempt tracking require centralized state.

5. **Backup Scanning**: The `MultiRepoScanner` provides redundancy by periodically scanning for PRs that may have been missed by webhooks.

## Multi-Repository Scanner

The `MultiRepoScanner` ensures complete coverage by:

1. **Periodic Scanning**: Every 5 minutes (configurable), scans all organization repositories
2. **Startup Initialization**: Immediately scans on server start to catch any PRs that failed while server was down
3. **Webhook Backup**: Catches PRs where webhooks were missed (network issues, etc.)
4. **Efficient Processing**: Only processes PRs with actual failing checks

```typescript
// Scanner workflow
1. List all org repositories via GitHubOrgService
2. For each repo, list open PRs
3. For each PR, check for failing CI checks
4. If failing checks found, trigger PipelineAutomation.triggerPRAnalysis()
5. Respects existing repository locks and queuing
```

## Self-Improvement Guidelines

The system follows these self-improvement principles:

1. **Track Fix Attempts** - Each PR has a maximum of 3 auto-fix attempts before requiring human intervention
2. **Learn from Failures** - Fix patterns and success rates are tracked for analysis
3. **Confidence Thresholds** - Auto-fixes are only applied when AI confidence exceeds 70%
4. **Progressive Escalation** - Failures escalate from auto-fix → human review → blocked
5. **One PR at a Time per Repository** - Only one PR can be auto-fixed at a time in each repository to prevent merge conflicts

## Repository Locking

To prevent merge conflicts and duplicate fixes, the system enforces a **one-PR-at-a-time** policy per repository:

- When a PR starts being auto-fixed, a lock is acquired for that repository
- Other PRs with failing pipelines are **queued** and will be processed after the current PR is:
  - Merged successfully, or
  - Has its fix completed (all checks pass), or
  - Analysis determines no auto-fixable issues
- Queued PRs are notified and processed in FIFO order
- The lock is released in these scenarios:
  - PR is closed or merged
  - All checks pass after a fix
  - Analysis completes with no fixable issues
  - Fix application fails
  - Validation pre-check fails

### Lock Release Events

The system uses the following events to coordinate lock release and queue processing:

| Event | Trigger | Lock Released | Queue Processed |
|-------|---------|---------------|-----------------|
| `analysis:complete` | No auto-fixable issues found | ✅ | ✅ |
| `analysis:complete` | Validator pre-check fails | ✅ | ✅ |
| `analysis:complete` | Auto-fix disabled | ✅ | ✅ |
| `fix:applied` | Fix committed and pushed | ❌ (wait for CI) | ❌ |
| `fix:failed` | Fix application error | ✅ | ✅ |
| `fix:successful` | Pipeline passed after fix | ✅ | ✅ |
| `pr:merged` | PR merged | ✅ | ✅ |
| `pr:closed` | PR closed | ✅ | ✅ |

This ensures:
- Clean git history without conflicting automated commits
- No duplicate fixes across PRs
- Predictable fix ordering
- Queue always gets processed, even when no fixes are possible

## Supported Failure Types

The system can analyze and potentially auto-fix these failure types:

| Type | Auto-fixable | Description |
|------|-------------|-------------|
| `lint` | ✅ | ESLint/code style issues |
| `format` | ✅ | Prettier/formatting issues |
| `typecheck` | ⚠️ | TypeScript type errors (partial) |
| `build` | ⚠️ | Build failures (partial) |
| `test` | ❌ | Test failures require manual review |
| `security` | ❌ | Security issues require manual review |

## Comment Commands

Users can interact with the automation via PR comments:

### Merge Commands

```
/approve       - Approve the PR for merging (adds review approval)
/merge         - Merge using default method (squash)
/merge squash  - Squash and merge
/merge merge   - Create a merge commit
/merge rebase  - Rebase and merge
```

### Control Commands

```
/hold          - Place a hold on the PR (adds 'do-not-merge' label)
/unhold        - Remove the hold
/autofix       - Request a new auto-fix attempt (resets attempt counter)
```

## Workflow

### Pipeline Failure Flow

```
1. CI Pipeline Fails
   └── GitHub sends check_run/workflow_run webhook
       └── PipelineMonitor receives event
           └── Check fix attempt count (max 3)
               ├── [Attempts < 3] Analyze failure with AI
               │   └── [Confidence > 70%] Apply auto-fix
               │       └── Commit and push fix
               │           └── CI re-runs automatically
               └── [Attempts >= 3] Request human intervention
                   └── Post comment + add label
```

### Pipeline Success Flow

```
1. All CI Checks Pass
   └── GitHub sends check_suite/workflow_run webhook
       └── PipelineMonitor receives event
           └── Post merge-ready comment
               └── List available commands
```

### Merge Approval Flow

```
1. User posts /approve comment
   └── MergeApprovalService validates:
       ├── All checks passing
       ├── No blocking labels (do-not-merge, wip)
       └── User has write access
           └── Add approving review
               └── Notify PR is ready for /merge

2. User posts /merge comment
   └── MergeApprovalService validates:
       ├── PR is approved
       ├── All checks passing
       └── PR is mergeable
           └── Execute merge with specified method
               └── Notify success/failure
```

## Configuration

### Environment Variables

```bash
# Required
GITHUB_TOKEN=ghp_xxx           # GitHub token with repo access
GITHUB_ORG=xorng-org           # GitHub organization name
WEBHOOK_SECRET=xxx             # Webhook signature secret
OPENROUTER_API_KEY=xxx         # OpenRouter API key for AI

# Optional - Pipeline Automation
BOT_USERNAME=xorng-bot         # Bot account username (default: xorng-bot)
PIPELINE_ENABLED=true          # Enable/disable pipeline automation (default: true)
SCAN_INTERVAL_MS=300000        # Multi-repo scanner interval in ms (default: 5 minutes)

# Optional - AI Service Retry Configuration
AI_MAX_RETRIES=3               # Max retry attempts for rate-limited requests (default: 3)
AI_RETRY_DELAY_MS=5000         # Initial delay before first retry in ms (default: 5000)
AI_MAX_RETRY_DELAY_MS=120000   # Maximum delay between retries in ms (default: 120000 = 2 min)

# Optional - AI Model Configuration
OPENROUTER_MODEL=mistralai/devstral-2512:free  # Primary model (default: mistralai/devstral-2512:free - FREE)
AI_FALLBACK_MODELS=xiaomi/mimo-v2-flash:free,kwaipilot/kat-coder-pro:free,tngtech/deepseek-r1t2-chimera:free
# Comma-separated list of fallback models. OpenRouter will automatically try the next model
# if the primary model fails (provider issues, rate limits, etc.)
# All defaults are FREE models ($0/token) - no cost for primary or fallbacks

# Optional - General
LOG_LEVEL=info                 # Logging level
AUTO_FIX_ENABLED=true          # Enable/disable auto-fix (default: true)
MERGE_APPROVAL_ENABLED=true    # Enable/disable merge approval (default: true)
MAX_FIX_ATTEMPTS=3             # Max auto-fix attempts per PR (default: 3)
```

### AI Service Model Fallback

OpenRouter's `models` array feature enables automatic failover when the primary model fails. The AI service now supports:

- **Automatic Fallback**: When the primary model returns a 5xx error or is unavailable, OpenRouter automatically tries the next model in the array
- **Provider Routing**: Uses OpenRouter's `route: 'fallback'` setting to route around degraded providers (<95% uptime)
- **Custom Fallback Models**: Configure via `AI_FALLBACK_MODELS` env var (comma-separated)
- **Default Fallbacks**: If not configured, uses FREE built-in fallbacks:
  - `xiaomi/mimo-v2-flash:free` - MiMo-V2 MoE model, #1 on SWE-bench (256K context)
  - `kwaipilot/kat-coder-pro:free` - KAT-Coder-Pro, optimized for coding tasks
  - `tngtech/deepseek-r1t2-chimera:free` - DeepSeek R1T2 Chimera, strong reasoning
  - `nvidia/nemotron-3-nano-30b-a3b:free` - NVIDIA Nemotron, efficient agentic AI

This addresses upstream provider failures (500 errors) without requiring local retry logic for model-level issues.

### AI Service Retry Logic

The AI service automatically retries failed requests with exponential backoff:

| Error Type | Retryable | Behavior |
|------------|-----------|----------|
| 429 Rate Limit | ✅ | Waits for rate limit reset time (from headers) or uses exponential backoff |
| 5xx Server Error | ✅ | Exponential backoff with jitter + automatic model fallback |
| Network Error | ✅ | Exponential backoff with jitter |
| 4xx Client Error (except 429) | ❌ | Fails immediately |
| Invalid Response | ❌ | Fails immediately |

**Retry Delays:**
- Default: 5s → 10s → 20s (exponential)
- With rate limit header: Uses `X-RateLimit-Reset` timestamp + 1s buffer
- Maximum delay: 2 minutes (configurable)

### GitHub Webhook Setup

Configure your organization webhook (recommended) or per-repository webhooks with these events:

**Required Events for Self-Healing Pipeline:**
- `check_run` - Individual CI check status
- `check_suite` - Grouped CI check status
- `workflow_run` - GitHub Actions workflow status
- `pull_request` - PR lifecycle events
- `issue_comment` - Commands like `/autofix`, `/approve`, `/merge`
- `pull_request_review` - Review status

**Additional Events (for issue processing):**
- `issues` - Issue lifecycle events

The automation server will automatically register webhooks for all repositories when `WEBHOOK_URL` is configured.

## API Endpoints

### Pipeline Analysis

```
POST /api/pipeline/analyze
{
  "owner": "org-name",
  "repo": "repo-name",
  "prNumber": 123,
  "runId": 456789,
  "conclusion": "failure",
  "fixType": "lint"
}
```

### Health Check

```
GET /health
{
  "status": "healthy",
  "services": {
    "pipelineMonitor": true,
    "mergeApproval": true
  },
  "stats": {
    "pipelinesMonitored": 150,
    "fixesAttempted": 45,
    "fixesSuccessful": 32,
    "mergesApproved": 28,
    "mergesCompleted": 25
  }
}
```

## Events Emitted

The `PipelineAutomation` class emits these events for external monitoring:

| Event | Description |
|-------|-------------|
| `fix:applied` | Auto-fix was successfully applied |
| `fix:failed` | Auto-fix attempt failed |
| `fix:successful` | Auto-fix resulted in passing CI |
| `pipeline:passed` | All pipeline checks passed |
| `pipeline:failed` | Pipeline checks failed |
| `merge:approved` | PR was approved for merge |
| `merge:completed` | PR was successfully merged |
| `merge:ready` | PR is ready for merge (all checks passing) |
| `intervention:requested` | Human intervention is required |
| `scanner:pr-analyzed` | Scanner triggered analysis for a PR |

The `MultiRepoScanner` class emits these events:

| Event | Description |
|-------|-------------|
| `scan:started` | Periodic scan has begun |
| `scan:completed` | Scan finished with results |
| `scan:error` | Scan encountered an error |
| `pr:found` | Found a PR with failing checks |
| `pr:processed` | Finished processing a failing PR |

## Example Usage

```typescript
import { 
  PipelineAutomation, 
  MultiRepoScanner,
  WebhookServer, 
  AIService,
  GitHubOrgService 
} from './server/index.js';

// Initialize services
const webhookServer = new WebhookServer({
  port: 3000,
  webhookSecret: process.env.WEBHOOK_SECRET,
});

const aiService = await AIService.create({
  apiKey: process.env.OPENROUTER_API_KEY,
});

const githubOrgService = new GitHubOrgService({
  token: process.env.GITHUB_TOKEN,
  organization: process.env.GITHUB_ORG,
});

// Create pipeline automation
const automation = new PipelineAutomation({
  token: process.env.GITHUB_TOKEN,
  organization: process.env.GITHUB_ORG,
  aiService,
  webhookServer,
  botUsername: 'xorng-bot',
  enableAutoFix: true,
  enableMergeApproval: true,
});

// Create multi-repo scanner for comprehensive coverage
const scanner = new MultiRepoScanner({
  token: process.env.GITHUB_TOKEN,
  organization: process.env.GITHUB_ORG,
  githubOrgService,
  pipelineAutomation: automation,
  scanIntervalMs: 5 * 60 * 1000, // 5 minutes
  enablePeriodicScan: true,
});

// Listen for events
automation.on('fix:applied', (data) => {
  console.log(`Fix applied to PR #${data.prNumber}`);
});

automation.on('merge:ready', (data) => {
  console.log(`PR #${data.prNumber} is ready for merge`);
});

scanner.on('scan:completed', (result) => {
  console.log(`Scanned ${result.reposScanned} repos, found ${result.failingPRs} failing PRs`);
});

// Start services
await webhookServer.start();
scanner.start(); // Begins periodic scanning
```

## Troubleshooting

### Auto-fix not working

1. Check if `AUTO_FIX_ENABLED=true`
2. Verify AI service is configured with valid API key
3. Check fix attempt count (max 3)
4. Review logs for analysis errors

### Merge commands not recognized

1. Ensure comment starts with `/` at beginning of line
2. Verify user has write access to repository
3. Check for blocking labels on PR

### Webhook not received

1. Verify webhook secret matches
2. Check webhook events are configured
3. Review server logs for signature errors

## Security Considerations

1. **Token Permissions** - Use a GitHub token with minimal required permissions
2. **Webhook Secret** - Always configure and verify webhook signatures
3. **AI Analysis** - Never include sensitive data in AI prompts
4. **Merge Restrictions** - Only users with write access can execute merges
5. **Label Protection** - Blocking labels prevent automated merges
