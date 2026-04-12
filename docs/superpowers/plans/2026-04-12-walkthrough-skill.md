# walkthrough-skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Create a Claude Code plugin repository for the `walkthrough` skill — an adaptive, interactive guide for understanding any codebase.

**Architecture:** Single SKILL.md file with two modes (`learn` / `debug`) and a default handler, packaged as a Claude Code plugin with `.claude-plugin/` metadata. No runtime code — all files are configuration and Markdown.

**Tech Stack:** Markdown, JSON, Git, GitHub CLI (`gh`)

---

## File Map

| Action | Path | Responsibility |
|--------|------|----------------|
| Create | `.claude-plugin/plugin.json` | Plugin identity and metadata |
| Create | `.claude-plugin/marketplace.json` | Marketplace listing |
| Create | `skills/walkthrough/SKILL.md` | Full skill logic (EN, patched) |
| Create | `README.md` | English-language documentation |
| Already exists | `docs/superpowers/specs/2026-04-12-walkthrough-skill-design.md` | Design spec (do not modify) |
| Already exists | `docs/superpowers/plans/2026-04-12-walkthrough-skill.md` | This plan (do not modify) |

---

### Task 1: Initialize git repository

**Files:**
- Creates: `.git/` at `walkthrough-skill/`

- [ ] **Step 1: Verify working directory**

```bash
pwd
# Expected: /home/arthur/dev/walkthrough-skill
```

- [ ] **Step 2: Initialize git**

```bash
cd /home/arthur/dev/walkthrough-skill && git init
# Expected: Initialized empty Git repository in /home/arthur/dev/walkthrough-skill/.git/
```

- [ ] **Step 3: Verify**

```bash
git status
# Expected: On branch main (or master), no commits yet
```

---

### Task 2: Create `.claude-plugin/plugin.json`

**Files:**
- Create: `.claude-plugin/plugin.json`

- [ ] **Step 1: Create the file**

```json
{
  "name": "walkthrough",
  "description": "Interactive guide for understanding any codebase. Two modes: /walkthrough learn (pedagogical, with level inference and study plan) and /walkthrough debug (focused on resolving a specific problem).",
  "version": "1.0.0",
  "author": {
    "name": "MaedaArthur",
    "email": "maedaarthur1607@gmail.com"
  },
  "homepage": "https://github.com/MaedaArthur/walkthrough-skill",
  "repository": "https://github.com/MaedaArthur/walkthrough-skill",
  "license": "MIT",
  "keywords": ["learning", "education", "debugging", "portfolio", "walkthrough"]
}
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool .claude-plugin/plugin.json > /dev/null && echo "VALID" || echo "INVALID"
# Expected: VALID
```

---

### Task 3: Create `.claude-plugin/marketplace.json`

**Files:**
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: Create the file**

```json
{
  "name": "walkthrough-skill",
  "description": "Marketplace for the walkthrough pedagogical skill for Claude Code",
  "owner": {
    "name": "MaedaArthur",
    "email": "maedaarthur1607@gmail.com"
  },
  "plugins": [
    {
      "name": "walkthrough",
      "description": "Interactive guide for understanding any codebase — Learn mode and Debug mode",
      "version": "1.0.0",
      "source": "./"
    }
  ]
}
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool .claude-plugin/marketplace.json > /dev/null && echo "VALID" || echo "INVALID"
# Expected: VALID
```

---

### Task 4: Create `skills/walkthrough/SKILL.md`

This is the core of the plugin. Full English translation of the original PT-BR SKILL.md plus 3 patches: default mode handler, vocabulary-based level inference (replaces explicit questionnaire), large project branch in Step 1.

**Files:**
- Create: `skills/walkthrough/SKILL.md`

- [ ] **Step 1: Create the file with full content**

