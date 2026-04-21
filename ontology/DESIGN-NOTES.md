# ADR-O — Design Notes

<div data-dn-section="document-intro">
<div data-dn-passage="document-purpose-and-reading-order" data-dn-record="meta">

This is a sidecar document to [`adr-o.ttl`](adr-o.ttl) acting as a self-contained ADL proxy for fast-paced changes during the first few design iterations. It records the decisions that went into each iteration of the ontology so that they can be triaged — strengthened, weakened, refuted, or promoted to a proper ADR — later. The notes span several topics and mix strong and weak claims, which is exactly why they don't belong in the ADL in this form.

The document is structured as a sequence of `## ADR-O <version>` sections, one per ontology iteration. Each section is an immutable historical record of the decisions that produced that version: later sections may explicitly reverse earlier decisions, but they never edit them in place. To know what is currently true, read the latest section; to know why, read backward.

Within each version section, entries are organized by the strength of the claim, so triage is easy: strong claims go straight into ADRs with minimal rework; softer defaults deserve deliberation before being promoted; explicit deferrals are reminders that a decision was considered and punted, not forgotten.

Every reference to an ADR points to the corresponding file in `ADL/`.

Where a passage has been **lifted** into the ADL as the durable record, an inline marker follows it, for example *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*. Markers are additive; the prose above them stays the historical source of truth for that iteration.

</div>
</div>

<div data-dn-section="machine-readable-markup">
<div data-dn-passage="machine-readable-contract" data-dn-record="meta">

## Machine-readable markup

This document also uses HTML `<div>` containers with `data-dn-*` attributes for programmatic scanning. `data-dn-version` identifies a version section, `data-dn-section` identifies subsection semantics, and `data-dn-passage` marks atomic passages. `data-dn-record` classifies passage status (`adl`, `deferral`, `superseded`, `meta`, `repo-only`, `partial-rationale`), while `data-dn-adl-refs` and `data-dn-roadmap` carry optional ADR and roadmap anchors. Inline prose markers remain authoritative for humans; attributes provide a parallel machine contract.

</div>
</div>

<div data-dn-version="0.1.0-draft" data-dn-record="meta" id="adro-0-1-0-draft">

## ADR-O 0.1.0-draft

<div data-dn-section="strong">
<div data-dn-passage="strong-decisions-010" data-dn-record="meta">

### Strong, well-articulated decisions

These are directly derivable from an existing ADR, from the survey in ADR-0000, or from the question-class analysis in the outreach article. They should be non-controversial and ready to promote into ADRs with minimal rework.

**Primary class name is `adr-o:DecisionRecord`, not `ArchitectureDecisionRecord`.** Domain-agnostic scope is an accepted decision (ADR-0002). Encoding "architecture" at the IRI level would make non-software uses structurally second-class; the significance filter that "architecture" carries is better captured explicitly in a future iteration than implicitly through class naming. *(→ [ADR-0002](/ADL/ADR-0002-scope.md))*

**Six core relational predicates: `supersedes`, `supersededBy`, `dependsOn`, `enables`, `conflictsWith`, `addresses`.** This vocabulary was identified in ADR-0000 as the fresh terms to mint, and every question class in the outreach article's taxonomy reduces to a traversal over one or more of them. The outreach article's `seeAlso` was not minted: `rdfs:seeAlso` already exists and is sufficient. *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*

**Splitting OntolAD's `affects` into `adr-o:addresses` and `adr-o:affects`.** ADR-0000 identified that OntolAD conflated "what problem does this decision resolve" with "what does this decision touch in the system" under a single `affects` property, and called the split out as a correction. This iteration applies the split: `addresses` is the problem side (range `Concern`); `affects` is the manifestation side (open range). The question-class analysis confirmed this is the predicate that answers both targeted questions and filter-by-system queries. *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*

**`adr-o:index` as an `xsd:integer` separate from `dcterms:identifier`.** The human-readable string ID (`ADR-0042`) is carried by `dcterms:identifier` but does not sort cleanly in SPARQL. Organizational-history queries need a native numeric ordering, which only a dedicated integer property delivers. This was not an obvious need from the ADRs alone — it surfaced during the question-class review. *(→ [ADR-0007](/ADL/ADR-0007-index-vs-identifier.md))*

**Status concept scheme with `Proposed`, `Accepted`, `Deprecated`, `Superseded`, `Rejected`, wired into `DecisionRecord` via a named `adr-o:Status` class and a functional `hasStatus`.** ADR-0000 called out these five values and the SKOS mechanism for expressing them. The top-concept declarations are editorial but consistent with standard SKOS practice. The wiring — `adr-o:Status` defined as `skos:Concept ∩ (skos:inScheme = adr-o:statusScheme)` and used as the range of `adr-o:hasStatus` — is what makes "status values are members of the scheme" a machine-readable fact rather than a prose convention. The equivalent-class definition means domain profiles that extend the scheme with additional statuses automatically inherit `adr-o:Status` membership without any ontology change; no separate subclassing is needed. `adr-o:hasStatus` is declared `owl:FunctionalProperty` (at most one status per record), and the core ontology contains a single `owl:AllDifferent` block over the five status individuals. The `AllDifferent` block is load-bearing: without it, under OWA + no UNA, a record misannotated with two statuses would not be flagged — the reasoner would silently entail `Proposed owl:sameAs Accepted` rather than reporting an inconsistency, quietly polluting every downstream query. With the `AllDifferent`, the same misannotation becomes an OWL-level contradiction that a conformant OWL 2 DL reasoner (HermiT, Pellet, FaCT++) reports loudly. Domain profiles that add new status individuals should extend the `AllDifferent` declaration to preserve this property. *(→ [ADR-0008](/ADL/ADR-0008-status-skos-and-integrity.md))*

A reasoner caveat worth recording: some lightweight OWL 2 RL implementations — including Python's `owlrl` — do not implement the RL rules (`eq-diff2`, `eq-diff3`) that expand an `owl:AllDifferent` block into pairwise `owl:differentFrom` entailments. Against such a reasoner, the duplicate-status check degrades silently: `Proposed sameAs Accepted` is entailed, but no contradiction is reported. This is an incompleteness in those reasoners, not in the ontology; pre-generating pairwise `owl:differentFrom` assertions as a workaround was explicitly rejected on the grounds that ADR-O should publish spec-correct OWL 2 and let reasoners be as complete as they claim to be. *(→ [ADR-0008](/ADL/ADR-0008-status-skos-and-integrity.md))*

**`Concern` as an abstract anchor class with no core concept scheme.** The extension mechanism is explicitly established in ADR-0002; this iteration just applies it. *(→ [ADR-0002](/ADL/ADR-0002-scope.md))*

**License: CC BY 4.0.** Directly per ADR-0001. The `dcterms:license` annotation on the ontology IRI is the machine-readable canonical declaration; the repository `LICENSE` file is supplementary. *(→ [ADR-0001](/ADL/ADR-0001-license.md))*

**Namespace: `https://w3id.org/adr-o#`.** Directly per README and ADR-0000. *(→ [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md); inception context in [ADR-0000](/ADL/ADR-0000-inception.md))*

**`adr-o:supersedes` as a sub-property of `prov:wasRevisionOf`.** ADR-0000 identified this alignment explicitly: the ADR supersession chain *is* a PROV-O revision chain, and modelling it as a sub-property means PROV-O-aware tooling automatically sees every supersession as a revision without any extra wiring. *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*

</div>
</div>

<div data-dn-section="softer">
<div data-dn-passage="softer-defaults-010" data-dn-record="meta">

### Softer defaults — "simplicity first, easier to add than to refactor"

These are picks made under the heuristic the user reaffirmed when asking for the first iteration. Each is reversible in a follow-up ADR; none of them forecloses a richer future.

**Single Turtle file under `ontology/`, no `examples/` or `shapes/` directories yet.** The ontology is small enough to fit in one file and one serialization; multiple files would introduce premature structure. JSON-LD and RDF/XML serializations can be generated from the canonical Turtle later. *(Repository convention only — not separately normative in the ADL; ontology packaging terms are in [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md).)*

**No `owl:imports` for PROV-O / SKOS / DC Terms.** The ontology uses these vocabularies purely as namespace prefixes. Pulling in PROV-O's full axiom set would burden every downstream reasoner for little gain at this stage; downstream tools that want the full axioms can opt in. This choice can be revisited if reasoning over the alignment (e.g. `supersedes ⊑ wasRevisionOf`) becomes important in practice. *(→ [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md))*

