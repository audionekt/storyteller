# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**storyteller** is a Claude Code plugin that automatically generates and maintains Storybook stories and component documentation from React source code. It eliminates the tedious work of keeping stories and docs in sync with component implementations.

## Commands

| Command | Purpose |
|---------|---------|
| `/storyteller:scan` | Discover React components and assess story coverage |
| `/storyteller:generate` | Generate Storybook stories for components |
| `/storyteller:sync` | Update existing stories to match current component APIs |
| `/storyteller:docs` | Generate or update MDX component documentation |
| `/storyteller:audit` | Report on story coverage, quality, and staleness |
| `/storyteller:help` | Show command reference |

## Workflow

```
/storyteller:scan       → Discover components, identify gaps
    ↓
/storyteller:generate   → Generate stories for uncovered components
    ↓
/storyteller:sync       → Keep stories in sync after component changes
    ↓
/storyteller:docs       → Generate MDX documentation
    ↓
/storyteller:audit      → Full coverage and quality report
```

## Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| `storyteller-scanner` | haiku | Fast component discovery and prop extraction |
| `storyteller-generator` | sonnet | Generate high-quality stories with meaningful states |
| `storyteller-reviewer` | sonnet | Verify story quality and component coverage |

## Configuration

Edit `.storyteller/config.json`:

- `framework`: `react` (only supported framework for now)
- `storybook_version`: `7` or `8` (CSF3 format)
- `story_format`: `csf3` (Component Story Format 3)
- `doc_format`: `mdx` (MDX documentation)
- `story_suffix`: `.stories.tsx`
- `doc_suffix`: `.mdx`
- `managed_marker`: `@storyteller-managed`
- `include_patterns`: Glob patterns for component discovery
- `exclude_patterns`: Glob patterns to skip

## Testing

```bash
claude --plugin-dir ./
```

Then try:
```
/storyteller:help
/storyteller:scan
```
