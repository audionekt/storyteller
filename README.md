<p align="center">
  <img src="./assets/logo.png" alt="storyteller" width="400" />
</p>

# Storyteller

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that automatically generates and maintains Storybook stories and component documentation from React source code.

## The Problem

Storybook stories fall behind quickly when you're rapidly iterating. You add a prop, rename a variant, refactor a component — and the stories quickly go stale. Writing stories by hand is tedious. Keeping them in sync can be painful..

## The Solution

Storyteller scans your React components, understands their props, types, and usage patterns, then generates rich Storybook stories and MDX documentation that actually mean something — not just boilerplate, but stories with realistic data, meaningful states (loading, error, empty, overflow), and proper documentation.

When your components change, storyteller keeps the stories in sync.

## Installation

Add the marketplace, then install the plugin:

```bash
# Add the marketplace
/plugin marketplace add audionekt/storyteller
```

Or test locally:

```bash
claude --plugin-dir /path/to/storyteller
```

## Quick Start

```bash
# See what you're working with
/storyteller:scan

# Generate stories for uncovered components
/storyteller:generate src/components/Button.tsx

# Update stories after component changes
/storyteller:sync

# Generate MDX docs
/storyteller:docs src/components/Button.tsx

# Full coverage report
/storyteller:audit
```

## Commands

| Command | What It Does |
|---------|-------------|
| `/storyteller:scan` | Discover React components and assess Storybook coverage |
| `/storyteller:generate` | Generate CSF3 stories for one or more components |
| `/storyteller:sync` | Detect component changes and update stories to match |
| `/storyteller:docs` | Generate or update MDX component documentation |
| `/storyteller:audit` | Full coverage, quality, and staleness report |
| `/storyteller:help` | Command reference |

## How It Works

### Scanning

Storyteller's scanner agent finds React components by looking for:
- Named/default function exports returning JSX
- `React.FC`, `React.forwardRef`, `React.memo` patterns
- TypeScript interfaces/types used as component props
- Existing `.stories.tsx` files to determine coverage

### Generation

The generator agent creates Storybook stories using CSF3 format with:
- A default story showing the component in its primary state
- Stories for each meaningful prop combination
- Interactive args for all public props
- Decorator setup (theme providers, routing, etc.) when detected
- Realistic mock data instead of placeholder values
- Edge case stories: empty, loading, error, overflow, disabled

### Syncing

When components change, storyteller can detect:
- Added/removed/renamed props
- Changed prop types or defaults
- New variants or states
- Removed stories for deleted components

Only `@storyteller-managed` stories are updated — hand-written stories are never touched.

### Documentation

MDX documentation includes:
- Component description and usage guidance
- Full props table with types, defaults, and descriptions
- Live code examples
- Design guidelines and accessibility notes

## Configuration

After first run, storyteller creates `.storyteller/config.json`:

```json
{
  "framework": "react",
  "storybook_version": "8",
  "story_format": "csf3",
  "doc_format": "mdx",
  "story_suffix": ".stories.tsx",
  "doc_suffix": ".mdx",
  "managed_marker": "@storyteller-managed",
  "include_patterns": ["src/components/**/*.tsx", "src/ui/**/*.tsx"],
  "exclude_patterns": ["**/*.test.*", "**/*.spec.*", "**/index.ts"]
}
```

## Currently Supported

- **Framework**: React (TypeScript + JavaScript)
- **Storybook**: v7 and v8 (CSF3 format)
- **Documentation**: MDX format

More frameworks planned for future releases.
