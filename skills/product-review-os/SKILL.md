---
name: product-review-os
description: Use when reviewing, auditing, planning, or refining a product change before implementation. Especially useful when a new feature, PRD, design, workflow, or code change must be checked against an existing product architecture, domain rules, UX patterns, lifecycle semantics, source-of-truth ownership, shipped-vs-planned boundaries, and related surfaces. Use review-first by default; do not implement until the architecture review is complete unless the user explicitly asks to implement an already-approved plan.
---

# Product Review OS

A reusable product-system review skill for preventing locally-correct features from creating system-level inconsistencies.

The skill is intentionally project-agnostic. Project and domain knowledge should come from repository context files, feature specs, current architecture documents, design decisions, and the implementation itself.

## Core principle

Review first. Implement second.

A feature specification is authoritative for the feature itself, but it must still be reconciled with the existing product system.

Never evaluate a non-trivial feature in isolation.

## When to use

Use this skill when the user asks to:

- review a PRD, product spec, or proposed feature
- audit an implementation against a PRD or design artifact
- critique a new screen, workflow, lifecycle, dashboard, or interaction
- identify blast radius across product surfaces
- check whether related areas were updated consistently
- find duplicate concepts, parallel workflows, or conflicting state models
- reconcile a new feature with existing product architecture
- create an implementation prompt after product review
- review a feature before giving it to Codex or another coding agent
- verify that shipped and planned functionality are clearly separated

## Inputs to read

Read all relevant inputs that exist before making a judgment:

1. Project context
2. Current architecture
3. Domain context
4. Design decisions / product principles
5. Feature PRD or spec
6. Current implementation, design artifact, code, or screenshots
7. Relevant prior decisions recorded in the repository

Recommended file names:

- `PROJECT-CONTEXT.md`
- `CURRENT-ARCHITECTURE.md`
- `DOMAIN-CONTEXT.md`
- `DESIGN-DECISIONS.md`
- feature PRDs/specs

Do not silently invent missing project rules. If a missing input materially affects correctness, call it out explicitly.

## Modes

Infer the mode from the request.

### Review

Use when the feature is not yet implemented or the user wants architectural/product guidance.

Output a review and recommended architecture. Do not implement unless asked afterward.

### Audit

Use when the feature already exists in code or a design artifact.

Compare:

- PRD/spec intent
- current architecture
- implementation
- related surfaces
- established product decisions

Return what is correct, what is missing, what is inconsistent, and the corrective plan.

### Implement approved plan

Use only when the user explicitly asks to implement an already-reviewed plan.

Before editing:

- re-read the approved architecture and acceptance criteria
- inspect the current code/artifact
- preserve unrelated successful areas
- make only the scoped changes
- run relevant validation

Do not reopen settled product decisions without a concrete conflict.

## Ordered review workflow

### 1. Understand the change

State the product change in concrete terms:

- user problem
- objective
- actors
- new or changed entities
- lifecycle transitions
- side effects
- data created or mutated
- new screens or interactions
- dependencies on other domains

Do not begin from UI components. Begin from product semantics.

### 2. Map canonical ownership

For every important business fact ask:

- Who owns this fact today?
- Is there already a canonical source of truth?
- Is the new feature creating another owner?
- Is another domain's logic being duplicated?
- Does the UI derive the value from the correct owner?

Flag any second source of truth, duplicated formula, copied lifecycle logic, or competing aggregate.

### 3. Map blast radius across product surfaces

Explicitly check whether the change affects:

- navigation
- dashboard
- global lists/registers
- canonical detail pages
- creation flows
- approval or review flows
- lifecycle/state displays
- documents and bundles
- history/audit
- permissions and separation of duties
- notifications
- reporting/exports
- search
- configuration
- AI surfaces
- mobile/responsive behavior
- empty/loading/error/stale states

Do not assume the PRD only affects the newly added screen.

### 4. Check lifecycle correctness

Verify that:

- states represent real business states
- alternative paths are not presented as sequential universal stages
- route-specific behavior remains route-specific
- future states are not shown as if shipped
- draft/create/submit/approve/issue/execute/pay/close semantics remain distinct
- correction, cancellation, reversal, supersession, and immutability are explicit
- a stepper is used only when the user actually moves through a shared sequence

Reject fake linear lifecycles that flatten branching domain behavior.

### 5. Check interaction consistency

Prefer interaction patterns that are already successful in the product.

Examples:

- common creation shell
- review-before-submit
- canonical record detail
- personal decision inbox
- immutable history
- contextual drilldowns
- source-aware evidence review
- explicit consequence before governed actions

Introduce a new interaction pattern only when the domain genuinely requires it.

Consistency is not sameness: reuse the shell, not irrelevant fields or semantics.

### 6. Check duplication and competing concepts

Look for:

- duplicate dashboards
- duplicate list/register pages
- duplicate approval/review paths
- competing status models
- duplicate financial or operational truth
- duplicate document ownership
- same action available in unrelated places with different semantics
- new terminology for an existing concept
- two UIs that answer the same user question

If two surfaces look similar, define the distinct question each one answers.

### 7. Check shipped vs planned boundaries

Classify functionality as needed:

- shipped/live
- current MVP
- planned
- future-state
- design/handoff-only

Planned functionality must not visually or behaviorally masquerade as live product functionality.

Do not expose future architecture merely because it exists in a roadmap.

### 8. Check failure modes and edge cases

At minimum consider:

