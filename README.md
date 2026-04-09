# plea

[![Website](https://img.shields.io/badge/website-plea.dev-blue)](https://plea.dev)

Adaptive clinical interview plugin for Claude Code. Uses medical diagnosis methods to extract requirements through binary yes/no questions — one at a time, re-assess after every answer, output a concrete `PLAN.md`.

Not a script, a diagnosis. Each question eliminates a branch of the decision tree.

## Install

```bash
claude plugins marketplace add sliday/claude-plugins && \
claude plugins install plea@sliday
```

## Commands

| Command | What it does |
|---------|-------------|
| `/plea:plea` | Default interview — asks depth, then binary questions |
| `/plea:short` | Quick ~5 questions, high threshold for adding more |
| `/plea:regular` | Standard ~15 questions, moderate threshold |
| `/plea:long` | Thorough 30+ questions, explores all branches |
| `/plea:chat` | Review and refine an existing PLAN.md |

## How It Works

Eight steps, executed in order:

1. **SCAN** — silently reads project context (codebase, existing plans, memory)
2. **PREFILL** — confirms derived facts ("I see Next.js + Prisma + PostgreSQL. Correct?")
3. **DEPTH** — asks interview depth: Quick, Standard, or Thorough
4. **INTERVIEW** — one binary question at a time, re-assess after every answer
5. **FINAL CHECK** — "Anything missing? Anything I should have asked but didn't?"
6. **SYNTHESIZE** — generates `PLAN.md` with files, changes, implementation order
7. **PERSIST** — saves session to `.plea/sessions/` for resumption
8. **OFFER** — asks whether to start execution

### The Clinical Method

The interview loop has two mandatory steps per cycle:

**A. ASK** — pick the single most diagnostic question (the one whose answer changes the plan the most). Each option includes a description of what Yes/No would mean.

**B. RE-ASSESS** — update the full picture before asking the next question:
- What do I now know? What hypotheses are eliminated?
- Which branches are closed? Which new ones opened?
- Show the delta: `+2 unlocked (auth details) · ~13 remaining` or `3 skipped (no DB) · ~9 remaining`

Every 4-5 binary questions, one open-ended question widens the picture.

## Depth Modes

Depth is a sensitivity dial, not a cap. All modes can grow beyond their starting target.

| Mode | Starts at | Adds questions when... |
|------|-----------|----------------------|
| Quick | ~5 | Critical blocker or contradiction discovered |
| Standard | ~15 | Complexity discovered in any axis |
| Thorough | 30+ | Any branch opens, proactively |

## Methodology

Rooted in proven frameworks:
- **Differential diagnosis** — binary questions that halve the solution space
- **TRIZ** — contradiction detection between answers
- **Kepner-Tregoe** — systematic situation appraisal
- **Adaptive clinical interview** — protocol is a tree, not a list

## Files Generated

- `PLAN.md` — execution plan in project root
- `.plea/sessions/*.json` — interview session data (questions, answers, contradictions)
- `CLAUDE.local.md` — key decisions appended (with permission)

## Credits

Concept: [Azamat Sultanov](https://github.com/sultanovazamat) & [Stas Kulesh](https://github.com/sliday)
