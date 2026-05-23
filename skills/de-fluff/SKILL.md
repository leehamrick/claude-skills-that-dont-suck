---
name: de-fluff
description: >
  Strip prompts down to what actually drives the output — removing irrelevant content (personal details, backstory, premature requirements, tangential context), flowery language, and verbal noise — then flag ambiguity and teach the user why each change matters. Sends the prompt to a separate context window (sub-agent) for analysis so the main conversation stays lean. Returns a line-by-line lesson showing what was cut or flagged, why, and what to write instead — so the user learns to write tighter, clearer prompts over time. Use this skill whenever the user says de-fluff, tighten this prompt, strip the fluff, compress this prompt, distill this, clean up this prompt, make this prompt leaner, clarify this prompt, or similar. Also trigger when the user pastes a long prompt and asks to shorten, reduce, or optimize it. This is a prompt preprocessor and teacher — it operates on input text the user provides, not on Claude's own responses.
---

# De-Fluff Skill

Strip prompts down to what actually drives the output. "Fluff" is anything in a prompt that doesn't help Claude produce a better result — and the biggest offender isn't flowery language, it's irrelevant content. Personal details, backstory, context that doesn't affect scope or function, information that's premature for the current task. Flowery language is also fluff, but it's the smaller problem. The sub-agent catches both, teaches the user why each piece was cut, and flags ambiguity. All analysis happens in a sub-agent so the work never pollutes the main context window.

## Fluff patterns — the lesson catalog

Every piece of fluff the sub-agent identifies gets tagged with a pattern name and a short lesson. These are the categories, ordered by impact — content noise first, then language noise.

### 1. Irrelevant personal information
The user's name, age, location, job title, or biographical details when none of it affects the task.
- **Lesson:** Claude doesn't need to know who you are to write your code, query, or document. Personal details consume context and add zero signal. Include them only when they directly shape the output — for example, your timezone matters for a scheduling function; your job title doesn't matter for a SQL query.
- Rather than: *"My name is John, I'm a 35 year old software engineer in Denver, and I need a Python script to parse CSV files"*
- Say: *"Write a Python script to parse CSV files. Input: [describe the CSV structure]."*

### 2. Backstory and motivation that doesn't affect scope
Why the user needs the thing, how they got here, what led them to this point — when none of it changes what Claude should build.
- **Lesson:** Claude builds to spec, not to story. "I've been struggling with this for weeks" doesn't change the output. "My boss asked me to do this" doesn't either. Save the backstory for humans. Tell Claude what to build, not why you need it built — unless the "why" genuinely constrains the "what."
- Rather than: *"We've been having issues with our deployment pipeline for months now and my team is really frustrated, so I was hoping you could help us write a script that checks if our Docker containers are healthy"*
- Say: *"Write a script that health-checks Docker containers. Check: [HTTP 200 on /health | process running | memory under threshold]. Output: [JSON | exit code | log line]."*

### 3. Premature details
Information that might matter later but doesn't affect the current task. Future requirements, hypothetical extensions, "down the road" features.
- **Lesson:** Claude works on the prompt in front of it. Future context dilutes current focus. Add details when they become relevant — not before. If you mention a future requirement, Claude may try to accommodate it now and over-engineer the solution.
- Rather than: *"Build a user registration form. Eventually we'll need SSO, LDAP integration, and multi-tenant support, but for now just email and password"*
- Say: *"Build a user registration form. Fields: email, password. Validate email format, enforce minimum 8-char password."*

### 4. Tangential technical context
Stack details, architecture decisions, or system descriptions that don't affect the specific task being asked.
- **Lesson:** Don't describe your whole system when you need help with one function. Claude needs to know what touches the task — not everything that exists. The more irrelevant context you provide, the more likely Claude will try to integrate with things you didn't ask it to touch.
- Rather than: *"We use React 18 with Next.js 14, Prisma ORM, PostgreSQL 15, Redis for caching, Stripe for payments, and SendGrid for email. I need a utility function that formats phone numbers."*
- Say: *"Write a TypeScript utility function that formats US phone numbers as (XXX) XXX-XXXX. Input: string of 10 digits. Handle common variations: spaces, dashes, dots, leading +1."*

