# ADR-O 0.1.0-draft — Design Notes

This is a sidecar document to [`adr-o.ttl`](adr-o.ttl), not an ADR. It records the decisions that went into the first iteration of the ontology so that they can be triaged — strengthened, weakened, refuted, or promoted to a proper ADR — later. The notes span several topics and mix strong and weak claims, which is exactly why they don't belong in the ADL in this form.

Entries are organized by the strength of the claim, so triage is easy: strong claims go straight into ADRs with minimal rework; softer defaults deserve deliberation before being promoted; explicit deferrals are reminders that a decision was considered and punted, not forgotten.

Every reference to an ADR points to the corresponding file in `ADL/`.

---

## Strong, well-articulated decisions

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

---

## Softer defaults — "simplicity first, easier to add than to refactor"

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

---

## Explicit deferrals — decisions to not decide right now

These are items that were considered during this iteration and deliberately punted. Listing them here is the antidote to silent forgetting: whoever revisits knows it was seen.

**`adr-o:hasType` / the "Type" field our own ADRs use.** Deferred by explicit user decision after the question-class review. Our own ADRs carry a free-form `Type` field ("Inception record", "Core design", "Project governance"); Tyree–Akerman calls this "Group". Nygard-style templates do not define an equivalent field. MADR does address grouping in [Support Categories](https://adr.github.io/madr/decisions/0010-support-categories.html), but the chosen pattern is repository organisation (e.g. one level of subfolders with local IDs), not a dedicated `Type`/`Group` metadata line in the ADR itself comparable to Tyree–Akerman or our front matter. The SKOS-extension pattern would make it trivial to add (`adr-o:hasType → skos:Concept`, no core scheme, same shape as `addresses`/`Concern`), but no axioms justifying a core predicate have been identified. This is an explicit decision to not decide — the meta-decision being that domain-specific categorisation doesn't yet belong in the core namespace.

**First-class `Force` / `Constraint` / `Rationale` / `Consequence` classes.** ADR-0000 flagged `Force` as an open question; Tyree–Akerman has "Constraints" and "Assumptions"; the outreach article's "system constraints" and "unused capacity" question classes would be answered naturally by first-class constraint nodes. None of these are modelled in this iteration; they all become datatype prose for now. The trade-off is that the two constraint-shaped question classes in the outreach article are not yet fully answerable — they require text retrieval against prose rather than graph traversal. Addressing this is the natural scope of a follow-up ADR.

**Kruchten-style directed `prevents` / `isPreventedBy` alongside symmetric `conflictsWith`.** Flagged as an open question in ADR-0000. Added later as sub-properties of `conflictsWith`, non-breaking.

**MADR-style decider / consulted / informed roles.** MADR distinguishes these; ADR-O does not in this iteration. `dcterms:creator` is the only authorship predicate. If a team needs the distinction, a domain profile can mint the role properties.

**`significance` / the "architectural" filter from ADR-0002.** Open question in ADR-0002. Not addressed here; the domain-agnostic core does not currently constrain what counts as record-worthy.

**SHACL shapes graph for validation.** Open question in ADR-0000. Not included. Implementers wanting cardinality or range constraints must encode them themselves for now.

**Additional serializations (JSON-LD, RDF/XML).** Turtle is canonical; others will be generated mechanically from it later.

**Domain profile registry.** Open question in ADR-0002. No registry, no central discovery mechanism, no informal SKOS collection in the core repo. Profiles are expected to publish in their own namespaces and be discoverable via whatever means the linked-data ecosystem already provides.

---

## Notes on reuse that aren't quite decisions

**Scope notes on reused predicates.** The ontology file carries `skos:scopeNote` annotations for `dcterms:title`, `dcterms:identifier`, `dcterms:date`, `dcterms:created`, `dcterms:modified`, `dcterms:creator`, `dcterms:references`, `dcterms:license`, `prov:wasAttributedTo`, and `prov:wasRevisionOf`, stating how ADR-O recommends each be used. This is editorial: no new terms are minted, no semantics are changed. It exists so implementers have one place to look rather than having to infer the intended conventions from examples.

**Versioning discipline.** No ADR has yet discussed the ontology's versioning strategy (semantic versioning? release cadence? deprecation policy?). This iteration uses `0.1.0-draft` informally. A dedicated ADR on versioning would be welcome before anyone depends on ADR-O for production tooling.
