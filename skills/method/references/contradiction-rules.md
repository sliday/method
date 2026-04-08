# Contradiction Detection Rules

Check for contradictions **after each question batch** and **during synthesis**. Two severity levels:

## Severity Levels

**Hard Contradiction** — logically incompatible answers. Must resolve before proceeding.
**Tension** — unusual combination that works but deserves explicit acknowledgment. Flag but allow.

## How to Present

```
Warning: Contradiction detected

  [Q3] "No persistent storage" — yes
  [Q8] "User accounts with login history" — yes

These are incompatible: login history requires persistent storage.
Which answer should I keep? (enter 3 or 8, or explain)
```

For tensions:
```
Note: Unusual combination detected

  [Q7] "Serverless deployment" — yes
  [Q12] "WebSocket real-time updates" — yes

Serverless + WebSockets is possible but adds complexity (connection management, cold starts).
Acknowledged? [Y] or change an answer? [7/12]
```

---

## Hard Contradictions

| Answer A | Answer B | Why |
|----------|----------|-----|
| No persistent storage | Needs user accounts / login / history / preferences | User data requires storage |
| No authentication | Role-based access control | Roles require knowing who the user is |
| No authentication | Per-resource permissions | Permissions require identity |
| No API surface | Needs rate limiting | Rate limiting applies to API endpoints |
| No API surface | Needs API versioning | Versioning applies to API endpoints |
| Stateless (no client state) | Offline support / local-first | Offline requires local state |
| No backend / frontend-only | Background jobs / async processing | Background jobs need a server |
| Monolith | Independent service deployment | Independent deployment = separate services |
| No database | Audit trail / soft deletes | These are database features |
| No testing | Contract testing / performance testing | These are specific test types |

## Tensions (Flag, Don't Block)

| Answer A | Answer B | Why It's Unusual |
|----------|----------|-----------------|
| Serverless | WebSocket / real-time connections | Serverless has connection limits and cold starts |
| No testing | High reliability requirement | Reliability without tests is risky |
| Monolith | Feature flags for rollout | Feature flags add complexity in monoliths |
| REST API | Real-time updates | REST isn't natively real-time; will need polling or separate WS |
| SQLite / file-based DB | Millions of records | Possible but may hit performance limits |
| No caching | Latency-sensitive + high volume | Performance may suffer without caching |
| Single-page app | SEO requirements | SPAs need SSR or prerendering for SEO |
| No environment config | Multiple deployment targets | Different targets usually need different config |
| GraphQL | Simple CRUD with 2-3 entities | GraphQL may be overkill for simple cases |
| Microservices | Solo developer / small team | Operational overhead may exceed benefits |

## Dynamic Contradictions

Beyond the static table above, check for these patterns:

1. **Scope creep**: early answers suggest small scope, later answers suggest large scope. Flag if the estimated file count doubles from synthesis.

2. **All-yes pattern**: if user answers "yes" to 80%+ of questions, warn:
   ```
   You've answered yes to most questions. This suggests a very large scope.
   Are you sure about all of these, or should we revisit a few key decisions?
   ```
   Then re-ask the 2-3 highest-impact questions.

3. **All-no pattern**: if user answers "no" to 80%+ of questions, ask:
   ```
   Most answers are "no", which suggests I may be asking the wrong questions.
   Can you describe in a sentence or two what this feature actually needs?
   ```

4. **Axis conflict**: if answers within the same axis contradict (e.g., "no database" on surface question but "new tables" on standard question), re-ask the surface question with the contradiction shown.

## When to Check

- **After each batch**: quick scan of new answers against all previous answers
- **During synthesis**: full cross-reference before writing PLAN.md
- **On re-ask**: after user resolves a contradiction, check if the resolution creates new contradictions
