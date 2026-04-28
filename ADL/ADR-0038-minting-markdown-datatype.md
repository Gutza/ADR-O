---
id: 38
type: Core vocabulary
status: Accepted
date: 2026-04-27
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0003
  - ADR-0017
---

# ADR-0038 — Minting `adr-o:md` as an Ergonomic Markdown Datatype Alias

## Context

ADR-0003 committed all prose-carrying literals in ADR-O to the IANA-rooted datatype IRI:

```
"…"^^<https://www.w3.org/ns/iana/media-types/text/markdown>
```

The decision was sound: the IRI is standards-grounded, authoritative, and avoids minting vocabulary that already exists. ADR-0017 subsequently narrowed the scope to `dcterms:description`, `skos:definition`, and `skos:note` sub-properties.

In practice, the 52-character IRI with mandatory angle brackets appears on every prose-carrying literal in the graph. The `sample.ttl` exemplar alone repeats it roughly thirty times. No Turtle prefix trick eliminates this: Turtle's prefixed-name grammar forbids a `/` in the local-name component, so `imt:text/markdown` is not valid portable Turtle. The only options are the bare angle-bracket form or a datatype alias.

ADR-0003 considered and rejected a custom `adr-o:markdown` on the grounds that it "invents vocabulary that already exists." That argument evaluated principled alignment against a competing coined IRI — it did not evaluate authoring ergonomics. Revisiting the question with that dimension in scope:

The external recognition value of the IANA IRI over a coined IRI is, at runtime, approximately zero. ADR-0003 itself records that the IRI "will be treated as an unknown custom datatype" by every major OWL reasoner and SPARQL engine. The standards-provenance signal is real but it is prestige for ontology documentation readers, not runtime semantics for machines. That signal can be preserved exactly — via `owl:equivalentClass` — while also minting the short alias.

The name `adr-o:md` rather than `adr-o:markdown` is justified by the regulated-vocabulary context. In a general-purpose vocabulary, short names sacrifice self-documentation. Here, consumers already need to know the namespace to make sense of `adr-o:DeliberationFact`, `adr-o:manifests`, or `adr-o:outcomeValence`; one more short name adds no marginal opacity. Furthermore, the literal's own content removes any remaining ambiguity: a reader seeing `"…prose…"^^adr-o:md` on a `dcterms:description` property infers `md = Markdown` immediately — there is nothing else it could plausibly be. `md` is also the universal colloquial shorthand for Markdown (file extension, tooling flags, community usage), not an ADR-O invention.

## Decision

Mint **`adr-o:md`** as a declared `rdfs:Datatype` with `owl:equivalentClass` pointing at the IANA IRI, and use it in place of the bare IANA IRI in `adr-o.ttl` and all ADR-O-authored Turtle files.

### Ontology materialization

```ttl
### https://w3id.org/adr-o#md
adr-o:md rdf:type rdfs:Datatype ;
         owl:equivalentClass <https://www.w3.org/ns/iana/media-types/text/markdown> ;
         rdfs:label "Markdown"@en ;
         rdfs:comment "Ergonomic alias for the IANA text/markdown datatype IRI. Use on all prose-carrying literals per ADR-0003 and ADR-0017."@en ;
         skos:scopeNote "adr-o:md and <https://www.w3.org/ns/iana/media-types/text/markdown> are declared equivalent. The IANA IRI remains the authoritative identity; adr-o:md is the serialization form used in ADR-O Turtle files."@en ;
         adr-o:justifiedBy adr-odr:ADR-0038-minting-markdown-datatype .
```

### Usage

With the standard `adr-o:` prefix declared, every prose-carrying literal is written as:

```turtle
dcterms:description "…"^^adr-o:md ;
skos:definition     "…"^^adr-o:md ;
skos:scopeNote      "…"^^adr-o:md ;
```

### Relationship to ADR-0003

This decision does not contradict ADR-0003. The authoritative datatype identity remains the IANA IRI; `adr-o:md` is a serialization convenience declared equivalent to it, not a competing definition. ADR-0003's governing principle — "do not invent vocabulary that already exists" — is satisfied because `adr-o:md` does not exist independently: it is defined solely as an alias for vocabulary that already exists.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Keep bare IANA IRI | Correct but verbose: 52 characters with mandatory angle brackets, repeated on every prose literal, with no valid Turtle prefix trick available. |
| `adr-o:markdown` | Same standing as `adr-o:md`; no meaningful gain in clarity given that the literal content already conveys the format, and the regulated-vocabulary context makes the extra six characters pure noise. |
| Turtle prefix aliasing (`@prefix md: <…/text/markdown>` then `"…"^^md:`) | The local-name component after the colon would be empty, producing a syntactically bizarre `^^md:` form. Valid in some parsers but confusing and non-idiomatic. |

## Consequences

**Positive:**

- Authoring verbosity drops substantially: `^^adr-o:md` versus `^^<https://www.w3.org/ns/iana/media-types/text/markdown>`.
- The IANA IRI's provenance is preserved via `owl:equivalentClass`, satisfying the spirit of ADR-0003.
- SHACL shapes can use either IRI; the planned `sh:datatype` constraint can target `adr-o:md` directly.
- The `adr-o:` prefix is already declared in every ADR-O Turtle file, so no new prefix block entry is required.

**Negative / risks:**

- Consumers who compare datatype IRIs as raw strings (rather than following `owl:equivalentClass`) will not automatically recognise `adr-o:md` as equivalent to the IANA IRI. This is the same limitation ADR-0003 already accepted for the IANA IRI itself vis-à-vis built-in reasoner datatypes; it is a documentation responsibility, not a structural flaw.
- Domain profiles or external files that use the bare IANA IRI remain valid and interoperable; no migration is required or implied.

## References

- [ADR-0003](/ADL/ADR-0003-prose-literals-markdown.md) — Prose literals are Markdown; establishes the IANA IRI as the authoritative datatype.
- [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md) — Narrows the scope of the Markdown datatype to `dcterms:description`, `skos:definition`, and `skos:note` sub-properties.
- [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md) — Ontology document shape; governs namespace and prefix conventions.
