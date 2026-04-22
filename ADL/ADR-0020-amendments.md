---
id: 20
type: Core vocabulary
status: Accepted
date: 2026-04-20
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amendedBy:
  - ADR-0024
---

# ADR-0020 — Amendment Model: Predicates and the Rejection of "Amended" Status

## Context

ADR-0019 established the need for a unified "Amendment" model to capture "soft" changes—clarifications and modifications—that do not warrant a full supersession. While a supersession replaces a record with a new identity, an amendment layers a modification onto an existing anchor record.

The design challenge was to determine the most efficient way to signal these modifications to consuming tools (agents and developers) without introducing redundant metadata that would increase authoring friction and risk synchronization errors.

## Decision

**ADR-O will model amendments through a pair of symmetric object properties, while explicitly rejecting the introduction of a corresponding "Amended" status.**

### 1. The Relational Vocabulary
The following predicates are minted in the `https://w3id.org/adr-o#` namespace:

- **`adr-o:amends`** — domain `adr-o:DecisionRecord`, range `adr-o:DecisionRecord`.
- **`adr-o:amendedBy`** — the `owl:inverseOf` of `adr-o:amends`.

**Cardinality is arbitrary ($0..N$).** A single decision record can be amended by zero, one, or many other records. This allows the original record to remain the authoritative hub while accumulating a series of "patches" over time.

### 2. Rejection of the `Amended` Status
Despite the common pattern of updating a record's status to reflect its state, **no `adr-o:Amended` status will be added to the status scheme.** 

The rationale is based on the **Reconstructability Boundary Rule** (ADR-0004): if a piece of information can be reconstructed from the graph topology, it should not be stored as a redundant label.

In this case, there are two possible signals for a modification:
1. **The State Signal:** A status property (`adr-o:hasStatus adr-o:Amended`).
2. **The Topological Signal:** The existence of one or more edges (`adr-o:amendedBy ?mod`).

Because the Topological Signal is the only way to actually *find* the modifications, the State Signal is a redundant "dirty bit." Requiring the author (or an agent) to update the status of the anchor record every time a new amendment is filed introduces a synchronization risk without adding any operational value to the query.

## Operational Logic for Tooling

The responsibility for detecting amendments is pushed to the tooling layer. To retrieve the "Current State" of a decision, a tool should follow this logic:

1. **Filter for Validity:** Identify records where `status` is NOT `Superseded` or `Rejected` (e.g., `Accepted` or `Proposed`).
2. **Check for Patches:** Query for any records linked via `adr-o:amendedBy`.
3. **Merge & Render:** If amendments exist, the tool must fetch them and merge them with the anchor record to present the current, modified truth. If no amendments exist, the anchor record is rendered as-is.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| **Functional Amendment (1:1)** | Would force a "chain" of amendments (A → B → C), effectively turning amendments into a second supersession chain and losing the "hub-and-patch" model. |
| **Introduce `adr-o:Amended` status** | Redundant. It mirrors the topological signal and adds a write operation (updating the status of the anchor) without improving the retrieval of the actual amendment nodes. |
| **Use `rdfs:seeAlso`** | Too vague; fails to capture the directed, temporal nature of a modification as an evolutionary event. |

## Consequences

**Positive.**
- **Leaner Ontology:** Avoids predicate and status proliferation.
- **Zero Sync-Risk:** The status of a record does not need to be "toggled" when a new amendment is created; the graph remains the single source of truth.
- **Consistency:** Aligns with the "substrate" philosophy—trusting the topology over redundant labels.

**Negative / risks.**
- **Tooling Dependency:** Consumers who only look at the `status` property will miss amendments. This is an explicit trade-off: we assume the tooling layer is "ontology-aware" and checks for `amendedBy` edges by default.

## References

- ADR-0004 — The KG Lives Under Tooling (Reconstructability Boundary Rule).
- ADR-0006 — Core Relational Predicates (Supersession model).
- ADR-0008 — Status Scheme (SKOS pattern).
- ADR-0019 — Unified Amendment Model (Collapse of clarifications/amendments).
