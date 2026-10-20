---
name: learning-planner
description: Use when the user talks about learning plans, study schedules, or wants to plan/organize/review their learning progress. Triggers on phrases like "学习计划", "规划学习", "下一阶段", "帮我安排", "学完了", "阶段结束", "总结进度", "plan my study", "learning roadmap", "what should I learn next", "review my progress", "course planning", "skill development plan", "arrange my learning", "学习清单", "学习路线", "学习总结", "阶段规划", organize study, study tracker, progress review. Use this skill whenever a user wants to create, update, or complete a structured learning phase WITH checkbox task tracking and progress summarization.
allowed-tools: [Read, Write, Edit, Glob, Grep, Bash, PowerShell]
---

# Learning Planner Skill

A long-running learning companion that helps users plan, track, and review their self-directed learning journey in structured phases. Works seamlessly with Obsidian-flavored Markdown.

## Core Rules

### File Structure

The skill manages a learning project directory with this layout:

```
<learning-project>/
├── CLAUDE.md                    # Project rules (written during init)
├── learning-goals.md            # Long-term + short-term goals (user-editable)
├── learning-log.md              # Cumulative phase summaries (primary context for planning)
│
├── YYYYMMDD-<topic>/            # One phase = one short-term learning goal
│   ├── phase-plan.md            # Phase overview + checkbox task list + self-check questions
│   ├── 01_<item>.md             # Decomposed learning content
│   ├── 02_<item>.md
│   └── ...                      # User may freely add files here
│
└── YYYYMMDD-<topic>/
    └── ...
```

**Naming rules:**
- Phase folders: `YYYYMMDD-<topic-slug>` (e.g., `20260517-machine-learning-basics`)
- Sub-files inside a phase: numbered `01_<item-slug>.md`, `02_<item-slug>.md`, etc.
- All preset filenames use English (for international compatibility)
- Phase folders and sub-files support any language for `<topic>` and `<item>` parts

### Obsidian Compatibility

- Use Obsidian-flavored markdown checkboxes: `- [ ]` (incomplete) / `- [x]` (complete)
- Use `[[wikilinks]]` for cross-references when appropriate
- Keep file content well-structured with `##` headings for Obsidian outline view
- Math formulas use `$$` (display) or `$` (inline) LaTeX

### Key Design Principles

**Token economy via `learning-log.md`:**
When planning a new phase, read `learning-log.md` (compressed history) + `learning-goals.md` + the most recent phase folder. Do NOT scan all historical phases unless the user explicitly requests a comprehensive review.

**Full scan only at phase completion:**
When ending a phase, scan the ENTIRE current phase folder — not just `phase-plan.md` but ALL `.md` files the user may have annotated, plus any new files they added. This captures user additions, comments, and questions.

---

## Intent Recognition

The user will speak in natural language. Map their intent to the correct flow:

| User might say (examples) | Intent → Flow |
|---|---|
| "帮我规划学习", "plan my study", "下一步学什么", "what should I learn next", "give me a learning task", "安排一下", "create a study plan" | **Flow 2: Plan new phase** |
| "我学完了", "这个阶段做完了", "I finished this phase", "阶段结束", "done with this", "完成了", "可以做总结了" | **Flow 3: End phase & summarize** |
| "初始化", "setup learning project", "init", "开始使用学习规划", "first time setup", "帮我建立学习档案" | **Flow 1: Initialize** |
| "回顾进度", "review my progress", "我之前学了什么", "show my learning log" | Read `learning-log.md` and report |
| "修改目标", "update goals", "调整方向", "change my learning direction" | Update `learning-goals.md` |

**When intent is ambiguous**, ask the user to clarify. Do NOT guess. A single follow-up question is enough.

**Recognizing phase completion even when implicit:**
Users often declare completion without formal language. Phrases like "终于啃完了", "这个搞定了", "花了X周总算学完了", "I finally finished this" should trigger Flow 3. When in doubt between casual chat and actual completion, ask: "听起来这个阶段结束了，需要我做阶段总结吗？"

**Recognizing planning requests:**
Phrases like "接下来学什么好", "帮我看看下一步", "推荐一下后续方向", "what should I focus on next" should trigger Flow 2.

---

## Flow 1: Initialize Project

Triggered on first use or when user explicitly requests initialization.

### Step 1: Check existing state
Check if `CLAUDE.md`, `learning-goals.md`, `learning-log.md` already exist in the current directory. If all three exist, inform the user the project is already initialized and ask if they want to re-initialize (will overwrite).

### Step 2: Gather user background
Ask questions one at a time (do not bombard):

1. **学习方向 / Learning direction** — What field or subject are you focusing on? (e.g., "AI-assisted protein engineering", "full-stack web development", "Japanese language")

2. **当前水平 / Current level** — What's your background? (e.g., "PhD student strong in biology, weak in programming", "self-taught programmer with 2 years experience", "complete beginner")

