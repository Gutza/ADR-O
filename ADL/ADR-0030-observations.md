---
id: 30
type: Core vocabulary
status: Accepted
date: 2026-04-24
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0028
---

# ADR-0030 — Observations: Closing the Learning Loop

## Context

ADR-0028 established the **Decision Transaction Principle (DTP)**: a `DecisionRecord` is an epistemic transaction frozen at $t_0$. This preserves the integrity of the record by preventing retrospective smearing. However, this creates a secondary problem: the ADL now records what was *intended* at $t_0$ (via `ExpectedOutcome`), but has no first-class mechanism to record what was *observed* at $t_n$.

Without a way to record observations, the ADL is a ledger of commitments that can never be reconciled with reality. The "learning delta"—the gap between expectation and realization—remains trapped in prose or exists nowhere at all. This renders the ADL useful for indexing history, but useless for organizational learning.

The challenge is to introduce observation records without:
1. Violating the DTP by retroactively mutating decisions.
2. Turning the ADL into a telemetry sink.
3. Encouraging the ADL to become a substitute for a project's general knowledge base.

### The Telemetry Tom Problem

Consider this scenario:
> A team uses ADR-O to record a decision about optimizing a SQL-heavy web page. They agree on a success metric: sub-100ms per page. They close the ADR with a `ExpectedOutcome` of `ExpectedGain` on latency, and agree to keep monitoring the performance after the fix is implemented.
>
> The next day, Tom—a well-intentioned developer—implements the fix. He also chooses a fancy way to monitor the performance: he wires application logging to attach `adr-o:Observation` triples to the project's ADL **every time someone loads that page**.
>
> When asked why he did that, Tom is quite wide-eyed: *"Isn't that what we agreed on? Aren't these legitimate observations? What did I do wrong?"*

Tom is technically correct: each page load *is* an observation. But his approach would fill the ADL with millions of telemetry events, drowning the decision graph in noise and transforming a knowledge system into a log aggregator.

## Decision

**ADR-O introduces `adr-o:Observation` as a first-class class, but defines it as a *verdict*, not an *event*.**

### 1. The Ontology

The ontology surface is kept deliberately slim. Only constructs that enable traversal queries unavailable through existing vocabularies are minted as first-class ADR-O terms.

#### Tier 1 — Core (minted in ADR-O)

| Term | Kind | Rationale |
|------|------|-----------|
| `adr-o:Observation` | Class | Closes the learning loop; no existing class carries the verdict/verification semantics. |
| `adr-o:verifies` | ObjectProperty | The epistemic bridge from $t_n$ back to an `ExpectedOutcome`. |
| `adr-o:hasVerdict` | FunctionalObjectProperty | Makes the graph actionable — enables the "which outcomes are Violated?" query. |
| `adr-o:ObservationVerdict` + `adr-o:observationVerdictScheme` | SKOS class + scheme | Controlled vocabulary required for the query above to work reliably. |

#### Tier 2 — Deferred to Existing Vocabularies

| Removed ADR-O term | Replacement | Predicate |
|--------------------|-------------|-----------|
| `adr-o:observedAt` | Dublin Core | `dcterms:date` (when of observation) or `dcterms:created` (when record was authored) |
| `adr-o:evidence` | PROV-O / DC | `prov:wasDerivedFrom` (derived from a document), `dcterms:references` (cited source), or `rdfs:seeAlso` (informational link) |

These are well-understood, widely-implemented predicates already present in the ontology's prefix declarations. Minting project-specific synonyms adds surface area without enabling new SPARQL traversal patterns.

