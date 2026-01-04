# Pipeline Automation - Self-Healing CI/CD

This document describes the automated pipeline fixing and human-in-the-loop merge approval system implemented in the XORNG automation server.

## Overview

The Pipeline Automation system provides:

1. **Automatic Pipeline Failure Detection** - Monitors CI/CD pipeline status via GitHub webhooks
2. **AI-Powered Failure Analysis** - Uses AI to analyze failures and suggest fixes
3. **Self-Healing Auto-Fix** - Automatically applies fixes for common failure types
4. **Human-in-the-Loop Merge Approval** - Requires human approval before merging

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                      GitHub Webhooks                             │
│  (check_run, check_suite, workflow_run, issue_comment)          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Webhook Server                               │
│              (Signature verification, event routing)            │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                  PipelineAutomation                              │
│  (Event coordination, fix tracking, human intervention)         │
└─────────────────────────────────────────────────────────────────┘
            │                                   │
            ▼                                   ▼
┌──────────────────────────┐      ┌──────────────────────────────┐
│    PipelineMonitor       │      │   MergeApprovalService       │
│  - Status monitoring     │      │  - Command processing        │
│  - Failure analysis      │      │  - Approval workflow         │
│  - Auto-fix application  │      │  - Merge execution           │
└──────────────────────────┘      └──────────────────────────────┘
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
  - Has its fix completed (all checks pass)
- Queued PRs are notified and processed in FIFO order
- The lock is released when the PR is closed, merged, or all checks pass

This ensures:
- Clean git history without conflicting automated commits
- No duplicate fixes across PRs
- Predictable fix ordering

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

# Optional
BOT_USERNAME=xorng-bot         # Bot account username
LOG_LEVEL=info                 # Logging level
AUTO_FIX_ENABLED=true          # Enable/disable auto-fix
MERGE_APPROVAL_ENABLED=true    # Enable/disable merge approval
MAX_FIX_ATTEMPTS=3             # Max auto-fix attempts per PR
```

### GitHub Webhook Setup

Configure your repository/organization webhook with these events:

- `check_run`
- `check_suite`
- `workflow_run`
- `workflow_job`
- `issue_comment`
- `pull_request`

### GitHub Workflow

The `self-healing.yml` workflow provides additional automation:

```yaml
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]
```

This workflow:
1. Triggers when CI completes
2. Analyzes failure logs
3. Applies simple fixes (lint/format)
4. Posts status comments on PRs

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
| `pipeline:passed` | All pipeline checks passed |
| `pipeline:failed` | Pipeline checks failed |
| `merge:approved` | PR was approved for merge |
| `merge:completed` | PR was successfully merged |
| `merge:ready` | PR is ready for merge (all checks passing) |
| `intervention:requested` | Human intervention is required |

## Example Usage

```typescript
import { 
  PipelineAutomation, 
  WebhookServer, 
  AIService 
} from './server/index.js';

// Initialize services
const webhookServer = new WebhookServer({
  port: 3000,
  webhookSecret: process.env.WEBHOOK_SECRET,
});

const aiService = await AIService.create({
  apiKey: process.env.OPENROUTER_API_KEY,
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

// Listen for events
automation.on('fix:applied', (data) => {
  console.log(`Fix applied to PR #${data.prNumber}`);
});

automation.on('merge:ready', (data) => {
  console.log(`PR #${data.prNumber} is ready for merge`);
});

// Start server
await webhookServer.start();
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
