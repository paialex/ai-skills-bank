# AEM Architecture Document Template

> **Rule**: For every question below, answer explicitly or mark **N/A (with justification)**. Silent omission is not allowed.

---

## Document Header

```markdown
# [Solution Name] - Architecture Document

| Field | Value |
|-------|-------|
| **Version** | 1.0 |
| **Date** | YYYY-MM-DD |
| **Authors** | [Names] |
| **Status** | Draft / In Review / Approved |
| **AEM Version** | AEM as a Cloud Service |
```

---

## 1. Context & Scope

### Questions to Answer

- What problem does this solution address?
- Who are the primary users (authors, end users, systems)?
- What are the explicit non-goals?
- What constraints shape the design (business, technical, organizational)?

### AEM-Specific

State clearly whether the system is:
- Content-heavy
- Experience-driven
- API/headless-driven
- Hybrid

### Template

```markdown
## 1. Context & Scope

### 1.1 Problem Statement

[What business problem does this solution address?]

### 1.2 Users & Stakeholders

| User Type | Role | Primary Needs |
|-----------|------|---------------|
| Content Authors | Create and manage content | [Needs] |
| End Users | Consume content | [Needs] |
| Systems | Integrate with AEM | [Needs] |

### 1.3 System Classification

This system is: **[Content-heavy | Experience-driven | API/headless-driven | Hybrid]**

Rationale: [Why this classification?]

### 1.4 Non-Goals

The following are explicitly out of scope:
- [Non-goal 1]
- [Non-goal 2]

### 1.5 Constraints

| Constraint Type | Description | Impact |
|-----------------|-------------|--------|
| Business | [Constraint] | [Impact] |
| Technical | [Constraint] | [Impact] |
| Organizational | [Constraint] | [Impact] |
```

---

## 2. System Overview

### Questions to Answer

- What is the role of AEM in this solution?
- What other systems are involved?
- What is the chosen architectural style?
- What responsibilities belong to AEM vs external systems?

### AEM-Specific

Clearly state what logic belongs in:
- AEM
- Edge/CDN
- Cloud backend

### Template

```markdown
## 2. System Overview

### 2.1 AEM Role

AEM serves as: **[Authoring hub | Content API | Rendering engine | Combination]**

Primary responsibilities:
- [Responsibility 1]
- [Responsibility 2]

### 2.2 System Context

[Include C4 Context Diagram or ASCII equivalent]

### 2.3 Architectural Style

This solution follows: **[Layered | Microservices | Event-driven | Hybrid]**

### 2.4 Responsibility Distribution

| Concern | AEM | Edge/CDN | Backend |
|---------|-----|----------|---------|
| Content authoring | ✓ | | |
| Content storage | ✓ | | |
| Content delivery | | ✓ | |
| Personalization | | | ✓ |
| Business logic | | | ✓ |
| Caching | | ✓ | |
```

---

## 3. Boundaries & Responsibilities (AEM-Critical)

### Questions to Answer

- What responsibilities are handled by AEM?
- What responsibilities are explicitly excluded from AEM?
- How are authoring, delivery, and backend concerns separated?
- What are the dependency direction rules?

### AEM-Specific

AEM must NOT be treated as a generic app server. Responsibilities must justify why logic is inside or outside AEM.

### Template

```markdown
## 3. Boundaries & Responsibilities

### 3.1 AEM Responsibilities

AEM is responsible for:
- [Responsibility with justification]

AEM is NOT responsible for:
- [Exclusion with justification]

### 3.2 Boundary Diagram

[ASCII or Mermaid diagram showing boundaries]

### 3.3 Separation of Concerns

| Concern | Layer | Justification |
|---------|-------|---------------|
| Authoring | AEM Author | [Why] |
| Presentation | AEM Publish / Edge | [Why] |
| Business Logic | Backend API | [Why] |
| Delivery | CDN | [Why] |

### 3.4 Dependency Rules

- AEM → Backend API: [Allowed / Not Allowed]
- Frontend → AEM: [Allowed / Not Allowed]
- Backend → AEM: [Allowed / Not Allowed - typically not allowed]
```

---

## 4. Content & Data

### Questions to Answer

- What data and content types exist?
- What is the system of record for each data type?
- What is the data provenance?
- Where is data persisted?
- What is the data lifecycle (creation → deletion)?
- Is the data mutable or immutable?

### AEM-Specific

- Avoid deep hierarchies and author-unfriendly structures
- Define use of Pages, Content Fragments, Experience Fragments

### Template

