---
id: 11
type: Core design
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0011 — Strict Graph: No Nygard Body Literals on the Record

## Context

Earlier sketches considered Nygard-style datatype properties (`context`, `decision`, `consequences`, `rationale`) on `DecisionRecord`, and a dedicated rejection reason literal on `Alternative`. The project moved to an atom-first graph (ADR-0010) where structure lives in `Consideration` and `*Fact` nodes, and human-facing Markdown is **materialised** by tooling from the graph (ADR-0004). ADR-0000’s inception list named `wasRejectedBecause`; that shortcut is superseded by expressing rejection through deliberation structure.

## Decision

**The ADR-O core does not define** Nygard body datatype properties on `adr-o:DecisionRecord`. Narrative content is carried on `Consideration` (and optional record-level `dcterms:description`), not as parallel opaque literals for each template section.

**Rejection rationale:** a rejected option is still an `adr-o:Alternative` in the option pool; reasons appear as `adr-o:DeliberationFact` nodes with negative deliberation valence (and/or related outcome-style facts), not as a separate literal-only escape hatch.

**Destructive change:** relative to an unreleased 0.1-style sketch, this shape is a breaking redesign; pre-1.0 technical preview scope meant no migration shim was required.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Keep body literals alongside the graph | Duplication and drift between prose blobs and structured atoms. |
| `wasRejectedBecause` datatype on `Alternative` | Two ways to say the same thing; deliberation facts give reusable IRIs for rejection rationale. |

## Consequences

**Positive.** Single source of truth for claims in the KG; aligns with tooling-mediated Markdown (ADR-0004).

**Negative / risks.** Raw Turtle authors cannot rely on filling four prose fields on the record node; they author atoms and facts.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `adr-o:DecisionRecord`, `adr-o:Alternative`, `adr-o:DeliberationFact` (comments on rejection and deliberation).
- ADR-0010 — Atom-first `Consideration` and `*Fact` link classes.
- ADR-0004 — The KG Lives Under Tooling.
- ADR-0000 — Inception Record (historical prototype list; superseded items noted in Context above).