**Hash IRIs (`https://w3id.org/adr-o#Foo`) rather than slash IRIs.** Standard for small self-contained vocabularies and what the README already committed to. Slash IRIs would allow per-term content negotiation but require more hosting infrastructure to be useful. *(→ [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md))*

**Nygard body sections (`context`, `decision`, `consequences`, `rationale`) as `rdf:langString` datatype properties, not first-class classes.** Without them, instances cannot represent even a textbook ADR. Promoting any of them to a first-class class (OntolAD-style `Rationale`, for example) is a non-breaking extension: the existing datatype predicates can coexist with object-property forms. The trade-off is that trade-off narratives, forces, and consequence items cannot currently be individually queried — only read as prose. *(Superseded in 0.2.0; removal and replacement recorded in [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md).)*

**`consequences` as a single property, not split positive / negative.** MADR splits this; Nygard does not; our own ADRs use a single "Consequences" section with sub-headings. A later iteration may introduce `adr-o:positiveConsequences` / `adr-o:negativeConsequences` as refinements without breaking the current property. *(0.1.0 sketch only; outcome shape in core is [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md) / [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md).)*

**`conflictsWith` as an `owl:SymmetricProperty`, not a directed Kruchten-style pair.** ADR-0000 flagged this as an open question. Symmetric is strictly simpler and matches the common "A and B are in tension" use case. Directed `prevents` / `isPreventedBy` can be added later as sub-properties without breaking existing `conflictsWith` triples. *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*

**`wasRejectedBecause` as a datatype property (`rdf:langString`), not an object property.** Keeps first-iteration authoring simple — a string reason on an `Alternative`. A later revision may introduce a parallel object property pointing to a structured `Rationale` node without breaking this one. *(Superseded in 0.2.0; see [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md).)*

**`adr-o:affects` with open range (`rdfs:Resource`).** Deliberately unconstrained in the core. Domain profiles are expected to tighten this — either by declaring the range of a sub-property or through SHACL shapes — once a domain's notion of "affected element" is settled. The trade-off is that an ADR-O instance placed into a core-only graph is not protected against nonsensical `affects` targets. *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*

**`dcterms:date` as the canonical date predicate, over `dcterms:issued` and `prov:generatedAtTime`.** All three are semantically close enough for the ADR use case; `dcterms:date` is the most widely recognised and the least semantically freighted. The scope note in the ontology calls this out so domain profiles don't pick a different predicate inconsistently. *(→ [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md))*

**`dcterms:creator` as the canonical authorship predicate.** Explicitly confirmed via clarifying question. Accepts either a literal string or an IRI pointing to a structured agent. The IRI form is preferred for graph-traversable authorship queries but not mandated; `prov:wasAttributedTo` is permitted when PROV-O tooling is already in play but is not the canonical form. *(→ [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md))*

**Version annotations: `owl:versionIRI <https://w3id.org/adr-o/0.1.0>` and `owl:versionInfo "0.1.0-draft"`.** A conservative first-public-draft versioning. The `-draft` suffix is a signal that breaking changes are expected before 0.1.0 is tagged as released. *(→ [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md))*

</div>
</div>

<div data-dn-section="deferrals">
<div data-dn-passage="explicit-deferrals-010" data-dn-record="deferral">

### Explicit deferrals — decisions to not decide right now

These are items that were considered during this iteration and deliberately punted. Listing them here is the antidote to silent forgetting: whoever revisits knows it was seen.

**`adr-o:hasType` / the "Type" field our own ADRs use.** Deferred by explicit user decision after the question-class review. Our own ADRs carry a free-form `Type` field ("Inception record", "Core design", "Project governance"); Tyree–Akerman calls this "Group". Nygard-style templates do not define an equivalent field. MADR does address grouping in [Support Categories](https://adr.github.io/madr/decisions/0010-support-categories.html), but the chosen pattern is repository organisation (e.g. one level of subfolders with local IDs), not a dedicated `Type`/`Group` metadata line in the ADR itself comparable to Tyree–Akerman or our front matter. The SKOS-extension pattern would make it trivial to add (`adr-o:hasType → skos:Concept`, no core scheme, same shape as `addresses`/`Concern`), but no axioms justifying a core predicate have been identified. This is an explicit decision to not decide — the meta-decision being that domain-specific categorisation doesn't yet belong in the core namespace.

**First-class `Force` / `Constraint` / `Rationale` / `Consequence` classes.** ADR-0000 flagged `Force` as an open question; Tyree–Akerman has "Constraints" and "Assumptions"; the outreach article's "system constraints" and "unused capacity" question classes would be answered naturally by first-class constraint nodes. None of these are modelled in this iteration; they all become datatype prose for now. The trade-off is that the two constraint-shaped question classes in the outreach article are not yet fully answerable — they require text retrieval against prose rather than graph traversal. Addressing this is the natural scope of a follow-up ADR.

**Kruchten-style directed `prevents` / `isPreventedBy` alongside symmetric `conflictsWith`.** Flagged as an open question in ADR-0000. Added later as sub-properties of `conflictsWith`, non-breaking.

**MADR-style decider / consulted / informed roles.** MADR distinguishes these; ADR-O does not in this iteration. `dcterms:creator` is the only authorship predicate. If a team needs the distinction, a domain profile can mint the role properties. *(Lifted by [ADR-0021](/ADL/ADR-0021-social-role-predicates.md).)*

**`significance` / the "architectural" filter from ADR-0002.** Open question in ADR-0002. Not addressed here; the domain-agnostic core does not currently constrain what counts as record-worthy.

**SHACL shapes graph for validation.** Open question in ADR-0000. Not included. Implementers wanting cardinality or range constraints must encode them themselves for now.

**Additional serializations (JSON-LD, RDF/XML).** Turtle is canonical; others will be generated mechanically from it later.

**Domain profile registry.** Open question in ADR-0002. No registry, no central discovery mechanism, no informal SKOS collection in the core repo. Profiles are expected to publish in their own namespaces and be discoverable via whatever means the linked-data ecosystem already provides.

</div>
</div>

<div data-dn-section="notes-on-reuse">
<div data-dn-passage="reuse-notes-010" data-dn-record="meta">

### Notes on reuse that aren't quite decisions

**Scope notes on reused predicates.** The ontology file carries `skos:scopeNote` annotations for `dcterms:title`, `dcterms:identifier`, `dcterms:date`, `dcterms:created`, `dcterms:modified`, `dcterms:creator`, `dcterms:references`, `dcterms:license`, `prov:wasAttributedTo`, and `prov:wasRevisionOf`, stating how ADR-O recommends each be used. This is editorial: no new terms are minted, no semantics are changed. It exists so implementers have one place to look rather than having to infer the intended conventions from examples. *(→ [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md))*

**Versioning discipline.** No ADR has yet discussed the ontology's versioning strategy (semantic versioning? release cadence? deprecation policy?). This iteration uses `0.1.0-draft` informally. A dedicated ADR on versioning would be welcome before anyone depends on ADR-O for production tooling. *(Still open — no ADL record yet.)*

</div>
</div>
</div>

<div data-dn-version="0.2.0-draft" data-dn-record="meta" id="adro-0-2-0-draft">

## ADR-O 0.2.0-draft

<div data-dn-section="iteration-character">
<div data-dn-passage="iteration-character-020" data-dn-record="meta">

This iteration is a destructive shape change: Nygard prose-on-record is replaced by an atom-first graph. There is no backward-compatibility shim. The ontology is at this stage a technical preview rather than a candidate for production use; sledgehammers from outside reviewers are explicitly invited.

### Architecture in one paragraph

A `DecisionRecord` carries an option pool (`hasAlternative`), an explicit chosen alternative (`chosenAlternative`), and three ordered `rdf:List`s of reified facts (`hasContext`, `hasDeliberation`, `hasOutcome`). Each fact (`ContextFact`, `DeliberationFact`, `OutcomeFact`) places exactly one reusable `Consideration` into a role; deliberation and outcome facts also carry a valence drawn from a small per-class enum. The same `Consideration` IRI may be placed in arbitrarily many facts across arbitrarily many records — that reuse, the YADR-style argument-identity property, is the central design property of this iteration and is achieved without RDF-star and without quads. *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md), [ADR-0012](/ADL/ADR-0012-alternatives-and-chosen-option.md), [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md))*

### Design assumption: the KG lives under tooling

