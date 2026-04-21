---
id: 22
type: Core vocabulary
status: Accepted
date: 2026-04-20
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0021
---

# ADR-0022 — `dcterms:creator` / `adr-o:authoredBy` Coexistence: The Tooling-Mediated Expansion Pattern

## Context

ADR-0021 introduced four RACI-mapped role predicates — `adr-o:authoredBy`, `adr-o:decidedBy`, `adr-o:consulted`, `adr-o:informed` — while keeping `dcterms:creator` as a "compatibility layer." The relationship between `dcterms:creator` and `adr-o:authoredBy` was left as coexistence, with a scope note on the former pointing authors toward the latter. That stance was correct but incomplete: it did not resolve what tooling should do when only one is present, or what the canonical source of truth for the Responsible party is when both disagree.

The design tension that remains is classic: **metadata compatibility** (`dcterms:creator` — widely recognized, accepts literals, used on the ontology document itself) vs **domain precision** (`adr-o:authoredBy` — OWL object property, `prov:Agent` range, SPARQL-traversable). Four resolution options were evaluated:

1. **Tooling-Mediated Expansion ("Shorthand"):** `dcterms:creator` is the ingress predicate; tooling promotes it to `adr-o:authoredBy`.
2. **Strict Graph (Purist):** deprecate `dcterms:creator` entirely; require agent IRIs everywhere.
3. **Metadata-only Demotion:** reframe `dcterms:creator` as "the typist/scribe" and `adr-o:authoredBy` as "the accountable record owner," making them semantically distinct rather than redundant.
4. **Agent-Pivot (PROV-O heavy):** link `DecisionRecord` to a `prov:Activity`, which in turn carries role-attributed `prov:Agent` edges.

This ADR resolves which pattern ADR-O adopts as its normative stance.

## Decision

ADR-O adopts **Option 1: the tooling-mediated expansion pattern.** The duality between `dcterms:creator` and `adr-o:authoredBy` is not a bug; it is a deliberate **compatibility layer**.

### Role of each predicate

- **`dcterms:creator`** is the **ingress predicate**: the easy authorship surface for LLM-drafted records, for manual Turtle authors, for importers who carry DC metadata, and for the ontology document itself. It accepts literals and agent IRIs. It makes ADR-O records look like documents to the rest of the web.
- **`adr-o:authoredBy`** (and `adr-o:decidedBy`, `adr-o:consulted`, `adr-o:informed`) are the **canonical role-topology predicates**: graph-native, IRI-only, `prov:Agent`-ranged, SPARQL-traversable. They make ADR-O records look like a social graph to ADR-O tooling.

### The expansion rule

Whenever a `DecisionRecord` carries `dcterms:creator` but lacks `adr-o:authoredBy`, tooling is **expected to treat the creator as the authored-by agent** for all social-role queries. Concretely:

- If `dcterms:creator` is a literal, tooling should resolve or mint a corresponding `prov:Agent` IRI and assert `adr-o:authoredBy` as a materialized edge.
- If `dcterms:creator` is already an agent IRI, tooling should assert `adr-o:authoredBy` pointing to the same IRI.
- This materialization is a **tooling responsibility**, not a graph assertion; the raw store need not carry it persistently unless the authoring workflow persists it.

### Conflict rule

If both `dcterms:creator` and `adr-o:authoredBy` are present and point to different agents, **`adr-o:authoredBy` is authoritative** for social-role queries. `dcterms:creator` may still carry the bibliographic/scribe attribution (e.g. "Jane formatted and filed this record on behalf of Bob") without contradicting Bob's role as the record's Responsible party.

### Grounding in ADR-0004

This pattern is consistent with ADR-0004's reconstructability boundary rule: the `adr-o:authoredBy` edge is reconstructable from `dcterms:creator` by reasonable tooling (name/IRI matching, agent-minting conventions) and therefore need not always be persisted by the author. The tooling layer is the right place to enforce the promotion.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Option 2: Strict Graph (deprecate `dcterms:creator`) | High authoring friction; breaks ontology-document self-description; severs DC ecosystem compatibility; requires an agent-minting governance process not yet specified. |
| Option 3: Metadata-only Demotion (scribe vs owner) | The scribe/owner distinction is real in some enterprise contexts but adds a semantic layer that most teams do not recognise and that the existing ADR conventions do not model; would require a new dedicated sub-predicate (e.g. `adr-o:recordedBy`) rather than reusing DC. |
| Option 4: Agent-Pivot (PROV-O Activity) | Significantly increases triple count and query complexity; complicates simple "who authored this record?" lookups; introduces `prov:Activity` as a required intermediate node for basic authorship, which is not proportionate to the problem. |

## Consequences

**Positive.**
- Low authoring friction: both LLM-drafted and hand-authored records remain easy to produce without mandating agent IRIs upfront.
- Full social-role graph is available after tooling expansion; no information is lost.
- `dcterms:creator` retains its ecosystem compatibility role; ADR-O records remain good DC citizens.
- The conflict rule (`adr-o:authoredBy` wins on disagreement) ensures a clear canonical answer for tooling without requiring synchronization of the two predicates by authors.

**Negative / risks.**
- If tooling does not implement the expansion, raw `dcterms:creator`-only records remain opaque to social-role queries; this is the same risk that exists today, but it is now a named tooling obligation rather than an ambiguity.
- The conflict rule requires that authoring tools communicate clearly when `dcterms:creator` and `adr-o:authoredBy` disagree; silent divergence could cause confusion during record review.

**Open questions.**
- The agent-minting strategy (how tooling resolves a literal name to a `prov:Agent` IRI, including de-duplication and namespace conventions) is not specified here; a future ADR or profile convention should address it.

## References

- ADR-0004 — The KG Lives Under Tooling (reconstructability boundary rule).
- ADR-0015 — Canonical Dublin Core and PROV Usage (establishes `dcterms:creator` as the authorship predicate).
- ADR-0021 — Social Role Predicates (introduces `adr-o:authoredBy` and peers; defines the RACI mapping and the normative nuance on "Responsible" as record responsibility, not execution responsibility).
