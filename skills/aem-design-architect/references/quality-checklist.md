# AEM Architecture Design Review Checklist

Quality gates and scoring criteria for AEM architecture documents.

---

## Scoring Overview

| Area | Max Score | Critical Threshold | Weight |
|------|-----------|-------------------|--------|
| 1. Context & Goals | 10 | 6 | Medium |
| 2. Boundaries & Responsibilities | 15 | 13 | **Critical** |
| 3. Content Design | 10 | 8 | Medium |
| 4. Key Flows | 10 | 7 | High |
| 5. Interfaces & Contracts | 15 | 10 | High |
| 6. Caching & Performance | 15 | 12 | **Critical** |
| 7. Reliability | 10 | 6 | Medium |
| 8. Security | 5 | 3 | Medium |
| 9. Operability | 5 | 4 | High |
| 10. Risks & Trade-offs | 5 | 3 | Medium |
| **Total** | **100** | | |

---

## Mandatory Pass Criteria

The following must be met regardless of total score:

| Criterion | Requirement |
|-----------|-------------|
| Caching strategy | Explicit and complete |
| Boundaries defined | AEM vs External clear |
| Cache invalidation | Strategy documented |
| Performance targets | Specified with rationale |
| Operability | Cloud Manager compatible |

**Failure to meet any mandatory criterion = architecture is invalid.**

---

## Detailed Scoring Rubric

### 1. Context & Goals (10 points)

| Criterion | Points | Check |
|-----------|--------|-------|
| Problem statement clear | 2 | [ ] |
| Users identified (authors, end users, systems) | 2 | [ ] |
| Non-goals explicitly stated | 2 | [ ] |
| System classification stated (content-heavy/experience/headless/hybrid) | 2 | [ ] |
| Success criteria defined | 2 | [ ] |

**Questions to validate:**
- Is it clear what problem this solves?
- Can I identify who uses this and why?
- Are non-goals preventing scope creep?

---

### 2. Boundaries & Responsibilities (15 points) - CRITICAL

| Criterion | Points | Check |
|-----------|--------|-------|
| AEM responsibilities explicitly listed | 3 | [ ] |
| Non-AEM responsibilities justified | 3 | [ ] |
| Authoring vs Delivery separation clear | 3 | [ ] |
| Backend vs AEM boundary clear | 3 | [ ] |
| Dependency direction rules stated | 3 | [ ] |

**Red flags:**
- [ ] AEM treated as generic app server
- [ ] Business logic in AEM without justification
- [ ] No clear separation of concerns
- [ ] Backend calling into AEM (anti-pattern)

**Questions to validate:**
- Can I explain to a new developer what goes where?
- Is AEM doing too much?
- Is the responsibility split justified?

---

### 3. Content Design (10 points)

| Criterion | Points | Check |
|-----------|--------|-------|
| Content types defined | 2 | [ ] |
| Content hierarchy documented | 2 | [ ] |
| Localization strategy clear | 2 | [ ] |
| Multi-site approach defined (if applicable) | 2 | [ ] |
| Content lifecycle documented | 2 | [ ] |

**Red flags:**
- [ ] Deep content hierarchies (>5 levels)
- [ ] Author-unfriendly structures
- [ ] Unclear ownership of content types
- [ ] No localization consideration for multi-language

---

### 4. Key Flows (10 points)

| Criterion | Points | Check |
|-----------|--------|-------|
| Authoring flow documented | 2 | [ ] |
| Delivery flow documented | 2 | [ ] |
| Cache hit path shown | 2 | [ ] |
| Cache miss path shown | 2 | [ ] |
| Failure scenarios covered | 2 | [ ] |

**Must have:**
- At least one flow showing cache invalidation
- Clear request/response path diagrams

---

### 5. Interfaces & Contracts (15 points)

| Criterion | Points | Check |
|-----------|--------|-------|
| AEM-exposed interfaces listed | 3 | [ ] |
| AEM-consumed interfaces listed | 3 | [ ] |
| Content models treated as contracts | 3 | [ ] |
| Integration patterns specified | 3 | [ ] |
| Failure handling for integrations | 3 | [ ] |

**Questions to validate:**
- Are Sling Model contracts clear (conceptually)?
- Do I know what happens when external APIs fail?
- Are integration patterns appropriate?

---

### 6. Caching & Performance (15 points) - CRITICAL

| Criterion | Points | Check |
|-----------|--------|-------|
| Performance targets specified (TTFB, LCP) | 3 | [ ] |
| CDN caching rules defined | 3 | [ ] |
| Dispatcher caching rules defined | 3 | [ ] |
| Cache key strategy documented | 3 | [ ] |
| Invalidation strategy complete | 3 | [ ] |