ADR-O graphs are authoritative substrate, never meant to be user surface (even when the user is a highly competent software developer). The expected deployment includes several layers between RDF and human authors / readers: LLMs that draft, summarise, and answer natural-language questions; APIs that mediate programmatic access with shape enforcement; MCP servers that expose structured tools to AI agents; GUIs for authoring, visualisation, and navigation. The ontology is designed for that layered world, not for a hypothetical use case in which someone hand-writes Turtle and reads the result without further mediation. Naming this assumption explicitly matters because several 0.2.0-draft decisions follow from it rather than from RDF / OWL principles alone, and future iterations should be able to rely on the same stance for design intentionality consistency. *(→ [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md))*

Decisions in this iteration that lean on the assumption:

- **Strict graph (no Nygard literals).** We expect the tooling to materialise Markdown for human readers from the structured graph; the KG does not need to carry (and maintain) redundant human-facing prose copies. *(→ [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md); stance [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md))*
- **SHACL deferred at technical-preview status.** Integrity rules (list-element typing, `chosenAlternative ∈ hasAlternative`, `Accepted ⇒ chosenAlternative present`, per-fact-class field cardinalities, valence-enum membership per fact class) can live in the API / MCP / GUI layer in the meantime, if needed at all. The OWL ontology stays clean of conditional cardinality rules; the planned SHACL companion will publish the same constraints declaratively when it ships. *(→ [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md))*
- **`rdf:List` ordering accepted despite OWL DL awkwardness.** List traversal is a tooling concern; raw OWL DL reasoning over list membership is not the primary access path. SPARQL handles lists cleanly, and the planned SHACL companion will enforce list-element typing. *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md); [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md))*
- **Authoring conventions documented in scope notes rather than axiomatised** (e.g. `onAlternative` required on `DeliberationFact`, optional on `OutcomeFact`, not used on `ContextFact`). The API / MCP / GUI layer can enforce these conventions long before SHACL ships. Even in LLM-heavy authoring contexts, structured tool boundaries make discipline tractable. *(→ [ADR-0012](/ADL/ADR-0012-alternatives-and-chosen-option.md); [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md))*
- **IRI convention for `Consideration` reuse documented but not formalised.** Tooling treats both record-local and org-shared patterns uniformly; the ontology imposes no axiom. *(→ [ADR-0018](/ADL/ADR-0018-consideration-iri-reuse-convention.md))*
- **`addresses` on both `DecisionRecord` and `Consideration`, with optional editorial rollup at the record level.** A rollup whose authoritative source is consideration-level only makes sense in a world where some layer (tooling, query rewriter, materialised view) reconciles the two when needed. *(→ [ADR-0013](/ADL/ADR-0013-addresses-dual-placement.md))*

Forward-looking implication: when choosing between two ontology shapes, the one that pushes incidental complexity into the tooling layer in exchange for a cleaner KG is the preferred trade-off — *unless* the complexity in question is something the tooling layer cannot reasonably reconstruct from the graph alone. That cut-off (what the tooling can reconstruct vs what must be in the graph) is the reconstructability boundary rule named in [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md), and future ADRs should make explicit which side of that boundary they are choosing when the question recurs.

Caveat: this assumption fails for users who consume ADR-O graphs raw, with no tooling at all. They will see a more skeletal model than the layered presentation suggests — text payloads on `Consideration` nodes rather than Nygard-shaped prose, integrity rules absent rather than enforced, ordering carried in `rdf:List`s that some lightweight readers handle awkwardly, the canonical/editorial split on `addresses` not reconciled. That is the deliberate trade-off, not an oversight.

</div>
</div>

<div data-dn-section="strong">
<div data-dn-passage="strong-decisions-020" data-dn-record="meta">

### Strong, well-articulated decisions

These are the load-bearing architectural commitments of 0.2.0-draft. Each was reached after a structured comparison against the alternatives in IDEAS and the principles in ADR-0000.

**Atom-level reification, not section-level (no Nygard divs).** The two reasonable shapes for first-class structure in an ADR graph are: section-level (treat each Nygard body slot as its own resource) and atom-level (introduce reusable claim atoms placed into roles). Section-level reification was rejected as "reinventing HTML divs" — it provides containers without identity, and identity is what the YADR anchor/alias mechanism shows is the genuinely valuable structural property. Atom-level reification, by contrast, captures the claim that the argument made for Option A during evaluation is *the same thing* as the positive consequence noted afterward, not a copy of it. A first-class atom class is the only way to encode that identity in RDF without resorting to text similarity. *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md))*

**Atom class named `Consideration`.** Rejected names included `Force` (overloaded with Nygard-style "forces" prose and physically connoted), `Argument` (implies contestation, fits poorly when the atom is genuinely neutral), and `Claim` (less idiomatic in the ADR literature). `Consideration` reads well across positive, negative, and neutral cases and matches how reviewers naturally describe the third bucket — "a consideration we noted but that didn't push the choice." *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md))*

**Strict graph: no Nygard body literals.** The four datatype properties `context`, `decision`, `consequences`, and `rationale` are removed. Tooling materialises Markdown for human readers from the graph; the ontology authoritatively represents structure, not prose. This forces the question "what was actually said?" to resolve to a `Consideration` IRI rather than a search inside a literal, which is what makes the YADR identity property usable. *(→ [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md))*

**Reified facts via classes, not RDF-star.** Binary edges between `DecisionRecord` and `Consideration` are insufficient: the same atom can occupy multiple roles in one record (framing vs deliberation vs outcome) and can carry orthogonal annotations (valence, ordering, eventually provenance). Two implementations achieve this: per-statement annotation (RDF-star or named graphs / quads) or reification via dedicated link classes. Reified link classes were chosen because they keep the spec inside plain RDF / OWL 2 — no RDF-star toolchain dependency, no quad assumption — and because they place the role information on a node that can itself be queried, ordered, and constrained. *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md))*

**Three reified link classes: `ContextFact`, `DeliberationFact`, `OutcomeFact`.** A single fact class would have to carry phase as a property and accommodate optional `onAlternative` and valence with branching cardinality rules; the three-class split makes phase a class fact and allows per-class field shapes without conditional constraints. `ContextFact` earns its own class because pure framing statements ("we run three Python microservices") are neither comparisons of options nor post-decision realities; collapsing them into `DeliberationFact` would require relaxing required fields and confuse query intent. The three classes are parallel in shape and naming for tooling consistency. *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md))*

**`rdf:List` ordering at the record level.** Each record carries up to three lists (`hasContext`, `hasDeliberation`, `hasOutcome`) that order the facts in the author's intended sequence. Ordering lives on the placement (the list membership), not on the `Consideration`, so the same atom may appear at different positions in different records or different roles. The trade-off is well-known: `rdf:List` is awkward in OWL DL reasoning over membership, but SPARQL handles it cleanly and the planned SHACL companion will encode list-element typing. *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md))*

**Per-class valence enums with single neutral (KISS).** `DeliberationFact` carries `Supports`, `Against`, or `Neutral`; `OutcomeFact` carries `Benefit`, `AcceptedCost`, `Risk`, or `FollowUp`. Splitting `Neutral` further (orthogonal vs in-scope-but-non-decisive) was considered and deferred — the single bucket is enough to record "this was noted but didn't push the choice", which is the practically observed need. Each enum is a SKOS concept scheme (`adr-o:deliberationValenceScheme`, `adr-o:outcomeValenceScheme`) wired into a named class (`DeliberationValence`, `OutcomeValence`) via the same `skos:Concept ∩ (skos:inScheme = …)` pattern used by `adr-o:Status`. Each scheme has its own `owl:AllDifferent` block over its members, with the same caveat about lightweight OWL 2 RL reasoners that do not implement `eq-diff2` / `eq-diff3` as already documented for the status `AllDifferent` in 0.1.0-draft. *(→ [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md))*

**`chosenAlternative` as an `owl:FunctionalProperty` with cardinality 0..1.** The functional declaration enforces "at most one chosen alternative per record" at the OWL level. Cardinality 0 is permitted to accommodate `Proposed` records that have not yet picked an option; the integrity rule "if `hasStatus = Accepted` then `chosenAlternative` is present" is an SHACL concern and is deferred. Multi-winner decisions ("use both X and Y") are modelled as a single composite `Alternative` rather than by relaxing cardinality, which keeps the predicate cleanly queryable as a single edge. *(→ [ADR-0012](/ADL/ADR-0012-alternatives-and-chosen-option.md))*