### 5. Filler openers
The throat-clearing before the actual request.
- "I'd be happy to..." / "Sure thing!" / "Could you please kindly..."
- **Lesson:** LLMs don't need warming up. Start with the verb. "Write...", "List...", "Explain..."
- Rather than: *"I was wondering if you could perhaps help me write a SQL query"*
- Say: *"Write a SQL query that..."*

### 6. Redundant restatements
Saying the same thing twice in different words, often for emphasis that doesn't land.
- **Lesson:** If you said it clearly once, the model heard it. Repetition burns tokens without adding signal.
- Rather than: *"Sort the results in ascending order, from smallest to largest"*
- Say: *"Sort ascending"*

### 7. Over-hedging
Qualifiers that soften the request without changing its meaning.
- "It's worth noting that..." / "Perhaps it would be beneficial to..." / "It might be useful to consider..."
- **Lesson:** Hedging is a social habit, not a technical one. Models respond to direct instructions. Hedging can actually reduce output quality by making the instruction ambiguous.
- Rather than: *"It might be worth considering adding error handling, if you think that would be appropriate"*
- Say: *"Add error handling"*

### 8. Verbose phrasing
Using many words where few will do.
- "In order to" → "To" / "Due to the fact that" → "Because" / "At this point in time" → "Now" / "In the event that" → "If"
- **Lesson:** Prepositions and conjunctions have short forms. Use them. Every extra word dilutes the signal-to-noise ratio of your prompt.
- Rather than: *"In order to make sure that the function is able to handle edge cases"*
- Say: *"Handle edge cases"*

### 9. Polite padding
Social pleasantries that cost tokens and add nothing to instruction clarity.
- "If you don't mind..." / "I'd really appreciate it if..." / "Thank you so much in advance!"
- **Lesson:** Politeness is for humans. Models parse instructions, not manners. Save the tokens for specificity.
- Rather than: *"If you don't mind, could you please also add some comments to the code? That would be really helpful, thank you!"*
- Say: *"Add comments"*

### 10. Meta-commentary
Describing what the prompt is instead of just being the prompt.
- "This prompt asks you to..." / "The following is a request for..." / "What I'm looking for here is..."
- **Lesson:** Don't narrate the prompt. The model knows it's reading a prompt. Just give the instruction.
- Rather than: *"What I'm looking for here is a function that validates email addresses"*
- Say: *"Write a function that validates email addresses"*

### 11. Flowery language
Adjective-heavy, emotional, or dramatic phrasing that adds color but not information.
- **Lesson:** "Craft an elegant, robust, beautifully architected solution" tells Claude nothing that "Write a clean solution" doesn't. Adjectives that don't map to measurable requirements are noise.
- Rather than: *"I need a really elegant, clean, well-structured, beautifully organized dashboard that perfectly displays all the critical metrics in a visually stunning way"*
- Say: *"Build a dashboard. Metrics: [list them]. Layout: [grid | sidebar + main | tabs]. Clean design, readable at a glance."*

## Ambiguity patterns — the clarity catalog

Fluff wastes tokens. Ambiguity wastes entire responses — Claude has to guess what you meant, and when it guesses wrong you burn a round trip correcting it. The sub-agent flags these and suggests specific rewrites.

### 12. Vague scope
The prompt doesn't specify how much, how many, how deep, or what format.
- **Lesson:** Claude will pick a default scope, and it might not be yours. Specify length, depth, count, or format upfront. One constraint sentence saves a whole re-prompt.
- Rather than: *"Explain how DNS works"*
- Say: *"Explain how DNS works in 3-4 paragraphs, covering recursive resolution and caching. Audience: someone who understands IP addresses but not DNS."*

### 13. Undefined pronouns and references
"It", "this", "that", "the thing", "the data" — when it's unclear what they refer to.
- **Lesson:** Claude doesn't have your screen or your mental context. Every pronoun should resolve to a noun within the prompt itself. If you're referencing something from earlier in the conversation, name it.
- Rather than: *"Now update it to handle the edge case we discussed"*
- Say: *"Update the validate_email() function to handle the empty-string edge case"*

