---
id: 13
type: Core vocabulary
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0013 — `addresses`: Canonical Consideration Tagging vs Record Rollup

## Context

`adr-o:addresses` links to `adr-o:Concern` (problem side). Fine-grained tagging often belongs on a specific `Consideration`; discovery and browsing sometimes benefit from a coarse tag at the `DecisionRecord` level. The ontology must allow both without forcing duplicate modelling strategies for future constructs (for example GADR-style templates).

## Decision

**Single property `adr-o:addresses`**, range **`adr-o:Concern`**, with **intentionally unrestricted domain** in OWL so both of the following are valid:

1. **Canonical:** `adr-o:Consideration` — “this atom is about concern C.”
2. **Editorial rollup:** `adr-o:DecisionRecord` — “this record is broadly about concern C” for discovery when a finer tag is unnecessary or redundant.

When **both** are present for the same concern, the **consideration-level** use is **authoritative** for precision; the record-level use is a permitted convenience. This is stated in the `rdfs:comment` on `adr-o:addresses` in [`ontology/adr-o.ttl`](/ontology/adr-o.ttl).

Under ADR-0004’s **reconstructability boundary rule**, this dual placement sits on both sides intentionally: fine-grained concern attachment belongs in-graph as first-class structure on `Consideration`, while record-level concern rollup is treated as an optional convenience signal that reasonable tooling can compute or reconcile from the atom layer.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Domain restricted to `DecisionRecord` only | Cannot tag atoms; loses fine-grained `addresses` on `Consideration`. |
| Two separate properties | Extra IRIs and author confusion for the same semantic relation to a `Concern`. |

## Consequences

**Positive.** One predicate for SPARQL over “what concerns appear anywhere on this record” with optional precision at the atom layer.

**Negative / risks.** Duplicate or conflicting tags if tooling does not reconcile rollup with atom-level data; documented precedence reduces ambiguity.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `adr-o:addresses`.
- ADR-0006 — Core relational predicates (`addresses` vs `affects`).
- ADR-0002 — Scope (`Concern` as abstract anchor).
- ADR-0004 — The KG Lives Under Tooling (reconstructability boundary rule).
