---
id: 40
type: Core vocabulary
status: Accepted
date: 2026-04-28
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0040 — `Verdict`: Reifying the Convergence Act

## Context

The fish model (see [*Decisions Are Fish-Shaped*](/Archive/Decisions%20Are%20Fish-Shaped.md)) identifies several structural moments in a well-formed decision record: the snout (triggering complaint), the head (sharpened context), the body (divergent deliberation), the waist (convergence), the tail (expected outcomes), and the wake (the observed effects). ADR-0037 closed the snout gap by introducing `adr-o:Complaint`; this ADR closes the waist gap.

The ontology currently models the divergent phase thoroughly: `DeliberationFact` nodes with `deliberationValence` (*Supports*, *Against*, *Neutral*) represent the arguments weighed against each alternative, and `adr-o:chosenAlternative` records the winner — but there is nothing between these two elements. The graph can tell you which alternative was chosen and what arguments were made for and against each one; it cannot tell you *how the deliberation closed* — which considerations were decisive, what trade-offs were consciously accepted, why the balance tipped.

This gap is the machine-readable equivalent of the Y-statement's *"accepting that [F]"* clause going unrecorded. The Y-statement structure ([*Y-Statements*](/Archive/Y-Statements.md)) makes explicit that choosing an alternative is not just selecting a winner — it is a deliberate act of accepting specific costs, risks, and compromises. The current ontology has nowhere to put that act.

There is a temptation to close this gap formally, by adding structure to the deliberation layer: argument weights, decisiveness flags, or a selection-rationale reification that enumerates which `DeliberationFacts` drove the outcome. [*ADR-O Modeling Scope Priorities*](/Archive/ADR-O%20Modeling%20Scope%20Priorities.md) argues explicitly against that approach. ADR-O's primary job is efficient ADL-scope navigation; the interior of a single record is evidence and authored judgment, not a machine-executable syllogism. The ADR scope earns formal structure when it enables *locatability* — giving an agent a first-class node to find and read —, not when it attempts to encode the internal logic of a human deliberation.

The right response to the waist gap is therefore not a formal argumentation vocabulary. It is a **reified convergence act**: a first-class node that makes the moment of closure locatable and carries, in authored prose, the author's account of how the deliberation ended — including what was consciously accepted.

Here's the parallel with `Complaint` that governs the design:

- `Complaint` reifies the mouth of the fish — the raw triggering act, prior to any analytical sharpening. Its immutability is ontological: the complaint is constitutive of the decision record's identity. A changed complaint is not a revised record; it is a different fish entirely. There is no grace period tied to external reference — the anchor is set at the moment the complaint is named, because *that's what's being deliberated*.
- The new class `Verdict` reifies the waist of the fish — the convergence act, the moment deliberation collapses into commitment. Its immutability is epistemic: it is Tier 1 core, mutable during drafting, frozen on external reference per the [Decision Transaction Principle](/Archive/The%20Decision%20Transaction%20Principle.md). The tail can move without producing a new fish; only when the verdict produces effects — when another record cites this one, or implementation begins — must the convergence freeze.

Both the `Complaint` and the `Verdict` are **witnessed acts**, not verifiable propositions, so neither is backed by `Claim`. A `Claim` carries truth value and is subject to observation and verification (ADR-0031); a witnessed act of judgment does not. These are categorically distinct, and the distinction must be preserved in the ontology.

## Decision

Introduce **`adr-o:Verdict`** as a first-class class, and **`adr-o:closedBy`** as its functional access predicate on `DecisionRecord`.

### `adr-o:Verdict`

```turtle
adr-o:Verdict rdf:type owl:Class ;
    rdfs:label "Verdict"@en ;
    rdfs:comment "The reified convergence act of a decision: the moment deliberation closed and a commitment was made. Carries the author's account of how the balance tipped, including trade-offs consciously accepted."@en ;
    skos:scopeNote "Not a Claim: carries no truth value. Tier 1 epistemic core — mutable during drafting, frozen on external reference per the Decision Transaction Principle. The prose content (dcterms:description typed adr-o:md) is where the Y-statement 'accepting that' clause lives."@en ;
    adr-o:justifiedBy adr-odr:ADR-0040-verdict-reification .
```

### `adr-o:closedBy`

```turtle
adr-o:closedBy rdf:type owl:ObjectProperty ,
                        owl:FunctionalProperty ;
    rdfs:domain adr-o:DecisionRecord ;
    rdfs:range adr-o:Verdict ;
    rdfs:label "closed by"@en ;
    rdfs:comment "Links a DecisionRecord to the Verdict that closed its deliberation. Functional: one convergence act per decision."@en ;
    adr-o:justifiedBy adr-odr:ADR-0040-verdict-reification .
```

