---
id: 29
type: Project governance
status: Accepted
date: 2026-04-24
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0010
---

# ADR-0029 — Provenance Correction: `Consideration` Identity Is an ADR-O Design Contribution

## Context

During the 0.2.0-draft design phase, the project introduced `Consideration` as a first-class reusable atom, allowing the same IRI to appear in multiple roles within a single `DecisionRecord`. The design notes and several normative documents attributed this structural property — particularly the ability to link a *Facing* concern to a *To-achieve* expected outcome via identity — to prior art in YADR (Yet Another Decision Record, the YAML port of MADR).

Two compounding errors were introduced:

1. **Misinterpretation of YADR's anchor/alias mechanism.** YADR's `&anchor` / `*alias` syntax is a *clerical deduplication* tool in YAML. It prevents copy-paste drift by aliasing identical strings within a single document. It does not encode a semantic claim about argument identity across roles; it does not link *Facing* to *To-achieve*; and it operates at the string level, not the entity level. Attributing "the argument-identity property" to YADR was a category error.

2. **Subsequent confusion of YADR with Y-statements.** Once the loop was framed as a "YADR property," references in some documents shifted to "Y-statement" (Olaf Zimmermann's formalization) as the attributed source. Y-statements also have no structural mechanism to link facing concerns to expected outcomes as identical entities. The attribution was wrong in both directions.

These errors compounded into a two-stage false naturalization: a novel ADR-O design choice was presented first as derived from YADR, then as an inherent property of Y-statements that ADR-O was merely "recovering."

## Decision

We correct the record:

**The intra-record `Consideration` identity property — the ability for the same `Consideration` IRI to appear simultaneously as a `ContextFact` (need) and as an `ExpectedOutcome` (expected gain), and the consequent machine-verifiable coherence check this enables — is an ADR-O original design contribution.**

It was not derived from YADR's clerical anchor/alias mechanism. It was not inherited from any Y-statement format. It emerged from first-principles reasoning about what it means to make a decision graph internally coherent: a decision is honest when the claim it set out to satisfy and the claim it asserts as its expected benefit are verifiably the same thing, not merely editorially similar prose.

This is documented from first principles in [`Archive/Consideration Reification - A Design Choice.md`](/Archive/Consideration%20Reification%20-%20A%20Design%20Choice.md).

### Policy for future documentation

When describing ADR-O's design in relation to prior art:

- **YADR** may be cited accurately as a YAML serialization of MADR that uses anchor/alias for clerical string deduplication within a single document. It is a useful reference for the *serialization* dimension of decision records.
- **Y-statements** (Zimmermann) may be cited accurately as a six-clause prose template. ADR-O maps those clauses to typed graph roles; this mapping is ADR-O's contribution.
- **ADR-O's intra-record `Consideration` identity** must be described as an ADR-O design property, not as a property inherited from or recovered from any prior format.

### Scope of corrections applied

The following files have been corrected in place under this ADR:

| File | Change |
|------|--------|
| `ADL/ADR-0010-consideration-and-reified-facts.md` | Removed "YADR-style identity" parenthetical from Consequences. |
| `ADL/ADR-0028-integrating-the-decision-transaction-principle.md` | Renamed Section 7 and reframed its content to own the coherence property as ADR-O's contribution; corrected adjacent references. |
| `ontology/DESIGN-NOTES.md` | Removed "YADR-style argument-identity property" attributions from the 0.2.0-draft section; corrected "YADR slot" to "Y-statement clause" where applicable. |
| `ontology/IDEAS.md` | Rewrote the "Argument reuse problem" section to accurately describe YADR's clerical mechanism and distinguish ADR-O's IRI-identity contribution. |
| `ontology/ROADMAP.md` | Replaced "Solves the YADR identity problem" with an accurate description of ADR-O's design. |
| `ontology/adr-o.ttl` | Removed "YADR-style identity property" and "YADR-style Y-statement loop closure" language from `adr-o:manifests` and `adr-o:Consideration` comments; corrected `adr-o:ExpectedCost` attribution. |

## Alternatives considered

| Alternative | Reason not chosen |
|---|---|
| Leave the attribution as historical footnote | The attribution was not a historical footnote; it was load-bearing justification for design choices, making it actively misleading. |
| Add a corrective note without rewriting | Annotating false premises is less clear than removing them; the false attribution adds no information value. |
| Record as supersession of ADR-0010 | ADR-0010's decisions remain sound; only a single parenthetical phrase in its Consequences was incorrect. An amendment is the appropriate scope. |

## Consequences

**Positive.**
- The ADR-O documentation now accurately attributes design innovations to their actual source.
- Future contributors can cite the coherence property as an ADR-O contribution without inheriting a false provenance chain.
- The distinction between YADR's serialization-level mechanism and ADR-O's graph-level identity is now explicit and documented.

**Negative / risks.**
- Any external writing that quoted the old "YADR-style identity" language will diverge from this corrected record. This is an acceptable cost: external secondary sources are not the project's responsibility to maintain.

## References

- [`Archive/Consideration Reification - A Design Choice.md`](/Archive/Consideration%20Reification%20-%20A%20Design%20Choice.md) — first-principles derivation of the three-layer architecture.
- [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md) — original `Consideration` and `*Fact` design (amended by this record).
- [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) — Decision Transaction Principle integration (corrected in-place, pre-commit).
- [Y-Statements](/Archive/Y-Statements.md) — accurate description of the Zimmermann formalization that ADR-O maps to graph roles.
