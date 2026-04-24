---
id: 16
type: Core vocabulary
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amendedBy:
  - ADR-0028
---

# ADR-0016 — `dcterms:version` for In-Record Iteration

## Context

Supersession (`adr-o:supersedes` / `adr-o:supersededBy`) models replacement by a **new** record identity. Teams also need to track **minor amendments or clarifications** to the **same** record IRI without starting a new node. Dublin Core’s `dcterms:version` is the standard hook for a human-readable version tag on a resource.

## Decision

**Declare `dcterms:version` as an `owl:AnnotationProperty`** with an ADR-O scope note: optional literal version tag (e.g. `1.2`) on a `DecisionRecord` for iterations **between** supersession events. Distinct from the supersession chain, which changes record identity.

This does **not** resolve future typed predicates such as `adr-o:amends` or `adr-o:clarifies`; those remain out of scope until minted in the ontology.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Encode minor edits only via new `DecisionRecord` nodes | Heavyweight for small clarifications; churns identity for the same logical ADR. |
| Ad hoc custom property | `dcterms:version` is standard and interoperable. |

## Consequences

**Positive.** Lightweight iteration signal on a stable IRI; complements ADR-0005’s temporal fields.

**Negative / risks.** String tags are not semver-governed by the ontology; governance stays a team convention until a separate policy exists.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `dcterms:version` declaration and scope note.
- ADR-0006 — `adr-o:supersedes` / `adr-o:supersededBy` (successor-record model).