### 14. Implicit expectations
The prompt assumes Claude knows something it wasn't told — output format, language, framework, audience, tone.
- **Lesson:** If you'd be disappointed by a wrong choice, it wasn't optional — it was an unstated requirement. Make it stated.
- Rather than: *"Build me a login form"*
- Say: *"Build a login form in React with Tailwind. Fields: email, password. Include client-side validation and a 'forgot password' link."*

### 15. Competing instructions
The prompt says two things that pull in opposite directions, or sets a constraint then immediately undermines it.
- **Lesson:** Claude will try to satisfy both and often compromise both. If you want two different things, make them two separate prompts, or explicitly state which takes priority.
- Rather than: *"Keep it short but make sure you cover every edge case in detail"*
- Say: *"Cover the 3 most common edge cases. 1-2 sentences each."*

### 16. Unmarked optional vs. required
The prompt lists several things but doesn't distinguish must-haves from nice-to-haves, so Claude treats everything as equally important (or equally skippable).
- **Lesson:** If some items are optional, say so. If some are critical, mark them. Otherwise Claude may skip your dealbreaker to make room for your nice-to-have.
- Rather than: *"Add error handling, logging, retry logic, rate limiting, and maybe some unit tests"*
- Say: *"Required: error handling, retry logic with exponential backoff. Optional if time: logging, rate limiting, unit tests."*

## What gets preserved

- Technical precision (domain terms, specific constraints, exact requirements)
- Structural intent (if the original uses numbered steps or sections for a reason, keep the structure)
- Tone directives (if the prompt specifies a voice or audience, keep that)
- Examples and edge cases (these are high-signal, not fluff)
- Role/persona instructions

## Workflow

### Step 1: Receive the prompt

The user triggers de-fluff in one of three ways. **The first two are preferred** because they keep the fluffy prompt out of the main context window entirely — which is the whole point of using a sub-agent.

**Method A — File input (best).** The user says something like:
- "de-fluff the prompt in draft.txt"
- "de-fluff ~/prompts/my-prompt.md"
- "de-fluff the file I just saved"

The main context only sees the filename. The sub-agent reads the file, processes it, and returns only the clean version and lessons. The fluffy prompt never enters the conversation.

**Method B — Clipboard input (good).** The user copies the prompt to their clipboard and says:
- "de-fluff my clipboard"
- "de-fluff what I just copied"

Read the clipboard contents using the appropriate platform command:
- Windows: `powershell -command "Get-Clipboard"`
- macOS: `pbpaste`
- Linux: `xclip -selection clipboard -o` or `xsel --clipboard --output`

Pipe the clipboard contents directly into the briefing file. The fluffy prompt never enters the conversation — only the trigger phrase does.

**Method C — Inline paste (fallback).** The user pastes the prompt directly into the chat:
- "de-fluff this: [prompt text]"

This still works, but the fluffy prompt is now in the main context window. The sub-agent analysis stays isolated, but the original noise is already in the conversation. This is the least ideal method. If the user does this, process it normally but include a tip in the output:

> **Tip:** Next time, save your prompt to a file and say "de-fluff draft.txt", or copy it and say "de-fluff my clipboard." That way the fluffy version never enters your context window.

### Step 2: Hand off to sub-agent

Determine the input source:
- **File:** Read the file path the user specified. Write its contents into the briefing's Input section.
- **Clipboard:** Read the clipboard using the platform-appropriate command (see Step 1). Write the clipboard contents into the briefing's Input section.
- **Inline:** Copy the user's pasted text into the briefing's Input section.

In all three cases, the briefing file is what the sub-agent reads — the input source doesn't change the analysis, only how the prompt gets into the briefing.

Write the briefing to `.handoffs/de-fluff-<timestamp>/briefing.md` following the handoff skill pattern. The briefing contains:

