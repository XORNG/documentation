# XORNG Node Configuration Guide

Configure AI model providers (nodes) for XORNG's intelligent operations and self-improvement capabilities.

## Overview

XORNG uses a **Node** abstraction to interact with various AI providers. Nodes enable:

- **Multi-Provider Support** - Use OpenAI, Anthropic, local models, or custom providers
- **Automatic Failover** - Fall back to alternate providers on errors
- **Cost Optimization** - Route to cheaper models for simple tasks
- **Self-Improvement** - Use nodes for automatic codebase improvements

---

## Quick Start

### Minimal Configuration

```yaml
# node.yaml
providers:
  - name: openai
    type: openai
    api_key: ${OPENAI_API_KEY}
    default_model: gpt-4o
```

### Multi-Provider Configuration

```yaml
# node.yaml
version: "1.0"

providers:
  # Primary provider for complex tasks
  - name: anthropic
    type: anthropic
    api_key: ${ANTHROPIC_API_KEY}
    default_model: claude-sonnet-4-20250514
    max_tokens: 8192
    
  # Secondary provider for simple tasks / fallback
  - name: openai
    type: openai
    api_key: ${OPENAI_API_KEY}
    default_model: gpt-4o-mini
    
  # Local model for sensitive data
  - name: local
    type: ollama
    base_url: http://localhost:11434
    default_model: llama3.2
    
routing:
  # Use Anthropic for code generation
  code_generation: anthropic
  
  # Use cheaper model for analysis
  analysis: openai
  
  # Use local for sensitive operations
  sensitive: local
  
  # Default fallback chain
  default:
    - anthropic
    - openai
    - local
```

---

## Provider Types

### OpenAI

```yaml
providers:
  - name: openai
    type: openai
    api_key: ${OPENAI_API_KEY}
    organization: ${OPENAI_ORG_ID}  # Optional
    base_url: https://api.openai.com/v1  # Optional, for proxies
    default_model: gpt-4o
    models:
      - name: gpt-4o
        context_window: 128000
        cost_per_1k_input: 0.005
        cost_per_1k_output: 0.015
      - name: gpt-4o-mini
        context_window: 128000
        cost_per_1k_input: 0.00015
        cost_per_1k_output: 0.0006
    retry:
      max_attempts: 3
      backoff_ms: 1000
```

### Anthropic

```yaml
providers:
  - name: anthropic
    type: anthropic
    api_key: ${ANTHROPIC_API_KEY}
    default_model: claude-sonnet-4-20250514
    models:
      - name: claude-sonnet-4-20250514
        context_window: 200000
        cost_per_1k_input: 0.003
        cost_per_1k_output: 0.015
      - name: claude-3-5-haiku-20241022
        context_window: 200000
        cost_per_1k_input: 0.00025
        cost_per_1k_output: 0.00125
    retry:
      max_attempts: 3
      backoff_ms: 1000
```

### Local Models (Ollama)

```yaml
providers:
  - name: local
    type: ollama
    base_url: http://localhost:11434
    default_model: llama3.2
    models:
      - name: llama3.2
        context_window: 128000
        cost_per_1k_input: 0  # Free
        cost_per_1k_output: 0
      - name: codellama
        context_window: 16000
        cost_per_1k_input: 0
        cost_per_1k_output: 0
    timeout_ms: 60000  # Longer timeout for local inference
```

### Azure OpenAI

```yaml
providers:
  - name: azure-openai
    type: azure
    api_key: ${AZURE_OPENAI_KEY}
    endpoint: https://your-resource.openai.azure.com
    api_version: "2024-02-01"
    deployments:
      - deployment_name: gpt-4o-deployment
        model_name: gpt-4o
        context_window: 128000
```

### Custom Provider

```yaml
providers:
  - name: custom-llm
    type: custom
    base_url: https://your-api.com/v1
    api_key: ${CUSTOM_API_KEY}
    headers:
      X-Custom-Header: value
    default_model: custom-model
    request_format: openai  # openai | anthropic | custom
```

---

## Automatic GitHub Development

XORNG can use configured nodes to automatically improve itself and your codebase through GitHub.

### Enable Self-Improvement

