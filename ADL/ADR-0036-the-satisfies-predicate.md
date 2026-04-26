---
id: 36
type: Core vocabulary
status: Accepted
date: 2026-04-26
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0036 — The `satisfies` Predicate for Intra-Record Syllogism

## Context

ADR-O's atom-first architecture (ADR-0010) successfully decomposes a decision into its constituent `Claim`s, `Alternative`s, and `ExpectedOutcome`s. However, a structural gap remains when attempting to reconstruct the full logic of a decision.

The **Y-statement roundtrip test** reveals this gap. A Y-statement is a six-clause sentence:

> *"In the context of [A], facing [B], we decided for [C] and neglected [D], to achieve [E], accepting that [F]."*

The ontology maps these clauses as follows:

| Clause | Mapping |
| :--- | :--- |
| *"In the context of..."* | `hasContext` → `ContextFact` → `Claim` |
| *"Facing..."* | `hasContext` → `ContextFact` → `Claim` (with `addresses` → `Concern`) |
| *"We decided for..."* | `chosenAlternative` → `Alternative` |
| *"And neglected..."* | `hasAlternative` ∖ `chosenAlternative` |
| *"To achieve..."* | `hasExpectedOutcome` → `ExpectedOutcome` (`outcomeValence: ExpectedGain`) |
| *"Accepting that..."* | `hasExpectedOutcome` → `ExpectedOutcome` (`outcomeValence: ExpectedCost`) |
| *The implicit "Therefore"* | ❌ No edge linking the "Facing" premise to the "Decided for" conclusion |

The "To achieve" and "Accepting that" clauses were closed by ADR-0028's `ExpectedOutcome` model. The one remaining gap is the implicit **"Therefore"**: the ontology can represent the facing concern and the chosen alternative as separate nodes, but it cannot express the syllogism — the logical bridge asserting that the chosen alternative was selected *because* it satisfies the facing concern.

Without this edge, the connection between a record's framing and its conclusion is implicit (coexistence in the same record) rather than explicit. Tooling cannot programmatically verify whether the chosen alternative actually addresses the stated facing concern, nor detect when a decision has drifted from its original framing.

This gap is distinct from:

- `adr-o:justifiedBy` (ADR-0026): Project-scope provenance from a resource back to its warranting `DecisionRecord`.
- `adr-o:addresses` (ADR-0013): Links a `Claim` to a `Concern` it resolves.
- `adr-o:deliberationValence` (ADR-0009): Weighs a `Claim` against an `Alternative`; expresses support, not satisfaction of a specific need.

## Decision

Introduce **`adr-o:satisfies`** as a first-class object property to encode the intra-record inference bridge.

### Definition

```ttl
adr-o:satisfies rdf:type owl:ObjectProperty ;
                rdfs:domain adr-o:Alternative ;
                rdfs:range adr-o:Claim ;
                rdfs:comment "ADR scope: indicates that an Alternative satisfies a Claim (typically the Claim manifested in a ContextFact that expresses the record's primary facing concern)."@en ;
                rdfs:label "satisfies"@en ;
                adr-o:justifiedBy adr-odr:ADR-0036-the-satisfies-predicate .
```

### Naming convention

The property is active-voice on `Alternative`. This is a deliberate departure from the passive-voice-on-`Claim` convention established in ADR-0032 for ADL-scope causal predicates (`recommendedBy`, `constrainedBy`, etc.). The distinction is principled: ADL-scope predicates are stated from the `Claim`'s perspective because inter-record constraints are facts about the claim's lineage. ADR-scope predicates like `satisfies` are stated from the `Alternative`'s perspective because they are assertions made during deliberation, in the deliberator's natural voice: "this option satisfies that need."

### Rules

1. **Domain/Range:** Only from `Alternative` to `Claim`.
2. **Scope:** Intended for use within a single `DecisionRecord` (ADR scope). Cross-record use is not prohibited by the ontology but carries no defined semantics here.
3. **Relationship to chosen alternative:** The `chosenAlternative` of a record should `satisfies` the `Claim`(s) manifested by `ContextFact`s that `addresses` the record's primary facing `Concern`. Multiple facing `Claim`s may exist; multiple `satisfies` arcs are permitted.
4. **Syllogism closure:** This predicate transforms the record from a collection of parts into a directed argument. The full SPARQL-traversable path is:

```sparql
?dr  adr-o:chosenAlternative  ?alt .
?alt adr-o:satisfies           ?claim .
?claim adr-o:addresses         ?concern .
```

Which corresponds to the graph shape:

```
DecisionRecord —chosenAlternative→ Alternative —satisfies→ Claim —addresses→ Concern
```

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Use `adr-o:addresses` | `addresses` is a general association between a `Claim` and a `Concern`; `satisfies` is a specific claim of resolution between an `Alternative` and a `Claim`. Different subjects, different semantics. |
| Use `adr-o:justifiedBy` | `justifiedBy` is Project-scope provenance; `satisfies` is ADR-scope deliberation logic. Reusing the same predicate across scopes would violate the scope discipline established in ADR-0025/0028. |
| `adr-o:justifies` (inverse direction, `Claim` → `Alternative`) | Considered as the natural reading "this concern justifies this choice." Rejected because the name creates semantic confusion with `adr-o:justifiedBy`, which serves an entirely different purpose (Project-scope provenance). Having two predicates sharing a root across different scopes and directions would be a persistent source of authoring errors. |
| Infer from `DeliberationFact` `Supports` valence | A `Supports` valence is a weight, not a satisfaction claim. A claim can support an alternative without that alternative satisfying the facing concern. The semantics are categorically different. |
| No predicate (keep implicit) | Leaves the Y-statement roundtrip lossy; prevents machine verification of decision coherence. |

## Consequences

**Positive.**

- **Roundtrip closed.** The Y-statement can now be decomposed and recomposed losslessly for all six clauses.
- **Coherence verification.** Tooling can query: "Does the chosen alternative actually satisfy any of the facing claims?" and surface records where no `satisfies` arc exists.
- **Linguistic precision.** Distinguishes "we considered this" (`DeliberationFact`) from "this solves that" (`satisfies`).
- **SPARQL traversal.** The full path from `Concern` ← `Claim` ← `Alternative` ← `DecisionRecord` is now machine-walkable in a single query.

**Negative / risks.**

- Additional authoring burden (though easily inferred by LLM writers from the Y-statement structure, and by human authors who understand the "Facing" clause).
- Rule 3 relies on the author correctly identifying the facing `Claim`(s); there is no structural enforcement until SHACL shapes are introduced.

## References

- [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md) — Deliberation valence
- [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md) — Atom-first reification
- [ADR-0012](/ADL/ADR-0012-alternatives-and-chosen-option.md) — Alternatives and chosen option
- [ADR-0013](/ADL/ADR-0013-addresses-dual-placement.md) — `adr-o:addresses` dual placement
- [ADR-0025](/ADL/ADR-0025-causal-network.md) — Causal topology and scopes
- [ADR-0026](/ADL/ADR-0026-ontology-provenance.md) — `adr-o:justifiedBy` and the `adr-odr` namespace
- [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) — Decision Transaction Principle
- [ADR-0031](/ADL/ADR-0031-rename-consideration-claim.md) — Rename `Consideration` to `Claim`
- [ADR-0032](/ADL/ADR-0032-unified-voice-and-naming.md) — Passive-voice convention for ADL-scope predicates
