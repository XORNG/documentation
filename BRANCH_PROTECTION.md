# Branch Protection Configuration Guide

This document describes the recommended branch protection settings for all XORNG repositories.

## Required Branch Protection Rules

### For `main` Branch

Apply these settings via **Settings > Branches > Add branch protection rule**:

#### Basic Protection
- [x] **Require a pull request before merging**
  - [x] Require approvals: **1** (minimum, 2 recommended for critical repos)
  - [x] Dismiss stale pull request approvals when new commits are pushed
  - [x] Require review from Code Owners
  - [x] Require approval of the most recent reviewable push

#### Status Checks
- [x] **Require status checks to pass before merging**
  - [x] Require branches to be up to date before merging
  - Required checks:
    - `CI Success` (from ci.yml)
    - `CodeQL Analysis` (from security.yml)
    - `Dependency Review` (from security.yml, for PRs)
    - `npm Security Audit` (from security.yml)

#### Additional Protections
- [x] **Require conversation resolution before merging**
- [x] **Require signed commits** (recommended for high-security repos)
- [x] **Require linear history** (recommended)
- [x] **Do not allow bypassing the above settings**
- [x] **Restrict who can push to matching branches**
  - Only allow repository admins and specific teams

#### Force Push and Deletion
- [ ] Allow force pushes (DISABLED)
- [ ] Allow deletions (DISABLED)

### For `develop` Branch

Similar to `main` but with slightly relaxed settings:

- [x] Require a pull request before merging
  - [x] Require approvals: **1**
  - [x] Dismiss stale approvals
- [x] Require status checks to pass
  - Required: `CI Success`
- [ ] Require signed commits (optional)

## Rulesets (Alternative to Branch Rules)

GitHub Rulesets provide more flexibility. Create a ruleset for:

```yaml
name: Protect main branches
enforcement: active
bypass_actors:
  - actor_type: OrganizationAdmin
    bypass_mode: always
target: branch
ref_name:
  include:
    - refs/heads/main
    - refs/heads/develop
rules:
  - type: pull_request
    parameters:
      required_approving_review_count: 1
      dismiss_stale_reviews_on_push: true
      require_code_owner_review: true
  - type: required_status_checks
    parameters:
      strict_required_status_checks_policy: true
      required_status_checks:
        - context: "CI Success"
        - context: "CodeQL Analysis"
  - type: non_fast_forward
  - type: deletion
```

## Environment Protection

For deployment environments (production, staging):

1. Go to **Settings > Environments**
2. Create environments with:
   - Required reviewers
   - Wait timer (e.g., 5 minutes for production)
   - Deployment branches (only `main` for production)

## Secret Scanning

Enable in **Settings > Security > Secret scanning**:

- [x] Secret scanning
- [x] Push protection (blocks commits containing secrets)

## Dependabot

Enable in **Settings > Security > Dependabot**:

- [x] Dependabot alerts
- [x] Dependabot security updates
- [x] Dependabot version updates (configured in `.github/dependabot.yml`)

## Code Scanning

Enable in **Settings > Security > Code scanning**:

- [x] CodeQL analysis (configured in `.github/workflows/security.yml`)
- Set alert severity threshold to **High** or above

## Security Advisories

Enable private vulnerability reporting:

1. **Settings > Security > Private vulnerability reporting**
2. Enable to allow security researchers to report issues privately

## Automation Script

For organizations with many repositories, use the GitHub CLI to apply settings:

```bash
#!/bin/bash
# apply-branch-protection.sh

REPOS=(
  "XORNG/core"
  "XORNG/automation"
  "XORNG/extension-vscode"
  "XORNG/node"
  "XORNG/template-base"
  "XORNG/template-knowledge"
  "XORNG/template-task"
  "XORNG/template-validator"
  "XORNG/knowledge-best-practices"
  "XORNG/knowledge-documentation"
  "XORNG/validator-code-review"
  "XORNG/validator-security"
  "XORNG/documentation"
)

for repo in "${REPOS[@]}"; do
  echo "Applying branch protection to $repo..."
  
  gh api \
    --method PUT \
    -H "Accept: application/vnd.github+json" \
    "/repos/$repo/branches/main/protection" \
    -f required_status_checks='{"strict":true,"contexts":["CI Success","CodeQL Analysis"]}' \
    -f enforce_admins=true \
    -f required_pull_request_reviews='{"dismiss_stale_reviews":true,"require_code_owner_reviews":true,"required_approving_review_count":1}' \
    -f restrictions=null \
    -f required_linear_history=true \
    -f allow_force_pushes=false \
    -f allow_deletions=false
done
```

## Verification Checklist

After setup, verify:

- [ ] Direct pushes to `main` are blocked
- [ ] PRs require at least 1 approval
- [ ] Status checks must pass before merge
- [ ] Force pushes are disabled
- [ ] Dependabot is creating PRs for vulnerable dependencies
- [ ] CodeQL is running and reporting issues
- [ ] Secret scanning is active
- [ ] CODEOWNERS file is being respected
