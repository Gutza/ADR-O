---
id: 10
type: Core design
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0010 — Atom-First Reification: `Consideration` and `*Fact` Link Classes

## Context

A decision record needs structured support for rationale, alternatives, and outcomes. Section-level reification (mirroring Nygard body slots as containers) gives boxes without stable identity for claims. Atom-first design instead uses reusable **consideration** nodes with stable IRIs, placed into roles by **reified link** individuals so the same atom can appear in framing, deliberation, and outcome with different annotations (valence, ordering, optional alternative link).

RDF-star or named-graph-per-statement were avoided to keep consumption on plain RDF 1.1 and OWL 2 without a quad or RDF-star toolchain dependency.

## Decision

**`adr-o:Consideration`** — class of reusable atomic claims or observations (prose and labels carried via standard predicates; see ADR-0017 for Markdown scope).

**Three reified link classes** (parallel shape, distinct roles):

- **`adr-o:ContextFact`** — places a `Consideration` in *framing* (no alternative, no valence).
- **`adr-o:DeliberationFact`** — places a `Consideration` in *deliberation* against an `Alternative`, with `adr-o:deliberationValence` (see ADR-0009).
- **`adr-o:OutcomeFact`** — places a `Consideration` in *outcome*, with `adr-o:outcomeValence` (see ADR-0009).

**Ordering:** each `adr-o:DecisionRecord` has up to three **`rdf:List`**-valued functional properties — `adr-o:hasContext`, `adr-o:hasDeliberation`, `adr-o:hasOutcome` — whose members are intended to be the respective `*Fact` individuals, in author order. List membership typing is not enforced in OWL DL; SHACL is deferred (ADR-0004).

**`adr-o:consideration`** links each fact to exactly one `Consideration` (`owl:FunctionalProperty`).

The class name **`Consideration`** was chosen over overloaded alternatives (e.g. `Force`, `Argument`) to read neutrally across positive, negative, and framing uses.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Single generic `Fact` class with a “phase” property | Branching cardinality and query intent; three classes keep phase as a class fact. |
| RDF-star for annotations | Toolchain and spec surface area; reified nodes remain plain RDF. |
| Section-level only | Fails stable identity for reusable claims across roles and records. |

## Consequences

**Positive.** Same `Consideration` IRI can be linked from multiple facts and records (YADR-style identity). SPARQL-friendly lists for narrative order.

**Negative / risks.** `rdf:List` is awkward for OWL DL reasoning over membership; primary access is SPARQL and tooling, per ADR-0004.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `adr-o:Consideration`, `adr-o:ContextFact`, `adr-o:DeliberationFact`, `adr-o:OutcomeFact`, `adr-o:hasContext`, `adr-o:hasDeliberation`, `adr-o:hasOutcome`, `adr-o:consideration`.
- ADR-0004 — The KG Lives Under Tooling.
- ADR-0009 — Deliberation and outcome valence schemes.
