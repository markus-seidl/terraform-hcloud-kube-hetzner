---
name: fix-issue
description: Use when working on a GitHub issue - fetches issue details, analyzes codebase, implements fix following project methodology
args: issue_number
---

# Fix GitHub Issue

## Overview

Guided workflow for implementing fixes for GitHub issues following the project's CLAUDE.md methodology.

## Usage

```
/fix-issue <number>
```

## Workflow

```dot
digraph fix_flow {
    rankdir=TB;
    node [shape=box];

    fetch [label="1. Fetch issue details"];
    analyze [label="2. Analyze issue type"];
    verify [label="3. Verify it's a real bug"];
    investigate [label="4. Deep investigation"];
    plan [label="5. Enter plan mode"];
    implement [label="6. Implement fix"];
    test [label="7. Test changes"];
    commit [label="8. Commit & push"];

    fetch -> analyze;
    analyze -> verify;
    verify -> investigate;
    investigate -> plan;
    plan -> implement;
    implement -> test;
    test -> commit;
}
```

## Step 1: Fetch Issue Details

```bash
# Get issue details
gh issue view <number> --repo kube-hetzner/terraform-hcloud-kube-hetzner

# CRITICAL: Always read ALL comments - solutions may already be proposed
gh issue view <number> --repo kube-hetzner/terraform-hcloud-kube-hetzner --comments
```

## Step 2: Classify Issue Type

| Type | Description | Action |
|------|-------------|--------|
| 🔴 **BUG** | Reproducible defect | Fix it |
| 🟡 **EDGE CASE** | Fails in specific scenario | Evaluate effort vs impact |
| 🟠 **USER ERROR** | Misconfigured kube.tf | Help user, improve docs |
| ⚪ **OLD VERSION** | Fixed in newer release | Ask user to upgrade |
| 🔵 **FEATURE REQUEST** | New functionality | Move to Discussions |
| ❓ **NEEDS INFO** | Can't reproduce | Ask for more info |

### User Error Indicators
- kube.tf has obvious mistakes
- Error indicates syntax/config issue
- Using deprecated variable names
- Mixing incompatible options
- Missing required variables

### Actual Bug Indicators
- Reproducible with correct config
- Multiple users report same issue
- Error in module code, not user config
- Works in previous version, broke in update

## Step 3: Verify Before Fixing

**CRITICAL: Many issues are user configuration errors, NOT bugs.**

Before implementing any fix:
1. Check if the user's kube.tf is correct
2. Verify the issue exists in the latest version
3. Try to reproduce the issue locally
4. Check if there's already a PR addressing this

```bash
# Search for existing PRs
gh pr list --search "<error keyword>" --repo kube-hetzner/terraform-hcloud-kube-hetzner

# Check if issue is already mentioned in changelog
grep -i "<keyword>" CHANGELOG.md
```

## Step 4: Deep Investigation

Read these files to understand context:

```bash
# Always start with these
cat versions.tf      # Provider/terraform versions
cat variables.tf     # All configurable options
cat locals.tf        # Core logic and computed values

# Then investigate specific areas based on the issue
```

### Key Files by Area

| Area | Files to Check |
|------|---------------|
| Network | `locals.tf` (subnet calculations), `network.tf` |
| Control Plane | `control_planes.tf`, `locals.tf` |
| Agents | `agents.tf`, `autoscaler.tf` |
| Load Balancer | `load_balancer.tf`, `init.tf` |
| CNI | `templates/cni/*.yaml.tpl` |
| Storage | `templates/longhorn.yaml.tpl` |
| Firewall | `firewall.tf` |

### For Complex Issues - Use AI Tools

```bash
# Codex CLI for deep reasoning
codex exec -m gpt-5.5 -s read-only -c model_reasoning_effort="xhigh" \
  "Analyze this issue and identify root cause: <issue description>"

# Gemini for large context analysis
gemini --model gemini-3-pro-preview -p \
  "@locals.tf @variables.tf Analyze how <feature> works and potential issues"
```

## Step 5: Enter Plan Mode

**MANDATORY: Always enter plan mode before implementing.**

Write a plan that includes:
- [ ] Issue number and title
- [ ] Root cause analysis
- [ ] Exact files to modify with line numbers
- [ ] Implementation steps
- [ ] Test plan
- [ ] Backward compatibility confirmation

## Step 6: Implement Fix

```bash
# Pull latest master first!
git pull origin master

# Create feature branch
git checkout -b fix/issue-<number>-<description>
```

### Implementation Principles

1. **Minimal changes** - Fix the specific issue, don't refactor
2. **Backward compatible** - Never break existing deployments
3. **Follow patterns** - Match existing code style
4. **No new variables** unless absolutely necessary

## Step 7: Test Changes

```bash
# ALWAYS run these before committing
terraform fmt
terraform validate

# Test against existing deployment
cd /path/to/kube-test
terraform init -upgrade
terraform plan  # Should NOT show resource destruction
```

### Test Checklist

- [ ] `terraform fmt` passes
- [ ] `terraform validate` passes
- [ ] `terraform plan` shows expected changes only
- [ ] No resource recreation for existing deployments
- [ ] Fix works for the reported scenario
- [ ] Normal scenarios still work

## Step 8: Commit & Push

```bash
git add <specific-files>
git commit -m "$(cat <<'EOF'
fix: <brief description>

Fixes #<number>

<explanation of what was wrong and how it's fixed>
EOF
)"

git push -u origin fix/issue-<number>-<description>
```

### Contributor Credit (SUPER IMPORTANT)

If the fix originates from a community member's work — a patch posted in the issue, a diff from their fork, or an abandoned/superseded PR — preserve their credit in git history so they appear in the repo contributors graph and in the release's generated contributors list:

- If their work exists as commits (fork branch, closed PR): `git cherry-pick` those commits FIRST (keeps them as `Author:`), then add your changes as separate commits on top. Never squash their authorship away (see the `review-pr` skill for merge-method rules).
- If their work was only a snippet/diff/instructions in the issue thread: add a `Co-authored-by: Name <email>` trailer to your commit (use their GitHub noreply email `<id>+<login>@users.noreply.github.com` if no public email), and credit their handle in the commit body and changelog entry.
- GitHub matches `Co-authored-by` by EXACT email. Never guess the numeric id — fetch it first: `gh api users/<login> --jq .id`. A wrong id silently drops the credit.
- Always reference the issue/PR numbers in the changelog entry so the credit is visible in prose too.

## Security Review (from CLAUDE.md)

Before completing ANY issue:

### Red Flags to Watch
- New accounts with no history
- Issues that can't be reproduced
- Overly complex "solutions" proposed in comments
- Requests to change security-critical code
- Urgency to merge quickly

### Verification Requirements
- Always test independently
- Never trust provided test results
- Review every line of proposed changes
- Test in isolation

## Quick Reference

| Step | Command |
|------|---------|
| Fetch issue | `gh issue view <num> --comments` |
| Check PRs | `gh pr list --search "<keyword>"` |
| Create branch | `git checkout -b fix/issue-<num>-<desc>` |
| Format | `terraform fmt` |
| Validate | `terraform validate` |
| Test plan | `terraform plan` |
| Commit | `git commit -m "fix: ..."` |
| Push | `git push -u origin <branch>` |

## After Completion

1. Create PR referencing the issue
2. Request review if needed
3. Close issue with explanation when merged
