# Issue Case Bank — Give Your AI Institutional Memory

AI coding assistants are good at general knowledge. They're blind to your project's specific history.

They don't know about the race condition your team fixed two years ago. Or the security validation bug that looked correct but wasn't. Or the three build pipeline failures you hit in a row when refactoring task ordering.

That knowledge lives in git history — and it's invisible to any AI by default.

This repo contains a **reusable prompt** that mines your git history, classifies past bugs and crashes, and produces a structured **Issue Case Bank** your AI can consult before writing code.

---

## The Idea

Most approaches to this problem are automated: feed git history into the LLM, let it figure out what's relevant.

This approach is different: **human-curated case banks.**

For every significant bug in your history, you write a case:
- What happened
- What the symptom was
- What the root cause was
- A **Takeaway** — one opinionated rule, specific to your codebase, that prevents this class of mistake from recurring

Then you classify each case two ways:
- **By component** — which files carry the most historical fragility
- **By bug class** — concurrency, null-safety, logic-error, security-gap, build-pipeline, etc.

The result is a **Hot Zones Map**: at a glance, you see which components have the most past issues and what kind of bugs dominate each one.

The difference from automated extraction: **a human wrote the Takeaway.** The AI didn't distill a rule from a commit diff — an engineer wrote it, with judgment, in context. That's what makes it actionable rather than just historical.

---

## What Gets Generated

### Hot Zones Map
A ranked table showing which components carry the most historical risk:


| Component            | Issues     | Dominant Bug Classes                               | Cases      |
|----------------------|------------|----------------------------------------------------|------------|
| Core Engine          | ████████ 8 | security-gap × 3, logic-error × 3, concurrency × 2 | 001–008   |
| Build Pipeline       | █████    5 | build-pipeline × 4, security-gap × 1              | 009–013    |
| Networking Layer     | ███      3 | null-safety × 2, serialization × 1                | 014–016    |


### Individual Cases
Each case follows a consistent structure:

```
## Case 001 — Race Condition in Event Dispatcher

**Component:** `src/EventDispatcher.swift`
**Bug class:** `concurrency`
**Severity:** CRITICAL

### What Happened
The singleton event dispatcher used a class-level mutable dictionary as
a scratchpad across concurrent calls. Under load, writes from one call
overwrote in-progress state from another.

### Observable Symptom
Intermittent dropped events under high attribution load.
Non-deterministic — only surfaced in production.

### Root Cause
Per-call state stored at class scope in a singleton with no synchronization.

### Fix Applied
Moved scratchpad to a local variable inside the dispatch method.
Each call now owns its own state.

### Takeaway
Any new state added at class scope in a singleton is a potential race
condition. Per-call computation belongs in local variables. Ask before
adding class properties: "Is this truly shared state, or per-call state?"
```

---

## How to Use

### Step 1 — Run the prompt
Open your project in Claude Code (or any AI coding assistant that supports context files).
Paste the full contents of [`GENERATE_ISSUE_CASES_PROMPT.md`](./GENERATE_ISSUE_CASES_PROMPT.md) into the session.

The AI will mine your git history, build the Hot Zones Map, and write the full case bank with Takeaway rules.

### Step 2 — Review and refine
The AI does the mining. You refine the Takeaways. The more specific and opinionated the Takeaway, the more useful it is.

### Step 3 — Keep it growing
Every time a non-obvious bug is fixed, add a case while the context is fresh. The bank compounds in value over time — new engineers and AI assistants both benefit.

---

## Why Not Just Automate It?

| Automated extraction | Human-curated case bank |
|----------------------|------------------------|
| AI reads raw diffs | Human distills the lesson |
| General pattern recognition | Codebase-specific Takeaway |
| Relevant at query time | Always available, pre-loaded |
| Grows with no effort | Grows with deliberate judgment |

The Takeaway is the unit of value. A raw commit diff tells the AI *what changed*. A Takeaway tells it *what rule to apply next time* — in this codebase, for this component, based on what actually went wrong.

---

## Compatibility

Works with any AI coding assistant that supports context file injection:
CLAUDE.md, AGENTS.md, .cursorrules, Copilot instructions, or equivalent.

---

## Origin

This approach emerged from an AI-assisted PRD session. A team member asked: *"Why should I help the AI do research if the AI isn't qualified enough?"* The answer: the AI lacked project-specific context. That question led to mining git history as a source of institutional memory — not automated, but curated.

Full writeup: [LinkedIn post](#) ← add link when published

---

## License

MIT — use freely, adapt for your stack.