**`addresses` allowed on both `DecisionRecord` and `Consideration`.** The canonical, fine-grained use is on `Consideration` (a specific atom is about a specific concern). The use on `DecisionRecord` is a permitted editorial rollup that human and tooling layers may apply for discovery convenience — "this whole record is broadly about latency" — and is not authoritative when both are present. This dual home leaves the future GADR-style `DecisionTemplate` design fully open: templates can either inherit `addresses` directly via a future shared superclass, or rely entirely on consideration-level concern tagging, without breaking either current use. The OWL declaration drops the previous `rdfs:domain DecisionRecord` restriction; the scope note spells out which use is canonical. *(→ [ADR-0013](/ADL/ADR-0013-addresses-dual-placement.md))*

**Two valence properties (`deliberationValence`, `outcomeValence`), not one.** A single `valence` predicate would either need a permissive union range or push range enforcement entirely into SHACL. Two properties make each ontology declaration self-documenting (clean `rdfs:range`) at the cost of one extra IRI; given the disjointness of the two enums by design, the two-property shape is more honest about what is actually being said. *(→ [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md))*

**Destruction without compatibility shim.** The four Nygard datatype properties (`context`, `decision`, `consequences`, `rationale`) and `wasRejectedBecause` are removed outright, not deprecated. `wasRejectedBecause` is now expressible as a `DeliberationFact` on the rejected `Alternative` with `Against` valence; keeping it would create two ways to say the same thing. The 0.1.0-draft ontology was never published or used outside the author's working copy, so no migration is required; the absence of compatibility scaffolding is a deliberate signal that 0.2.0 is a clean break. *(→ [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md))*

</div>
</div>

<div data-dn-section="softer">
<div data-dn-passage="softer-defaults-020" data-dn-record="meta">

### Softer defaults — "simplicity first, easier to add than to refactor"

These are picks made under the same simplicity-first heuristic that 0.1.0 used. Each is reversible without breaking existing 0.2.0-draft graphs.

**`onAlternative` required on `DeliberationFact`, optional on `OutcomeFact`, not used on `ContextFact`.** A deliberation always weighs an option, so requiring the link aligns with intent and lets the planned SHACL companion catch malformed authoring at validation time. An outcome usually pertains to the chosen alternative (and so the link is left implicit), but counterfactual outcomes about rejected options ("if we had gone with B, this risk would have materialised") are valuable enough to allow explicit linking. Context is by construction not about any specific option. These are authoring conventions documented in scope notes; SHACL enforcement is deferred. *(→ [ADR-0012](/ADL/ADR-0012-alternatives-and-chosen-option.md))*

**Composite multi-winner decisions modelled as a single `Alternative`, not by relaxing `chosenAlternative` cardinality.** Trades a small amount of authoring friction (the author must mint a combined option such as `:alt-X-and-Y`) for keeping `chosenAlternative` queryable as a single functional edge. Reversible if the trade-off proves wrong in practice. *(→ [ADR-0012](/ADL/ADR-0012-alternatives-and-chosen-option.md))*

**`consideration` and `onAlternative` declared `owl:FunctionalProperty`.** A fact carries exactly one consideration and at most one alternative. This catches malformed facts at OWL-DL reasoning time without requiring SHACL. *(→ [ADR-0012](/ADL/ADR-0012-alternatives-and-chosen-option.md))*

**No SHACL shapes shipped, even though the new architecture relies on rules SHACL would express well.** The 0.2.0-draft is a technical preview; a SHACL companion would harden the integrity story (list-element typing, `chosenAlternative ∈ hasAlternative`, `Accepted ⇒ chosenAlternative present`, per-fact-class field cardinalities, valence-enum membership), but shipping it now would amount to over-investment in a shape the community has not yet had a chance to challenge. Listed below in the deferrals. *(Deferred — no ADL record; rationale aligned with [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md).)*

**Version annotations: `owl:versionIRI <https://w3id.org/adr-o/0.2.0>` and `owl:versionInfo "0.2.0-draft"`.** The major-version bump from 0.1 to 0.2 reflects the destructive break; the `-draft` suffix is preserved to signal that further breaking churn is expected before 0.2.0 is tagged as released. *(→ [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md))*

</div>
</div>

<div data-dn-section="iri-convention-consideration-reuse">
<div data-dn-passage="iri-convention-consideration-reuse-020" data-dn-record="meta">

### IRI convention for `Consideration` reuse

Reusing the same `Consideration` IRI across multiple records is the YADR-style identity property and the central design payoff of atom-first reification. The ontology imposes no axiom about where these IRIs live; the convention below is editorial, not normative. Tools (renderers, MCP servers, validation layers) are expected to treat both patterns uniformly. *(→ [ADR-0018](/ADL/ADR-0018-consideration-iri-reuse-convention.md))*

- **Record-local default.** An atom that emerges in a single decision lives as a fragment under that record, e.g. `<https://example.org/adl/ADR-0042#cons-ecosystem-maturity>`. Easy to author, no governance overhead, works for atoms that may never be reused.
- **Org-shared promotion.** An atom deliberately reused across records, or one underpinning a future GADR-style decision template, lives under a separate organisational namespace, e.g. `<https://example.org/considerations/ecosystem-maturity>`. Cross-record queries about this atom no longer have to traverse a record-specific IRI.
- **Bridge between the two.** When promoting a record-local atom to org-shared, the recommended bridge is `owl:sameAs` from the shared IRI to the local one. Existing references in the originating record continue to resolve; new records reference the shared IRI directly.

The convention does not add ontology axioms. Future work may formalise the scope distinction (e.g. an `adr-o:scope` predicate) without breaking existing IRIs. *(→ [ADR-0018](/ADL/ADR-0018-consideration-iri-reuse-convention.md))*

</div>
</div>

<div data-dn-section="deferrals">
<div data-dn-passage="explicit-deferrals-020" data-dn-record="deferral">

### Explicit deferrals — decisions to not decide right now

These are items considered during this iteration and deliberately punted. Listing them is the antidote to silent forgetting.

**Goals / `to-achieve` (the YADR slot).** No first-class home for the "intended outcomes the choice is meant to produce" notion. Two upgrade paths are noted: a `Goal` subclass of `Consideration` with `intendsToAchieve` semantics, used in `ContextFact` or `DeliberationFact`; or a dedicated predicate `intendsToAchieve` from `DecisionRecord` directly to a `Goal` node. Authors who need this slot in 0.2.0-draft are recommended to express it as `ContextFact`s ("we want low p99 latency") or as positive-valence `DeliberationFact`s on the chosen option, accepting that the goal/observation distinction is currently lost.

**SHACL shapes graph for validation.** Promoted from "deferred without commitment" (0.1.0) to "deferred at technical-preview status" (0.2.0). The new architecture relies more heavily on SHACL than the 0.1.0 design did — list-element typing, chosen-alternative integrity, per-fact-class field shapes, valence-enum membership per fact class. None of these are shipped. A future iteration is expected to publish a shapes graph alongside the OWL.

**GADR-style `DecisionTemplate` class.** A template is a record-shaped node carrying `hasAlternative` and `DeliberationFact`s but no `chosenAlternative` and no `OutcomeFact`s. The 0.2.0-draft architecture is shape-compatible with this future class and even invites it: `addresses` may appear on both `DecisionRecord` and `Consideration`, which preserves room for a template superclass or consideration-only tagging without breaking either pattern. Explicitly deferred until the templating mechanism is needed in practice — the class is not declared in 0.2.0-draft.

**`amends` and `clarifies` as typed link predicates** (and matching `Amended`, `Clarified` status individuals). Mentioned in IDEAS as gaps in 0.1.0; still gaps in 0.2.0. This deferral was later lifted by adopting a unified amendment model: `clarifies` is collapsed into amendment semantics, and `Amended`/`Clarified` statuses are explicitly rejected in favor of topology-based detection via `amendedBy`. *(Lifted by [ADR-0019](/ADL/ADR-0019-clarifications-are-amendments.md) and [ADR-0020](/ADL/ADR-0020-amendments.md).)*

**Fitness functions / confirmation predicates.** MADR's "Confirmation" slot — how will you know this decision is being followed? — has no first-class predicate in 0.2.0. Likely shape: a property from `DecisionRecord` to a test, CI rule, or SHACL shape. Deferred.

**Splitting `Neutral` into orthogonal vs in-scope-but-non-decisive.** Considered and KISS-rejected for this iteration. Adding a second neutral concept to `deliberationValenceScheme` later is non-breaking.

**Kruchten-style directed `prevents` / `isPreventedBy`** alongside symmetric `conflictsWith`. Still deferred from 0.1.0 reasoning.

