---
id: 37
type: Core vocabulary
status: Accepted
date: 2026-04-27
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0037 — `Complaint`: Modeling the Pre-Analytical Triggering Situation

## Context

The document ([*Decisions Are Fish-Shaped*](/Archive/Decisions%20Are%20Fish-Shaped.md)) makes a structural claim about decision records: every well-formed record has a *snout* — the raw, pre-analytical triggering situation that caused deliberation to begin. It arrives in ordinary language, at ordinary resolution:

> *"Implementing MFA"*  
> *"The report export takes too long"*  
> *"CI is too slow"*

The methodology document that predates the fish framing ([*ADR-O Methodology From First Principles*](/Archive/ADR-O%20Methodology%20From%20First%20Principles.md)) correctly identified that these raw statements are *complaints, not decision contexts*, and argued that recording a complaint as the "Context" of a record is a category error:

> *"A complaint is an observation of a symptom, not a decision context. When we record a complaint as the 'Context' of an ADR, we have already surrendered."*

That argument stands. The complaint does not belong in the head of the fish — the `ContextFact` that manifests the primary facing `Claim`. The head requires a sharpened, falsifiable need: *"Deployments must complete in under ten minutes to support five releases per day"*, not *"CI is too slow."* The translation from complaint to need is real work, and collapsing the two is a methodological failure.

However, the methodology document drew the wrong conclusion from the right observation. It treated the complaint as something to be *discarded after translation* — a scaffolding that vanishes once the real context is articulated. The fish shape reveals that this discard destroys information.

The complaint is not just a rough draft of the head. It is a structurally distinct element with its own epistemic role: it is the **cause of existence** of the `DecisionRecord`. Without it, there is no meeting, no ticket, no conversation — no fish. It names the situation, in ordinary language, as perceived by a specific stakeholder at a specific moment. That perception is the primary evidence that a real organizational pain point, felt by real people, warranted a decision record at all.

This has a further implication that the methodology document overlooked: the person who names the complaint tends to be the primary stakeholder (or at least they know who it is) — the one who felt the pain strongly enough to call the meeting, open the ticket, or initiate the deliberation. Discarding the complaint discards this stakeholder identity along with it.

The current ontology has no vocabulary for the snout. The summary table in *Decisions Are Fish-Shaped* records this gap explicitly: the ADR-O model column for the snout reads *n/a*. Every other part of the fish is materialized — head, body, convergence, tail, wake — but the record's cause of existence is invisible to the graph.

## Decision

Introduce **`adr-o:Complaint`** as a first-class class, and **`adr-o:promptedBy`** as its functional access predicate on `DecisionRecord`, together with **`adr-o:namedBy`** for stakeholder provenance on the `Complaint` itself.

### `adr-o:Complaint`

```turtle
adr-o:Complaint rdf:type owl:Class ;
    rdfs:subClassOf prov:Entity ;
    rdfs:label "Complaint"@en ;
    rdfs:comment "The raw, pre-analytical triggering statement that caused a DecisionRecord to exist. Named by a stakeholder at a specific moment, in ordinary language, before any sharpening or translation into a falsifiable need."@en ;
    skos:scopeNote "A Complaint is not a Claim: it carries no truth value and is not verifiable. It is the cause of existence of the DecisionRecord, not a proposition within it. A Complaint is unconditionally immutable from the moment the DecisionRecord is first committed to the ADL — if the Complaint changes, it is a different fish. The stakeholder who names the Complaint is typically the primary pain-holder whose concern motivates the deliberation."@en ;
    adr-o:justifiedBy adr-odr:ADR-0037-complaint-reification .
```

### `adr-o:promptedBy`

```turtle
adr-o:promptedBy rdf:type owl:ObjectProperty ,
                          owl:FunctionalProperty ;
    rdfs:domain adr-o:DecisionRecord ;
    rdfs:range adr-o:Complaint ;
    rdfs:label "prompted by"@en ;
    rdfs:comment "Links a DecisionRecord to its Complaint — the singular, unconditionally immutable triggering situation that caused the record to exist. Functional by normative intent: exactly one Complaint per record."@en ;
    adr-o:justifiedBy adr-odr:ADR-0037-complaint-reification .
```

### `adr-o:namedBy`

