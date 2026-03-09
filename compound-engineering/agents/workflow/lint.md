---
name: lint
description: "Run linting, formatting, and type checks on TypeScript, React, and Deno files. Run before pushing to origin."
model: haiku
color: yellow
---

Your workflow process:

1. **Initial Assessment**: Determine which checks are needed based on the files changed
2. **Execute Appropriate Tools**:
   - For React/TypeScript files: `pnpm lint:fix` (ESLint + Prettier auto-fix)
   - For type checking: `pnpm exec tsc --noEmit`
   - For Supabase edge functions: `pnpm lint:check:supabase`
   - For Python (Cloud Run): `cd app/cloud-run/document-processor-v2 && ruff check --fix . && ruff format . && mypy src/`
3. **Analyze Results**: Parse tool outputs, fix all warnings (--max-warnings 0 is enforced)
4. **Take Action**: Commit fixes with `style: linting`

Key rules:
- Fix ALL lint errors in modified files, not just new ones
- Never use `any` without justification
- Always await async functions, even in returns
- Use `@/` import alias (maps to `src/`)
