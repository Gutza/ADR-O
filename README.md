# ADR-O

ADR-O is a work-in-progress RDF/OWL ontology for Architecture Decision Records (ADRs).

The goal is simple: represent architectural decisions as a machine-traversable, linked graph rather than isolated markdown files, so both humans and tools can understand not just what was decided, but why.

## Status

Early days. The project is in inception/discovery mode and actively evolving.

Expect:
- rapid iteration on vocabulary and modeling choices;
- incomplete documentation and examples;
- occasional breaking changes as core concepts are clarified.

## Why ADR-O

Traditional ADRs are useful for people but hard for machines to query consistently. ADR-O aims to make decision history explicit and durable by modeling:
- a decision and its context/rationale;
- alternatives (including rejected options and rejection reasons);
- typed relationships between decisions (for example, `supersedes`, `dependsOn`, `enables`, `conflictsWith`);
- links to concerns addressed by a decision.

This makes decision logs navigable as a graph, supporting reliable traversal and richer analysis than keyword search over prose.

## Design Direction (current)

The current direction is to build a small, focused ontology that:
- reuses established vocabularies where possible (notably Dublin Core Terms, PROV-O, and SKOS);
- defines only ADR-specific concepts and predicates that are missing from existing standards;
- aligns with ISO 42010 / OntolAD concepts without taking on hard dependencies that reduce portability.

## Project Orientation

Start here:
- `MANIFESTO.md` — project motivation and north star.
- `ADL/ADR-0000-inception.md` — inception record and rationale for the greenfield approach.

## Intended Outcomes

ADR-O is intended to enable:
- machine-interpretable decision history across system evolution;
- consistent querying of architectural rationale and trade-offs;
- better support for AI-assisted architecture and implementation workflows.

## Namespace and License

- Namespace: `https://w3id.org/adr-o#`
- License: Creative Commons Attribution 4.0 International (CC BY 4.0)

## Contributing (for now)

Contributions and critique are welcome, especially around:
- core class/property naming;
- relationship semantics and constraints;
- interoperability with existing linked-data and architecture-description tooling.

As this is pre-1.0 ontology work, please open discussions/issues with proposed use cases and examples whenever possible.
