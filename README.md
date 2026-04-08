# plea

[![Website](https://img.shields.io/badge/website-plea.dev-blue)](https://plea.dev)

Adaptive clinical interview plugin for Claude Code. Uses medical diagnosis methods to extract requirements through binary yes/no questions — one at a time, re-assess after every answer, output a concrete `PLAN.md`.

Not a script, a diagnosis. Each question eliminates a branch of the decision tree.

## Install

```bash
claude plugins add-marketplace sliday/claude-plugins && \
claude plugins install plea@sliday
```

## Usage

```
/plea:plea I need a REST API for user management
```

## How It Works

Six phases, executed in order:

1. **SCAN** — silently reads project context (codebase, existing plans, memory)
2. **PREFILL** — confirms derived facts ("I see Next.js + Prisma + PostgreSQL. Correct?")
3. **INTERVIEW** — one binary question at a time, re-assess after every answer
4. **SYNTHESIZE** — generates `PLAN.md` with files, changes, implementation order
5. **PERSIST** — saves session to `.plea/sessions/` for resumption
6. **OFFER** — asks whether to start execution

### The Clinical Method

The interview loop has two mandatory steps per cycle:

**A. ASK** — pick the single most diagnostic question (the one whose answer changes the plan the most)

**B. RE-ASSESS** — update the full picture before asking the next question:
- What do I now know? What hypotheses are eliminated?
- Which branches are closed? Which new ones opened?
- What is now the most informative next question?

Questions don't follow a linear checklist. After re-assessment, the next question may jump to a completely different axis.

## Depth Modes

| Mode | Questions | When |
|------|-----------|------|
| compact | ~5 | Small, well-defined changes |
| standard | ~15 | New feature or module |
| thorough | 30+ | New project or architecture decision |

Auto-detected from request complexity. The total is a living number — it changes as branches open and close.

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
