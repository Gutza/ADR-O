---
id: 41
type: Core design
status: Accepted
date: 2026-05-05
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends: [6, 8]
---

# ADR-0041 - Deprecation Requires a Deprecating Record

## Context

ADR-O distinguishes two lifecycle outcomes for prior decisions:

- `Superseded`: the prior record has a direct successor and is linked through `adr-o:supersededBy`.
- `Deprecated`: the prior record is no longer recommended and has no direct replacement.

This distinction is correct, but current relational vocabulary does not capture a key governance fact: deprecation is itself an explicit decision event and therefore must be anchored in a DecisionRecord.

Without a dedicated relation, a record can be marked `Deprecated` with no machine-traversable link to the record that justified this status transition.

## Decision

Adopt a dedicated deprecation relation family for DecisionRecord-to-DecisionRecord links:

1. Mint `adr-o:deprecates` and its inverse `adr-o:deprecatedBy`.
2. Use these predicates only for deprecation governance ("record A deprecates record B"), not as replacement semantics.
3. Keep replacement semantics exclusive to `adr-o:supersedes` / `adr-o:supersededBy`.
4. Normative authoring rule: when a record is set to `Deprecated`, at least one deprecating record must be linked via `adr-o:deprecatedBy` (or equivalently from the other side using `adr-o:deprecates`).

## Alternatives considered

| Alternative | Reason not chosen |
|---|---|
| Reuse `adr-o:supersedes` / `adr-o:supersededBy` for deprecation | Incorrectly asserts replacement semantics for records that have no successor. |
| Reuse `adr-o:amends` / `adr-o:amendedBy` for deprecation | Amendment keeps the anchor record in force; deprecation withdraws recommendation. |
| Keep deprecation unlinked and rely on prose only | Loses machine-traversable provenance for why deprecation happened. |

## Consequences

**Positive.**

- Preserves a clean semantic boundary between replacement and deprecation.
- Makes deprecation provenance queryable at ADL scope.
- Supports integrity checks: deprecated records can be validated for explicit deprecation source.

**Negative / risks.**

- Adds two predicates to the core relation set and corresponding authoring discipline.
- Existing deprecated records may require backfilling links during migration.

## References

- [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md) - Core DecisionRecord relation vocabulary.
- [ADR-0008](/ADL/ADR-0008-status-skos-and-integrity.md) - Lifecycle status scheme including Deprecated and Superseded.
