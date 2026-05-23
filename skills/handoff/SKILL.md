---
name: handoff
description: >
  Delegate a task to a sub-agent to preserve the main context window. Detects misuse and teaches: hard misuses (trivial tasks, dialogue-only tasks, unresolvable ambiguity, intermediate decisions needed) pause with a structured lesson before dispatching; soft misuses (bloated briefing, context dumping, missing deliverables) proceed and surface a Handoff Quality debrief after execution. Use this skill whenever the user asks to hand off, delegate, offload, farm out, or spin off a task, or when they say things like do this in the background, don't clutter the context, use a sub-agent for this, or handle this separately. Also trigger when the user implies context preservation, such as we're getting long can you do X on the side or run this without bloating our conversation. This skill is general-purpose and works for code, writing, research, data analysis, SQL, file processing, or any self-contained task. Even if the user doesn't use the word handoff, trigger whenever they want work done outside the main conversation thread to keep it clean.
---

# Handoff Skill

Delegate discrete tasks to a sub-agent while keeping the main context window lean. The main thread's only job is to write a concise work order and read back a concise receipt. All heavy lifting happens in the sub-agent's isolated workspace.

## Core principle

The doorway in (briefing) and the doorway out (summary) are always narrow. The sub-agent can read files, write code, run scripts — whatever the task requires internally. But what crosses the boundary between main thread and sub-agent is minimal by design.

## Communication format

All briefings and summaries use compressed natural language: clear, no articles or filler, still human-readable. No full sentences where a phrase will do. No hedging, no transitions, no process narrative.

Example — full English vs. compressed:

- Full: "Please refactor the validate_token function in src/auth.py to handle the case where refresh tokens have expired. Currently it throws an unhandled exception."
- Compressed: "Refactor validate_token() in src/auth.py:45-82. Currently throws unhandled exception on expired refresh token. Should attempt silent refresh, fail only if refresh token itself expired. Preserve function signature."

Same clarity, fewer tokens.

## Misuse catalog

Before dispatching any handoff, Claude checks the task against these patterns. Hard patterns pause execution and teach before proceeding. Soft patterns proceed silently and surface in a debrief after execution.

### Hard patterns — pause before dispatching

| # | Pattern | Detection signal |
|---|---------|-----------------|
| 1 | **Trivial task** | Task fits in one short sentence, no real work — a rename, lookup, one-liner |
| 2 | **Requires dialogue** | User wants to think through, explore, or discuss — not produce a deliverable |
| 3 | **Intermediate decisions needed** | Task has design forks where user must choose before work can continue |
| 4 | **Unresolvable ambiguity** | Too underscoped to write a meaningful briefing; would immediately generate questions.md |

### Soft patterns — proceed, debrief after

| # | Pattern | Detection signal |
|---|---------|-----------------|
| 5 | **Bloated briefing** | Authored content exceeds ~150–200 words before file references |
| 6 | **Context dump** | Claude pulls raw conversation into the briefing rather than distilling |
| 7 | **Missing deliverables** | Can't concretely fill the Deliverables section — no format, path, or success criteria |

### Lesson format

All lessons — hard warnings and soft debrief entries — use this structure:

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

## Workflow

### Step 0: Pre-dispatch check

Before triaging the task, check it against hard patterns 1–4 in the misuse catalog.

If a hard pattern is detected:
1. Present the structured lesson using the lesson format from the catalog
2. Stop — do not proceed to Step 1
3. Wait for the user to respond. If they say "hand off anyway", proceed from Step 1 as normal.

Example — Trivial task:

> You asked me to hand off: "rename `getUserById` to `fetchUserById` in auth.js"
>
> ### Trivial task
> **Detected:** Single rename in one file — no real work for a sub-agent.
> **Why this is a problem:** Spawning a sub-agent has overhead: briefing, spawn, summary. For a task this small, that overhead exceeds the work itself.
> **Rather than:** Handing off any task that mentions delegation
> **Instead:** Reserve handoff for tasks with real scope — multiple steps, file analysis, research, or anything that would take more than a minute inline.
>
> *To proceed anyway, say "hand off anyway."*

Example — Requires dialogue:

> You asked me to hand off: "help me think through the best approach for the auth refactor"
>
> ### Requires dialogue
> **Detected:** Task is exploratory — it needs back-and-forth to reach a conclusion, not a deliverable a sub-agent can produce independently.
> **Why this is a problem:** Sub-agents can't ask follow-up questions mid-task. They'll make assumptions and produce work you'll likely discard.
> **Rather than:** Handing off any task that involves deciding, discussing, or exploring
> **Instead:** Work through this inline where we can iterate together. Use handoff once there's a clear deliverable to produce.
>
> *To proceed anyway, say "hand off anyway."*

If no hard patterns are detected, proceed to Step 1.

### Step 1: Triage the task

Before building a briefing, assess what the task needs:

- **Freestanding task** — no conversation context required. User says "write a bash script that backs up postgres daily." Briefing is just the instruction plus any file references. Fast, cheap.
- **Context-dependent task** — task relies on decisions, constraints, or reasoning from the conversation. Distill the relevant thread into the briefing. Never dump raw conversation — synthesize only what the sub-agent can't infer from the filesystem.

If you can't determine which category it falls into, it's freestanding until proven otherwise.

### Step 2: Build the briefing

Write to `.handoffs/<slug>/briefing.md` where `<slug>` is a short descriptive name (e.g., `refactor-auth`, `q3-query`, `cache-research`).

**Format — include only sections the task requires:**

