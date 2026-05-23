# Sherpa — Design Spec

**Date:** 2026-05-23
**Skill:** sherpa
**Companion skill:** sensei (separate spec)

---

## Overview

Sherpa is a guided onboarding and progression skill for Claude users. It assesses where a user currently is on the Claude learning curve, maps their knowledge terrain, and produces a personalized learning path to get them where they want to go. Designed primarily for beginners — people who don't know what they don't know — but useful for any user who wants a structured picture of their gaps.

The mountain metaphor is intentional and should be used throughout: *base camp, summit, route, waypoints, crevasses, gear, acclimatize, uncharted terrain.* It makes the experience feel like an adventure with a guide rather than a quiz.

---

## Scope — v1

**In scope:**
- Onboarding quiz (first run)
- Concept map of 7 knowledge domains
- Learning path (ordered by gap severity)
- Global learner profile (`~/.claude/sherpa-profile.md`)
- Check-in mode (subsequent runs)
- Profile import + portability guidance
- Environment detection + adaptation (Claude Code, Claude web)
- Beginner-friendly README (required deliverable alongside the skill)

**Out of scope (v2+):**
- Project scanning and skill suggestions based on current repo
- Environment recommendation ("you'd be better off in Claude Code")
- Sensei integration beyond reading the profile

---

## Two-skill system

Sherpa and Sensei are independent skills that share a profile file:

- **Sherpa** owns writing `~/.claude/sherpa-profile.md`
- **Sensei** reads it for calibration but works without it
- Both append to the Observations log in the profile
- Neither requires the other to be installed

---

## Mode detection

When Sherpa is invoked, it checks for `~/.claude/sherpa-profile.md`:

| State | Mode |
|-------|------|
| File doesn't exist | Onboarding (full quiz) |
| File exists | Check-in (brief update) |
| User says "sherpa reset" | Force onboarding, overwrite profile |

**Before the quiz — import check:**

> *"Before we start — have you used Sherpa before on another device?"*
> - A) Yes, I have a profile I'd like to bring over
> - B) No, starting fresh

If A: guide user to copy profile to `~/.claude/sherpa-profile.md`, then switch to check-in mode.

---

## Environment detection

Sherpa detects environment before the quiz starts. If detection is ambiguous, ask:

> *"Where are you using Claude right now?"*
> - A) In a web browser (claude.ai)
> - B) In the terminal (Claude Code)
> - C) Inside my code editor (VS Code or similar)

**Capability table:**

| Capability | Claude Code | Claude Web | IDE Extension |
|------------|-------------|------------|---------------|
| Skills | ✅ `~/.claude/skills/` | ✅ claude.ai/customize/skills | TBD |
| CLAUDE.md | ✅ | ❌ | TBD |
| Hooks | ✅ | ❌ | TBD |
| File system access | ✅ | ❌ | TBD |
| MCP servers | ✅ | TBD | TBD |

**Adaptation:**

- *Claude Code:* Full capability. Learning path includes skills, CLAUDE.md, hooks, workflow integration.
- *Claude web:* Skills available (imported via claude.ai/customize/skills), but no CLAUDE.md, hooks, or file system. Learning path focuses on prompt craft, conversation flow, skills. Install instructions adapted to web UI.
- *IDE extension:* TBD — flag as unknown in profile, advise user to check docs.

---

## Onboarding quiz

### Interface principles

- **One question at a time, always** — never multipart
- **Multiple choice wherever possible** — open answer only when the range is too wide to list
- **Plain language** — no jargon without explanation. Not "context window" — say "how much Claude can hold in mind at once"
- **Short options** — user should be able to read and pick in under 10 seconds
- **Warm and encouraging** — Sherpa is a guide, not an interviewer
- **Branching** — next question depends on previous answer
- **Sherpa tone throughout** — mountain metaphors, friendly puns

### Full question set (16–19 questions depending on path)

---

**Base camp — who are you**

Q1:
> *"First, let me get my bearings. What kind of work do you do?"*
> - A) I work in tech / I'm a developer
> - B) I work in a non-technical field
> - C) I'm a student
> - D) Something else

Q2 (if B, C, or D):
> *"How comfortable are you with technical tools — things like the command line, config files, or developer software?"*
> - A) Not at all — I prefer to avoid them
> - B) I can follow instructions but it's not natural
> - C) Pretty comfortable — I can figure most things out

