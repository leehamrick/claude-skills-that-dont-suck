# Claude Code Skills

A collection of custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code).

## Available Skills

| Skill | Description | Dependencies |
|-------|-------------|--------------|
| **de-fluff** | Strip prompts down to what drives the output — removes irrelevant content, flowery language, and verbal noise, flags ambiguity, and teaches you why each change matters. Runs in a sub-agent to preserve your main context window. | handoff |
| **handoff** | Delegate tasks to a sub-agent to keep your main context window clean. General-purpose — works for code, writing, research, data analysis, or any self-contained task. | none |
| **sherpa** | Guide you through the Claude learning curve. Assesses your skills via a guided quiz, maps your knowledge terrain across 7 domains, and builds a personalized learning path to your summit. Saves a global learner profile that persists across sessions. | none |

## Install

### Option 1: Copy the SKILL.md (simplest)

Download the `SKILL.md` from the skill's folder and place it at:

```
~/.claude/skills/<skill-name>/SKILL.md
```

For example:
```bash
# macOS / Linux
mkdir -p ~/.claude/skills/de-fluff ~/.claude/skills/handoff
cp skills/de-fluff/SKILL.md ~/.claude/skills/de-fluff/
cp skills/handoff/SKILL.md ~/.claude/skills/handoff/
```

```powershell
# Windows (PowerShell)
mkdir "$env:USERPROFILE\.claude\skills\de-fluff" -Force
mkdir "$env:USERPROFILE\.claude\skills\handoff" -Force
Copy-Item skills\de-fluff\SKILL.md "$env:USERPROFILE\.claude\skills\de-fluff\"
Copy-Item skills\handoff\SKILL.md "$env:USERPROFILE\.claude\skills\handoff\"
```

### Installing Sherpa (beginner-friendly guide)

Sherpa works with Claude Code (desktop/terminal) and Claude on the web. Pick your platform:

---

**Claude web (claude.ai) — no installation required:**

1. Click this link to download the skill file: [Download SKILL.md](https://raw.githubusercontent.com/leehamrick/claude-skills-that-dont-suck/master/skills/sherpa/SKILL.md)
   - In your browser, that will open a page of text. Right-click anywhere on the page and choose **Save As** (or **Save Page As**). Save it as `SKILL.md` somewhere easy to find, like your Desktop.
2. Go to [claude.ai/customize/skills](https://claude.ai/customize/skills)
3. Click the button to add a new skill and upload the `SKILL.md` file you just saved
4. Start a **new** conversation at claude.ai
5. Type: `sherpa`

---

**On Mac (Claude Code):**

1. Open the Terminal app — search for "Terminal" in Spotlight (press `Cmd + Space`, type Terminal, press Enter)
2. Paste this command and press Enter:
   ```bash
   mkdir -p ~/.claude/skills/sherpa && curl -o ~/.claude/skills/sherpa/SKILL.md "https://raw.githubusercontent.com/leehamrick/claude-skills-that-dont-suck/master/skills/sherpa/SKILL.md"
   ```
3. Start a **new** Claude Code session
4. Type: `sherpa`

---

**On Windows (Claude Code):**

1. Open PowerShell — press the Windows key, type `PowerShell`, press Enter
2. Paste this command and press Enter:
   ```powershell
   mkdir "$env:USERPROFILE\.claude\skills\sherpa" -Force; Invoke-WebRequest -Uri "https://raw.githubusercontent.com/leehamrick/claude-skills-that-dont-suck/master/skills/sherpa/SKILL.md" -OutFile "$env:USERPROFILE\.claude\skills\sherpa\SKILL.md"
   ```
3. Start a **new** Claude Code session
4. Type: `sherpa`

---

**Did it work?** After typing `sherpa`, Claude should greet you and ask a question about where you're using it. If nothing happens, make sure you started a *new* session — skills only load at the start of a conversation.

### Option 2: Clone the whole repo

```bash
git clone https://github.com/<your-username>/claude-skills.git
```

Then copy the skills you want into `~/.claude/skills/`.

## Usage

Start a new Claude Code session after installing. Skills trigger on natural language — no slash commands needed.

**de-fluff** (preserves context window — use file or clipboard input):
- `de-fluff draft.txt` — reads from a file (best, keeps noise out of context)
- `de-fluff my clipboard` — reads from clipboard (good)
- `de-fluff this: [paste]` — inline (works, but noise enters context)

**handoff**:
- `hand off this task: [description]`
- `do this in the background: [description]`

## Dependencies

Some skills depend on others. Check the table above — install dependencies first. For example, **de-fluff** requires **handoff** to be installed for the sub-agent pattern to work.

## Contributing

Each skill lives in its own folder under `skills/` with a single `SKILL.md`. To add a new skill:

1. Create `skills/<skill-name>/SKILL.md`
2. Follow the frontmatter format (see existing skills for reference)
3. Add it to the table in this README
4. Note any dependencies

## License

MIT
