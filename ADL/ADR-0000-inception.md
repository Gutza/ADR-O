---
id: 0
type: Inception record
status: Accepted
date: 2026-04-17
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0000 — Inception Record: A Greenfield RDF Ontology for Architecture Decision Records

## Context

Architecture Decision Records (ADRs) are a lightweight but valuable practice for capturing significant software design choices together with their context and rationale. The practice was popularised by Michael Nygard's 2011 blog post and has since been formalised in templates ranging from the minimal Nygard format (title / status / context / decision / consequences) to the richer MADR and Tyree–Akerman templates. The broader academic field is Architectural Knowledge Management (AKM).

The dominant storage medium for ADRs is Markdown files in a source repository (`doc/adr/`). This works well for human readers but poorly for machines: the ADL (Architecture Decision Log) cannot be queried, reasoned over, or linked to other knowledge assets. An RDF representation would address this gap, enabling SPARQL queries across a decision log, integration with other linked data, and OWL reasoning over decision relationships.

### Survey of existing work

A survey conducted on 2026-04-17 found the following candidates.

#### Kruchten (2004) — "An Ontology of Architectural Design Decisions in Software-Intensive Systems"

The foundational conceptual work in this space. Defines decision attributes (state, scope, chronology) and a rich vocabulary of inter-decision relationships: *supersedes*, *depends-on*, *prevents*, *enables*, *is-related-to*. Also defines a decision state machine with states including *idea*, *tentative*, *decided*, *approved*, *challenged*, *obsolete*.

This is the intellectual ancestor of everything in the AKM space. However, it was published as a conference paper; no OWL or RDF artefact was ever produced.

#### ArchiMind / UvA-SA-ontology (de Graaf et al., 2011–2018)

A semantic wiki for software architecture documentation, produced by the VU Amsterdam group. Hosted at `github.com/kadevgraaf/ArchiMind`. The repository contains an example knowledge-base file (`Example SA doc ontology.rdf`) that is labelled "Architecture Knowledge Domain Ontology."

On inspection, the ontology is not a standalone artefact. It is a populated instance dump from an OntoWiki installation, serialised as RDF/XML with opaque numeric fragment identifiers (e.g. `Ontology1308582601209#7`), hard-coded `localhost` namespace URIs, and no stable IRI scheme. The associated AK-Finder tool is a useful proof-of-concept for SPARQL-based retrieval of architectural knowledge, but the ontology itself is not reusable. The project is abandoned (last commit 2018).

**Verdict: not reusable.**

#### OntolAD (Guessi et al., SAC 2015)

A formal OWL 2 ontology for the ISO/IEC/IEEE 42010 standard for architecture descriptions. Produced at the University of São Paulo. Licensed CC BY 4.0. The OWL file (`ontolad.owl`) was obtained and inspected in full.

**Genuine strengths.** OntolAD is clean, well-formed OWL 2 XML serialisation (OWL API 3.5). It is grounded in ISO 42010, which is the standard that formally defines what an architecture description *is*. The class hierarchy is principled: `ArchitectureDescriptionElement` as a covering superclass, with `ArchitectureDecision`, `ArchitectureRationale`, `ArchitectureDescription`, `ArchitectureViewpoint`, `Concern`, `Stakeholder`, and others as named subclasses. Data properties include `Status`, `Author`, `IssueDate`, `Version`, `Summary`, and `ChangeHistory` — a useful starting set for ADR metadata. The `affects` object property (Decision → ArchitectureDescriptionElement) and `isJustifiedBy` / `justifies` inverse pair (Decision ↔ Rationale) are architecturally correct.

**Critical weaknesses for the ADR use case.**

1. *Isolation.* OntolAD imports nothing. It does not align with PROV-O, DC Terms, SKOS, FOAF, or any other standard vocabulary. Every concept is defined from scratch. A consumer wishing to use it alongside provenance tooling, metadata standards, or controlled vocabularies must perform all alignment work themselves.

