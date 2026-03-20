# Generate ISSUE_CASES.md from Git History

Paste this entire prompt into your Claude Code session (or any AI coding assistant
that supports long context / system prompts).

---

## TASK

Mine this repository's full git history across all branches and generate
`.claude/skills/issue-cases/ISSUE_CASES.md` — an engineering issue case bank
with a hot zones map and two-axis classification (Component × Bug Class).

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

This is the first thing a reader sees — it shows which components carry
the most historical risk at a glance.

Also add a one-sentence reading note below the table: what the distribution
reveals about where the codebase is historically fragile and what kind of
mistakes dominate.

---

## Step 3 — Write Individual Cases

Write one case per confirmed issue using this structure:

```markdown
## Case NNN — [Short Name]

**Component:** `file/path` or layer name
**Bug class:** [see taxonomy]
**Severity:** CRITICAL / HIGH / MEDIUM / LOW / BLOCKER
**Ticket:** [TICKET-XXXXX or —]
**Commit:** `hash`
**Branch:** [branch name or —]

### What Happened
[1–3 sentences: what the bug was and where it lived]

### Observable Symptom
[How it manifested: crash, silent wrong output, build failure, test flake, etc.]

### Root Cause
[The technical reason it happened]

### Fix Applied
[What was changed]

### Takeaway
[The rule or pattern that prevents this class of bug in future.
Make this SPECIFIC to this codebase — not generic advice.
One opinionated sentence a new engineer or AI assistant can act on immediately.]
```

The **Takeaway** is the most important field. It should read like advice
from a senior engineer who lived through the bug — not a generic best practice.

---

## Step 4 — Assemble the Full File

Create `.claude/skills/issue-cases/ISSUE_CASES.md` in this order:

```markdown
---
name: issue-cases
description: Historical engineering issue bank — real bugs, crashes, and logic
  errors mined from the [PROJECT NAME] git history. Includes a hot zones map
  and two-axis (Component × Bug Class) classification. Read before modifying
  historically fragile components.
type: reference
---

# Issue Case Bank — [PROJECT NAME] Engineering History

[1-sentence summary of what this file is and where the data came from]

## Case Index

| # | Name | Component | Bug Class | Severity | Commit | Ticket |
|---|------|-----------|-----------|----------|--------|--------|
| [001](#...) | ... | ... | `class` | SEVERITY | `hash` | — |

---

**How to use this file:**
- Check the Hot Zones Map first — it shows which components carry the most risk.
- When modifying a component, look up its cases by component name in the index.
- When writing new async/threading/null-handling code, look up cases by bug class.
- Apply each case's Takeaway — it distills the anti-pattern into an actionable rule.

---

## Hot Zones Map

[generated in Step 2]

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

[Individual cases follow, ordered by severity then case number]
```

---

## Step 5 — Update CLAUDE.md (or equivalent)

Add a **"Before Making Code Changes"** section. Use active language —
not "consult," but "incorporate":

```markdown
## Before Making Code Changes

When implementing any feature or technical design that touches the components below:
1. Read the relevant cases from `.claude/skills/issue-cases/ISSUE_CASES.md`
2. Explicitly incorporate each case's **Takeaway** rule into your implementation approach
3. Call out which past issues are relevant and how the new code avoids repeating them

Do this **before writing any code** — not as a post-review step.

| Component | Relevant cases |
|-----------|---------------|
| [component from hot zones] | 001, 002, ... |
```

Map every component from the Hot Zones Map to its case numbers.

---

## Step 6 — Add the Pre-Edit Hook

Create `.claude/hooks/hot-zone-check.sh` — a shell script that fires
automatically before any Edit/Write tool call, checks whether the target file
is a known hot zone, and outputs the relevant cases to the AI as context.

```bash
#!/bin/bash
# Hot Zone Check — fires before Edit/Write tool calls.
# Reads the file path from stdin (JSON tool input), checks if it matches
# a known hot-zone component, and prints the relevant ISSUE_CASES.md cases.

FILE_PATH=$(cat | python3 -c "
import sys, json
try:
    d = json.load(sys.stdin)
    print(d.get('file_path', d.get('path', '')))
except:
    print('')
" 2>/dev/null)

if [ -z "$FILE_PATH" ]; then
  exit 0
fi

MSG=""

# Add one elif block per hot-zone component from the Hot Zones Map.
# Use grep -qE with a regex matching the file or directory pattern.

if echo "$FILE_PATH" | grep -qE "YourHotZoneFile1\.(ext)"; then
  MSG="HOT ZONE — ComponentName: read ISSUE_CASES.md cases 001, 002, 003 and apply their Takeaway rules before writing code."

elif echo "$FILE_PATH" | grep -qE "YourHotZoneDir2/"; then
  MSG="HOT ZONE — ComponentName2: read ISSUE_CASES.md cases 004, 005 and apply their Takeaway rules before writing code."

# ... add a block for each row in the Hot Zones Map
fi

if [ -n "$MSG" ]; then
  echo "$MSG"
fi

exit 0
```

Make it executable:
```bash
chmod +x .claude/hooks/hot-zone-check.sh
```

Register it in `.claude/settings.local.json` (create if it doesn't exist;
merge into existing JSON if it does — do not overwrite other keys):

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "bash /absolute/path/to/your/project/.claude/hooks/hot-zone-check.sh"
          }
        ]
      }
    ]
  }
}
```

Use the **absolute path** to the hook script.

---

## Step 7 — Update Any Persona Skills (Optional)

If the project has AI persona skills (e.g. a senior engineer persona used in
PRD or design sessions), add a reference to the issue bank in their
"Read First" section. The persona should consult the case bank before
classifying any new signal or feature as "ready to implement."

---

## Keeping the Bank Current

The case bank compounds in value over time. After every non-obvious bug fix:

1. Add a new case while the context is fresh
2. Write the Takeaway while the engineer who fixed it can still articulate why
3. Update the Hot Zones Map bar and case list for the affected component
4. Update the `CLAUDE.md` lookup table if it's a new component
5. Add a new `grep -qE` block to the hook if it's a new hot-zone file

The goal: a new IC joining the team six months from now gets the distilled
judgment of everyone who broke the codebase before them — and so does the
AI working alongside them.

---

## Severity Definitions

| Severity | Meaning |
|----------|---------|
| CRITICAL | Data corruption, security bypass, crash in production hot path |
| HIGH | Logic error producing wrong output, signing/validation incorrectness |
| MEDIUM | Crash on edge-case input, silent feature disabled, flaky CI |
| LOW | Maintenance, cleanup, non-functional |
| BLOCKER | Build or pipeline could not complete |
