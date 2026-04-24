---
id: 21
type: Core vocabulary
status: Accepted
date: 2026-04-20
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amendedBy:
  - ADR-0022
  - ADR-0028
---

# ADR-0021 — Social Role Predicates for Decision Records (RACI-Mapped)

## Context

ADR-O currently treats authorship via `dcterms:creator` (clarified in ADR-0015), with optional literal or IRI values. This is sufficient for bibliographic metadata, but insufficient for accountability queries over the social structure of decisions.

In practice, a decision record usually involves at least four distinct social roles: who authored the record artifact, who had authority to decide, who was consulted, and who needed to be informed. Without explicit predicates for these roles, questions such as "who is accountable for this decision?" or "which teams were consulted before this class of decisions?" degrade into prose retrieval.

Horizon 2 in the roadmap identifies this as the highest-priority social-graph gap. The ontology needs explicit, queryable role predicates while remaining consistent with ADR-O's domain-specific naming style and OWL semantics.

## Decision

ADR-O introduces the following role predicates on `adr-o:DecisionRecord`:

- `adr-o:authoredBy`
- `adr-o:decidedBy`
- `adr-o:consulted`
- `adr-o:informed`

All four are modeled as `owl:ObjectProperty` and point to agent IRIs (recommended range: `prov:Agent`).

### RACI Mapping (explicit)

The four predicates map conceptually to RACI as follows:

- **Responsible (R)** -> `adr-o:authoredBy`
- **Accountable (A)** -> `adr-o:decidedBy`
- **Consulted (C)** -> `adr-o:consulted`
- **Informed (I)** -> `adr-o:informed`

### Normative nuance on "Responsible"

In ADR-O, `adr-o:authoredBy` captures **record responsibility**: authorship/accountability for the `DecisionRecord` artifact itself.

It does **not** necessarily assert **execution responsibility** for implementing the underlying decision in operational delivery. Implementing teams may overlap with authors, but the model does not assume identity between these responsibilities.

### Interoperability with existing metadata predicates

`dcterms:creator` remains the canonical metadata predicate for broad compatibility and may continue to appear on records. `adr-o:authoredBy` is the graph-native role predicate for traversable social-role queries in ADR-O.

Because these role predicates are object properties, they do not take literal values. Literal-person strings remain valid on `dcterms:creator` as legacy or lightweight metadata.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Use RACI-literal property names (`responsible`, `accountable`, `consulted`, `informed`) | Less aligned with ADR-O's existing domain-specific naming style and introduces avoidable ambiguity around "Responsible" as execution ownership. |
| Reuse only `dcterms:creator` and/or `prov:wasAttributedTo` | Captures attribution but not the full four-role social graph needed for accountability and stakeholder queries. |
| Allow mixed literal-or-IRI objects on the new role predicates | OWL-inconsistent if modeled as `owl:ObjectProperty`; object properties link resources, not literals. |

## Consequences

**Positive.**
- Makes social accountability first-class and SPARQL-traversable.
- Cleanly separates record authorship from decision authority.
- Preserves ADR-O's domain-specific vocabulary while documenting explicit RACI correspondence.
- Reduces semantic drift by documenting that ADR-O "Responsible" means record responsibility, not necessarily implementation ownership.

**Negative / risks.**
- Introduces additional authoring obligations; tooling should assist with role capture.
- If agent IRIs are not consistently minted, teams may fall back to literals on `dcterms:creator` and underuse the new graph edges.

## Post-acceptance ontology follow-up (exact changes)

Apply the following in `ontology/adr-o.ttl`:

1. Add declarations for `adr-o:authoredBy`, `adr-o:decidedBy`, `adr-o:consulted`, `adr-o:informed` as `owl:ObjectProperty`.
2. Set domain of each predicate to `adr-o:DecisionRecord`.
3. Set range to `prov:Agent` (or leave open with a scope note if preserving maximal flexibility is preferred).
4. Add `rdfs:comment`/`skos:scopeNote` text documenting:
   - explicit RACI mapping;
   - `authoredBy` as record responsibility, not execution responsibility;
   - object-property IRI-only expectation.
5. Update `dcterms:creator` scope note to clarify coexistence: metadata compatibility path vs graph-native role path.
6. Ensure README / roadmap wording for the social-graph item uses the finalized names and removes "literal-or-IRI" wording for object properties.

## References

- [`ontology/ROADMAP.md`](/ontology/ROADMAP.md) — Horizon 2.1 Social Graph.
- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — current metadata and object-property baseline.
- ADR-0004 — The KG Lives Under Tooling.
- ADR-0015 — Canonical Dublin Core and PROV Usage for ADR-O.