Q2 (if A):
> *"How much development work do you do?"*
> - A) It's my main job
> - B) Some coding as part of other work
> - C) I used to code but not much anymore

Q3:
> *"What computer are you on?"*
> - A) Mac
> - B) Windows
> - C) Linux

Q4:
> *"Do you use Claude on more than one device?"*
> - A) Yes
> - B) No, just this one

*(If A → plant a flag: "Good to know — some of your setup lives on each machine separately. We'll cover that on the trail.")*

---

**Prior gear — what you're bringing**

Q5:
> *"Have you used other AI tools before Claude — like ChatGPT, Copilot, or Gemini?"*
> - A) Yes, regularly
> - B) A little
> - C) Claude is my first

Q6 (if A or B):
> *"Which ones?"* *(open answer — just names)*

Q7 (if A or B):
> *"What was your experience like?"*
> - A) Loved it — used it all the time
> - B) Useful but frustrating at times
> - C) Tried it, didn't really stick

---

**Checking the map — what you know about Claude**

Each misconception is its own question, framed as a simple scenario. Wrong answers become teaching moments, not failures.

Q8:
> *"When you close Claude and come back tomorrow — what do you think happens to our conversation?"*
> - A) It remembers everything
> - B) Fresh start — no memory of before
> - C) Not sure

Q9:
> *"Can Claude look things up on the internet?"*
> - A) Yes, it can search the web
> - B) No — it only knows what it learned in training
> - C) Not sure

Q10:
> *"If Claude tells you something confidently, how much do you trust it?"*
> - A) It's an AI — I assume it's accurate
> - B) Mostly, but I double-check important things
> - C) I always verify before I use it

---

**Current altitude — where you are now**

Q11:
> *"How long have you been using Claude?"*
> - A) Just getting started (days or weeks)
> - B) A few months
> - C) More than a year

Q12:
> *"What do you mostly use Claude for?"* *(can pick more than one)*
> - A) Writing and editing
> - B) Coding or technical work
> - C) Research and questions
> - D) Thinking through problems
> - E) Something else

Q13:
> *"How do you usually work with Claude?"*
> - A) One question, use the answer, done
> - B) Back-and-forth conversation
> - C) Big tasks — I hand it off and let it work

Q14:
> *"What's been working well?"* *(short open answer)*

Q15:
> *"What's been frustrating or not working?"* *(short open answer)*

---

**Crevasses — danger zones**

Q16:
> *"Do you work with sensitive information — like patient records, legal documents, or confidential business data?"*
> - A) Yes, regularly
> - B) Sometimes
> - C) No

Q17 (if A or B):
> *"Does your organization have compliance requirements around this data — like HIPAA, CJIS, or similar rules?"*
> - A) Yes, specific requirements
> - B) General "be careful" policies
> - C) I'm not sure

Q18:
> *"Does your workplace have rules about which AI tools you're allowed to use?"*
> - A) Yes — approved tools only
> - B) No specific rules
> - C) Not sure

---

**The summit**

Q19:
> *"What do you most want to be able to do with Claude that you can't do well yet?"* *(short open answer)*

---

**Environment add-on** (one question, auto-selected based on detected environment)

- *Claude Code:* "Are you using any skills, CLAUDE.md, or hooks yet?"
- *Claude web:* "Are you mostly having conversations, or trying to get Claude to produce finished work products?"
- *IDE:* "Are you using Claude mostly for code suggestions, or longer multi-step tasks?"

---

## Concept map

After the quiz, Sherpa synthesizes answers into a terrain map across 7 domains. Each domain is rated: **solid ground**, **uncharted**, or **crevasse** (active misconception).

| Domain | What it covers |
|--------|---------------|
| Prompt craft | Clear instructions, examples, scope, specificity |
| Claude's limits | Memory, hallucination, web access, context degradation |
| Conversation flow | Iteration, feedback, knowing when to start fresh |
| Context management | What to include, CLAUDE.md, /compact, session hygiene |
| Delegation | Sub-agents, handoff, when not to do it yourself |
| Workflow integration | Skills, hooks, automation |
| Data safety | What to share, what not to, compliance awareness |