**MADR-style decider / consulted / informed roles.** Still deferred from 0.1.0. *(Lifted by [ADR-0021](/ADL/ADR-0021-social-role-predicates.md).)*

**`significance` / "architectural" filter.** Still deferred from 0.1.0.

**Additional serializations (JSON-LD, RDF/XML).** Still deferred from 0.1.0.

**Domain profile registry.** Still deferred from 0.1.0.

**`adr-o:hasType` / Group categorization.** Still deferred from 0.1.0.

</div>
</div>

<div data-dn-section="still-holds">
<div data-dn-passage="still-holds-020" data-dn-record="meta">

### What 0.1.0-draft decisions still hold

This iteration changes the body shape of `DecisionRecord`, but a substantial portion of 0.1.0 is preserved. Each item below is carried forward with no semantic change.

- `adr-o:DecisionRecord` as the central class name (not `ArchitectureDecisionRecord`); domain-agnostic scope. *(→ [ADR-0002](/ADL/ADR-0002-scope.md))*
- Namespace `https://w3id.org/adr-o#`; hash IRIs. *(→ [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md))*
- Single Turtle file under `ontology/`; no `examples/` or `shapes/` directories yet. *(Repo convention; see note at 0.1.0 softer defaults.)*
- No `owl:imports` for PROV-O / SKOS / DC Terms. *(→ [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md))*
- License CC BY 4.0. *(→ [ADR-0001](/ADL/ADR-0001-license.md))*
- Status concept scheme, the five status individuals (`Proposed`, `Accepted`, `Deprecated`, `Superseded`, `Rejected`), `adr-o:Status` wired as `skos:Concept ∩ (skos:inScheme = adr-o:statusScheme)`, `hasStatus` functional, `owl:AllDifferent` over the five statuses with the same RL-reasoner caveat. *(→ [ADR-0008](/ADL/ADR-0008-status-skos-and-integrity.md))*
- `adr-o:index` as `xsd:integer` separate from `dcterms:identifier`. *(→ [ADR-0007](/ADL/ADR-0007-index-vs-identifier.md))*
- `adr-o:supersedes` as a sub-property of `prov:wasRevisionOf`. *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*
- The five surviving relational predicates from the 0.1.0 "six core" set: `supersedes`, `supersededBy`, `dependsOn`, `enables`, `conflictsWith`. (`addresses` survives as well but is refined: canonical use on `Consideration`, optional editorial rollup on `DecisionRecord`; see "Strong, well-articulated decisions" above.) *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md); `addresses` refinement [ADR-0013](/ADL/ADR-0013-addresses-dual-placement.md))*
- The `addresses`/`affects` split (problem side vs manifestation side); `affects` unchanged (open `rdfs:Resource` range). *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*
- `Concern` as an abstract anchor class with no core concept scheme. *(→ [ADR-0002](/ADL/ADR-0002-scope.md))*
- `dcterms:date` as the canonical date predicate; `dcterms:creator` as the canonical authorship predicate. *(→ [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md))*
- `conflictsWith` as `owl:SymmetricProperty`, not directed Kruchten pair. *(→ [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*
- Editorial `skos:scopeNote`s on reused predicates (`dcterms:title`, `dcterms:identifier`, `dcterms:date`, `dcterms:created`, `dcterms:modified`, `dcterms:creator`, `dcterms:references`, `dcterms:license`, `prov:wasAttributedTo`, `prov:wasRevisionOf`). *(→ [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md))*

</div>
</div>

<div data-dn-section="reversed">
<div data-dn-passage="reversed-020" data-dn-record="superseded">

### What 0.1.0-draft decisions are reversed

- **The four Nygard datatype properties (`context`, `decision`, `consequences`, `rationale`) are removed outright.** The 0.1.0 reasoning was "without them, instances cannot represent even a textbook ADR"; the 0.2.0 architecture replaces "represent prose on the record" with "represent structure on the record and let tooling materialise prose," so the original justification no longer applies. *(→ [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md))*
- **`wasRejectedBecause` as a datatype property is removed.** Its function is now served by a `DeliberationFact` on the rejected `Alternative` with `Against` valence, which also gives the rejection rationale a reusable IRI rather than an opaque literal. *(→ [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md))*
- **`consequences` as a single property** (the deferred-split decision in 0.1.0) is moot: there is no `consequences` property at all in 0.2.0. Its role is replaced by `OutcomeFact` with the four-valued outcome valence enum, which subsumes both the positive/negative split that MADR documents and the YADR-style `accepting-that` distinction (now `AcceptedCost`). *(→ [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md), [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md))*
- **The deferral of "first-class `Force` / `Constraint` / `Rationale` / `Consequence` classes" is resolved (in part).** A single `Consideration` class together with three `*Fact` link classes covers the practical need that motivated all four candidate classes; the four-class proliferation is no longer the upgrade path. *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md))*

</div>
</div>

<div data-dn-section="versioning-note">
<div data-dn-passage="versioning-note-020" data-dn-record="meta">

### Versioning note

The jump from `0.1.0-draft` to `0.2.0-draft` is a destructive shape change: every triple using one of the removed predicates becomes invalid, and the new `Consideration` / `*Fact` infrastructure has no autogenerable correspondence to the old prose properties. Under any sane semantic-versioning interpretation this is a major break. Two facts make the cost negligible: the 0.1.0-draft ontology was never published or used outside the author's working copy, and the `-draft` suffix on both versions signals exactly this kind of pre-release churn.

The convention adopted here — promoting `DESIGN-NOTES.md` itself to a versioned document with one section per ontology iteration — is the project's own answer to the question "where do we record the rationale for these large iterations without writing a flurry of micro-ADRs in `ADL/`?" Each version section is immutable history; later sections reverse earlier decisions explicitly rather than editing them in place. That property, applied to the design notes themselves, is the meta-application of the principle ADR-O exists to defend. *(Meta-process; subsequent micro-ADRs for vocabulary are linked inline above.)*

</div>
</div>
</div>

<div data-dn-version="0.2.1-draft" data-dn-record="meta" id="adro-0-2-1-draft">

## ADR-O 0.2.1-draft

<div data-dn-section="iteration-character">

### Iteration character

<div data-dn-passage="iteration-character-overview" data-dn-record="adl" data-dn-adl-refs="0010,0011,0012,0013,0004,0015,0016,0017">

This iteration is purely tactical and non-destructive. The 0.2.0-draft architectural shape — atom-first reification, three `*Fact` link classes, two valence enums, `rdf:List` ordering, the "KG lives under tooling" stance — is preserved without modification. The 0.2.1-draft goal is to close the gap between the 0.2.0 ontology and the governing ADRs: specifically to apply ADR-0003's Markdown datatype convention, acknowledge the post-factum and real-time ADR authoring cases established in ADR-0005 in the relevant scope note, and add one small metadata predicate (`dcterms:version`) that the ROADMAP identified as an immediate gap. Changes are predominantly additive (new scope notes, new `owl:AnnotationProperty` declarations, one literal retyping), with one editorial correction to an existing `rdfs:comment` on `adr-o:Consideration`. No existing terms are removed, repurposed, or semantically altered. *(Shape preserved: [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md)–[ADR-0013](/ADL/ADR-0013-addresses-dual-placement.md), [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md); this iteration’s adds: [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md), [ADR-0016](/ADL/ADR-0016-dcterms-version-in-record.md), [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md).)*

</div>
</div>

<div data-dn-section="decisions-applied">

### Decisions from prior drafts applied in this iteration

<div data-dn-passage="adr-0003-markdown-applied" data-dn-record="adl" data-dn-adl-refs="0017,0003">

**ADR-0003 Markdown datatype convention — applied, with scope narrowed under the 0.2.0 architecture.**

ADR-0003 was accepted one day after 0.2.0-draft shipped and explicitly deferred the ontology application to "the next ontology iteration." That iteration is now. The convention lands as `skos:scopeNote` annotations on the affected annotation properties; no `rdfs:range <…/text/markdown>` assertions are added (ADR-0003's "working answer" was "annotation now, SHACL constraint when shipped", and the SHACL companion remains deferred). *(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md); base decision [ADR-0003](/ADL/ADR-0003-prose-literals-markdown.md))*

The scope of the Markdown convention is narrowed in two respects relative to ADR-0003's original "Properties in scope" sentence, both grounded in tracing the ADR-0005 vim vignette through the 0.2.0 graph:

1. **`*Fact` prose properties — none exist, none minted.** ADR-0003 referred to "the corresponding prose properties on `ContextFact`, `DeliberationFact`, and `OutcomeFact` when authors attach prose to a fact directly." At the time, the 0.2.0 atom-first shape was still being internalized. Under the 0.2.0 architecture, `*Fact` instances are pure structural placement nodes: they carry only `adr-o:consideration`, `adr-o:onAlternative` (where applicable), and a valence. Prose is reached transitively via `adr-o:consideration` → the linked `Consideration` IRI. Tracing the vignette confirms this: `:df-1` and `:df-2` carry zero literal properties. There are no Fact-level prose properties to retype, and none will be minted; the Markdown convention does not apply to `*Fact` instances at all. *(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md))*