```markdown
---
name: walkthrough
description: >
  Interactive guide for understanding any codebase — personal projects, open source, legacy code, or anything else.
  Trigger when: user says they don't understand the code, asks how something works,
  wants to learn the architecture, is stuck on a bug, wants to add the project to their portfolio.
  Explicit activation: /walkthrough learn | /walkthrough debug
---

# Walkthrough Skill

Interactive guide that adapts explanations to the user's level. Never explains everything at once — uses checkpoints to avoid wasting tokens and to ensure the user absorbed the content before moving on.

**Token economy principle:** be dense. No recaps, no "as I mentioned before", no long introductions. Short analogies beat paragraphs. Glossary is inline, never a separate section.

---

## DEFAULT (no mode specified)

Ask the user:
> "Do you want to understand the project from scratch (`learn`) or track down a specific problem (`debug`)?"

Wait for the answer, then route to the correct mode.

---

## MODE LEARN `/walkthrough learn`

### Step 1 — Find existing context

Before reading the code, look for:
- `CLAUDE.md` at the project root
- `README.md` or `README`
- `docs/` folder
- Header comments in main files

**If context found:** use as base. Read code only to fill what's missing.

**If not found:** read code directly. At the end, suggest the user create a README or CLAUDE.md with the generated summary — it will speed up future walkthroughs.

**Large project branch:** if the project has 10+ relevant files or multiple architectural layers → generate the architecture map first, then ask:
> "Which part do you want to dive into?"

Do not attempt to walk through everything at once. Wait for the user to choose before starting blocks.

---

### Step 2 — Architecture map

Generate a quick, dense map of the project. Include:
- Detected stack (languages, frameworks, external services)
- Relevant folder structure (ignore node_modules, .git, etc.)
- Main flow in 1 paragraph: what comes in, what happens, what goes out
- Text diagram if the architecture has multiple layers

Example diagram format:
\```
[Browser] → [Next.js pages/] → [API routes/] → [Supabase]
                                      ↓
                              [Django backend]
\```

Keep it concise. The map should fit in ~20 lines.

---

### Step 3 — Level inference

No calibration questions. Infer the user's level silently from the vocabulary they use when responding to what they want to explore or when asking follow-up questions.

- Technical terms used correctly → intermediate/advanced
- Behavior described without technical terms → beginner
- Mix of both → beginner with some exposure

Never disclose the inferred level. Use it only to calibrate explanations throughout the walkthrough.

---

### Step 4 — Walkthrough in blocks

Divide the code into logical blocks (by responsibility, not by file). For each block:

1. Name the block: **"[block name] — what it does"**
2. Explain at the inferred level
3. Inline glossary: when a new term appears for the first time, explain it in parentheses on the same line. Ex: *"here we use middleware (code that runs between the request and the response)"*
4. End with a checkpoint:

> "Does this make sense? Reply 'yes', 'no', or ask a question before we continue."

**Do not advance to the next block without a response.**

If the user replies "no" or asks a question: explain with a different analogy, then checkpoint again.
If "yes": advance.

---

### Step 5 — Study plan

After finishing all selected blocks, generate a personalized study plan based on what was explained and the inferred level. Format:

\```
## Your study plan for this project

**Why this matters for your portfolio:** [1-2 sentences about what this project demonstrates]

**Priority 1 — [Topic]** (Level: beginner/intermediate)
Why: appears in [part of the code] and is essential for debugging/evolving the project
Search: "[exact term to look up]"

**Priority 2 — [Topic]** ...

**Priority 3 — [Topic]** ...
\```

Limit to 5 topics. Prioritize by impact on the current project, not by what "would be good to know in general".

---

## MODE DEBUG `/walkthrough debug`

### Step 1 — Listen to the problem

Ask the user to describe the problem in their own words:

> "Tell me what's happening — what did you expect it to do, and what is it doing instead?"

**While reading the description, infer the user's level from vocabulary:**
- Uses technical terms correctly → intermediate/advanced
- Describes behavior without technical terms → beginner
- Mix of both → beginner with some exposure

Do not make the diagnosis explicit. Use the level internally to calibrate the explanation.

---

### Step 2 — Locate the problem

Find the most likely point of failure in the code. Prioritize:
1. The file/function mentioned by the user
2. The flow that produces the described behavior
3. External dependencies (APIs, database, environment variables)

Show the user **where** the problem is before explaining **why**.

> "The problem is likely in `[file]`, ~line X, in this function: [short code snippet]"

---

### Step 3 — Explain only what's needed

Explain only what is necessary to understand and fix the bug. No extra context.

Use the inferred level:
- **Beginner:** analogy + what changes in practice
- **Intermediate:** what the code is doing vs. what it should do
- **Advanced:** the tradeoff or edge case that caused the problem

---

### Step 4 — Focused recommendation

End with 1-2 study topics directly tied to the gap that caused the bug:

> "This bug appeared because [concept X] works differently than expected. Worth studying:
> - **[topic]** — search: '[term]'
> - **[topic]** — search: '[term]'"

No full study plan in debug mode. Full focus on the problem.

---

## General rules

- **Never explain everything at once.** Checkpoints exist to save tokens and ensure absorption.
- **Glossary is always inline.** Never create a separate glossary section.
- **No recaps.** Do not repeat what was already covered in prior blocks.
- **Short analogies.** One analogy sentence beats three technical paragraphs for beginners.
- **If no project docs found:** suggest at the end that the user creates documentation — explain that it will speed up future walkthroughs and improve explanation quality.
```

