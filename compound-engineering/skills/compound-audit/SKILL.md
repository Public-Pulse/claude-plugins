---
name: compound-audit
description: "Audit docs/solutions/ to find learnings not yet promoted to .claude/rules/. Use to discover which solved problems should become permanent AI rules."
argument-hint: "[--verbose]"
color: orange
disable-model-invocation: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash(wc -l *)
  - Bash(ls *)
  - AskUserQuestion
---

# compound-audit Skill

Scan all solution docs and compare against existing rules to find promotion gaps.

---

## Pipeline

<critical_sequence name="rule-audit" enforce_order="strict">

<step number="1" required="true">
### Step 1: Inventory

**Gather all solution docs:**
1. Glob `docs/solutions/**/*.md` to find all solution files
2. For each file, read the YAML frontmatter to extract: `title`, `type`, `tags`, `date`
3. Skip files that are templates or empty

**Gather all existing rules:**
1. Glob `.claude/rules/*.md` to find all rule files
2. For each rule file, extract all imperative statements (lines containing MUST, MUST NOT, SHOULD)
3. Build a keyword index of rule topics (function names, patterns, identifiers mentioned)

**Report inventory:**
- Total solution docs found
- Total rule files found
- Total existing rules (imperative statements)
</step>

<step number="2" required="true" depends_on="1">
### Step 2: Gap Analysis

For each solution doc, determine if its key learnings are already covered by existing rules:

1. **Extract key identifiers** from the solution: function names, file paths, patterns, error messages
2. **Search rules** for those identifiers using the keyword index from Step 1
3. **Classify** the solution as:
   - **Covered** -- key learning already exists as a rule (show which rule file and line)
   - **Partially covered** -- related rule exists but doesn't capture this specific learning
   - **Not covered** -- no matching rule found
   - **Not promotable** -- too context-specific, one-time fix, or would fail P2 litmus test

For each "not covered" solution, assess promotion potential:
- **P2 check:** Would Claude repeat this mistake without a rule?
- **P3 check:** Can the learning be distilled into a concrete, verifiable statement?
- If both pass: mark as **promotable**
- If either fails: mark as **not promotable** with reason

If `$ARGUMENTS` contains `--verbose`: show the reasoning for each classification.
</step>

<step number="3" required="true" depends_on="2">
### Step 3: Report

Present a summary table:

```
## Audit Results

### Summary
- Solutions scanned: N
- Already covered: N
- Partially covered: N
- Promotable (action needed): N
- Not promotable: N

### Promotable Solutions (run /compound-rules on these)

| Solution | Key Learning | Suggested Target |
|----------|-------------|-----------------|
| path/to/solution.md | Brief description | rules/target.md |
| ... | ... | ... |

### Partially Covered (review these)

| Solution | Existing Rule | Gap |
|----------|--------------|-----|
| ... | ... | ... |
```

After the table, suggest: "Run `/compound-engineering:compound-rules <path>` on any promotable solution to create the rule."

If there are more than 5 promotable solutions, ask: "Want me to process the top 3 highest-impact ones now using `/compound-rules`?"
</step>

</critical_sequence>

---

## Error Handling

- **No solutions directory:** "No `docs/solutions/` directory found. Nothing to audit."
- **No solution files:** "No .md files found in `docs/solutions/`. Nothing to audit."
- **No rule files:** "No `.claude/rules/*.md` files found. All solutions are unpromoted."

## Integration

**Invoked by:** `/compound-audit` or `/compound-audit --verbose`
**Routes to:** `/compound-rules` for individual promotions
