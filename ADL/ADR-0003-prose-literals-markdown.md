---
id: 3
type: Core vocabulary
status: Accepted
date: 2026-04-19
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amendedBy:
  - ADR-0038
---

# ADR-0003 — Prose Literals Are Markdown

## Context

Markdown is the de facto format for human-facing technical prose across three otherwise unrelated tooling cohorts that will likely consume or produce ADR-O graphs:
1. MADR, the dominant ADR template in active use, is Markdown.
2. The LLM tooling that will likely draft and process ADRs operates in Markdown by default — it's what the models were trained on for technical prose, and what the surrounding agent infrastructure passes around by default.
3. The linked-data ecosystem has independently moved the same way for human-readable annotation content, to the point where pyLODE — the leading Python OWL documentation generator — carries an open community request for Markdown handling on `rdfs:comment`, `skos:definition`, and `dcterms:description`.

**ADR-O commits to Markdown for its prose-carrying literals on the strength of that convergence.**

### Closing off two more ambitious framings

Two ambitious framings of MADR compatibility were considered and rejected before settling on this narrower commitment.

**Strict-superset proposition:** ADR-O as a strict superset of MADR, the way Markdown is a strict superset of HTML: every MADR document already a valid ADR-O document, lossless round-trip, zero migration cost. Although that sounds exciting at first, it is a category error. Markdown has a *syntactic*-containment relationship to HTML: the subset format's bytes are valid syntax in the superset format, an HTML document is already a valid Markdown document. By contrast, Turtle and Markdown are incommensurable syntaxes — a Markdown document isn't a "simple" Turtle document — and no design ingenuity can manufacture the syntactic-containment relation that the underlying grammars forbid.

**Semantic equivalence proposition** as a defensible fallback: if syntactic containment is off the table (above), then enforce semantic round-trip — a MADR document parses into an ADR-O graph that serializes back to a semantically equivalent MADR document. This doesn't work specifically because ADR-O is the superior format. ADR-O's formalized deliberation valences (`Supports`, `Against`, `Neutral`) and outcome valences (`Benefit`, `AcceptedCost`, `Risk`, `FollowUp`) are typed structures over reified facts; MADR's "Good, because…" / "Bad, because…" lists are prose bullets. Converting prose into typed triples and subsequently regenerating prose from those typed triples does not reproduce the original text; strong losslessness would require the Turtle graph to store the original prose as primary and treat the typed triples as derived, which would wag the dog by inverting the purpose of the ontology;

What survives both rejections is a design principle, not a structural relationship: prose literals in ADR-O are Markdown. No claim of compatibility, syntactic or semantic, with MADR as a document class.

### The standardization landscape

RDF 1.1 (W3C Recommendation, February 2014) defines `rdf:HTML` and `rdf:XMLLiteral` but no Markdown analogue, for historical reasons: `text/markdown` was provisionally registered with IANA in November 2014 — months after RDF 1.1 was finalized — and codified by RFC 7763 in March 2016. Current RDF 1.2 working drafts do not add the datatype either.

The W3C maintains a namespace, `https://www.w3.org/ns/iana/media-types/`, whose explicit purpose is to mint a stable IRI for every IANA-registered media type. This makes `https://www.w3.org/ns/iana/media-types/text/markdown` a real, standards-grounded identifier rather than a coined one — the relevant distinction for an ontology trying to avoid minting vocabulary that already exists.

Out of scope: Markdown-LD / MD-LD and similar projects that embed RDF *inside* Markdown. They are solving the inverse problem (RDF as payload, Markdown as carrier).

## Decision

**ADR-O explicitly adopts a Markdown datatype for all prose-carrying literals**, with the datatype IRI rooted in the W3C IANA media-types namespace:

```
"…the literal text…"^^<https://www.w3.org/ns/iana/media-types/text/markdown>
```

### Properties in scope

The convention applies to every literal that carries multi-paragraph or formatting-bearing prose. In the present ontology that is `dcterms:description` (used today on the ontology IRI itself, expected on `DecisionRecord` and `Consideration` instances); `skos:prefLabel`, `skos:definition`, and `skos:note` (and its sub-properties) on `Consideration`, on the SKOS concept individuals, and on the concept schemes; and the corresponding prose properties on `ContextFact`, `DeliberationFact`, and `OutcomeFact` when authors attach prose to a fact directly rather than through its `consideration` link. `rdfs:label` is not in scope — a label is a short string identifier, not prose.

### Choice of IRI form

