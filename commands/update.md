---
description: Update an existing Claude Workflow System installation with the latest components from GitHub
---

# Claude Workflow Update

You are updating an existing Claude Workflow System installation in the current project.

## Pre-flight: Check for Plugin Updates

Before starting, check if a newer version of this plugin is available.

Current plugin version: **1.2.0**

```bash
REMOTE_VERSION=$(curl -s https://raw.githubusercontent.com/prakashpz/claude-workflow-plugin/main/.claude-plugin/plugin.json 2>/dev/null | grep -o '"version": "[^"]*"' | cut -d'"' -f4)
echo "Remote version: $REMOTE_VERSION"
```

If a newer version is available, display:
```
╭─────────────────────────────────────────────────────────────╮
│  ⚠ Plugin update available: 1.2.0 → {remote_version}        │
│                                                              │
│  The plugin itself has updates. To get new plugin features:  │
│  /plugin update claude-workflow@agdata-corp-workflows            │
│                                                              │
│  Continuing with workflow component update...                │
╰─────────────────────────────────────────────────────────────╯
```

Then continue with the update (don't block).

## Update Process

### Step 1: Verify Existing Installation

Check that `.claude/` directory exists. If not, inform the user:
"No existing workflow installation found. Use `/claude-workflow:setup` to create a new installation."

### Step 2: Detect Current Configuration

Read `.claude/settings.json` if it exists to determine:
- Current variant (if any)
- Custom configurations
- Project metadata settings

If settings.json doesn't exist, assume base installation.

### Step 3: Fetch Latest from GitHub

```bash
rm -rf /tmp/claude-workflow-update
git clone --depth 1 https://github.com/prakashpz/claude-workflow.git /tmp/claude-workflow-update
```

### Step 4: Compare Versions

Check the VERSION file:
- Current: Read from `.claude/VERSION` if exists
- Latest: Read from `/tmp/claude-workflow-update/base/VERSION`

Report version information to user:
```
Workflow System Versions:
  Current: {installed_version}
  Latest:  {latest_version}

New in this version:
  • (list changes from remote CHANGELOG or VERSION notes)
```

### Step 5: Ask Update Scope

Use AskUserQuestion to ask what to update:

- **All components**: Commands, agents, tasks, and workflows
- **Commands only**: Just update commands
- **Agents only**: Just update agents
- **Tasks only**: Just update tasks
- **Workflows only**: Just update GitHub workflows

### Step 6: Perform Update (Variant-Aware)

**Before copying any files, determine base exclusions.** This ensures variant-replaced base components are not re-added during updates.

#### Step 6a: Load Base Exclusions

Check for existing exclusions in `.claude/settings.json` under `installation.baseExclusions`. If present, use them.

If not present (upgrading from pre-1.2.0), detect the active variant:
1. Read `.claude/settings.json` for variant info, or detect from installed variant agent files
2. Fetch the variant's `manifest.json` from `/tmp/claude-workflow-update/variants/{variant}/manifest.json`
3. Build exclusion lists using the same logic as setup Step 6a:
   - Check for `baseExclusions` field in manifest
   - Check for `replaces` declarations in variant components
   - Apply smart defaults: if variant has agents, exclude base `code-writer`; if variant has commands/tasks with matching names, exclude those base files
4. Save the detected exclusions to `.claude/settings.json` under `installation.baseExclusions` for future updates

Display what will be excluded:
```
Variant-aware update:
  Active variant: {variant_name}
  Base components excluded (replaced by variant):
    Agents: {list}
    Commands: {list}
    Tasks: {list}
```

#### Step 6b: Copy Updated Components

For each selected component type, copy files from the temp directory **respecting exclusion lists**:

```
# For each component type selected for update:
For each file in /tmp/claude-workflow-update/base/{type}/*:
  If filename (without .md) is NOT in the exclusion list for that type:
    Copy to .claude/{type}/

# Also update variant components if variant directory exists:
If variant was detected and /tmp/claude-workflow-update/variants/{variant}/ exists:
  Copy all variant agents/commands/tasks to .claude/ (overlay)
```

Use overlay strategy:
- New files are added
- Existing files are updated
- Local customizations in CLAUDE.md are preserved
- **Excluded base components are never re-added**

Important: Preserve `.claude/settings.json` projectMetadata section and installation section.

### Step 7: Check Project Metadata

After updating, check if project metadata is configured:

```bash
grep -q '"projectMetadata"' .claude/settings.json 2>/dev/null
```

If project metadata is not configured, inform user:
```
New feature available: Project Integrations

Configure Linear, Coda, and GitHub integrations for your project:
  /SetupProjectMeta

This enables:
  • Issue tracking scoped to your Linear project
  • PRD storage in your Coda document
  • Automatic GitHub repository detection
```

### Step 7b: Clean Up Legacy Base Components

If this is upgrading a pre-1.2.0 installation (no `installation.baseExclusions` was found in Step 6a), **remove the superseded base files** that were previously installed alongside the variant:

```
# For each excluded agent:
For each agent_name in excluded_agents:
  If .claude/agents/{agent_name}.md exists:
    Remove it (replaced by variant agent)
    Log: "Removed base agent '{agent_name}' (replaced by variant)"

# For each excluded command:
For each command_name in excluded_commands:
  If .claude/commands/{command_name}.md exists:
    Remove it (replaced by variant command)
    Log: "Removed base command '{command_name}' (replaced by variant)"

# For each excluded task:
For each task_name in excluded_tasks:
  If .claude/tasks/{task_name}.md exists:
    Remove it (replaced by variant task)
    Log: "Removed base task '{task_name}' (replaced by variant)"
```

Display cleanup results:
```
Legacy installation upgraded to variant-aware layout:
  Removed {N} superseded base components
  Variant agents will now be used for coding tasks
```

### Step 8: Update Version Marker

Copy VERSION file to `.claude/VERSION`:
```bash
cp /tmp/claude-workflow-update/base/VERSION .claude/VERSION
```

### Step 9: Cleanup

```bash
rm -rf /tmp/claude-workflow-update
```

### Step 10: Report Results

Show:
- Components updated
- New version installed
- Any new files that were added
- New commands available (if any)
- Reminder to review changes

```
Update Complete!

  Version: {old_version} → {new_version}

  Components updated:
    ✓ Commands ({N} files, {excluded} excluded by variant)
    ✓ Agents ({N} files, {excluded} excluded by variant)
    ✓ Tasks ({N} files, {excluded} excluded by variant)

  Variant-aware filtering:
    ✓ Base exclusions applied for '{variant_name}' variant
    Excluded: {list of excluded base components}

  Next steps:
    • Review updated commands with /help
    • Run /claude-workflow:status to verify
```

## Preserving Customizations

The update process preserves:
- `CLAUDE.md` (never overwritten)
- `.claude/settings.json` (merged, not replaced — both projectMetadata and installation sections preserved)
- Project metadata configuration
- Any custom agents/commands not from base/variant
- Variant-specific components (always updated from latest variant source)

The update process **removes** (only for legacy upgrades):
- Base components that are superseded by the active variant's specialized versions
- This is intentional — these base files cause Claude to use generic agents instead of variant specialists
