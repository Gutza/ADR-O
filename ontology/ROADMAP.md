# ADR-O Strategic Gap Analysis & Roadmap

This document is the living strategic roadmap for ADR-O. It was originally drafted before any real-world modeling had been attempted with the ontology; it has since been aggressively revised in light of the **Dogfood Report** (see `DOGFOOD-REPORT.md`), which documents the conversion of ADR-0001 and ADR-0005 into RDF graphs using ADR-O 0.2.1. Where the original roadmap reasoned from template audits and structural review alone, this revision is grounded in observed friction during actual modeling.

The single most important finding from that exercise is the one that reframes the entire roadmap: **ADR-O 0.2.1 is a high-performance Decision Index. It is not yet a Knowledge System.** It answers "what was decided and why" as a set of weighted claims with reliable IRI identity. It cannot yet answer "what was the logical structure of the argument," "what criteria were the alternatives evaluated against," or "who made this call." Closing that gap — moving from Index to Knowledge System — is what the horizons below are organized around.

## 1. SWOT Analysis

| **STRENGTHS** (Current Core) | **WEAKNESSES** (Current Gaps) |
| :--- | :--- |
| **Atom-First Identity:** Solves the YADR identity problem via `Consideration` nodes reused across records and roles with stable IRI identity. | **Social Blindness:** No model for the "Human Graph" — who decided, who was consulted, who was informed. `dcterms:creator` captures authorship only. |
| **Option Pool Integrity:** `hasAlternative` + `chosenAlternative` + valenced `DeliberationFact`s give machine-traversable decision outcomes without text parsing. | **Criteria Blindness:** No model for structured comparative evaluation. Comparison tables (ADR-0001's license matrix) must be collapsed into prose claims, destroying the Feature-Graph. |
| **Domain Agnostic:** Portable across any professional domain (`ADR-0002`). | **Epistemic Narrowness:** Models problems (`Concern`) but not positive targets (`Goal`). Teleological arguments (ADR-0005's "log all *in order to* enable X") lose their spine. |
| **Standard Alignment:** Native use of PROV-O, SKOS, and DC Terms; no heavyweight `owl:imports`. | **Argumentative Flatness:** Considerations are a bucket of weighted claims, not a chain of logic. "Therefore" links, prerequisite relationships between premises, and syllogistic structure are invisible to SPARQL. |
| **Tooling-First Philosophy:** Clean KG substrate designed for AI/MCP mediation per ADR-0004. | **Temporal Staticity:** No mechanism for "Revisit" triggers or decay conditions. Zombie Decisions — records that stay `Accepted` long after their assumptions expired — are undetectable. |

| **OPPORTUNITIES** (Future Growth) | **THREATS** (Risks) |
| :--- | :--- |
| **Criteria / Evaluation modeling:** Bridging the Matrix gap would make ADR-O useful for structured decision analysis, not just narrative capture. | **Tooling Dependency:** The "substrate" model fails if the tooling layer never materialises. The ontology is designed for a layered world; without the layers it reads as skeletal. |
| **Pattern Libraries (GADR):** A `DecisionTemplate` class would enable cross-project reuse and organizational governance queries impossible in any existing ADR format. | **Noise Volume:** "Log all" (`ADR-0005`) may create a data swamp without `adr-o:significance` filters. The risk scales with adoption. |
| **Real-Time Instrumentation:** The ADR-0005 ambient-transcription design is validated by the dogfood exercise — the 0.2.1 model already handles real-time records without friction. | **Complexity Drift:** Each Horizon 2/3 item adds predicates. The Index/Knowledge System framing is the guard against over-engineering: every addition must be traceable to a concrete query the current model cannot answer. |
| **Automated Compliance:** SHACL shapes would harden the integrity rules currently living only in scope notes and tooling conventions. | |

## 2. Compatibility Baseline

Alignment status between `adr-o.ttl` and each governing ADR, updated to reflect 0.2.1-draft:

- **ADR-0003 (Prose Literals Are Markdown):** ✅ Applied in 0.2.1-draft. `dcterms:description`, `skos:definition`, and `skos:note` carry Markdown scope notes; the ontology header literal is retyped. `skos:prefLabel` explicitly excluded (label, not prose — see DESIGN-NOTES 0.2.1-draft). SHACL `sh:datatype` enforcement deferred to the shapes companion. Bulk retyping of existing `skos:definition` literals on status/valence individuals deferred to a future data-cleanup pass. **ADR-0017** restates the narrowed property scope as implemented.
- **ADR-0004 (The KG Lives Under Tooling):** ✅ Compatible; no ontology changes needed. The ontology's `rdf:List` ordering, scope-notes-over-axioms pattern, and deferred SHACL checks are all consistent with it. Stylistic option (not required): adding `rdfs:seeAlso` to individual ADR IRIs from the ontology header would make the relationship bidirectional.
- **ADR-0005 (Log All Decisions in the ADL):** ✅ Compatible; no ontology changes needed. Both real-time (`Status=Proposed`, appendable `hasDeliberation` list) and post-factum (`dcterms:date` / `dcterms:created` split) authoring modes are supported. The `dcterms:created` scope note was updated in 0.2.1 to name the post-factum case explicitly.
- **ADR-0006–ADR-0018 (ontology vocabulary and shape, post-factum from design notes):** ✅ Aligned with `adr-o.ttl` 0.2.1-draft — relational predicates, indexing, SKOS status and valences, atom-first facts, alternatives, ontology document shape, DC/PROV conventions, `dcterms:version`, Markdown scope as implemented, and editorial `Consideration` IRI patterns. See individual files under `ADL/`.

## 3. Actionable Goals (The Roadmap)

The three horizons below correspond to three distinct ambitions. Horizon 1 is closed. Horizons 2 and 3 together trace the path from Decision Index to Knowledge System.

---

### Horizon 1: Tactical Alignment — *CLOSED (delivered in v0.2.1)*

**What this horizon was about:** Closing the gap between the governing ADRs and the `.ttl` file. Every item here was a consistency obligation, not a design choice — things the ontology had already committed to but not yet applied.

- **[x] Markdown Datatype Rollout:** Applied `^^<https://www.w3.org/ns/iana/media-types/text/markdown>` to `dcterms:description`, `skos:definition`, and `skos:note` (newly declared). `skos:prefLabel` and `*Fact`-level prose properties excluded — see DESIGN-NOTES 0.2.1-draft for the vim-vignette-grounded rationale. Ontology header `dcterms:description` retyped.
- **[x] DESIGN-NOTES Deferral Tracking:** The 0.2.1 DESIGN-NOTES section records the ADR-0003 deferral application and its two scope-narrowing re-readings as an immutable historical record.
- **[x] Semantic Documentation Polish:** `dcterms:created` scope note updated to explicitly name post-factum ADRs. `DecisionRecord` `rdfs:comment` left as-is per DESIGN-NOTES recommendation. `adr-o:Consideration` `rdfs:comment` corrected to remove the erroneous `skos:prefLabel` as a prose-carrier.
- **[x] Basic Metadata Expansion:** `dcterms:version` declared as `owl:AnnotationProperty` for tracking per-record iteration separately from the supersession chain.

---

### Horizon 2: Professional-Grade Record — *Next*

**What this horizon is about:** Making the Index more complete and more trustworthy as a record of what actually happened — who decided, against what criteria, toward what goals, and for how long. These features are the difference between a record that captures the *conclusion* of a decision and one that captures the *decision as an institutional act*. The dogfood exercise confirmed that every item in this horizon surfaced as observable friction when modeling real ADRs; none of these are theoretical gaps.

The governing test for any proposed addition here: *can a current ADR-O 0.2.1 SPARQL query answer this question?* If no, and if the question is one a decision log should be able to answer, the addition is justified. If the question can be answered from prose retrieval alone and structural modeling adds no traversal power, defer it.

#### 2.1 The Social Graph (RACI) — *Priority: highest*

The dogfood report gives Organizational Context the single ❌ in the final verdict table. Both dogfooded records carry a `Type` field and name parties beyond the author; neither has a landing place in 0.2.1.

- [ ] Introduce `adr-o:decidedBy`, `adr-o:consulted`, and `adr-o:informed` as `owl:ObjectProperty` (range: an agent IRI or literal, consistent with `dcterms:creator`). These decouple the *Author* (who wrote the record) from the *Decision Maker* (who had authority) and the *Consulted* and *Informed* parties. Without them, "who is accountable for this decision?" requires reading prose.
- [ ] Introduce `adr-o:hasType` as an `owl:AnnotationProperty` pointing to a SKOS concept (no core scheme — same extension pattern as `adr-o:Concern`). The Type field appeared in both dogfooded records ("Project Governance," "Core Design"); it is epistemic metadata about the record, not domain-specific content. It has been deferred since 0.1.0-draft on the grounds that domain-specific categorisation doesn't belong in the core namespace — but Type categorises the *record*, not the *domain*. That argument no longer holds.

#### 2.2 Criteria and Evaluation — *Priority: high (new gap, not in prior roadmap)*

This is the dogfood report's most structurally novel finding, and it was entirely absent from the original roadmap. ADR-0001's license comparison table — four alternatives evaluated against four binary criteria — could not be modeled as structured data. It was collapsed into prose `Consideration` nodes, destroying the Feature-Graph. The current model is a Claim-Graph; this item would add a Feature-Graph layer alongside it.

- Introduce `adr-o:Criterion` as a new class (parallel to `adr-o:Concern` — an abstract anchor class with no core concept scheme, extensible via SKOS profiles).
- Introduce `adr-o:EvaluationFact` as a new reified link class, parallel to `DeliberationFact`, placing a `Criterion` evaluation onto an `Alternative`. The fact carries the criterion, the alternative, and a result (initially a boolean or a short literal; a richer value type is deferred). This is the shape that makes "show me all alternatives evaluated against criterion X" answerable in SPARQL.
- The design work required before this can ship: deciding whether `EvaluationFact` is a sibling of `*Fact` classes (its own list on `DecisionRecord`, `hasEvaluation`) or a property of `Alternative` directly. The former is more consistent with the existing reified architecture; the latter is simpler to author. This requires a dedicated design sketch and likely an ADL entry before the `.ttl` is touched.

#### 2.3 Epistemic Expansion: Goals — *Priority: medium*

The original roadmap listed this as "Epistemic Expansion (Goals & Trade-offs)" and combined Goals with a trade-off refinement. The trade-off gap (`accepting-that` / `AcceptedCost`) was already closed in 0.2.0-draft with the `AcceptedCost` outcome valence. What remains is the Goals half.

ADR-0005 is structured as a teleological argument: log all decisions *in order to* achieve a specific future capability. The ontology has `Concern` for problems but no first-class anchor for positive targets. Without a `Goal` class or `intendsToAchieve` predicate, teleological arguments must be expressed as `ContextFact`s or positive-valence `DeliberationFact`s, losing the distinction between "a fact we observed" and "an outcome we are aiming for."

- Introduce `adr-o:Goal` as a new class (again, abstract anchor, no core concept scheme). An `intendsToAchieve` predicate from `DecisionRecord` to `Goal` is the minimal addition; a richer design may allow `Goal` nodes to be shared across records, enabling "show me all decisions that contributed to goal X."

#### 2.4 Temporal Governance — *Priority: low-medium*

- Add `adr-o:revisitCondition` (a literal or IRI describing the trigger that should prompt re-evaluation of this record). This prevents Zombie Decisions — records that stay `Accepted` indefinitely after the assumptions that justified them have expired. ADR-0005's stance ("log all decisions because you don't know which will matter later") makes temporal governance *more* important, not less: a large log without decay signals becomes noise.
- The simpler `adr-o:revisitDate` (a literal `xsd:date`) is a useful fallback for teams that prefer time-based triggers. Both can coexist.

---

### Horizon 3: Knowledge System — *Strategic / Long-Term*

**What this horizon is about:** Moving from a record that captures what happened to a system that can reason about why decisions form the patterns they do — across records, across projects, across time. Every item here adds either *relational expressivity* (richer links between records or between records and external artefacts) or *structural enforcement* (SHACL shapes that harden the integrity rules currently living in prose scope notes). None of these are urgent; all of them are load-bearing for the long-term vision.

The governing test for this horizon is different from Horizon 2: the question is not "can 0.2.1 answer this by SPARQL?" but "does this move ADR-O from an Index that requires a human to interpret to a Knowledge System that a machine can traverse without ambiguity?" The distinction matters because Horizon 3 features are expensive — they change the structural surface area of the ontology significantly — and should only ship when the Horizon 2 model is stable enough to build on.

#### 3.1 Argumentative Topology — *new gap, sharpened from prior "Evidence Chain" item*

The original roadmap listed "The Evidence Chain" — linking `Consideration` nodes to external evidence artefacts (benchmarks, documents). That remains valid but is a subset of a larger gap the dogfood exercise named more precisely: **the ontology has no way to model logical dependency between considerations**.

ADR-0005 is structured as a syllogism: Premise 1 (economics of medium) + Premise 2 (tooling availability) → Conclusion (log all). The ontology can list both premises as `ContextFact`s ordered in an `rdf:List`, but it cannot say "Premise 2 presupposes Premise 1" or "the Conclusion follows from both." The `rdf:List` captures sequence; it does not capture inference. This means the *structure* of arguments is invisible to any query that doesn't read prose.

Two complementary items follow from this:

- **Logical dependency links:** A property (tentatively `adr-o:presupposes` or `adr-o:entails`) between `Consideration` nodes that models prerequisite relationships. This is the "Therefore" link the dogfood report identifies as missing.
- **Evidence anchoring:** A property from `Consideration` to an external `Evidence` node (a benchmark result, a specification, a test report) that moves rationale from "prose claim" to "grounded provenance." This was the original "Evidence Chain" item; it is now positioned as the second half of Argumentative Topology, not a standalone item.

Both are research-grade additions that require careful design before any `.ttl` change. A dedicated ADL entry is a prerequisite.

#### 3.2 Fine-Grained Evolution (`amends`, `clarifies`)

The supersession chain (`adr-o:supersedes` / `adr-o:supersededBy`) models full replacement. It cannot model partial modification (a later decision changes only one clause of an earlier one that remains otherwise in force) or pure clarification (no change to the decision, only additional interpretive context added). Both are common governance events that are currently expressed as prose "see also" links with no machine-traversable semantics.

- Introduce `adr-o:amends` and `adr-o:clarifies` as typed link predicates.
- Introduce `Amended` and `Clarified` as new members of `adr-o:statusScheme` — a record that has been partially amended should reflect that in its status so that queries asking "which records are fully current as-written?" can filter correctly. An `Amended` record is still in force; a `Superseded` record is not. This distinction is currently impossible to make by graph traversal.

#### 3.3 Pattern-Based Design (GADR / `DecisionTemplate`)

A `DecisionTemplate` is a record-shaped node that carries `hasAlternative` and `DeliberationFact`s but no `chosenAlternative`, no `OutcomeFact`s, no `dcterms:date`, and no RACI predicates. It is a cross-project archetype — a standing problem context with a known option space, not yet instantiated into any particular decision. The JabRef / Eclipse Winery example from the GADR paper (two projects using the same Java build-tool template but reaching opposite decisions) demonstrates why this is genuinely useful and why modeling it as a `Proposed` record doesn't work: a `Proposed` record is project-specific and is expected to resolve; a template is deliberately left open.

- Introduce `adr-o:DecisionTemplate` as a new class.
- Introduce `adr-o:instantiates` as a property from `DecisionRecord` to `DecisionTemplate`, providing both discovery ("which records were derived from shared organizational patterns?") and provenance (which instantiations drifted from the template over time).
- The 0.2.1 architecture is already shape-compatible with this — `addresses` allows dual-home on both `DecisionRecord` and `Consideration`, preserving room for a template superclass without breaking either existing use.

#### 3.4 Integrity Layer (SHACL)

The planned SHACL shapes companion has been deferred since 0.1.0-draft and grows more important with each Horizon 2/3 addition. The constraints currently living in scope notes and tooling conventions are not enforced at the graph level:

- `chosenAlternative ∈ hasAlternative` (the chosen option must be declared in the option pool)
- `Accepted → chosenAlternative present` (an accepted record must have a decision)
- Per-fact-class field cardinalities (`DeliberationFact` requires `onAlternative`; `ContextFact` forbids it)
- Valence-enum membership per fact class
- `rdf:List` element typing (list members must be of the declared fact class)
- Markdown datatype on prose-carrying properties (`sh:datatype` on `dcterms:description`, `skos:definition`, `skos:note`)

SHACL should ship no later than the first Horizon 3 item, because Horizon 3 additions (especially Argumentative Topology) will introduce new structural constraints that cannot be safely deferred to tooling conventions alone.

## 4. Deferred Items (Explicitly Not Scheduled)

These items have been considered in one or more prior iterations and deliberately held back. They are listed here so that the decision to defer them is visible and can be revisited with new evidence.

- **`adr-o:significance`:** Shape unresolved (literal? SKOS concept? numeric?). ADR-0005 recommends it as a post-entry annotation rather than an entry gate. Deferred until a concrete use case drives the shape decision.
- **MADR-style confirmation / fitness predicates:** No first-class predicate yet for "how will we know this decision is being followed?" (e.g., links from `DecisionRecord` to a CI check, test suite, policy rule, or SHACL shape). Deferred pending a concrete shape that adds traversal value beyond prose and aligns with the Horizon 2 "Professional-Grade Record" scope.
- **Kruchten-style directed `prevents` / `isPreventedBy`:** Symmetric `conflictsWith` is sufficient for all observed cases. Can be added as sub-properties later without breaking existing triples.
- **Splitting `Neutral` (deliberation valence):** The single `Neutral` bucket ("noted but didn't push the choice") is sufficient for all observed cases. Refining into orthogonal vs. in-scope-but-non-decisive is non-breaking when needed.
- **Versioning-discipline ADR:** The absence of a dedicated ADR on semantic versioning, release cadence, and deprecation policy is a growing debt. The project has shipped three `-draft` iterations without resolving this. It should be written before anyone depends on ADR-O in production.
- **Additional serializations (JSON-LD, RDF/XML):** Mechanical from the canonical Turtle. Deferred until there is a publishing infrastructure to host them.
- **Domain profile registry:** No registry, no central discovery mechanism. Profiles publish in their own namespaces. Deferred indefinitely — this is an ecosystem problem, not an ontology problem.
- **`rdfs:seeAlso` per-ADR back-references from the ontology header:** A taste call. The ontology cites its public docs but not individual ADR IRIs. Non-required; deferred.
