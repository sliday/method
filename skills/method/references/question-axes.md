# Question Axes — Adaptive Interview Taxonomy

This reference defines the 10 question axes used during the INTERVIEW phase. Each axis targets a distinct architectural concern. Questions are organized by depth level.

## How to Use This File

1. Start with **high-priority axes** (Scope, Data Model, Auth) — these shape everything else
2. Select questions matching the chosen depth: compact = surface only, standard = surface + standard, thorough = all levels
3. **Skip axes** when the skip condition is met (derived from prior answers or codebase scan)
4. **Adapt dynamically**: if an answer reveals unexpected complexity on an axis, promote it to deeper questions even in compact mode
5. Questions are seeds — rephrase them to fit the specific request context. Do not read them verbatim if the wording feels generic.

## Depth Budget

| Mode | Target Questions | Batches |
|------|-----------------|---------|
| compact | ~5 | 1 |
| standard | ~15 | 3 |
| thorough | 30+ | 6+ |

---

## Axis 1: Scope & Boundaries

**Priority:** 1 (always ask first)
**Skip if:** request explicitly defines boundaries (e.g., "add a field to the User model")

### Surface
- Does this change live inside one module/file, or does it cross multiple areas?

### Standard
- Does this require new API endpoints or routes?
- Does this touch existing data models or database schemas?
- Are there other features that depend on the area being changed?

### Deep
- Is this a breaking change for existing consumers (other services, frontend, CLI)?
- Does this need a migration path from the current behavior?
- Should this be behind a feature flag for gradual rollout?

### Adaptation Rules
- If "crosses multiple areas" → increase priority of Dependencies axis
- If "breaking change" → always ask about migration path regardless of depth

---

## Axis 2: Data Model

**Priority:** 2
**Skip if:** request is purely UI/cosmetic with no data changes

### Surface
- Does this need persistent storage (database, file system, external store)?

### Standard
- Does this introduce new tables, collections, or data structures?
- Are there relationships between this data and existing entities?
- What's the expected data volume (tens, thousands, millions of records)?

### Deep
- Should records support soft deletes or is hard delete acceptable?
- Is an audit trail needed (who changed what, when)?
- Does the data have a retention policy or expiration?
- Is the schema likely to evolve frequently after launch?

### Adaptation Rules
- If "no persistent storage" → skip all standard and deep questions on this axis
- If "millions of records" → promote Performance axis to high priority
- If "audit trail needed" → flag as non-functional requirement in PLAN.md

---

## Axis 3: Authentication & Access

**Priority:** 3
**Skip if:** codebase scan shows no auth layer AND request doesn't mention users/access

### Surface
- Does this feature require authentication (knowing who the user is)?

### Standard
- Should it use the existing auth system or need its own?
- Is role-based access control needed (admin vs regular user vs viewer)?

### Deep
- Are there per-resource permissions (user can only edit their own items)?
- Does this need API key or service-to-service authentication?
- Is there a guest/anonymous access mode?
- Does auth state affect what data is visible (row-level security)?

### Adaptation Rules
- If "no auth needed" → skip entire axis
- If "use existing auth" → skip deep questions unless role-based access is yes
- If "per-resource permissions" → flag complexity warning in PLAN.md

---

## Axis 4: API Design

**Priority:** 4
**Skip if:** no API surface (internal library, CLI tool, pure UI change)

### Surface
- Does this expose an API that other code or services will call?

### Standard
- REST, GraphQL, RPC, or something else?
- Does it need pagination for list endpoints?
- What request/response format? (JSON, protobuf, form data)

### Deep
- Should the API be versioned? If so, URL-based or header-based?
- Does it need rate limiting?
- Are there webhook or callback requirements?
- Should responses follow a standard error format (RFC 7807, custom)?

### Adaptation Rules
- If "no API" → skip entire axis
- If "GraphQL" → skip pagination question (handled by GraphQL natively)
- If "rate limiting" → add to non-functional requirements

---

## Axis 5: State & Caching

**Priority:** 5
**Skip if:** stateless request-response only (simple CRUD with no UI)

### Surface
- Does this feature need client-side state management?

### Standard
- Is server-side caching needed for performance?
- Does the UI need real-time updates (WebSocket, SSE, polling)?

### Deep
- Should the UI use optimistic updates (show success before server confirms)?
- What's the cache invalidation strategy?
- Is offline support or local-first behavior needed?
- Does state need to sync across multiple tabs or devices?

