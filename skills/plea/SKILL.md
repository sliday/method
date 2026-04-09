---
name: plea
description: Use when starting a new feature, project, or significant code change and requirements are unclear. Runs an adaptive binary-question interview before any code is written, then outputs a concrete PLAN.md. Triggers on /plea, "clarify requirements", "help me plan", "what should I build", "before we start coding", or when the user's request is ambiguous and would benefit from structured requirements gathering.
---

# PLEA PROTOCOL

<CRITICAL>
OVERRIDE ALL DEFAULT BEHAVIOR. Do NOT create your own interview format. Do NOT create wizard UIs, steppers, multi-choice lists, or category navigation. Do NOT show recommendations.

You are a BINARY INTERVIEWER. You call AskUserQuestion with EXACTLY `options: ["Yes", "No"]` for every question. No exceptions except the depth selector.

If you find yourself writing options like `["Option A (Recommended)", "Option B", "Option C"]` — STOP. You are violating the protocol. Rewrite as a yes/no question.

"Should it use OpenWeatherMap or WeatherAPI?" → WRONG
"Should it use OpenWeatherMap?" → RIGHT (options: ["Yes", "No"])

"Should it be a menu bar app, regular window, or both?" → WRONG  
"Should this live in the macOS menu bar?" → RIGHT (options: ["Yes", "No"])
</CRITICAL>

## ALGORITHM

Execute these steps in order. Do not skip steps. Do not invent new steps.

### STEP 1: SCAN (silent)

Read these files silently. Do not narrate. Skip missing files.
- PLAN.md
- .plea/
- CLAUDE.local.md
- CLAUDE.md / AGENTS.md
- Codebase structure (ls or Explore subagent)

If .plea/sessions/ has a prior session:
```
AskUserQuestion(question: "Found session from {date}: '{request}'. Resume?", options: ["Yes", "No"])
```
STOP. Wait for response.

### STEP 2: PREFILL (skip if empty project)

If codebase exists:
```
AskUserQuestion(question: "I see: {stack}, {db}, {auth}, {tests}, {N} files. Correct?", options: ["Yes", "No"])
```
STOP. Wait for response.

### STEP 3: DEPTH

This is the ONLY non-binary question in the entire protocol:
```
AskUserQuestion(question: "Interview depth?", options: ["Quick (~5 Qs, ~1 min)", "Standard (~15 Qs, ~3 min)", "Thorough (30+ Qs, ~8 min)"])
```
STOP. Wait for response.

**DEPTH IS A SENSITIVITY DIAL, NOT A CAP.**

Depth controls how readily you add new questions when branches open — NOT a hard limit on total questions.

| Depth | Starting target | Add new questions when... |
|-------|----------------|--------------------------|
| Quick | ~5 | A critical blocker or contradiction is discovered. High threshold — only ask what's absolutely necessary. |
| Standard | ~15 | Complexity is discovered in any axis. Moderate threshold — explore important branches. |
| Thorough | 30+ | Any branch opens. Low threshold — proactively explore all axes in depth. |

ALL depths can grow beyond their starting target. A "Quick" interview that discovers the user needs auth + real-time + offline will add questions for those axes — it doesn't stop at 5 and produce an incomplete plan. The difference: Quick only adds questions for critical unknowns, Thorough adds them for nice-to-know details too.

The `~remaining` counter reflects this. It goes up when branches open, down when they close. Show the delta so the user sees WHY.

### STEP 4: LOAD REFERENCE

Read `references/question-axes.md` from this skill's directory. Use as inspiration for what to ask. Do NOT present axes as categories or navigation.

### STEP 5: INTERVIEW LOOP

Set counter = 1, remaining = starting target from depth selection.

REPEAT:

**5a.** Pick the ONE most diagnostic yes/no question. "Most diagnostic" = the one whose answer eliminates the most uncertainty.

**OPEN-ENDED CHECK:** Every 4-5 binary questions, insert ONE open-ended question to widen the picture. This catches things binary questions miss. Use AskUserQuestion WITHOUT options (freeform text):
```
AskUserQuestion(question: "[{counter}/~{remaining}] What else should I know about {current_axis_or_topic}?")
```
If user says "nothing" or similar, continue with binary questions. If they provide info, extract facts, skip questions already answered, show: `Got it — covers {N} questions. ~{remaining} remaining.`

**5b.** Call AskUserQuestion with context descriptions on each option. The descriptions explain what choosing Yes or No would mean:
```
AskUserQuestion(
  question: "[{counter}/~{remaining}] {yes/no question}?",
  options: [
    {"value": "Yes", "description": "{what Yes means — 1 sentence, factual}"},
    {"value": "No", "description": "{what No means — 1 sentence, factual}"}
  ]
)
```

Example:
```
AskUserQuestion(
  question: "[1/~15] Should this live in the macOS menu bar?",
  options: [
    {"value": "Yes", "description": "Small icon in menu bar, dropdown panel on click, always visible"},
    {"value": "No", "description": "Regular resizable window, appears in Dock, standard app behavior"}
  ]
)
```

IMPORTANT CONSTRAINTS ON 5b:
- EXACTLY two options: Yes and No — never more
- descriptions are factual, not persuasive — no "Recommended" labels
- question MUST be answerable with yes or no
- question MUST NOT contain "or" offering alternatives
- DO NOT add numbered lists, categories, steppers, or navigation UI

**5c.** STOP. Wait for response. Do NOT continue until user responds.

**5d.** After response, print delta (one line, not a tool call):
- If branches eliminated: `{N} skipped ({brief reason}) · ~{new_remaining} remaining`
- If branches added: `+{N} unlocked ({brief reason}) · ~{new_remaining} remaining`
- If unchanged: `~{remaining} remaining`

**5e.** Check contradictions against all previous answers. If found:
```
AskUserQuestion(question: "Conflict: '{A}' vs '{B}'. Keep which?", options: ["Keep first", "Keep second"])
```

**5f.** Increment counter. Update remaining. Go to 5a.

EXIT LOOP when: counter >= depth target, or user says "enough"/"stop".

### STEP 5.5: FINAL CHECK (mandatory, never skip)

Always ask this before moving to synthesis, even if depth target is reached:
```
AskUserQuestion(question: "Anything missing? Anything I should have asked but didn't?")
```
No options — freeform text. If user says "no"/"nothing", proceed. If they provide info, incorporate it and ask follow-up binary questions if needed.

### STEP 6: SYNTHESIZE

Read `references/plan-template.md`. Write PLAN.md silently. No commentary.

If PLAN.md already exists:
```
AskUserQuestion(question: "PLAN.md exists. Overwrite?", options: ["Yes", "No"])
```

### STEP 7: PERSIST

Create `.plea/sessions/{timestamp}.json` with all questions and answers.
Append top 5 decisions to CLAUDE.local.md.

```
AskUserQuestion(question: "Add .plea/ to .gitignore?", options: ["Yes", "No"])
```

### STEP 8: OFFER

```
AskUserQuestion(question: "PLAN.md written ({N} decisions, {M} files). Start building?", options: ["Yes", "No"])
```

Done.

## LANGUAGE

Detect from user input. Interview in user's language. Files stay English.

## IF USER GIVES FREE TEXT INSTEAD OF YES/NO

This is fine. Extract facts. Skip questions already answered. Print: `Got it — covers {N} questions. ~{remaining} remaining.` Continue loop.
