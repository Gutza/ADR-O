# ADR-O — Design Notes

This is a sidecar document to [`adr-o.ttl`](adr-o.ttl) acting as a self-contained ADL proxy for fast-paced changes during the first few design iterations. It records the decisions that went into each iteration of the ontology so that they can be triaged — strengthened, weakened, refuted, or promoted to a proper ADR — later. The notes span several topics and mix strong and weak claims, which is exactly why they don't belong in the ADL in this form.

The document is structured as a sequence of `## ADR-O <version>` sections, one per ontology iteration. Each section is an immutable historical record of the decisions that produced that version: later sections may explicitly reverse earlier decisions, but they never edit them in place. To know what is currently true, read the latest section; to know why, read backward.

Within each version section, entries are organized by the strength of the claim, so triage is easy: strong claims go straight into ADRs with minimal rework; softer defaults deserve deliberation before being promoted; explicit deferrals are reminders that a decision was considered and punted, not forgotten.

Every reference to an ADR points to the corresponding file in `ADL/`.

## ADR-O 0.1.0-draft

### Strong, well-articulated decisions

These are directly derivable from an existing ADR, from the survey in ADR-0000, or from the question-class analysis in the outreach article. They should be non-controversial and ready to promote into ADRs with minimal rework.

**Primary class name is `adr-o:DecisionRecord`, not `ArchitectureDecisionRecord`.** Domain-agnostic scope is an accepted decision (ADR-0002). Encoding "architecture" at the IRI level would make non-software uses structurally second-class; the significance filter that "architecture" carries is better captured explicitly in a future iteration than implicitly through class naming.

**Six core relational predicates: `supersedes`, `supersededBy`, `dependsOn`, `enables`, `conflictsWith`, `addresses`.** This vocabulary was identified in ADR-0000 as the fresh terms to mint, and every question class in the outreach article's taxonomy reduces to a traversal over one or more of them. The outreach article's `seeAlso` was not minted: `rdfs:seeAlso` already exists and is sufficient.

**Splitting OntolAD's `affects` into `adr-o:addresses` and `adr-o:affects`.** ADR-0000 identified that OntolAD conflated "what problem does this decision resolve" with "what does this decision touch in the system" under a single `affects` property, and called the split out as a correction. This iteration applies the split: `addresses` is the problem side (range `Concern`); `affects` is the manifestation side (open range). The question-class analysis confirmed this is the predicate that answers both targeted questions and filter-by-system queries.

**`adr-o:index` as an `xsd:integer` separate from `dcterms:identifier`.** The human-readable string ID (`ADR-0042`) is carried by `dcterms:identifier` but does not sort cleanly in SPARQL. Organizational-history queries need a native numeric ordering, which only a dedicated integer property delivers. This was not an obvious need from the ADRs alone — it surfaced during the question-class review.

**Status concept scheme with `Proposed`, `Accepted`, `Deprecated`, `Superseded`, `Rejected`, wired into `DecisionRecord` via a named `adr-o:Status` class and a functional `hasStatus`.** ADR-0000 called out these five values and the SKOS mechanism for expressing them. The top-concept declarations are editorial but consistent with standard SKOS practice. The wiring — `adr-o:Status` defined as `skos:Concept ∩ (skos:inScheme = adr-o:statusScheme)` and used as the range of `adr-o:hasStatus` — is what makes "status values are members of the scheme" a machine-readable fact rather than a prose convention. The equivalent-class definition means domain profiles that extend the scheme with additional statuses automatically inherit `adr-o:Status` membership without any ontology change; no separate subclassing is needed. `adr-o:hasStatus` is declared `owl:FunctionalProperty` (at most one status per record), and the core ontology contains a single `owl:AllDifferent` block over the five status individuals. The `AllDifferent` block is load-bearing: without it, under OWA + no UNA, a record misannotated with two statuses would not be flagged — the reasoner would silently entail `Proposed owl:sameAs Accepted` rather than reporting an inconsistency, quietly polluting every downstream query. With the `AllDifferent`, the same misannotation becomes an OWL-level contradiction that a conformant OWL 2 DL reasoner (HermiT, Pellet, FaCT++) reports loudly. Domain profiles that add new status individuals should extend the `AllDifferent` declaration to preserve this property.

