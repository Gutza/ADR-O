---
id: 26
type: Core vocabulary
status: Accepted
date: 2026-04-22
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amendedBy:
  - ADR-0028
---

# ADR-0026 — Ontology Provenance: The `adr-odr` Namespace

## Context

ADR-O is a decision log about its own design, but its ontology file (`adr-o.ttl`) contains only the *results* of those decisions—the vocabulary and axioms. There is currently no machine-readable link between a term in the ontology and the ADR that justifies its existence or shape.

This creates a knowledge gap: a consumer encountering `adr-o:chosenAlternative` sees a functional property but cannot traverse to the reasoning in ADR-0012.

We need a mechanism that:
1. Links ontology terms to their justifying ADRs.
2. Remains consistent with the Schema vs. Data boundary (no embedding the ADL in the ontology).
3. Provides stable, dereferenceable IRIs for the ADRs themselves.
4. Avoids namespace collisions between ADR-O vocabulary and ADR-O's own decision records.

## Decision

**1. Apply `adr-o:justifiedBy` to the ontology terms.**

The `adr-o:justifiedBy` predicate, established in **ADR-0025** as the Project scope causal link from a materialized resource back to its decision, is applied here to the ontology's own terms. An ontology term is a resource that was "materialized" by a design decision; therefore, it is naturally the subject of `adr-o:justifiedBy`.

Every `adr-o:` term and axiom in the ontology should carry at least one `adr-o:justifiedBy` triple pointing to the ADR (or ADRs) that established it.

**2. Establish the `adr-odr` namespace for ADR-O Decision Records.**

```turtle
@prefix adr-odr: <https://w3id.org/adr-o/ADL/> .
```

The name `adr-odr` stands for **ADR-O Decision Record**. This separates the ontology's *vocabulary* (`adr-o:`) from its *instances* (`adr-odr:`). A consumer can use the `adr-o:` namespace for their own decisions without colliding with the decisions that shaped the ontology itself.

**3. Use w3id redirects to make ADR IRIs live.**

The `adr-odr:` namespace is mapped via w3id.org to the GitHub Pages rendering of the ADL:
`https://w3id.org/adr-o/ADL/ADR-0001-license` → `https://gutza.github.io/ADR-O/ADL/ADR-0001-license`

This ensures that following a `justifiedBy` edge leads directly to the human-readable rationale, rendered as HTML, bypassing the GitHub UI wrapper.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Mint a separate `adr-o:justifiedBy` as an annotation property | Redundant; the domain predicate from ADR-0025 already captures the "resource → justification" relationship. |
| Embed ADR content in ontology | Category error (schema vs. data); versioning cascade; breaks domain-agnosticism. |
| Use GitHub `blob` URLs | Branch-dependent; fragile; wraps content in GitHub UI; not stable IRIs. |
| No citation | Leaves the ontology as an "oracle" without evidence; violates the project's core value of traceability. |

## Consequences

**Positive.**
- The ontology becomes a **navigable evidence chain**.
- Consumers can discover the *why* behind any term by following a link.
- The `adr-odr` namespace prevents collisions between the ontology's history and consumers' data.
- w3id redirects provide stable, human-friendly access to the ADL.
- Using the same predicate for both system artifacts and ontology terms reinforces the causal model's universality.

**Negative / risks.**
- Requires discipline to keep citations in sync when ADRs are amended or superseded.
- w3id redirect configuration is an external dependency (mitigated by the fact that w3id is the industry standard for this purpose).

## References

- [ADR-0025](/ADL/ADR-0025-causal-network.md) — establishes `adr-o:justifiedBy` as a causal predicate.
- [ADR-0000](/ADL/ADR-0000-inception.md) — established the intent for traceability.
- [ADR-0002](/ADL/ADR-0002-scope.md) — domain-agnosticism requires a clean boundary.
- [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md) — the KG as substrate, tooling as mediator.
- [ADR-0005](/ADL/ADR-0005-log-all-decisions.md) — the ADL as a first-class knowledge asset.