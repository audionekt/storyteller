---
description: Generate or update MDX component documentation for Storybook. Creates rich docs with usage examples, props tables, and guidelines.
user-invocable: true
disable-model-invocation: true
argument-hint: "<file-or-directory> [--all]"
---

# Storyteller Docs

Generate MDX documentation files for React components, designed for Storybook's Docs addon.

## Pre-flight Check

1. Check if `.storyteller/config.json` exists — if not, suggest `/storyteller:scan`
2. Read config for doc conventions (`doc_format`, `doc_suffix`)
3. Parse arguments:
   - **File path**: Generate docs for that component
   - **Directory**: Generate docs for all components in directory
   - **`--all`**: Generate docs for all components with managed stories

## Phase 1: Identify Targets

### Single File

```
/storyteller:docs src/components/Button.tsx
```

1. Verify component exists
2. Check if story file exists (docs reference stories)
3. Check if doc file already exists
   - If managed (`@storyteller-managed`): will regenerate
   - If custom: warn and skip

### Directory or All

1. Find all components in scope
2. Filter to those with existing story files (docs require stories)
3. Filter out those with existing custom (non-managed) docs
4. Report what will be generated, ask for confirmation if > 5

If a component has no stories:
```
⚠ {ComponentName} has no stories — generating docs requires stories first.
  Run /storyteller:generate {path} first, then /storyteller:docs
```

## Phase 2: Gather Context

For each component, read:
- Component source file (props, types, description comments)
- Story file (to reference stories in Canvas blocks)
- JSDoc/TSDoc comments on the component and props
- Related design system documentation if present
- README or docs in the component directory

## Phase 3: Generate Documentation

### Spawn Generator Agent

For each component (parallelize for multiple):

```
Task(subagent_type="storyteller-generator", run_in_background=true, prompt="
Generate MDX documentation for this component.

Component source:
{component file contents}

Story file:
{story file contents}

Story exports available: {list of exported story names}

Component description (from JSDoc/comments):
{extracted description or 'none found'}

Template reference:
{contents of ${CLAUDE_PLUGIN_ROOT}/templates/docs-mdx.md}

Generate complete MDX documentation including:
1. Component description — what it does and when to use it
2. Usage example — minimal code to get started
3. Canvas examples — reference existing stories with <Canvas of={...} />
4. Props section — using <Controls /> for automatic prop table
5. Guidelines — when to use, when not to use, accessibility
6. Keep the @storyteller-managed marker and CUSTOM DOCS preservation marker

Base the description on actual component behavior, not assumptions.
Derive accessibility notes from actual aria attributes, roles, and keyboard handling in the source.
")
```

## Phase 4: Write Documentation Files

For each generated doc:

1. **Determine output path**: Same directory as component, with `.mdx` suffix
   - `src/components/Button.tsx` → `src/components/Button.mdx`
   - Follow existing doc convention if detected

2. **Preserve custom sections**: If updating existing managed docs, preserve content below `CUSTOM DOCS` marker

3. **Write the file**

## Phase 5: Report

```
✓ Documentation generated

Created:
  ✓ src/components/Button.mdx
    Sections: Description, Usage, 4 Canvas examples, Props, Guidelines
  ✓ src/components/Card.mdx
    Sections: Description, Usage, 3 Canvas examples, Props, Guidelines

Skipped:
  — src/components/Header.mdx (exists, not managed)
  — src/components/Modal.tsx (no stories — run /storyteller:generate first)

Next steps:
  npx storybook dev         → Preview documentation
  /storyteller:audit        → Full quality report
```

## Edge Cases

- **No stories exist**: Cannot generate meaningful docs without stories to reference. Suggest `/storyteller:generate` first.
- **Component has JSDoc**: Extract and use JSDoc comments as the primary description.
- **Complex props**: For union types, generics, or callback props, generate clear type documentation.
- **Compound components**: Document the full compound API (e.g., `Menu`, `Menu.Item`, `Menu.Trigger`).
- **Storybook Docs addon not installed**: Warn that `@storybook/addon-docs` is needed.

## Doc Quality Checklist

The generator should ensure:
- [ ] Description explains WHAT the component does (not HOW it's implemented)
- [ ] Usage example is copy-pasteable and works
- [ ] Canvas blocks reference real story exports
- [ ] Accessibility section is based on actual component behavior
- [ ] Guidelines are practical, not generic
