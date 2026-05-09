# Changelog

All notable changes to the Claude Workflow Plugin will be documented in this file.

## [1.2.0] - 2026-02-26

### Added
- **Variant-aware base component filtering**: When installing a variant, the plugin now excludes base components that are replaced by the variant's specialized versions. This ensures Claude uses variant-specific agents (e.g., `react-component-writer`, `nextjs-specialist`) instead of the generic base `code-writer` agent.
- **`baseExclusions` manifest support**: Variant manifests can now include a `baseExclusions` field to explicitly declare which base agents, commands, and tasks should be skipped during installation.
- **Smart exclusion defaults**: Even without explicit `baseExclusions`, the plugin automatically detects overlap — if a variant provides coding agents, the base `code-writer` is excluded; if variant commands/tasks match base names, base versions are excluded.
- **Installation metadata**: Setup now records exclusion information in `.claude/settings.json` under `installation.baseExclusions` so updates respect the same filtering.
- **Agent routing section in CLAUDE.md**: Variant installations now include an "Agent Routing" section in CLAUDE.md that documents which variant agents to use for coding tasks.
- **Legacy installation detection**: Status command detects pre-1.2.0 installations with both base and variant agents, and recommends running update to apply variant-aware filtering.

### Changed
- **Setup Step 6 redesigned**: Now performs variant-aware filtering before copying base components, preventing generic agents from shadowing specialized variant agents.
- **Update command**: Now respects base exclusions during updates, preventing re-introduction of excluded base components. Also retroactively applies exclusions for pre-1.2.0 installations.
- **Status report**: Now shows "Agent Routing" status (variant-aware vs legacy) and lists which base components were excluded.
- **Variants display**: Now shows which base components each variant replaces, not just what it adds.

### Fixed
- **Variant agents underutilized**: Previously, base commands (`ImplementFeature`, `FixBug`, `RefactorCode`) and tasks (`feature-workflow`, `bug-workflow`) hardcoded routing to the base `code-writer` agent. When a variant was installed, Claude would follow these references and use the generic agent instead of the variant's specialized agents. The new filtering removes these base components when a variant provides replacements.

### How to Update
Existing users should update the plugin:

```
/plugin update claude-workflow@agdata-corp-workflows
```

After updating the plugin, run `/claude-workflow:update` in each project to apply variant-aware filtering to existing installations. This will remove superseded base components and record the exclusions.

## [1.1.0] - 2026-02-04 (Workflow System 2.0.0 compatibility)

### Workflow System Update
- **Compatible with Workflow System 2.0.0**: Running `/claude-workflow:setup` or `/claude-workflow:update` now installs the v2.0.0 base layer which includes the full PRD-to-Cycle planning workflow
- **New base layer content**: 23 commands (was 15), 10 agents (was 6), 18 tasks (was 12), 5 schemas (was 3)
- **New planning commands**: PRDIntake, PRDValidate, PRDEnrich, PRDFeasibility, PRDSequence, Breakdown, CyclePlan, CycleCommit, CycleStatus, CycleSummary, CycleRetro
- **New planning agents**: prd-validator, technical-analyst, cycle-planner, status-aggregator
- **Existing installations**: Run `/claude-workflow:update` to get the new planning workflow components

## [1.1.0] - 2026-01-17

### Added
- **Auto-update notifications**: Plugin now checks for updates and notifies users when a new version is available
- **Create New Variant**: During setup, users can now create custom variants for unsupported project types (Swift, Flutter, Rust, Go, etc.)
- **Project metadata configuration**: Setup now includes Linear, Coda, and GitHub integration configuration
- **New command**: `/claude-workflow:check-update` - Manually check for plugin updates
- **Version display**: All commands now show current plugin version and notify if updates are available

### Changed
- Setup process now includes Step 7 for configuring project integrations
- Variants command now shows option to create custom variants
- Status command now shows project integration status (Linear, Coda, GitHub)
- Update command now checks for and notifies about plugin updates separately from workflow updates

### How to Update
Existing users should update the plugin to get these new features:

```
/plugin update claude-workflow@agdata-corp-workflows
```

Or reinstall:
```
/plugin uninstall claude-workflow
/plugin install claude-workflow@agdata-corp-workflows
```

## [1.0.0] - 2026-01-15

### Initial Release
- `/claude-workflow:setup` - Set up workflow system in a project
- `/claude-workflow:update` - Update existing installation
- `/claude-workflow:variants` - List available variants
- `/claude-workflow:status` - Check installation status
- Support for base and nextjs-development variants
- Automatic fetching from GitHub repository
