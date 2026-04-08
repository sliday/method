---
name: method
description: Use when starting a new feature, project, or significant code change and requirements are unclear. Runs an adaptive binary-question interview before any code is written, then outputs a concrete PLAN.md. Triggers on /method, "clarify requirements", "help me plan", "what should I build", "before we start coding", or when the user's request is ambiguous and would benefit from structured requirements gathering.
---

# Clarify — Adaptive Interview for Code Planning

EXECUTE THIS PROTOCOL EXACTLY. Do not summarize, do not present tables of options, do not offer defaults. Follow each phase in order.

## MANDATORY RULES — VIOLATION OF ANY RULE MEANS YOU ARE DOING IT WRONG

1. **BINARY QUESTIONS ONLY.** Every question has exactly one answer: yes or no. NEVER present multiple-choice options. NEVER present tables of alternatives. Decompose "A or B?" into TWO separate binary questions: "Is it A?" then "Is it B?"
2. **USE AskUserQuestion TOOL.** Every question batch MUST be delivered via the `AskUserQuestion` tool. Do NOT print questions as regular text output and wait. You MUST call the tool.
3. **BATCHES OF UP TO 5.** Present at most 5 questions per AskUserQuestion call. Never dump all questions at once.
4. **NO ASSUMPTIONS.** Do not assume answers. Do not offer "default plans." Do not say "if you want to skip questions." The entire point is to ASK.
5. **NO TABLES OF OPTIONS.** Do not present comparison tables. Do not list pros/cons of approaches. Ask binary questions instead.
6. **ONE PHASE AT A TIME.** Complete each phase fully before moving to the next. Do not skip phases.
7. **WAIT FOR ANSWERS.** After each AskUserQuestion call, STOP. Do not continue until the user responds.

---

## Phase 1: SCAN

Gather context SILENTLY. Do not show the user what you're reading. Just read these files (skip missing ones):

1. `PLAN.md` in project root
2. `.clarify/` directory for prior sessions
3. `CLAUDE.local.md` for stored decisions
4. `CLAUDE.md` / `AGENTS.md` for project conventions

Then scan the codebase structure:
- If small project (<50 files): read directory listing directly
- If large project: use `Explore` subagent to summarize tech stack, frameworks, file count, test framework, auth, database, deployment target

If `.clarify/sessions/` has a prior session for this project, use AskUserQuestion:
```
question: "Found interrupted session from {date}: '{original_request}'. Resume?"
options: ["Yes, resume", "No, start fresh"]
```

Store all discovered facts internally. Move to Phase 2.

---

## Phase 2: PREFILL

If the project is empty (no source files), SKIP this phase entirely. Go to Phase 3.

If there IS a codebase, present discovered facts and confirm via AskUserQuestion:

```
question: "Based on your project:\n1. Stack: {X}\n2. Database: {X}\n3. Auth: {X}\n4. Testing: {X}\n5. Size: {N} files\n\nAll correct?"
options: ["All correct", "Some corrections needed"]
```

If corrections needed, ask which items via another AskUserQuestion call.

Move to Phase 3.

---

## Phase 3: INTERVIEW

This is the core. You will ask binary questions in batches using AskUserQuestion.

### Step 3.1: Depth

Estimate depth from request complexity, then confirm via AskUserQuestion:

```
question: "Interview depth for '{request}': ~{N} questions. Change?"
options: ["Compact (~5)", "Standard (~15)", "Thorough (30+)", "Accept suggested"]
```

This is the ONE exception where options are allowed — it's a depth setting, not a design question.

### Step 3.2: Load Question Taxonomy

Read `references/question-axes.md` from this skill's directory. Select questions from axes by priority.

### Step 3.3: Interview Loop

REPEAT until depth target reached or user says "enough":

1. **Select up to 5 binary questions** from the highest-priority unasked axes.

2. **Call AskUserQuestion with this EXACT format:**
   ```
   question: "[{answered}/{~total}]\n\n1. [scope] Does this change cross multiple modules?\n2. [data] Does this need persistent storage?\n3. [auth] Does this require authentication?\n4. [api] Does this expose an API?\n5. [testing] Are tests required?\n\nAnswer with: y n y y n (or 1:y 2:n etc)"
   ```
   Do NOT use the `options` parameter for question batches. The user types freeform answers like "y n y y n".

