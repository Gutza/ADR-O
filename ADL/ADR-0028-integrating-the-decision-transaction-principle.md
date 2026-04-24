---
id: 28
type: Core design
status: Accepted
date: 2026-04-23
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0005
  - ADR-0006
  - ADR-0016
  - ADR-0021
  - ADR-0025
  - ADR-0026
amendedBy:
  - ADR-0030
supersedes:
  - ADR-0018
---

# ADR-0028 - Integrating the Decision Transaction Principle

## Context

The **Decision Transaction Principle** establishes that a `DecisionRecord` is an epistemic transaction: it represents the closed state of deliberation at the moment of commitment ($t_0$). The boundary between $t_0$ (what was decided) and $t_n$ (what later became true) must be impermeable.

An analysis of the current ADL and ontology revealed several points where this boundary was potentially porous or where the ontology risked encouraging retrospective "smearing" of information. This record resolves those tensions.

## Decision

We adopt the following resolutions to align the ADL and ontology with the Transaction Principle:

### 1. `dcterms:version` is restricted to Tier 2 changes

The `dcterms:version` predicate (introduced in ADR-0016) may only be used for **Tier 2 (Presentation Layer)** changes: typos, grammar, formatting, and title clarifications. Any change to the **Tier 1 (Epistemic Core)**, including decisions, premises, alternatives, or outcome commitments, must be recorded as a new `DecisionRecord` that `adr-o:amends` the original. Using version bumps to silently modify the decision logic is a violation of the ledger's integrity.

### 2. Operational risks are practitioner/tooling responsibilities

The risks identified in the real-time and post-factum authoring models established in ADR-0005, specifically **hindsight bias** in reconstructed records and **draft-smearing** in real-time instrumentation, are recognized as operational risks. The ontology does not introduce artificial constraints to prevent these; instead, they are flagged as concerns for:

- Practitioners to manage through discipline (for example, explicit "reconstructed from" notes).
- Tooling to mitigate through UI/UX (for example, warnings when `dcterms:date` is much earlier than `dcterms:created`).

### 3. Record classification is a practitioner concern

The ambiguity of whether retrospective changes to `adr-o:hasType` or `adr-o:hasStatus` constitute an epistemic change is deferred to the practitioner. The ontology remains unopinionated on this axis, treating record classification as a metadata concern rather than a structural transaction.

### 4. Project-scope provenance is unidirectional: Resource → Decision

A `DecisionRecord` is a statement of intent, not an agent of change. It cannot know its own effects in the world at $t_0$. Therefore:

- **Delete `adr-o:affects`** (introduced in ADR-0006)
- **Delete `adr-o:materializes`** (introduced in ADR-0025)

The only valid Project-scope bridge is **`adr-o:justifiedBy`** from resource to decision. This is the only honest direction: an artifact can claim justification from a transaction, but a transaction cannot claim to have materialized an artifact. "What does this decision affect?" becomes a **graph query**, not a predicate.

### 5. `adr-o:justifiedBy` provenance is sufficient

The pattern where a resource points to a superseded ADR via `adr-o:justifiedBy` (as established in ADR-0026) is accepted as correct. The fact that a resource was justified by a decision that was *later* superseded is a historical fact, not a contradiction. The provenance chain remains honest by preserving the link to the decision that existed at the moment of the resource's creation.

### 6. Inter-ADR `adr-o:Consideration` reuse is changed to reference-based

ADR-0018's convention of reusing the same `adr-o:Consideration` IRI across records (identity reuse) creates a transaction-boundary vulnerability: editing a shared atom silently mutates all decisions that manifest it, allowing retrospective rewrite of premises without a new `DecisionRecord`.

**We replace inter-record identity reuse with reference reuse:**

- Every `DecisionRecord` manifests its own set of `Consideration` individuals (one IRI per record, per claim).
- When a consideration is reused across records, the new record mints a **distinct IRI** with the same (or slightly adapted) content.
- The relationship to the original is captured via a new pair of predicates: **`adr-o:derivedFrom`** with inverse **`adr-o:derives`** (`Consideration` ↔ `Consideration`).
- The `adr-o:derivedFrom`/`adr-o:derives` chain may be traversed to find the original authoring context or the derivatives of the original, but each local copy remains stable and isolated within its own record's transaction.

