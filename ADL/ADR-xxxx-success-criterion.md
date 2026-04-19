# Architecture Design Discussion — Design-Time Measurability: `adr-o:successCriterion` as a Direct Datatype Property

| Field        | Value |
|--------------|-------|
| ID           | TBD |
| Type         | Core vocabulary |
| Status       | Proposed |
| Date         | 2026-04-18 |
| Author       | Bogdan Stăncescu <bogdan@moongate.ro> |

## Context

ADR-O exists to close the organizational knowledge chasm between what was decided and why. MADR — the most widely deployed human-facing ADR template — includes an optional Confirmation section: a slot for the author to record how the implementation or compliance with a decision can be verified. Assessed through a software-development lens, this maps naturally to CI gates, ArchUnit constraints, and automated fitness functions; assessed that way, it is clearly outside ADR-O's scope.

Reframed for the domain-agnostic case the scope decision (ADR-0002) establishes, the concept resolves to something different and considerably more valuable: **a design-time measurability hypothesis**. When an HR team restructures a performance-review process, a finance team moves to rolling forecasts, or a logistics team adopts a new distribution model, the team has at the moment of decision a concrete belief about how they will recognize success — retention scores, forecast variance, delivery-time variance. That belief is as much a decision artifact as the rationale or the list of rejected alternatives. Without a record of it, the lessons-learned session two years later can assess whether the outcome was good but cannot assess whether the team's prediction model was good. The second is the more transferable organizational learning.

This is squarely within decision recording. A design-time measurability hypothesis is recorded at authoring time, expresses what the team believed at the moment of decision, and has no dependency on any external verification infrastructure. The question before this ADR is therefore not whether to add the concept but how.

### The `FollowUp` gap

The existing `OutcomeValence` scheme (`Benefit`, `AcceptedCost`, `Risk`, `FollowUp`) does not cover this. `FollowUp` is triggered work: a successor task, a future ADR, an ongoing concern to monitor. A design-time measurability hypothesis is an epistemic commitment about observability — not work to be done, but a stated belief about what to watch and how to interpret it. The semantic distinction is real: a `FollowUp` says "we need to do X"; a success criterion says "we will know this decision is working if Y." Collapsing the two loses the distinction that makes introspective audits and lessons-learned sessions tractable.

### Naming: `SuccessCriterion` vs `Hypothesis`

`Hypothesis` is the more epistemically precise label — it acknowledges that the stated criterion is a prediction, not a guarantee, and it preserves epistemic humility about whether the chosen measure will in fact track what matters. `SuccessCriterion` is the more operationally grounded label — it implies a concrete, actionable criterion and is the vocabulary domain practitioners in HR, finance, and logistics actually use (KPIs, acceptance criteria, success metrics). The practical argument for `SuccessCriterion` outweighs the epistemological argument for `Hypothesis`: the concept is most useful when it is legible to domain practitioners without translation, and those practitioners already have `SuccessCriterion` in their vocabulary.

## Decision

**Add `adr-o:successCriterion` as a non-functional datatype property on `adr-o:DecisionRecord`.**

A single record may carry multiple success criteria in prose (one per distinct observable, each in one or more language variants). The property hangs directly off `DecisionRecord` rather than being routed through the `OutcomeFact` / `Consideration` machinery.

### Why a direct datatype property rather than a `Consideration` node

A success criterion is a natural-language statement of measurement intent: "we will know this decision is working if retention scores improve by 10% over two annual cycles." Its value to the graph is that it exists and is attributable to a specific decision — not that it participates in structural relationships with other nodes. A success criterion does not `address` a `Concern`, does not have a deliberation valence, is not weighed against an `Alternative`, and does not reappear in a different role within the same record or across other records. It is decision-specific by nature: the measurability hypothesis for an HR review restructuring is not an atom anyone reuses verbatim in a finance or logistics decision.