```markdown
## 4. Content & Data

### 4.1 Content Types

| Content Type | Storage | System of Record | Mutable |
|--------------|---------|------------------|---------|
| Pages | AEM | AEM | Yes |
| Content Fragments | AEM | AEM | Yes |
| Assets | DAM | AEM | Yes |
| User Data | External | [System] | [Yes/No] |

### 4.2 Content Hierarchy

[ASCII tree showing content structure]

### 4.3 Localization Strategy

| Aspect | Approach |
|--------|----------|
| Language copies | [MSM / Translation / Manual] |
| Regional variations | [Shared / Overridden] |
| Fallback behavior | [Strategy] |

### 4.4 Content Lifecycle

[State diagram showing content lifecycle]

### 4.5 Multi-site Strategy

| Site | Shared Content | Unique Content |
|------|----------------|----------------|
| Site A | [What] | [What] |
| Site B | [What] | [What] |
```

---

## 5. Key Flows

### Questions to Answer

- How does content flow from author to end user?
- What is the request/response path?
- What happens on cache hit vs cache miss?
- How are failures handled?

### AEM-Specific

At least one flow must explicitly describe cache invalidation.

### Template

```markdown
## 5. Key Flows

### 5.1 Authoring Flow

[Sequence diagram: Author → Preview → Publish]

### 5.2 Content Delivery Flow

[Flowchart: Request → CDN → Dispatcher → Publish → Response]

### 5.3 Cache Flow

[Flowchart showing cache hit/miss decision tree]

### 5.4 Cache Invalidation Flow

[Sequence diagram: Activation → Flush → Invalidation]

### 5.5 Failure Scenarios

| Scenario | Detection | Response | Recovery |
|----------|-----------|----------|----------|
| Publish down | [How] | [What] | [How] |
| Backend slow | [How] | [What] | [How] |
| CDN failure | [How] | [What] | [How] |
```

---

## 6. Interfaces & Integrations

### Questions to Answer

- What external integrations exist?
- What contracts are exposed by AEM?
- What contracts are consumed by AEM?
- What integration patterns are used?
- How are integration failures handled?

### AEM-Specific

Content models are **public contracts** and must be treated as such.

### Template

```markdown
## 6. Interfaces & Integrations

### 6.1 AEM Exposes

| Interface | Type | Contract | Consumers |
|-----------|------|----------|-----------|
| Content API | REST/JSON | [Contract] | [Who] |
| GraphQL | GraphQL | [Contract] | [Who] |
| Pages | HTML | [Contract] | [Who] |

### 6.2 AEM Consumes

| Interface | Provider | Pattern | Failure Handling |
|-----------|----------|---------|------------------|
| Product API | [System] | [Sync/Async] | [Strategy] |
| User API | [System] | [Sync/Async] | [Strategy] |

### 6.3 Integration Patterns

| Pattern | Use Case | Trade-offs |
|---------|----------|------------|
| Sync API call | [When] | [Trade-offs] |
| Async messaging | [When] | [Trade-offs] |
| Edge-side includes | [When] | [Trade-offs] |

### 6.4 Content Model Contracts

[Conceptual description of Sling Models and Content Fragment models]
```

---

## 7. Caching & Performance (MANDATORY)

### Questions to Answer

- What are the performance targets?
- What is cached?
- Where is caching applied?
- How is cache invalidation handled?
- What happens on cache miss?

### AEM-Specific (Hard Rule)

If caching strategy is missing or vague → architecture is invalid.

### Template

```markdown
## 7. Caching & Performance

### 7.1 Performance Targets

| Metric | Target | Rationale |
|--------|--------|-----------|
| TTFB | <200ms | [Why] |
| LCP | <2.5s | [Why] |
| Cache hit ratio | >95% | [Why] |

### 7.2 Caching Layers

| Layer | What | TTL | Invalidation |
|-------|------|-----|--------------|
| CDN | [Content types] | [Duration] | [Method] |
| Dispatcher | [Content types] | [Duration] | [Method] |
| Browser | [Content types] | [Duration] | [Method] |

### 7.3 Cache Key Strategy

| Content Type | Cache Key Components |
|--------------|---------------------|
| HTML pages | URL + [selectors] |
| API responses | URL + [params] |
| Assets | URL + hash |

### 7.4 Invalidation Strategy

| Trigger | Scope | Method |
|---------|-------|--------|
| Content publish | [Scope] | Flush / TTL |
| Code deploy | [Scope] | Full flush |
| Config change | [Scope] | [Method] |

### 7.5 Cache Miss Handling

[Describe what happens when cache is cold]
```

---

## 8. Reliability & Resilience

### Questions to Answer

- What happens if AEM Publish is unavailable?
- What happens if external systems fail?
- What is the blast radius of failures?
- How is graceful degradation handled?

### Template

```markdown
## 8. Reliability & Resilience

### 8.1 Failure Scenarios

| Component | Failure Mode | Impact | Mitigation |
|-----------|--------------|--------|------------|
| AEM Publish | Unavailable | [Impact] | [Mitigation] |
| Backend API | Slow/Down | [Impact] | [Mitigation] |
| CDN | Unavailable | [Impact] | [Mitigation] |

### 8.2 Blast Radius

[Diagram showing failure impact zones]

### 8.3 Graceful Degradation

| Scenario | Degraded Experience |
|----------|---------------------|
| Backend down | [Fallback behavior] |
| Personalization fails | [Fallback behavior] |
```

