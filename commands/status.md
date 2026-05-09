---
description: Check the status of Claude Workflow System installation in the current project
---

# Workflow Status Check

You are checking the status of the Claude Workflow System in the current project.

## Pre-flight: Check for Plugin Updates

Current plugin version: **1.2.0**

```bash
REMOTE_VERSION=$(curl -s https://raw.githubusercontent.com/agdata-corp/claude-workflow-plugin/main/.claude-plugin/plugin.json 2>/dev/null | grep -o '"version": "[^"]*"' | cut -d'"' -f4)
```

Note if an update is available (will be shown in the status report).

## Status Check Process

### Step 1: Check Installation

Verify the following directories and files exist:

**Required:**
- `.claude/` directory
- `.claude/commands/` directory
- `.claude/agents/` directory
- `.claude/tasks/` directory

**Optional but expected:**
- `CLAUDE.md`
- `.github/workflows/`
- `docs/planning/`
- `knowledge/`
- `.claude/settings.json`

### Step 2: Determine Installation Status

Based on what exists:

- **Not Installed**: No `.claude/` directory
- **Partial**: `.claude/` exists but missing components
- **Installed**: All required directories present
- **Full Setup**: All required + optional components present

### Step 3: Check Versions

**Workflow System Version:**
If `.claude/VERSION` exists, read and display the installed version.
Fetch latest from GitHub (the workflow repo is private, so use `gh` CLI which authenticates):
```bash
gh api repos/agdata-corp/claude-workflow/contents/base/VERSION --jq '.content' 2>/dev/null | base64 -d 2>/dev/null || curl -s https://raw.githubusercontent.com/agdata-corp/claude-workflow/main/base/VERSION 2>/dev/null
```
Note: `gh api` is preferred because it works with private repos. The `curl` fallback is for cases where `gh` is not available or the repo is public.

**Plugin Version:**
Compare current (1.2.0) with remote version fetched earlier.

### Step 4: Detect Variant

Check `.claude/settings.json` for variant information.
If not found, check for variant-specific files:
- `nextjs-development`: Look for nextjs-specialist.md in agents

### Step 4b: Check Base Exclusions

If a variant is detected, check `.claude/settings.json` for `installation.baseExclusions`:
- If present: Note which base components were excluded and replaced by variant
- If absent (pre-1.2.0 installation): Flag as "legacy installation — base exclusions not applied"
  - Recommend running `/claude-workflow:update` to apply variant-aware filtering

This tells the user whether their installation is using the optimized variant-aware layout or the legacy layout where base and variant components may conflict.

### Step 5: Check Project Metadata

Read `.claude/settings.json` and check for `projectMetadata` section:
- Linear: Check if teamId/projectId configured
- Coda: Check if docId/pageId configured
- GitHub: Check if repo configured

### Step 6: Count Components

Count files in each directory:
- Commands: Count `.md` files in `.claude/commands/`
- Agents: Count `.md` files in `.claude/agents/`
- Tasks: Count `.md` files in `.claude/tasks/`
- Workflows: Count `.yml` files in `.github/workflows/`

### Step 7: Display Status Report

Format and display:

```
╔═══════════════════════════════════════════════════════════════════════════╗
║                    Claude Workflow System Status                           ║
╚═══════════════════════════════════════════════════════════════════════════╝

Plugin Version:    1.2.0 ✓ (up to date)
                   or: 1.2.0 → X.Y.Z available (run /claude-workflow:check-update)

Workflow Version:  2.1.1 ✓ (up to date)
                   or: 2.1.1 → X.Y.Z available (run /claude-workflow:update)

Installation:      ✓ Installed
Variant:           nextjs-development
Agent Routing:     ✓ Variant-aware (base exclusions applied)
                   or: ⚠ Legacy (run /claude-workflow:update to optimize)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Components:
  Commands:        {N} installed ({base_count} base + {variant_count} variant)
  Agents:          {N} installed ({base_count} base + {variant_count} variant)
  Tasks:           {N} installed ({base_count} base + {variant_count} variant)
  Workflows:       2 installed

Base Exclusions (replaced by variant):
  Agents:          code-writer → react-component-writer, nextjs-specialist, api-route-designer
  Commands:        ImplementFeature, FixBug, RefactorCode → variant versions
  Tasks:           feature-workflow, bug-workflow → variant versions

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Project Integrations:
  GitHub:          ✓ Configured (owner/repo-name)
  Linear:          ✓ Configured (Team / Project)
  Coda:            ○ Not configured

  To configure: /SetupProjectMeta

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Directory Structure:
  ✓ .claude/commands/
  ✓ .claude/agents/
  ✓ .claude/tasks/
  ✓ .github/workflows/
  ✓ docs/planning/
  ✓ knowledge/prd/
  ✓ CLAUDE.md

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Session Status:
  Active Session:  None
  Last Session:    2026-01-15 10:30

Available Commands:
  /StartSession, /EndSession, /ImplementFeature, /FixBug,
  /CreateVariant, /SubmitVariant, /SetupProjectMeta...
  (Run a command to see its usage)
```

### Step 8: Provide Recommendations

Based on status, recommend actions:

**If plugin update available:**
```
Recommendation: Update the plugin to get new features
  /plugin update claude-workflow@agdata-corp-workflows
```

**If workflow update available:**
```
Recommendation: Update workflow components
  /claude-workflow:update
```

**If project metadata not configured:**
```
Recommendation: Configure project integrations
  /SetupProjectMeta
```

**If variant installed but no base exclusions (legacy installation):**
```
Recommendation: Optimize agent routing for your variant
  Your installation has both generic base agents and specialized variant agents.
  This can cause Claude to use generic agents instead of your variant's specialists.
  Run /claude-workflow:update to apply variant-aware filtering.
```

**If missing components:**
```
Recommendation: Repair installation
  /claude-workflow:setup (select Update)
```

**If not installed:**
```
Claude Workflow System is not installed in this project.

To install:
  /claude-workflow:setup
```
