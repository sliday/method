---
description: Review and refine an existing PLAN.md through conversation
argument-hint: <what to change or review in the plan>
---

Read PLAN.md from the project root. If it doesn't exist, tell the user to run /plea:run first.

If PLAN.md exists, read it and start a conversation about it. The user wants to correct, refine, or extend the plan.

The user's feedback is: $ARGUMENTS

Rules:
- Show the relevant section of PLAN.md being discussed
- Ask binary yes/no questions (options: ["Yes", "No"]) to confirm changes, one at a time
- After each confirmed change, update PLAN.md immediately
- When done, show a summary of what changed