```
# De-Fluff: Prompt Distillation + Clarity Check + Lesson

## Objective
Two passes on the following prompt:
1. REDUCE — remove filler, hedging, redundancy, polite padding, verbose phrasing, meta-commentary. Preserve all technical specifics, constraints, examples, and structural intent.
2. CLARIFY — flag ambiguity that would force Claude to guess: vague scope, undefined references, implicit expectations, competing instructions, unmarked optional-vs-required items. For each ambiguity found, propose a specific rewrite that resolves it. When proposing a rewrite for an ambiguity, DO NOT invent requirements — instead, show the user a template with bracketed placeholders for the missing specifics they need to fill in.

The goal is to help the user learn to write tighter, clearer prompts on their own.

## Rules — Reduction
- The primary target is CONTENT NOISE: information that doesn't affect what Claude should build. Personal details (name, age, location, job title) unless they directly shape the output. Backstory and motivation unless the "why" constrains the "what." Premature future requirements. Tangential system architecture. Strip all of it.
- The secondary target is LANGUAGE NOISE: filler openers, redundant restatements, over-hedging, verbose phrasing, polite padding, meta-commentary, flowery adjectives.
- Never remove or alter a technical requirement, constraint, or example
- Compress verbose phrasing ("in order to" → "to", "due to the fact that" → "because")
- Strip filler openers and closers ("I'd like you to..." → just state the instruction)
- Remove redundant restatements — if the same idea appears twice, keep the clearer version
- Remove meta-commentary about what the prompt is ("This prompt asks..." → just ask)
- Remove hedging qualifiers that add no meaning ("perhaps", "maybe", "it might be worth")
- Replace flowery adjectives with measurable constraints or remove them ("elegant, beautiful" → specify what makes it good, or cut it)
- Keep the prompt's structural format (bullets, numbered steps) if it serves clarity
- Do NOT rephrase into your own preferred style — stay faithful to the author's intent
- Do NOT add anything that wasn't in the original
- When removing content noise, briefly note in the lesson why that information doesn't serve the task — the user may not realize it's irrelevant

## Rules — Clarity
- Flag ambiguity only when Claude would genuinely be forced to guess — not every prompt needs a word count or audience specified
- For each ambiguity, propose a rewrite using [bracketed placeholders] for the specifics the user needs to supply, not invented defaults
- Distinguish between ambiguity (Claude has to guess) and flexibility (the user intentionally left it open). If "build me a landing page" doesn't specify a framework, that's ambiguity worth flagging. If "write a poem about rain" doesn't specify a form, that's probably intentional flexibility — don't flag it unless the lack of constraint would likely produce a disappointing result
- Competing instructions are the highest-priority ambiguity — they almost always produce bad output

## Input
<If file input: read the file contents and paste them here>
<If clipboard input: read the clipboard contents and paste them here>
<If inline paste: paste the user's prompt here>

## File writing — platform awareness
- Detect the OS from the file paths in this briefing. If paths use backslashes or start with a drive letter (C:\, D:\, etc.), you are on Windows.
- On Windows: use PowerShell to write files. Do NOT use bash heredocs, `cat >`, or Unix-style shell commands — they will fail or corrupt output. Use PowerShell's `Set-Content` or `Out-File`, or the Write tool if available.
- On macOS/Linux: bash heredocs or the Write tool are fine.
- Always create the output directory before writing if it doesn't exist. On Windows: `New-Item -ItemType Directory -Force -Path "..."`. On macOS/Linux: `mkdir -p "..."`.

## Deliverables
Write three files:

1. `output/clean.md` — the de-fluffed AND clarified prompt, ready to use. Fluff is removed outright. Ambiguities are rewritten with [bracketed placeholders] where the user needs to fill in specifics.

2. `output/lessons.md` — the teaching breakdown. For each finding (fluff or ambiguity), format as:

   ### Finding N
   **Type:** FLUFF or AMBIGUITY
   **Pattern:** [pattern name from catalog]
   **Original:** "[exact text from the user's prompt]"
   **De-fluffed / Clarified:** "[what it became, or REMOVED if fully cut]"
   **Why:** [1-2 sentence explanation]
   **Rather than:** [the problematic phrasing pattern, generalized]
   **Say:** [the better phrasing pattern, generalized — something the user can apply to future prompts]

   Order findings by their position in the original prompt (top to bottom). Group adjacent findings if they're the same pattern. List AMBIGUITY findings after FLUFF findings in a separate section.

3. `output/SUMMARY.md` — compressed summary:
   - Original word count → reduced word count (percentage reduction)
   - Count of findings by pattern category
   - Count of ambiguities found, with severity (high = Claude must guess, medium = Claude will pick a reasonable default but might miss your intent)
   - Top 1-2 habits to break (the patterns that appeared most often)
   - Top ambiguity to resolve (the single ambiguity most likely to produce a bad response if left unaddressed)
```

