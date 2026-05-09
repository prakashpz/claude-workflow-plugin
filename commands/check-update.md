---
description: Check if a newer version of the Claude Workflow plugin is available
---

# Check for Plugin Updates

You are checking if there's a newer version of the Claude Workflow plugin available.

## Process

### Step 1: Get Current Plugin Version

The current plugin version is defined in the plugin configuration. Read and remember it.

Current plugin version: **1.2.0**

### Step 2: Fetch Remote Version

Fetch the latest plugin.json from GitHub to check the remote version:

```bash
curl -s https://raw.githubusercontent.com/prakashpz/claude-workflow-plugin/main/.claude-plugin/plugin.json 2>/dev/null || echo '{"version": "unknown"}'
```

Parse the JSON response to extract the `version` field.

### Step 3: Compare Versions

Compare the current version (1.2.0) with the remote version.

- If remote version > current version: Update is available
- If remote version = current version: Plugin is up to date
- If remote version < current version: You have a newer version (development)
- If remote version is "unknown": Could not check (network issue)

### Step 4: Display Result

**If update available:**
```
╔════════════════════════════════════════════════════════════╗
║  Plugin Update Available!                                   ║
║                                                             ║
║  Current version: 1.2.0                                     ║
║  Latest version:  X.Y.Z                                     ║
║                                                             ║
║  To update, run:                                            ║
║    /plugin update claude-workflow@agdata-corp-workflows         ║
║                                                             ║
║  Or reinstall:                                              ║
║    /plugin uninstall claude-workflow                        ║
║    /plugin install claude-workflow@agdata-corp-workflows        ║
╚════════════════════════════════════════════════════════════╝
```

**If up to date:**
```
✓ Claude Workflow plugin is up to date (version 1.2.0)
```

**If network error:**
```
⚠ Could not check for updates. Check your network connection.
  Current version: 1.2.0
```

## What's New Section

If an update is available, also try to fetch and display the latest changes:

```bash
curl -s https://raw.githubusercontent.com/prakashpz/claude-workflow-plugin/main/CHANGELOG.md 2>/dev/null | head -50
```

Show the most recent changelog entries if available.