The choice between a direct literal on `DecisionRecord` and a `Consideration` node reduces to whether IRI identity adds value. Both carry prose as plain-text literals; the only thing a `Consideration` node adds is an IRI. That IRI is valuable when the same content needs to be recognizably the same thing across records, across roles within a record, or as the subject or object of further predicates. None of those conditions hold for success criteria. An IRI here would have nowhere meaningful to go, and the authoring overhead of minting one is not justified.

### Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Fifth `OutcomeValence` concept (`SuccessCriterion`) in `outcomeValenceScheme`, content carried by a `Consideration` node via an `OutcomeFact` | The atom-first reification pattern is motivated by Consideration reuse across records and roles. Success criteria are decision-specific by nature: the criterion for an HR review restructuring is not the kind of atom anyone reuses verbatim in a different decision. The reification overhead — minting an `OutcomeFact` IRI, a `Consideration` IRI, wiring `consideration` and `outcomeValence` and `onAlternative` — is not justified when no reuse benefit materialises. |
| Dedicated `SuccessCriterion` subclass of `Consideration` with its own predicate | Same reification-overhead objection; additionally introduces a class whose sole distinguishing axiom would be "a Consideration that is a success criterion", which adds vocabulary without adding expressiveness. |
| Captured as `FollowUp`-valenced `OutcomeFact` | Conflates triggered work with epistemic commitment about observability; loses the semantic distinction documented in the Context section above. |
| No dedicated slot; captured in prose inside other Consideration nodes | Violates the atom-first principle at a higher level: if the measurability hypothesis is buried in the prose of a `Benefit` or `FollowUp` Consideration, it is not retrievable as a distinct claim. The whole point of the query classes in the outreach article is that organizational knowledge should be directly traversable, not reconstructed from prose. |

## Consequences

**Positive.**
- Design-time measurability hypotheses are now first-class, directly queryable on `DecisionRecord` nodes without traversing reification chains.
- Internationalization is provided by construction: the same criterion can be expressed in multiple languages as independent tagged literals, with no additional modelling required.
- The authoring story is simple: a single triple per criterion per language variant.
- The concept is domain-agnostic and immediately legible to HR, finance, logistics, and software practitioners alike.
- Introspective audits and lessons-learned sessions gain a graph-traversable anchor: "retrieve all decisions in this category together with what we said we'd measure" is now a SPARQL query, not a text search.

**Negative / risks.**
- The property is a plain-text literal, so the measurability criteria themselves cannot be further structured (e.g., linked to a `Concern` via `addresses`, given a metric IRI, or made the subject of a `Consideration` reuse). Teams that want richer structure for their success criteria will need to express that structure in an `OutcomeFact` / `Consideration` node alongside the `successCriterion` value, accepting that the two representations are not automatically reconciled.
- Introduces a second pattern for outcome-related content (datatype property alongside the `OutcomeFact` / `Consideration` machinery), which may cause authoring inconsistency if the principle distinguishing the two patterns (terminal prose vs. structurally decomposed claims) is not clearly communicated.

**Open questions.**
- Should the planned SHACL companion recommend or require at least one `successCriterion` on records with `hasStatus = adr-o:Accepted`? The case for recommending is strong (a decision accepted without a stated measurability criterion is a gap in organizational knowledge), but mandating it may be too strict for domain profiles whose notion of measurability is informal.
- Should `adr-o:successCriterion` be scoped to `DecisionRecord` only, or should a future GADR-style `DecisionTemplate` class inherit or share it? The property's domain is currently `DecisionRecord`; if a template class is introduced, its relationship to this property should be settled at that time.

## References

- ADR-0002 — Ontology Scope (this ADL). Establishes the domain-agnostic mandate; the domain-agnostic reframing of MADR's Confirmation slot is the proximate motivation for this decision.
- ADR-O DESIGN-NOTES, 0.2.0-draft section — "Fitness functions / confirmation predicates" deferral; "Design assumption: the KG lives under tooling"; the terminal-vs-structural distinction applied here extends the principle stated there.
- MADR template — the Confirmation section is the human-facing analogue of this property.
- Kopp & Armbruster, GADR (2019) — the `FollowUp` valence gap analysis draws on GADR's separation of generic decision knowledge from project-specific outcome recording.