```ttl
### https://w3id.org/adr-o#Observation
adr-o:Observation rdf:type owl:Class ;
                  rdfs:subClassOf [ owl:onProperty adr-o:verifies ;
                                    owl:someValuesFrom adr-o:ExpectedOutcome ] ;
                  rdfs:comment "A verification transaction that evaluates one or more ExpectedOutcomes. An Observation is a discrete epistemic moment (review, test run, monitoring window, or audit) that yields a verdict."@en ;
                  rdfs:label "Observation"@en ;
                  adr-o:justifiedBy adr-odr:ADR-0030-observations .

### https://w3id.org/adr-o#verifies
adr-o:verifies rdf:type owl:ObjectProperty ;
               rdfs:domain adr-o:Observation ;
               rdfs:range adr-o:ExpectedOutcome ;
               rdfs:comment "Links an Observation to the ExpectedOutcome it evaluates. A single observation may verify multiple outcomes."@en ;
               rdfs:label "verifies"@en ;
               adr-o:justifiedBy adr-odr:ADR-0030-observations .

### https://w3id.org/adr-o#hasVerdict
adr-o:hasVerdict rdf:type owl:ObjectProperty ,
                          owl:FunctionalProperty ;
                 rdfs:domain adr-o:Observation ;
                 rdfs:range adr-o:ObservationVerdict ;
                 rdfs:comment "The epistemic conclusion of an Observation. Functional: exactly one verdict per observation."@en ;
                 rdfs:label "has verdict"@en ;
                 adr-o:justifiedBy adr-odr:ADR-0030-observations .

### https://w3id.org/adr-o#ObservationVerdict
adr-o:ObservationVerdict rdf:type owl:Class ;
                         owl:equivalentClass [ owl:intersectionOf ( skos:Concept
                                                                  [ rdf:type owl:Restriction ;
                                                                    owl:onProperty skos:inScheme ;
                                                                    owl:hasValue adr-o:observationVerdictScheme
                                                                  ] ) ;
                                              rdf:type owl:Class ] ;
                         rdfs:subClassOf skos:Concept ;
                         rdfs:comment "The verdict of an Observation."@en ;
                         rdfs:label "Observation Verdict"@en ;
                         adr-o:justifiedBy adr-odr:ADR-0030-observations .

### https://w3id.org/adr-o#observationVerdictScheme
adr-o:observationVerdictScheme rdf:type owl:NamedIndividual ,
                                        skos:ConceptScheme ;
                               dcterms:title "Observation Verdict Scheme"@en ;
                               skos:hasTopConcept adr-o:Satisfied ,
                                                  adr-o:Violated ,
                                                  adr-o:Inconclusive .

adr-o:Satisfied    rdf:type owl:NamedIndividual , skos:Concept ;
                   skos:inScheme adr-o:observationVerdictScheme ;
                   skos:prefLabel "Satisfied"@en .

adr-o:Violated     rdf:type owl:NamedIndividual , skos:Concept ;
                   skos:inScheme adr-o:observationVerdictScheme ;
                   skos:prefLabel "Violated"@en .

adr-o:Inconclusive rdf:type owl:NamedIndividual , skos:Concept ;
                   skos:inScheme adr-o:observationVerdictScheme ;
                   skos:prefLabel "Inconclusive"@en .
```

### 2. The "Telemetry Tom Test"

To prevent the ADL from becoming a telemetry sink, authoring tools and reviewers apply the **Telemetry Tom Test** to every proposed `Observation` triple:

> **Can this `Observation` exist without a human (or agent) making an epistemic judgment?**
>
> - If the answer is *"Yes, I can just emit this from a log file automatically,"* then it is **telemetry**, not an observation. **Reject.**
> - If the answer is *"No, this requires someone to look at data and decide whether the ExpectedOutcome was met,"* then it is an **Observation**. **Accept.**

Telemetry emits *data*. Observations emit *verdicts*. An `Observation` is only valid if it contains a verdict (`adr-o:hasVerdict`) that evaluates an `ExpectedOutcome`.

### 3. DTP Alignment

Observations are **new transactions at $t_n$**. They do not modify `DecisionRecords`.

```turtle
# The decision remains frozen
:ADR-0042  adr-o:hasExpectedOutcome [
             adr-o:manifests :cons-latency-under-100ms ;
             adr-o:outcomeValence adr-o:ExpectedGain ] .

# The observation is a new, separate transaction
:obs-2026-06-15 a adr-o:Observation ;
                adr-o:verifies :expected-outcome-latency ;
                adr-o:hasVerdict adr-o:Satisfied ;
                dcterms:date "2026-06-15"^^xsd:date ;
                prov:wasDerivedFrom <https://monitoring.example.org/report/123> .
```

## Alternatives Considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| **No observations in ADL** | Leaves the learning loop open; supersessions have no explicit trigger. |
| **Add outcome-realized status to `DecisionRecord`** | Retrospective smearing; violates DTP. |
| ** observations as `Consideration` nodes** | Conflates a prediction with a verification; destroys the $t_0/t_n$ distinction. |
| ** observations as generic `prov:Entity`** | Lacks the specific semantics of verification and verdict. |

## Consequences

**Positive.**
- **Learning loop closed:** We can now query *"Which ExpectedOutcomes are violated?"* to identify when decisions need revisiting.
- **DTP preserved:** No $t_n$ information ever enters a $t_0$ record.
- **Telemetry protected:** The "verdict" requirement and "Tom Test" provide a structural and social guard against ADL bloat.
- **Supersession grounded:** A `supersedes` relationship can now be explicitly triggered by an `Observation` with `adr-o:Violated` verdict.
- **Slim ontology surface:** By deferring observation date to `dcterms:date` and evidence provenance to `prov:wasDerivedFrom` / `dcterms:references`, ADR-O mints only what is strictly needed for verdict-based traversal. No project-specific synonyms for well-understood DC/PROV predicates.

**Negative / risks.**
- `Observation` is a new class that authoring tools must support.
- The distinction between telemetry and observation requires human/agent judgment (though the "verdict" requirement helps).
- Authors must know to reach for `dcterms:date` and `prov:wasDerivedFrom` rather than having ADR-O aliases guide them; documentation and tooling templates must make this explicit.

## References

- [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) — the Decision Transaction Principle.
- [ADR-0025](/ADL/ADR-0025-causal-network.md) — `justifiedBy` as the only Project-scope bridge.
- [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md) — `ExpectedOutcome` and outcome valences.