### Step 3: Present results

Read the sub-agent's output and present to the user:

1. Show the **reduction stats** (word count before → after, percentage)
2. Show the **top habits to break** — the fluff patterns that appeared most
3. Walk through the **fluff lessons** — each finding as a "rather than X, say Y"
4. Walk through the **ambiguity findings** — each one with the original, why it's ambiguous, and a suggested rewrite with [bracketed placeholders] for the user to fill in
5. Show the **de-fluffed + clarified prompt** in full so they can review and fill in any placeholders
6. Ask: "Want to use this version, edit it, or try again?"

The teaching breakdown is the primary output. The clean prompt is the bonus. Ambiguity findings are presented as questions the user needs to answer — not assumptions the skill makes for them.

The user approves, tweaks, fills in placeholders, or requests another pass. The clean prompt never enters the main context until the user explicitly adopts it.

## Example

**Input prompt (94 words):**
> Hi, my name is Sarah and I'm a frontend developer at a mid-size fintech startup in Austin. We've been struggling with our codebase for the past few months and my manager really wants us to clean things up. I was wondering if you could perhaps help me write a React component that displays a sortable data table. Eventually we'll need it to support pagination, CSV export, and real-time updates, but for now just basic sorting would be great. Oh, and if you don't mind, please make it really elegant and well-structured. Thanks so much!

**De-fluffed + clarified (35 words + 2 placeholders):**
> Write a React component: sortable data table. Columns: [list column names and types]. Sort: click column header to toggle asc/desc. Data source: [props array | API endpoint | static].

**Reduction: 63%**

**Lessons:**

| # | Type | Pattern | What was cut | Why |
|---|------|---------|-------------|-----|
| 1 | FLUFF | Irrelevant personal info | "my name is Sarah and I'm a frontend developer at a mid-size fintech startup in Austin" | None of this affects the component spec. Your name, title, company size, and city don't change what gets built. |
| 2 | FLUFF | Backstory | "We've been struggling with our codebase for the past few months and my manager really wants us to clean things up" | This explains *why* you need it, but doesn't affect *what* to build. Claude builds to spec, not to story. |
| 3 | FLUFF | Filler opener | "I was wondering if you could perhaps help me" | Start with the verb: "Write a React component..." |
| 4 | FLUFF | Premature details | "Eventually we'll need it to support pagination, CSV export, and real-time updates" | Future requirements dilute current focus. Claude may over-engineer to accommodate things you didn't ask for yet. Add these when you're ready to build them. |
| 5 | FLUFF | Polite padding | "Oh, and if you don't mind, please" / "Thanks so much!" | Just state the requirement. |
| 6 | FLUFF | Flowery language | "really elegant and well-structured" | These adjectives don't map to measurable requirements. What does "elegant" mean? Clean code? Minimal dependencies? Specify the actual quality you want. |
| 7 | AMBIGUITY | Vague scope | "a sortable data table" — what columns? what data types? | "Columns: [list column names and types]" — fill in your specific columns |
| 8 | AMBIGUITY | Implicit expectations | No data source specified — props? API? hardcoded? | "Data source: [props array \| API endpoint \| static]" — fill in how the data arrives |

**Top habit to break:** Content noise — 40 of your 94 words were personal info and backstory that don't affect the output. That's 43% of your prompt doing nothing.
**Top ambiguity to resolve:** Column structure — Claude can't build a data table without knowing what's in it.

## Edge cases

- **Very short prompts** (under 20 words): Tell the user the prompt is already lean, no reduction needed. Don't force changes.
- **Prompts that are mostly technical** (code, SQL, configs): These are usually already dense. Focus only on any surrounding natural language, don't touch the technical content.
- **Prompts with intentional repetition** (emphasis, rhetorical): Use judgment. If repetition serves a purpose (e.g., reinforcing a critical constraint), keep it.