A reasoner caveat worth recording: some lightweight OWL 2 RL implementations — including Python's `owlrl` — do not implement the RL rules (`eq-diff2`, `eq-diff3`) that expand an `owl:AllDifferent` block into pairwise `owl:differentFrom` entailments. Against such a reasoner, the duplicate-status check degrades silently: `Proposed sameAs Accepted` is entailed, but no contradiction is reported. This is an incompleteness in those reasoners, not in the ontology; pre-generating pairwise `owl:differentFrom` assertions as a workaround was explicitly rejected on the grounds that ADR-O should publish spec-correct OWL 2 and let reasoners be as complete as they claim to be.

**`Concern` as an abstract anchor class with no core concept scheme.** The extension mechanism is explicitly established in ADR-0002; this iteration just applies it.

**License: CC BY 4.0.** Directly per ADR-0001. The `dcterms:license` annotation on the ontology IRI is the machine-readable canonical declaration; the repository `LICENSE` file is supplementary.

**Namespace: `https://w3id.org/adr-o#`.** Directly per README and ADR-0000.

**`adr-o:supersedes` as a sub-property of `prov:wasRevisionOf`.** ADR-0000 identified this alignment explicitly: the ADR supersession chain *is* a PROV-O revision chain, and modelling it as a sub-property means PROV-O-aware tooling automatically sees every supersession as a revision without any extra wiring.

### Softer defaults — "simplicity first, easier to add than to refactor"

These are picks made under the heuristic the user reaffirmed when asking for the first iteration. Each is reversible in a follow-up ADR; none of them forecloses a richer future.

**Single Turtle file under `ontology/`, no `examples/` or `shapes/` directories yet.** The ontology is small enough to fit in one file and one serialization; multiple files would introduce premature structure. JSON-LD and RDF/XML serializations can be generated from the canonical Turtle later.

**No `owl:imports` for PROV-O / SKOS / DC Terms.** The ontology uses these vocabularies purely as namespace prefixes. Pulling in PROV-O's full axiom set would burden every downstream reasoner for little gain at this stage; downstream tools that want the full axioms can opt in. This choice can be revisited if reasoning over the alignment (e.g. `supersedes ⊑ wasRevisionOf`) becomes important in practice.

**Hash IRIs (`https://w3id.org/adr-o#Foo`) rather than slash IRIs.** Standard for small self-contained vocabularies and what the README already committed to. Slash IRIs would allow per-term content negotiation but require more hosting infrastructure to be useful.

**Nygard body sections (`context`, `decision`, `consequences`, `rationale`) as `rdf:langString` datatype properties, not first-class classes.** Without them, instances cannot represent even a textbook ADR. Promoting any of them to a first-class class (OntolAD-style `Rationale`, for example) is a non-breaking extension: the existing datatype predicates can coexist with object-property forms. The trade-off is that trade-off narratives, forces, and consequence items cannot currently be individually queried — only read as prose.

**`consequences` as a single property, not split positive / negative.** MADR splits this; Nygard does not; our own ADRs use a single "Consequences" section with sub-headings. A later iteration may introduce `adr-o:positiveConsequences` / `adr-o:negativeConsequences` as refinements without breaking the current property.

**`conflictsWith` as an `owl:SymmetricProperty`, not a directed Kruchten-style pair.** ADR-0000 flagged this as an open question. Symmetric is strictly simpler and matches the common "A and B are in tension" use case. Directed `prevents` / `isPreventedBy` can be added later as sub-properties without breaking existing `conflictsWith` triples.

**`wasRejectedBecause` as a datatype property (`rdf:langString`), not an object property.** Keeps first-iteration authoring simple — a string reason on an `Alternative`. A later revision may introduce a parallel object property pointing to a structured `Rationale` node without breaking this one.

**`adr-o:affects` with open range (`rdfs:Resource`).** Deliberately unconstrained in the core. Domain profiles are expected to tighten this — either by declaring the range of a sub-property or through SHACL shapes — once a domain's notion of "affected element" is settled. The trade-off is that an ADR-O instance placed into a core-only graph is not protected against nonsensical `affects` targets.

