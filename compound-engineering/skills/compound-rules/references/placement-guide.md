# Placement Guide

Domain-to-file mapping for `.claude/rules/*.md`. Used by the compound-rules skill to determine where a rule should go.

**Important:** The skill always verifies against actual files on disk via Glob. This guide is supplementary — if a new rule file exists on disk, use its `paths:` frontmatter as the source of truth.

## Domain Mapping

| Domain | Target File | Path Scope |
|--------|-------------|------------|
| Edge functions, Deno, API handlers | `backend.md` | `app/supabase/functions/**/*.ts` |
| Citation handling, draft citations | `citations.md` | `app/src/components/correspondence/**`, `app/src/lib/citation-utils.ts` |
| Linting, async/await, engineering principles | `code-quality.md` | *(no paths = always loaded)* |
| SQL migrations, RLS, pgTAP, backfills | `database.md` | `app/supabase/migrations/**`, `app/supabase/tests/**/*.sql` |
| shadcn-ui, Figma, design tokens, Storybook | `design-system.md` | `app/src/components/ui/**`, `app/src/**/*.stories.tsx` |
| React, TanStack Query, hooks, components | `frontend.md` | `app/src/**/*.{ts,tsx}` |
| Python, document processor | `python.md` | `app/cloud-run/document-processor-v2/**/*.py` |
| Test patterns, factories, coverage | `testing.md` | `app/src/**/*.test.*`, `app/supabase/functions/**/*.test.ts` |

## Placement Principles

### Consumer-Based (P4)

Rules go where they are **consumed**, not where the related code is produced.

Examples:
- TanStack Query mutation pattern -> `frontend.md` (consumed when writing React code)
- Supabase RLS policy -> `database.md` (consumed when writing migrations)
- Edge function auth check -> `backend.md` (consumed when writing Deno handlers)

### No Match?

- If the rule applies to all code regardless of domain -> `code-quality.md`
- If the rule is universal and applies across AI tools (not just Claude) -> `AGENTS.md` (manual edit)
- Do NOT create new rule files in v1. Inform the user to create one manually if needed.

### H2 Section Convention

Each rule file uses `## Section Name` to group related rules. When inserting:

1. Find the H2 section that best matches the rule's topic
2. Append the rule at the end of that section (before the next H2)
3. If no section matches, create a new H2 following the existing naming style
