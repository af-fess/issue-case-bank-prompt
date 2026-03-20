# Generate ISSUE_CASES.md from Git History

Paste this entire prompt into your Claude Code session (or any AI coding assistant
that supports long context / system prompts).

---

## TASK

Mine this repository's full git history across all branches and generate
`ISSUE_CASES.md` — an engineering issue case bank with a hot zones map
and two-axis classification (Component × Bug Class).

---

## Step 1 — Mine the git history

Search all commits for bugs, crashes, logic errors, architectural issues,
security flaws, and hotfixes using these keywords:

```
fix, bug, crash, issue, error, fail, wrong, broken, incorrect, hotfix, patch,
revert, regression, workaround, overflow, leak, null, cast, undefined, race,
deadlock, memory, corrupt, invalid, mismatch, vulnerability, security
```

For each matching commit collect:
- Commit hash + branch (if identifiable)
- Short description of the issue
- Component / file affected
- What the fix was
- Severity (see definitions below)
- Bug class (see taxonomy below)

---

## Step 2 — Build the Hot Zones Map

Before writing individual cases, analyze the full set of issues and produce a
**hot zones map**: a ranked table of components by issue count, with the
dominant bug class per component shown visually.

Format:

```markdown
## Hot Zones Map

| Component | Issues | Dominant Bug Classes | Cases |
|-----------|--------|----------------------|-------|
| `FileName.ext` / layer name | ████████ 8 | logic-error × 4, concurrency × 3 | 001, 003, 007... |
| `AnotherFile.ext` / layer name | █████ 5 | null-safety × 2, serialization × 2 | 002, 005... |
```

Bar width: 1 block (█) per issue, max 10 blocks.

Add a one-sentence reading note below the table: what the distribution reveals
about where the codebase is historically fragile and what kind of mistakes dominate.

---

## Step 3 — Write Individual Cases

Write one case per confirmed issue:

```markdown
## Case NNN — [Short Name]

**Component:** `file/path` or layer name
**Bug class:** [see taxonomy]
**Severity:** CRITICAL / HIGH / MEDIUM / LOW / BLOCKER
**Commit:** `hash`

### What Happened
[1–3 sentences: what the bug was and where it lived]

### Observable Symptom
[How it manifested: crash, silent wrong output, build failure, test flake, etc.]

### Root Cause
[The technical reason it happened]

### Fix Applied
[What was changed]

### Takeaway
[One opinionated rule, specific to this codebase, that prevents this class of bug.
Read like advice from a senior engineer who lived through it — not generic best practice.]
```

The **Takeaway** is the most important field.

---

## Step 4 — Assemble the Full File

Create `ISSUE_CASES.md` in this order:

1. Frontmatter (name, description, type: reference)
2. Case Index table (number, name, component, bug class, severity, commit)
3. Usage note (how to use the file)
4. Hot Zones Map (from Step 2)
5. Bug Class Reference table
6. Individual cases, ordered by severity then case number

---

## Bug Class Reference

| Class | What it covers |
|-------|---------------|
| `concurrency` | Race conditions, thread-unsafe shared state, unsynchronized globals |
| `null-safety` | Null/nil dereferences, missing guards at API or system boundaries |
| `type-system` | Integer overflow, wrong type assumptions, platform size differences |
| `logic-error` | Wrong conditions, off-by-one, parameter confusion, silent wrong output |
| `memory-safety` | Use-after-free, dangling pointers, buffer overread, lifetime violations |
| `serialization` | Encoding/decoding errors, binary vs string confusion, format mismatch |
| `api-contract` | Violated preconditions, unexpected input, undocumented assumptions |
| `state-management` | Singleton misuse, mutable shared state, lifecycle ordering bugs |
| `security-gap` | Detection disabled, validation bypassed, insecure default configuration |
| `build-pipeline` | Circular task dependencies, hardcoded paths, missing task ordering |
| `retain-cycle` | (iOS/Swift/ObjC) Circular strong references causing memory leaks |
| `main-thread` | (iOS/macOS) UI framework called off main thread, or main thread blocked |

Add project-specific classes if needed.

---

## Severity Definitions

| Severity | Meaning |
|----------|---------|
| CRITICAL | Data corruption, security bypass, crash in production hot path |
| HIGH | Logic error producing wrong output, signing/validation incorrectness |
| MEDIUM | Crash on edge-case input, silent feature disabled, flaky CI |
| LOW | Maintenance, cleanup, non-functional |
| BLOCKER | Build or pipeline could not complete |

---

## Keeping the Bank Current

After every non-obvious bug fix, add a new case while the context is fresh.
The bank compounds in value over time — new engineers and AI assistants both benefit.