The W3C namespace describes its convention in two slightly different voices: the normative prose says the RDF identifier for a media type is "the URI… of this directory concatenated with that media type" — yielding the bare form — while the worked example illustrates the resulting class IRI with a `#Resource` fragment (`…/image/png#Resource`). Both forms appear in published RDF in the wild.

**ADR-O adopts the bare form `https://www.w3.org/ns/iana/media-types/text/markdown`**: it is what the namespace's normative text names, it's shorter in serialised Turtle, and is what most extant tooling that recognises any IANA-rooted IRI compares against. If a future W3C clarification establishes the `#Resource` form as canonical, the change is non-destructive — datatype IRIs in stored data can be aliased rather than rewritten, and the ontology can publish both forms in parallel for a deprecation window.

### Why the IANA IRI rather than a custom `adr-o:markdown`

Minting `adr-o:markdown` would invent vocabulary that already exists. The IANA-rooted W3C IRI is authoritative, standards-grounded, and reusable by any ontology that needs the same datatype.

### Where the constraint lands

A `skos:scopeNote` (and, where appropriate, an `rdfs:range` annotation) on the affected datatype properties, to be added in the next ontology iteration, plus an `sh:datatype` constraint in the planned SHACL companion graph when that ships.

### Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Bare `xsd:string` | Makes no commitment, so every consuming tool guesses how to render the literal and SHACL validation of literal format is inexpressible. |
| Custom `adr-o:markdown` datatype IRI | Invents vocabulary that already exists; the IANA-rooted W3C IRI provides exactly this datatype with stronger provenance. |
| `rdf:HTML` | The nearest W3C-standard analogue, but misrepresents the content type. Tools that honour `rdf:HTML` will apply HTML rendering to text that is not HTML, producing visible errors rather than silent degradation. |
| MADR strict-superset framing | Not an implementation option but a rejected framing of the underlying compatibility goal; the Context section explains why it fails. |

A two-format approach — Markdown and Turtle as co-equal canonical serialisations of an ADR-O record — was raised and dismissed without a row in the table: it would require restructuring ADR-O's serialization strategy at a level well above this property-level decision, with no upside proportionate to the complexity.

## Consequences

**Positive:** Tooling has a contract: renderers, LLM writer services, and SHACL validators can act on the datatype IRI as a stable signal rather than guessing from content. SHACL validation of literal format becomes expressible (`sh:datatype <…/text/markdown>`), unlocking integrity checks that bare `xsd:string` literals foreclose. Domain profiles inherit the convention by default for any prose property they introduce on top of the core.

**Negative / risks:** The IANA-rooted IRI is not recognised by any major OWL reasoner or SPARQL engine as a built-in datatype. It will be treated as an unknown custom datatype — legal under OWL semantics, but no built-in datatype reasoning applies, and conformance checkers that do not specifically handle this IRI will silently accept any string as a member of it. This is a documentation-and-tooling responsibility, not a structural flaw, but it must be on record.

**Open questions:**
- Which Markdown dialect is assumed? CommonMark is the obvious specification-level default; GitHub-Flavoured Markdown is the practical reality for most teams. This ADR takes no position; whether one should be taken in the core or deferred to domain profiles is the live question.
- Whether `rdfs:range <https://www.w3.org/ns/iana/media-types/text/markdown>` should be asserted directly on the affected properties in the OWL file, deferred entirely to SHACL, or both. The working answer is "annotation now, SHACL constraint when shipped", but the choice is not finalized.

## References

- IETF RFC 7763 (March 2016). *The text/markdown Media Type.* https://www.rfc-editor.org/info/rfc7763 — establishes `text/markdown` as the IANA-registered media type for Markdown content.
- W3C IANA Media Types namespace. https://www.w3.org/ns/iana/media-types/ — the artefact that makes `https://www.w3.org/ns/iana/media-types/text/markdown` a standards-grounded identifier rather than an invented one.
- pyLODE issue #68. https://github.com/RDFLib/pyLODE/issues/68 — open community request for Markdown literal handling on `rdfs:comment`, `skos:definition` and sibling SKOS note properties, and `dcterms:description`. Cited as a signal of independent community demand for the same convention; not a claim that pyLODE already implements the IANA IRI form.
- ADR-0002 — Ontology Scope (this ADL). Establishes that decisions like this one apply to the domain-agnostic core; domain-specific profiles inherit the convention but may further constrain it (e.g. by pinning a Markdown dialect).
- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) and [`ontology/DESIGN-NOTES.md`](/ontology/DESIGN-NOTES.md) — the artefacts the convention will land in once the next ontology iteration applies it.