### Adaptation Rules
- If "no client-side state" AND "no caching" → skip deep questions
- If "real-time updates" → check for contradiction with "serverless" deployment
- If "offline support" → significant complexity increase; flag in PLAN.md

---

## Axis 6: Error Handling

**Priority:** 6
**Skip if:** compact mode AND no API surface AND no user-facing component

### Surface
- Are there user-facing error messages this feature needs to display?

### Standard
- Should failed operations be retried automatically?
- Is a structured error response format needed?

### Deep
- Does this need a circuit breaker for external service calls?
- Should failed items go to a dead letter queue or retry queue?
- Is partial success acceptable (batch operations)?
- What's the logging/alerting strategy for errors?

### Adaptation Rules
- If "no user-facing errors" AND "no API" → skip entire axis
- If "circuit breaker" → flag as infrastructure requirement
- If "partial success" → important for batch/bulk operations

---

## Axis 7: Testing Strategy

**Priority:** 7
**Skip if:** user explicitly says "no tests" (still flag as tension if reliability matters)

### Surface
- Are tests required for this change?

### Standard
- Unit tests, integration tests, or both?
- Should it use the project's existing test framework and patterns?

### Deep
- Are performance/load tests needed?
- Is contract testing needed (API consumers)?
- What's the target test coverage for this feature?
- Should tests include error/edge case scenarios or just happy path?

### Adaptation Rules
- If "no tests" → skip axis but check for tension with reliability requirements
- If "use existing framework" → note framework name from codebase scan
- If "contract testing" → flag as dependency on API design decisions

---

## Axis 8: Deployment & Infrastructure

**Priority:** 8
**Skip if:** compact mode AND extending existing service with no infra changes

### Surface
- Is this a new service/app or extending an existing one?

### Standard
- Does this need new environment variables or configuration?
- Should it use feature flags for rollout?

### Deep
- Is blue-green or canary deployment needed?
- Does this need its own scaling configuration?
- Is there a rollback plan if deployment fails?
- Does it require new infrastructure (queue, cache layer, CDN)?

### Adaptation Rules
- If "extending existing" → skip most deep questions
- If "new service" → all deep questions become relevant
- If "feature flags" → note in implementation order

---

## Axis 9: Performance & Scale

**Priority:** 9
**Skip if:** compact mode AND low-traffic internal tool

### Surface
- Is this feature latency-sensitive (user waits for response)?

### Standard
- What's the expected request volume or data throughput?
- Are there background jobs or async processing needs?

### Deep
- Does this need connection pooling or resource management?
- Are there specific query optimization concerns?
- Should responses be paginated, streamed, or chunked?
- Is there a performance budget (e.g., < 200ms response time)?

### Adaptation Rules
- If "not latency-sensitive" AND "low volume" → skip deep questions
- If "millions of records" from Data Model axis → auto-promote this axis
- If "performance budget" → add to non-functional requirements

---

## Axis 10: UI/UX

**Priority:** 10 (conditional — only if frontend component exists)
**Skip if:** no frontend/UI component

### Surface
- Does this feature have a frontend/UI component?

### Standard
- New pages/screens or modifications to existing ones?
- Does it need to be mobile-responsive?
- Is there an existing component library or design system to follow?

### Deep
- Are there accessibility requirements (WCAG level)?
- Does it need loading states, skeleton screens, or progress indicators?
- Is there an empty state design needed?
- Does it need to work without JavaScript (SSR/progressive enhancement)?

### Adaptation Rules
- If "no UI component" → skip entire axis
- If "existing component library" → note library name from codebase scan
- If "accessibility requirements" → add to non-functional requirements

---

## Dependencies & Integration (Bonus Axis)

**Priority:** conditional — promote if "crosses multiple areas" from Scope axis
**Skip if:** self-contained single-module change

### Surface
- Does this integrate with any external services or APIs?

### Standard
- Are there third-party libraries or SDKs to integrate?
- Does this depend on other team's work or another feature landing first?

### Deep
- What happens if an external dependency is unavailable?
- Are there API contracts or schemas that both sides must agree on?
- Is there a sandbox/staging environment for the external service?

### Adaptation Rules
- If "no external dependencies" → skip standard and deep
- If "dependency unavailable" → connect to Error Handling axis (circuit breaker)
