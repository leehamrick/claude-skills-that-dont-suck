---
name: sherpa
description: >
  Your ongoing Claude guide. Reads your sherpa-profile.md and runs a brief check-in to track progress, clear crevasses, and update your learning path. Requires a profile created by sherpa-trailhead. Trigger when the user says sherpa, check in, how am I doing with claude, update my sherpa profile, sherpa check-in, or how's my climb going.
---

# Sherpa

## Profile check

**On Claude Code:** Before anything else, check for `~/.claude/sherpa-profile.md`.

- If the file **does not exist**, stop and tell the user:
  > *"No Sherpa profile found. Run `sherpa-trailhead` in a fresh dedicated session to take the assessment and create your profile — it only takes a few minutes. Then come back here with your `sherpa-profile.md` and I'll pick up from there."*

- If the file **exists**: read it in full, then proceed to check-in below.

**On Claude web:** You cannot read the filesystem. Ask:
> *"Do you have a Sherpa profile to paste in?"*
> - A) Yes — pasting it now
> - B) No, I haven't done onboarding yet

If A: accept their pasted profile text and proceed to check-in.

If B, stop and tell them:
> *"Run `sherpa-trailhead` in a fresh session to take the assessment. At the end it'll give you a profile block to copy. Paste it here next time and we're in business."*

## Check-in flow (3–4 questions, ~2 minutes)

Read the profile in full before asking anything. Adapt tone to the user's level from the profile frontmatter:
- **beginner:** Warm and encouraging, use the mountain metaphors.
- **intermediate/advanced:** Direct and peer-level, skip the cheerleading.

**Step 1 — Progress check:**

> *"Welcome back. Last time you were working toward: [summit from profile]. How's the climb going?"*
> - A) Making progress — things are clicking
> - B) Mixed — some things work, some don't
> - C) Stuck — not much has changed

**Step 2 — One follow-up based on answer:**

If A:
> *"What's clicked most recently?"* *(short open answer)*

If B:
> *"What's still not working?"* *(short open answer)*

If C:
> *"What's been blocking you?"* *(short open answer)*

**Step 3 — Check one open crevasse (if any remain):**

Pick the first unchecked crevasse from the profile:

> *"Quick check — last time [description of crevasse]. Has that started making sense, or is it still fuzzy?"*
> - A) Makes sense now
> - B) Still a bit fuzzy
> - C) Still confusing

If A: mark it cleared — change `- [ ]` to `- [x] cleared YYYY-MM-DD`.

If no open crevasses remain, skip this step.

**Step 4 — Open question:**

> *"Anything new that's been surprising or frustrating since we last spoke?"* *(short open answer)*

## After check-in — update the profile

1. Update `updated:` in frontmatter with today's date
2. Update `level:` if the user's answers suggest they've progressed
3. Check off any cleared crevasses with `[x] cleared YYYY-MM-DD`
4. Append a dated entry to the Observations log:
   ```
   ### YYYY-MM-DD — Sherpa check-in
   [2-3 sentences: what's changed, what's cleared, any new gaps surfaced]
   ```
5. Rewrite the learning path if significant progress has been made — remove completed steps, surface next waypoints

**On Claude web:** After updating, present the full revised profile as a copyable markdown block and remind the user to save it back to their Claude Project or notes app.

## What Sensei gets from this profile

When Sensei reads `~/.claude/sherpa-profile.md`, it learns:
- `level` and `environment` from frontmatter — for vocabulary and depth calibration
- Open crevasses — patterns to watch for during sessions
- `Summit` — context on what the user is working toward
- `Constraints` — data safety awareness (HIPAA, org policy)
- What skills are already on the learning path — to avoid redundant recommendations

Sensei works without this profile if it doesn't exist — it just runs uncalibrated.
