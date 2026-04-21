---
id: 15
type: Core vocabulary
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0015 — Canonical Dublin Core and PROV Usage for ADR-O

## Context

ADR-O delegates record metadata largely to Dublin Core Terms and aligns supersession with PROV-O (ADR-0006). Several DC and PROV predicates apply to real-world ADR practice in overlapping ways; the ontology fixes **recommended** usage via `skos:scopeNote` annotations on the reused properties so implementers have one place to look.

Temporal authoring for post-factum records is policy in ADR-0005; the graph encodes the split between **when the decision held** vs **when the record was written** using DC terms.

## Decision

**Canonical predicates (as declared and annotated in [`ontology/adr-o.ttl`](/ontology/adr-o.ttl)):**

- **`dcterms:date`** — canonical **decision date** for a `DecisionRecord` (`xsd:date`). Preferred over `dcterms:issued` and `prov:generatedAtTime` for this purpose (scope note in ontology file).
- **`dcterms:created`** — optional **record creation** timestamp when the log distinguishes decision time from record lifecycle. Scope note names the **post-factum** pattern: decision date vs record authorship (ADR-0005).
- **`dcterms:creator`** — canonical **authorship**; literal or agent IRI; IRI preferred for traversable attribution. `prov:wasAttributedTo` remains permitted when PROV tooling is primary (scope note).
- **`dcterms:identifier`** — string record ID; use `adr-o:index` for sortable ordering (ADR-0007).
- **`dcterms:modified`**, **`dcterms:references`** — optional lifecycle and citation uses per scope notes.
- **`prov:wasRevisionOf`** — super-property for `adr-o:supersedes` (ADR-0006); PROV scope note clarifies usage.

Annotation properties used on the ontology document itself (`dcterms:title`, `dcterms:description`, `dcterms:license`, etc.) follow the same DC semantics.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Mix `dcterms:issued` and `dcterms:date` interchangeably | Ambiguous queries; one canonical date predicate for the decision moment. |
| `prov:generatedAtTime` as primary for records | DC terms are more widely recognised for bibliographic record metadata in this profile. |

## Consequences

**Positive.** Consistent metadata queries across ADR-O graphs; post-factum temporal split is expressible.

**Negative / risks.** Consumers must read scope notes or this ADR; not all DC usage in the wild matches this profile.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `dcterms:created`, `dcterms:creator`, `dcterms:date`, `dcterms:identifier`, `dcterms:modified`, `dcterms:references`, `prov:wasAttributedTo`, `prov:wasRevisionOf` (declarations and scope notes).
- ADR-0005 — Log All Decisions in the ADL (post-factum and real-time authoring).
- ADR-0007 — `adr-o:index` vs `dcterms:identifier`.
