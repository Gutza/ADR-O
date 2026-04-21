---
id: 6
type: Core vocabulary
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0006 — Core Relational Predicates and PROV Alignment

## Context

ADR-0000 established the intent to mint a small set of ADR-specific object properties linking `DecisionRecord` nodes and related resources, informed by prior art (including Kruchten-style relationships and OntolAD’s lessons). The ontology needed a clear split between *what problem a decision responds to* and *what in the world it changes*, plus alignment with PROV-O for supersession as revision history.

Background discussion for early ontology iterations is preserved informally in project design notes; this ADR records the decision as implemented in the canonical ontology.

## Decision

**The ADR-O core publishes the following relational vocabulary** (all in `https://w3id.org/adr-o#` unless noted):

- **`adr-o:addresses`** — links a subject to an `adr-o:Concern` (problem / ASR side). Range is `adr-o:Concern`; domain is intentionally unrestricted so the predicate can attach to `adr-o:Consideration` (canonical fine-grained use) and to `adr-o:DecisionRecord` (optional editorial rollup). See ADR-0013 for the dual-placement convention.
- **`adr-o:affects`** — domain `adr-o:DecisionRecord`, range `rdfs:Resource` (open): the *manifestation* side (what the decision touches or constrains in the system or organisation).
- **`adr-o:dependsOn`**, **`adr-o:enables`** — pairwise links between `DecisionRecord` individuals for dependency and enablement semantics.
- **`adr-o:supersedes`** / **`adr-o:supersededBy`** — supersession chain between records; `adr-o:supersedes` is declared **`rdfs:subPropertyOf`** `prov:wasRevisionOf` so PROV-O-aware tooling treats supersession as revision.
- **`adr-o:conflictsWith`** — `owl:SymmetricProperty` between two `DecisionRecord` individuals for “in tension” relationships. Directed Kruchten-style `prevents` / `isPreventedBy` remain a possible non-breaking extension later.

**Cross-links between `addresses` and `affects`** use `rdfs:seeAlso` on those properties; the core does **not** mint a dedicated `adr-o:seeAlso` when `rdfs:seeAlso` suffices.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Single `affects` for both problem and manifestation (OntolAD-style conflation) | Loses the distinction needed for targeted “why this?” vs filter-by-system queries; separated in ADR-O as `addresses` vs `affects`. |
| Directed `conflictsWith` only | Symmetric relation is simpler for the common “A and B are in tension” case; directed variants can be added as sub-properties later. |
| Supersession not aligned with PROV-O | Aligning `adr-o:supersedes` under `prov:wasRevisionOf` gives interoperable revision semantics without importing a full PROV axiom bundle as `owl:imports`. |

## Consequences

**Positive.** SPARQL can traverse decision-to-decision and decision-to-concern graphs with explicit predicates; PROV-O consumers see supersession as revision.

**Negative / risks.** Open range on `affects` allows undisciplined targets until domain profiles or SHACL tighten usage.

**Open questions.** Whether to add `prevents` / `isPreventedBy` as sub-properties of `conflictsWith` when a use case requires directed conflict edges.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `adr-o:addresses`, `adr-o:affects`, `adr-o:dependsOn`, `adr-o:enables`, `adr-o:conflictsWith`, `adr-o:supersedes`, `adr-o:supersededBy`, `prov:wasRevisionOf` (scope note).
- ADR-0000 — Inception Record (vocabulary intent and survey).
- ADR-0002 — Scope (`Concern`, domain-agnostic core).
