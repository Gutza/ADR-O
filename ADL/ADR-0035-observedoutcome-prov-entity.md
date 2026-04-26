---
id: 35
type: Core vocabulary
status: Accepted
date: 2026-04-26
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0035 — `ObservedOutcome` as a First-Class Provenance Entity

## Context

ADR-0030 and ADR-0033 established `ObservedOutcome` as the mechanism for closing the learning loop—recording what actually happened after a decision was made. However, `ObservedOutcome` currently exists as a bare class without formal alignment to provenance standards.

A true observation is not just a claim; it is a **provenance event**. It was produced at a specific time, by a specific agent, and is backed by specific evidence. Without these anchors, an `ObservedOutcome` is structurally indistinguishable from a prediction that happens to have a later date.

The Decision Transaction Principle (ADR-0028) requires that we distinguish between what was believed at t₀ and what was observed at tₙ. For that distinction to be machine-verifiable, the observation must be a first-class provenance artifact.

An earlier PROV alignment debate also clarified that `rdfs:subClassOf` is a universal commitment, not a soft hint: every instance of the subclass must satisfy the superclass semantics. For `prov:Entity`, this means fixed aspects, attributable provenance, and traceable lifecycle. `ObservedOutcome` satisfies this test cleanly as a tₙ artifact minted at observation time, without cross-record identity coupling.

By contrast, extending the same alignment to `Claim` remains more nuanced because ADR-O deliberately uses identity only for intra-record coherence and `derivedFrom` for inter-record reference. ADR-0035 therefore makes the narrower, lower-risk move that is defensible for every instance now: align `ObservedOutcome` with `prov:Entity` and leave `Claim` out of scope.

## Decision

**`adr-o:ObservedOutcome` is a subclass of `prov:Entity`.**

This alignment is honest: an observation is a digital artifact with fixed aspects (what was seen, when, by whom) that can be cited and traced.

To make this citizenship legitimate, we add the following predicates:

### 1. The Observer (RACI: Responsible)
```ttl
adr-o:observedBy rdf:type owl:ObjectProperty ;
                rdfs:domain adr-o:ObservedOutcome ;
                rdfs:range prov:Agent ;
                rdfs:comment "The agent who performed the observation."@en ;
                rdfs:label "observed by"@en ;
                adr-o:justifiedBy adr-odr:ADR-0035 .
```
This extends the RACI pattern to the learning loop. Every observation must be attributable.

### 2. The Evidence (Grounding)
```ttl
adr-o:evidence rdf:type owl:ObjectProperty ;
               rdfs:domain adr-o:ObservedOutcome ;
               rdfs:range rdfs:Resource ;
               rdfs:comment "External evidence supporting the observation (log, report, metric)."@en ;
               rdfs:label "evidence"@en ;
               adr-o:justifiedBy adr-odr:ADR-0035 .
```
This is the "Telemetry Tom" guardrail: an observation without evidence is just another claim.

### 3. The Timestamp
```ttl
adr-o:observedAt rdf:type owl:AnnotationProperty ;
                 rdfs:domain adr-o:ObservedOutcome ;
                 rdfs:range xsd:date ;
                 rdfs:comment "The date the observation was made (tₙ)."@en ;
                 rdfs:label "observed at"@en ;
                 adr-o:justifiedBy adr-odr:ADR-0035 .
```
While `dcterms:date` is used on `DecisionRecord`, we mint `observedAt` here to create a distinct temporal anchor. This preserves the temporal distance between t₀ and tₙ as a first-class queryable property.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Subclass `Claim` | Category error: observations are realizations, not propositions |
| Use `dcterms:date` | Acceptable, but `observedAt` explicitly marks the domain-specific tₙ horizon, and mirrors DecisionRecord's `adr-o:decidedAt` |
| Skip `prov:Entity` | Leaves observations as floating facts without provenance |
| Use `prov:wasAttributedTo` | `observedBy` is more idiomatic for the observation role |
| Align both `Claim` and `ObservedOutcome` now | `ObservedOutcome` is clean; `Claim` still carries cross-record identity interpretation risk |

## Consequences

**Positive.**
- **Verifiable learning loop**: "Who observed this violation, and when?" becomes a SPARQL query.
- **Evidence-backed**: Observations must point to something outside the ontology.
- **PROV-O interoperability**: `ObservedOutcome` can participate in wider provenance chains.
- **Symmetry**: The tₙ side of the loop now has comparable metadata to the t₀ side.

**Negative / risks.**
- Authors must provide more metadata per observation.
- Tooling must surface these new fields.

## References

- [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) — DTP
- [ADR-0030](/ADL/ADR-0030-observations.md) — Observations
- [ADR-0033](/ADL/ADR-0033-observation-discovery.md) — Discovery types
- [Archive Note: PROV Entity Subclassing Debate](/Archive/prov-entity-subclassing.md)
- PROV-O: https://www.w3.org/TR/prov-o/