```turtle
adr-o:namedBy rdf:type owl:ObjectProperty ;
    rdfs:domain adr-o:Complaint ;
    rdfs:range prov:Agent ;
    rdfs:label "named by"@en ;
    rdfs:comment "The agent who named the triggering situation and thereby initiated the DecisionRecord. Typically the primary stakeholder whose pain motivated the deliberation. Distinct from adr-o:authoredBy (who wrote the record) and adr-o:decidedBy (who is accountable for the decision)."@en ;
    adr-o:justifiedBy adr-odr:ADR-0037-complaint-reification .
```

### Design notes

**`Complaint` is not a `Claim`.** A `Claim` is a verifiable atomic proposition with truth value under observation. A `Complaint` is pre-semantic: it names a situation as perceived, not a proposition that can be confirmed or refuted. The two must remain categorically separate or the boundary between the snout and the head collapses.

**`Complaint` subclasses `prov:Entity`.** ADR-0035 established the test for this alignment: *every instance of the subclass must satisfy the superclass semantics — fixed aspects, attributable provenance, and traceable lifecycle.* `Complaint` passes all three. Fixed aspects: the raw text is unconditionally immutable, an even stronger fixedness guarantee than `ObservedOutcome` carries. Attributable provenance: `namedBy` provides attribution and `dcterms:created` provides the timestamp, mirroring the `observedBy` + `observedAt` pattern on `ObservedOutcome`; `prov:wasDerivedFrom` can point at the source artifact (ticket, thread, agenda item) without new predicates. Lifecycle: a `Complaint` has a degenerate but valid lifecycle — minted once, never modified — which satisfies the constraint both technically and in spirit (it's not an anonymous, forever malleable entity). The cross-record identity coupling risk that kept `Claim` out of scope does not apply here: a `Complaint` is explicitly singular to its record by design, and the immutability regime prevents the semantic drift that made aligning `Claim` risky.

**`promptedBy` is functional:** one fish, one snout — a `DecisionRecord` has exactly one `Complaint`. If two distinct triggering situations are present, there are two fish; the second fish's complaint is the moment of recognition that a separate decision was needed.

**Immutability regime.** The `Complaint`'s immutability is prior to and stricter than the Tier 1 freeze that applies to the rest of the record. The head, body, convergence, and tail freeze on external reference. The `Complaint` is immutable from first commit, unconditionally, because a changed complaint means a different conversation — a different fish — not a revised record of the same one.

**`namedBy` is distinct from the existing social-role predicates.** `authoredBy` (RACI: Responsible for the record artifact), `decidedBy` (RACI: Accountable), `consulted`, and `informed` all operate on the `DecisionRecord`. `namedBy` operates on the `Complaint` and captures a different role: the stakeholder who perceived and articulated the pain, who may or may not be the author or decision-maker (although they are very likely going to be at the very least informed about the decision, their actual RACI role is not deterministic at design time, and it's separate from the implicit stakeholder role asserted by `namedBy`).

### Instantiation: the MFA example

```turtle
# The stakeholder who felt the pain
:agent-cso rdf:type prov:Agent ;
    rdfs:label "Chief Security Officer" .

# The snout
:complaint-mfa rdf:type adr-o:Complaint ;
    dcterms:description "Implementing MFA"^^<https://www.w3.org/ns/iana/media-types/text/markdown> ;
    dcterms:created "2026-03-14"^^xsd:date ;
    adr-o:namedBy :agent-cso .

# The head (sharpened need — a Claim, not the Complaint)
:claim-mfa-need rdf:type adr-o:Claim ;
    dcterms:description "All user-facing services must require a second authentication factor to comply with the new ISO 27001 audit requirement by Q3 2026."^^<https://www.w3.org/ns/iana/media-types/text/markdown> .

:ctx-mfa-need rdf:type adr-o:ContextFact ;
    adr-o:manifests :claim-mfa-need .

# The decision record
:adr-0042-mfa rdf:type adr-o:DecisionRecord ;
    dcterms:title "MFA implementation approach" ;
    adr-o:index 42 ;
    adr-o:hasStatus adr-o:Accepted ;
    adr-o:decidedAt "2026-03-28"^^xsd:date ;
    adr-o:promptedBy :complaint-mfa ;
    adr-o:hasContext ( :ctx-mfa-need ) ;
    adr-o:authoredBy :agent-cso ;
    adr-o:decidedBy :agent-cto .
```

The gap between `"Implementing MFA"` (Complaint) and the sharpened need in the `ContextFact` is now named, queryable, and preserved. A SPARQL query across the ADL can surface the raw complaint and the translated need for every record, making the quality of the translation — or its absence — visible.

## Alternatives considered

| Alternative | Reason not chosen |
|---|---|
| Plain datatype property (`adr-o:primeMover` as `xsd:string` on `DecisionRecord`) | Loses provenance: no way to attach who named the complaint, when, or from which source artifact. The stakeholder identity — the primary motivation for first-class modeling — is discarded. |
| Fold into `ContextFact` | Reintroduces the conflation the methodology document correctly identified: the complaint is not the head, it is prior to the head. Mixing them collapses the snout into the head and destroys the fish's diagnostic value. |
| `adr-o:PrimeMover` as class name | Accurate but carries Aristotelian/scholastic baggage inappropriate for a domain-agnostic ontology intended for use across HR, finance, strategy, and software contexts. |
| `adr-o:Trigger` / `adr-o:triggeredBy` as predicate | Accurate causally; rejected because "triggered" carries a second, well-established meaning in mental health discourse that would create friction in the HR and organizational decision-making domains this ontology is explicitly designed to serve. |
| Make `Complaint` the subject of `adr-o:triggers` (complaint → record direction) | Inverts the navigation convention: every other structural predicate on `DecisionRecord` reads outward from the record as subject. Reversing direction for the snout alone would be an inconsistency without compensating benefit. |
| Skip `prov:Entity` subclassing; keep `Complaint` as a bare class | Leaves the snout as a floating fact without attributable provenance, inconsistent with the standard ADR-0035 established for `ObservedOutcome`. `Complaint` passes the three-part PROV test (fixed aspects, attributable provenance, degenerate but valid lifecycle) and the cross-record identity risk that excluded `Claim` does not apply. Skipping the alignment would be the weaker move without a compensating reason. |

## Consequences

**Positive.**

- The fish is fully materialized. Every structural element identified in *Decisions Are Fish-Shaped* now has an ontological home; the *n/a* in the model column of that document's summary table is closed.
- The complaint-to-need translation gap is machine-queryable. Tooling can surface records where the complaint and the primary `ContextFact` `Claim` are present, making translation quality visible at ADL scale.
- The primary stakeholder is a first-class graph node. `namedBy` on `Complaint` captures a social role that was previously invisible: the person who felt the pain, independently of who authored the record or who made the decision.
- PROV traceability extends to the snout. `Complaint` as `prov:Entity` — a subclassing that passes the three-part test established in ADR-0035 — allows source artifact IRIs (tickets, threads, agenda items) to be attached via `prov:wasDerivedFrom` without new predicates, and positions the snout as a fully traceable provenance artifact alongside `DecisionRecord` and `ObservedOutcome`.
- The methodology document's anti-complaint argument is honored, not reversed. Complaints remain excluded from the deliberation structure; they now have their own place before it.

**Negative / risks.**

- Adds one class and two object properties to the core. Given that the snout is the only unmodeled part of the canonical fish shape, the addition is warranted; the operational risk is temporary validation lag until SHACL is implemented. Normatively, `promptedBy` is exactly-one for every well-formed record.
- `namedBy` introduces a fourth social-role predicate alongside `authoredBy`, `decidedBy`, `consulted`, and `informed`. Authors must understand the distinction. The scope notes on `namedBy` make this explicit; tooling guidance should reinforce it.

## References

- [*Decisions Are Fish-Shaped*](/Archive/Decisions%20Are%20Fish-Shaped.md) — The structural account of the snout as prime mover; source of the *n/a* gap this ADR closes.
- [*ADR-O Methodology From First Principles*](/Archive/ADR-O%20Methodology%20From%20First%20Principles.md) — The original anti-complaint argument.
- [ADR-0002](/ADL/ADR-0002-ontology-scope.md) — Domain-agnostic scope; informs the naming choices that avoid domain-specific connotations.
- [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md) — Atom-first reification; establishes `Claim` as verifiable proposition, from which `Complaint` is explicitly distinguished.
- [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md) — Dublin Core and PROV-O usage; `Complaint` subclasses `prov:Entity` consistently with `DecisionRecord` and `ObservedOutcome`.
- [ADR-0021](/ADL/ADR-0021-social-role-predicates.md) — Social role predicates; `namedBy` is a fourth distinct social role not covered by `authoredBy`, `decidedBy`, `consulted`, or `informed`.
- [ADR-0035](/ADL/ADR-0035-observedoutcome-prov-entity.md) — `ObservedOutcome` as `prov:Entity`; establishes the three-part test (fixed aspects, attributable provenance, traceable lifecycle) used here to justify the same alignment for `Complaint`.
