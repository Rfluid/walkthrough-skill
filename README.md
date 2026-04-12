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