2. **`skos:prefLabel` — excluded, treated as a label.** ADR-0003 listed `skos:prefLabel` alongside `skos:definition` and `skos:note`. ADR-0003 also explicitly stated "rdfs:label is not in scope — a label is a short string identifier, not prose." The vim vignette shows `skos:prefLabel` functioning identically to a label throughout: `:alt-vim-migration` carries `skos:prefLabel "Migrate to vim"@en`; `:cons-vscode-plugin-investment` carries `skos:prefLabel "VSCode plugins enable fast onboarding"@en`. These are noun-phrase atom titles, not Markdown prose. The prose is in `dcterms:description`. Retyping these as `^^…/text/markdown` would create the contract-mismatch ADR-0003 exists to avoid. `skos:prefLabel` is therefore excluded; it joins `rdfs:label` and `dcterms:title` as short-label predicates outside the Markdown convention. *(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md))*

The narrowed in-scope set is: `dcterms:description`, `skos:definition`, `skos:note` (and its sub-properties: `skos:scopeNote`, `skos:editorialNote`, `skos:historyNote`, `skos:changeNote`, `skos:example`). *(Same restatement: [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md); original datatype decision [ADR-0003](/ADL/ADR-0003-prose-literals-markdown.md).)*

</div>

<div data-dn-passage="consideration-comment-correction" data-dn-record="adl" data-dn-adl-refs="0017">

**`adr-o:Consideration` `rdfs:comment` corrected — corollary of the `skos:prefLabel` exclusion.** The 0.2.0-draft `rdfs:comment` on `adr-o:Consideration` stated: *"The text payload is carried via standard predicates such as `dcterms:description` and `skos:prefLabel`."* This directly contradicts the `skos:prefLabel` exclusion decision above: `Consideration` is the primary prose-carrier in the ontology, yet its own class comment named a short-label predicate as a prose carrier. The sentence is corrected to: *"The prose payload is carried via `dcterms:description` (primary, Markdown-typed per ADR-0003), `skos:definition` (formal definition, Markdown-typed), and `skos:note` sub-properties (supplementary annotation prose, Markdown-typed); `skos:prefLabel` carries a short noun-phrase label, not prose."* This is the only edit to an existing `adr-o:` term annotation in this iteration; it is a correction, not a semantic change. *(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md))*

</div>

<div data-dn-passage="vim-vignette-prose-landing" data-dn-record="adl" data-dn-adl-refs="0017,0005">

**Where Markdown prose actually lands — the vim vignette as a concrete trace.**

The ADR-0005 ambient-transcription vignette (the five-minute 2032 vim-migration proposal-and-rejection) produces a `DecisionRecord`, one `Alternative`, two `DeliberationFact` placements, and two `Consideration` atoms. Every literal in the resulting graph falls into one of two buckets:

- *Carries Markdown prose:* `Consideration.dcterms:description` (Alice's full spoken rationale, normalised — this is the primary home for prose in any ADR-O record); `Consideration.skos:note` sub-properties (supplementary annotation prose, e.g. clarifying which plugin generation the consideration refers to); `Consideration.skos:definition` (formal-definition payload when present); `DecisionRecord.dcterms:description` (record-level prose summary).
- *Does not carry Markdown prose:* all three `*Fact` classes (zero literal properties, as noted above); `Alternative.skos:prefLabel` (`"Migrate to vim"@en`); `Consideration.skos:prefLabel` (`"VSCode plugins enable fast onboarding"@en`); `DecisionRecord.dcterms:title`, `dcterms:identifier`, `dcterms:date`, and all other non-prose metadata predicates.

*(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md); vignette policy [ADR-0005](/ADL/ADR-0005-log-all-decisions.md))*

</div>

<div data-dn-passage="literal-retyped-in-ontology" data-dn-record="adl" data-dn-adl-refs="0017" data-dn-roadmap="H1">

**One literal retyped in the ontology itself.** The ontology header `dcterms:description` at line 17 (the "A domain-agnostic RDF/OWL 2 ontology for Architecture Decision Records…" literal) is retyped to `^^<https://www.w3.org/ns/iana/media-types/text/markdown>`. This is the single in-ontology data literal flagged by the ROADMAP. Bulk retyping of existing `skos:definition` literals on status/valence individuals is deferred to a future data-cleanup pass (likely co-shipped with the SHACL companion), which will also be the moment to verify whether any existing `skos:prefLabel` literals were inadvertently written as prose rather than labels. *(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md))*

</div>

<div data-dn-passage="post-factum-created-scope-note" data-dn-record="adl" data-dn-adl-refs="0015,0005">

**ADR-0005 post-factum and real-time ADR cases — acknowledged in `dcterms:created` scope note.** The 0.2.0-draft `dcterms:created` scope note ("optional record-creation timestamp, distinct from `dcterms:date` when the log distinguishes decision date from record lifecycle") was implicitly about post-factum ADRs without naming them. The ROADMAP flagged this as requiring an explicit name. The scope note is expanded to state: *"The typical case is a post-factum ADR, where the decision happened earlier than the record was authored: `dcterms:date` captures when the decision was made, `dcterms:created` captures when the record itself was written."*

The `DecisionRecord` `rdfs:comment` ("A record of a single, load-bearing decision…") is left as-is per ROADMAP recommendation: ADR-0005 now supplies the correct reading of "load-bearing" as a retrospective property, and a defensive rewrite of the `rdfs:comment` would add weight without adding clarity. *(Temporal split: [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md); policy [ADR-0005](/ADL/ADR-0005-log-all-decisions.md))*

</div>
</div>

<div data-dn-section="strong">

### Strong, well-articulated decisions

<div data-dn-passage="dcterms-version-added" data-dn-record="adl" data-dn-adl-refs="0016" data-dn-roadmap="H1">

**`dcterms:version` declared as `owl:AnnotationProperty` for per-record iteration tracking.** The ROADMAP identified the absence of an in-record version predicate as a gap: the supersession chain (`adr-o:supersedes` / `adr-o:supersededBy`) models replacement by a successor record with a new identity, but it cannot track minor amendments to the same record that fall short of warranting a supersession. `dcterms:version` fills this gap — it carries a literal version tag (e.g. `"1.2"`) against a `DecisionRecord` IRI that remains stable across those amendments. The scope note explicitly disambiguates it from the supersession chain. At 0.2.1-draft time, amendment predicates were still deferred; that deferral was later lifted by the unified amendment decisions in ADR-0019/ADR-0020. *(→ [ADR-0016](/ADL/ADR-0016-dcterms-version-in-record.md); later lift: [ADR-0019](/ADL/ADR-0019-clarifications-are-amendments.md), [ADR-0020](/ADL/ADR-0020-amendments.md))*

</div>

<div data-dn-passage="skos-note-declared" data-dn-record="adl" data-dn-adl-refs="0017">

**`skos:note` declared as `owl:AnnotationProperty`.** SKOS defines `skos:note` as the parent of the note family (`skos:scopeNote`, `skos:editorialNote`, `skos:historyNote`, `skos:changeNote`, `skos:example`). The ontology already used `skos:scopeNote` extensively (as it was declared) but did not formally register `skos:note` itself. Declaring it now makes the Markdown scope note on the parent property explicit and lets implementers reliably attach supplementary prose to `Consideration` nodes via any sub-property without finding an undeclared parent. *(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md))*

</div>
</div>

<div data-dn-section="softer">

### Softer defaults — "simplicity first, easier to add than to refactor"

<div data-dn-passage="markdown-scope-note-only" data-dn-record="adl" data-dn-adl-refs="0017">

**Scope-note-only for the Markdown convention (no `rdfs:range`).** OWL gives annotation-property range declarations no DL semantics; asserting `rdfs:range <…/text/markdown>` would be editorial rather than operative. Keeping enforcement in the planned SHACL companion (where `sh:datatype` carries real enforcement weight) is the cleaner split. The scope notes are the convention's current home. *(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md))*

</div>

