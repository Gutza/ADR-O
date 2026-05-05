---
id: 8
type: Core vocabulary
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amendedBy:
  - ADR-0041
---

# ADR-0008 — Lifecycle Status as a SKOS Scheme with OWL Integrity

## Context

ADR-0000 required a controlled status vocabulary rather than an uncontrolled string. SKOS concept schemes are the project’s standard pattern for such enumerations (see ADR-0002 for extension). For status values, the ontology also needs reasoner-visible distinctness so that a functional `hasStatus` edge does not silently collapse distinct statuses under the open-world assumption.

## Decision

**Status values are SKOS concepts in `adr-o:statusScheme`**, with at least these five core individuals: `adr-o:Proposed`, `adr-o:Accepted`, `adr-o:Deprecated`, `adr-o:Superseded`, `adr-o:Rejected`.

**`adr-o:Status`** is the class of status values: equivalent to `skos:Concept` intersected with “in scheme `adr-o:statusScheme`”, mirroring the usual ADR-O pattern for controlled values.

**`adr-o:hasStatus`** — `owl:ObjectProperty`, **`owl:FunctionalProperty`**, domain `adr-o:DecisionRecord`, range `adr-o:Status`: at most one status per record.

**`owl:AllDifferent`** is asserted over the five core status individuals so that duplicate-status misuse becomes an OWL inconsistency when a complete OWL 2 DL reasoner is used, rather than entailing unintended equality.

**Reasoner caveat:** Some OWL 2 RL implementations (for example Python’s `owlrl`) may not implement the RL rules that fully expand `AllDifferent` into pairwise inequality in all cases. Incomplete reasoners can weaken duplicate-detection; this is a tooling limitation, not a specification error. Domain profiles that add new status individuals should extend `AllDifferent` or use explicit `owl:differentFrom` as needed.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| `xsd:string` status on the record | No shared vocabulary; poor query and validation ergonomics. |
| Rely on functional property alone without `AllDifferent` | Under OWA, distinct individuals need not be different; duplicate annotations could merge statuses silently. |

## Consequences

**Positive.** Consistent lifecycle vocabulary; functional status per record; DL reasoners can flag contradictory duplicate status annotations.

**Negative / risks.** Profiles extending the scheme must maintain integrity (`skos:inScheme`, optional `AllDifferent` updates).

## References

- [`adr-o.ttl`](/adr-o.ttl) — `adr-o:hasStatus`, `adr-o:Status`, `adr-o:statusScheme`, status individuals, status `owl:AllDifferent` block.
- ADR-0000 — Inception Record (status list intent).
- ADR-0002 — Scope (SKOS extension pattern).