- missing source data
- invalid or stale source data
- concurrent changes
- partial completion
- rejection
- cancellation
- reversal
- supersession
- permission denied
- self-action / separation of duties conflicts
- empty states
- loading states
- unavailable dependency
- duplicate submission/action
- mobile constraints

For AI-assisted surfaces also consider:

- insufficient evidence
- conflicting evidence
- stale analysis
- model failure
- deterministic fallback

### 9. Check cross-domain boundaries

Ask:

- Is one domain mutating another domain's canonical facts?
- Is one domain importing another domain's internal persistence model?
- Is a bridge/projection/port more appropriate?
- Does a future domain get prematurely reimplemented in the current feature?
- Does the current product distinguish intake, evidence, approval, liability, execution, payment, and reconciliation correctly?

Prefer explicit interfaces between bounded contexts over shared accidental ownership.

### 10. Check UX hierarchy

For every affected screen, ask one question:

**What primary user question does this screen answer?**

Typical answers:

- Dashboard: What needs attention across the business?
- List/register: Which record am I looking for or need to act on?
- Detail: What is happening with this record?
- Decision page: What do I need to know to decide?
- Creation: What must I provide to create/submit this object?
- Configuration: How is behavior controlled?

Reject pages that answer too many unrelated questions at once.

### 11. Check information density

Do not treat completeness as showing every field.

For lists and dashboards:

- prioritize scanability
- show operationally meaningful fields
- move secondary metadata into detail views
- avoid table sprawl
- avoid repeating canonical detail information

### 12. Check AI behavior when present

Apply this rule:

**Rules decide. The system calculates. AI explains, summarizes, compares, and highlights.**

AI must not become the canonical source of deterministic facts.

AI outputs should be:

- grounded in permitted records
- source-aware
- clearly advisory when inferential
- resilient to model failure
- visually secondary to authoritative system state

## Global product principles

Apply these unless project/domain context explicitly overrides them:

1. Do not evaluate features in isolation.
2. One business fact should have one canonical owner.
3. Do not create parallel workflows for the same decision.
4. Prefer contextual integration over disconnected side systems.
5. Preserve immutable historical facts when history matters.
6. Corrections should be explicit transitions, replacements, supersessions, or linked reversals.
7. Planned functionality must not look shipped.
8. Route-specific behavior must stay route-specific.
9. Do not turn branching lifecycles into fake linear steppers.
10. Reuse established interaction patterns when appropriate.
11. Creation and submission are different actions when side effects differ.
12. Review/decision screens should expose consequences before governed actions.
13. Lists should prioritize operational decisions, not every available field.
14. UI should represent business semantics, not backend structure.
15. AI may explain deterministic facts but must not become their source of truth.
16. Important claims should be traceable to system records when the product supports evidence.
17. Debug, test, fixture, and design controls should not leak into normal product mode.
18. New features should degrade gracefully when optional services fail.
19. The canonical record page should remain the primary lifecycle context unless architecture explicitly says otherwise.
20. Avoid exposing future architecture unless it helps the user's current task.
21. Do not duplicate a domain's formulas in another domain for convenience.
22. If a feature is an MVP bridge, preserve future upgrade paths without pretending the future model already exists.
23. Do not infer financial, operational, or compliance truth from labels when the underlying domain has a more precise state.
24. A user-facing lifecycle visualization must reflect actual reachable paths, not merely roadmap order.

## Required review output

Unless the user asks for another format, return these sections:

### Summary

Concise product-level judgment.

### What is correct

Identify parts that already align with the spec and product system.

### System-level issues

Rank issues as:

- Critical
- Important
- Minor

For each issue include:

- why it matters
- affected surfaces
- recommended correction

### Related surfaces checked

State which related parts were inspected and whether they changed correctly.

### Architecture recommendation

Describe the corrected product architecture or interaction model.

### Things to preserve

Protect successful existing behavior from unnecessary redesign.

### Acceptance criteria

Write concrete, testable checks.

### Implementation readiness

Choose one:

- Ready to implement
- Ready after minor corrections
- Requires architecture correction first

Do not produce implementation code in Review/Audit mode unless the user asks for it.

## Corrective prompt generation

When the user asks for a prompt for Codex or another implementation agent:

- scope the change tightly
- list what must not change
- include the product reasoning behind critical constraints
- define hard failure conditions
- define visual/behavior QA
- include shipped-vs-planned rules
- require related-surface updates
- require implementation to preserve canonical owners and lifecycle semantics

A good corrective prompt should prevent the implementation agent from solving only the local screen.

## Recommended project context structure

Projects can adopt this structure:

```text
/product-os
  PRODUCT-REVIEW-PRINCIPLES.md
  PRODUCT-REVIEW-CHECKLIST.md

/project-context
  PROJECT-CONTEXT.md
  CURRENT-ARCHITECTURE.md
  DESIGN-DECISIONS.md

/domains/<domain>
  DOMAIN-CONTEXT.md
  DESIGN-DECISIONS.md
```

The exact paths are optional. Prefer repository conventions if equivalent documents already exist.

## Decision recording

When a review produces a durable product rule, suggest recording it in the project's `DESIGN-DECISIONS.md` rather than relying on chat history.

Good design decisions are short, stable, and semantic.

Examples:

- Dashboard is not a duplicate operational list.
- Variations never mutate the issued Original.
- Future Finance stages are not exposed as current Product Mode lifecycle.
- Creation flows reuse the established step-based shell unless domain semantics require otherwise.

Do not record temporary implementation details as product principles.

## Final standard

The review is successful when a new feature is not merely correct in isolation, but coherent with the whole product system.