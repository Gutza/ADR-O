---
id: 23
type: Ontology vocabulary
status: Accepted
date: 2026-04-21
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0023 — Add hasType Annotation Property

## Context

The roadmap identifies a persistent gap in Horizon 2 social-graph completion: decision records expose accountable actors but still lack a first-class way to express record-level type metadata. Dogfooded records consistently contain a Type field (for example, "Project Governance" and "Core Design"), and this field classifies the decision record itself rather than any domain object.

The historic deferral argument has been that domain categorization does not belong in the core namespace. That boundary is still valid for domain semantics, but it does not apply to epistemic record metadata. Without an explicit predicate, Type remains trapped in prose and cannot be queried or validated consistently across records.

## Decision

Introduce `adr-o:hasType` as an `owl:AnnotationProperty` from `adr-o:DecisionRecord` to a SKOS concept IRI.

No core concept scheme is added in ADR-O itself. Type values are supplied by extension profiles, following the same extension pattern already used for `adr-o:Concern`.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Keep Type as prose only | Prevents consistent SPARQL traversal and weakens record-level comparability across ADRs. |
| Model Type as `owl:ObjectProperty` | Overstates ontological commitment for metadata intended as lightweight record annotation. |
| Add a mandatory core ADR-O type scheme | Violates ADR-O's domain-agnostic extension model and imposes premature taxonomy governance. |

## Consequences

### Positive

Record categorization becomes queryable while preserving ADR-O's extension-first design. Teams can apply profile-defined SKOS schemes without modifying the core namespace.

### Negative / risks

Without profile-level guidance, teams may create inconsistent local type vocabularies, reducing cross-project interoperability.

### Open questions

Should future SHACL companion shapes recommend cardinality or controlled-scheme checks for `adr-o:hasType` in profile layers?

## References

- `ontology/ROADMAP.md` (Horizon 2, Social Graph item on `adr-o:hasType`)
