---
description: Generate Storybook CSF3 stories for React components. Creates meaningful stories with realistic data, edge cases, and proper documentation.
user-invocable: true
disable-model-invocation: true
argument-hint: "<file-or-directory> [--all] [--force]"
---

# Storyteller Generate

Generate high-quality Storybook stories for React components.

## Pre-flight Check

1. Check if `.storyteller/config.json` exists — if not, suggest `/storyteller:scan` first
2. Read config for story conventions (suffix, format, patterns)
3. Parse arguments:
   - **File path**: Generate for that specific component
   - **Directory path**: Generate for all components in directory
   - **`--all`**: Generate for all components missing stories (use last-scan.json)
   - **`--force`**: Overwrite existing managed stories

## Phase 1: Identify Targets

### Single File

```
/storyteller:generate src/components/Button.tsx
```

1. Verify the file exists and contains a React component
2. Check if a story file already exists
3. If story exists and not `--force`:
   - If managed (`@storyteller-managed`): suggest `/storyteller:sync` instead
   - If custom: warn and skip (never overwrite hand-written stories)

### Directory

```
/storyteller:generate src/components/
```

1. Use scanner to find all components in directory
2. Filter to those without stories (unless `--force`)
3. Report what will be generated and ask for confirmation

### All Missing

```
/storyteller:generate --all
```

1. Read `.storyteller/last-scan.json` (if doesn't exist, run scan first)
2. Collect all components without stories
3. Report count and ask for confirmation if more than 5

## Phase 2: Context Gathering

Before generating stories, gather project context:

### Detect Environment

Look for (in parallel):
- `.storybook/preview.tsx` — global decorators, parameters
- `.storybook/main.ts` — addons, framework config
- `tsconfig.json` — path aliases for imports
- Theme files — design tokens, theme objects
- Existing stories — conventions to follow

### For Each Component

Read the component file and extract:
- Component name and export type (named/default)
- Full props interface/type
- Default prop values
- Internal state patterns (loading, error)
- Context dependencies (providers needed)
- Children patterns (if accepts children)

## Phase 3: Generate Stories

### Single Component

Spawn the generator agent:

```
Task(subagent_type="storyteller-generator", prompt="
Generate Storybook stories for this component.

Component file:
{full component source code}

Component name: {name}
File path: {path}
Story file path: {computed story path}

Project context:
- Storybook version: {from config}
- Global decorators: {from .storybook/preview.tsx or 'none detected'}
- Path aliases: {from tsconfig.json or 'none'}
- Existing story conventions: {from detected patterns}

Template reference:
{contents of ${CLAUDE_PLUGIN_ROOT}/templates/story-csf3.md}

Generate the complete story file content.
")
```

### Multiple Components (Parallel)

For directories or `--all`, spawn generator agents in parallel (max 3 concurrent):

```
Task(subagent_type="storyteller-generator", run_in_background=true, prompt="
Generate stories for: {component_name}
{... same context as single component ...}
")
```

Wait for all agents, then collect results.

## Phase 4: Write Story Files

For each generated story:

1. **Determine output path**:
   - Colocated: `src/components/Button.stories.tsx` (same dir as component)
   - Follow existing convention if detected

2. **Check for conflicts**:
   - If file exists and is NOT managed → skip with warning
   - If file exists and IS managed → only write if `--force`
   - If file doesn't exist → write

3. **Write the file** with the generator's output

## Phase 5: Review (Optional)

If more than 3 stories were generated, run the reviewer:

```
Task(subagent_type="storyteller-reviewer", prompt="
Review these generated stories for quality and accuracy.

{For each generated story:}
Component: {name}
Component source: {source}
Generated story: {story content}

Check accuracy, coverage, and quality. Report any issues.
")
```

If reviewer finds issues, report them but still save the files (they can be synced later).

## Phase 6: Report

```
✓ Stories generated

Generated:
  ✓ src/components/Button.stories.tsx (7 stories)
  ✓ src/components/Card.stories.tsx (5 stories)
  ✓ src/components/Modal.stories.tsx (8 stories)

Skipped:
  — src/components/Header.stories.tsx (already exists, not managed)

{If reviewer ran:}
Quality review:
  Button: PASS
  Card: NEEDS_WORK — missing loading state story
  Modal: PASS

Total: {N} story files, {M} individual stories

Next steps:
  npx storybook dev                     → Preview in Storybook
  /storyteller:docs src/components/      → Generate documentation
  /storyteller:audit                     → Full quality audit
```

## Phase 7: Update State

Update `.storyteller/last-scan.json` with new coverage data.

## Edge Cases

- **Component with no props**: Generate minimal stories (Default, WithChildren if applicable)
- **Generic components** (`<T>`): Use a concrete type parameter in stories
- **Compound components** (Menu.Item, Tabs.Panel): Generate stories for the root, include sub-components
- **Higher-order components**: Generate stories for the wrapped result
- **Context-dependent components**: Detect required providers and add decorators
- **Components with side effects**: Add parameters to disable side effects in stories
- **Very large component files**: Focus on the exported component, skip internal helpers

## Error Handling

- **Generator agent fails**: Report error, suggest running for that component individually
- **File write fails**: Report permission error
- **Component can't be analyzed**: Skip with warning, suggest manual story creation
