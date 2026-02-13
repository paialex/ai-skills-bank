---
name: aem-design-architect
description: AEM architecture documentation skill focused on creating design documents, NOT code. Use when asked to create architecture documents, design documentation, technical specifications, solution designs, or architectural diagrams for AEM projects. Generates structured markdown documents with Mermaid/ASCII diagrams following C4 model, arc42, and AEM-specific patterns. Covers AEMaaCS, content architecture, caching strategies, system boundaries, and integration patterns. Triggers on requests for "architecture document", "design document", "technical design", "solution architecture", "document the architecture", or visualization of AEM systems.
---

# AEM Design Architect Skill

Create comprehensive AEM architecture documentation with visualizations.

---

## Purpose

This skill creates **design and architecture documentation** for AEM projects. It does NOT implement code - it produces structured documents that communicate architectural decisions, system boundaries, and design rationale.

**Output**: Markdown documents with embedded Mermaid/ASCII diagrams.

---

## Document Scope Selection

**ALWAYS ask the user first:**

> What type of documentation do you need?
> - **Quick Design** - Essential sections for early-stage planning (default)
> - **Focused Sections** - Specific sections only (tell me which ones)
> - **Full Document** - Comprehensive 14-section architecture document

If no response within the conversation flow, default to **Quick Design**.

### Quick Design Sections
1. Context & Scope
2. System Overview
3. Boundaries & Responsibilities
4. Key Flows (with diagrams)
5. Caching & Performance Strategy
6. Risks & Trade-offs

### Full Document Sections
All 14 sections from [references/document-template.md](references/document-template.md)

---

## Workflow

### Phase 1: Requirements Gathering

**Step 1.1: Determine Input Source**

Check for and process:
1. **Attached files** (.txt, .doc, .docx, .md) - Extract requirements
2. **Existing codebase** - Analyze if user points to AEM project
3. **Conversation** - Ask discovery questions

**Step 1.2: Discovery Questions**

Ask relevant questions (adapt based on context):

**Business Context**
- What problem does this solution address?
- Who are the users (authors, end users, systems)?
- What are the explicit non-goals?

**AEM Role**
- Is this content-heavy, experience-driven, API/headless, or hybrid?
- What systems integrate with AEM?
- What is AEM's primary role (authoring hub, content API, rendering engine)?

**Content & Delivery**
- How many sites/brands/languages?
- What's the publishing frequency and content volatility?
- What are the performance targets (TTFB, cache hit ratio)?

**Integrations**
- What external APIs does AEM consume?
- What APIs does AEM expose?
- Are there backend services for business logic?

### Phase 2: Document Generation

**Step 2.1: Select Architectural Patterns**

Choose patterns based on context:

| Pattern | Use When |
|---------|----------|
| **C4 Model** | Need clear zoom levels (Context → Containers → Components) |
| **arc42** | Need comprehensive coverage with quality scenarios |
| **ADRs** | Need to document key decisions with rationale |
| **Flow-centric** | Need to emphasize request/response paths |

**Step 2.2: Generate Document Structure**

Read [references/document-template.md](references/document-template.md) for the full template.

**Step 2.3: Create Diagrams**

Follow [references/diagram-patterns.md](references/diagram-patterns.md) for:
- **Mermaid** - For complex flows, sequences, class relationships
- **ASCII** - For simple structures, file trees, layer diagrams

### Phase 3: Validation

Before presenting the document:

1. **Verify all questions answered** - Each section question must have answer or N/A with justification
2. **Check AEM-specific rules** - See quality gates below
3. **Validate diagrams** - Ensure consistency with text
4. **Review caching strategy** - Must be explicit and complete

---

## AEM-Specific Rules

### Mandatory Principles

1. **Content as data** - Treat content models as public contracts
2. **Edge-first performance** - "Performance is won at the edge, not in Java"
3. **Separation of concerns** - Never mix authoring and delivery
4. **Cache-driven design** - Performance requirements must drive caching strategy

### What NOT to Include

- Component HTL code
- OSGi configurations
- Java class implementations
- Environment-specific values (URLs, secrets, sizing)

### What MUST be Included

- System boundaries and responsibilities
- Caching strategy at all levels (CDN, Dispatcher, client)
- Cache invalidation approach
- Content model contracts (conceptual)
- Integration failure handling

---

## Quality Gates (AEM-Weighted)

| Area | Minimum Score | Weight |
|------|---------------|--------|
| Boundaries & Responsibilities | 13/15 | Critical |
| Caching & Performance | 12/15 | Critical |
| Operability | 4/5 | High |
| Interfaces & Contracts | 10/15 | High |
| Content Design | 8/10 | Medium |

**Hard Rule**: If caching strategy is missing or vague → architecture is invalid.

---

## Diagram Guidelines

### When to Use Mermaid

- Sequence diagrams (request flows, cache invalidation)
- Flowcharts (decision trees, cache hit/miss)
- Class diagrams (service dependencies, model contracts)
- State diagrams (content lifecycle)
- C4 diagrams (context, container, component views)

### When to Use ASCII

- Layer diagrams (simple stacks)
- Component file structures (tree views)
- Quick boundary sketches
- Inline architecture notes

### Diagram Requirements

Every document MUST include specifications for:

1. **AEM Context Diagram** - Users, CDN, Author, Publish, backends
2. **Caching Flow Diagram** - Request path, cache hit/miss, invalidation
3. **Component/Container Diagram** - Internal AEM structure (optional for Quick Design)

---

## References

| Reference | Use When |
|-----------|----------|
| [references/document-template.md](references/document-template.md) | Full 14-section template with all questions |
| [references/diagram-patterns.md](references/diagram-patterns.md) | Mermaid and ASCII diagram templates |
| [references/aem-patterns.md](references/aem-patterns.md) | AEM-specific architectural patterns |
| [references/quality-checklist.md](references/quality-checklist.md) | Design review scorecard |

---

## Quick Reference: Document Structure

```
# [Solution Name] - Architecture Document

## Document Info
- Version, Date, Authors, Status

## 1. Context & Scope
- Problem, Users, Non-goals, Constraints

## 2. System Overview
- AEM role, Other systems, Architectural style

## 3. Boundaries & Responsibilities
- AEM vs External, Authoring vs Delivery vs Backend

## 4. Content & Data
- Types, System of record, Lifecycle, Localization

## 5. Key Flows
- Authoring, Delivery, Cache hit/miss, Failures

## 6. Interfaces & Integrations
- External APIs, AEM contracts, Failure handling

## 7. Caching & Performance (MANDATORY)
- CDN, Dispatcher, Invalidation, Cache keys

## 8. Reliability & Resilience
- Failure scenarios, Blast radius, Degradation

## 9. Security
- AuthN, AuthZ, Data protection

## 10. Deployment & Operations
- Pipelines, Monitoring, Rollback

## 11. Cost & Scalability
- Cost drivers, Scaling approach

## 12. Compliance & Governance
- Regulatory, Policy constraints

## 13. Risks & Trade-offs
- Key trade-offs, Risks, Open decisions

## 14. Design Review Summary
- Score, Weak areas, Recommendation
```

---

## Output Format

Generate documents in **Markdown** with:
- Clear heading hierarchy (H1 for title, H2 for sections, H3 for subsections)
- Mermaid diagrams in fenced code blocks (```mermaid)
- ASCII diagrams in fenced code blocks (```)
- Tables for structured data
- Blockquotes for important notes and rules
- Checklists where appropriate

---

## Meta-Rule (Non-Negotiable)

> **In AEM architectures, performance, caching, and boundaries matter more than internal implementation details.**

Any document that ignores this is not cloud-ready.