<div data-dn-passage="skos-note-no-subproperty-assertion" data-dn-record="adl" data-dn-adl-refs="0017">

**`skos:note` declared without asserting it as a sub-property of anything.** SKOS is not `owl:import`-ed, so asserting sub-property relationships with SKOS terms would require importing the full SKOS schema or adding potentially surprising axioms. The declaration stands alone; sub-property entailments are a matter of the SKOS schema, not something this ontology needs to assert. *(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md))*

</div>
</div>

<div data-dn-section="deferrals">

### Explicit deferrals — decisions to not decide right now

<div data-dn-passage="bulk-skos-definition-retyping" data-dn-record="deferral" data-dn-roadmap="H3.4">

**Bulk literal retyping of existing `skos:definition` strings on status/valence individuals.** The status-scheme and valence-scheme individuals carry `skos:definition` literals that are currently bare strings (no `^^…/text/markdown` type tag). Retyping them is a data-cleanup task, not a shape change; it is deferred to a future data pass, likely co-shipped with the SHACL companion, when all literal typing can be validated in one pass. *(Still deferred; convention when applied: [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md).)*

</div>

<div data-dn-passage="prefLabel-retyping-closed" data-dn-record="adl" data-dn-adl-refs="0017">

**`skos:prefLabel` bulk retyping — permanently off the table.** `skos:prefLabel` literals in ADR-O graphs are short noun-phrase labels, not prose, and will not be retyped to the Markdown datatype. This is not a deferral; it is a closed decision (see "Decisions from prior drafts applied in this iteration" above). *(→ [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md))*

</div>

<div data-dn-passage="shacl-companion-deferred" data-dn-record="deferral" data-dn-roadmap="H3.4">

**SHACL shapes graph for validation.** Still deferred, promoted to "deferred pending stable 0.x shape" now that 0.2.1 closes the immediate annotation gaps. When shipped, the companion will add `sh:datatype <…/text/markdown>` constraints on the three prose-carrier properties, list-element typing, and the integrity rules that scope notes currently carry in prose.

</div>

<div data-dn-passage="significance-deferred" data-dn-record="deferral" data-dn-roadmap="4">

**`adr-o:significance` as a relative-importance annotation.** Still deferred; ADR-0005 explicitly recommends it as a post-entry annotation rather than an entry gate, but its shape (a literal, a SKOS concept, a numeric value?) is not resolved. Deferred to a future ADR.

</div>

<div data-dn-passage="versioning-discipline-adr-deferred" data-dn-record="deferral" data-dn-roadmap="4">

**A dedicated versioning-discipline ADR.** DESIGN-NOTES 0.1.0 flagged the absence of a versioning-strategy ADR (semantic versioning? release cadence? deprecation policy?) as worth writing before anyone depends on ADR-O in production. Still pending; the project has grown two more `-draft` iterations without resolving this, which makes the debt larger, not smaller.

</div>

<div data-dn-passage="seeAlso-backrefs-deferred" data-dn-record="deferral" data-dn-roadmap="4">

**`rdfs:seeAlso` per-ADR back-references from the ontology.** ROADMAP §2 notes that the ontology cites its public docs via `rdfs:seeAlso` but does not cite individual ADR IRIs. Adding them would make the relationship bidirectional — a taste call marked "not required." Remains deferred.

</div>
</div>

<div data-dn-section="still-holds">

### What 0.2.0-draft decisions still hold

<div data-dn-passage="still-holds-list" data-dn-record="adl" data-dn-adl-refs="0010,0011,0009,0012,0004,0013,0008,0007,0006,0014,0001">

This iteration changes nothing structural. Every 0.2.0 commitment is preserved:

- Atom-first reification: `Consideration` as a first-class reusable atom; `*Fact` link classes as placement nodes. *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md))*
- Three reified link classes (`ContextFact`, `DeliberationFact`, `OutcomeFact`); no Nygard body literals. *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md), [ADR-0011](/ADL/ADR-0011-strict-graph-no-nygard-body.md))*
- Per-class valence enums: `deliberationValenceScheme` (`Supports`, `Against`, `Neutral`); `outcomeValenceScheme` (`Benefit`, `AcceptedCost`, `Risk`, `FollowUp`); `owl:AllDifferent` blocks over each set. *(→ [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md))*
- `chosenAlternative` as `owl:FunctionalProperty` (0..1 cardinality); multi-winner decisions modelled as composite `Alternative`. *(→ [ADR-0012](/ADL/ADR-0012-alternatives-and-chosen-option.md))*
- `rdf:List` ordering at the record level (`hasContext`, `hasDeliberation`, `hasOutcome`). *(→ [ADR-0010](/ADL/ADR-0010-consideration-and-reified-facts.md))*
- The "KG lives under tooling" design assumption (per ADR-0004): naming decisions, SHACL deferral, scope-notes-over-axioms pattern, `addresses` dual-home. *(→ [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md), [ADR-0013](/ADL/ADR-0013-addresses-dual-placement.md))*
- Status concept scheme and the five status individuals; SKOS-scheme wiring for `Status`, `DeliberationValence`, `OutcomeValence`. *(→ [ADR-0008](/ADL/ADR-0008-status-skos-and-integrity.md), [ADR-0009](/ADL/ADR-0009-deliberation-and-outcome-valence.md))*
- `adr-o:index` as `xsd:integer`; `adr-o:supersedes` as sub-property of `prov:wasRevisionOf`. *(→ [ADR-0007](/ADL/ADR-0007-index-vs-identifier.md), [ADR-0006](/ADL/ADR-0006-core-relational-predicates.md))*
- No `owl:imports` for PROV-O / SKOS / DC Terms. *(→ [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md))*
- Hash IRIs; single Turtle file under `ontology/`; CC BY 4.0 license. *(→ [ADR-0014](/ADL/ADR-0014-ontology-document-shape.md), [ADR-0001](/ADL/ADR-0001-license.md); single-file note is repo-only.)*

</div>
</div>

<div data-dn-section="reversed">

### What 0.2.0-draft decisions are reversed

<div data-dn-passage="reversed-none" data-dn-record="meta">

None.

</div>
</div>

<div data-dn-section="versioning-note">

### Versioning note

<div data-dn-passage="versioning-note-021" data-dn-record="adl" data-dn-adl-refs="0015,0016,0017">

The 0.2.0 → 0.2.1 bump is non-destructive. The complete set of changes: two new `owl:AnnotationProperty` declarations (`skos:note` and `dcterms:version`), each with a scope note; new scope notes on two existing annotation-property declarations (`dcterms:description`, `skos:definition`); one expanded scope note on an existing declaration (`dcterms:created`); one literal retyped in the ontology header (`dcterms:description`, bare string → Markdown datatype); one `rdfs:comment` corrected on `adr-o:Consideration` (removed the erroneous `skos:prefLabel` as a prose-carrier, named the three Markdown-typed prose predicates explicitly). No terms removed, no terms repurposed, no existing triples invalidated. A patch-version bump is the correct semantic-versioning signal for a `-draft` line under these conditions: no `adr-o:` terms were added or removed, and the one `adr-o:` annotation edited is a correction of a documentation error. *(Normative restatement of these deltas: [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md), [ADR-0016](/ADL/ADR-0016-dcterms-version-in-record.md), [ADR-0017](/ADL/ADR-0017-markdown-properties-as-implemented.md).)*

</div>
</div>

</div>

<div data-dn-version="0.2.2-draft" data-dn-record="meta" id="adro-0-2-2-draft">

## ADR-O 0.2.2-draft

<div data-dn-section="iteration-character">

### Iteration character

<div data-dn-passage="iteration-character-overview-022" data-dn-record="adl" data-dn-adl-refs="0020">

This iteration is a small, non-destructive vocabulary extension focused on amendments. The 0.2.1-draft shape remains intact: no classes are removed or repurposed, no cardinalities are tightened, and no existing records are invalidated. The change is to operationalize the amendment model from ADR-0020 by minting two relational predicates (`adr-o:amends`, `adr-o:amendedBy`) and by explicitly rejecting status inflation (`adr-o:Amended` is not added). The goal is to represent patch-style evolution directly in graph topology rather than in redundant status labels. *(→ [ADR-0020](/ADL/ADR-0020-amendments.md))*

</div>
</div>

<div data-dn-section="strong">

### Strong, well-articulated decisions

<div data-dn-passage="amendment-predicates-added-022" data-dn-record="adl" data-dn-adl-refs="0020">

