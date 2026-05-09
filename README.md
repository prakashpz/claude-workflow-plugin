***

**CLIENT REPOSITORY NOTICE**

**Prepared for: AGData**

This repository is provided as a starter project for your team to own, manage,
and modify based on your training. Going forward, your team is fully responsible
for all maintenance, customization, and support of this codebase.

| | |
|---|---|
| **Version** | Plugin v1.2.0 — Compatible with Workflow System v2.2.1 |
| **Support Contact** | Jared Munn (Jared.Munn@agdata.com) |
| **Original Creator** | Johann Beukes (johann@red-blue.ai) — Attribution only, not for support |
| **License** | Apache 2.0 — See LICENSE and NOTICE files |

**DISCLAIMER**: This software is provided "AS IS" without warranties of any kind,
express or implied. There are no guarantees of fitness for any particular purpose.
Your team is solely responsible for determining its appropriateness for your use.
See the LICENSE file for complete terms.

***

# Claude Workflow Plugin

![Plugin](https://img.shields.io/badge/Plugin-1.2.0-green) ![Workflow System](https://img.shields.io/badge/Workflow_System-2.0.0-blue) ![License](https://img.shields.io/badge/License-MIT-yellow)

A Claude Code plugin that sets up and manages the Claude Workflow System in your projects. This plugin acts as an installer - it fetches the workflow system from GitHub and configures it in your target project.

## Table of Contents

- [Overview](#overview)
- [What's New in 1.2.0](#whats-new-in-120)
- [Installation](#installation)
- [Commands Reference](#commands-reference)
- [Quick Start Guide](#quick-start-guide)
- [Creating Custom Variants](#creating-custom-variants)
- [How It Works](#how-it-works)
- [Troubleshooting](#troubleshooting)
- [Development](#development)

---

## Overview

The Claude Workflow Plugin provides a streamlined way to set up the Claude Workflow System in any project. Instead of manually cloning repositories or running scripts, you simply install the plugin once and then run setup commands in any project directory.

### Key Features

- **One-time installation**: Install the plugin once, use it across all your projects
- **Interactive setup**: Guided prompts help you choose the right configuration
- **Variant selection**: Choose from multiple project-type variants (base, Next.js, etc.)
- **Create custom variants**: Build variants for unsupported project types (Swift, Flutter, Rust, etc.)
- **Project integrations**: Configure Linear, Coda, and GitHub during setup
- **Auto-update notifications**: Get notified when new versions are available
- **Update support**: Keep your workflow installation current with the latest changes
- **Status checking**: Verify your installation and see what's configured

### Important Distinction

**The plugin is the installer, NOT the installed content.**

When you run `/claude-workflow:setup`, the plugin:
1. Fetches the workflow system from GitHub
2. Copies only the `base/` and `variants/` content to your project
3. Does NOT copy the plugin itself or other installer components

Your project receives:
- `.claude/commands/` - Workflow commands
- `.claude/agents/` - AI agents
- `.claude/tasks/` - Orchestration tasks
- `.github/workflows/` - CI/CD workflows
- `docs/` and `knowledge/` - Documentation structure
- `CLAUDE.md` - Project guidance

---

## What's New in 1.2.0

### Variant-Aware Base Component Filtering

Previously, when installing a variant (e.g., `nextjs-development`), all base components were installed alongside variant-specific ones. This caused Claude to use the generic `code-writer` agent instead of the variant's specialized agents (like `react-component-writer` or `nextjs-specialist`), because base commands hardcoded routing to `code-writer`.

**Now**: The plugin intelligently excludes base components that are replaced by the variant:
- If a variant provides coding agents, the base `code-writer` is excluded
- If a variant provides its own `ImplementFeature`, `FixBug`, etc., base versions are excluded
- Variants declare exclusions via `baseExclusions` in their manifest

```
Variant-Aware Installation:
  Variant 'nextjs-development' provides specialized agents.
  Base components skipped (replaced by variant):
    Agents:   code-writer → react-component-writer, nextjs-specialist, api-route-designer
    Commands: ImplementFeature, FixBug, RefactorCode → variant versions
    Tasks:    feature-workflow, bug-workflow → variant versions
```

### Legacy Installation Upgrade

Existing installations (pre-1.2.0) can be upgraded:
1. Update the plugin: `/plugin update claude-workflow@agdata-corp-workflows`
2. Run `/claude-workflow:update` in your project
3. The update detects the legacy layout and applies variant-aware filtering

### Status Report Enhancements

The status command now shows:
- **Agent Routing** status (variant-aware vs legacy)
- **Base Exclusions** listing (which components were replaced by variant)
- Recommendations for legacy installations

---

## Installation

### Method 1: From Plugin Marketplace (Recommended)

First, add the RedBlue workflows marketplace to Claude Code:

```
/plugin marketplace add https://github.com/agdata-corp/claude-workflow-plugin.git
```

Then install the plugin:

```
/plugin install claude-workflow@agdata-corp-workflows
```

### Method 2: Local Development/Testing

If you've cloned the repository locally:

```bash
claude --plugin-dir /path/to/claude-workflow-plugin
```

### Updating the Plugin

When you see an update notification:

```
/plugin update claude-workflow@agdata-corp-workflows
```

Or reinstall:
```
/plugin uninstall claude-workflow
/plugin install claude-workflow@agdata-corp-workflows
```

### Verify Installation

After installation, verify the plugin is available:

```
/claude-workflow:status
```

If you see the status output (even if it says "Not Installed"), the plugin is working correctly.

---

## Commands Reference

### `/claude-workflow:setup`

**Description**: Set up the Claude Workflow System in the current project directory.

**Usage**:
```
/claude-workflow:setup
```

**Process**:
1. Checks for plugin updates (notifies if available)
2. Checks for git (required for fetching from GitHub)
3. Detects if this is a fresh setup or update/replace
4. Fetches the latest workflow system from GitHub
5. Presents available variants for selection (including "Create New Variant")
6. Asks for project name and description
7. Copies and configures all workflow components
8. Configures project integrations (Linear, Coda, GitHub)
9. Generates customized CLAUDE.md

**Options during setup**:
- **Fresh setup**: Creates new `.claude/` directory structure
- **Update**: Syncs new components while preserving your customizations
- **Replace**: Backs up existing setup and performs fresh install

**Variant options**:
- **base**: Core workflow system for any project type
- **nextjs-development**: For Next.js projects
- **Create New Variant**: Build a custom variant for your project type

---

### `/claude-workflow:update`

**Description**: Update an existing workflow installation to the latest version.

**Usage**:
```
/claude-workflow:update
```

**Options**:
- **Full update**: Updates all components (commands, agents, tasks, workflows)
- **Selective update**: Choose specific components to update
- **Preserve customizations**: Your local changes are preserved by default

Also checks for plugin updates and notifies if new version available.

---

### `/claude-workflow:variants`

**Description**: List all available project variants.

**Usage**:
```
/claude-workflow:variants
```

**Output**: Displays each variant with:
- Name and description
- Project type it's designed for
- Components included (agents, commands, tasks)
- Option to create custom variants

**Current Variants**:

| Variant | Type | Description |
|---------|------|-------------|
| `base` | Generic | Core workflow system for any project type |
| `nextjs-development` | Next.js | Full-stack Next.js with App Router, React, TypeScript |

---

### `/claude-workflow:status`

**Description**: Check the current workflow installation status.

**Usage**:
```
/claude-workflow:status
```

**Output includes**:
- Plugin version (with update notification if available)
- Workflow version (with update notification if available)
- Installation status (Not Installed / Partial / Installed / Full Setup)
- Detected variant
- Component counts (commands, agents, tasks, workflows)
- Project integrations status (GitHub, Linear, Coda)
- Directory structure verification
- Session status (if any active sessions)
- Available commands list
- Recommendations

---

### `/claude-workflow:check-update`

**Description**: Manually check for plugin updates.

**Usage**:
```
/claude-workflow:check-update
```

**Output**: Shows current vs latest version and update instructions.

---

## Quick Start Guide

### Step 1: Install the Plugin

In Claude Code, add the marketplace and install the plugin:

```
/plugin marketplace add https://github.com/agdata-corp/claude-workflow-plugin.git
/plugin install claude-workflow@agdata-corp-workflows
```

### Step 2: Navigate to Your Project

```bash
cd /path/to/your/project
```

### Step 3: Run Setup

```
/claude-workflow:setup
```

Follow the interactive prompts:

1. **Setup Type** (if `.claude/` exists):
   - Update - Sync new components
   - Replace - Fresh install with backup
   - Cancel - Abort

2. **Select Variant**:
   - `base` - For any project type
   - `nextjs-development` - For Next.js projects
   - `Create New Variant` - For unsupported project types

3. **Project Information**:
   - Project name (defaults to directory name)
   - Project description

4. **Project Integrations**:
   - GitHub (auto-detected)
   - Linear team and project
   - Coda document and page

### Step 4: Configure GitHub (Optional)

If you want CI/CD workflows for automated code review:

1. Go to your GitHub repository Settings
2. Navigate to Secrets and variables → Actions
3. Add `ANTHROPIC_API_KEY` with your API key

### Step 5: Start Using Workflows

Run your first workflow command:
```
/StartSession
```

---

## Creating Custom Variants

Don't see a variant for your project type? Create one during setup!

### During Setup

1. Run `/claude-workflow:setup`
2. Select **"Create New Variant"**
3. Provide variant details:
   - Name (e.g., `swift-ios`, `flutter-mobile`)
   - Description
   - Project type
4. Setup continues with your new variant

### After Setup

Enhance your variant by adding:
- Custom agents in `.claude/agents/`
- Custom commands in `.claude/commands/`
- Custom tasks in `.claude/tasks/`

### Contributing Your Variant

Share your variant with the community:

```
/SubmitVariant your-variant-name
```

This creates a branch and pull request - all contributions go through review.

---

## How It Works

### Architecture

```
Claude Code Plugin System
         │
         ▼
┌─────────────────────────────┐
│   claude-workflow plugin    │  ← Installed in Claude Code
│   (installer/manager)       │
│   Version: 1.2.0            │
└─────────────────────────────┘
         │
         │ Fetches from GitHub
         ▼
┌─────────────────────────────┐
│  agdata-corp/claude-workflow  │  ← GitHub Repository
│  ├── base/                  │
│  ├── variants/              │
│  ├── scripts/ (NOT copied)  │
│  └── docs/ (NOT copied)     │
└─────────────────────────────┘
         │
         │ Copies base/ + selected variant/
         ▼
┌─────────────────────────────┐
│     Your Project            │  ← Target Project
│  ├── .claude/               │
│  │   ├── commands/          │
│  │   ├── agents/            │
│  │   ├── tasks/             │
│  │   └── settings.json      │
│  ├── .github/workflows/     │
│  ├── docs/                  │
│  ├── knowledge/             │
│  └── CLAUDE.md              │
└─────────────────────────────┘
```

### Version Checking

The plugin checks for updates by fetching the remote `plugin.json`:
- Compares local version (1.2.0) with remote version
- Notifies if update available
- Provides update instructions

### What Gets Copied

**Base layer (filtered by variant)**:
- Commands, agents, and tasks from `base/` — minus any excluded by the active variant
- 2 GitHub workflows (code-review, security-review) — always included
- Project templates and directory structure

**Variant layer (if selected)**:
- Specialized agents, commands, and tasks that replace excluded base components
- Variant-specific CLAUDE.md template with agent routing section

**With base-only install**: All 29 commands, 10 agents, 20 tasks installed
**With variant install**: Base components filtered — e.g., Next.js variant excludes `code-writer`, `ImplementFeature`, `FixBug`, `RefactorCode`, `feature-workflow`, `bug-workflow` from base and provides specialized replacements

**Never copied**:
- `scripts/` - Python installation scripts
- `docs/` - Repository documentation (not project docs)

---

## Troubleshooting

### Plugin Not Found

**Symptom**: Commands like `/claude-workflow:setup` aren't recognized

**Solutions**:
1. Verify plugin installation: `/plugin list`
2. Re-add marketplace and reinstall:
   ```
   /plugin marketplace add https://github.com/agdata-corp/claude-workflow-plugin.git
   /plugin install claude-workflow@agdata-corp-workflows
   ```

### Update Notification But Can't Update

**Symptom**: See update available but `/plugin update` doesn't work

**Solutions**:
1. Uninstall and reinstall:
   ```
   /plugin uninstall claude-workflow
   /plugin install claude-workflow@agdata-corp-workflows
   ```

### Git Clone Fails

**Symptom**: "Failed to clone repository" error during setup

**Solutions**:
1. Check internet connection
2. Verify git is installed: `git --version`
3. Check GitHub access

### Project Integrations Not Working

**Symptom**: Can't configure Linear or Coda during setup

**Solutions**:
1. Ensure MCP servers are configured for Linear/Coda
2. Skip during setup, configure later with `/SetupProjectMeta`

---

## Development

### Plugin Structure

```
claude-workflow-plugin/
├── .claude-plugin/
│   ├── plugin.json      # Plugin manifest (version 1.2.0)
│   └── marketplace.json # Marketplace config
├── commands/
│   ├── setup.md         # Main setup command
│   ├── update.md        # Update command
│   ├── variants.md      # List variants
│   ├── status.md        # Status check
│   └── check-update.md  # Version check
├── agents/
│   └── workflow-setup.md # Intelligent setup agent
├── CHANGELOG.md         # Version history
└── README.md            # This file
```

### Testing Locally

1. Clone the repository:
   ```bash
   git clone https://github.com/agdata-corp/claude-workflow-plugin.git
   ```

2. Run Claude Code with the plugin:
   ```bash
   claude --plugin-dir /path/to/claude-workflow-plugin
   ```

3. Test in a sample project directory

### Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

---

## Requirements

| Component | Requirement |
|-----------|-------------|
| Claude Code | Latest version with plugin support |
| Git | Required for fetching from GitHub |
| Internet | Required during setup (fetches from GitHub) |

---

## Support

- [GitHub Issues](https://github.com/agdata-corp/claude-workflow-plugin/issues)
- [Main Workflow Repository](https://github.com/agdata-corp/claude-workflow)
- [Changelog](CHANGELOG.md)
