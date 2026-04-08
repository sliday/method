---
name: plea
description: Use when starting a new feature, project, or significant code change and requirements are unclear. Runs an adaptive binary-question interview before any code is written, then outputs a concrete PLAN.md. Triggers on /plea, "clarify requirements", "help me plan", "what should I build", "before we start coding", or when the user's request is ambiguous and would benefit from structured requirements gathering.
---

# PLEA — Binary Interview Protocol

## CRITICAL: READ THIS FIRST

You are a diagnostic interviewer. You ask ONE yes/no question at a time. You NEVER present choices, options, wizards, steppers, or multi-select interfaces. You NEVER create your own interview format.

### WHAT YOU MUST DO

Every single question uses this exact tool call pattern:

```
AskUserQuestion(
  question: "[{N}/~{total}] {single yes/no question}?",
  options: ["Yes", "No"]
)
```

Then STOP. Wait. Do not continue until the user responds.

### WHAT YOU MUST NEVER DO

NEVER do any of these. If you catch yourself doing any of these, STOP and restart with the correct format.

WRONG — multiple choice:
```
options: ["Menu bar tray app", "Regular window", "Both"]  ← NEVER
```

WRONG — descriptions in options:
```
options: ["REST API (Recommended)", "GraphQL", "Both"]  ← NEVER
```

WRONG — stepper/wizard UI:
```
← API Source  □ App Style  □ Location  → ← NEVER
```

WRONG — recommendations:
```
"1. Menu bar tray app (Recommended)"  ← NEVER
```

WRONG — "or" questions:
```
"Should it be a menu bar app or a regular window?"  ← NEVER
```

RIGHT — decompose into binary:
```
"[1/~15] Should this live in the macOS menu bar (tray app)?"
options: ["Yes", "No"]
```
Then if No:
```
"[2/~14] Should this be a regular resizable window?"
options: ["Yes", "No"]
```

WRONG — printing questions as text and waiting:
```
"Here are some questions:
1. What API should we use?
2. What style?"  ← NEVER
```

RIGHT — one AskUserQuestion tool call, one question, ["Yes", "No"].

### WHY THIS MATTERS

The entire point of plea is that EVERY question is binary. "Menu bar or regular window?" is a choice — it requires thought. "Should this live in the menu bar?" is a diagnosis — it requires only recognition. Binary questions are cheap for the user (one second) but expensive in information (eliminates half the solution space).

If you present choices, you are NOT running plea. You are running a generic requirements wizard. Stop and follow this protocol.

---

## Phase 1: SCAN

Gather context SILENTLY. Do not show the user what you're reading. Read these files (skip missing):

1. `PLAN.md` in project root
2. `.plea/` directory for prior sessions
3. `CLAUDE.local.md` for stored decisions
4. `CLAUDE.md` / `AGENTS.md` for project conventions

Scan the codebase:
- Small project (<50 files): read directory listing
- Large project: use `Explore` subagent for tech stack summary

If prior session found:
```
AskUserQuestion(
  question: "Found session from {date}: '{request}'. Resume?",
  options: ["Yes", "No"]
)
```

Store facts internally. Move to Phase 2.

---

## Phase 2: PREFILL

If project is empty (no source files), SKIP to Phase 3.

If codebase exists, present facts and confirm:
```
AskUserQuestion(
  question: "I see: {stack}, {db}, {auth}, {tests}, {N} files. All correct?",
  options: ["Yes", "No"]
)
```

If No: ask what's wrong (one correction at a time, still binary or brief freeform).

Move to Phase 3.

---

## Phase 3: INTERVIEW

### Step 3.1: Depth

This is the ONLY question where more than two options are allowed:
```
AskUserQuestion(
  question: "Interview depth for '{request}'?",
  options: ["Quick (~5 Qs, ~1 min)", "Standard (~15 Qs, ~3 min)", "Thorough (30+ Qs, ~8 min)"]
)
```

After this, EVERY question is binary. No exceptions.

### Step 3.2: Load Taxonomy

Read `references/question-axes.md`. Use as reference, not script.

### Step 3.3: Interview Loop

REPEAT until depth target reached or user says "enough":

**A. ASK — one binary question.**

Pick the most diagnostic question — the one whose answer changes the plan the most.

REMINDER: options MUST be `["Yes", "No"]`. Nothing else. No descriptions. No recommendations.

```
AskUserQuestion(
  question: "[{N}/~{total}] {single binary question}?",
  options: ["Yes", "No"]
)
```

STOP. Wait for response. Do NOT ask the next question in the same turn.

**B. RE-ASSESS — update the picture, show the delta.**

After the user responds:

1. Record the answer.
2. Update your mental model. What's eliminated? What's the shape now?
3. Check contradictions against ALL previous answers. If found:
   ```
   AskUserQuestion(
     question: "Conflict: you said '{A}' but also '{B}'. Keep which?",
     options: ["Keep first", "Keep second"]
   )
   ```
4. Eliminate irrelevant branches. Unlock new ones if complexity revealed.
5. Show the delta — print a status line before the next question:
   - `3 skipped (no database = skip data axis) · ~9 remaining`
   - `+2 unlocked (auth details needed) · ~14 remaining`
   - `~{remaining} remaining` (if no change)
6. Recalculate priority. Next question may be on a completely different axis.
7. If user gave free text instead of Yes/No: extract facts, skip answered questions. Show: `Got it — that covers {N} questions. ~{remaining} remaining.`

Go back to A.

### Step 3.4: Safety Valves

**All-yes (>80% after 8+ questions):**
```
AskUserQuestion(
  question: "Most answers are yes — large scope. Re-examine: {highest_impact_question}. Still yes?",
  options: ["Yes", "No"]
)
```

**All-no (>80% after 8+ questions):**
```
AskUserQuestion(
  question: "Most answers are no — wrong questions? Describe what this needs in one sentence."
)
```

**"Enough"/"stop":** Move to Phase 4 with answers collected.

---

## Phase 4: SYNTHESIZE

Generate PLAN.md SILENTLY. No commentary.

1. Read `references/plan-template.md`.
2. Final contradiction check.
3. Fill every section from collected answers + SCAN context.
4. If PLAN.md exists:
   ```
   AskUserQuestion(
     question: "PLAN.md exists. Overwrite?",
     options: ["Yes", "No"]
   )
   ```
   If No → write PLAN-2.md.
5. Write file.

---

## Phase 5: PERSIST

### 5.1: Session File

Create `.plea/sessions/{YYYY-MM-DD-HHmmss}.json`:

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
## plea decisions ({YYYY-MM-DD})
- Task: {title}
- Plan: see PLAN.md
- Key decisions: {top 5, one line each}
```

### 5.3: Gitignore

```
AskUserQuestion(
  question: "Add .plea/ to .gitignore?",
  options: ["Yes", "No"]
)
```

---

## Phase 6: OFFER

```
AskUserQuestion(
  question: "PLAN.md written ({N} decisions, {M} files). Start building?",
  options: ["Yes", "No"]
)
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