**`dcterms:date` as the canonical date predicate, over `dcterms:issued` and `prov:generatedAtTime`.** All three are semantically close enough for the ADR use case; `dcterms:date` is the most widely recognised and the least semantically freighted. The scope note in the ontology calls this out so domain profiles don't pick a different predicate inconsistently.

**`dcterms:creator` as the canonical authorship predicate.** Explicitly confirmed via clarifying question. Accepts either a literal string or an IRI pointing to a structured agent. The IRI form is preferred for graph-traversable authorship queries but not mandated; `prov:wasAttributedTo` is permitted when PROV-O tooling is already in play but is not the canonical form.

**Version annotations: `owl:versionIRI <https://w3id.org/adr-o/0.1.0>` and `owl:versionInfo "0.1.0-draft"`.** A conservative first-public-draft versioning. The `-draft` suffix is a signal that breaking changes are expected before 0.1.0 is tagged as released.

### Explicit deferrals — decisions to not decide right now

These are items that were considered during this iteration and deliberately punted. Listing them here is the antidote to silent forgetting: whoever revisits knows it was seen.

**`adr-o:hasType` / the "Type" field our own ADRs use.** Deferred by explicit user decision after the question-class review. Our own ADRs carry a free-form `Type` field ("Inception record", "Core design", "Project governance"); Tyree–Akerman calls this "Group". Nygard-style templates do not define an equivalent field. MADR does address grouping in [Support Categories](https://adr.github.io/madr/decisions/0010-support-categories.html), but the chosen pattern is repository organisation (e.g. one level of subfolders with local IDs), not a dedicated `Type`/`Group` metadata line in the ADR itself comparable to Tyree–Akerman or our front matter. The SKOS-extension pattern would make it trivial to add (`adr-o:hasType → skos:Concept`, no core scheme, same shape as `addresses`/`Concern`), but no axioms justifying a core predicate have been identified. This is an explicit decision to not decide — the meta-decision being that domain-specific categorisation doesn't yet belong in the core namespace.

**First-class `Force` / `Constraint` / `Rationale` / `Consequence` classes.** ADR-0000 flagged `Force` as an open question; Tyree–Akerman has "Constraints" and "Assumptions"; the outreach article's "system constraints" and "unused capacity" question classes would be answered naturally by first-class constraint nodes. None of these are modelled in this iteration; they all become datatype prose for now. The trade-off is that the two constraint-shaped question classes in the outreach article are not yet fully answerable — they require text retrieval against prose rather than graph traversal. Addressing this is the natural scope of a follow-up ADR.

**Kruchten-style directed `prevents` / `isPreventedBy` alongside symmetric `conflictsWith`.** Flagged as an open question in ADR-0000. Added later as sub-properties of `conflictsWith`, non-breaking.

**MADR-style decider / consulted / informed roles.** MADR distinguishes these; ADR-O does not in this iteration. `dcterms:creator` is the only authorship predicate. If a team needs the distinction, a domain profile can mint the role properties.

**`significance` / the "architectural" filter from ADR-0002.** Open question in ADR-0002. Not addressed here; the domain-agnostic core does not currently constrain what counts as record-worthy.

**SHACL shapes graph for validation.** Open question in ADR-0000. Not included. Implementers wanting cardinality or range constraints must encode them themselves for now.

**Additional serializations (JSON-LD, RDF/XML).** Turtle is canonical; others will be generated mechanically from it later.

**Domain profile registry.** Open question in ADR-0002. No registry, no central discovery mechanism, no informal SKOS collection in the core repo. Profiles are expected to publish in their own namespaces and be discoverable via whatever means the linked-data ecosystem already provides.

### Notes on reuse that aren't quite decisions

**Scope notes on reused predicates.** The ontology file carries `skos:scopeNote` annotations for `dcterms:title`, `dcterms:identifier`, `dcterms:date`, `dcterms:created`, `dcterms:modified`, `dcterms:creator`, `dcterms:references`, `dcterms:license`, `prov:wasAttributedTo`, and `prov:wasRevisionOf`, stating how ADR-O recommends each be used. This is editorial: no new terms are minted, no semantics are changed. It exists so implementers have one place to look rather than having to infer the intended conventions from examples.

**Versioning discipline.** No ADR has yet discussed the ontology's versioning strategy (semantic versioning? release cadence? deprecation policy?). This iteration uses `0.1.0-draft` informally. A dedicated ADR on versioning would be welcome before anyone depends on ADR-O for production tooling.

## ADR-O 0.2.0-draft

This iteration is a destructive shape change: Nygard prose-on-record is replaced by an atom-first graph. There is no backward-compatibility shim. The ontology is at this stage a technical preview rather than a candidate for production use; sledgehammers from outside reviewers are explicitly invited.

### Architecture in one paragraph

A `DecisionRecord` carries an option pool (`hasAlternative`), an explicit chosen alternative (`chosenAlternative`), and three ordered `rdf:List`s of reified facts (`hasContext`, `hasDeliberation`, `hasOutcome`). Each fact (`ContextFact`, `DeliberationFact`, `OutcomeFact`) places exactly one reusable `Consideration` into a role; deliberation and outcome facts also carry a valence drawn from a small per-class enum. The same `Consideration` IRI may be placed in arbitrarily many facts across arbitrarily many records — that reuse, the YADR-style argument-identity property, is the central design property of this iteration and is achieved without RDF-star and without quads.

### Design assumption: the KG lives under tooling

ADR-O graphs are authoritative substrate, never meant to be user surface (even when the user is a highly competent software developer). The expected deployment includes several layers between RDF and human authors / readers: LLMs that draft, summarise, and answer natural-language questions; APIs that mediate programmatic access with shape enforcement; MCP servers that expose structured tools to AI agents; GUIs for authoring, visualisation, and navigation. The ontology is designed for that layered world, not for a hypothetical use case in which someone hand-writes Turtle and reads the result without further mediation. Naming this assumption explicitly matters because several 0.2.0-draft decisions follow from it rather than from RDF / OWL principles alone, and future iterations should be able to rely on the same stance for design intentionality consistency.

Decisions in this iteration that lean on the assumption:

- **Strict graph (no Nygard literals).** We expect the tooling to materialise Markdown for human readers from the structured graph; the KG does not need to carry (and maintain) redundant human-facing prose copies.
- **SHACL deferred at technical-preview status.** Integrity rules (list-element typing, `chosenAlternative ∈ hasAlternative`, `Accepted ⇒ chosenAlternative present`, per-fact-class field cardinalities, valence-enum membership per fact class) can live in the API / MCP / GUI layer in the meantime, if needed at all. The OWL ontology stays clean of conditional cardinality rules; the planned SHACL companion will publish the same constraints declaratively when it ships.
- **`rdf:List` ordering accepted despite OWL DL awkwardness.** List traversal is a tooling concern; raw OWL DL reasoning over list membership is not the primary access path. SPARQL handles lists cleanly, and the planned SHACL companion will enforce list-element typing.
- **Authoring conventions documented in scope notes rather than axiomatised** (e.g. `onAlternative` required on `DeliberationFact`, optional on `OutcomeFact`, not used on `ContextFact`). The API / MCP / GUI layer can enforce these conventions long before SHACL ships. Even in LLM-heavy authoring contexts, structured tool boundaries make discipline tractable.
- **IRI convention for `Consideration` reuse documented but not formalised.** Tooling treats both record-local and org-shared patterns uniformly; the ontology imposes no axiom.
- **`addresses` on both `DecisionRecord` and `Consideration`, with optional editorial rollup at the record level.** A rollup whose authoritative source is consideration-level only makes sense in a world where some layer (tooling, query rewriter, materialised view) reconciles the two when needed.

Forward-looking implication: when choosing between two ontology shapes, the one that pushes incidental complexity into the tooling layer in exchange for a cleaner KG is the preferred trade-off — *unless* the complexity in question is something the tooling layer cannot reasonably reconstruct from the graph alone. That cut-off (what the tooling can reconstruct vs what must be in the graph) is a load-bearing design boundary that future ADRs should make explicit when the question recurs.

Caveat: this assumption fails for users who consume ADR-O graphs raw, with no tooling at all. They will see a more skeletal model than the layered presentation suggests — text payloads on `Consideration` nodes rather than Nygard-shaped prose, integrity rules absent rather than enforced, ordering carried in `rdf:List`s that some lightweight readers handle awkwardly, the canonical/editorial split on `addresses` not reconciled. That is the deliberate trade-off, not an oversight.

### Strong, well-articulated decisions

These are the load-bearing architectural commitments of 0.2.0-draft. Each was reached after a structured comparison against the alternatives in IDEAS and the principles in ADR-0000.

**Atom-level reification, not section-level (no Nygard divs).** The two reasonable shapes for first-class structure in an ADR graph are: section-level (treat each Nygard body slot as its own resource) and atom-level (introduce reusable claim atoms placed into roles). Section-level reification was rejected as "reinventing HTML divs" — it provides containers without identity, and identity is what the YADR anchor/alias mechanism shows is the genuinely valuable structural property. Atom-level reification, by contrast, captures the claim that the argument made for Option A during evaluation is *the same thing* as the positive consequence noted afterward, not a copy of it. A first-class atom class is the only way to encode that identity in RDF without resorting to text similarity.

**Atom class named `Consideration`.** Rejected names included `Force` (overloaded with Nygard-style "forces" prose and physically connoted), `Argument` (implies contestation, fits poorly when the atom is genuinely neutral), and `Claim` (less idiomatic in the ADR literature). `Consideration` reads well across positive, negative, and neutral cases and matches how reviewers naturally describe the third bucket — "a consideration we noted but that didn't push the choice."

**Strict graph: no Nygard body literals.** The four datatype properties `context`, `decision`, `consequences`, and `rationale` are removed. Tooling materialises Markdown for human readers from the graph; the ontology authoritatively represents structure, not prose. This forces the question "what was actually said?" to resolve to a `Consideration` IRI rather than a search inside a literal, which is what makes the YADR identity property usable.

**Reified facts via classes, not RDF-star.** Binary edges between `DecisionRecord` and `Consideration` are insufficient: the same atom can occupy multiple roles in one record (framing vs deliberation vs outcome) and can carry orthogonal annotations (valence, ordering, eventually provenance). Two implementations achieve this: per-statement annotation (RDF-star or named graphs / quads) or reification via dedicated link classes. Reified link classes were chosen because they keep the spec inside plain RDF / OWL 2 — no RDF-star toolchain dependency, no quad assumption — and because they place the role information on a node that can itself be queried, ordered, and constrained.

**Three reified link classes: `ContextFact`, `DeliberationFact`, `OutcomeFact`.** A single fact class would have to carry phase as a property and accommodate optional `onAlternative` and valence with branching cardinality rules; the three-class split makes phase a class fact and allows per-class field shapes without conditional constraints. `ContextFact` earns its own class because pure framing statements ("we run three Python microservices") are neither comparisons of options nor post-decision realities; collapsing them into `DeliberationFact` would require relaxing required fields and confuse query intent. The three classes are parallel in shape and naming for tooling consistency.

**`rdf:List` ordering at the record level.** Each record carries up to three lists (`hasContext`, `hasDeliberation`, `hasOutcome`) that order the facts in the author's intended sequence. Ordering lives on the placement (the list membership), not on the `Consideration`, so the same atom may appear at different positions in different records or different roles. The trade-off is well-known: `rdf:List` is awkward in OWL DL reasoning over membership, but SPARQL handles it cleanly and the planned SHACL companion will encode list-element typing.

**Per-class valence enums with single neutral (KISS).** `DeliberationFact` carries `Supports`, `Against`, or `Neutral`; `OutcomeFact` carries `Benefit`, `AcceptedCost`, `Risk`, or `FollowUp`. Splitting `Neutral` further (orthogonal vs in-scope-but-non-decisive) was considered and deferred — the single bucket is enough to record "this was noted but didn't push the choice", which is the practically observed need. Each enum is a SKOS concept scheme (`adr-o:deliberationValenceScheme`, `adr-o:outcomeValenceScheme`) wired into a named class (`DeliberationValence`, `OutcomeValence`) via the same `skos:Concept ∩ (skos:inScheme = …)` pattern used by `adr-o:Status`. Each scheme has its own `owl:AllDifferent` block over its members, with the same caveat about lightweight OWL 2 RL reasoners that do not implement `eq-diff2` / `eq-diff3` as already documented for the status `AllDifferent` in 0.1.0-draft.

**`chosenAlternative` as an `owl:FunctionalProperty` with cardinality 0..1.** The functional declaration enforces "at most one chosen alternative per record" at the OWL level. Cardinality 0 is permitted to accommodate `Proposed` records that have not yet picked an option; the integrity rule "if `hasStatus = Accepted` then `chosenAlternative` is present" is an SHACL concern and is deferred. Multi-winner decisions ("use both X and Y") are modelled as a single composite `Alternative` rather than by relaxing cardinality, which keeps the predicate cleanly queryable as a single edge.

**`addresses` allowed on both `DecisionRecord` and `Consideration`.** The canonical, fine-grained use is on `Consideration` (a specific atom is about a specific concern). The use on `DecisionRecord` is a permitted editorial rollup that human and tooling layers may apply for discovery convenience — "this whole record is broadly about latency" — and is not authoritative when both are present. This dual home leaves the future GADR-style `DecisionTemplate` design fully open: templates can either inherit `addresses` directly via a future shared superclass, or rely entirely on consideration-level concern tagging, without breaking either current use. The OWL declaration drops the previous `rdfs:domain DecisionRecord` restriction; the scope note spells out which use is canonical.

**Two valence properties (`deliberationValence`, `outcomeValence`), not one.** A single `valence` predicate would either need a permissive union range or push range enforcement entirely into SHACL. Two properties make each ontology declaration self-documenting (clean `rdfs:range`) at the cost of one extra IRI; given the disjointness of the two enums by design, the two-property shape is more honest about what is actually being said.

**Destruction without compatibility shim.** The four Nygard datatype properties (`context`, `decision`, `consequences`, `rationale`) and `wasRejectedBecause` are removed outright, not deprecated. `wasRejectedBecause` is now expressible as a `DeliberationFact` on the rejected `Alternative` with `Against` valence; keeping it would create two ways to say the same thing. The 0.1.0-draft ontology was never published or used outside the author's working copy, so no migration is required; the absence of compatibility scaffolding is a deliberate signal that 0.2.0 is a clean break.

### Softer defaults — "simplicity first, easier to add than to refactor"

These are picks made under the same simplicity-first heuristic that 0.1.0 used. Each is reversible without breaking existing 0.2.0-draft graphs.

**`onAlternative` required on `DeliberationFact`, optional on `OutcomeFact`, not used on `ContextFact`.** A deliberation always weighs an option, so requiring the link aligns with intent and lets the planned SHACL companion catch malformed authoring at validation time. An outcome usually pertains to the chosen alternative (and so the link is left implicit), but counterfactual outcomes about rejected options ("if we had gone with B, this risk would have materialised") are valuable enough to allow explicit linking. Context is by construction not about any specific option. These are authoring conventions documented in scope notes; SHACL enforcement is deferred.

**Composite multi-winner decisions modelled as a single `Alternative`, not by relaxing `chosenAlternative` cardinality.** Trades a small amount of authoring friction (the author must mint a combined option such as `:alt-X-and-Y`) for keeping `chosenAlternative` queryable as a single functional edge. Reversible if the trade-off proves wrong in practice.

**`consideration` and `onAlternative` declared `owl:FunctionalProperty`.** A fact carries exactly one consideration and at most one alternative. This catches malformed facts at OWL-DL reasoning time without requiring SHACL.

**No SHACL shapes shipped, even though the new architecture relies on rules SHACL would express well.** The 0.2.0-draft is a technical preview; a SHACL companion would harden the integrity story (list-element typing, `chosenAlternative ∈ hasAlternative`, `Accepted ⇒ chosenAlternative present`, per-fact-class field cardinalities, valence-enum membership), but shipping it now would amount to over-investment in a shape the community has not yet had a chance to challenge. Listed below in the deferrals.

**Version annotations: `owl:versionIRI <https://w3id.org/adr-o/0.2.0>` and `owl:versionInfo "0.2.0-draft"`.** The major-version bump from 0.1 to 0.2 reflects the destructive break; the `-draft` suffix is preserved to signal that further breaking churn is expected before 0.2.0 is tagged as released.

### IRI convention for `Consideration` reuse

Reusing the same `Consideration` IRI across multiple records is the YADR-style identity property and the central design payoff of atom-first reification. The ontology imposes no axiom about where these IRIs live; the convention below is editorial, not normative. Tools (renderers, MCP servers, validation layers) are expected to treat both patterns uniformly.

- **Record-local default.** An atom that emerges in a single decision lives as a fragment under that record, e.g. `<https://example.org/adl/ADR-0042#cons-ecosystem-maturity>`. Easy to author, no governance overhead, works for atoms that may never be reused.
- **Org-shared promotion.** An atom deliberately reused across records, or one underpinning a future GADR-style decision template, lives under a separate organisational namespace, e.g. `<https://example.org/considerations/ecosystem-maturity>`. Cross-record queries about this atom no longer have to traverse a record-specific IRI.
- **Bridge between the two.** When promoting a record-local atom to org-shared, the recommended bridge is `owl:sameAs` from the shared IRI to the local one. Existing references in the originating record continue to resolve; new records reference the shared IRI directly.

The convention does not add ontology axioms. Future work may formalise the scope distinction (e.g. an `adr-o:scope` predicate) without breaking existing IRIs.

### Explicit deferrals — decisions to not decide right now

These are items considered during this iteration and deliberately punted. Listing them is the antidote to silent forgetting.

**Goals / `to-achieve` (the YADR slot).** No first-class home for the "intended outcomes the choice is meant to produce" notion. Two upgrade paths are noted: a `Goal` subclass of `Consideration` with `intendsToAchieve` semantics, used in `ContextFact` or `DeliberationFact`; or a dedicated predicate `intendsToAchieve` from `DecisionRecord` directly to a `Goal` node. Authors who need this slot in 0.2.0-draft are recommended to express it as `ContextFact`s ("we want low p99 latency") or as positive-valence `DeliberationFact`s on the chosen option, accepting that the goal/observation distinction is currently lost.

**SHACL shapes graph for validation.** Promoted from "deferred without commitment" (0.1.0) to "deferred at technical-preview status" (0.2.0). The new architecture relies more heavily on SHACL than the 0.1.0 design did — list-element typing, chosen-alternative integrity, per-fact-class field shapes, valence-enum membership per fact class. None of these are shipped. A future iteration is expected to publish a shapes graph alongside the OWL.

**GADR-style `DecisionTemplate` class.** A template is a record-shaped node carrying `hasAlternative` and `DeliberationFact`s but no `chosenAlternative` and no `OutcomeFact`s. The 0.2.0-draft architecture is shape-compatible with this future class and even invites it: `addresses` may appear on both `DecisionRecord` and `Consideration`, which preserves room for a template superclass or consideration-only tagging without breaking either pattern. Explicitly deferred until the templating mechanism is needed in practice — the class is not declared in 0.2.0-draft.

**`amends` and `clarifies` as typed link predicates** (and matching `Amended`, `Clarified` status individuals). Mentioned in IDEAS as gaps in 0.1.0; still gaps in 0.2.0. Adding them later as further relational predicates and SKOS concepts in the status scheme is non-breaking.

**Fitness functions / confirmation predicates.** MADR's "Confirmation" slot — how will you know this decision is being followed? — has no first-class predicate in 0.2.0. Likely shape: a property from `DecisionRecord` to a test, CI rule, or SHACL shape. Deferred.

**Splitting `Neutral` into orthogonal vs in-scope-but-non-decisive.** Considered and KISS-rejected for this iteration. Adding a second neutral concept to `deliberationValenceScheme` later is non-breaking.

**Kruchten-style directed `prevents` / `isPreventedBy`** alongside symmetric `conflictsWith`. Still deferred from 0.1.0 reasoning.

**MADR-style decider / consulted / informed roles.** Still deferred from 0.1.0.

**`significance` / "architectural" filter.** Still deferred from 0.1.0.

**Additional serializations (JSON-LD, RDF/XML).** Still deferred from 0.1.0.

**Domain profile registry.** Still deferred from 0.1.0.

**`adr-o:hasType` / Group categorization.** Still deferred from 0.1.0.

### What 0.1.0-draft decisions still hold

This iteration changes the body shape of `DecisionRecord`, but a substantial portion of 0.1.0 is preserved. Each item below is carried forward with no semantic change.

- `adr-o:DecisionRecord` as the central class name (not `ArchitectureDecisionRecord`); domain-agnostic scope.
- Namespace `https://w3id.org/adr-o#`; hash IRIs.
- Single Turtle file under `ontology/`; no `examples/` or `shapes/` directories yet.
- No `owl:imports` for PROV-O / SKOS / DC Terms.
- License CC BY 4.0.
- Status concept scheme, the five status individuals (`Proposed`, `Accepted`, `Deprecated`, `Superseded`, `Rejected`), `adr-o:Status` wired as `skos:Concept ∩ (skos:inScheme = adr-o:statusScheme)`, `hasStatus` functional, `owl:AllDifferent` over the five statuses with the same RL-reasoner caveat.
- `adr-o:index` as `xsd:integer` separate from `dcterms:identifier`.
- `adr-o:supersedes` as a sub-property of `prov:wasRevisionOf`.
- The five surviving relational predicates from the 0.1.0 "six core" set: `supersedes`, `supersededBy`, `dependsOn`, `enables`, `conflictsWith`. (`addresses` survives as well but is refined: canonical use on `Consideration`, optional editorial rollup on `DecisionRecord`; see "Strong, well-articulated decisions" above.)
- The `addresses`/`affects` split (problem side vs manifestation side); `affects` unchanged (open `rdfs:Resource` range).
- `Concern` as an abstract anchor class with no core concept scheme.
- `dcterms:date` as the canonical date predicate; `dcterms:creator` as the canonical authorship predicate.
- `conflictsWith` as `owl:SymmetricProperty`, not directed Kruchten pair.
- Editorial `skos:scopeNote`s on reused predicates (`dcterms:title`, `dcterms:identifier`, `dcterms:date`, `dcterms:created`, `dcterms:modified`, `dcterms:creator`, `dcterms:references`, `dcterms:license`, `prov:wasAttributedTo`, `prov:wasRevisionOf`).

### What 0.1.0-draft decisions are reversed

- **The four Nygard datatype properties (`context`, `decision`, `consequences`, `rationale`) are removed outright.** The 0.1.0 reasoning was "without them, instances cannot represent even a textbook ADR"; the 0.2.0 architecture replaces "represent prose on the record" with "represent structure on the record and let tooling materialise prose," so the original justification no longer applies.
- **`wasRejectedBecause` as a datatype property is removed.** Its function is now served by a `DeliberationFact` on the rejected `Alternative` with `Against` valence, which also gives the rejection rationale a reusable IRI rather than an opaque literal.
- **`consequences` as a single property** (the deferred-split decision in 0.1.0) is moot: there is no `consequences` property at all in 0.2.0. Its role is replaced by `OutcomeFact` with the four-valued outcome valence enum, which subsumes both the positive/negative split that MADR documents and the YADR-style `accepting-that` distinction (now `AcceptedCost`).
- **The deferral of "first-class `Force` / `Constraint` / `Rationale` / `Consequence` classes" is resolved (in part).** A single `Consideration` class together with three `*Fact` link classes covers the practical need that motivated all four candidate classes; the four-class proliferation is no longer the upgrade path.

### Versioning note

The jump from `0.1.0-draft` to `0.2.0-draft` is a destructive shape change: every triple using one of the removed predicates becomes invalid, and the new `Consideration` / `*Fact` infrastructure has no autogenerable correspondence to the old prose properties. Under any sane semantic-versioning interpretation this is a major break. Two facts make the cost negligible: the 0.1.0-draft ontology was never published or used outside the author's working copy, and the `-draft` suffix on both versions signals exactly this kind of pre-release churn.

The convention adopted here — promoting `DESIGN-NOTES.md` itself to a versioned document with one section per ontology iteration — is the project's own answer to the question "where do we record the rationale for these large iterations without writing a flurry of micro-ADRs in `ADL/`?" Each version section is immutable history; later sections reverse earlier decisions explicitly rather than editing them in place. That property, applied to the design notes themselves, is the meta-application of the principle ADR-O exists to defend.
