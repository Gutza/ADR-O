---
id: 42
type: Project governance
status: Accepted
date: 2026-05-05
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends: [5, 25, 28]
deprecates: [4, 5]
---

# ADR-0042 - All Project Documents Are Deliverables

## Context

ADR-O already removed significance filtering for decision logging (ADR-0005): in-scope decisions are recorded regardless of perceived magnitude. In practice, however, document treatment remained inconsistent. Some documents were treated as ordinary project artifacts, while others were shaped as ADRs only because they appeared likely to influence future decisions.

This creates ambiguity at authoring time:

- the same kind of artifact may or may not become an ADR depending on subjective expectations about downstream impact;
- ADRs risk becoming a mixed bucket of true decisions and non-decision documents;
- indexing and traversal lose semantic clarity because "is a document" and "is a decision record" are conflated.

The model already distinguishes decision records from other resources (`adr-o:DecisionRecord` versus generic referenced resources and project artifacts). The missing piece is a governance rule that removes the remaining ambiguity.

## Decision

Adopt the following project policy:

1. **All project documents are deliverables**, regardless of whether they affect future decisions, and regardless of how much they affect future decisions.
2. **ADR status is reserved for decision records only.** A document is represented as an ADR only when its primary purpose is to record a decision transaction.
3. **Non-decision documents remain first-class project artifacts** and are linked as resources (for example through `dcterms:references` and project-scope provenance links), without being disguised as ADRs.
4. **When in doubt, classify by artifact intent, not predicted influence.** If the artifact records a commitment, it is an ADR; otherwise it is a document deliverable.

### Classification criteria (operational test)

Apply the following guard clauses in order:

1. **If the artifact does not record a concrete commitment made in this ADL, it is not an ADR.**
2. **If the artifact is primarily guidance for ADR-O practitioners in general (method essay, explainer, playbook), it is a document deliverable, not a local ADR.**
3. **If the artifact explains or defends decisions more than it records a discrete transaction delta, split it:**
   - keep a minimal ADR with normative decision deltas;
   - move rationale, vignettes, and pedagogy to a companion document.
4. **If in doubt between ADR and document, default to document and promote to ADR only when a local, machine-traversable decision transaction is explicit.**

### Signals of a disguised document

The following signals indicate probable misclassification as ADR:

- The text is self-contained as a standalone essay/explainer even if ADR links are removed.
- The primary audience is external practitioners rather than maintainers of this ADL.
- Most content is argumentation, examples, or pedagogy; the normative decision delta is thin.
- The record could be moved to `Archive/` or `Docs/` with little or no semantic loss to ADL transaction history.

### Reclassification guidance

When an existing ADR is reclassified as document:

- Preserve the text as a project deliverable (do not delete the content).
- Remove it from ADR indexing as a decision record.
- Keep references from ADRs to that document where it still provides rationale or guidance.
- Create a replacement ADR only if a local governance or ontology commitment must remain machine-traversable in the ADL.

### Deprecation-and-Lift Index

- **ADR-0004 - The KG Lives Under Tooling** -> **Deprecated and lifted** to [`/Archive/ADR-O%20Tooling-Mediation%20Principle.md`](/Archive/ADR-O%20Tooling-Mediation%20Principle.md); registry action: `status: Deprecated`, `deprecatedBy: 42`; rationale: primarily methodological guidance for ADR-O practitioners.
- **ADR-0005 - Log All Decisions in the ADL** -> **Deprecated and lifted** to [`/Archive/Inclusion-First%20ADL%20Authoring%20Principle.md`](/Archive/Inclusion-First%20ADL%20Authoring%20Principle.md); registry action: `status: Deprecated`, `deprecatedBy: 42`; rationale: inclusion-first authoring doctrine for ADR-O ADLs, not a local ADL transaction.
- **ADR-0025 - Causal Topology: Scopes, Modal Constraints, and Materialization** -> **Amended and lifted** to [`/Archive/ADR-O%20Causality%20Scopes.md`](/Archive/ADR-O%20Causality%20Scopes.md); registry action: `status: Accepted`, `amendedBy: [42]`; rationale: ontology-justifying decisions remain active while prose is lifted to a companion article.
- **ADR-0028 - Integrating the Decision Transaction Principle** -> **Amended and lifted** to [`/Archive/Decision%20Transaction%20Principle.md`](/Archive/Decision%20Transaction%20Principle.md); registry action: `status: Accepted`, `amendedBy: [42]`; rationale: ontology-justifying decisions remain active while prose is lifted to a companion article.

## Alternatives considered

| Alternative | Reason not chosen |
|---|---|
| Keep current mixed practice | Preserves ambiguity and inconsistent authoring outcomes. |
| Classify as ADR only when expected future impact is high | Reintroduces significance gatekeeping by another name and keeps decisions subjective. |
| Convert all project documents into ADRs | Collapses useful type boundaries and dilutes the meaning of `DecisionRecord`. |

## Consequences

**Positive.**

- Removes classification ambiguity and authoring hesitation.
- Preserves ADR semantic precision: ADR index tracks decisions, not all documentation.
- Keeps documentation comprehensive without forcing non-decision artifacts into decision shape.
- Improves machine-traversable clarity across ADR scope vs project-scope resources.
- Clarifies the scope boundary between local ADL commitments and general practitioner guidance.

**Negative / risks.**

- Requires discipline in triage when creating new artifacts.
- Some existing artifacts may need reclassification over time to align with this policy.
- Transitional churn is expected while legacy "disguised ADRs" are split or moved.

## References

- [ADR-0005](/ADL/ADR-0005-log-all-decisions.md) - Establishes all in-scope decisions are logged without significance filtering.
- [ADR-0025](/ADL/ADR-0025-causal-network.md) - Defines project-scope provenance and deliverable relationships.
- [ADR-0039](/ADL/ADR-0039-references.md) - Standardizes bibliographic links to non-DecisionRecord resources.
- [/Archive/ADR-O%20Tooling-Mediation%20Principle.md](/Archive/ADR-O%20Tooling-Mediation%20Principle.md) - Lifted practitioner article replacing ADR-0004 as the canonical guidance text.
- [/Archive/Inclusion-First%20ADL%20Authoring%20Principle.md](/Archive/Inclusion-First%20ADL%20Authoring%20Principle.md) - Lifted practitioner article replacing ADR-0005 as the canonical guidance text.
- [/Archive/ADR-O%20Causality%20Scopes.md](/Archive/ADR-O%20Causality%20Scopes.md) - Lifted companion article for ADR-0025.
- [/Archive/Decision%20Transaction%20Principle.md](/Archive/Decision%20Transaction%20Principle.md) - Lifted companion article for ADR-0028.
