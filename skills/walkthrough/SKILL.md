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

Always ask before starting. Never assume the user wants a full walkthrough, they may just want a quick explanation of one part, or they may want to debug a specific issue.
---

## MODE LEARN `/walkthrough learn`

### Step 1 — Find existing context

Before reading the code, look for:
- **Instruction Manifests:** `README.md`, or `CONTRIBUTING.md` for high-level rules.
- **Agent Manifests:** `CLAUDE.md`, `AGENTS.md` or `AGENT.md` at the project root
- **Project Knowledge:** The `docs/` folder or dedicated `/architecture` files for system design context.
- **Agent Workspaces:** Specialized directories like `.agent/`, `.cursor/rules`, or `.windsurf/` that contain behavioral constraints.
- **Embedded Guidelines:** Header comments in entry files or "Instruction" blocks at the top of main modules.
- **Custom Prompts:** Any `.prompt/` or `.rules` files stored in the root or config directories.

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

### Step 2.5 — Focus and prior knowledge

After presenting the architecture map, ask:

> "Which part do you want to explore first?"

Once the user picks a part, ask two more questions (can be asked together):

> "What specifically do you want to understand about it? And what do you already know about this area?"

**Rules:**
- Wait for the user to choose before asking the follow-up pair.
- Do not skip this step for small projects — knowing what the user already knows always matters.
- Use the answers as the primary input for level calibration in the next step.

---

### Step 3 — Level calibration

Use the answers from Step 2.5 as the **primary source** to determine the user's level:

- Describes what they already know in technical terms → intermediate/advanced
- Says "I don't know anything about this" or uses only behavioral descriptions → beginner
- Mix: knows some concepts but not others → beginner with exposure

**Fine-tune** using vocabulary observed in their follow-up responses throughout the walkthrough (technical terms used correctly = nudge up; confusion with basic terms = nudge down).

Never disclose the calibrated level. Use it only to shape explanations and analogies.

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

### Step 6 — Study guide document

After presenting the study plan, ask:

> "Would you like me to generate a study guide file with everything we covered? It'll be saved to `docs/walkthrough/study-guide.md`."

If the user confirms, create the folder `docs/walkthrough/` (if it doesn't exist) and write `study-guide.md` with the following structure:

```markdown
# Study Guide — [Project Name]
_Session: [date] | Level: [beginner / intermediate / advanced]_

## Project Architecture
[Summary of the architecture map from Step 2 — stack, folder structure, main flow diagram]

## Session Summary
[Synthesis of the walkthrough blocks covered: what was explained, key concepts introduced, analogies used]

## Study Plan
[Expanded version of each topic from Step 5, enriched with session context]

### [Topic] — [Why it matters for this project]
- **Where it appears:** [file, function, or layer in this codebase]
- **Starting point:** [first thing to search or read]
- **Going deeper:** [what to study after the basics, concrete search terms]
- **Goal:** [what understanding this topic looks like in practice]
```

**Level shapes the document:**
- **Beginner:** include analogies used during the session, explain why each topic matters before what to search
- **Intermediate:** focus on how each topic connects to this specific codebase, fewer analogies
- **Advanced:** concise, emphasizes tradeoffs and deeper references, skip foundational explanations

Expand the study plan beyond the 5-topic limit from Step 5 if the session covered more ground. Cover everything relevant — this document is meant to be kept as a reference, not read in one sitting.

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

### Step 5 — Debug guide document

After presenting the focused recommendation, ask:

> "Would you like me to generate a debug guide for this problem? It'll be saved to `docs/walkthrough/debug-[topic].md`."

Use a short slug for `[topic]` based on the problem (e.g., `debug-auth-token-expiry.md`, `debug-null-pointer-api.md`).

If the user confirms, create the folder `docs/walkthrough/` (if it doesn't exist) and write the file with the following structure:

```markdown
# Debug Guide — [Problem Description]
_Session: [date] | Level: [beginner / intermediate / advanced]_

## The Problem
[What was happening, what the user expected, what was observed instead]

## Root Cause
[Why it happened — explained at the user's level]

## Architecture Context
[Only the relevant part: the component, function, or flow where the bug lived]

## Fix Applied
[What was changed or what the user needs to change — concise]

## Study Plan
[Focused on the concepts that caused this bug and how to prevent similar ones]

### [Topic] — [Why this caused the bug]
- **The gap:** [what wasn't understood that led to the bug]
- **Starting point:** [first thing to search or read]
- **Going deeper:** [what to study to truly master this]
- **Goal:** [what it looks like to not make this mistake again]
```

**Level shapes the document:**
- **Beginner:** explain the root cause with an analogy, make the fix feel approachable, study plan is step-by-step
- **Intermediate:** focus on the "what it should do vs. what it did" contrast, study plan emphasizes edge cases
- **Advanced:** root cause as a tradeoff or edge case, study plan goes deep into internals or specs

---

## General rules

- **Never explain everything at once.** Checkpoints exist to save tokens and ensure absorption.
- **Glossary is always inline.** Never create a separate glossary section.
- **No recaps.** Do not repeat what was already covered in prior blocks.
- **Short analogies.** One analogy sentence beats three technical paragraphs for beginners.
- **If no project docs found:** suggest at the end that the user creates documentation — explain that it will speed up future walkthroughs and improve explanation quality.