---

## 9. Security

### Questions to Answer

- How is authentication handled?
- How is authorization enforced?
- What sensitive data exists?
- How is data protected in transit and at rest?

### Template

```markdown
## 9. Security

### 9.1 Authentication

| Context | Method | Provider |
|---------|--------|----------|
| Author | [Method] | [Provider] |
| End user | [Method] | [Provider] |
| API | [Method] | [Provider] |

### 9.2 Authorization

| Resource | Access Control | Enforcement Point |
|----------|----------------|-------------------|
| Content | [ACLs] | [Where] |
| APIs | [Method] | [Where] |

### 9.3 Data Protection

| Data Type | In Transit | At Rest |
|-----------|------------|---------|
| Content | [Method] | [Method] |
| User data | [Method] | [Method] |
```

---

## 10. Deployment & Operations

### Questions to Answer

- How is the solution deployed?
- How are environments promoted?
- How is rollback handled (code, content, config)?
- What is monitored?
- What signals indicate unhealthy behavior?

### AEM-Specific

Operations must assume no shell access and immutable infrastructure.

### Template

```markdown
## 10. Deployment & Operations

### 10.1 Deployment Pipeline

[Diagram: Dev → Stage → Prod via Cloud Manager]

### 10.2 Environment Promotion

| Artifact | Promotion Method | Rollback Method |
|----------|------------------|-----------------|
| Code | Cloud Manager | [Method] |
| Content | Packages / Replication | [Method] |
| Config | Cloud Manager | [Method] |

### 10.3 Observability

| Signal | Source | Alert Threshold |
|--------|--------|-----------------|
| Error rate | [Source] | [Threshold] |
| Cache hit ratio | [Source] | [Threshold] |
| Response time | [Source] | [Threshold] |

### 10.4 Feature Flags

| Feature | Flag | Default | Override |
|---------|------|---------|----------|
| [Feature] | [Flag] | [Default] | [How] |
```

---

## 11. Cost & Scalability

### Questions to Answer

- What are the main cost drivers?
- How does cost scale with traffic or usage?
- What guardrails limit cost growth?

### Template

```markdown
## 11. Cost & Scalability

### 11.1 Cost Drivers

| Driver | Variable | Optimization |
|--------|----------|--------------|
| CDN egress | Traffic | [Strategy] |
| Author tier | Users | [Strategy] |
| Backend APIs | Calls | [Strategy] |

### 11.2 Scaling Approach

| Component | Scaling Model | Limits |
|-----------|---------------|--------|
| Publish | Auto-scale | [Limits] |
| Backend | [Model] | [Limits] |

### 11.3 Cost Guardrails

- [Guardrail 1]
- [Guardrail 2]
```

---

## 12. Compliance & Governance

### Questions to Answer

- Are there regulatory or policy constraints?
- How is compliance enforced by design?

### Template

```markdown
## 12. Compliance & Governance

### 12.1 Regulatory Requirements

| Regulation | Scope | Compliance Approach |
|------------|-------|---------------------|
| GDPR | [Scope] | [Approach] |
| Accessibility | [Scope] | [Approach] |

### 12.2 Policy Constraints

| Policy | Enforcement |
|--------|-------------|
| [Policy] | [How enforced] |
```

---

## 13. Risks & Trade-offs

### Questions to Answer

- What key trade-offs were made?
- What are the main risks?
- What decisions are deferred?

### AEM-Specific

Explicitly state what NOT to do in AEM and why.

### Template

```markdown
## 13. Risks & Trade-offs

### 13.1 Key Trade-offs

| Decision | Trade-off | Rationale |
|----------|-----------|-----------|
| [Decision] | [What we gave up] | [Why] |

### 13.2 Identified Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk] | [H/M/L] | [H/M/L] | [Strategy] |

### 13.3 AEM Anti-patterns Avoided

| Anti-pattern | Why Avoided |
|--------------|-------------|
| AEM as app server | [Reason] |
| Heavy server-side logic | [Reason] |

### 13.4 Deferred Decisions

| Decision | Defer Until | Owner |
|----------|-------------|-------|
| [Decision] | [When] | [Who] |
```

---

## 14. Design Review Summary

### Template

```markdown
## 14. Design Review Summary

### 14.1 Scorecard

| Area | Score | Notes |
|------|-------|-------|
| Boundaries & Responsibilities | /15 | |
| Caching & Performance | /15 | |
| Interfaces & Contracts | /15 | |
| Operability | /5 | |
| Content Design | /10 | |
| **Total** | /60 | |

### 14.2 Weak Areas

- [Area 1]: [What needs improvement]
- [Area 2]: [What needs improvement]

### 14.3 Recommendation

**[ACCEPT / REVISE / REJECT]**

Rationale: [Why this recommendation]

### 14.4 Required Actions Before Implementation

- [ ] [Action 1]
- [ ] [Action 2]
```