```
# <descriptive title>

## Objective
<What needs to be done. 1-3 sentences max, compressed.>

## Context
<Only if context-dependent. Distilled decisions/constraints from conversation. Not a transcript summary — the reasoning that shapes the task.>

## Inputs
<File references with targeted guidance. Never "read src/auth.py" — always "src/auth.py:45-82, validate_token() — currently throws on expired refresh tokens." Include inline excerpts for critical sections under 20 lines. Omit this section if no file inputs needed.>

## Deliverables
<What to produce and where to put it. Paths to output files.>

## Constraints
<Boundaries, conventions, things to avoid. Omit if none.>
```

**Briefing size gate:** If the briefing is growing beyond roughly 150-200 words of authored content (file path references don't count), stop. Either the task isn't well-scoped for handoff or it should be split into multiple handoffs. Do not ship a bloated briefing — break the task down instead.

While writing the briefing, watch for soft patterns from the misuse catalog:
- **Bloated briefing** (pattern 5): flag if authored content exceeds ~150–200 words
- **Context dump** (pattern 6): flag if you find yourself pulling raw conversation text rather than distilling decisions and constraints
- **Missing deliverables** (pattern 7): flag if you can't concretely fill the Deliverables section — no format, path, or success criteria

Do not interrupt execution. Flag any triggered patterns internally to surface in Step 4.

### Step 3: Spawn the sub-agent

```
Delegated task. Read briefing: .handoffs/<slug>/briefing.md
Save all output to: .handoffs/<slug>/output/
When complete, write: .handoffs/<slug>/output/SUMMARY.md

Summary format — compressed natural language, max 100 words:
1. What was produced (1-2 sentences)
2. Assumptions made, if any (list only)
3. Files created (paths only)
4. Follow-up items, if any (list only)

If summary requires more than 100 words to be meaningful, task was too broad — state this and recommend how to split it.

CONFIDENCE PROTOCOL:
- Normal ambiguity: make best judgment, document in summary under Assumptions
- Blocking ambiguity (proceeding would likely require rework): write questions to .handoffs/<slug>/questions.md and exit. Max 3 questions, compressed.
- More than 3 blocking questions: do not write questions.md. Instead write a single note to .handoffs/<slug>/needs-scoping.md explaining what's missing. Task needs better scoping, not Q&A.
```

Sub-agent has full tool access: read/write files, run scripts, install packages, bash. All intermediate work (scratch files, test scripts, logs) stays in `.handoffs/<slug>/`.

### Step 4: Handle the response

Check what came back:

**Happy path — SUMMARY.md exists:**
Read it and present to the user. This is the only content that enters the main context from the handoff. If the result is a short value (single function, one-liner, brief answer), include the substance inline alongside the summary. Otherwise, present summary and file paths — let the user decide what to pull in.

After presenting the summary, check whether any soft patterns were flagged during Step 2. If yes, append a **Handoff Quality** section using the lesson format from the misuse catalog:

---
**Handoff Quality**

[One entry per soft pattern detected, using the lesson format: Pattern name / Detected / Why this is a problem / Rather than / Instead]

---

Example:

---
**Handoff Quality**

### Bloated briefing
**Detected:** Briefing reached ~240 words before file references.
**Why this is a problem:** Large briefings signal a task that isn't well-scoped. The sub-agent gets noise alongside the signal, and the briefing is harder to revise or reuse.
**Rather than:** Writing everything you know about the problem
**Instead:** Distill to what the sub-agent can't infer from the files. 150 words is usually enough; if not, the task may need splitting.

---

If no soft patterns were flagged, nothing extra appears — do not add the section.

**Questions path — questions.md exists:**
Present the questions to the user. This is also a signal: the briefing wasn't targeted enough. Answer the questions, amend the briefing, relaunch. Don't treat this as normal — it means the handoff could have been scoped better.

**Needs scoping path — needs-scoping.md exists:**
Present the scoping feedback. The task needs to be rethought or broken down before it's ready for handoff. Don't try to fix it with more Q&A — restructure the ask.

### Step 5: Revisions

When revising a previous handoff's output, create a new handoff with a distilled briefing. Do not make the new sub-agent read the chain of previous briefings and outputs.

The revision briefing should contain:
- Current state of the output (what exists now, distilled — not "go read handoff 1, 2, and 3")
- What needs to change
- Same targeted file references as any other briefing, pointed at the previous output files

The distilled briefing for revision N should be roughly the same size as revision 1's briefing. It's a replacement, not an accumulation. Previous handoffs remain in `.handoffs/` for your reference but the sub-agent never needs them.

## Workspace structure

```
.handoffs/
├── refactor-auth/
│   ├── briefing.md
│   └── output/
│       ├── SUMMARY.md
│       └── auth.py
├── q3-query/
│   ├── briefing.md
│   ├── questions.md          ← sub-agent needed clarity
│   └── output/
│       ├── SUMMARY.md
│       └── query.sql
└── cache-research/
    ├── briefing.md
    ├── needs-scoping.md      ← task wasn't ready
    └── output/                ← empty, sub-agent stopped
```

Don't clean up old handoffs unless the user asks. They serve as reference without consuming context.

## When NOT to hand off

- Task requires back-and-forth dialogue with the user (sub-agent can't ask follow-ups beyond the initial questions.md)
- Task is trivially small — faster to just do it inline
- User needs to see intermediate work to make decisions
- Task can't be expressed in a concise briefing — needs more scoping first

## Quick reference

- "Hand off: write a migration for the users table" → full workflow
- "Delegate test writing to a sub-agent" → full workflow
- "What handoffs have we done?" → list .handoffs/ directories
- "Show me results from the auth handoff" → read that SUMMARY.md
- "Redo the last handoff with changes" → new handoff with distilled briefing
