---
id: 9
type: Core vocabulary
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0009 — Deliberation and Outcome Valence as SKOS Schemes

## Context

Deliberation (weighing options) and outcomes (post-decision characterisation) need disjoint controlled vocabularies: deliberation speaks to *evaluation of alternatives*; outcomes speak to *what happened or follows* after the decision. Collapsing both into one predicate would force a permissive union range or push all checking to SHACL.

## Decision

**Two separate object properties with disjoint ranges:**

- **`adr-o:deliberationValence`** — domain `adr-o:DeliberationFact`, range `adr-o:DeliberationValence`. Values are SKOS concepts in **`adr-o:deliberationValenceScheme`**: `adr-o:Supports`, `adr-o:Against`, `adr-o:Neutral` (core set). Property is `owl:FunctionalProperty`.

- **`adr-o:outcomeValence`** — domain `adr-o:OutcomeFact`, range `adr-o:OutcomeValence`. Values are SKOS concepts in **`adr-o:outcomeValenceScheme`**: `adr-o:Benefit`, `adr-o:AcceptedCost`, `adr-o:Risk`, `adr-o:FollowUp` (core set). Property is `owl:FunctionalProperty`.

**Named classes** `adr-o:DeliberationValence` and `adr-o:OutcomeValence` follow the same `skos:Concept` ∩ `skos:inScheme` pattern as `adr-o:Status` (see ADR-0008).

**`owl:AllDifferent`** blocks over each scheme’s core individuals assert pairwise distinctness for DL reasoners, with the same lightweight-RL caveat as for status (see ADR-0008).

**KISS choice:** a single `Neutral` deliberation bucket; splitting “orthogonal” vs “in-scope but non-decisive” is deferred and would be non-breaking if added later.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| One `valence` property with union range | Self-documentation and clean `rdfs:range` per role suffer; enforcement moves entirely to SHACL. |
| Finer-grained neutral split in v1 | Not needed for the technical preview; adds scheme churn without proven demand. |

## Consequences

**Positive.** Clear query semantics; enums are extensible via domain profiles using the same SKOS scheme pattern.

**Negative / risks.** Two IRIs to learn instead of one; authors must pick the correct property for the fact class.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `adr-o:deliberationValence`, `adr-o:outcomeValence`, `adr-o:DeliberationValence`, `adr-o:OutcomeValence`, schemes, individuals, `AllDifferent` blocks.
- ADR-0008 — Status scheme and integrity pattern (parallel design).
- ADR-0004 — The KG Lives Under Tooling (SHACL companion deferred; list-member typing likewise deferred).
