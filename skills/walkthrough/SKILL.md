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
```
[Browser] → [Next.js pages/] → [API routes/] → [Supabase]
                                      ↓
                              [Django backend]
```

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

```
## Your study plan for this project

**Why this matters for your portfolio:** [1-2 sentences about what this project demonstrates]

**Priority 1 — [Topic]** (Level: beginner/intermediate)
Why: appears in [part of the code] and is essential for debugging/evolving the project
Search: "[exact term to look up]"

**Priority 2 — [Topic]** ...

**Priority 3 — [Topic]** ...
```

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