**Crucially, this restriction does not apply at ADR scope.** Identity reuse within a single `DecisionRecord` remains permitted and encouraged—it is the mechanism for intra-record coherence (see Section 7).

### 7. ADR Scope Identity: The Coherence Property

A `DecisionRecord` is a closed transaction. Within that transaction, the same `Consideration` IRI may be placed into different roles. This is not a DTP violation; it is a **[design claim](/Archive/Consideration%20Reification%20-%20A%20Design%20Choice.md)**.

ADR-O maps the Y-statement clauses to typed graph roles:
- **Facing:** A need or constraint stated as the desired outcome (a `Consideration` in `ContextFact`, e.g. need: "answer all calls within 30 seconds", as opposed to a vague specification like "we're taking too long to answer the phone").
- **To achieve:** A goal or desired state (a `Consideration` in `ExpectedOutcome` with `adr-o:ExpectedGain` valence).

The Y-statement format states both clauses as prose strings. ADR-O goes further: when these two roles manifest the **same `Consideration` IRI**, the graph asserts: *"We decided for X because it satisfies the exact need we were facing."* This is a structural coherence check — the machine-verifiable proof that a decision addresses its trigger — that has no counterpart in any Y-statement format or template.

If a decision produces a benefit that was *not* the original need, the `ExpectedOutcome` manifests a **different** `Consideration` (e.g. outcome: "answer all calls within 60 seconds"). The graph honestly records a decision that mitigates the issue without fully resolving it.

**The ontology must preserve this distinction.** Collapsing `Consideration` into `Fact` would destroy the ability to distinguish between:
- "This decision solved the problem it set out to solve" (identity reuse)
- "This decision solved a different problem than it set out to solve" (no identity reuse)

This is why `Consideration` remains a first-class class. The indirection is where the methodology lives.

### 8. `OutcomeFact` becomes `ExpectedOutcome`

A deep dive into `adr-o:OutcomeFact` revealed an epistemic smear: the class combined $t_0$ predictions with $t_n$ realizations, violating the DTP.

**We resolve this by:**

- **Renaming** `adr-o:OutcomeFact` to `adr-o:ExpectedOutcome`.
- **Renaming** `adr-o:hasOutcome` to `adr-o:hasExpectedOutcome`.
- **Rebalancing the valence scheme** to ensure consistent "expectation" voice:
  - `adr-o:Benefit` → `adr-o:ExpectedGain`
  - `adr-o:AcceptedCost` → `adr-o:ExpectedCost`
  - `adr-o:Risk` → `adr-o:ExpectedRisk`
  - `adr-o:FollowUp` → `adr-o:ExpectedDependency`

All expected outcomes are now $t_0$ claims about predicted effects. Realized outcomes are explicitly out of scope for the ontology.

## Ontology Materialisation

The following predicates are introduced to implement the referential reuse pattern:

```ttl
###  https://w3id.org/adr-o#derivedFrom
adr-o:derivedFrom rdf:type owl:ObjectProperty ;
                  owl:inverseOf adr-o:derives ;
                  rdfs:domain adr-o:Consideration ;
                  rdfs:range adr-o:Consideration ;
                  rdfs:comment "A Consideration is derived from another Consideration. This preserves semantic connectivity between records while ensuring each record's local copy remains immutable under the Decision Transaction Principle."@en ;
                  rdfs:label "derived from"@en ;
                  adr-o:justifiedBy adr-odr:ADR-0028-integrating-the-decision-transaction-principle .


###  https://w3id.org/adr-o#derives
adr-o:derives rdf:type owl:ObjectProperty ;
              rdfs:domain adr-o:Consideration ;
              rdfs:range adr-o:Consideration ;
              rdfs:comment "A Consideration serves as the source from which another Consideration is derived."@en ;
              rdfs:label "derives"@en ;
              adr-o:justifiedBy adr-odr:ADR-0028-integrating-the-decision-transaction-principle .
```

**Expected Outcome refactoring:**

