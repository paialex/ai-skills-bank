---
name: grill-me-well-done-generic
description: >
  Use this skill when the user provides any plan, idea, document, draft,
  strategy, research proposal, content plan, business plan, technical plan,
  creative concept, learning plan, or project outline that needs to be challenged,
  improved, and made more decision-ready. The skill dynamically identifies the
  domain, required expertise, reviewer personas, evaluation criteria, risks,
  alternatives, and produces an enhanced version of the original document.
metadata:
  version: "1.0.0"
---

# Grill-Me-Well-Done Generic Skill

## Purpose

This skill transforms an initial plan, idea, or document into a stronger, clearer,
more complete, more realistic, and more defensible version.

It works across domains, including but not limited to:

- engineering
- research
- business
- education
- content creation
- product strategy
- marketing
- creative writing
- operations
- personal planning
- academic work
- entrepreneurship
- public communication
- policy
- design
- finance
- career planning

The skill must not assume the domain in advance.

It must infer the required expertise from the user’s input and, when needed,
ask a short interview to identify the correct reviewer personas and evaluation criteria.

---

## Core Principle

Do not simply rewrite the document.

First understand it, challenge it, improve it, and only then produce the enhanced version.

The skill behaves like a custom expert review board assembled specifically for the provided document.

---

## Inputs

The user may provide:

- a rough idea
- a plan
- a document
- a draft
- an outline
- a proposal
- a strategy
- a research question
- a content plan
- a business concept
- a technical design
- a creative concept
- notes or bullet points

If the input is incomplete, proceed with best effort and mark assumptions clearly.

---

## Phase 1 — Intake and Domain Discovery

Analyze the provided material and identify:

- document type
- domain or field
- intended audience
- purpose
- desired outcome
- level of maturity
- decision context
- risks if the document is wrong
- missing information
- implicit assumptions
- required expertise areas

If the domain, audience, or success criteria are unclear, ask a short interview.

---

## Phase 2 — Short Interview

Ask only the minimum questions needed to assemble the right review panel.

Default interview questions:

1. Who is the final audience for this document?
2. What does success look like?
3. What are the main constraints?
4. What level of challenge do you want: light, serious, or brutal?
5. Should the final output be strategic, practical, academic, creative, executive, or implementation-ready?

Optional questions depending on context:

- What is the deadline?
- What resources are available?
- What risks worry you most?
- Are there any non-negotiables?
- Should the output preserve the original tone or fully restructure it?

If the user does not answer, infer reasonable assumptions and continue.

---

## Phase 3 — Dynamic Expert Panel Creation

Create reviewer personas based on the document.

Do not use a fixed set of personas.

Instead, infer the right ones from the content.

Examples:

For a research proposal:
- Methodology Reviewer
- Literature Review Reviewer
- Statistical Validity Reviewer
- Ethics Reviewer
- Publication Reviewer

For a content creator plan:
- Audience Strategist
- Narrative Reviewer
- Platform Growth Reviewer
- Brand Consistency Reviewer
- Monetization Reviewer

For a business plan:
- Market Analyst
- Financial Reviewer
- Operations Reviewer
- Customer Acquisition Reviewer
- Risk Reviewer

For an engineering plan:
- Domain Expert
- Safety Reviewer
- Cost Reviewer
- Reliability Reviewer
- Implementation Reviewer

For a personal career plan:
- Career Strategist
- Skills Gap Reviewer
- Market Reality Reviewer
- Execution Coach
- Risk Reviewer

Always include at least:

1. Domain Expert
2. Skeptical Reviewer
3. Practical Execution Reviewer
4. Risk Reviewer
5. Audience/User/Stakeholder Reviewer

Add specialized reviewers only when the document requires them.

---

## Phase 4 — Dynamic Evaluation Criteria

Create evaluation criteria based on the document’s purpose.

Possible criteria include:

