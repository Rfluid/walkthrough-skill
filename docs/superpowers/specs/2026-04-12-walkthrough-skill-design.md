# walkthrough-skill — Design Spec
_Date: 2026-04-12_

## Overview

A Claude Code plugin that provides an interactive, adaptive guide for understanding any codebase — regardless of origin. Two explicit modes (`learn`, `debug`) plus a default handler when no mode is specified.

---

## Repository Structure

```
walkthrough-skill/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   └── walkthrough/
│       └── SKILL.md
├── docs/
│   └── superpowers/
│       └── specs/
│           └── 2026-04-12-walkthrough-skill-design.md
└── README.md
```

---

## SKILL.md — Design

### Language
English throughout. Tone: direct, dense, no padding.

### Frontmatter

```yaml
---
name: walkthrough
description: >
  Interactive guide for understanding any codebase — personal projects, open source, legacy code, or anything else.
  Trigger when: user says they don't understand the code, asks how something works,
  wants to learn the architecture, is stuck on a bug, wants to add the project to their portfolio.
  Explicit activation: /walkthrough learn | /walkthrough debug
---
```

### Default Mode Handler (new)

When invoked as `/walkthrough` without a mode:
> Ask: "Do you want to understand the project from scratch (`learn`) or track down a specific problem (`debug`)?"
Wait for response, then route.

---

### MODE LEARN `/walkthrough learn`

**Step 1 — Find existing context**
- Look for `CLAUDE.md`, `README.md`, `docs/`
- If found: use as base, read code only to fill gaps
- If not found: read code directly; suggest creating docs at the end
- **Large project branch:** if 10+ relevant files or multiple architectural layers → generate architecture map first, then ask "Which part do you want to dive into?" before starting blocks. Do not attempt to walk through everything at once.

**Step 2 — Architecture map**
- Detected stack (languages, frameworks, external services)
- Relevant folder structure (ignore node_modules, .git, etc.)
- Main flow in 1 paragraph: what comes in, what happens, what goes out
- Text diagram if multiple layers (fits in ~20 lines)

**Step 3 — Level inference (replaces explicit questionnaire)**
- No calibration questions asked. Level is inferred silently from vocabulary used when the user responds to what they want to explore.
- Technical terms used correctly → intermediate/advanced
- Behavior described without technical terms → beginner
- Mix of both → beginner with some exposure
- Level is never disclosed to the user. Used only to calibrate explanations.

**Step 4 — Walkthrough in blocks**
- Divide by responsibility, not by file
- Per block: name it, explain at inferred level, inline glossary only (never a separate section), end with checkpoint:
  > "Does this make sense? Reply 'yes', 'no', or ask a question before we continue."
- Do not advance without a response. If "no" or a question: different analogy, then checkpoint again.

**Step 5 — Study plan**
- Generated after all selected blocks are done
- Personalized to inferred level and what was explained
- Format: portfolio value (1-2 lines) + max 5 priorities, each with: why it matters in this project + exact search term
- Prioritize by impact on current project, not general knowledge

---

### MODE DEBUG `/walkthrough debug`

**Step 1 — Listen to the problem**
Ask: "Tell me what's happening — what did you expect it to do, and what is it doing instead?"
Infer level from vocabulary. Never disclose the inferred level.

**Step 2 — Locate the problem**
- Search: file/function the user mentioned → flow producing the behavior → external dependencies
- Show WHERE before explaining WHY:
  > "The problem is likely in `[file]`, ~line X, in this function: [short snippet]"

**Step 3 — Explain only what's needed**
- Beginner: analogy + practical impact
- Intermediate: what the code does vs. what it should do
- Advanced: the tradeoff or edge case that caused the problem

**Step 4 — Focused recommendation**
1-2 study topics directly tied to the gap that caused the bug. No full study plan.

---

### General Rules

- Never explain everything at once. Checkpoints exist to save tokens and ensure absorption.
- Inline glossary only. Never a separate glossary section.
- No recaps. Never repeat what was covered in prior blocks.
- Short analogies. One analogy sentence > three technical paragraphs for beginners.
- If no project docs found: suggest creating a README or CLAUDE.md at the end.

---

## plugin.json

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

---

## marketplace.json

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
      "description": "Interactive guide for understanding Claude Code-generated projects — Learn mode and Debug mode",
      "version": "1.0.0",
      "source": "./"
    }
  ]
}
```

---

## README.md

- Language: English
- MIT badge at top
- What it does (2-3 lines)
- Installation section with both commands
- How to use: `learn` and `debug`, each 2-3 lines describing the flow
- How it works: context search, level inference (vocabulary-based), checkpoints, study plan

---

## Git & GitHub

- `git init && git add . && git commit -m "feat: initial walkthrough skill"`
- Create public GitHub repo via `gh repo create MaedaArthur/walkthrough-skill --public`
- Push: `git remote add origin ... && git push -u origin main`