- [ ] **Step 2: Verify file exists and is non-empty**

```bash
wc -l skills/walkthrough/SKILL.md
# Expected: 130+ lines
```

---

### Task 5: Create `README.md`

**Files:**
- Create: `README.md`

- [ ] **Step 1: Create the file**

```markdown
# walkthrough-skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An adaptive, interactive guide for understanding any codebase — legacy code, open source projects, personal work, or anything in between. Two modes: **learn** for structured exploration and **debug** for pinpointing a specific problem. Adjusts explanation depth automatically based on vocabulary inferred from the user's responses — no questionnaires.

---

## Installation

```bash
/plugin marketplace add MaedaArthur/walkthrough-skill
/plugin install walkthrough@walkthrough-skill
```

---

## How to use

### `/walkthrough learn`

Maps the project architecture, then asks which part you want to explore. Walks through the codebase in logical blocks with checkpoints — won't move on until you've confirmed you understood. Ends with a personalized study plan tied to the specific code you just reviewed.

### `/walkthrough debug`

Asks you to describe what's broken and what you expected. Locates the most likely failure point in the code, explains only what's needed to understand and fix it, and closes with 1-2 targeted study topics tied to the gap that caused the bug.

### `/walkthrough` (no mode)

Prompts you to choose between `learn` and `debug`.

---

## How it works

**Context search:** Before reading code, looks for `CLAUDE.md`, `README.md`, or a `docs/` folder to use as a base. Falls back to reading the code directly if none are found.

**Level inference:** No calibration questions. Your level (beginner / intermediate / advanced) is inferred silently from the vocabulary you use in responses. Never disclosed — used only to calibrate explanations.

**Checkpoints:** Each block ends with a checkpoint. The walkthrough doesn't advance until you confirm you understood or ask a question. Keeps explanations focused and token-efficient.

**Study plan:** At the end of a `learn` session, generates a prioritized study plan based on what was covered and your inferred level — tied to the actual code, not generic topics.
```

- [ ] **Step 2: Verify file exists**

```bash
wc -l README.md
# Expected: 50+ lines
```

---

### Task 6: Initial commit

**Files:**
- All files created in Tasks 2–5 + existing docs/

- [ ] **Step 1: Stage all files**

```bash
git add .claude-plugin/plugin.json \
        .claude-plugin/marketplace.json \
        skills/walkthrough/SKILL.md \
        README.md \
        docs/
```

- [ ] **Step 2: Verify staged files**

```bash
git status
# Expected: 6 files staged, nothing unstaged
```

- [ ] **Step 3: Commit**

```bash
git commit -m "feat: initial walkthrough skill"
# Expected: [main (root-commit) xxxxxxx] feat: initial walkthrough skill
#            6 files changed, ...
```

---

### Task 7: Create GitHub repository and push

**Files:**
- No file changes — remote operations only

- [ ] **Step 1: Create public repo on GitHub**

```bash
gh repo create MaedaArthur/walkthrough-skill \
  --public \
  --description "Interactive guide for understanding any codebase — Learn and Debug modes for Claude Code"
# Expected: ✓ Created repository MaedaArthur/walkthrough-skill on GitHub
```

- [ ] **Step 2: Add remote and push**

```bash
git remote add origin https://github.com/MaedaArthur/walkthrough-skill.git
git push -u origin main
# Expected: Branch 'main' set up to track remote branch 'main' from 'origin'.
```

- [ ] **Step 3: Verify on GitHub**

```bash
gh repo view MaedaArthur/walkthrough-skill --web
# Expected: Opens browser to repo page showing README
```