### Design notes

**`Verdict` is not a `Claim`,** per the distinction established in ADR-0031. A `Claim` is a verifiable atomic proposition: it has truth value, it can be placed into deliberation or outcome roles, and it is subject to verification against observations. A `Verdict` is a witnessed act of judgment: it names what happened at a specific moment, it is authored once and frozen, and it is not the kind of thing that can be confirmed or refuted by evidence. Collapsing the two would corrupt the epistemic architecture that makes the deliberation-to-observation chain coherent.

**`Verdict` does not subclass `prov:Entity`.** The PROV alignment test established in ADR-0035 and applied again in ADR-0037 requires three properties: fixed aspects (content immutable), attributable provenance (authored by someone), and a traceable lifecycle. A `Verdict` passes the first two but fails the third in a meaningful way: it has no lifecycle beyond its minting. Unlike `Complaint` (which has a source artifact — a ticket, a thread, an agenda item — traceable via `prov:wasDerivedFrom`) or `ObservedOutcome` (which has evidence and a tₙ timestamp), a `Verdict` is a pure authored act with no downstream lifecycle or external traceability requirement. The PROV alignment does not add value here and is omitted.

**`closedBy` is functional.** One deliberation, one convergence act. If a record's deliberation never converges — a `Proposed` record still under discussion, a `Rejected` record declined without deliberation closing — then `closedBy` is absent. The presence of `closedBy` is itself a machine-readable signal that deliberation reached a point of authored closure.

**The predicate direction follows the established convention.** Every structural predicate on `DecisionRecord` reads outward from the record as subject: `promptedBy`, `hasContext`, `hasDeliberation`, `chosenAlternative`, `hasExpectedOutcome`; `closedBy` follows this pattern. The `Verdict` does not "close" the record in a push sense; the record asserts that it was closed by this act.

**`closedBy` is named for neutrality.** The predicate name was chosen over `resolvedBy`, `settledBy`, and `concludedBy`. `resolvedBy` carries a positive valence — it implies the problem was satisfactorily solved — which is not guaranteed; a verdict is passed regardless of whether the outcome is good. `concludedBy` is too weak: a conclusion can be revisited; a closed deliberation cannot. `closedBy` says only that the open space is no longer open. It maps directly to the geometric intuition of the fish model's waist.

**Immutability regime.** The `Verdict` is Tier 1 epistemic core per the Decision Transaction Principle ([*The Decision Transaction Principle*](/Archive/The%20Decision%20Transaction%20Principle.md)): mutable during drafting, frozen when another record references this one or when implementation begins. This is categorically different from the `Complaint`'s regime. The `Complaint` is constitutive of the record's identity — a changed complaint is a different fish, with no grace period. The `Verdict` is the current state of convergence, which is genuinely provisional until it produces effects. The tail can move without producing a new fish. Once frozen, however, the `Verdict` cannot be edited in place: any change to the convergence act requires a new record that amends or supersedes the original.

**The prose is the content.** A `Verdict` carries a `dcterms:description` typed `^^adr-o:md`. This is where the Y-statement's *"accepting that [F]"* clause lives in machine-locatable form: the author's account of how deliberation closed, which arguments proved decisive, and what costs or risks were consciously accepted. It is not a formal encoding of the deliberation's logic; it is the authored record of an act of judgment.

### Instantiation: the MFA example (continued from ADR-0037)

```turtle
# The convergence act
:verdict-mfa rdf:type adr-o:Verdict ;
    dcterms:description """
We chose TOTP-based MFA via an authenticator app over SMS-based MFA and hardware tokens.
The deciding factor was the audit requirement's explicit preference for authenticator-app
approaches, which tipped the balance despite the higher onboarding friction. SMS was ruled
out on security grounds (SIM-swap risk). Hardware tokens were ruled out on cost and
logistics grounds.

We accept that some users will experience onboarding friction and that we will need
to invest in support documentation. We accept the dependency on third-party authenticator
apps for which we have no direct control over availability.
"""^^adr-o:md ;
    dcterms:created "2026-03-28"^^xsd:date .

# The decision record (extending the ADR-0037 example)
:adr-0042-mfa adr-o:closedBy :verdict-mfa ;
    adr-o:chosenAlternative :alt-totp .
```

The gap between the deliberation (which records arguments for and against each alternative) and the chosen alternative is now bridged by a locatable, immutable authored statement. An agent navigating the ADL can find the `Verdict`, read it, and understand not just *what* was chosen but *how the choice was made*.

## Alternatives considered

