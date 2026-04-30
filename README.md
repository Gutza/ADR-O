# ADR-O

**ADR-O is the RDF/OWL 2 ontology for Any Decision Record (ADR)**, *modeling any set of decisions worth preserving, with their context, rationales, and alternatives as machine-traversable graphs*, so both humans and AI agents can recover not just what was decided, but why — and how each decision relates to every other.

> The acronym explicitly traces its roots back to software architecture, where *ADR = Architecture Decision Record*. The acronym was preserved specifically for its rich heritage; see [ADR-0002](/ADL/ADR-0002-ontology-scope.md) for more on this.

The canonical ontology is **[`adr-o.ttl`](/adr-o.ttl)**, currently at version **`0.6.0-alpha`**. The project's own ADR lifecycle and status tracking are maintained in the **[ADL Index file](/ADL/ADL.yaml)**.

## Why ADR-O

Traditional ADRs are useful for people but hard for machines to query consistently. A folder of Markdown files answers "what did we decide?" only if you can read every file. ADR-O makes decision history explicit and durable as a graph, so an agent can answer "what constrains this choice?", "what was the reasoning that led away from this approach?", or "what is the full supersession chain from the founding decision to what is in effect today?" — by traversal, not by reading prose.

The model is organized around a `DecisionRecord` as the central node, connected outward at two levels:

**At ADL scope** — across records — typed links form the traversable backbone: `supersedes`, `amends`, `dependsOn`, `enables`, `conflictsWith`, and the causal-modal family (`constrainedBy`, `prohibitedBy`, `recommendedBy`, `discouragedBy`, `permittedBy`). This is where machine-readable reasoning lives.

**At ADR scope** — inside a single record — reified fact nodes make each part of the decision locatable without prose parsing: a `Complaint` (the raw triggering situation, via `promptedBy`), `ContextFact` and `DeliberationFact` nodes manifesting reusable `Claim` atoms, a `Verdict` (the reified convergence act, via `closedBy`), `ExpectedOutcome` commitments, and `ObservedOutcome` nodes closing the learning loop at tₙ. Social roles (`authoredBy`, `decidedBy`, `consulted`, `informed`) and bibliographic links (`dcterms:references`) round out the record.

All prose-carrying literals are typed `^^adr-o:md` (an ergonomic alias for the IANA text/markdown datatype), making them renderer-friendly without losing OWL compliance.

## Design direction

ADR-O aims for the **smallest complete domain-agnostic core**: the vocabulary is not locked to any domain's class names or concerns — domain-specific terms belong in **profiles** in their own namespaces. Software architecture teams are a natural first-adopter audience, but the core is designed for any decision-making context.

The ontology **reuses** Dublin Core Terms, aligns supersession with PROV-O (`supersedes` as a sub-property of `prov:wasRevisionOf`), and uses SKOS for controlled values. It **defines only** ADR-specific terms that are missing from those stacks. It **aligns with** ISO 42010-style separation of concerns (and lessons from OntolAD) without importing heavy external ontologies as `owl:imports`; this discipline is codified in [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md).

The modeling priority is **efficient agent navigation of the ADL**, not formal completeness inside individual records. ADR scope earns formal structure when it enables locatability — giving an agent a first-class node to find and read — not when it attempts to encode the internal logic of a human deliberation. See [*ADR-O Modeling Scope Priorities*](/Archive/ADR-O%20Modeling%20Scope%20Priorities.md) for the full argument.

## Minimal example

A complete (minimal) `DecisionRecord` in Turtle — one complaint, two alternatives, one chosen, one context fact and one deliberation fact each:

```turtle
@prefix :        <https://example.org/adl/> .
@prefix adr-o:   <https://w3id.org/adr-o#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix xsd:     <http://www.w3.org/2001/XMLSchema#> .
@prefix skos:    <http://www.w3.org/2004/02/skos/core#> .

# Complaint — the triggering situation
:complaint-001 a adr-o:Complaint ;
    dcterms:description "Our REST API responses are too slow under peak load."^^adr-o:md .

# Claims — reusable atomic propositions
:claim-latency-sla a adr-o:Claim ;
    skos:prefLabel "Latency SLA at risk"@en ;
    dcterms:description "P99 latency exceeds the 200 ms SLA under loads above 500 RPS."^^adr-o:md .

:claim-cache-complexity a adr-o:Claim ;
    skos:prefLabel "Cache invalidation complexity"@en ;
    dcterms:description "Introducing a cache layer adds invalidation complexity and risk of stale data."^^adr-o:md .

# Alternatives
:alt-cache a adr-o:Alternative ;
    skos:prefLabel "Add Redis caching layer"@en .

:alt-scale-db a adr-o:Alternative ;
    skos:prefLabel "Scale up the database"@en .

# Decision Record
:ADR-001 a adr-o:DecisionRecord ;
    dcterms:identifier    "ADR-001" ;
    adr-o:index           1 ;
    dcterms:title         "API Latency: caching vs. database scaling"@en ;
    adr-o:decidedAt       "2025-11-03"^^xsd:date ;
    adr-o:hasStatus       adr-o:Accepted ;
    adr-o:promptedBy      :complaint-001 ;
    adr-o:hasAlternative  :alt-cache, :alt-scale-db ;
    adr-o:chosenAlternative :alt-cache ;

    adr-o:hasContext (
        [ a adr-o:ContextFact ;
          adr-o:manifests :claim-latency-sla ]
    ) ;

    adr-o:hasDeliberation (
        [ a adr-o:DeliberationFact ;
          adr-o:onAlternative    :alt-cache ;
          adr-o:manifests        :claim-cache-complexity ;
          adr-o:deliberationValence adr-o:Against ]
    ) .
```

The full vocabulary is documented in the ontology and in the ADL — `ExpectedOutcome`, `ObservedOutcome`, `Verdict`, social roles, inter-record predicates, bibliographic links.

## Repository map

| Location | Content |
|--|--|
| [`MANIFESTO.md`](/MANIFESTO.md) | Motivation and north star. |
| [`adr-o.ttl`](/adr-o.ttl) | Canonical OWL 2 ontology (Turtle). |
| [`BACKLOG.md`](/BACKLOG.md) | Open questions and backlog. |
| [`ADL/ADL.yaml`](/ADL/ADL.yaml) | The ADL Index file. |
| [`Archive/index.md`](/Archive/index.md) | References, outreach materials, and methodology documents. |

## Namespace and license

- **Namespace:** `https://w3id.org/adr-o#`
- **License:** Creative Commons Attribution 4.0 International (CC BY 4.0) — see [`LICENSE`](/LICENSE) in the repository; the ontology document also declares `dcterms:license` for machine-readable use.

## Contributing

Contributions and critique are welcome, especially on class and property naming, relationship semantics, constraints, and interoperability with linked-data and architecture-description tooling.

For substantive vocabulary changes, a new **ADL** record helps. When reporting issues, **Turtle fragments** or **SPARQL** examples that show the intended shape or query make proposals much easier to evaluate.

As pre-1.0 ontology work, expect iteration; please prefer discussions and issues with concrete use cases and examples.
