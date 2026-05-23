# Handoff Teaching Catalog Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a misuse catalog with structured lessons to the handoff skill, triggered by mistakes at runtime.

**Architecture:** Three additions to `skills/handoff/SKILL.md`: (1) a new Misuse Catalog section with 7 named patterns and lesson format, (2) a Step 0 pre-dispatch check for hard patterns, and (3) soft-pattern detection hooks in Steps 2 and 4. Frontmatter description updated to reflect new capability.

**Tech Stack:** Markdown only — no code, no dependencies.

---

### Task 1: Add the misuse catalog section

**Files:**
- Modify: `skills/handoff/SKILL.md`

Insert a new `## Misuse catalog` section immediately before the existing `## Workflow` section. This section defines all 7 patterns, their detection signals, and the lesson format used by both hard and soft lessons.

- [ ] **Step 1: Read the current file to find the insertion point**

Open `skills/handoff/SKILL.md` and locate the line that reads `## Workflow`. The new section goes immediately above it.

- [ ] **Step 2: Insert the catalog section**

Insert the following block immediately before `## Workflow`:

```markdown
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

```

- [ ] **Step 3: Verify the section is in the right place**

Read `skills/handoff/SKILL.md` and confirm: `## Misuse catalog` appears immediately before `## Workflow`, and the tables and lesson format block are intact.

---

### Task 2: Add Step 0 to the workflow

**Files:**
- Modify: `skills/handoff/SKILL.md`

Insert a new `### Step 0: Pre-dispatch check` immediately before the existing `### Step 1: Triage the task`.

- [ ] **Step 1: Find the insertion point**

Locate the line `### Step 1: Triage the task` in `skills/handoff/SKILL.md`. The new Step 0 goes immediately above it.

- [ ] **Step 2: Insert Step 0**

Insert the following block immediately before `### Step 1: Triage the task`:

```markdown
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

```

- [ ] **Step 3: Verify placement**

Read `skills/handoff/SKILL.md` and confirm: `### Step 0` appears immediately before `### Step 1: Triage the task`, and the examples are intact.

---

### Task 3: Modify Step 2 and Step 4

**Files:**
- Modify: `skills/handoff/SKILL.md`

Add soft-pattern detection to Step 2, and a Handoff Quality debrief hook to Step 4's happy path.

- [ ] **Step 1: Modify Step 2 — add soft-pattern flagging**

Locate the end of the `### Step 2: Build the briefing` section (just before `### Step 3`). Append the following paragraph:

```markdown
While writing the briefing, watch for soft patterns from the misuse catalog:
- **Bloated briefing** (pattern 5): flag if authored content exceeds ~150–200 words
- **Context dump** (pattern 6): flag if you find yourself pulling raw conversation text rather than distilling decisions and constraints
- **Missing deliverables** (pattern 7): flag if you can't concretely fill the Deliverables section — no format, path, or success criteria

Do not interrupt execution. Flag any triggered patterns internally to surface in Step 4.
```

- [ ] **Step 2: Modify Step 4 — add Handoff Quality debrief**

Locate the `**Happy path — SUMMARY.md exists:**` block inside `### Step 4: Handle the response`. After the existing instruction ("Read it and present to the user..."), append:

```markdown
After presenting the summary, check whether any soft patterns were flagged during Step 2. If yes, append a **Handoff Quality** section using the lesson format from the catalog:

---
**Handoff Quality**

[One entry per soft pattern detected]

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
```

- [ ] **Step 3: Verify both modifications**

Read `skills/handoff/SKILL.md` and confirm:
- Step 2 ends with the soft-pattern flagging paragraph
- Step 4's happy path includes the Handoff Quality debrief block with example

---

### Task 4: Update frontmatter description

**Files:**
- Modify: `skills/handoff/SKILL.md`

The current description doesn't mention misuse detection. Update it to reflect the new capability.

- [ ] **Step 1: Update the description field**

Replace the existing `description:` value in the frontmatter with:

```yaml
description: >
  Delegate a task to a sub-agent to preserve the main context window. Detects misuse and teaches: hard misuses (trivial tasks, dialogue-only tasks, unresolvable ambiguity, intermediate decisions needed) pause with a structured lesson before dispatching; soft misuses (bloated briefing, context dumping, missing deliverables) proceed and surface a Handoff Quality debrief after execution. Use this skill whenever the user asks to hand off, delegate, offload, farm out, or spin off a task, or when they say things like do this in the background, don't clutter the context, use a sub-agent for this, or handle this separately. Also trigger when the user implies context preservation, such as we're getting long can you do X on the side or run this without bloating our conversation. This skill is general-purpose and works for code, writing, research, data analysis, SQL, file processing, or any self-contained task. Even if the user doesn't use the word handoff, trigger whenever they want work done outside the main conversation thread to keep it clean.
```

- [ ] **Step 2: Verify the frontmatter**

Confirm the file still has valid YAML frontmatter: `---` open, `name:`, `description:`, `---` close.

---

### Task 5: Verify and commit

**Files:**
- Modify: `skills/handoff/SKILL.md`

Full read-through to confirm the skill is coherent end-to-end, then commit.

- [ ] **Step 1: Full read-through checklist**

Read `skills/handoff/SKILL.md` from top to bottom and verify:
- [ ] Frontmatter is valid YAML
- [ ] `## Misuse catalog` section exists before `## Workflow`
- [ ] Catalog has 4 hard patterns and 3 soft patterns in tables
- [ ] Lesson format block is present and complete
- [ ] `### Step 0` exists immediately before `### Step 1: Triage the task`
- [ ] Step 0 includes two worked examples (Trivial task, Requires dialogue)
- [ ] Step 0 ends with "If no hard patterns are detected, proceed to Step 1"
- [ ] Step 2 ends with soft-pattern flagging paragraph
- [ ] Step 4 happy path includes Handoff Quality debrief block with example
- [ ] No existing content was accidentally removed or duplicated

- [ ] **Step 2: Commit**

```bash
git add skills/handoff/SKILL.md
git commit -m "feat(handoff): add misuse catalog with structured teaching lessons"
```

Expected output: `1 file changed` on `skills/handoff/SKILL.md`.