```ttl
###  https://w3id.org/adr-o#ExpectedOutcome
adr-o:ExpectedOutcome rdf:type owl:Class ;
                     rdfs:comment "A reified link placing a Consideration into the expected results of a DecisionRecord. An ExpectedOutcome represents a claim made at the moment of decision about what the chosen alternative is intended to produce (ExpectedGain), what costs are knowingly accepted (ExpectedCost), what risks are acknowledged (ExpectedRisk), or what subsequent work is triggered (ExpectedDependency). Within a DecisionRecord's hasExpectedOutcome list, an ExpectedOutcome represents a t0 commitment, not a tn observation."@en ;
                     rdfs:label "Expected Outcome"@en ;
                     adr-o:justifiedBy adr-odr:ADR-0028-integrating-the-decision-transaction-principle .


###  https://w3id.org/adr-o#hasExpectedOutcome
adr-o:hasExpectedOutcome rdf:type owl:ObjectProperty ,
                          owl:FunctionalProperty ;
                  rdfs:domain adr-o:DecisionRecord ;
                  rdfs:range rdf:List ;
                  rdfs:comment "Links a DecisionRecord to an ordered list of ExpectedOutcomes — the commitments made at the moment of decision regarding intended benefits, accepted costs, acknowledged risks, and triggered follow-ups. This is the structural landing place for the \"to achieve\" and \"accepting that\" clauses of a Y-statement."@en ;
                  rdfs:label "has expected outcome"@en ;
                  adr-o:justifiedBy adr-odr:ADR-0028-integrating-the-decision-transaction-principle .


###  https://w3id.org/adr-o#outcomeValence
adr-o:outcomeValence rdf:type owl:ObjectProperty ,
                      owl:FunctionalProperty ;
                 rdfs:domain adr-o:ExpectedOutcome ;
                 rdfs:range adr-o:ExpectedOutcomeValence ;
                 rdfs:comment "The intended valence of an ExpectedOutcome: ExpectedGain, ExpectedCost, ExpectedRisk, or ExpectedDependency. This is a t0 classification of the commitment, not a tn classification of a result."@en ;
                 rdfs:label "outcome valence"@en ;
                 adr-o:justifiedBy adr-odr:ADR-0028-integrating-the-decision-transaction-principle .


###  https://w3id.org/adr-o#ExpectedOutcomeValence
adr-o:ExpectedOutcomeValence rdf:type owl:Class ;
                           owl:equivalentClass [ owl:intersectionOf ( skos:Concept
                                                                  [ rdf:type owl:Restriction ;
                                                                    owl:onProperty skos:inScheme ;
                                                                    owl:hasValue adr-o:expectedOutcomeValenceScheme
                                                                  ]
                                                                ) ;
                                              rdf:type owl:Class
                                            ] ;
                           rdfs:subClassOf skos:Concept ;
                           rdfs:comment "A valence for an ExpectedOutcome, capturing the nature of the t0 commitment."@en ;
                           rdfs:label "Expected Outcome Valence"@en ;
                           adr-o:justifiedBy adr-odr:ADR-0028-integrating-the-decision-transaction-principle .


###  https://w3id.org/adr-o#expectedOutcomeValenceScheme
adr-o:expectedOutcomeValenceScheme rdf:type owl:NamedIndividual ,
                                                 skos:ConceptScheme ;
                                  dcterms:title "Expected Outcome Valence Scheme"@en ;
                                  rdfs:comment "The controlled vocabulary of expected outcomes at t0. All valences are claims about what the decision is expected to produce; none are guarantees."@en ;
                                  skos:hasTopConcept adr-o:ExpectedGain ,
                                                     adr-o:ExpectedCost ,
                                                     adr-o:ExpectedRisk ,
                                                     adr-o:ExpectedDependency .
```

The following predicates are **removed** from the ontology:

