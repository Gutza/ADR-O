# ADR-O Backlog

Working notes on open questions, candidate features, and deferred decisions. Items graduate to ADRs when they're ready; until then they live here – and then they are aggressively pruned from this document.

## Open Items

### SHACL shapes companion
**Triage: Critical,** required to move towards production.

The single most load-bearing deferred item, as it blocks reliable tooling.

Examples of integrity rules currently living in `skos:scopeNote` and authoring conventions (partial list, needs rigorous mapping):

- `chosenAlternative ∈ hasAlternative`
- `Accepted` status requires `chosenAlternative` present
- `closedBy` present on `Accepted`/`Superseded` records
- `promptedBy` present on all records (exactly one `Complaint`)
- `DeliberationFact` requires `onAlternative`; `ContextFact` forbids it
- Valence-enum membership per fact class
- `rdf:List` element typing
- `dcterms:description` typed `^^adr-o:md` on prose-carrying nodes

### `DecisionTemplate` / GADR pattern
**Triage: Low Priority** – nice idea, future music; it might tie in to other opportunities and wider scopes.

GADR (Kopp & Armbruster, ZEUS 2019) argues there is a meaningful second record type: a cross-project archetype carrying context and an option space but no `chosenAlternative`, no `ExpectedOutcome`s, no RACI predicates, no date. Modeling this as `Proposed` doesn't work — a `Proposed` record is project-specific and expected to resolve; a template is deliberately left open.

Candidate shape:
- `adr-o:DecisionTemplate` — new class
- `adr-o:instantiates` — `DecisionRecord` → `DecisionTemplate`, providing both discovery and provenance

Practical query this enables: "which of our decisions were derived from shared organizational or community patterns?" Currently unanswerable by traversal.

The JabRef/Eclipse Winery example is the concrete motivating case: two projects, same Java build-tool template, opposite outcomes (Gradle vs Maven) because they weighted the same shared deliberation differently.

### Temporal governance / revisit conditions
**Triage: Reactive Stance** – interesting idea, but I'd rather see a concrete need/request for it rather than solve it for academic completeness.

No mechanism for decay. A record stays `Accepted` indefinitely even after its founding assumptions have expired — a Zombie Decision. This risk scales with `adr-o:significance`-less "log all" density (ADR-0005).

Candidate predicates:
- `adr-o:revisitCondition` — literal or IRI describing the trigger for re-evaluation
- `adr-o:revisitDate` — `xsd:date` fallback for time-based triggers

Low urgency until the ADL gets dense enough that zombie records are an actual navigation problem.

### Conformance checks / fitness functions
**Triage: Reactive Stance** – interesting idea, but I'd rather see a concrete need/request for it rather than solve it for academic completeness.

MADR is the only ADR format that explicitly asks "how will you know this decision is being followed?" ADR-O has no `conformanceCheck` predicate pointing to a CI rule, test suite, SHACL shape, or policy document. For agent-navigable compliance queries this is a real gap — but the shape is underspecified. Deferred until a concrete use case drives the design.