3. **可用时间 / Available time** — How much time can you commit per day or per week? (e.g., "1-2 hours/day on weekdays", "10 hours/week")

4. **长期目标 / Long-term goals** — What do you ultimately want to achieve? This is broad and directional. (e.g., "publish a paper using AI in my domain", "switch career to ML engineer", "pass JLPT N2")

### Step 3: Create files

**CLAUDE.md** — project rules for Claude Code:
```markdown
# Learning Project Configuration

This directory is managed by the learning-planner skill. File structure:

- `learning-goals.md` — Long-term and short-term learning goals (feel free to edit directly)
- `learning-log.md` — Cumulative phase summaries (managed by Claude Code)
- `YYYYMMDD-<topic>/` — One folder per learning phase
  - `phase-plan.md` — Phase plan with checkbox task list
  - `0X_<item>.md` — Individual learning content files

## Rules for Claude Code

1. When planning a new phase, read `learning-goals.md` + `learning-log.md` + the most recent phase folder
2. When ending a phase, scan ALL files in the current phase folder, then append a summary to `learning-log.md`
3. Always use `- [ ]` / `- [x]` checkbox format for task lists (Obsidian compatible)
4. Phase folder naming: `YYYYMMDD-<topic-slug>`
5. Preset filenames (`phase-plan.md`, `learning-goals.md`, `learning-log.md`) use English; content files can use any language
```

**learning-goals.md** — user's goals:
```markdown
# Learning Goals

## Long-term Goals
<!-- Broad, directional goals. Add or modify freely. -->

- [ ] [Fill in your long-term goal 1]
- [ ] [Fill in your long-term goal 2]

## Current Short-term Goals
<!-- Current phase and upcoming phases. Managed by you and Claude Code. -->

### Active
- [ ] [Current phase topic] → see `YYYYMMDD-<topic>/`

### Planned
<!-- Upcoming phases will be listed here -->
```

**learning-log.md** — empty archive:
```markdown
# Learning Log

Cumulative summaries of completed learning phases. Managed by Claude Code during phase completion (Flow 3).

---
<!-- Phase summaries will be appended below -->
```

### Step 4: Offer first planning
Ask: "项目已初始化。需要我现在帮你规划第一个学习阶段吗？"
- Yes → proceed to Flow 2
- No → tell user they can edit `learning-goals.md` first, then come back anytime

---

## Flow 2: Plan New Phase

Triggered when user requests a new learning phase.

### Step 1: Read context
Read these files to understand current state:
1. `learning-goals.md` — long-term direction, active/planned short-term goals
2. `learning-log.md` — what was covered, what was left unfinished, user's expressed interests
3. The most recent phase folder (ALL `.md` files within) — recent details, user annotations

### Step 2: Clarify scope (if not specified by user)
Ask only what's not clear from the user's request:

- **Topic/focus**: What specific topic or skill should this phase cover? If user didn't specify, suggest based on `learning-goals.md` and `learning-log.md` history.
- **Scale**: How large should this phase be? If user didn't specify, ask:
  - **S**: Small (3-5 days, 2-3 tasks) — focused deep-dive on one narrow topic
  - **M**: Medium (1-2 weeks, 3-6 tasks) — a sub-topic with progressive tasks
  - **L**: Large (2-4 weeks, 5-10 tasks) — a complete knowledge module
- **Special requirements**: Any constraints, preferences, or specific formats?

### Step 3: Generate phase
Create a new folder `YYYYMMDD-<topic>/` with:

**phase-plan.md** template:
```markdown
# Phase Plan: <Topic>

> **Period:** YYYY-MM-DD to YYYY-MM-DD (estimated)
> **Scale:** <S/M/L>
> **Status:** Active

## Overview
<!-- 2-3 sentences: what this phase covers and why -->

## Task List
<!-- Check off items as you complete them. Add notes, links, questions inline. -->

### Core Tasks
- [ ] **Task 1:** <description>
- [ ] **Task 2:** <description>
- [ ] **Task 3:** <description>

### Optional / Stretch
- [ ] **Bonus:** <description>

## Self-Check Questions
<!-- Try to answer these before marking the phase complete -->

1. [ ] <question 1>
2. [ ] <question 2>
3. [ ] <question 3>

## Resources
<!-- Key references, papers, tutorials, tools -->

- 
- 

## Notes
<!-- Your reflections, difficulties, discoveries during this phase -->

---

*Phase plan created by Claude Code on YYYY-MM-DD. Edit freely.*
```

**0X_<item>.md** files — one per core task:
```markdown
# 0X · <Item Title>

> **Goal:** <what the user should be able to do/understand after this>
> **Estimated time:** <hours/days>
> **Related tasks:** [[../phase-plan.md]]

---

## Content

<learning content here, structured with ## headings for Obsidian outline>

---

## Self-Check

- [ ] <key concept or skill to verify>

## My Notes
<!-- Add your own notes, questions, links, code snippets here -->

```

