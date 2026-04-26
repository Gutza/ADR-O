---
id: 32
type: Core vocabulary
status: Accepted
date: 2026-04-26
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0025
---

# ADR-0032 — Unified Voice and Naming for ADL-Scope Causal Predicates

## Context

ADR-0025 introduced a family of predicates to express how a `Claim` in a later decision is influenced by an `ExpectedOutcome` of a prior decision. In the current ontology, this family is:

```turtle
adr-o:constrainedBy    # Claim → ExpectedOutcome (MUST)
adr-o:prohibitedBy     # Claim → ExpectedOutcome (MUST NOT)
adr-o:recommends       # Claim → ExpectedOutcome (SHOULD)
adr-o:discourages      # Claim → ExpectedOutcome (SHOULD NOT)
adr-o:enabledBy        # Claim → ExpectedOutcome (MAY)
```

This has two problems:

**1. Morphological Collision**
The predicate `adr-o:enables` already exists as a Record→Record relationship (from ADR-0006). `adr-o:enabledBy` looks like its inverse, but it has a different domain (`Claim`) and range (`ExpectedOutcome`). They are semantically disjoint but linguistically identical, creating ambiguity for both humans and agents.

**2. Voice Inconsistency**
`recommends` and `discourages` use the active voice, implying the `Claim` is the actor. But in a causal chain, the `Claim` is the *recipient* of influence from the prior `ExpectedOutcome`. The current naming suggests the later decision tells the prior outcome what to do, which is logically reversed.

## Decision

Unify the ADL-scope causal family under a consistent passive voice, all with `Claim` as domain and `ExpectedOutcome` as range.

### The corrected family

| Modal | New Predicate | Reading |
|:---|:---|:---|
| **MUST** | `adr-o:constrainedBy` | (unchanged) |
| **MUST NOT** | `adr-o:prohibitedBy` | (unchanged) |
| **SHOULD** | `adr-o:recommendedBy` | "this claim is recommended by that outcome" |
| **SHOULD NOT** | `adr-o:discouragedBy` | "this claim is discouraged by that outcome" |
| **MAY** | `adr-o:permittedBy` | "this claim is permitted by that outcome" |

### Why this direction (Passive, not Active)

We could have flipped the direction (`Outcome` → `Claim`) and used active voice (e.g., `adr-o:discourages`). This would be logically equivalent but would violate the **Decision Transaction Principle (DTP)**.

Under DTP, a `DecisionRecord` is a frozen transaction at t₀. If we used `ExpectedOutcome` as the subject, we would be adding edges *from* past outcomes *to* future claims. This requires mutating old records to add new outgoing edges whenever a later decision recognizes a constraint.

By making the `Claim` the subject and the predicate passive (`discouragedBy`), the edge is asserted **in the current record**. The current decision recognizes it is being influenced by the past; the past record remains untouched. The graph encodes the *recognition* of the constraint, which is the honest epistemic fact.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| `adr-o:allows` / `adr-o:allowedBy` | `permittedBy` more precisely captures the "MAY" modal strength (permissible vs. enabled). |
| Flip direction (`Outcome` → `Claim`) | Violates DTP; requires mutating past records. |
| Keep `enabledBy` but fix voice | Still collides with `adr-o:enables` (Record→Record). |

## Consequences

**Positive.**
- Voice is consistent across the entire modal family.
- The `adr-o:enables` / `adr-o:permittedBy` collision is resolved.
- The graph direction aligns with the epistemic direction of recognition (present → past).
- SPARQL queries for "what constrains this claim" now use a consistent `*By` pattern.

**Negative / risks.**
- Breaking change for any graphs using the three renamed predicates.
- Requires migration of existing `recommends`, `discourages`, and `enabledBy` triples.

## References

- [ADR-0025](/ADL/ADR-0025-causal-network.md) — original causal topology.
- [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md) — `adr-o:enables` (Record→Record).
- [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) — DTP.

## Ontology Materialisation

```ttl
###  https://w3id.org/adr-o#recommendedBy
adr-o:recommendedBy rdf:type owl:ObjectProperty ;
                    rdfs:domain adr-o:Claim ;
                    rdfs:range adr-o:ExpectedOutcome ;
                    rdfs:comment "ADL scope: links a Claim to a prior decision's ExpectedOutcome that recommends it. Modal strength: SHOULD."@en ;
                    rdfs:label "recommended by"@en ;
                    adr-o:justifiedBy adr-odr:ADR-0032-unified-voice-and-naming .

###  https://w3id.org/adr-o#discouragedBy
adr-o:discouragedBy rdf:type owl:ObjectProperty ;
                     rdfs:domain adr-o:Claim ;
                     rdfs:range adr-o:ExpectedOutcome ;
                     rdfs:comment "ADL scope: links a Claim to a prior decision's ExpectedOutcome that discourages it. Modal strength: SHOULD NOT."@en ;
                     rdfs:label "discouraged by"@en ;
                     adr-o:justifiedBy adr-odr:ADR-0032-unified-voice-and-naming .

###  https://w3id.org/adr-o#permittedBy
adr-o:permittedBy rdf:type owl:ObjectProperty ;
                  rdfs:domain adr-o:Claim ;
                  rdfs:range adr-o:ExpectedOutcome ;
                  rdfs:comment "ADL scope: links a Claim to a prior decision's ExpectedOutcome that permits it. Modal strength: MAY."@en ;
                  rdfs:label "permitted by"@en ;
                  adr-o:justifiedBy adr-odr:ADR-0032-unified-voice-and-naming .

###  Remove from ontology:
# adr-o:recommends
# adr-o:discourages
# adr-o:enabledBy
```
