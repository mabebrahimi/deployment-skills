# Product Review OS

Reusable review-first product architecture skill for Codex and other file-aware agents.

## What it does

Product Review OS reviews a feature or implementation against the whole product system before implementation proceeds.

It focuses on:

- canonical ownership / source of truth
- blast radius across related surfaces
- lifecycle correctness
- cross-domain boundaries
- UX consistency
- duplicate workflows and concepts
- shipped vs planned separation
- failure and edge cases
- AI responsibility boundaries

## Modes

### Review

Use before implementation.

Prompt example:

```text
Use Product Review OS.

Mode: Review
Project: <project>
Domain: <domain>

Read the project context, architecture, design decisions, and this feature PRD.
Review the feature against the entire product system.
Do not implement yet.
```

### Audit

Use after a design or implementation exists.

```text
Use Product Review OS.

Mode: Audit
Project: <project>
Domain: <domain>

Compare the PRD, current architecture, design decisions, and current implementation.
Tell me what is correct, what related surfaces were or were not updated, and what should be fixed.
Do not implement yet.
```

### Implement approved plan

Use only after review:

```text
Use Product Review OS.

Mode: Implement approved plan
Implement the approved correction plan and acceptance criteria.
Preserve unrelated successful product behavior.
```

## Recommended project files

Copy and fill the templates in `templates/`:

```text
PROJECT-CONTEXT.md
CURRENT-ARCHITECTURE.md
DESIGN-DECISIONS.md
DOMAIN-CONTEXT.md
```

These files make product judgment portable across new chats and agent sessions instead of relying on conversation memory.

## Key principle

> Review first. Implement second.

The feature PRD is authoritative for the feature, but the feature must still fit the existing product architecture.