---
id: 39
type: Core vocabulary
status: Accepted
date: 2026-04-27
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0039 — Bibliographic References via `dcterms:references`

## Context

Decision records routinely cite external resources: academic papers, standards documents, methodology texts, architecture guides, prior-art implementations. These citations appear in the References section of every non-trivial ADR. Currently ADR-O has no vocabulary for them.

The existing inter-record predicate family (`dependsOn`, `enables`, `amends`, `supersedes`, `conflictsWith`) is causal and evolutionary — it expresses load-bearing relationships between `DecisionRecord` instances in the ADL. A bibliographic reference is categorically different: it says "this resource informed or is relevant to this decision" without asserting a causal constraint. Forcing bibliography into the causal family would overload predicates that have precise modal semantics (MUST, SHOULD, MAY) with a weaker, untyped relation.

Two further gaps compound the problem. First, references in practice are not bare IRIs — they carry human-readable labels and brief annotations explaining their relevance. A bare `<https://arxiv.org/abs/2004.03661>` is nearly useless to a reader; *"CSI-based Human Activity Recognition — surveys signal processing techniques for classifying human activities; informs the movement and breathing pattern classifiers"* is not. Second, the same external resource may be cited by multiple records in the same ADL. Any model that forces per-record inline annotation of external resources will produce duplication; any model that separates resource descriptions from record citations needs a clean linking predicate.

Both gaps are resolved by a pattern already established in the DC Terms vocabulary.

## Decision

Use **`dcterms:references`** on `DecisionRecord` as the bibliographic citation predicate, with range `rdfs:Resource`. Annotate cited resources using `dcterms:title` and `dcterms:description` (typed `^^adr-o:md`) as subject-level triples on the resource IRI, either inline in the same file or in a shared resource-descriptions graph.

No new ADR-O terms are introduced. This is a usage decision, not a vocabulary addition.

### Predicate

`dcterms:references` is defined by Dublin Core Terms as: *"A related resource that is referenced, cited, or otherwise pointed to by the described resource."* This is precisely the bibliographic relation needed. ADR-O already imports and uses DC Terms extensively; adding `dcterms:references` to the authorized predicate set requires no new namespace and no new ontology terms.

### Resource annotation pattern

Cited resources are annotated as independent subjects in the graph:

```turtle
:ADR-001 dcterms:references <https://arxiv.org/abs/2004.03661> .

<https://arxiv.org/abs/2004.03661>
    dcterms:title "CSI-based Human Activity Recognition" ;
    dcterms:description "Surveys CSI signal processing techniques for classifying human activities; informs the movement and breathing pattern classifiers."^^adr-o:md .
```

This pattern has three properties worth noting. The `dcterms:references` arc on the record and the annotation triples on the resource IRI are independent: the record links to the IRI, and the IRI is described separately. A second record citing the same resource adds only a new `dcterms:references` arc; the annotation block is already present and shared. And the resource description is itself queryable — tooling can retrieve all references with titles and annotations in a single SPARQL path without parsing Markdown.

### Scope

`dcterms:references` covers all non-ADL external citations: papers, standards, specifications, methodology documents, web resources, and archive files within the repository that are not `DecisionRecord` instances. For ADR-to-ADR relationships that are informational rather than causal — "see also" rather than "depends on" — `rdfs:seeAlso` remains available and is semantically appropriate for its weak "related resource" meaning.

### File organization

For small ADLs or single-record files, resource annotation blocks inline in the same Turtle file are compact and readable (as demonstrated in `sample.ttl`). For larger ADLs where the same resources are cited across many records, a dedicated resource-descriptions graph (e.g. `references.ttl`) avoids duplication and allows the annotations to be maintained independently. Both organizations are valid; the predicate and annotation pattern are identical in either case.

## Alternatives considered

| Alternative | Reason not chosen |
|---|---|
| Reuse `dependsOn` or other causal predicates | Causal predicates carry modal semantics (MUST, SHOULD, MAY) established in ADR-0025/0032. Bibliography is not causal; conflating the two pollutes the causal topology and makes it unqueryable as a distinct layer. |
| Mint `adr-o:cites` or `adr-o:references` | DC Terms already defines `dcterms:references` with exactly the right semantics. Minting a parallel term would violate the design principle of ADR-0002: do not define vocabulary that already exists. |
| `rdfs:seeAlso` for all references | Too weak and too broad: `rdfs:seeAlso` carries no bibliographic semantics and is already used for generic related-resource associations. Bibliography deserves a predicate that signals intent clearly. |
| Inline annotation only (no separate resource IRI subjects) | Forces duplication when the same resource is cited by multiple records; prevents shared resource description graphs; prevents SPARQL queries over the reference set. |
| Bare IRIs only, no annotation | Strips the human-readable label and relevance note — the most useful part of a references section — from the graph. Tooling would have to dereference every cited IRI to surface a title, which is impractical and brittle. |

## Consequences

**Positive.**

- Bibliographic citations are first-class graph citizens: queryable, labelable, and sharable across records.
- No new ADR-O terms: `dcterms:references` is already in the DC Terms namespace, which every ADR-O file already declares.
- The annotation pattern (`dcterms:title` + `dcterms:description` on the resource IRI) is idiomatic RDF and requires no ADR-O-specific knowledge to understand or consume.
- Shared resource descriptions are naturally deduplication-friendly: one annotation block, many citing records.
- The distinction between causal inter-record relationships (ADL scope) and bibliographic citations (informational) is now explicit and queryable.

**Negative / risks.**

- Authors must remember to annotate cited resources as well as assert the `dcterms:references` arc; a bare IRI without annotation is valid but defeats the purpose. SHACL shapes (deferred, per ADR-0004) could eventually recommend `dcterms:title` on any resource reached via `dcterms:references`.
- For very large ADLs, deciding whether resource annotations live inline or in a shared file is an organizational choice left to the ADL maintainer. This is a tooling and convention question, not an ontology question.

## References

- [ADR-0002](/ADL/ADR-0002-ontology-scope.md) — Domain-agnostic scope and the principle of not minting vocabulary that already exists.
- [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md) — DC Terms usage in ADR-O; `dcterms:references` extends the authorized DC Terms predicate set.
- [ADR-0025](/ADL/ADR-0025-causal-network.md) — Causal topology and scope architecture; establishes the ADL-scope causal predicate family that bibliographic references must not pollute.
- [ADR-0038](/ADL/ADR-0038-minting-markdown-datatype.md) — `adr-o:md` datatype alias; used for `dcterms:description` on annotated resource IRIs.
- Dublin Core Terms: `dcterms:references` — https://www.dublincore.org/specifications/dublin-core/dcmi-terms/#references
