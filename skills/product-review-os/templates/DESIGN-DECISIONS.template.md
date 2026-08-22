# Design Decisions

Record only durable product decisions that should survive across chats, agents, and implementation cycles.

Keep decisions short, semantic, and stable.

## Decisions

- 

## Decision format

Use:

- **Decision:** <short rule>
  - Why: <brief rationale>
  - Applies to: <project/domain/surfaces>
  - Override condition: <when this rule may be intentionally broken>

## Good examples

- **Decision:** Dashboard is not a duplicate operational list.
  - Why: Dashboard answers what needs management attention, while the register answers which record to open.
  - Applies to: Dashboard and list/register surfaces.
  - Override condition: Only if the product explicitly has no separate operational register.

- **Decision:** Creation flows reuse the established step-based shell unless domain semantics require otherwise.
  - Why: Preserve interaction consistency without forcing identical fields.
  - Applies to: New record creation flows.
  - Override condition: A materially simpler action does not benefit from staged creation.

## Do not record

- temporary implementation details
- one-off fixture values
- current CSS choices
- transient bugs
- model-specific prompt wording
