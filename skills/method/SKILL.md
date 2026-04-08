---
name: method
description: Use when starting a new feature, project, or significant code change and requirements are unclear. Runs an adaptive binary-question interview before any code is written, then outputs a concrete PLAN.md. Triggers on /method, "clarify requirements", "help me plan", "what should I build", "before we start coding", or when the user's request is ambiguous and would benefit from structured requirements gathering.
---

# Method — Adaptive Interview for Code Planning

EXECUTE THIS PROTOCOL EXACTLY. Follow each phase in order. Do not skip ahead.

## MANDATORY RULES

1. **ONE QUESTION AT A TIME.** Ask exactly ONE binary question per AskUserQuestion call. Never batch multiple questions.
2. **YES/NO OPTIONS ONLY.** Every AskUserQuestion call MUST use `options: ["Yes", "No"]`. No other options. No freeform text prompts.
3. **USE AskUserQuestion TOOL.** Every question MUST use the tool. Never print questions as text.
4. **NO ASSUMPTIONS.** Never assume answers. Never offer defaults. Never skip questions.
5. **NO TABLES.** No comparison tables, no pros/cons lists, no multi-choice.
6. **WAIT AFTER EACH QUESTION.** Call AskUserQuestion, then STOP. Do not continue until the user responds.
7. **SHOW PROGRESS.** Every question starts with `[{N}/{~total}]` counter.

### Question Format

Every question follows this exact pattern:

```
question: "[{N}/{~total}] {Single binary question}?"
options: ["Yes", "No"]
```

Example:
```
question: "[3/~15] Does this need persistent storage?"
options: ["Yes", "No"]
```

If the user responds with text instead of clicking Yes/No, incorporate the information and adjust. This is valuable signal — don't reject it.

---

## Phase 1: SCAN

Gather context SILENTLY. Do not narrate what you're reading. Read these files (skip missing):

1. `PLAN.md` in project root
2. `.clarify/` directory for prior sessions
3. `CLAUDE.local.md` for stored decisions
4. `CLAUDE.md` / `AGENTS.md` for project conventions

Scan the codebase:
- Small project (<50 files): read directory listing
- Large project: use `Explore` subagent for tech stack summary

If prior session found:
```
question: "Found session from {date}: '{request}'. Resume?"
options: ["Yes", "No"]
```

Store facts internally. Move to Phase 2.

---

## Phase 2: PREFILL

If project is empty (no source files), SKIP to Phase 3.

If codebase exists, present facts and confirm:
```
question: "I see: {stack}, {db}, {auth}, {tests}, {N} files. All correct?"
options: ["Yes", "No"]
```

If No: ask what's wrong via AskUserQuestion (one correction at a time).

Move to Phase 3.

---

## Phase 3: INTERVIEW

One question at a time. Adaptive — each answer shapes the next question.

### Step 3.1: Depth

```
question: "~15 questions for '{request}'. OK, or change depth?"
options: ["Yes", "No"]
```

If No:
```
question: "Shorter interview (~5 questions)?"
options: ["Yes", "No"]
```
If Yes → compact. If No → thorough (30+).

### Step 3.2: Load Taxonomy

Read `references/question-axes.md`. Use it to select questions from axes by priority.

### Step 3.3: Interview Loop

Start a counter at 1. REPEAT until depth target reached or user says "enough":

1. **Pick the single most important unasked question.** Prioritize:
   - Higher-priority axes first (Scope > Data > Auth > API > ...)
   - Questions that eliminate the most branches
   - Questions unlocked by previous answers
   - Skip anything derivable from PREFILL

2. **Ask it:**
   ```
   question: "[{N}/{~total}] {question}?"
   options: ["Yes", "No"]
   ```

3. **STOP. Wait for response.**

4. **Process the answer:**
   - If "Yes" or "No" → record and continue
   - If free text → extract all facts, skip questions already answered by it, acknowledge: "Got it — that also answers {X}. Moving on."
   - Increment counter

5. **After each answer, check for contradictions** against all previous answers. If contradiction found:
   ```
   question: "Conflict: you said '{A}' but also '{B}'. Keep which?"
   options: ["Keep first", "Keep second"]
   ```

6. **Adapt:** If answer eliminates an axis, remove its questions and reduce total estimate. If answer reveals complexity, add deeper questions and increase estimate.

7. **Go to step 1.**

### Step 3.4: Safety Valves

**All-yes (>80% after 8+ questions):**
```
question: "Most answers are yes — large scope. Re-examine the biggest decision: {highest_impact_question}. Still yes?"
options: ["Yes", "No"]
```

**All-no (>80% after 8+ questions):**
```
question: "Most answers are no — am I asking wrong questions? Can you describe what this needs in one sentence?"
```
(This is the ONE exception where freeform text is requested.)

**"Enough" / "stop":** Move to Phase 4 immediately with answers collected so far.

---

## Phase 4: SYNTHESIZE

Generate PLAN.md SILENTLY. No commentary.

1. Read `references/plan-template.md`.
2. Final contradiction check.
3. Fill every section from collected answers + SCAN context.
4. If PLAN.md exists:
   ```
   question: "PLAN.md exists. Overwrite?"
   options: ["Yes", "No"]
   ```
   If No → write PLAN-2.md.
5. Write file.

---

## Phase 5: PERSIST

### 5.1: Session File

Create `.clarify/sessions/{YYYY-MM-DD-HHmmss}.json`:

```json
{
  "request": "",
  "timestamp": "",
  "depth": "",
  "total_questions": 0,
  "questions": [
    { "id": 1, "axis": "", "text": "", "answer": "", "source": "user" }
  ],
  "contradictions": [],
  "plan_file": "PLAN.md"
}
```

### 5.2: CLAUDE.local.md

Append:
```markdown
## method decisions ({YYYY-MM-DD})
- Task: {title}
- Plan: see PLAN.md
- Key decisions: {top 5, one line each}
```

### 5.3: Gitignore

```
question: "Add .clarify/ to .gitignore?"
options: ["Yes", "No"]
```

---

## Phase 6: OFFER

```
question: "PLAN.md written ({N} decisions, {M} files). Start building?"
options: ["Yes", "No"]
```

If Yes: suggest execution skills if available.
If No: end session.

---

## Language

- Detect from user input. Interview in user's language.
- All files stay in English.

## Error Recovery

| Situation | Action |
|-----------|--------|
| No request | Ask: "What do you want to build?" |
| Codebase too large | Top-level only |
| Write fails | Output as text |
| User abandons | Save partial with `"complete": false` |
| Prior session, different request | Ask: "Start fresh?" |