- clarity
- completeness
- feasibility
- originality
- evidence quality
- audience fit
- strategic value
- practical usefulness
- risk awareness
- ethical soundness
- cost realism
- time realism
- operational readiness
- creativity
- persuasiveness
- accuracy
- implementation readiness
- scalability
- maintainability
- scientific validity
- emotional impact
- market fit
- differentiation
- consistency
- decision quality

Select only the criteria relevant to the provided document.

Do not over-evaluate simple documents.

---

## Phase 5 — Tree-of-Thoughts Review

Use branching review internally.

Explore several possible improvement paths:

```text
Root: User-provided document
  ├── Path A: Improve clarity
  ├── Path B: Improve correctness
  ├── Path C: Improve feasibility
  ├── Path D: Improve impact
  ├── Path E: Reduce risk
  ├── Path F: Improve audience fit
  ├── Path G: Challenge the core direction
  └── Path H: Explore stronger alternatives
```

For each relevant path:

1. Identify weaknesses.
2. Identify hidden assumptions.
3. Ask what could fail.
4. Consider alternatives.
5. Decide whether to improve, replace, or preserve the original idea.

Do not expose private reasoning chains.

Expose only useful conclusions.

---

## Phase 6 — Grill-Me Challenge

Challenge the document through the dynamically created personas.

Each persona should answer:

* What is strong?
* What is weak?
* What is missing?
* What could fail?
* What assumption is dangerous?
* What would make this more credible?
* What would make this more useful?
* What should be removed?
* What should be added?
* What alternative should be considered?

The challenge must be direct but constructive.

The goal is improvement, not criticism for its own sake.

---

## Phase 7 — Scoring

Score the current document only on relevant criteria.

Use a 1–5 scale.

Example:

```text
Clarity: 3/5
Feasibility: 2/5
Audience Fit: 4/5
Risk Awareness: 2/5
Evidence Quality: 3/5
Execution Readiness: 2/5
```

Then summarize:

* strongest elements
* weakest elements
* top risks
* missing decisions
* highest-impact improvements

---

## Phase 8 — Alternative Options

Generate alternatives only when useful.

Possible alternatives:

* simpler version
* more ambitious version
* lower-risk version
* lower-cost version
* more creative version
* more evidence-based version
* more audience-focused version
* more execution-ready version

Do not force alternatives when the document is already narrow and clear.

---

## Phase 9 — Enhanced Document

Rewrite or enhance the original document.

The enhanced version may include:

* title
* objective
* context
* target audience
* problem statement
* desired outcome
* scope
* non-goals
* assumptions
* constraints
* proposed approach
* alternatives considered
* risks
* mitigations
* evidence needed
* execution plan
* success criteria
* open questions
* final recommendation

Use only the sections that make sense for the document type.

Do not make every output look like a software architecture document.

Adapt the structure to the domain.

---

## Phase 10 — Final Output Format

Default response structure:

```text
1. Intake Summary
2. Expert Panel Created
3. Evaluation Criteria
4. Grill-Me Findings
5. Risk & Gap Analysis
6. Alternative Options
7. Enhanced Document
8. Final Recommendation
9. Open Questions
```

If the user asks for a clean final document only, provide only:

```text
Enhanced Document
```

---

## Behavior Rules

* Be field-agnostic.
* Infer expertise dynamically.
* Ask only necessary questions.
* Do not overcomplicate simple plans.
* Do not invent facts.
* Mark assumptions clearly.
* Preserve useful parts of the original document.
* Remove weak, redundant, or unsupported parts.
* Challenge premature conclusions.
* Improve both content and structure.
* Adapt tone to the intended audience.
* Produce something the user can actually use.

---

## Quality Bar

The final enhanced document should be suitable for one of the following:

* stakeholder review
* expert review
* publishing
* execution
* decision-making
* research refinement
* creative development
* business validation
* personal planning
* academic review

The final result must be stronger than the original in clarity, usefulness,
credibility, and decision quality.