```yaml
# node.yaml
version: "1.0"

providers:
  - name: developer
    type: anthropic
    api_key: ${ANTHROPIC_API_KEY}
    default_model: claude-sonnet-4-20250514
    # This provider will be used for code generation
    capabilities:
      - code_generation
      - code_review
      - documentation

self_improvement:
  enabled: true
  
  # Which provider to use for improvements
  provider: developer
  
  # GitHub configuration
  github:
    token: ${GITHUB_TOKEN}
    organization: XORNG
    
    # Repositories to auto-improve
    repositories:
      - core
      - node
      - validator-code-review
      - validator-security
      - knowledge-documentation
      - knowledge-best-practices
      
    # Branch naming convention
    branch_prefix: xorng/auto-improvement
    
    # Require human approval before merge
    require_approval: true
    
    # Minimum review approvals before auto-merge
    min_approvals: 1
    
  # Types of improvements to make
  improvement_types:
    - bug_fixes           # Fix identified bugs
    - performance         # Optimize performance
    - documentation       # Update docs
    - code_quality        # Refactoring
    - security            # Security fixes
    - new_features        # From GitHub issues
    
  # Scheduling
  schedule:
    # Run improvements daily at 2 AM
    cron: "0 2 * * *"
    # Or on specific triggers
    triggers:
      - github_issue_labeled: "auto-improvement"
      - error_threshold_exceeded: 10
      - weekly_review: true
```

### GitHub Issue-Driven Development

When issues are created in XORNG repositories:

```yaml
# node.yaml (continued)
issue_processing:
  enabled: true
  
  # Labels that trigger automatic processing
  trigger_labels:
    - auto-improvement
    - enhancement
    - bug
    
  # Provider for analyzing issues
  provider: developer
  
  # Workflow for processing issues
  workflow:
    1_analyze:
      # Understand the issue
      action: analyze_issue
      output: issue_analysis
      
    2_plan:
      # Create implementation plan
      action: create_plan
      input: issue_analysis
      output: implementation_plan
      
    3_implement:
      # Generate code changes
      action: implement_changes
      input: implementation_plan
      output: code_changes
      
    4_test:
      # Run tests on changes
      action: run_tests
      input: code_changes
      output: test_results
      
    5_review:
      # Self-review the changes
      action: code_review
      input: [code_changes, test_results]
      output: review_results
      
    6_submit:
      # Create PR if all checks pass
      action: create_pull_request
      input: [code_changes, review_results]
      condition: review_results.approved == true
```

### Example: Automatic Bug Fix Workflow

1. **Issue Created**: User creates issue "Fix memory leak in TokenTracker"
2. **XORNG Analyzes**: Uses configured node to understand the issue
3. **Plan Generated**: Creates step-by-step fix plan
4. **Code Generated**: Implements the fix using AI
5. **Tests Run**: Executes test suite
6. **PR Created**: Opens pull request with changes
7. **Human Review**: Developer reviews and approves
8. **Auto-Merge**: XORNG merges after approval

```yaml
# Example PR created by XORNG
# PR Title: fix: resolve memory leak in TokenTracker (#42)
#
# PR Body:
# ## Summary
# Fixes memory leak identified in issue #42.
#
# ## Changes
# - Added cleanup in TokenTracker.dispose()
# - Implemented WeakMap for reference tracking
# - Added unit tests for memory management
#
# ## Testing
# - [x] Unit tests pass
# - [x] Memory profiling shows no leak
# - [x] Integration tests pass
#
# ## Auto-generated by XORNG
# Provider: anthropic/claude-sonnet-4-20250514
# Confidence: 94%
```

---

## Model Selection & Routing

### Task-Based Routing

```yaml
routing:
  rules:
    # Complex code generation → Best model
    - task_type: code_generation
      complexity: high
      provider: anthropic
      model: claude-sonnet-4-20250514
      
    # Simple code completion → Fast model
    - task_type: code_completion
      complexity: low
      provider: openai
      model: gpt-4o-mini
      
    # Code review → Thorough model
    - task_type: code_review
      provider: anthropic
      model: claude-sonnet-4-20250514
      
    # Documentation → Cost-effective
    - task_type: documentation
      provider: openai
      model: gpt-4o-mini
      
    # Security analysis → Most capable
    - task_type: security_analysis
      provider: anthropic
      model: claude-sonnet-4-20250514
      
    # Sensitive data → Local only
    - task_type: any
      data_sensitivity: high
      provider: local
      model: llama3.2
```