| Alternative | Reason not chosen |
|---|---|
| Add a `decisive` boolean or weight property to `DeliberationFact` | Addresses argument salience but not the convergence act itself. An agent could identify which arguments were flagged as decisive but still could not find the "accepting that" statement — the authored account of what was knowingly traded off. Adds complexity at the deliberation layer without closing the waist gap. |
| Reify a `SelectionRationale` that links back to specific decisive `DeliberationFacts` | Builds a formal argumentation sub-vocabulary inside the ADR scope, which [*ADR-O Modeling Scope Priorities*](/Archive/ADR-O%20Modeling%20Scope%20Priorities.md) explicitly argues against. The ADR scope earns structure for locatability, not for syllogism-encoding. Cross-linking back to `DeliberationFacts` crosses the line. |
| Extend `chosenAlternative` with an annotation for rationale prose | `chosenAlternative` is a structural pointer (record → alternative); attaching prose to the predicate itself has no natural OWL mechanism and would conflate the pointer with the convergence act. The two are semantically distinct. |
| Carry the rationale as a prose literal directly on `DecisionRecord` | Makes the convergence statement invisible to graph traversal: no IRI, no first-class node, no ability to assert immutability or distinguish the act from other prose on the record. This is exactly the gap the reification is designed to close. |
| Use `skos:note` or `dcterms:description` on `DecisionRecord` for the rationale | Same problem as the prose literal option — no first-class identity, no locatability guarantee, no structural signal to an agent that this particular prose is the convergence act rather than a general description. |

## Consequences

**Positive.**

- The waist of the fish is fully materialized. The summary table in *Decisions Are Fish-Shaped* recorded the waist gap as *partially covered*; `closedBy` → `Verdict` closes it completely alongside `chosenAlternative`.
- The Y-statement's *"accepting that [F]"* clause has a machine-locatable home. An agent can find the `Verdict` for any record and read the author's account of what was consciously traded off, without parsing prose scattered across the deliberation section.
- The `Complaint`/`Verdict` symmetry is complete. The two non-`Claim` witnessed acts that bookend the deliberation phase — the triggering situation at the mouth of the fish and the convergence act at the waist — are now both first-class graph nodes with the same epistemic type and parallel access predicates. Their immutability regimes differ in one respect: `Complaint` is unconditionally immutable from first commit; `Verdict` is Tier 1 epistemic core, frozen on external reference per the DTP.
- The presence of `closedBy` is a machine-readable signal of deliberation closure. Records without a `Verdict` are identifiably open or non-deliberative, without requiring status inspection.

**Negative / risks.**

- Adds one class and one object property to the core. The addition is targeted and closes a named gap; the operational risk is the same temporary validation lag as ADR-0037 until SHACL ships.
- Authors must understand that `Verdict` is not a summary of the deliberation but the act of closing it. The distinction matters for immutability: a deliberation summary might be updated as understanding improves; a convergence act cannot be. Tooling guidance and authoring conventions should reinforce this.
- `closedBy` is optional in the OWL model (no cardinality restriction on `DecisionRecord`). For records in `Accepted` or `Superseded` status, its absence is a data quality gap rather than a valid state. SHACL will enforce this when it ships; until then, it is an authoring convention.

## References

- [*Decisions Are Fish-Shaped*](/Archive/Decisions%20Are%20Fish-Shaped.md) — Structural account of the fish model; source of the waist gap this ADR closes.
- [*ADR-O Modeling Scope Priorities*](/Archive/ADR-O%20Modeling%20Scope%20Priorities.md) — Establishes that ADR scope earns structure for locatability, not syllogism-encoding; governs the choice of prose-carrying reification over a formal argumentation vocabulary.
- [*Y-Statements*](/Archive/Y-Statements.md) — The irreducible decision structure; the *"accepting that [F]"* clause is the primary content the `Verdict` carries.
- [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md) — Atom-first reification; establishes `Claim` as verifiable proposition, from which `Verdict` is explicitly distinguished.
- [ADR-0025](/ADL/ADR-0025-causal-network.md) — Three-scope causal architecture; the distribution of modeling complexity across scopes that makes the prose-carrying reification the right choice at ADR scope.
- [ADR-0031](/ADL/ADR-0031-rename-consideration-claim.md) — Establishes `Claim` as the verifiable proposition type; the `Verdict`/`Claim` distinction follows directly.
- [ADR-0035](/ADL/ADR-0035-observedoutcome-prov-entity.md) — Three-part PROV-O alignment test; applied here to justify *not* subclassing `prov:Entity` for `Verdict`.
- [ADR-0037](/ADL/ADR-0037-complaint-reification.md) — `Complaint` reification; the parallel design that governs `Verdict`'s epistemic type, freeze regime, and predicate naming.
- [*The Decision Transaction Principle*](/Archive/The%20Decision%20Transaction%20Principle.md) — Defines the Tier 1 / Tier 2 editability model and the external-reference freezing trigger that governs `Verdict`'s immutability.