**Hard requirements:**
- [ ] Cache hit ratio target stated
- [ ] Invalidation trigger → scope → method documented
- [ ] statfileslevel justified
- [ ] TTLs for each content type

**Red flags:**
- [ ] Vague caching ("we'll cache at Dispatcher")
- [ ] No invalidation strategy
- [ ] Personalization breaking cache with no mitigation
- [ ] Missing CDN consideration

---

### 7. Reliability (10 points)

| Criterion | Points | Check |
|-----------|--------|-------|
| AEM Publish failure handling | 2 | [ ] |
| External system failure handling | 2 | [ ] |
| Blast radius defined | 2 | [ ] |
| Graceful degradation approach | 2 | [ ] |
| Recovery procedures outlined | 2 | [ ] |

---

### 8. Security (5 points)

| Criterion | Points | Check |
|-----------|--------|-------|
| Authentication approach | 1 | [ ] |
| Authorization enforcement | 1 | [ ] |
| Service user patterns | 1 | [ ] |
| Data protection (transit/rest) | 1 | [ ] |
| Sensitive data handling | 1 | [ ] |

---

### 9. Operability (5 points) - HIGH PRIORITY

| Criterion | Points | Check |
|-----------|--------|-------|
| Cloud Manager deployment documented | 1 | [ ] |
| Monitoring signals defined | 1 | [ ] |
| Rollback strategy (code + content) | 1 | [ ] |
| Observability approach | 1 | [ ] |
| Feature flags (if applicable) | 1 | [ ] |

**Must assume:**
- No shell access
- Immutable infrastructure
- Pipeline-driven configuration

---

### 10. Risks & Trade-offs (5 points)

| Criterion | Points | Check |
|-----------|--------|-------|
| Key trade-offs documented | 1 | [ ] |
| Alternatives considered | 1 | [ ] |
| Risks identified with mitigation | 1 | [ ] |
| AEM anti-patterns avoided | 1 | [ ] |
| Open decisions tracked | 1 | [ ] |

**Must include:**
- What NOT to do in AEM and why
- Cache vs freshness trade-off explicitly stated

---

## AEM-Specific Red Flags

Automatic point deductions:

| Red Flag | Deduction |
|----------|-----------|
| AEM as backend/app server | -10 |
| No caching strategy | -15 (invalid) |
| Heavy server-side logic without justification | -5 |
| Personalization breaking cache with no solution | -5 |
| No performance targets | -5 |
| Ignoring Cloud Service constraints | -5 |

---

## Review Summary Template

```markdown
## Design Review Summary

### Scorecard

| Area | Score | Threshold | Status |
|------|-------|-----------|--------|
| Context & Goals | /10 | 6 | ✓/✗ |
| Boundaries & Responsibilities | /15 | 13 | ✓/✗ |
| Content Design | /10 | 8 | ✓/✗ |
| Key Flows | /10 | 7 | ✓/✗ |
| Interfaces & Contracts | /15 | 10 | ✓/✗ |
| Caching & Performance | /15 | 12 | ✓/✗ |
| Reliability | /10 | 6 | ✓/✗ |
| Security | /5 | 3 | ✓/✗ |
| Operability | /5 | 4 | ✓/✗ |
| Risks & Trade-offs | /5 | 3 | ✓/✗ |
| **Total** | **/100** | | |

### Mandatory Criteria

| Criterion | Met |
|-----------|-----|
| Caching strategy explicit | ✓/✗ |
| Boundaries defined | ✓/✗ |
| Cache invalidation documented | ✓/✗ |
| Performance targets specified | ✓/✗ |
| Cloud Manager compatible | ✓/✗ |

### Red Flags Detected

- [ ] None
- [ ] [Red flag with explanation]

### Weak Areas

1. [Area]: [What needs improvement]
2. [Area]: [What needs improvement]

### Strong Areas

1. [Area]: [What was done well]
2. [Area]: [What was done well]

### Recommendation

**[ACCEPT / REVISE / REJECT]**

**Rationale:** [Explain the recommendation]

### Required Actions Before Implementation

- [ ] [Action 1]
- [ ] [Action 2]
- [ ] [Action 3]
```

---

## Quick Decision Matrix

| Total Score | Mandatory Met | Recommendation |
|-------------|---------------|----------------|
| ≥80 | Yes | **ACCEPT** |
| 60-79 | Yes | **ACCEPT with conditions** |
| 60-79 | No | **REVISE** |
| 40-59 | Yes | **REVISE** |
| 40-59 | No | **REJECT** |
| <40 | Any | **REJECT** |

---

## AEM Meta-Rule Reminder

> **In AEM, performance is won at the edge, not in Java.**

Any architecture that ignores this is not cloud-ready.