### Cost-Based Routing

```yaml
routing:
  strategy: cost_optimized
  
  budgets:
    daily_limit_usd: 10.00
    warning_threshold_usd: 8.00
    
  cost_tiers:
    # Use expensive models only when needed
    - condition: complexity == "high" OR task_critical == true
      provider: anthropic
      model: claude-sonnet-4-20250514
      
    # Default to cheaper models
    - condition: default
      provider: openai
      model: gpt-4o-mini
      
    # Fall back to local when budget exceeded
    - condition: daily_budget_exceeded
      provider: local
      model: llama3.2
```

### Fallback Chains

```yaml
routing:
  fallback:
    # Primary → Secondary → Tertiary
    chain:
      - provider: anthropic
        model: claude-sonnet-4-20250514
        timeout_ms: 30000
        
      - provider: openai
        model: gpt-4o
        timeout_ms: 30000
        
      - provider: local
        model: llama3.2
        timeout_ms: 60000
        
    # When to fallback
    fallback_on:
      - rate_limit_exceeded
      - timeout
      - api_error
      - model_unavailable
```

---

## Token Management

```yaml
tokens:
  tracking:
    enabled: true
    
    # Use tiktoken for accurate counting
    tokenizer: tiktoken
    encoding: cl100k_base
    
  limits:
    # Per-request limits
    max_input_tokens: 100000
    max_output_tokens: 16000
    
    # Daily limits
    daily_input_limit: 1000000
    daily_output_limit: 200000
    
  optimization:
    # Truncate context when approaching limits
    auto_truncate: true
    truncation_strategy: middle  # start | middle | end
    
    # Cache repeated requests
    cache_enabled: true
    cache_ttl_seconds: 3600
```

---

## Environment Variables

All configuration values support environment variable substitution:

```bash
# .env file
# Primary AI Provider
OPENAI_API_KEY=sk-your-openai-key
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key

# Azure (optional)
AZURE_OPENAI_KEY=your-azure-key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com

# GitHub for self-improvement
GITHUB_TOKEN=ghp_your-github-token
GITHUB_ORG=XORNG

# Local model (optional)
OLLAMA_HOST=http://localhost:11434

# Cost controls
DAILY_TOKEN_BUDGET=100000
DAILY_COST_BUDGET_USD=10.00
```

---

## Full Example Configuration

```yaml
# node.yaml - Complete Example
version: "1.0"

providers:
  - name: anthropic
    type: anthropic
    api_key: ${ANTHROPIC_API_KEY}
    default_model: claude-sonnet-4-20250514
    models:
      - name: claude-sonnet-4-20250514
        context_window: 200000
        cost_per_1k_input: 0.003
        cost_per_1k_output: 0.015
    retry:
      max_attempts: 3
      backoff_ms: 1000

  - name: openai
    type: openai
    api_key: ${OPENAI_API_KEY}
    default_model: gpt-4o-mini
    models:
      - name: gpt-4o-mini
        context_window: 128000
        cost_per_1k_input: 0.00015
        cost_per_1k_output: 0.0006
        
  - name: local
    type: ollama
    base_url: http://localhost:11434
    default_model: llama3.2

routing:
  rules:
    - task_type: code_generation
      provider: anthropic
    - task_type: analysis
      provider: openai
    - data_sensitivity: high
      provider: local
  fallback:
    chain: [anthropic, openai, local]

tokens:
  tracking:
    enabled: true
    tokenizer: tiktoken
  limits:
    daily_input_limit: 500000

self_improvement:
  enabled: true
  provider: anthropic
  github:
    token: ${GITHUB_TOKEN}
    organization: XORNG
    repositories: [core, node, validator-code-review]
    require_approval: true
  schedule:
    cron: "0 2 * * *"
```

---

## Next Steps

- [Automation Guide](./AUTOMATION.md) - Configure self-improvement workflows
- [Installation Guide](./INSTALLATION.md) - Complete setup instructions
- [API Reference](./API_REFERENCE.md) - Node API documentation
