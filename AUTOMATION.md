# XORNG Automation Configuration Guide

Configure XORNG's self-improvement system for automatic codebase enhancements, issue resolution, and continuous improvement.

## Overview

XORNG's automation system enables:

- **Self-Improvement** - Automatic code enhancements based on feedback
- **Issue Resolution** - Auto-fix bugs from GitHub issues
- **Continuous Learning** - Store successful patterns in long-term memory
- **Human-in-the-Loop** - Approval gates for safe deployment

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    XORNG AUTOMATION SYSTEM                          │
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐          │
│  │   INPUTS     │    │  PROCESSING  │    │   OUTPUTS    │          │
│  ├──────────────┤    ├──────────────┤    ├──────────────┤          │
│  │ GitHub Issues│───▶│ AI Analysis  │───▶│ Pull Request │          │
│  │ Error Logs   │───▶│ Validation   │───▶│ Config Update│          │
│  │ Token Metrics│───▶│ Human Review │───▶│ New Sub-Agent│          │
│  │ Telemetry    │───▶│ Testing      │───▶│ Documentation│          │
│  └──────────────┘    └──────────────┘    └──────────────┘          │
│                              │                                      │
│                              ▼                                      │
│                    ┌──────────────────┐                            │
│                    │  Long-Term Memory │                            │
│                    │  (Pattern Storage)│                            │
│                    └──────────────────┘                            │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Quick Start

### 1. Configure automation.yaml

```yaml
# automation.yaml
version: "1.0"

# Enable the automation system
enabled: true

# GitHub Integration
github:
  token: ${GITHUB_TOKEN}
  organization: XORNG
  
# AI Provider for automation tasks
provider:
  name: anthropic
  model: claude-sonnet-4-20250514

# Human approval settings
approval:
  required: true
  auto_merge_after_approval: true
```

### 2. Set Environment Variables

```bash
# .env
GITHUB_TOKEN=ghp_your-github-token
ANTHROPIC_API_KEY=sk-ant-your-key
```

### 3. Start Automation Service

```bash
cd ~/xorng/automation
npm start
```

---

## Input Sources

### GitHub Issues

XORNG monitors GitHub issues across configured repositories:

```yaml
# automation.yaml
github:
  token: ${GITHUB_TOKEN}
  organization: XORNG
  
  # Repositories to monitor
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
      
  # Issue labels that trigger automation
  trigger_labels:
    - auto-improvement    # General improvements
    - bug                 # Bug fixes
    - enhancement         # Feature requests
    - performance         # Optimization
    - security            # Security fixes
    - documentation       # Doc updates
    
  # Labels to ignore
  ignore_labels:
    - wontfix
    - duplicate
    - manual-only
```

### Error Telemetry

Collect and analyze errors from running systems:

```yaml
# automation.yaml
telemetry:
  enabled: true
  
  # Error collection
  errors:
    # Minimum occurrences before triggering improvement
    threshold: 5
    
    # Time window for counting
    window_hours: 24
    
    # Error categories to monitor
    categories:
      - runtime_error
      - type_error
      - validation_error
      - timeout
      - memory_leak
      
  # Where to store telemetry
  storage:
    type: redis
    url: ${REDIS_URL}
    retention_days: 30
```

### Token Usage Metrics

Optimize based on token consumption patterns:

```yaml
# automation.yaml
metrics:
  tokens:
    # Track usage per sub-agent
    per_agent: true
    
    # Track usage per task type
    per_task: true
    
    # Trigger optimization when usage exceeds threshold
    optimization_threshold: 0.8  # 80% of budget
    
  # Metrics collection endpoint
  endpoint:
    type: prometheus
    port: 9090
```

### Performance Metrics

Monitor and improve performance:

```yaml
# automation.yaml
metrics:
  performance:
    # Track response times
    response_time:
      target_p50_ms: 500
      target_p99_ms: 2000
      
    # Track success rates
    success_rate:
      target: 0.95
      
    # Trigger improvements when targets missed
    auto_improve: true
```

---

## Processing Pipeline

### Issue Processing Workflow

```yaml
# automation.yaml
workflows:
  issue_processing:
    enabled: true
    
    steps:
      # Step 1: Analyze the issue
      - name: analyze
        action: analyze_issue
        provider: anthropic
        model: claude-sonnet-4-20250514
        prompts:
          system: |
            You are analyzing a GitHub issue for the XORNG project.
            Determine the type of change needed, affected files, and complexity.
          user: |
            Issue Title: {{issue.title}}
            Issue Body: {{issue.body}}
            Labels: {{issue.labels}}
            Repository: {{issue.repository}}
        outputs:
          - change_type      # bug_fix | enhancement | refactor | docs
          - affected_files   # List of files to modify
          - complexity       # low | medium | high
          - implementation_plan
          
      # Step 2: Generate implementation
      - name: implement
        action: generate_code
        provider: anthropic
        model: claude-sonnet-4-20250514
        condition: analysis.complexity != "high"
        inputs:
          - analysis.implementation_plan
          - analysis.affected_files
        outputs:
          - code_changes     # Map of file -> changes
          - tests            # New/modified tests
          
      # Step 3: Validate changes
      - name: validate
        action: run_validation
        steps:
          - lint: eslint
          - typecheck: tsc
          - test: vitest
          - security: validator-security
        fail_on_error: true
        
      # Step 4: Self-review
      - name: review
        action: code_review
        provider: anthropic
        model: claude-sonnet-4-20250514
        inputs:
          - code_changes
          - test_results
        outputs:
          - review_passed    # boolean
          - review_comments  # List of comments
          - confidence       # 0-100
          
      # Step 5: Create PR (if review passed)
      - name: submit
        action: create_pull_request
        condition: review.review_passed AND review.confidence > 80
        inputs:
          - code_changes
          - review_comments
        config:
          branch_prefix: xorng/auto
          require_approval: true
          labels:
            - auto-generated
            - needs-review
```

### Error-Driven Improvement

```yaml
# automation.yaml
workflows:
  error_improvement:
    enabled: true
    
    trigger:
      type: error_threshold
      threshold: 5
      window_hours: 24
      
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
