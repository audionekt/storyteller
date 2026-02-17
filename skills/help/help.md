---
description: Show available Storyteller commands, workflow, and current configuration
user-invocable: true
disable-model-invocation: true
---

# Storyteller Help

Display the Storyteller command reference, typical workflow, and current configuration.

## Commands

| Command | Purpose |
|---------|---------|
| `/storyteller:scan` | Discover React components and assess Storybook story coverage |
| `/storyteller:generate` | Generate CSF3 stories for one or more components |
| `/storyteller:sync` | Detect component API changes and update managed stories to match |
| `/storyteller:docs` | Generate or update MDX component documentation |
| `/storyteller:audit` | Full coverage, quality, and staleness report |
| `/storyteller:help` | Show this help message |

## Typical Workflow

```
/storyteller:scan       → See what components exist and what's missing stories
    ↓
/storyteller:generate   → Generate stories for uncovered components
    ↓
/storyteller:sync       → After component changes, sync stories to match
    ↓
/storyteller:docs       → Generate MDX documentation alongside stories
    ↓
/storyteller:audit      → Full quality and coverage report
```

## Quick Examples

```
/storyteller:scan                              → Scan entire project
/storyteller:generate src/components/Button.tsx → Generate stories for Button
/storyteller:generate src/components/           → Generate stories for all components in directory
/storyteller:sync                              → Sync all managed stories
/storyteller:docs src/components/Button.tsx     → Generate MDX docs for Button
/storyteller:audit                             → Full audit report
```

## Configuration

Storyteller stores config in `.storyteller/config.json`:

| Setting | Default | Description |
|---------|---------|-------------|
| `framework` | `react` | Component framework |
| `storybook_version` | `8` | Storybook version (7 or 8) |
| `story_format` | `csf3` | Story format |
| `doc_format` | `mdx` | Documentation format |
| `story_suffix` | `.stories.tsx` | Story file suffix |
| `doc_suffix` | `.mdx` | Documentation file suffix |
| `managed_marker` | `@storyteller-managed` | Marker for auto-generated stories |
| `include_patterns` | `["src/components/**/*.tsx"]` | Glob patterns for component discovery |
| `exclude_patterns` | `["**/*.test.*", ...]` | Glob patterns to skip |

## Key Concepts

### Managed vs Custom Stories

- **Managed stories** have `// @storyteller-managed` as their first line
- Storyteller only creates and updates managed story files
- Hand-written stories are **never** modified
- Within a managed file, everything below `// ---- CUSTOM STORIES` is preserved

### How Scanning Works

The scanner agent finds React components by detecting:
- Exported functions/consts returning JSX
- `React.FC`, `React.forwardRef`, `React.memo` patterns
- TypeScript prop interfaces and types

### What Gets Generated

Stories include:
- Default state with realistic props
- One story per variant (if component has variants)
- Edge case stories: empty, loading, error, disabled, overflow
- Proper decorators for detected providers
- Interactive args for all public props

## Instructions

1. Check if `.storyteller/config.json` exists
2. If it exists, display current configuration
3. Always display the command reference above
4. Suggest the most appropriate next command:
   - No config → suggest `/storyteller:scan` to get started
   - Config exists, no stories → suggest `/storyteller:generate`
   - Stories exist → suggest `/storyteller:sync` or `/storyteller:audit`