**Presentation:** Plain language summary only. No scores, no grades. Example:

> *"Here's your terrain: Prompt craft looks solid — you're already thinking about what you're asking. Claude's limits has a crevasse we need to clear first: Claude doesn't remember our conversations, which explains some of the frustration you mentioned. Context management and delegation are uncharted — good ground to cover once we've sorted the crevasse."*

**Learning path ordering:** Crevasses first → biggest gaps second → nice-to-haves last. Each step maps to a concept and, where applicable, a skill to install.

---

## Profile format

Location: `~/.claude/sherpa-profile.md` (global, not project-specific)

```markdown
---
updated: 2026-05-23
environment: claude-code
os: windows
level: beginner
multi-device: true
---

## Summit
[stated goal from Q19]

## Solid ground
[strengths identified from quiz]

## Uncharted
[gaps ordered by priority]

## Crevasses
- [ ] Thinks Claude remembers between sessions
- [x] Thinks Claude can browse the web — cleared 2026-05-23

## Constraints
- Data sensitivity: HIPAA
- Org policy: approved tools only

## Learning path
1. [concept] How Claude memory actually works
2. [concept] Writing a clear prompt
3. [skill] Install: de-fluff
4. [concept] Context management basics

## Observations
### 2026-05-23 — Sherpa onboarding
[brief synthesis of quiz answers and initial assessment]

### 2026-05-30 — Sherpa check-in
[what changed, what clicked, new gaps surfaced]

### 2026-05-30 — Sensei
[something Sensei flagged during a session]
```

**Rules:**
- YAML frontmatter for metadata Sensei parses quickly
- Crevasse checkboxes: `[ ]` unresolved, `[x]` cleared with date
- Observations log is **append-only** — never overwritten, always dated and sourced
- Learning path is **rewritten** on each check-in as user progresses
- Onboarding answers stay in Observations permanently

---

## Check-in mode

3–4 targeted questions, ~2 minutes. Sherpa reads the profile before asking anything.

**Flow:**

1. Open with progress check:
> *"Welcome back. Last time we talked, you were working on [goal from profile]. How's the climb going?"*
> - A) Making progress — things are clicking
> - B) Mixed — some things work, some don't
> - C) Stuck — not much has changed

2. One follow-up based on answer

3. Check on open crevasses (one at a time):
> *"Quick check — last time you weren't sure whether Claude remembers our conversations. Has that started making sense?"*
> - A) Yes, makes sense now
> - B) Still a bit fuzzy
> - C) Still confusing

4. Open question:
> *"Anything new that's been frustrating or surprising since we last spoke?"*

**After check-in, Sherpa:**
- Updates frontmatter (`updated:`, `level:` if changed)
- Checks off cleared crevasses with date
- Appends dated entry to Observations log
- Rewrites learning path if significant progress made

---

## Profile portability

**Triggered by:** multi-device answer in quiz, OR always shown at end of onboarding as standing advice.

**Advice adapted to OS and technical comfort:**

| User type | Recommendation |
|-----------|----------------|
| Non-technical, Windows | Store profile in OneDrive — available on any signed-in Windows device |
| Non-technical, Mac | Store profile in iCloud Drive — follows you across Macs |
| Technical (any OS) | Add `~/.claude/` to dotfiles repo — profile, skills, and settings travel together |

Sherpa also flags what else lives locally and doesn't auto-travel: installed skills, settings.json. User knows what to re-setup on a new machine.

---

## README requirement

A beginner-friendly installation guide is a **required deliverable** alongside the skill. It must:

- Assume zero technical knowledge
- Provide separate step-by-step instructions for Windows, Mac, and Claude web
- Explain what Claude Code is before asking users to open a terminal
- Include a "did it work?" check at the end
- Use no jargon without plain-language explanation
- Address the chicken-and-egg problem: user needs Claude to install Sherpa, but Sherpa teaches Claude

---

## What Sensei gets from this profile

When Sensei reads `~/.claude/sherpa-profile.md`, it learns:
- User's level and OS
- Active crevasses to watch for
- Goals and summit
- Compliance constraints
- What skills are already on the learning path

Sensei uses this to calibrate lesson depth, vocabulary, and which patterns to prioritize. If no profile exists, Sensei works independently without calibration.
