---
id: 27
type: Core vocabulary
status: Accepted
date: 2026-04-23
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0010
---

# ADR-0027 — Rename `adr-o:consideration` to `adr-o:manifests`

## Context

ADR-0010 established the atom-first architecture: `Consideration` nodes are reusable atoms of claim/observation, and `*Fact` nodes (`ContextFact`, `DeliberationFact`, `OutcomeFact`) are reified links that place those atoms into roles. The predicate linking them was named `adr-o:consideration`.

While functional, `adr-o:consideration` as a noun-predicate is linguistically underspecified. It tells us what is on the object side of the triple, but not what the relationship *is*. When reading a triple like `:fact-1 adr-o:consideration :cons-A`, the reader must infer the semantics from documentation rather than the predicate itself.

Several verb-based alternatives were evaluated:
- `hasConsideration`: Too weak; suggests containment rather than role-manifestation.
- `usesConsideration`: Too operational; implies the Fact is an agent employing a tool.
- `representsConsideration`: Too distant; suggests the Fact is a proxy for the Consideration rather than a specific instance of it.
- `placesConsideration`: Accurate to the ADR-0010 wording, but clunky.

## Decision

**Rename the predicate `adr-o:consideration` to `adr-o:manifests`.**

This predicate remains:
- **Domain:** union of `adr-o:ContextFact`, `adr-o:DeliberationFact`, and `adr-o:OutcomeFact`
- **Range:** `adr-o:Consideration`
- **Characteristic:** `owl:FunctionalProperty` (at most one Consideration per Fact)

### Rationale

The name `manifests` precisely captures the relationship between the atom and its role-bound instance:

1. **Symmetry with the Project scope.** In ADR-0025, `adr-o:materializes` links a `DecisionRecord` to a system artifact (the decision becomes a real thing) in the Project scope. `adr-o:manifests` does the same in the ADR scope: a `Consideration` (abstract claim) becomes manifest as a `*Fact` (contextualized claim).
2. **Avoids identity collapse.** It is distinct from `owl:sameAs`. A Fact *manifests* a Consideration; it is not the Consideration itself.
3. **Avoids agentic metaphor.** Unlike `uses`, `manifests` is a declarative state, not an operation. A Fact doesn't "do" anything; it simply is the manifestation of the Consideration in a role.
4. **Concise yet precise.** It is a single verb that reads naturally in English: *"This deliberation fact manifests consideration X."*
5. **Preserves the Y-statement mapping.** In the Y-statement roundtrip, each clause *manifests* the underlying consideration.

### Cardinality Clarification

The rename does not change the cardinality. The predicate remains `owl:FunctionalProperty` (0..1). The intended authoring convention—that every Fact should manifest **exactly one** Consideration—remains a tooling/SHACL responsibility, not an OWL axiom, to avoid closed-world reasoning issues in the core ontology.

## Consequences

**Positive.**
- The ontology is more self-documenting; the predicate name carries the conceptual weight of the reification pattern.
- Linguistic consistency with `adr-o:materializes`.
- Removes the noun-predicate ambiguity.

**Negative / risks.**
- Breaking change for any existing SPARQL queries or Turtle data using `adr-o:consideration`. Since the ontology is pre-1.0, this is acceptable.

## References

- [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md) — the original reification design.
- [ADR-0025](/ADL/ADR-0025-causal-network.md) — `materializes` / `justifiedBy` parallel.
- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — predicate declaration.