3. **STOP and wait for the user's response.** Do not proceed until you have answers.

4. **Parse the response.** Accept:
   - `y n y y n` — space-separated, matches question order
   - `1:y 2:n 3:y 4:y 5:n` — numbered
   - `yes no yes yes no` — words
   - Free text — extract intent, ask about unclear items

5. **After parsing, check for contradictions** against all previous answers. Load `references/contradiction-rules.md` if needed. If contradiction found, present it via AskUserQuestion:
   ```
   question: "Contradiction:\n  [{id}] '{question_a}' = yes\n  [{id}] '{question_b}' = yes\n\n{explanation}. Which is correct?"
   options: ["Keep #{a}", "Keep #{b}", "Let me explain"]
   ```

6. **Adapt the next batch:** If an answer eliminates an axis, remove its questions. If an answer reveals complexity, add deeper questions for that axis.

7. **Go to step 1** for the next batch.

### Step 3.4: Safety Valves

**All-yes (>80%):** After the batch where this is detected, use AskUserQuestion:
```
question: "You've said yes to most questions — large scope. Let me re-confirm the 2 biggest:\n\n1. {highest_impact_question}?\n2. {second_highest_impact_question}?\n\nStill yes to both?"
options: ["Yes to both", "Let me reconsider"]
```

**All-no (>80%):** Use AskUserQuestion:
```
question: "Most answers are no — I may be asking wrong questions. In a sentence, what does this feature actually need?"
```

**"Enough" / "stop":** Immediately move to Phase 4 with answers collected so far.

---

## Phase 4: SYNTHESIZE

Generate PLAN.md. Do this SILENTLY — no commentary, just write the file.

1. Read `references/plan-template.md` from this skill's directory.
2. Run final contradiction check across all answers.
3. Fill every section of the template using collected answers and codebase context from SCAN.
4. If PLAN.md already exists, use AskUserQuestion:
   ```
   question: "PLAN.md already exists. Overwrite or create PLAN-2.md?"
   options: ["Overwrite", "Create PLAN-2.md"]
   ```
5. Write the file using the Write tool.

---

## Phase 5: PERSIST

### 5.1: Session File

Create `.clarify/sessions/` directory. Write `{YYYY-MM-DD-HHmmss}.json`:

```json
{
  "request": "{original request}",
  "timestamp": "{ISO 8601}",
  "depth": "{compact|standard|thorough}",
  "total_questions": 0,
  "total_batches": 0,
  "codebase_summary": "{from SCAN}",
  "prefill": { "facts": [], "corrections": [] },
  "questions": [
    { "id": 1, "batch": 1, "axis": "", "text": "", "answer": "", "source": "user" }
  ],
  "contradictions": [],
  "plan_file": "PLAN.md"
}
```

### 5.2: CLAUDE.local.md

Append key decisions:

```markdown
## clarify decisions ({YYYY-MM-DD})
- Task: {short title}
- Plan: see PLAN.md
- Key decisions:
  - {top 5 most impactful decisions, one line each}
```

### 5.3: Gitignore

If `.clarify/` not in `.gitignore`, use AskUserQuestion:
```
question: "Add .clarify/ to .gitignore? (contains local session data)"
options: ["Yes", "No"]
```

---

## Phase 6: OFFER

Use AskUserQuestion:
```
question: "PLAN.md written ({N} decisions, {M} files planned). Start execution?"
options: ["Yes, start building", "No, I'll review first"]
```

If yes: suggest using execution skills if available (superpowers:executing-plans, superpowers:using-git-worktrees).
If no: end session.

---

## Language

- Detect user's language from input. Conduct interview in that language.
- ALL files (PLAN.md, JSON, CLAUDE.local.md) stay in English.
- Axis labels always English.

---

## Error Recovery

| Situation | Action |
|-----------|--------|
| No request provided | AskUserQuestion: "What do you want to build? (one sentence)" |
| Codebase too large | Scan top-level only, note in PLAN.md |
| Write fails | Output PLAN.md content as text |
| User abandons | Save partial session with `"complete": false` |
| Prior session, different request | AskUserQuestion: "Found prior session for '{X}'. Start fresh?" |
