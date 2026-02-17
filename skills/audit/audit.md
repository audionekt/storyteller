---
description: Comprehensive audit of Storybook story coverage, quality, and staleness across the project. Produces an actionable report.
user-invocable: true
disable-model-invocation: true
argument-hint: "[directory]"
---

# Storyteller Audit

Run a comprehensive audit of Storybook story coverage, quality, and staleness. This is the full health check for your component documentation.

## Pre-flight Check

1. Check if `.storyteller/config.json` exists — if not, suggest `/storyteller:scan`
2. Read config for conventions
3. Parse arguments:
   - **Directory**: Audit that directory only
   - **No argument**: Audit entire project

## Phase 1: Scan (Component Discovery)

Run a fresh scan to get current state:

```
Task(subagent_type="storyteller-scanner", prompt="
Full project component scan.

Configuration:
{contents of .storyteller/config.json}

Find ALL React components and ALL story files.
For each component, report:
- Component name and file path
- Prop count and types
- Story file path (if exists)
- Whether story is managed (@storyteller-managed)
- File modification dates (component vs story)

Also detect:
- Storybook version and configuration
- Global decorators and parameters
- Story naming conventions
")
```

## Phase 2: Quality Review

For each component that HAS stories, run the reviewer:

### Parallel Review (batch of 3-5)

```
Task(subagent_type="storyteller-reviewer", run_in_background=true, prompt="
Review stories for: {ComponentName}

Component source:
{component file contents}

Story file:
{story file contents}

Check:
1. Accuracy — do stories match the current component API?
2. Coverage — are important states represented?
3. Quality — realistic data, proper typing, CSF3 format?
4. Staleness — are there prop mismatches suggesting drift?

Output a structured review with a quality score (1-10).
")
```

Collect all reviewer results.

## Phase 3: Compute Metrics

### Coverage Metrics

```
Coverage:
  Components total:     {N}
  With stories:         {N} ({%})
  With managed stories: {N} ({%})
  With docs (MDX):      {N} ({%})
  Without stories:      {N} ({%})
```

### Quality Metrics (from reviewer)

```
Quality:
  Average score:    {N}/10
  PASS:             {N} stories
  NEEDS_WORK:       {N} stories
  FAIL:             {N} stories
```

### Staleness Metrics

```
Staleness:
  Up to date:       {N}
  Possibly stale:   {N} (component modified after story)
  Confirmed stale:  {N} (prop mismatches detected)
```

## Phase 4: Generate Report

### Summary Dashboard

```
📊 Storyteller Audit Report
════════════════════════════

Coverage:  [████████░░░░░░░░] 52% (26/50 components)
Quality:   [██████████░░░░░░] 67% (avg 6.7/10)
Freshness: [████████████░░░░] 81% (21/26 up to date)

Overall health: FAIR
```

### Coverage Breakdown

```
── Coverage by Directory ──────────────────────────

src/components/         ████████████░░░░  75% (15/20)
src/ui/                 ████░░░░░░░░░░░░  25% (3/12)
src/features/           ██████████░░░░░░  60% (6/10)
src/layouts/            ░░░░░░░░░░░░░░░░   0% (0/8)
```

### Priority List

```
── High Priority (generate stories) ──────────────

These components have the most props and no stories:

1. src/components/DataTable.tsx (24 props) — complex, high-value
2. src/components/Form.tsx (18 props) — user-facing, many states
3. src/ui/Modal.tsx (15 props) — used across the app
4. src/components/Chart.tsx (12 props) — visual, needs stories

Quick fix:
  /storyteller:generate src/components/DataTable.tsx
  /storyteller:generate --all
```

### Stale Stories

```
── Stale Stories (sync needed) ────────────────────

1. src/components/Button.stories.tsx
   Component changed: 3 days ago
   Story last updated: 2 weeks ago
   Changes: +2 props (loading, iconPosition), variant type narrowed

2. src/components/Input.stories.tsx
   Component changed: 1 day ago
   Story last updated: 1 week ago
   Changes: +1 prop (errorMessage), removed deprecated 'error' prop

Quick fix:
  /storyteller:sync
```

### Quality Issues

```
── Quality Issues ─────────────────────────────────

FAIL:
  ✗ src/components/Nav.stories.tsx (score: 3/10)
    - Missing required props in Default story
    - Placeholder data ("test", "lorem ipsum")
    - No edge case stories

NEEDS_WORK:
  ⚠ src/ui/Card.stories.tsx (score: 5/10)
    - Missing loading state story
    - Args don't cover all variants
```

### Documentation Coverage

```
── Documentation ──────────────────────────────────

Components with MDX docs:  12/50 (24%)
Managed docs:               8/12

Missing docs for key components:
  - DataTable, Form, Modal, Chart

Quick fix:
  /storyteller:docs --all
```

## Phase 5: Recommendations

```
── Recommendations ────────────────────────────────

1. Generate stories for 24 uncovered components
   /storyteller:generate --all

2. Sync 5 stale managed stories
   /storyteller:sync

3. Fix 2 failing stories (quality < 4)
   /storyteller:generate --force src/components/Nav.tsx

4. Add MDX docs for 38 components
   /storyteller:docs --all

Estimated effort: ~15 minutes with storyteller
```

## Phase 6: Save Report

Write audit results to `.storyteller/last-audit.json`:
```json
{
  "timestamp": "{ISO timestamp}",
  "coverage": {
    "total": 50,
    "with_stories": 26,
    "managed": 20,
    "with_docs": 12,
    "percentage": 52
  },
  "quality": {
    "average_score": 6.7,
    "pass": 18,
    "needs_work": 6,
    "fail": 2
  },
  "staleness": {
    "up_to_date": 21,
    "possibly_stale": 3,
    "confirmed_stale": 2
  }
}
```

## Edge Cases

- **No stories at all**: Focus report on coverage, skip quality section
- **All stories hand-written**: Report coverage but note no managed stories to sync
- **Very large project (100+ components)**: Batch reviewer agents, show progress
- **No Storybook installed**: Note this prominently, still report component inventory
