---
id: 19
type: Core vocabulary
status: Accepted
date: 2026-04-20
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0019 — Unified Amendment Model: Collapsing Clarifications and Amendments

## Context

Existing ADR-O relational predicates handle total replacement via the `adr-o:supersedes` / `adr-o:supersededBy` chain. However, real-world decision evolution often involves "soft" changes: a later record that either clarifies the original intent without changing the decision, or amends a specific layer of the decision without replacing the entire record.

In prior design discussions, a distinction was made between **Amendments** (which modify the decision) and **Clarifications** (which merely add interpretive detail). While semantically distinct, the key question is whether this distinction is operationally useful in a machine-readable graph.

From a query perspective, a consumer wishing to understand the "Current State" of a decision must retrieve all linked modifications regardless of whether they are "clarifications" or "amendments." Separating them into two predicates would require a union query for every "Current State" retrieval, adding complexity to SPARQL and the ontology without providing a corresponding increase in traversal power or an actionable label for automation.

## Decision

**ADR-O will collapse "Clarifications" and "Amendments" into a single unified concept: the Amendment.**

There will be no separate predicate or status for clarifications. Any record that provides additional detail, modifies a part of a prior decision, or updates the interpretation of a record is an amendment.

### Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| **Distinct `amends` and `clarifies` predicates** | Adds a fine-grained nuance that generates complexity (and potentially confusion) without providing operational value for graph traversal; increases the number of predicates (and the number of decisions to be made at authoring time) without optimizing "Current State" queries. |
| **Use `rdfs:seeAlso` for clarifications** | Too vague; fails to capture the directed, temporal nature of a clarification as an evolutionary event in the decision's lifecycle. |
| **Treat all as `supersedes`** | Too blunt; supersession implies the original record is no longer in force. Amendments allow a record to remain the "anchor" while carrying a chain of modifications. |

## Consequences

**Positive.**
- **Simplified Query Logic:** Tooling can retrieve the "evolutionary delta" of a decision using a single predicate rather than a union of multiple "soft-link" predicates.
- **Reduced Cognitive Load:** Authors do not need to decide if their record is a "clarification" or an "amendment"—a distinction that is often subjective and depends on the reader's perspective.
- **Leaner Ontology:** Avoids predicate proliferation by focusing on the operational outcome (the need to load a related record) rather than the semantic nuance.

**Negative / risks.**
- **Loss of Precision:** The graph no longer explicitly signals whether a linked record *changes* the decision or merely *explains* it. This nuance must be recovered from the prose in the `Consideration` nodes.

## References

- ADR-0006 — Core Relational Predicates (established the launch of `supersedes`).
- ADR-0008 — Status Scheme (established the SKOS pattern for status individuals).
- Roadmap Horizon 3 (identified the gap for fine-grained evolution).
