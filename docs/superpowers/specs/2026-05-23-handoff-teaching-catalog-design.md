# Handoff Teaching Catalog — Design Spec

**Date:** 2026-05-23
**Skill:** handoff
**Change:** Add a misuse catalog with structured lessons triggered by mistakes

---

## Problem

The handoff skill is purely mechanical — it tells Claude how to delegate but never teaches the user when or why to use it, or what misuse looks like. Users can hand off trivial tasks, dump raw context, or skip deliverable specs with no feedback.

---

## Design

### Teaching model

- Teaching is **triggered by mistakes**, not always shown
- **Severity determines intervention timing:**
  - Hard misuses → pause before dispatching, structured lesson, override available
  - Soft misuses → proceed silently, structured debrief after execution

---

## Misuse catalog

### Hard patterns (pause before dispatching)

| # | Pattern | Detection signal |
|---|---------|-----------------|
| 1 | **Trivial task** | Task fits in one short sentence, no real work — a rename, lookup, one-liner |
| 2 | **Requires dialogue** | User wants to think through, explore, or discuss — not produce a deliverable |
| 3 | **Intermediate decisions needed** | Task has design forks where user must choose before work can continue |
| 4 | **Unresolvable ambiguity** | Too underscoped to write a meaningful briefing; would immediately generate questions.md |

### Soft patterns (proceed, debrief after)

| # | Pattern | Detection signal |
|---|---------|-----------------|
| 5 | **Bloated briefing** | Authored content exceeds ~150–200 words before file references |
| 6 | **Context dump** | Claude pulls raw conversation into the briefing rather than distilling |
| 7 | **Missing deliverables** | Can't concretely fill the Deliverables section — no format, path, or success criteria |

---

## Lesson format

Used for both hard warnings and soft debrief entries:

```
### [Pattern name]
**Detected:** [what specifically triggered this]
**Why this is a problem:** [1-2 sentences]
**Rather than:** [the problematic pattern, generalized]
**Instead:** [what to do]
```

Hard lessons append:
> *To proceed anyway, say "hand off anyway."*

Soft lessons are grouped under a **Handoff Quality** section appended after the execution summary. Framed as "for next time." Multiple soft issues each get their own entry.

---

## Workflow integration

### Step 0 (new): Pre-dispatch check

Before any triage, check the task against hard patterns 1–4. If one triggers:
1. Present the structured lesson
2. Stop — do not proceed
3. If user says "hand off anyway", proceed from Step 1 as normal

### Step 2 (modified): Build the briefing

During briefing writing, watch for soft patterns 5–7. No interruption — flag internally.

### Step 4 (modified): Handle the response

After presenting the execution summary, if any soft patterns were flagged in Step 2, append **Handoff Quality** debrief. If none flagged, nothing extra appears.

---

## What doesn't change

- The existing workflow steps 1–4 are otherwise unchanged
- The core mechanics (briefing format, sub-agent spawn, summary/questions/needs-scoping paths) stay the same
- The catalog and lesson format live in new sections added before the workflow

---

## Lesson examples

### Hard — Trivial task

> You asked me to hand off: "rename `getUserById` to `fetchUserById` in auth.js"
>
> ### Trivial task
> **Detected:** Single rename in one file — no real work for a sub-agent.
> **Why this is a problem:** Spawning a sub-agent has overhead: briefing, spawn, summary. For a task this small, that overhead exceeds the work itself.
> **Rather than:** Handing off any task that mentions delegation
> **Instead:** Reserve handoff for tasks with real scope — multiple steps, file analysis, research, or anything that would take more than a minute inline.
>
> *To proceed anyway, say "hand off anyway."*

### Soft — Bloated briefing

> **Handoff Quality**
>
> ### Bloated briefing
> **Detected:** Briefing reached ~240 words before file references.
> **Why this is a problem:** Large briefings signal a task that isn't well-scoped. The sub-agent gets noise alongside the signal, and the briefing is harder to revise or reuse.
> **Rather than:** Writing everything you know about the problem
> **Instead:** Distill to what the sub-agent can't infer from the files. 150 words is usually enough; if not, the task may need splitting.
