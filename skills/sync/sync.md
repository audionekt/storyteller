---
description: Detect component API changes and update managed Storybook stories to match. Only touches @storyteller-managed files — never modifies hand-written stories.
user-invocable: true
disable-model-invocation: true
argument-hint: "[file-or-directory]"
---

# Storyteller Sync

Detect changes in React component APIs and update managed stories to match. This is the core "keep stories in sync" command.

## Pre-flight Check

1. Check if `.storyteller/config.json` exists — if not, suggest `/storyteller:scan`
2. Read config for conventions
3. Parse arguments:
   - **File path**: Sync that specific component's story
   - **Directory**: Sync all managed stories in directory
   - **No argument**: Sync all managed stories in project

## Phase 1: Find Managed Stories

### If path specified:
- Find the story file for the specified component
- Verify it's managed (`@storyteller-managed` marker)

### If no path:
- Use scanner to find all `.stories.tsx` files
- Filter to managed files only (contain `@storyteller-managed`)

If no managed stories found:
```
No managed stories found.

Managed stories are created by /storyteller:generate and marked with @storyteller-managed.
Hand-written stories are never modified.

Run /storyteller:generate to create managed stories.
```

## Phase 2: Detect Changes

For each managed story, compare the component source against the story:

### Use Scanner Agent (Parallel)

```
Task(subagent_type="storyteller-scanner", prompt="
Compare this component's current API with its story file.

Component source:
{component file contents}

Current story file:
{story file contents}

Detect:
1. Props added to component but missing from stories
2. Props removed from component but still in stories
3. Prop types that changed (e.g., string → enum)
4. Prop defaults that changed
5. New variants or states not covered
6. Component renamed or restructured
7. Import path changes

Report each change with severity:
- BREAKING: Story will fail (removed prop used in args, type mismatch)
- ADDITION: New prop not covered (story works but incomplete)
- COSMETIC: Minor change (default value, description)
")
```

### Categorize Results

```
Changes detected:

BREAKING (stories will fail):
  ⚠ Button.tsx: prop 'variant' type changed from string to 'primary' | 'secondary' | 'ghost'
  ⚠ Card.tsx: prop 'image' renamed to 'imageUrl'

ADDITIONS (new props not covered):
  + Button.tsx: new prop 'loading: boolean'
  + Modal.tsx: new props 'onClose', 'closeOnOverlayClick'

COSMETIC (minor updates):
  ~ Input.tsx: default value for 'placeholder' changed

No changes:
  ✓ Header.stories.tsx — up to date
  ✓ Footer.stories.tsx — up to date
```

## Phase 3: Confirmation

If changes were detected, ask for confirmation:

```
{count} managed stories need updating.

Breaking changes: {count} (stories will fail without update)
Additions: {count} (new props to cover)
Cosmetic: {count} (minor updates)

Update all? [y/n/selective]
```

If "selective", let user choose which files to sync.

## Phase 4: Regenerate Stories

For each story that needs updating:

### Preserve Custom Stories

Before regenerating, extract the custom section:
1. Find the `// ---- CUSTOM STORIES` marker in the existing story file
2. Save everything below that marker
3. After regeneration, append the preserved custom section

### Spawn Generator Agents (Parallel)

```
Task(subagent_type="storyteller-generator", run_in_background=true, prompt="
Regenerate the managed portion of this story file.

Component source (CURRENT):
{current component file contents}

Previous story file:
{current story file contents}

Changes detected:
{list of changes from Phase 2}

Existing custom stories to preserve:
{custom section content, or 'none'}

Regenerate the managed stories section, incorporating:
1. Fix all breaking changes
2. Add stories for new props/variants
3. Update cosmetic changes
4. Keep the same story organization style
5. Preserve story names where possible (for stable URLs)

IMPORTANT: Everything below the CUSTOM STORIES marker must be preserved exactly.
")
```

## Phase 5: Write Updated Stories

For each regenerated story:

1. Read the existing file one more time (safety check)
2. Verify it still has `@storyteller-managed` marker
3. Write the updated content
4. Verify the custom section was preserved

## Phase 6: Report

```
✓ Stories synced

Updated:
  ✓ src/components/Button.stories.tsx
    - Fixed: variant type constraint
    - Added: Loading story
    - Updated: 2 existing stories

  ✓ src/components/Card.stories.tsx
    - Fixed: imageUrl prop rename
    - Preserved: 3 custom stories

Skipped:
  — src/components/Nav.stories.tsx (not managed)

Already up to date:
  ✓ src/components/Header.stories.tsx
  ✓ src/components/Footer.stories.tsx

Next steps:
  npx storybook dev         → Preview updated stories
  /storyteller:audit        → Verify quality
```

## Phase 7: Update State

Update `.storyteller/last-scan.json` with refreshed coverage data.

## Edge Cases

- **Component deleted**: Warn but don't delete the story file (user may want to keep it)
- **Component moved**: Detect if a similarly named component exists elsewhere, suggest updating import path
- **Custom section corrupted**: If marker is missing but file is managed, warn and regenerate carefully
- **Merge conflicts in stories**: If story file has conflict markers, report and skip
- **Story file has syntax errors**: Report but still attempt sync

## Safety Guarantees

1. **Never touch non-managed files** — if `@storyteller-managed` marker is missing, skip
2. **Always preserve custom section** — content below the marker is sacred
3. **Backup on conflict** — if something looks wrong, warn before writing
4. **Dry run friendly** — the change detection (Phase 2) report shows exactly what will change
