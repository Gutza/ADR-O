---
id: 31
type: Core vocabulary
status: Accepted
date: 2026-04-24
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0010
  - ADR-0017
  - ADR-0027
  - ADR-0028
---

# ADR-0031 — Rename `adr-o:Consideration` to `adr-o:Claim`

## Context

ADR-O's atom-first architecture (ADR-0010) reifies the smallest units of a decision record as `adr-o:Consideration` nodes. These atoms are placed into roles by `*Fact` classes.

The term "consideration" was chosen in ADR-0010 to read neutrally across positive, negative, and framing uses. It worked when ADR-O was framed as a record of a deliberation process — a consideration is something you entertain during thinking.

However, ADR-0028 (the Decision Transaction Principle) and ADR-0030 (Observations) fundamentally changed the ontology's epistemic posture. A `DecisionRecord` is no longer just a record of a process; it is a **transaction** — a frozen set of commitments at $t_0$.

The `Observation` model in ADR-0030 exposes the linguistic mismatch. An `Observation` links to an `ExpectedOutcome`, which in turn `manifests` a `Consideration`. The `Observation` then yields a `verdict` (`Satisfied`, `Violated`, `Inconclusive`).

You cannot verify a "consideration." A consideration is a thought in a head. You can only verify a **claim** — a proposition that is either true or false (or inconclusive).

The distinction is also about stance: "consideration" is passive (something merely present in deliberation), while "claim" is active (something that someone actively asserted about the world). That active posture better fits an ADR, which is meant to capture commitments rather than just thoughts.

## Decision

**Rename the class `adr-o:Consideration` to `adr-o:Claim`.**

This is a breaking change to the core class name. All related terms and documentation must be updated.

### The Epistemic Shift

| Role | Before: `Consideration` | After: `Claim` |
|:---|:---|:---|
| **In ContextFact** | "We are considering this condition" | "We claim this condition exists as a premise" |
| **In DeliberationFact** | "We are considering this pro" | "We claim this alternative has this property" |
| **In ExpectedOutcome** | "We consider this will happen" | "We claim this will happen" |
| **In Observation** | "We verify this consideration" | "We verify this claim" |

A `Claim` is a proposition. It has a truth value. This makes the `Observation` → `ExpectedOutcome` → `Claim` chain logically sound: the observation tests whether the claim manifested in the expected outcome is true.

### Ontology Materialisation

```ttl
### https://w3id.org/adr-o#Claim
adr-o:Claim rdf:type owl:Class ;
            rdfs:comment "A reusable atomic proposition that can be asserted as a fact in one or more roles within a DecisionRecord. A Claim is a statement that can be verified — it has a truth value. This replaces the term 'Consideration' to align with the Decision Transaction Principle (ADR-0028) and the verification model (ADR-0030)."@en ;
            rdfs:label "Claim"@en ;
            adr-o:justifiedBy adr-odr:ADR-0031-rename-consideration-to-claim .

### Update manifests range
adr-o:manifests rdfs:range adr-o:Claim .

### Update derivedFrom / derives
adr-o:derivedFrom rdfs:domain adr-o:Claim ;
                  rdfs:range adr-o:Claim .

adr-o:derives rdfs:domain adr-o:Claim ;
              rdfs:range adr-o:Claim .
```

### Documentation updates

- **`adr-o:manifests`** comment: "Links a reified fact to the **Claim** it manifests."
- **`adr-o:derivedFrom` / `adr-o:derives`** comments: replace "Consideration" with "Claim".
- **ADR-0010, 0017, 0027, 0028** references and internal prose must be updated to use "Claim".

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| **Keep `Consideration`** | Epistemically dishonest after ADR-0030; you don't verify considerations, you verify claims. |
| **`Proposition`** | Technically accurate but less idiomatic in the ADR literature than "claim" or "consideration". |
| **`Fact`** | Already used by the `*Fact` classes; would create confusing class/property name collisions. |
| **`Claim` for some roles, `Consideration` for others** | Splits the atom-first principle; the whole point of the atom is that it's the same thing in different roles. |

## Consequences

**Positive.**
- The verification loop in ADR-0030 becomes logically coherent: an `Observation` verifies a `Claim`.
- The terminology aligns with the Decision Transaction Principle: the ADL is a record of claims made at $t_0$.
- The "Claim" metaphor better supports the `derivedFrom`/`derives` relationship — one claim is derived from another.

**Negative / risks.**
- Breaking change to the core class IRI. All existing instances must be migrated (or aliased via `owl:equivalentClass` during a transition period).
- Documentation churn across multiple ADRs and design notes.

## References

- [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md) — original reification design.
- [ADR-0027](/ADL/ADR-0027-facts-manifest-considerations.md) — the `manifests` predicate.
- [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) — DTP.
- [ADR-0030](/ADL/ADR-0030-observations.md) — the verification loop.