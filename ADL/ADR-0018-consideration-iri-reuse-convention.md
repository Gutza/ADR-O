---
id: 18
type: Core vocabulary
status: Superseded
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
supersededBy:
  - ADR-0028
---

# ADR-0018 — Editorial Convention for `Consideration` IRI Reuse

## Context

Atom-first design (ADR-0010) pays off when the same `Consideration` IRI is reused across records and roles. Authors need practical patterns for **record-local** atoms vs **organisation-shared** atoms without the core ontology prescribing a formal `scope` property in v0.2.x.

Naive authoring tends to fail in two opposite directions:

- **Always local, never promoted.** Equivalent atoms are copied per record under new fragment IRIs. The graph loses cross-record identity, making global queries over recurring rationale weaker and forcing tooling to rely on brittle text similarity.
- **Always global, from day one.** Early uncertain phrasings are minted as if they were stable shared atoms, creating noisy identity governance and accidental coupling between records that only look similar.

## Decision

**Editorial convention only** (documented in the `rdfs:comment` on `adr-o:Consideration` in [`ontology/adr-o.ttl`](/ontology/adr-o.ttl), not as additional OWL axioms):

- **Record-local** — fragment IRIs under the hosting `DecisionRecord` when an atom may never leave that record.
- **Org-shared** — a stable namespace for atoms deliberately reused across records or templates.
- **Promotion** — when promoting local to shared, use **`owl:sameAs`** from the shared IRI to the earlier local IRI so existing references keep resolving.

Recommended authoring practice:

- Start **record-local by default** when a claim is new, provisional, or tightly bound to one ADR.
- Promote to **org-shared** when an atom is intentionally reused across records, templates, or policy guidance.
- Keep promotions explicit and reviewable: a promotion should be a deliberate editorial act, not an implicit side effect of copy-paste.
- In tooling surfaces, prefer showing a canonical shared IRI when one exists, while preserving local aliases through `owl:sameAs`.

Future work may introduce an explicit `adr-o:scope` or similar **without** breaking existing IRIs.

### Examples

**Naive local-only (undesirable):**

- `ADR-0042#cons-3`: "Retraining cost is weeks per engineer."
- `ADR-0051#cons-2`: "Engineer retraining takes multiple weeks."

Both atoms are effectively the same rationale, but without promotion they remain disconnected. Querying for recurring organisational concerns (for example onboarding/training cost) becomes partial unless tooling performs lexical guesswork.

**Naive global-only (undesirable):**

- `https://example.org/adl/consideration/ui-consistency-matters`

Minted as shared during a single meeting draft, then later narrowed to one product context. Reusing this IRI elsewhere creates false equivalence and accidental coupling between unrelated decisions.

**Recommended pattern (counter-example to both):**

1. Record `ADR-0042#cons-3` locally during deliberation.
2. When the same rationale recurs, mint shared IRI `https://example.org/adl/consideration/retraining-cost-weeks`.
3. Assert `https://example.org/adl/consideration/retraining-cost-weeks owl:sameAs ADR-0042#cons-3`.
4. Point new `*Fact` links to the shared IRI; retain old links for stability.

This keeps early authoring lightweight while converging toward reusable identity when reuse is real.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Mandatory global IRIs for all `Consideration` nodes | Heavy governance for atoms that never leave one record. |
| Axiom forcing scope or home record | Premature; editorial guidance suffices for the technical preview. |

## Consequences

**Positive.** Clear human/tooling guidance while keeping the OWL surface minimal.

**Negative / risks.** Discipline is social and tool-mediated; mis-identity remains possible without SHACL rules for IRI shape and without editorial review of promotions.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `adr-o:Consideration` `rdfs:comment`.
- ADR-0010 — Atom-first reification.
- ADR-0004 — The KG Lives Under Tooling (reconstructability boundary rule).
