# ADR-O

ADR-O is an RDF/OWL 2 ontology for Any Decision Record (ADR): any load-bearing decision — in any domain — worth preserving with its context, rationale, and alternatives as a machine-traversable graph. The goal is to represent decisions as a linked graph rather than isolated files, so both humans and tools can recover not just what was decided, but why — including how decisions relate to one another, including the explicit inference from framing concerns to selected options.

> The acronym explicitly traces its roots back to software architecture, where ADR = Architecture Decision Record, and it was preserved in this project's name specifically for its rich heritage. See [ADR-0002](/ADL/ADR-0002-ontology-scope.md) for more on this.

## Status

The canonical ontology is **[`ontology/adr-o.ttl`](/ontology/adr-o.ttl)** at **`0.5.2-draft`** (`owl:versionIRI` `https://w3id.org/adr-o/0.5.2`). Treat this as a **technical preview**: versions in the 0.x line include intentional breaking redesigns before a stable release.

A **SHACL** shapes companion for validation is planned but **not shipped**; some integrity rules are documented as authoring conventions or deferred to tooling layers until that ships. See [`ontology/DESIGN-NOTES.md`](/ontology/DESIGN-NOTES.md) for the per-version rationale.

ADR lifecycle and status tracking are maintained in the **[ADL Index file](/ADL/ADL.yaml)** (kept in sync with the ADR markdown files); use the index as the operational source of truth.

**Human-facing documentation:** [https://gutza.github.io/ADR-O/](https://gutza.github.io/ADR-O/) (also linked from the ontology via `rdfs:seeAlso`).

## Why ADR-O

Traditional ADRs are useful for people but hard for machines to query consistently. ADR-O makes decision history explicit and durable as a graph:

- **`DecisionRecord`** as the hub: metadata via Dublin Core Terms; typed links to other records (for example `supersedes`, `amends`, `amendedBy`, `dependsOn`, `enables`, `conflictsWith`, `addresses`) plus the ADL-scope causal family (`constrainedBy`, `prohibitedBy`, `recommendedBy`, `discouragedBy`, `permittedBy`); bibliographic links via `dcterms:references` to external or non-record resources; explicit snout modeling via `promptedBy` to a `Complaint` (with `namedBy` stakeholder provenance); explicit social-role predicates (`authoredBy`, `decidedBy`, `consulted`, `informed`); an option pool (`hasAlternative`) and a chosen option (`chosenAlternative`). Resource provenance flows inward only: `justifiedBy` from a resource to the decision that warranted it — never outward from the decision.
- **Atom-first content:** reusable **`Claim`** nodes placed by reified **`ContextFact`**, **`DeliberationFact`**, and **`ExpectedOutcome`** links into framing, deliberation, and expected-outcome roles, with ordering on the record and valence schemes for deliberation and expected-outcome facts. Status and valences use SKOS concept schemes. Cross-record `Claim` reuse is reference-based via `derivedFrom`/`derives` rather than identity reuse, preserving each record's epistemic transaction boundary.
- **Design stance:** the RDF graph is the authoritative substrate. Human-readable Markdown remains a companion representation for discussion and publication, while the graph carries the canonical semantics. Where the graph does carry prose literals (primarily on `Claim`, `DecisionRecord`, and annotated referenced resources via `dcterms:description`, `skos:definition`, and `skos:note`), they are typed as `^^adr-o:md`, an ergonomic alias equivalent to `^^<https://www.w3.org/ns/iana/media-types/text/markdown>` per ADR-0038 and ADR-0003 (see DESIGN-NOTES for the full argument).

Together, this supports reliable graph traversal and richer analysis than keyword search over prose alone. Agents and services can consume **explicit triples** instead of re-inferring a decision graph from unstructured text every time.

## Design direction

ADR-O aims for the **smallest complete domain-agnostic core**: the vocabulary is not locked to any domain's class names or concerns — domain-specific terms belong in **profiles** in their own namespaces. Software architecture teams are a natural first-adopter audience, but the core is designed for any decision-making context.

The ontology **reuses** Dublin Core Terms, aligns supersession with PROV-O (`supersedes` as a sub-property of `prov:wasRevisionOf`), and uses SKOS for controlled values. It **defines only** ADR-specific terms that are missing from those stacks. It **aligns with** ISO 42010-style separation of concerns (and lessons from OntolAD) without importing heavy external ontologies as `owl:imports`; this document-shape discipline is codified in [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md) (with inception context in [ADR-0000](/ADL/ADR-0000-inception.md)).

## Repository map (start here)

| Location | Content |
|--|--|
| [`MANIFESTO.md`](/MANIFESTO.md) | Motivation and north star. |
| [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) | Canonical OWL 2 ontology (Turtle). |
| [`ontology/DESIGN-NOTES.md`](/ontology/DESIGN-NOTES.md) | Immutable notes per ontology iteration. |
| [`ontology/IDEAS.md`](/ontology/IDEAS.md) | Design exploration and backlog-style ideas. |
| [`ADL/ADL.yaml`](/ADL/ADL.yaml) | The ADL Index file. |
| [`Archive/index.md`](/Archive/index.md) | Archived references and outreach materials. |

## Intended outcomes

ADR-O is intended to enable:

- machine-interpretable decision history across system (or organisational) evolution;
- consistent querying of rationale, options, and trade-offs;
- better support for AI-assisted decision workflows grounded in explicit graphs.

## Namespace and license

- **Namespace:** `https://w3id.org/adr-o#`
- **License:** Creative Commons Attribution 4.0 International (CC BY 4.0) — see [`LICENSE`](/LICENSE) in the repository; the ontology document also declares `dcterms:license` for machine-readable use.

## Contributing

Contributions and critique are welcome, especially on class and property naming, relationship semantics, constraints, and interoperability with linked-data and architecture-description tooling.

For substantive vocabulary changes, grounding in [`ontology/DESIGN-NOTES.md`](/ontology/DESIGN-NOTES.md) and/or a new **ADL** record helps. When reporting issues, **Turtle fragments** or **SPARQL** examples that show the intended shape or query make proposals much easier to evaluate.

As pre-1.0 ontology work, expect iteration; please prefer discussions and issues with concrete use cases and examples.