```ttl
adr-o:affects       # Removed: violated DTP by allowing DecisionRecord → Resource
adr-o:materializes  # Removed: violated DTP by allowing DecisionRecord → Resource
adr-o:OutcomeFact   # Renamed to ExpectedOutcome
adr-o:hasOutcome    # Renamed to hasExpectedOutcome
```

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Allow `dcterms:version` for Tier 1 updates | This would enable silent mutation of committed deliberation and violate transaction boundary integrity. |
| Encode anti-bias constraints directly in ontology axioms | These concerns are operational and context-sensitive; hard ontology constraints would overfit process behavior. |
| Require retroactive rewrite of `adr-o:justifiedBy` links after supersession | This would erase historical provenance and produce dishonest post-hoc cleanup. |
| `owl:sameAs` or IRI reuse for shared considerations | Enables silent retrospective mutation of decision premises; violates the DTP. |
| Versioned `Consideration` nodes | Introduces substantial complexity to the atom layer; `derivedFrom`/`derives` reference reuse provides sufficient connectivity with simpler semantics. |
| **Collapse `Consideration` into `*Fact` nodes** | Destroys intra-record coherence. Without a shared atom, you cannot structurally assert that a benefit satisfies a specific need. You're left with two prose strings and a hope they mean the same thing. |
| Keep `affects` / `materializes` as forward links | Directly violates the DTP; decisions cannot speak about effects in the world. |
| Use `justifies` as inverse of `justifiedBy` | Still risks authors using it as a "what happens" link; the query-layer solution is the only honest one. |
| Retain `OutcomeFact` name | The "Outcome" name encourages $t_n$ interpretations; the DTP requires an explicit $t_0$ commitment. |
| Create separate `RealizedOutcome` | Out of scope; the ADL records deliberations, not measurements. |
| Keep `FollowUp` as a separate predicate | `FollowUp` is an expected outcome of the decision; separating it creates an inconsistent voice. |
| Use `expectedOutcomeValence` as predicate | Redundant; the domain `ExpectedOutcome` already conveys the modality. |

## The Near-Miss

During the drafting of this ADR, the team seriously considered whether the `Consideration` node was over-engineering. The argument was: *"If we only do referential reuse across records, why not just put prose on Facts?"*

This was almost accepted. But the intra-record coherence analysis revealed that the `Consideration` node carries a specific, non-redundant semantic load: it is the only way to express that a **benefit is the satisfaction of a need**.

The near-miss serves as a reminder: the ontology's indirection is there to make the implicit methodology of a Y-statement explicit and queryable.

## Consequences

**Positive.**
- The ledger's integrity is strengthened by strictly bounding the use of `dcterms:version`.
- Every `DecisionRecord` has a stable, immutable set of premises; no record can be silently modified by editing a shared atom.
- **ADR scope identity** enables machine-verifiable decision coherence: the same `Consideration` IRI appearing in both a `ContextFact` and an `ExpectedOutcome` is a structural assertion that the decision addresses the exact need it was triggered by.
- The ontology remains lean by refusing to encode operational behaviors as structural axioms.
- Provenance chains remain honest rather than being "cleaned" retrospectively.
- Semantic connectivity between related considerations is preserved via `adr-o:derivedFrom`/`adr-o:derives`.
- The ontology removes predicates (`affects`, `materializes`) that encouraged DTP violations, forcing "what is affected" to be a query over the graph rather than a property of the decision.
- **Outcome vocabulary is epistemically honest:** by renaming `OutcomeFact` to `ExpectedOutcome` and rebalancing the valences, the ontology explicitly models commitments rather than results, closing the $t_0/t_n$ smear.
- **Consistency of voice:** all four expected outcome valences now share a predictive modality, making the graph's claims uniform.

**Negative / risks.**
- Practitioners must have the discipline to use `amends` rather than version bumps for core changes; without SHACL enforcement, this is a social convention.
- Tooling must handle the "copy-and-link" pattern for consideration reuse rather than simple IRI reuse.
- Graph volume increases slightly due to duplicated considerations (one per record).
- The rename of `OutcomeFact` to `ExpectedOutcome` is a breaking change for any existing instances.

## References

- **Decision Transaction Principle** - established in the project's thought experiment on Y-statement round-trips.
- [ADR-0005](/ADL/ADR-0005-log-all-decisions.md) - real-time and post-factum authoring.
- [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md) - introduced `affects` (now removed).
- [ADR-0016](/ADL/ADR-0016-dcterms-version-in-record.md) - `dcterms:version`.
- [ADR-0018](/ADL/ADR-0018-consideration-iri-reuse-convention.md) - original IRI reuse convention (now amended by this record).
- [ADR-0021](/ADL/ADR-0021-social-role-predicates.md) - social role predicates.
- [ADR-0025](/ADL/ADR-0025-causal-network.md) - introduced `materializes` (now removed).
- [ADR-0026](/ADL/ADR-0026-ontology-provenance.md) - `adr-o:justifiedBy`.
- [Y-Statements](/Archive/Y-Statements.md) - the prior-art format whose clause structure ADR-O maps to typed graph roles.
- [Consideration Reification: A Design Choice](/Archive/Consideration%20Reification%20-%20A%20Design%20Choice.md)
