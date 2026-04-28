---
id: 17
type: Core vocabulary
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amendedBy:
  - ADR-0031
  - ADR-0038
---

# ADR-0017 — Markdown Prose Properties as Implemented in ADR-O 0.2.1

## Context

ADR-0003 commits prose-carrying literals to the IANA `text/markdown` datatype and listed several properties in scope. After the 0.2 atom-first shape landed, `ContextFact`, `DeliberationFact`, and `OutcomeFact` are **structural placement nodes** without their own prose properties: prose is reached via `adr-o:consideration` → `adr-o:Consideration`. Separately, `skos:prefLabel` behaves as a **short label** on `Alternative` and `Consideration`, not as long-form Markdown prose.

This ADR **narrows and clarifies** ADR-0003 for the current ontology **without** replacing ADR-0003 as the original datatype decision.

## Decision

**In-scope for Markdown typing** (SHOULD use `^^<https://www.w3.org/ns/iana/media-types/text/markdown>` for prose payloads, with enforcement deferred to the planned SHACL companion per ADR-0003):

- **`dcterms:description`** — on `DecisionRecord` (summary) and `Consideration` (primary prose payload); also the ontology’s own `dcterms:description` on the ontology IRI.
- **`skos:definition`** — formal definitions on `Consideration` and SKOS individuals where prose.
- **`skos:note`** and its **sub-properties** (`skos:scopeNote`, `skos:editorialNote`, etc.) — supplementary annotation prose on `Consideration` where used.

**Out of scope for Markdown typing:**

- **`skos:prefLabel`** — short noun-phrase **labels**, same role as titles; not treated as Markdown prose carriers.
- **`*Fact` nodes** — no dedicated prose datatype properties; facts link to `Consideration` for text.

**Enforcement:** scope notes on the above properties in [`ontology/adr-o.ttl`](/ontology/adr-o.ttl); **`sh:datatype`** for Markdown in the deferred SHACL shapes graph. OWL annotation property range assertions are editorial only for this purpose. This follows ADR-0004’s reconstructability boundary rule: shape-level datatype enforcement is treated as tooling-layer validation rather than encoded as stronger OWL constraints that do not carry the intended operational semantics for annotation properties.

Bulk retyping of legacy bare-string `skos:definition` literals on small scheme individuals may be done in a later data pass alongside SHACL.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Assert `rdfs:range` to Markdown on all prose properties in OWL | Annotation property ranges do not yield DL semantics; SHACL remains the enforcement layer. |
| Extend ADR-0003 text in place | Preserving ADR-0003 as-is plus this clarifying ADR keeps a clear audit trail. |

## Consequences

**Positive.** Aligns ADR-0003 with actual instance shape; avoids contract mismatch on `prefLabel` and fact nodes.

**Negative / risks.** Readers must consult ADR-0017 alongside ADR-0003 for the full current convention list.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — scope notes on `dcterms:description`, `skos:definition`, `skos:note`; `adr-o:Consideration` `rdfs:comment`; ontology header `dcterms:description` literal typing.
- ADR-0003 — Prose Literals Are Markdown (datatype and original scope).
- ADR-0015 — Canonical Dublin Core usage.
- ADR-0010 — `*Fact` nodes as structural placements.
- ADR-0004 — The KG Lives Under Tooling (reconstructability boundary rule).
