# ADR-O

ADR-O is an RDF/OWL 2 ontology for Any Decision Record (ADR): any load-bearing decision — in any domain — worth preserving with its context, rationale, and alternatives as a machine-traversable graph. The goal is to represent decisions as a linked graph rather than isolated files, so both humans and tools can recover not just what was decided, but why — including how decisions relate to one another.

> The acronym explicitly traces its roots back to software architecture, where ADR = Architecture Decision Record, and it was preserved in this project's name specifically for its rich heritage. See [ADR-0002](/ADL/ADR-0002-ontology-scope.md) for more on this.

## Status

The canonical ontology is **[`ontology/adr-o.ttl`](/ontology/adr-o.ttl)** at **`0.4.2-draft`** (`owl:versionIRI` `https://w3id.org/adr-o/0.4.2`). Treat this as a **technical preview**: versions in the 0.x line include intentional breaking redesigns before a stable release.

A **SHACL** shapes companion for validation is planned but **not shipped**; some integrity rules are documented as authoring conventions or deferred to tooling layers until that ships. See [`ontology/DESIGN-NOTES.md`](/ontology/DESIGN-NOTES.md) for the per-version rationale.

**Human-facing documentation:** [https://gutza.github.io/ADR-O/](https://gutza.github.io/ADR-O/) (also linked from the ontology via `rdfs:seeAlso`).

## Why ADR-O

Traditional ADRs are useful for people but hard for machines to query consistently. ADR-O makes decision history explicit and durable as a graph:

- **`DecisionRecord`** as the hub: metadata via Dublin Core Terms; typed links to other records (for example `supersedes`, `amends`, `amendedBy`, `dependsOn`, `enables`, `conflictsWith`, `addresses`); explicit social-role predicates (`authoredBy`, `decidedBy`, `consulted`, `informed`); an option pool (`hasAlternative`) and a chosen option (`chosenAlternative`). Resource provenance flows inward only: `justifiedBy` from a resource to the decision that warranted it — never outward from the decision.
- **Atom-first content:** reusable **`Claim`** nodes placed by reified **`ContextFact`**, **`DeliberationFact`**, and **`ExpectedOutcome`** links into framing, deliberation, and expected-outcome roles, with ordering on the record and valence schemes for deliberation and expected-outcome facts. Status and valences use SKOS concept schemes. Cross-record `Claim` reuse is reference-based via `derivedFrom`/`derives` rather than identity reuse, preserving each record's epistemic transaction boundary.
- **Design stance:** the RDF graph is the authoritative substrate. Human-readable Markdown is expected to be **materialised from** the graph by tools, not stored as the core shape of the record. Where the graph does carry prose literals (primarily on `Claim` and `DecisionRecord` via `dcterms:description`, `skos:definition`, and `skos:note`), they are typed as `^^<https://www.w3.org/ns/iana/media-types/text/markdown>` per ADR-0003 (see DESIGN-NOTES for the full argument).

Together, this supports reliable graph traversal and richer analysis than keyword search over prose alone. Agents and services can consume **explicit triples** instead of re-inferring a decision graph from unstructured text every time.

## Design direction

ADR-O aims for the **smallest complete domain-agnostic core**: the vocabulary is not locked to any domain's class names or concerns — domain-specific terms belong in **profiles** in their own namespaces. Software architecture teams are a natural first-adopter audience, but the core is designed for any decision-making context.

The ontology **reuses** Dublin Core Terms, aligns supersession with PROV-O (`supersedes` as a sub-property of `prov:wasRevisionOf`), and uses SKOS for controlled values. It **defines only** ADR-specific terms that are missing from those stacks. It **aligns with** ISO 42010-style separation of concerns (and lessons from OntolAD) without importing heavy external ontologies as `owl:imports`; this document-shape discipline is codified in [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md) (with inception context in [ADR-0000](/ADL/ADR-0000-inception.md)).

## Repository map (start here)

| | |
|--|--|
| [`MANIFESTO.md`](/MANIFESTO.md) | Motivation and north star. |
| [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) | Canonical OWL 2 ontology (Turtle). |
| [`ontology/DESIGN-NOTES.md`](/ontology/DESIGN-NOTES.md) | Immutable notes per ontology iteration (0.1.0-draft, 0.2.0-draft, 0.2.1-draft, 0.2.2-draft, 0.2.3-draft, 0.2.4-draft, 0.2.5-draft, 0.2.6-draft, 0.3.0-draft, 0.4.0-draft, 0.4.1-draft, 0.4.2-draft). |
| [`ontology/IDEAS.md`](/ontology/IDEAS.md) | Design exploration and backlog-style ideas. |
| [ADR-0000](/ADL/ADR-0000-inception.md) | Inception and survey of prior art. |
| [ADR-0001](/ADL/ADR-0001-license.md) | License (CC BY 4.0). |
| [ADR-0002](/ADL/ADR-0002-ontology-scope.md) | ADR = Any Decision Record; domain-agnostic core and extension pattern. |
| [ADR-0003](/ADL/ADR-0003-prose-literals-markdown.md) | Markdown as the format for prose-carrying literals. |
| [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md) | KG as substrate: tooling mediates all user interaction. |
| [ADR-0005](/ADL/ADR-0005-log-all-decisions.md) | No entry gate beyond domain relevance; real-time and post-factum ADRs are first-class. |
| [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md) | Core relational predicates and PROV alignment (`addresses`, `supersedes`, …). |
| [ADR-0007](/ADL/ADR-0007-index-vs-identifier.md) | `adr-o:index` vs `dcterms:identifier`. |
| [ADR-0008](/ADL/ADR-0008-status-skos-and-integrity.md) | Lifecycle status as a SKOS scheme with OWL integrity. |
| [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md) | Deliberation and expected-outcome valence schemes. |
| [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md) | Atom-first `Consideration` and `*Fact` link classes (term renamed to `Claim` by ADR-0031). |
| [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md) | Strict graph: no Nygard body literals on the record. |
| [ADR-0012](/ADL/ADR-0012-alternatives-and-chosen-option.md) | Alternatives, chosen option, functional edges. |
| [ADR-0013](/ADL/ADR-0013-addresses-dual-placement.md) | `addresses`: canonical vs record-level rollup. |
| [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md) | Ontology document: namespace, version IRIs, no `owl:imports`. |
| [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md) | Canonical Dublin Core and PROV usage. |
| [ADR-0016](/ADL/ADR-0016-dcterms-version-in-record.md) | `dcterms:version` for in-record iteration. |
| [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md) | Markdown prose properties as implemented (clarifies ADR-0003). |
| [ADR-0018](/ADL/ADR-0018-consideration-iri-reuse-convention.md) | ~~Editorial `Consideration` IRI reuse convention.~~ *(Superseded by ADR-0028: identity reuse replaced by reference reuse via `derivedFrom`/`derives`.)* |
| [ADR-0019](/ADL/ADR-0019-clarifications-are-amendments.md) | Unified amendment model: clarifications collapse into amendments. |
| [ADR-0020](/ADL/ADR-0020-amendments.md) | Amendment predicates and rejection of an `Amended` status. |
| [ADR-0021](/ADL/ADR-0021-social-role-predicates.md) | Social role predicates (`authoredBy`, `decidedBy`, `consulted`, `informed`) with explicit RACI mapping. |
| [ADR-0022](/ADL/ADR-0022-creator-authoredby-coexistence.md) | `dcterms:creator` / `adr-o:authoredBy` coexistence: tooling-mediated expansion pattern. |
| [ADR-0023](/ADL/ADR-0023-add-hastype-annotation-property.md) | `adr-o:hasType` as record-level type metadata via profile-defined SKOS concepts. |
| [ADR-0024](/ADL/ADR-0024-sequential-ids-for-all-decision-records.md) | Sequential IDs for all records, including amendments; topology carries amendment role. |
| [ADR-0025](/ADL/ADR-0025-causal-network.md) | Three-scope causal topology and project materialization/provenance predicates. |
| [ADR-0026](/ADL/ADR-0026-ontology-provenance.md) | Ontology provenance via `adr-odr` namespace and `adr-o:justifiedBy` citations. |
| [ADR-0027](/ADL/ADR-0027-facts-manifest-considerations.md) | `adr-o:manifests` replaces `adr-o:consideration` for Fact → Consideration placement (`Claim` in current core vocabulary). |
| [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) | Decision Transaction Principle: removes `affects`/`materializes`, adds `derivedFrom`/`derives`, renames `OutcomeFact` → `ExpectedOutcome` and rebalances outcome valences. |
| [ADR-0029](/ADL/ADR-0029-provenance-correction-consideration-identity.md) | Corrects provenance attribution for intra-record Consideration identity as an ADR-O contribution (current term: `Claim`). |
| [ADR-0030](/ADL/ADR-0030-observations.md) | Adds `Observation` with `verifies` and `hasVerdict` as the t_n validation bridge back to t_0 `ExpectedOutcome` commitments. |
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
