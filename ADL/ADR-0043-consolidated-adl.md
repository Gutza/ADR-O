---
id: 43
type: Core vocabulary
status: Accepted
date: 2026-05-05
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0043 — The Consolidated ADL

## Context

An ADL grows through accretion. Each new `DecisionRecord` implicitly or explicitly modifies the interpretive landscape for all prior records: it may amend a naming convention, supersede an architectural approach, or deprecate a commitment that circumstances have rendered moot. Without active management, the corpus becomes internally inconsistent — a patchwork whose real operative meaning can only be reconstructed by someone who has read everything and holds the synthesis in their head.

Legislatures face the same problem when a body of law grows through accretion. The response, developed over centuries of practice, is a structural distinction between two layers: the **enacted layer** (the Statutes at Large, Hansard — the authoritative historical record of what was passed, verbatim, in sequence) and the **consolidated layer** (the United States Code, legislation.gov.uk's point-in-time view — the operative practical reference, actively maintained to reflect current in-force law). The enacted layer is never rewritten; the consolidated layer is never appended to. Each does its job cleanly because it is not asked to do the other's.

Most ADL practices collapse these two layers into one. A folder of Markdown files is both the historical record and the operative reference — and it serves neither role well. The "what is currently decided about X?" question requires reading every ADR, tracing supersession chains, and holding the synthesis mentally. The ADR with `status: Superseded` is still present and must be consciously filtered; the ADR that partially amends another leaves the original in an ambiguous half-active state. The practitioner — or the agent — must reconstruct the consolidated view on every query.

ADR-O already provides the graph machinery for the enacted layer: `adr-o:amends`, `adr-o:supersedes`, `adr-o:deprecates`, and their inverses form a traversable supersession topology. What is missing is a formal structure for the consolidated layer: a maintained operative surface that answers "what are we currently committed to?" without requiring full ADL traversal.

### Why not SPARQL alone?

A natural response is to attempt computing the consolidated view on demand via graph queries; after all, SPARQL traversal over the amendment chain can reliably identify which `DecisionRecord`s are currently active and which have been superseded. But it cannot resolve partial amendments. When ADR-007 amends ADR-003, the amendment may touch one clause of one `ExpectedOutcome` while leaving the rest of ADR-003 fully in force. No query can determine which prose survives and which is replaced without reading both records and *understanding the natural language of the change.* The gap between "ADR-007 amends ADR-003" (machine-readable) and "ADR-007 changes the filename convention but not the directory structure" (only recoverable from prose) is where SPARQL stops and language understanding begins.

### Why LLM tooling can close this gap

The [Tooling-Mediation Principle](/Archive/ADR-O%20Tooling-Mediation%20Principle.md) establishes that ADR-O graphs are substrate, mediated by a tooling layer. The reconstructability boundary rule follows: *if a representation detail can be reconstructed from the graph by reasonable tooling, keep it out of the ontology shape.*

LLM tooling is exactly the "reasonable tooling" that can reconstruct consolidated meaning from the enacted layer. Given an `amends` chain and the prose of the records involved, an LLM can reliably extract what remains operative and what has been replaced. This is not algorithmic (SPARQL can't do it), but it is tractable and (reasonably) repeatable as a bounded LLM task.

The consolidation algorithm is a **3-way diff**: the amended ADR (the old state), the amending ADR (the change instruction), and the current consolidated view (the cache to be updated). The LLM reads these three bounded inputs and produces an edited cache. Critically, this composability means *ADL size is irrelevant to consolidation cost*: each consolidation event processes only its three inputs, regardless of how large the ADL has grown. The algorithm does not get harder over time.

### What gets consolidated: `ExpectedOutcome`s only

The content of the consolidated layer is determined by answering the question "what are we currently committed to?" — not "how did we get here?" Context facts, deliberation facts, complaints, and verdicts all belong to the second question. They are provenance: the record of the reasoning path that led to the commitment. The first question is answered entirely by `ExpectedOutcome` nodes, which are t₀ claims about what the chosen alternative is intended to produce.

The consolidated layer is therefore a maintained collection of operative `ExpectedOutcome` content — the commitments extracted from all active records, edited in place as amendments modify them, with the provenance structure left entirely in the frozen enacted layer where it belongs.

## Decision

We introduce a two-layer architecture for ADL management, and the formal vocabulary to represent the consolidated layer in the graph.

### Layer 1: The Enacted Layer (existing)

The existing ADL of `DecisionRecord` nodes, each frozen at t₀ per the [Decision Transaction Principle](/Archive/Decision%20Transaction%20Principle.md), constitutes **the enacted layer which is immutable.** It preserves the full deliberative history — complaints, context facts, deliberation facts, alternatives, verdicts, expected outcomes — for every decision ever made, regardless of whether it turned out to be right, wrong, or suboptimal. The decisions' links (`amends`, `supersedes`, and `deprecates`) form the graph topology from which the consolidated layer is derived.

### Layer 2: The Consolidated Layer (introduced here)

**The consolidated layer is a single `adr-o:ConsolidatedADL` instance containing an ordered collection of `adr-o:ConsolidatedOutcome` entries.** It is explicitly and deliberately a **cache** — a disposable, regenerable derived artifact whose sole purpose is to answer "what are we currently committed to?" without requiring full ADL traversal. Its epistemic status is the opposite of the enacted layer: eternally mutable, never authoritative, always reconstructable from source.

The appropriate mental model is a **debug-build compiled binary**: the source files (the frozen ADRs) are authoritative, while the compiled output (the consolidated view) is disposable and can always be rebuilt from source, at compute cost. The binary is kept around only because recompiling is expensive, not because it has independent value. The "debug-build" qualifier is specific: every entry in the compiled output carries a back-link to the source line that produced it (mental model: [source map](https://developer.mozilla.org/en-US/docs/Glossary/Source_map) symbols). Those links are the primary structural value of the representation — they make the operative surface navigable back to the deliberative record.

### The consolidation algorithm

When a new `DecisionRecord` is accepted, a consolidation strategy is triggered depending on the new record's relationship to the ADL graph:

1. **No inter-record relations** (`amends`, `supersedes`, `deprecates` absent): the new record's `ExpectedOutcome` nodes are simply appended to the `ConsolidatedADL` as new `ConsolidatedOutcome` entries, each linked back to its source via `adr-o:consolidatedFrom`.

2. **Amending record** (`adr-o:amends` present): the LLM performs a 3-way diff — the amended ADR, the amending ADR, the current consolidated view — and surgically edits the affected `ConsolidatedOutcome` entries in place. Entries may have their prose modified; entries whose corresponding commitments are entirely replaced by the amendment may have their `consolidatedFrom` link re-pointed to the amending ADR's relevant `ExpectedOutcome`. Some new entries might be added.

3. **Superseding record** (`adr-o:supersedes` present): the superseded record's `ConsolidatedOutcome` entries are removed from the collection. Their nodes remain in the ADL graph, reachable via the enacted layer, but they are not part of the consolidated view. The superseding record's `ExpectedOutcome` nodes are added as new entries.

4. **Deprecating record** (`adr-o:deprecates` present): the deprecated record's `ConsolidatedOutcome` entries are removed. The deprecating record's own `ExpectedOutcome` nodes, if any, are added.

After successful processing, the `consolidatedUpTo` pointer on the `ConsolidatedADL` is advanced to the newly-processed record.

In all cases, the LLM's input is bounded: it reads at most the two directly related ADRs plus the current consolidated view; it never needs to re-read the full ADL.

### Staleness

The `ConsolidatedADL` carries an `adr-o:consolidatedUpTo` pointer to the most recently incorporated `DecisionRecord`. Staleness is computed by comparing this pointer to the most recent `Accepted` record in the ADL: if they differ, the consolidated view requires recompilation. This check is a pure SPARQL query over the enacted layer — no LLM involvement — and can be run automatically after every new record is accepted.

Staleness is anchored to `DecisionRecord` identity, not to wall-clock time. "Last consolidated to include ADR-0042" is more stable, more meaningful, and more consistent with the Decision Transaction Principle than "last updated on Wednesday." The ADR IRI is frozen; the date is a mutable annotation.

### Orphan detection

The inverse predicate `adr-o:consolidatedIn` on `ExpectedOutcome` enables a first-class integrity check: an `ExpectedOutcome` from an active (non-superseded, non-deprecated) `DecisionRecord` that carries no `adr-o:consolidatedIn` link is **orphaned** — its commitment is not currently reflected in the consolidated view. This is either correct (the commitment was edited away by an amendment and the old node was intentionally left unlinked) or a signal that consolidation missed something.

The orphan-detection query is:

```sparql
SELECT ?eo ?dr WHERE {
  ?dr adr-o:hasExpectedOutcome/rdf:rest*/rdf:first ?eo .
  ?dr adr-o:hasStatus adr-o:Accepted .
  FILTER NOT EXISTS { ?eo adr-o:consolidatedIn ?_ }
  FILTER NOT EXISTS { ?dr adr-o:supersededBy ?_ }
  FILTER NOT EXISTS { ?dr adr-o:deprecatedBy ?_ }
}
```

This query is cheap, structural, and requires no LLM involvement. It should be run after every consolidation event as an integrity check.

### Vocabulary introduced

#### `adr-o:ConsolidatedADL`

The class for the consolidated layer document. One instance per ADL in normal use; this is not a design flexibility hedge but a design commitment. Multiple consolidated documents for the same ADL would recreate the normative fragmentation the consolidated layer exists to prevent.

`ConsolidatedADL` is not a `DecisionRecord` and carries none of the epistemic structure of the enacted layer. It does not participate in the Decision Transaction Principle. It is not a PROV entity: the PROV alignment test established in ADR-0035 (fixed aspects, attributable provenance, traceable lifecycle) fails at the first criterion — `ConsolidatedADL` is explicitly eternally mutable.

#### `adr-o:ConsolidatedOutcome`

An entry in a `ConsolidatedADL`. Each entry carries the current operative text of an `ExpectedOutcome` — potentially edited since its source if subsequent amendments have modified the commitment. Entries are individually addressable (each has an IRI), individually replaceable, and ordered within the collection by `adr-o:consolidationOrder`.

`ConsolidatedOutcome` is not a `Claim` and carries no truth value in the deliberative sense (ADR-0031). It is not subject to verification or observation. It is a cache slot: its content is the current best rendering of a commitment, its structural value is the `consolidatedFrom` back-link, and its lifecycle is entirely managed by the consolidation tooling layer.

#### `adr-o:consolidatedFrom` / `adr-o:consolidatedIn`

The source-map predicate pair. `adr-o:consolidatedFrom` on a `ConsolidatedOutcome` points to the `ExpectedOutcome` from which the entry was originally projected. `adr-o:consolidatedIn` is its inverse: on an `ExpectedOutcome`, it points to the `ConsolidatedOutcome` that currently represents it.

`consolidatedFrom` is not declared `owl:FunctionalProperty`. In the common case it is one-to-one, but a `ConsolidatedOutcome` produced by an amendment that merges or bridges two overlapping `ExpectedOutcome`s from related records may legitimately point to both sources. Tooling should treat single-source entries as the default and multi-source entries as an exceptional but valid state.

The direction of these predicates follows the existing convention: the forward link (`consolidatedFrom`) is on the derived entity pointing to its source, consistent with `adr-o:derivedFrom` on `Claim`, `adr-o:realizedFrom` on `ObservedOutcome`, and `adr-o:justifiedBy` on resources generally.

#### `adr-o:inConsolidatedADL`

The membership predicate from `ConsolidatedOutcome` to `ConsolidatedADL`. Declared `owl:FunctionalProperty`: one entry belongs to exactly one consolidated document. This follows the `skos:inScheme` pattern used throughout the ontology for concept-to-scheme membership, with the additional constraint of functionality that SKOS deliberately omits. The functional declaration reflects the design commitment to a singleton `ConsolidatedADL`: if a `ConsolidatedOutcome` could belong to two `ConsolidatedADL` instances, the singleton would be violated.

#### `adr-o:consolidatedUpTo`

The staleness anchor on `ConsolidatedADL`. Points to the most recently incorporated `DecisionRecord`. Declared `owl:FunctionalProperty`: there is exactly one "current as of" pointer per consolidated document. Advancing this pointer is the final step of every consolidation event; its absence or staleness is the trigger for recompilation.

#### `adr-o:consolidationOrder`

An ordering index on `ConsolidatedOutcome` within its `ConsolidatedADL`. Managed entirely by the consolidation tooling layer; the ontology defines the property but imposes no ordering constraints. Follows the same pattern as `adr-o:index` on `DecisionRecord`, scoped to the consolidated layer.

## Ontology Materialisation

```turtle
###  https://w3id.org/adr-o#ConsolidatedADL
adr-o:ConsolidatedADL rdf:type owl:Class ;
    rdfs:label "Consolidated ADL"@en ;
    rdfs:comment "A disposable, LLM-maintained cache of the currently operative commitments across an ADL. Contains an ordered collection of ConsolidatedOutcome entries, each tracing to its source ExpectedOutcome via adr-o:consolidatedFrom."@en ;
    skos:scopeNote "Not a DecisionRecord and not part of the enacted layer. Eternally mutable: entries may be edited, replaced, or removed as the ADL evolves. One instance per ADL in normal use — multiple instances would recreate the normative fragmentation the consolidated layer exists to prevent. Does not subclass prov:Entity: the PROV alignment test fails at fixed-aspects; ConsolidatedADL is explicitly mutable by design."@en ;
    adr-o:justifiedBy adr-odr:ADR-0043-consolidated-adl .


###  https://w3id.org/adr-o#ConsolidatedOutcome
adr-o:ConsolidatedOutcome rdf:type owl:Class ;
    rdfs:label "Consolidated Outcome"@en ;
    rdfs:comment "An entry in a ConsolidatedADL representing the current operative text of an ExpectedOutcome. Carries a back-link to its source ExpectedOutcome via adr-o:consolidatedFrom. Mutable: may be edited in place by subsequent amendments without producing a new IRI."@en ;
    skos:scopeNote "Not a Claim: carries no deliberative truth value (ADR-0031). Not subject to verification or observation. Its structural value is the source-map link (adr-o:consolidatedFrom) connecting the operative surface to the frozen enacted layer. Analogous to a debug-build symbol table entry."@en ;
    adr-o:justifiedBy adr-odr:ADR-0043-consolidated-adl .


###  https://w3id.org/adr-o#consolidatedFrom
adr-o:consolidatedFrom rdf:type owl:ObjectProperty ;
    owl:inverseOf adr-o:consolidatedIn ;
    rdfs:domain adr-o:ConsolidatedOutcome ;
    rdfs:range adr-o:ExpectedOutcome ;
    rdfs:label "consolidated from"@en ;
    rdfs:comment "Links a ConsolidatedOutcome to the source ExpectedOutcome from which it was projected. The forward source-map link: from operative surface to frozen enacted record."@en ;
    skos:scopeNote "Not declared owl:FunctionalProperty: multi-source entries are the normal result of any in-place amendment. The original ExpectedOutcome contributes the semantic substance; the amending ExpectedOutcome contributes the modification. Both links are required for the source map to be honest — pointing to the amending record alone would falsely imply the substance originated there; pointing to the original alone would falsely imply the current text is unchanged. Single-source entries are the common case only for commitments that have never been amended."@en ;
    adr-o:justifiedBy adr-odr:ADR-0043-consolidated-adl .


###  https://w3id.org/adr-o#consolidatedIn
adr-o:consolidatedIn rdf:type owl:ObjectProperty ;
    rdfs:domain adr-o:ExpectedOutcome ;
    rdfs:range adr-o:ConsolidatedOutcome ;
    rdfs:label "consolidated in"@en ;
    rdfs:comment "Inverse of adr-o:consolidatedFrom. Links a source ExpectedOutcome to its corresponding ConsolidatedOutcome. Absence of this link on an ExpectedOutcome from an active DecisionRecord is a meaningful signal: the commitment is not currently reflected in the consolidated view."@en ;
    adr-o:justifiedBy adr-odr:ADR-0043-consolidated-adl .


###  https://w3id.org/adr-o#inConsolidatedADL
adr-o:inConsolidatedADL rdf:type owl:ObjectProperty ,
                                  owl:FunctionalProperty ;
    rdfs:domain adr-o:ConsolidatedOutcome ;
    rdfs:range adr-o:ConsolidatedADL ;
    rdfs:label "in consolidated ADL"@en ;
    rdfs:comment "Links a ConsolidatedOutcome to the ConsolidatedADL it belongs to. Functional: one entry belongs to exactly one consolidated document."@en ;
    skos:scopeNote "Follows the skos:inScheme membership pattern with the additional owl:FunctionalProperty constraint that SKOS omits. Functionality enforces the singleton design of ConsolidatedADL: a ConsolidatedOutcome cannot belong to two consolidated documents without violating the one-consolidated-view-per-ADL commitment."@en ;
    adr-o:justifiedBy adr-odr:ADR-0043-consolidated-adl .


###  https://w3id.org/adr-o#consolidatedUpTo
adr-o:consolidatedUpTo rdf:type owl:ObjectProperty ,
                                 owl:FunctionalProperty ;
    rdfs:domain adr-o:ConsolidatedADL ;
    rdfs:range adr-o:DecisionRecord ;
    rdfs:label "consolidated up to"@en ;
    rdfs:comment "Links a ConsolidatedADL to the most recently incorporated DecisionRecord. Staleness is computed by comparing this pointer to the most recent Accepted record in the ADL."@en ;
    skos:scopeNote "Anchored to DecisionRecord IRI, not to a timestamp, per the Decision Transaction Principle. 'Last consolidated to include ADR-0042' is more stable and meaningful than 'last updated on Wednesday'. Advancing this pointer is the final step of every consolidation event."@en ;
    adr-o:justifiedBy adr-odr:ADR-0043-consolidated-adl .


###  https://w3id.org/adr-o#position
adr-o:consolidationOrder rdf:type owl:DatatypeProperty ,
                        owl:FunctionalProperty ;
    rdfs:domain adr-o:ConsolidatedOutcome ;
    rdfs:range xsd:integer ;
    rdfs:label "consolidation order"@en ;
    rdfs:comment "Ordering index of a ConsolidatedOutcome within its ConsolidatedADL. Managed by the consolidation tooling layer; the ontology defines the property and range but imposes no ordering constraints or contiguity requirements."@en ;
    adr-o:justifiedBy adr-odr:ADR-0043-consolidated-adl .
```

### Instantiation example

A minimal `ConsolidatedADL` after two accepted records, where ADR-002 amends one commitment from ADR-001:

```turtle
# The consolidated document
:consolidatedADL-project-x rdf:type adr-o:ConsolidatedADL ;
    dcterms:title "Project X — Consolidated ADL"@en ;
    adr-o:consolidatedUpTo :ADR-002 .

# Entry originating from ADR-001 (unchanged by ADR-002)
:co-001-1 rdf:type adr-o:ConsolidatedOutcome ;
    adr-o:inConsolidatedADL :consolidatedADL-project-x ;
    adr-o:consolidatedFrom :adr-001-eo-1 ;
    adr-o:consolidationOrder 1 ;
    dcterms:description "Event handlers will be co-located with the domain objects they serve."^^adr-o:md .

# Entry originating from ADR-001 but edited in place by ADR-002's amendment
:co-001-2 rdf:type adr-o:ConsolidatedOutcome ;
    adr-o:inConsolidatedADL :consolidatedADL-project-x ;
    adr-o:consolidatedFrom :adr-001-eo-2 ;     # original source retained
    adr-o:consolidationOrder 2 ;
    dcterms:description "Module files will be named in kebab-case."^^adr-o:md .
    # Note: original text was "snake_case"; ADR-002 amended this in place.
    # ADR-001's :adr-001-eo-2 is NOT orphaned — this entry still traces to it.

# Entry originating from ADR-002 (a new commitment, not an amendment of anything)
:co-002-1 rdf:type adr-o:ConsolidatedOutcome ;
    adr-o:inConsolidatedADL :consolidatedADL-project-x ;
    adr-o:consolidatedFrom :adr-002-eo-1 ;
    adr-o:consolidationOrder 3 ;
    dcterms:description "All public API endpoints will be versioned with a /v{n}/ path prefix."^^adr-o:md .
```

In the enacted layer, `:adr-001-eo-2` carries `adr-o:consolidatedIn :co-001-2`; `:adr-002-eo-1` carries `adr-o:consolidatedIn :co-002-1`. An orphan-detection query finds nothing: all `ExpectedOutcome`s from active records are represented.

## Alternatives considered

| Alternative | Reason not chosen |
|---|---|
| No consolidated layer; answer "what is decided?" via live LLM query over the full ADL | Eliminates the O(n²) cost problem only by paying it on every query rather than amortising it. For an ADL of 43 records at time of writing, already non-trivial; at 200 records, prohibitive. Also provides no stable IRI for staleness metadata and no source-map links for navigation. |
| Consolidate full ADR summaries rather than `ExpectedOutcome`s only | Conflates the operative surface with provenance. Context facts and deliberation facts answer "how did we get here?" — a question the enacted layer already answers. Including them in the consolidated view doubles the content without adding operative value and obscures the commitments behind the reasoning that produced them. |
| Auto-regenerate the consolidated view from scratch on every consolidation event | Correct in principle — the consolidated view IS always regenerable from source. But full-ADL regeneration pays the O(n²) cost at every event. The 3-way diff is strictly cheaper and sufficient: the current consolidated view already embeds all prior consolidation work. |
| Anchor staleness to `dcterms:modified` timestamp rather than `DecisionRecord` IRI | Timestamps are mutable annotations on the graph. "Last updated on Wednesday" tells tooling when, not what: it cannot determine whether an ADR accepted on Thursday has been incorporated without comparing dates to ADR indices. `consolidatedUpTo` pointing to a frozen IRI is unambiguous. |
| Allow multiple `ConsolidatedADL` instances per ADL, e.g. one per topic area | Recreates the normative fragmentation the consolidated layer exists to solve: "what is currently decided about auth?" would require consulting the auth-topic consolidated view, which may not cover cross-cutting decisions. A single operative surface is the point. Topic-based filtering is a query concern, not a document-structure concern. |
| Represent the entry collection as `rdf:List` (parallel to `hasContext`, `hasDeliberation`, `hasExpectedOutcome`) | `rdf:List` is immutable in RDF: modifying one member requires rewriting the entire list. The "eternally mutable cache" nature of `ConsolidatedADL` requires individually addressable, individually replaceable entries. The `adr-o:consolidationOrder` + `adr-o:inConsolidatedADL` pattern gives each entry an independent IRI and an independent update path. |
| Subclass `prov:Entity` for `ConsolidatedADL` or `ConsolidatedOutcome` | The PROV alignment test established in ADR-0035 requires fixed aspects (content immutable after commitment), attributable provenance, and traceable lifecycle. `ConsolidatedADL` and `ConsolidatedOutcome` fail the first criterion by design: they are explicitly mutable. Applying PROV subclassing would misrepresent their nature and create misleading interoperability claims. |
| Use `rdf:Seq` (container) for the ordered collection instead of `adr-o:consolidationOrder` | `rdf:Seq` membership via `rdf:_n` predicates does not support surgical replacement of individual members without renumbering. `adr-o:consolidationOrder` as a data property on each `ConsolidatedOutcome` allows any entry to be updated, reordered, or removed independently. Gaps and non-contiguous integers are acceptable; only relative order matters. |
| Name the cache document `ConsolidatedView` or `ArchitecturePosition` | `ConsolidatedView` implies a query-layer construct (database terminology for a named query). `ArchitecturePosition` suggests a substantive editorial document, importing an authority the cache does not have. `ConsolidatedADL` names the thing directly — a consolidated version of the ADL — and preserves the legislative framing from which the two-layer architecture derives. |
| Declare `adr-o:consolidatedFrom` as `owl:FunctionalProperty` | Multi-source entries are the normal result of any in-place amendment: the original `ExpectedOutcome` contributes the semantic substance and the amending `ExpectedOutcome` contributes the modification, and both links are required for the source map to be honest. Declaring functional would make every amended entry an OWL inconsistency. |

## Consequences

**Positive.**

- The ADL now has a formal operative surface. "What are we currently committed to?" is answered by reading the `ConsolidatedADL`, not by reading and mentally synthesising every active record.
- Consolidation cost is bounded and composition-independent. Each consolidation event processes three bounded inputs regardless of ADL size; the O(n²) cost of full-ADL regeneration is paid only on first build or after catastrophic cache loss.
- Orphan detection is a free structural query. An `ExpectedOutcome` from an active record with no `consolidatedIn` link is automatically identifiable as a potential consolidation gap, without LLM involvement.
- Bidirectional navigation between layers is structural, not heuristic. From any `ConsolidatedOutcome`, an agent reaches the originating `ExpectedOutcome` and from there the full `DecisionRecord` — complaint, context, deliberation, verdict — in two hops. From any `ExpectedOutcome`, an agent confirms whether its commitment is currently operative or orphaned in one hop.
- Staleness detection is a pure graph query. Comparing `consolidatedUpTo` to the most recent `Accepted` record requires no LLM involvement and can run automatically after every record acceptance.
- The two-layer separation is formally honoured. The enacted layer remains immutable, append-only, and authoritative. The consolidated layer is explicitly mutable, cache-like, and derived. Neither is asked to do the other's job.

**Negative / risks.**

- Cache drift is possible if consolidation is not run after every record acceptance. A stale `ConsolidatedADL` is worse than no consolidated layer at all: it gives a false sense of currency. The staleness check mitigates this; discipline enforces it.
- `ConsolidatedOutcome` entries are mutable by design, which creates a surface for inadvertent semantic drift if tooling allows direct editing without triggering the consolidation algorithm. Authoring conventions and tooling guardrails should restrict direct edits to LLM-mediated consolidation events only.
- LLM consolidation is non-deterministic: two consolidation runs over the same inputs may produce prose that differs in wording while agreeing in meaning. This is acceptable — the consolidated view is a cache, not an authoritative record — but practitioners must not treat `ConsolidatedOutcome` prose as canonical formulations.
- The `consolidationOrder` integer requires maintenance: insertions and deletions may leave gaps or require renumbering. Tooling is responsible for managing this; gaps are structurally harmless but may confuse ordering queries that assume contiguity.
- The singleton `ConsolidatedADL` design does not accommodate multi-team ADLs where different groups maintain separate operative views of the same record corpus. This is an accepted constraint: the problem of multiple perspectives on the same ADL is deferred to a future record if it arises.

## References

- [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) — The Decision Transaction Principle; establishes `ExpectedOutcome` as a t₀ commitment and the enacted layer's immutability regime. The consolidated layer is designed to be its epistemic inverse.
- [ADR-0031](/ADL/ADR-0031-rename-consideration-claim.md) — Establishes `Claim` as the verifiable proposition type; `ConsolidatedOutcome` is explicitly not a `Claim`.
- [ADR-0035](/ADL/ADR-0035-observedoutcome-prov-entity.md) — The three-part PROV-O alignment test (fixed aspects, attributable provenance, traceable lifecycle); applied here to justify *not* subclassing `prov:Entity` for `ConsolidatedADL` or `ConsolidatedOutcome`.
- [ADR-0041](/ADL/ADR-0041-deprecation-requires-a-deprecating-record.md) — Deprecation semantics; governs how deprecated records' `ConsolidatedOutcome` entries are handled during consolidation.
- [ADR-0042](/ADL/ADR-0042-all-documents-are-deliverables.md) — Establishes the Archive/ADR duality: active ADRs may gain practitioner-facing companion articles in `Archive/`; deprecated ADRs are lifted there as their operative successors. ADR-0042 deprecated ADR-0004 under this convention, making the Archive article below its operative successor. ADR-0028's companion article below operates under the same convention but in the companion role, not the successor role, since ADR-0028 remains active.
- [*Decision Transaction Principle*](/Archive/Decision%20Transaction%20Principle.md) — Practitioner-facing companion to ADR-0028 (maintained under ADR-0042); the t₀/tₙ boundary that defines what the enacted layer preserves and what the consolidated layer is permitted to discard.
- [*ADR-O Tooling-Mediation Principle*](/Archive/ADR-O%20Tooling-Mediation%20Principle.md) — Operative successor to ADR-0004 (deprecated by ADR-0042); lifted and maintained under ADR-0042. The reconstructability boundary rule that licenses LLM tooling as the consolidation engine and governs which representation details belong in the graph vs. the tooling layer.
