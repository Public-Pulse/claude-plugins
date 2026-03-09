---
name: compound-rules
description: "Promote institutional learnings from docs/solutions/ into .claude/rules/ files. Use when a solved problem should become a permanent AI rule."
argument-hint: "[solution-file-path] [--auto] [--dry-run]"
disable-model-invocation: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Edit
  - AskUserQuestion
  - Bash(ls *)
  - Bash(wc -l *)
preconditions:
  - A solved problem exists (in docs/solutions/ or conversation context)
---

# compound-rules Skill

Promote solved problems into `.claude/rules/*.md` files following Anthropic's official best practices.

## Anthropic Principles (Active Validation)

These are engineering constraints checked before every proposal:

- **P2 Litmus Test** -- "Would removing this cause Claude to make mistakes?" If Claude could infer it from reading code, skip.
- **P3 Concrete & Verifiable** -- Rules must be actionable: "MUST use `maybeSingle()` for optional results" not "Handle results properly."
- **P4 Consumer-Based Placement** -- Rules go where they are **consumed**, not produced. A TanStack Query pattern goes in `frontend.md`, not `backend.md`.
- **P5 Single Source of Authority** -- One file owns each topic. Grep for duplicates before adding.
- **P6 Under 200 Lines** -- Per file. Longer files degrade adherence. Warn when approaching.
- **P7 Emphasis Markers** -- MUST/MUST NOT for critical rules. SHOULD for recommendations.
- **P10 Enforceable vs Advisory** -- MUST-level rules should have enforcement (linter, hook, CI). Flag gaps.

Sources: [Claude Code memory docs](https://code.claude.com/docs/en/memory), [Claude Code best practices](https://code.claude.com/docs/en/best-practices), [How Anthropic teams use Claude Code](https://www-cdn.anthropic.com/58284b19e702b49db9302d5b6f135ad8871e7658.pdf)

---

## Pipeline

<critical_sequence name="rule-promotion" enforce_order="strict">

<step number="1" required="true">
### Step 1: Extract Rule Candidate

**Parse input:**
- If `$ARGUMENTS` contains a `.md` file path: read that file
- If no file path: use conversation context (e.g., after `/workflows:compound`)
- If `$ARGUMENTS` contains `--auto`: skip confirmation in Step 4
- If `$ARGUMENTS` contains `--dry-run`: output proposal only, no writes

**From the learning, extract:**
- **Symptom:** what went wrong
- **Root cause:** why it happened
- **Rule:** what to do differently (imperative voice)

A single source may yield multiple rules -- process each independently.

**Validate each candidate:**
1. **P2 check:** Would Claude make this mistake without the rule? If no, skip it.
2. **P3 check:** Is the rule specific enough to verify? If vague, rephrase or skip.
3. **P10 check:** If MUST-level, does enforcement exist (hook, linter, CI)? If not, flag the gap.

If a candidate fails P2 or P3, inform the user:
- Offer to rephrase, skip, or suggest a hook (P8) if the rule is absolute and deterministic.
</step>

<step number="2" required="true" depends_on="1">
### Step 2: Determine Placement

**Dynamic discovery (not a static table):**

1. Glob `.claude/rules/*.md` to find all rule files
2. Read each file's YAML `paths:` frontmatter to understand scope
3. Match the learning's domain to the file whose paths cover where the rule is **consumed** (P4)
4. Consult [placement-guide.md](./references/placement-guide.md) for domain-to-file mapping as guidance
5. If no file matches: default to `code-quality.md` (no path scope = always loaded)
6. If the rule is universal and cross-tool: inform "This belongs in AGENTS.md Critical Rules. Add manually."

**Insertion point:** Find the H2 section in the target file that best matches the rule's topic. If none fits, create a new H2 section matching the existing naming style.
</step>

<step number="3" required="true" depends_on="2">
### Step 3: Validate

1. **Duplicate check (P5):** Grep target file for key identifiers (function names, patterns).
   - Identical: skip, show existing location
   - Conflicting: flag, propose amendment
   - Similar: show both, ask user
   - Wrong file: propose move (P4)

2. **File length (P6):** Run `wc -l` on target file.
   - Over 200 lines: warn
   - Adding would push over 200: suggest splitting

3. **Consistency (P7):** Verify MUST/SHOULD levels don't contradict across files.

4. **Authority (P5):** Confirm target file is the single authority for this topic.
</step>

<step number="4" required="true" depends_on="3">
### Step 4: Propose and Apply

**Show the user:**

```
Proposed rule for `.claude/rules/<target>.md` under ## <Section>:

<exact markdown to insert>

Rationale: <which principles justify this placement>
Warnings: <file length, near-duplicates, missing enforcement>
```

**Then ask:** "Apply this? (or tell me what to change)"

- **Confirmed:** apply using Edit tool
- **Changes requested:** adjust and re-validate
- **Skipped:** no changes

**Flag behavior:**
- `--auto`: apply without confirmation if all validations pass
- `--dry-run`: output proposal, do not write

**After applying**, report: file path, line count before/after, principle citations.
</step>

</critical_sequence>

---

## Error Handling

- **File not found:** "File not found: `<path>`. Provide a valid .md path or describe the learning."
- **Not markdown:** "Expected .md file, got `<ext>`. Provide a markdown path."
- **No extractable rule:** "No actionable rule found. The learning may be too context-specific for a rule."
- **AGENTS.md target:** "This rule is universal. Add to AGENTS.md Critical Rules manually."

## Integration

**Invoked by:** `/compound-rules <path>` or suggested after `/workflows:compound`
**Invokes:** nothing (terminal skill)
