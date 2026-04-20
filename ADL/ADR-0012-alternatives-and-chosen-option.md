# ADR-0012 — Alternatives: Option Pool, Chosen Option, Functional Edges

| Field        | Value |
|--------------|-------|
| ID           | ADR-0012 |
| Type         | Core design |
| Status       | Accepted |
| Date         | 2026-04-19 |
| Author       | Bogdan Stăncescu <bogdan@moongate.ro> |

## Context

Decisions need a machine-traversable answer to “what options existed?” and “which option won?” Multi-winner outcomes (“use both X and Y”) must be representable without breaking a single functional “chosen” edge.

## Decision

**`adr-o:Alternative`** — class of first-class option nodes linked from a record via **`adr-o:hasAlternative`** (full option pool).

**`adr-o:chosenAlternative`** — `owl:ObjectProperty`, **`owl:FunctionalProperty`**, domain `adr-o:DecisionRecord`, range `adr-o:Alternative`: **at most one** chosen alternative per record. Cardinality zero is allowed (e.g. `Proposed`). The chosen alternative should also appear in `hasAlternative`; full membership and consistency checks are deferred to SHACL (ADR-0004).

**Multi-winner:** mint a **composite** `Alternative` representing the combined choice rather than relaxing functional cardinality on `chosenAlternative`.

**Per-fact wiring:**

- **`adr-o:consideration`** — `owl:FunctionalProperty` from each `*Fact` to its `Consideration`.
- **`adr-o:onAlternative`** — `owl:FunctionalProperty`, domain union of `DeliberationFact` and `OutcomeFact`, range `Alternative`. **Authoring convention** (SHACL-deferred): required on `DeliberationFact`; optional on `OutcomeFact` (implicit chosen option when absent; explicit for counterfactuals about rejected options); not used on `ContextFact`.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Multiple `chosenAlternative` objects | Harder to query “the” decision outcome; composite `Alternative` keeps one functional edge. |
| Infer choice only from deliberation valence aggregates | Ambiguous; `chosenAlternative` is authoritative for “what was decided?”. |

## Consequences

**Positive.** Clear SPARQL paths for option enumeration and chosen option.

**Negative / risks.** Composite alternatives require authoring discipline to name and reuse consistently.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `adr-o:Alternative`, `adr-o:hasAlternative`, `adr-o:chosenAlternative`, `adr-o:consideration`, `adr-o:onAlternative`.
- ADR-0010 — Reified facts and lists.
- ADR-0004 — The KG Lives Under Tooling.