**`adr-o:amends` / `adr-o:amendedBy` added as a directed amendment pair over `DecisionRecord`.** `adr-o:amends` links a patch record to an anchor record it amends; `adr-o:amendedBy` is the declared inverse for anchor-to-patches traversal. Both use `DecisionRecord` as domain and range, and both are intentionally non-functional (0..N) so one anchor can accumulate multiple amendment records over time. This preserves the supersession chain for identity replacement while adding a separate channel for layered modification. *(→ [ADR-0020](/ADL/ADR-0020-amendments.md))*

</div>

<div data-dn-passage="amended-status-rejected-022" data-dn-record="adl" data-dn-adl-refs="0020,0004,0008">

**`adr-o:Amended` status explicitly rejected.** Amendment presence is treated as topological fact (`?anchor adr-o:amendedBy ?patch`) rather than as a duplicated state label on `hasStatus`. This follows ADR-0004's reconstructability boundary rule: if tooling can derive a condition from graph structure, ADR-O should not store a second, synchronization-prone indicator for the same condition. The status scheme remains unchanged (`Proposed`, `Accepted`, `Deprecated`, `Superseded`, `Rejected`). *(→ [ADR-0020](/ADL/ADR-0020-amendments.md); rule source [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md), status scheme [ADR-0008](/ADL/ADR-0008-status-skos-and-integrity.md))*

</div>
</div>

<div data-dn-section="softer">

### Softer defaults — "simplicity first, easier to add than to refactor"

<div data-dn-passage="dcterms-version-scope-note-tightened-022" data-dn-record="adl" data-dn-adl-refs="0020,0016">

**`dcterms:version` scope note wording tightened to "amended" only.** The 0.2.1 wording referenced "amended or clarified"; after ADR-0020's unification, clarifications are represented as amendments, so the scope note now names only amendment. This is editorial alignment, not a semantic change to `dcterms:version` itself. *(→ [ADR-0020](/ADL/ADR-0020-amendments.md); original declaration [ADR-0016](/ADL/ADR-0016-dcterms-version-in-record.md))*

</div>
</div>

<div data-dn-section="deferrals">

### Explicit deferrals — decisions to not decide right now

<div data-dn-passage="deferral-lifted-amends-clarifies-022" data-dn-record="adl" data-dn-adl-refs="0020">

**The 0.2.0 deferral on typed amendment links is now lifted.** `amends` is implemented, `clarifies` is not added as a separate predicate, and status additions (`Amended`, `Clarified`) are explicitly rejected as redundant. The open work is now tooling behavior (merge-and-render of anchor plus amendments), not core vocabulary design.

</div>
</div>

<div data-dn-section="still-holds">

### What 0.2.1-draft decisions still hold

<div data-dn-passage="still-holds-list-022" data-dn-record="meta">

All 0.2.1-draft structural decisions remain in force: atom-first reification, three `*Fact` classes, strict graph (no Nygard literals), per-class valence schemes, `chosenAlternative` as functional, and `rdf:List` ordering. The Markdown-typing convention and `dcterms:version` declaration from 0.2.1 also remain unchanged in intent. 0.2.2 adds amendment topology; it does not revise the 0.2.1 substrate.

</div>
</div>

<div data-dn-section="reversed">

### What 0.2.1-draft decisions are reversed

<div data-dn-passage="reversed-none-022" data-dn-record="meta">

None.

</div>
</div>

<div data-dn-section="versioning-note">

### Versioning note

<div data-dn-passage="versioning-note-022" data-dn-record="adl" data-dn-adl-refs="0020">

The 0.2.1 → 0.2.2 bump is non-destructive and additive: two object properties are introduced (`amends`, `amendedBy`), one existing scope note is tightened for terminology consistency (`dcterms:version`), and ontology version metadata is advanced to `0.2.2-draft`. No removals or behavioral reversals occur. Patch-level increment is therefore the correct signal for this draft line.

</div>
</div>

</div>

<div data-dn-version="0.2.3-draft" data-dn-record="meta" id="adro-0-2-3-draft">

## ADR-O 0.2.3-draft

<div data-dn-section="iteration-character">

### Iteration character

<div data-dn-passage="iteration-character-overview-023" data-dn-record="adl" data-dn-adl-refs="0021">

This iteration is a small, non-destructive vocabulary extension focused on social-role topology for decision records. The 0.2.2-draft substrate remains intact; no classes are removed or repurposed, no cardinalities are tightened, and no existing triples are invalidated. The change applies ADR-0021 by adding four `DecisionRecord` role predicates (`authoredBy`, `decidedBy`, `consulted`, `informed`) and by clarifying how these role edges coexist with the existing metadata predicate `dcterms:creator`. *(→ [ADR-0021](/ADL/ADR-0021-social-role-predicates.md))*

</div>
</div>

<div data-dn-section="strong">

### Strong, well-articulated decisions

<div data-dn-passage="social-role-predicates-added-023" data-dn-record="adl" data-dn-adl-refs="0021" data-dn-roadmap="H2.1">

**`adr-o:authoredBy`, `adr-o:decidedBy`, `adr-o:consulted`, and `adr-o:informed` added as object properties on `DecisionRecord`.** All four are declared as `owl:ObjectProperty` with `rdfs:domain adr-o:DecisionRecord` and `rdfs:range prov:Agent`, making social-role accountability directly queryable in graph topology. This closes the previously deferred MADR-style social-role gap and operationalizes the roadmap's Horizon 2.1 "Social Graph (RACI)" item. *(→ [ADR-0021](/ADL/ADR-0021-social-role-predicates.md))*

</div>

<div data-dn-passage="raci-nuance-record-vs-execution-023" data-dn-record="adl" data-dn-adl-refs="0021">

**Explicit RACI mapping with a constrained meaning of "Responsible".** The mapping is documented as `authoredBy` = Responsible, `decidedBy` = Accountable, `consulted` = Consulted, `informed` = Informed. Crucially, "Responsible" is constrained to **record responsibility** (authorship/accountability for the ADR artifact), and is not asserted as execution ownership for implementation work. This avoids semantic drift while preserving domain-specific naming. *(→ [ADR-0021](/ADL/ADR-0021-social-role-predicates.md))*

</div>
</div>

<div data-dn-section="softer">

### Softer defaults — "simplicity first, easier to add than to refactor"

<div data-dn-passage="creator-scope-note-coexistence-023" data-dn-record="adl" data-dn-adl-refs="0021,0015">

**`dcterms:creator` remains canonical metadata, now explicitly coexisting with social-role predicates.** The scope note is tightened to state that `dcterms:creator` remains the compatibility-oriented metadata predicate, while `authoredBy`/`decidedBy`/`consulted`/`informed` are the graph-native role edges for traversable social topology. This is an additive clarification, not a replacement of ADR-0015 semantics. *(→ [ADR-0021](/ADL/ADR-0021-social-role-predicates.md); baseline [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md))* *(Expansion semantics and conflict rule formalized by [ADR-0022](/ADL/ADR-0022-creator-authoredby-coexistence.md).)*

</div>
</div>

<div data-dn-section="deferrals">

### Explicit deferrals — decisions to not decide right now

<div data-dn-passage="deferral-lifted-social-roles-023" data-dn-record="adl" data-dn-adl-refs="0021">

**The MADR-style social-role deferral is now lifted.** Earlier design-note passages that deferred decider/consulted/informed role predicates are now resolved by ADR-0021 and implemented in 0.2.3-draft.

</div>
</div>

<div data-dn-section="still-holds">

### What 0.2.2-draft decisions still hold

<div data-dn-passage="still-holds-list-023" data-dn-record="meta">

All 0.2.2-draft decisions remain in force, including amendment topology (`amends`/`amendedBy`), unchanged status scheme, and the broader 0.2.x atom-first architecture. The 0.2.3 additions are orthogonal social-role predicates and documentation alignment.

</div>
</div>

<div data-dn-section="reversed">

### What 0.2.2-draft decisions are reversed

<div data-dn-passage="reversed-none-023" data-dn-record="meta">

None.

</div>
</div>

<div data-dn-section="versioning-note">

### Versioning note

<div data-dn-passage="versioning-note-023" data-dn-record="adl" data-dn-adl-refs="0021">

The 0.2.2 → 0.2.3 bump is non-destructive and additive: four object properties are introduced (`authoredBy`, `decidedBy`, `consulted`, `informed`), one existing scope note (`dcterms:creator`) is clarified for coexistence semantics, and ontology version metadata is advanced to `0.2.3-draft`. No removals or behavioral reversals occur. Patch-level increment is therefore the correct signal for this draft line.

</div>
</div>

</div>