### Step 4: Update learning-goals.md
Update the `## Current Short-term Goals` section:
- Set the new phase as `### Active`
- Move any previous phase from Active to a new `### Completed` subsection (or let Flow 3 handle this)

### Step 5: Summarize for user
Present a concise overview of what was created and the key tasks.

---

## Flow 3: End Phase & Summarize

Triggered when user indicates they've completed (or want to close) the current phase.

### Step 1: Identify current phase
Determine which phase the user is referring to:
- If they mention a specific phase, use that
- Otherwise, find the most recent phase folder (youngest `YYYYMMDD-<topic>/`)
- If ambiguous, ask

### Step 2: Full scan of phase folder
Read EVERY file in the current phase folder, including:
- `phase-plan.md` — checkbox status, user annotations
- All `0X_<item>.md` files — user's notes, comments, questions, additions
- Any other `.md` or text files the user created

Pay special attention to:
- **Unchecked items** → record as "incomplete / carried forward"
- **User comments/questions** → these reveal difficulties or interests
- **User-added links/references** → indicates emerging interests
- **New files** the user created → may contain important insights
- **Patterns in user annotations** → e.g., consistently struggling with math means next phase should address fundamentals

### Step 3: Generate phase summary
Append to `learning-log.md`:

```markdown
## Phase: <topic> (`YYYYMMDD-<topic>/`)

**Completed:** YYYY-MM-DD
**Duration:** <actual time spent, if known>

### Completion Status
- [x] Task 1 — done
- [x] Task 2 — done
- [ ] Task 3 — *not completed, carried forward*

### Key Takeaways
<!-- 2-4 bullet points capturing the most important things learned -->

- 

### Challenges & Open Questions
<!-- Difficulties the user encountered, questions they raised -->

- 

### Emerging Interests
<!-- New directions or topics the user showed interest in -->

- 

### Recommendations for Next Phase
<!-- Based on what was learned and what's left, what should the next phase cover? -->

1. 
2. 

---
```

### Step 4: Update learning-goals.md
- Move the completed phase from `### Active` to `### Completed`
- Add any new short-term goals identified from emerging interests

### Step 5: Offer next phase
Ask: "阶段总结已完成。需要规划下一个学习阶段吗？"
- Yes → proceed to Flow 2
- No → done

---

## Edge Cases

### No history (first phase)
When planning the first phase after init, there is no `learning-log.md` history and no previous phase folder. In this case:
- Read `learning-goals.md` for direction
- Read `CLAUDE.md` for any additional context the user added
- Ask the user about their immediate learning priority
- Generate the first phase based on the user's stated goals and current level

### User jumps between phases
If the user wants to work on a non-latest phase (e.g., revisiting an old topic):
- Treat the specified phase as the "current" one for Flow 3 purposes
- In `learning-log.md`, note that this is a return to an earlier topic
- Do NOT assume the latest folder is the active one — always confirm

### User changes goals mid-phase
If the user updates `learning-goals.md` during a phase:
- Acknowledge the change
- Ask if the current phase should be adjusted or completed early
- Do NOT silently redirect — the user may want to finish the current phase as-is

### User wants to restart a failed/incomplete phase
- Review the original phase folder to understand what was planned
- Ask what went wrong (too much? too hard? lost interest? time constraints?)
- Adjust the new phase accordingly (smaller scale, different approach, prerequisites)

### Multiple phase folders exist, none clearly "active"
- Ask the user which one is current
- If they're not sure, list the folders with brief descriptions from their `phase-plan.md`

### User asks for a comprehensive retrospective
- Read ALL phase folders and `learning-log.md`
- Generate a big-picture summary covering all completed phases
- This is token-expensive but explicitly requested, so it's fine

### User did NOT complete most tasks
- Don't judge or push. Simply record what was and wasn't done
- Ask if unfinished tasks should be carried to the next phase or dropped
- Consider whether the phase was too ambitious → suggest smaller scale next time

### Phase folder is empty or corrupted
- If `phase-plan.md` doesn't exist, ask the user what happened
- Reconstruct what you can from `learning-log.md` context
- Generate a minimal phase-plan.md if needed

---

## Templates Reference

### phase-plan.md
See Flow 2, Step 3 for the full template.

### learning-log.md entry
See Flow 3, Step 3 for the entry template.

### learning-goals.md
See Flow 1, Step 3 for the initial template.

---

## Quick Reference for Claude Code

| Situation | Action |
|---|---|
| User says "plan/规划/安排" | → Flow 2 |
| User says "done/完了/结束/完成了" | → Flow 3 |
| User says "init/初始化/setup" | → Flow 1 |
| User says "review/回顾/进度" | → Read learning-log.md, report |
| User says "update goals/修改目标" | → Edit learning-goals.md |
| Intent unclear | → Ask one clarifying question |
| No prior phases exist | → Use learning-goals.md only, ask user for priority |
| Multiple incomplete phases | → Ask user which is current |
