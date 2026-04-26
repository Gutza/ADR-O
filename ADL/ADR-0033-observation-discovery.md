---
id: 33
type: Core vocabulary
status: Accepted
date: 2026-04-26
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0030
---

# ADR-0033 — Observed Outcomes as First-Class Facts with Discovery Types

## Context

**ADR-0030** introduced `adr-o:Observation` to close the learning loop. Its premise was:
1. A `DecisionRecord` at t₀ makes commitments via `ExpectedOutcome`s.
2. At tₙ, we need to record whether those commitments were met.
3. `Observation` was designed as a verification record: it `verifies` an `ExpectedOutcome` and has a `Verdict`.

This model works perfectly if we want to answer "did we get what we asked for?"

**The gap:** in practice, the learning loop produces two kinds of knowledge:
1. **Verification:** "We decided X, and we observed X (or not)" – this was correctly modeled by the verification pattern in ADR-0030;
2. **Discovery:** "We decided Y, and as a result, we now realize Z" – this is impossible to model.

Sometimes a `DecisionRecord` produces a realized effect that was **never expected** at t₀. It might be:
- **Deducible:** A logical consequence that could have been predicted at t₀ but wasn't considered until tₙ.
- **Emergent:** A practical consequence that only became visible at tₙ when the world reacted to the decision's effects.

Under ADR-0030, there is no place to put a discovered fact that doesn't verify an existing `ExpectedOutcome`. You cannot link a new decision's `Claim` back to a prior decision's `ExpectedOutcome` if that outcome was never specified. You are forced to either:
1. Point to the `DecisionRecord` directly (coarse, loses the "why").
2. Retroactively add an `ExpectedOutcome` to the old record (DTP violation).
3. Bury the discovery in prose.

This breaks the causality chain. The graph should be able to answer: *"What specific realized effect of ADR-0042 made this claim in ADR-0135 necessary?"*

## Decision

**Reframe `adr-o:Observation` as a general `*Fact` class that manifests a `Claim` outside the ADR scope, where verification of an expectation is an optional role, not a defining characteristic.**

**Naming adjustment:** rename `adr-o:Observation` to `adr-o:ObservedOutcome` as the canonical class name for this concept, for KISS and symmetry with `adr-o:ExpectedOutcome`.

### 1. Structural Alignment

`ObservedOutcome` now follows the same atom-first pattern as `ContextFact`, `DeliberationFact`, and `ExpectedOutcome`:

- **Domain:** `adr-o:ObservedOutcome`
- **Manifests:** Exactly one `adr-o:Claim` (via `adr-o:manifests`, functional)
- **Source:** Exactly one `adr-o:DecisionRecord` (via `adr-o:realizedFrom`, functional)
- **Provenance:** Optional `dcterms:date` and `prov:wasDerivedFrom` (evidence)

### 2. The Verification Role (Optional)

Verification is now a *role* an observed outcome may play, not its definition:

- `adr-o:verifies` → `adr-o:ExpectedOutcome` (optional, functional)
- `adr-o:hasVerdict` → `adr-o:ObservationVerdict` (optional, functional; requires `verifies` to be present)

### 3. The Discovery Dimension

Every `ObservedOutcome` must carry a `discoveryType` to be epistemically honest:

```ttl
adr-o:discoveryType rdf:type owl:ObjectProperty , owl:FunctionalProperty ;
                    rdfs:domain adr-o:ObservedOutcome ;
                    rdfs:range adr-o:DiscoveryType .

adr-o:DiscoveryType rdf:type owl:Class ;
                     rdfs:subClassOf skos:Concept .

adr-o:Expected    rdf:type owl:NamedIndividual , skos:Concept ;
                  skos:prefLabel "Expected"@en ;
                  skos:definition "The outcome was explicitly predicted at t₀."@en .

adr-o:Deducible   rdf:type owl:NamedIndividual , skos:Concept ;
                  skos:prefLabel "Deducible"@en ;
                  skos:definition "The outcome was a logical consequence of the decision, but not explicitly predicted."@en .

adr-o:Emergent    rdf:type owl:NamedIndividual , skos:Concept ;
                  skos:prefLabel "Emergent"@en ;
                  skos:definition "The outcome arose from interaction with the environment and was not predictable at t₀."@en .
```

### 4. Causal Integration

`ObservedOutcome` now serves as a valid anchor for ADL-scope causal predicates (ADR-0025/0032). A `Claim` in a later record can be:
- `adr-o:constrainedBy` an `ObservedOutcome`
- `adr-o:prohibitedBy` an `ObservedOutcome`
- `adr-o:recommendedBy` an `ObservedOutcome`
- ...etc.

This means the causality chain can now flow:
`DecisionRecord (t₀)` → `ObservedOutcome (tₙ)` → `Claim (tₙ₊₁)`

## Alternatives Considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| **Keep verification mandatory** | Forces retroactive DTP violations or loses discovered knowledge. |
| **New class `DiscoveredFact`** | Unnecessary class proliferation; discovery vs verification is a property of the observation, not a different kind of thing. |
| **Allow Claims to point to DecisionRecords** | Coarse; loses the ability to cite *which* effect of the decision is being relied upon. |
| **Allow `ExpectedOutcome` to be created at tₙ** | DTP violation; misrepresents a prediction as a reality. |

## Consequences

**Positive.**
- **Causality chain is complete:** `Claim` → `ObservedOutcome` → `DecisionRecord` is a valid, honest path for any discovery, expected or not.
- **Terminology is less surprising:** `ObservedOutcome` is explicit as a realized result and pairs naturally with `ExpectedOutcome`.
- **DTP preserved:** No retroactive editing of t₀ records.
- **Epistemic honesty:** The `discoveryType` explicitly distinguishes between "we forgot to expect this" and "we couldn't have expected this."
- **Structural consistency:** `ObservedOutcome` is now just the tₙ counterpart to `ExpectedOutcome`.

**Negative / risks.**
- `adr-o:verifies` and `adr-o:hasVerdict` are no longer required by the class definition; tooling must handle optionality.
- Teams must now choose a `discoveryType` — this is a new authoring obligation.

## References

- [ADR-0030](/ADL/ADR-0030-observations.md) — original observation model.
- [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) — DTP.
- [ADR-0025](/ADL/ADR-0025-causal-network.md) — causal topology.
- [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md) — `*Fact` pattern.