2. *`ArchitectureDecision` is barely specified.* The OWL 2 equivalent-class axiom defines an `ArchitectureDecision` as: an `ArchitectureDescriptionElement` that `affects` some `ArchitectureDescriptionElement` and `isJustifiedBy` some `ArchitectureRationale`. This is necessary but far from sufficient. There is no `AlternativeOption` class, no inter-decision relationships (the Kruchten vocabulary is cited in the paper's references but not implemented), no decision lifecycle beyond a bare `xsd:string` `Status` property, and no modelling of what a decision *addresses* (the problem / ASR) separately from what it *changes* (the consequences).

3. *`Status` is an uncontrolled string.* Without a SKOS concept scheme, the property cannot be reasoned over and there is no canonical vocabulary for values like `proposed`, `accepted`, `superseded`, `deprecated`, or `rejected`.

4. *Heavy closure axioms.* The axiom set is optimised for 42010 conformance checking, not for lightweight ADR storage. The open-world-assumption mitigations add reasoning overhead that is irrelevant to the ADR use case.

5. *`affects` conflates two distinct relations.* In ADR terms, a decision both *addresses* a problem (the context / ASR) and *has consequences* for the system. OntolAD uses a single property for both, losing the distinction.

**Verdict: the concepts are correct and the licence is permissive, but the ontology's isolation and incompleteness make it a poor foundation. Importing it and wiring everything downstream is more work than defining the needed concepts afresh, with better alignment.**

#### ISO/IEC/IEEE 42010 itself

The standard defines a semi-formal UML conceptual model. OntolAD is the only known OWL 2 formalisation of it. The standard is relevant as a reference vocabulary but not directly reusable as an RDF artefact.

## Decision

**Develop a greenfield RDF/OWL 2 ontology for ADRs**, importing established W3C vocabularies for all concepts they already cover and defining only the genuinely novel ADR-specific terms.

The ontology will be small and focused. Approximate scope:

- **Reuse without modification:** DC Terms (metadata), PROV-O (provenance, supersession chains, agent attribution), SKOS (controlled status vocabulary and any other concept schemes).
- **Align to, but not import:** ISO 42010 / OntolAD terminology will inform naming and definitions; `owl:equivalentClass` or `rdfs:seeAlso` annotations will record the alignment without creating a hard dependency on OntolAD's closed island.
- **Define fresh:** `ArchitectureDecisionRecord`, `Alternative`, `hasStatus` (with range `skos:Concept`), `supersedes`, `dependsOn`, `enables`, `conflictsWith`, `addresses` (linking a decision to the Concern / ASR it resolves), `hasAlternative`, `wasRejectedBecause`.

The status concept scheme will include at minimum: `Proposed`, `Accepted`, `Deprecated`, `Superseded`, `Rejected`. The `supersedes` property, combined with PROV-O's `prov:wasRevisionOf`, will make the decision history machine-traversable.

### Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Import OntolAD as base layer | Isolation cost exceeds reuse benefit; axiom set is mismatched to the use case; all downstream wiring (SKOS, PROV-O, DC Terms) would have to be done anyway |
| Extend ArchiMind ontology | Not a reusable ontology; opaque IRIs; abandoned project |
| Adopt Kruchten 2004 directly | No OWL artefact exists; the conceptual model is excellent input but not a starting point |
| Wait for a community standard | No active standardisation effort was found; the gap between the ADR practitioner community and the semantic web community shows no sign of closing organically |

## Consequences

**Positive.** The resulting ontology will be natively aligned with the broader linked data ecosystem from day one. Consumers can use standard SPARQL, PROV-O reasoning, and SKOS tooling without any bridging layer. The ontology will be small enough to be fully understood and maintained by a single person.

**Negative / risks.** Without an existing community, adoption depends entirely on outreach. The alignment to ISO 42010 / OntolAD will be informal (annotation-level), which means automatic interoperability with OntolAD-based tools is not guaranteed.

**Open questions.**
- Whether to publish under a namespace with long-term stability guarantees (e.g. `w3id.org` redirect) or a project-owned domain.
- Whether `conflictsWith` should be a symmetric property or two directed properties (`prevents`, `isPreventedBy`) following Kruchten's model.
- How to represent the *forces* / trade-off narrative that many ADR templates include — as a datatype property on the record, or as a first-class `Force` class linked to both the decision and affected quality attributes.
- Whether to define a SHACL shapes graph alongside the OWL ontology to support validation of ADR instances without requiring a full OWL reasoner.

## References

- Nygard, M. (2011). *Documenting Architecture Decisions.* Blog post. https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions
- Kruchten, P. (2004). An Ontology of Architectural Design Decisions in Software-Intensive Systems. *2nd Groningen Workshop on Software Variability Management*, pp. 54–61.
- Kruchten, P., Lago, P., van Vliet, H. (2006). Building Up and Reasoning About Architectural Knowledge. *QoSA 2006*, LNCS 4214.
- de Graaf, K.A., Liang, P., Tang, A., van Vliet, H. (2014). An Exploratory Study on Ontology Engineering for Software Architecture Documentation. *Computers in Industry*, 65(7), 1053–1064.
- Guessi, M., Moreira, D.A., Abdalla, G., Oquendo, F., Nakagawa, E.Y. (2015). OntolAD: a Formal Ontology for Architectural Descriptions. *SAC 2015*, pp. 1417–1424. doi:10.1145/2695664.2695739, saved to [/Archive/OntolAD/](/Archive/OntolAD/)
- ISO/IEC/IEEE 42010:2011. *Systems and Software Engineering — Architecture Description.*
- W3C PROV-O: https://www.w3.org/TR/prov-o/
- SKOS: https://www.w3.org/TR/skos-reference/
- Dublin Core Terms: https://www.dublincore.org/specifications/dublin-core/dcmi-terms/
