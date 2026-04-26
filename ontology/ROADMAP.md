# ADR-O Strategic Gap Analysis & Roadmap

This document is the living strategic roadmap for ADR-O. It was originally drafted before any real-world modeling had been attempted with the ontology; it has since been aggressively revised in light of the **Dogfood Report** (see `DOGFOOD-REPORT.md`), which documents the conversion of ADR-0001 and ADR-0005 into RDF graphs using ADR-O 0.2.1. Where the original roadmap reasoned from template audits and structural review alone, this revision is grounded in observed friction during actual modeling.

The single most important finding from that exercise is the one that reframes the entire roadmap: **ADR-O 0.2.x is a high-performance Decision Index – it is not yet a Knowledge System.** It answers "what was decided and why" as a set of weighted claims with reliable IRI identity. With 0.2.3, social-role attribution (`authoredBy` / `decidedBy` / `consulted` / `informed`) became first-class. With 0.2.5, ADR-0025 added an **ADL scope** causal topology; ADR-0032 later unified that family to (`constrainedBy`, `prohibitedBy`, `recommendedBy`, `discouragedBy`, `permittedBy`) and aligned naming with DTP-safe directionality. The six clauses of a Y-statement still map onto ADR-O's ADR scope constructs, and the **ADL scope** "Therefore" link is machine-traversable with RFC 2119 modal strength. What the model still cannot do is answer three Y-statement questions losslessly on roundtrip: *"to achieve what?"* (no `Goal` class, no teleological predicate — `ExpectedOutcome`+`ExpectedGain` is retrospective, not intentional), *"why not the neglected option?"* (no `Criterion` class, no structured evaluation — comparative rationale collapses into prose), and *"how do the premises inside one record connect to the conclusion?"* (no ADR scope `Claim→Claim` inference edges — ADR scope syllogisms are invisible to SPARQL). Closing those three remaining gaps is what the horizons below are organized around.

## 1. SWOT Analysis

### STRENGTHS (What We Do Well)
- **Atom-First Identity:** `Consideration` nodes with stable IRI identity enable machine-verifiable intra-record coherence; cross-record connectivity uses reference reuse via `adr-o:derivedFrom`/`adr-o:derives` (ADR-0028).
- **Social Graph (RACI):** `authoredBy`, `decidedBy`, `consulted`, and `informed` provide a first-class RACI mapping; `dcterms:creator` coexists as a DC-compatibility layer per ADR-0022. Social-role queries are now answerable without reading prose.
- **Option Pool Integrity:** `hasAlternative` + `chosenAlternative` + valenced `DeliberationFact`s give machine-traversable decision outcomes without text parsing.
- **Domain Agnostic:** Portable across any professional domain (`ADR-0002`).
- **Standard Alignment:** Native use of PROV-O, SKOS, and DC Terms; no heavyweight `owl:imports`.
- **Tooling-First Philosophy:** Clean KG substrate designed for AI/MCP mediation per ADR-0004.

### WEAKNESSES** (Known Gaps)
- **Criteria Blindness:** No model for structured comparative evaluation. Comparison tables (ADR-0001's license matrix) must be collapsed into prose claims, destroying the Feature-Graph.
- **Epistemic Narrowness:** Models problems (`Concern`) but not positive targets (`Goal`). The Y-statement's *"to achieve"* clause has no first-class landing place: `OutcomeFact`+`Benefit` is retrospective (what was realised), whereas *"to achieve"* is teleological (what was intended). The ontology cannot currently distinguish them.
- **Partial Argumentative Flatness:** **ADL scope** "Therefore" links shipped with ADR-0025 and were renamed by ADR-0032 (`constrainedBy` / `prohibitedBy` / `recommendedBy` / `discouragedBy` / `permittedBy`, RFC 2119 modal strength). What remains invisible to SPARQL is the *ADR scope* dimension: presupposition or entailment relationships between `Claim` nodes *within one record's deliberation* — the Y-statement's `Facing → Decided` bridge — have no inference-edge representation.
- **Temporally Static:** No mechanism for "Revisit" triggers or decay conditions. Zombie Decisions — records that stay `Accepted` long after their assumptions expired — are undetectable.

### OPPORTUNITIES (Growth Horizon)
- **Criteria / Evaluation modeling:** Bridging the Matrix gap would make ADR-O useful for structured decision analysis, not just narrative capture.
- **Pattern Libraries (GADR):** A `DecisionTemplate` class would enable cross-project reuse and organizational governance queries impossible in any existing ADR format.
- **Real-Time Instrumentation:** The ADR-0005 ambient-transcription design is validated by the dogfood exercise — the 0.2.1 model already handles real-time records without friction.
- **Automated Compliance:** SHACL shapes would harden the integrity rules currently living only in scope notes and tooling conventions.

### THREATS (Failure Modes)
- **Tooling Dependency:** The "substrate" model fails if the tooling layer never materialises. The ontology is designed for a layered world; without the layers it reads as skeletal.
- **Noise Volume:** "Log all" (`ADR-0005`) may create a data swamp without `adr-o:significance` filters. The risk scales with adoption.
- **Complexity Drift:** Each Horizon 2/3 item adds predicates. The Index/Knowledge System framing is the guard against over-engineering: every addition must be traceable to a concrete query the current model cannot answer.

## 2. Compatibility Baseline

Alignment status between `adr-o.ttl` and each governing ADR, updated to reflect 0.4.3-draft:

- **ADR-0003 (Prose Literals Are Markdown):** ✅ Applied in 0.2.1-draft. `dcterms:description`, `skos:definition`, and `skos:note` carry Markdown scope notes; the ontology header literal is retyped. `skos:prefLabel` explicitly excluded (label, not prose — see DESIGN-NOTES 0.2.1-draft). SHACL `sh:datatype` enforcement deferred to the shapes companion. Bulk retyping of existing `skos:definition` literals on status/valence individuals deferred to a future data-cleanup pass. **ADR-0017** restates the narrowed property scope as implemented.
- **ADR-0004 (The KG Lives Under Tooling):** ✅ Compatible; no ontology changes needed. The ontology's `rdf:List` ordering, scope-notes-over-axioms pattern, and deferred SHACL checks are all consistent with it. Stylistic option (not required): adding `rdfs:seeAlso` to individual ADR IRIs from the ontology header would make the relationship bidirectional.
- **ADR-0005 (Log All Decisions in the ADL):** ✅ Compatible; no ontology changes needed. Both real-time (`Status=Proposed`, appendable `hasDeliberation` list) and post-factum (`dcterms:date` / `dcterms:created` split) authoring modes are supported. The `dcterms:created` scope note was updated in 0.2.1 to name the post-factum case explicitly.
- **ADR-0006–ADR-0032 (ontology vocabulary and shape, post-factum from design notes):** ✅ ADL-level direction is aligned through ADR-0032. The `.ttl` baseline is now 0.4.3-draft and includes unified amendment semantics from ADR-0019/ADR-0020 (`amends`/`amendedBy`, no separate clarification predicate, no `Amended` status), social-role predicates from ADR-0021 (`authoredBy`, `decidedBy`, `consulted`, `informed`), coexistence semantics from ADR-0022 (`dcterms:creator` and `adr-o:authoredBy`), record-level typing from ADR-0023 (`adr-o:hasType`), sequential-ID policy alignment from ADR-0024, ADL-scope causal topology from ADR-0025 as amended by ADR-0032 (`constrainedBy`, `prohibitedBy`, `recommendedBy`, `discouragedBy`, `permittedBy`), and ontology provenance linkage from ADR-0026 (`adr-odr` namespace and `justifiedBy` annotations). See individual files under `ADL/`.

## 3. Actionable Goals (The Roadmap)

The three horizons below correspond to three distinct ambitions. Horizon 1 is closed. Horizons 2 and 3 together trace the path from Decision Index to Knowledge System.

<div data-dn-section="governing-tests-roadmap">

### The Governing Tests

<div data-dn-passage="governing-tests-framework-roadmap" data-dn-record="adl" data-dn-adl-refs="0025">

Two tests govern every proposed addition to Horizons 2 and 3. An addition is justified only when it passes at least one; it is deferred if prose retrieval alone can answer the question and structural modeling adds no traversal power.

**Test 1 — SPARQL traversability:** Can a current ADR-O SPARQL query answer this question without reading prose? If no, and the question is one a decision log should answer structurally, the addition is justified.

**Test 2 — Y-statement roundtrip:** Can ADR-O decompose a six-clause [Y-statement](/Archive/Y-Statements.md) from markdown into triples and recompose those triples back into a Y-statement without information loss? A clause is *lossless* if its content maps onto a named, machine-traversable ADR-O construct; it is *lossy* if it collapses into a prose literal or is indistinguishable from another clause (or by using meta-prose in the triples to describe the shape of the clause).

</div>

<div data-dn-passage="y-statement-roundtrip-analysis-roadmap" data-dn-record="adl" data-dn-adl-refs="0025">

The current roundtrip status after 0.2.5-draft / ADR-0025:

| Y-Statement Clause | ADR-O Construct | Roundtrip Status |
| :--- | :--- | :--- |
| *"In the context of..."* | `hasContext` → `ContextFact` → `Consideration` (type: `Concern` or situational) | ✅ Lossless |
| *"Facing..."* | `hasContext` → `ContextFact` → `Consideration` (`addresses` a `Concern`) | ✅ Lossless (structural) |
| *"We decided for..."* | `chosenAlternative` → `Alternative` | ✅ Lossless |
| *"And neglected..."* | `hasAlternative` ∖ `chosenAlternative` | ⚠️ Lossy — options are named, but *why* the chosen option was preferred over each neglected one requires `Criterion`; without it, comparative rationale collapses into prose `DeliberationFact`s |
| *"To achieve..."* | `hasOutcome` → `OutcomeFact` (valence: `Benefit`) | ⚠️ Lossy — `Benefit` is retrospective (what was realised); *"to achieve"* is teleological (what was intended). No `Goal` class exists to hold the intent. |
| *"Accepting that..."* | `hasOutcome` → `OutcomeFact` (valence: `AcceptedCost`) | ✅ Lossless |
| *[implicit: Facing → Decided link]* | — | ❌ Missing — premises (`ContextFact`s) and conclusion (`chosenAlternative`) are co-located but have no inference edge within a record; ADR scope syllogistic structure is invisible to SPARQL |

The three lossy/missing entries are the three items Horizons 2 and 3 are now organised around.

</div>

</div>

### Horizon 1: Tactical Alignment — *CLOSED (delivered in v0.2.1)*

**What this horizon was about:** Closing the gap between the governing ADRs and the `.ttl` file. Every item here was a consistency obligation, not a design choice — things the ontology had already committed to but not yet applied.

- **[x] Markdown Datatype Rollout:** Applied `^^<https://www.w3.org/ns/iana/media-types/text/markdown>` to `dcterms:description`, `skos:definition`, and `skos:note` (newly declared). `skos:prefLabel` and `*Fact`-level prose properties excluded — see DESIGN-NOTES 0.2.1-draft for the vim-vignette-grounded rationale. Ontology header `dcterms:description` retyped.
- **[x] DESIGN-NOTES Deferral Tracking:** The 0.2.1 DESIGN-NOTES section records the ADR-0003 deferral application and its two scope-narrowing re-readings as an immutable historical record.
- **[x] Semantic Documentation Polish:** `dcterms:created` scope note updated to explicitly name post-factum ADRs. `DecisionRecord` `rdfs:comment` left as-is per DESIGN-NOTES recommendation. `adr-o:Consideration` `rdfs:comment` corrected to remove the erroneous `skos:prefLabel` as a prose-carrier.
- **[x] Basic Metadata Expansion:** `dcterms:version` declared as `owl:AnnotationProperty` for tracking per-record iteration separately from the supersession chain.

### Horizon 2: Professional-Grade Record — *Next*

**What this horizon is about:** Making the Index more complete and more trustworthy as a record of what actually happened — who decided, against what criteria, toward what goals, and for how long. These features are the difference between a record that captures the *conclusion* of a decision and one that captures the *decision as an institutional act*. The dogfood exercise confirmed that every item in this horizon surfaced as observable friction when modeling real ADRs; none of these are theoretical gaps.

The governing tests for proposed additions are defined in the [Governing Tests](#the-governing-tests) section above. The primary test for Horizon 2 items is SPARQL traversability: if the current model cannot answer a question a decision log should answer, and prose retrieval alone adds no traversal power, the addition is justified.

#### 2.1 The Social Graph (RACI) — *Priority: highest*

The dogfood report gave Organizational Context the single ❌ in the final verdict table. Both dogfooded records carried a `Type` field and named parties beyond the author; in 0.2.3, social-role predicates now provide a first-class landing place for those actors.

- [x] Introduce `adr-o:authoredBy`, `adr-o:decidedBy`, `adr-o:consulted`, and `adr-o:informed` as `owl:ObjectProperty` predicates (object: agent IRI, e.g. `prov:Agent`). These expose an explicit RACI mapping in domain terms — *Responsible* as record authorship (`authoredBy`, not necessarily execution ownership), *Accountable* as decision authority (`decidedBy`), plus *Consulted* and *Informed* parties. Without them, "who is accountable for this decision?" requires reading prose. Implemented in `0.2.3-draft` via ADR-0021.
- [x] Introduce `adr-o:hasType` as an `owl:AnnotationProperty` pointing to a SKOS concept (no core scheme — same extension pattern as `adr-o:Concern`). The Type field appeared in both dogfooded records ("Project Governance," "Core Design"); it is epistemic metadata about the record, not domain-specific content. It has been deferred since 0.1.0-draft on the grounds that domain-specific categorisation doesn't belong in the core namespace — but Type categorises the *record*, not the *domain*. That argument no longer holds. Implemented in `0.2.4-draft` via ADR-0023.

#### 2.2 Criteria and Evaluation — *Priority: high*

This is the dogfood report's most structurally novel finding, and it was entirely absent from the original roadmap. The license comparison table in ADR-0001 — four alternatives evaluated against four binary criteria — could not be modeled as structured data. During the dogfood experiment it was collapsed into prose `Consideration` nodes, destroying the Feature-Graph. The current model is a Claim-Graph; this item would add a Feature-Graph layer alongside it. In Y-statement terms, this is the *"because [Criterion X]"* bridge implicit in *"We decided for A and neglected B"*: without `Criterion`, the roundtrip table row for *"And neglected..."* stays ⚠️ lossy.

- Introduce `adr-o:Criterion` as a new class (parallel to `adr-o:Concern` — an abstract anchor class with no core concept scheme, extensible via SKOS profiles).
- Introduce `adr-o:EvaluationFact` as a new reified link class, parallel to `DeliberationFact`, placing a `Criterion` evaluation onto an `Alternative`. The fact carries the criterion, the alternative, and a result (initially a boolean or a short literal; a richer value type is deferred). This is the shape that makes "show me all alternatives evaluated against criterion X" answerable in SPARQL.
- The design work required before this can ship: deciding whether `EvaluationFact` is a sibling of `*Fact` classes (its own list on `DecisionRecord`, `hasEvaluation`) or a property of `Alternative` directly. The former is more consistent with the existing reified architecture; the latter is simpler to author. This requires a dedicated design sketch and likely an ADL entry before the `.ttl` is touched.

#### 2.3 Teleological Intent: Goals — *Priority: highest*

The original roadmap listed this as "Epistemic Expansion (Goals & Trade-offs)" at `medium` priority. The Y-statement roundtrip analysis upgrades it: *"to achieve"* is the clause where the current model is most misleading, not merely incomplete. `OutcomeFact`+`Benefit` and a `Goal` are both positive-valence nodes, but they are epistemically different kinds of claim — the first records what happened after the decision, the second records what the author *intended* to happen before it. Conflating them destroys the hypothesis-vs-fact distinction that is the backbone of any reasoning about whether a decision achieved its purpose.

The trade-off half of the original item (`accepting-that` / `AcceptedCost`) was already closed in 0.2.0-draft. What remains is the intent half.

- Introduce `adr-o:Goal` as a new class (abstract anchor, no core concept scheme — same extension pattern as `adr-o:Concern`). An `adr-o:intendsToAchieve` predicate from `DecisionRecord` to `Goal` is the minimal addition; a richer design may allow `Goal` nodes to be shared across records, enabling "show me all decisions that contributed to goal X" as a SPARQL query rather than a prose search.
- The design boundary to decide before shipping: whether `Goal` is a subclass of `Concern` (goals and concerns as two poles of the same concern space), a sibling class, or entirely separate. The minimal ship is sibling + `intendsToAchieve`; the richer option would allow `Concern` and `Goal` to be linked (e.g., a `Goal` is the resolution of a `Concern`), which is a later extension not required for the first version.

#### 2.4 ADR Scope Argumentative Topology — *Priority: medium-high*

> **Status: partially implemented in `adr-o.ttl`.** The ADL-scope half is implemented (`constrainedBy`, `prohibitedBy`, `recommendedBy`, `discouragedBy`, `permittedBy`) via ADR-0025 + ADR-0032. The ADR-scope half remains open: neither `adr-o:presupposes` nor `adr-o:entails` exists in the ontology. Still blocked on the dedicated ADL entry that must resolve direction (presupposition vs. entailment) and the scope of `Facing → Decided` bridge edges before any `.ttl` change.

ADR-0025 (amended by ADR-0032) closed the *ADL scope* argumentative gap: a `Claim` in a later decision can now point back to an `ExpectedOutcome` from a prior decision via modal-strength predicates (`constrainedBy`, `prohibitedBy`, `recommendedBy`, `discouragedBy`, `permittedBy`). What remains is the *ADR scope* dimension: logical dependency between `Claim` nodes *within one record's deliberation*.

ADR-0005's context list is a syllogism — Premise 1 (economics of medium) + Premise 2 (tooling availability) → Conclusion (log all) — but the `rdf:List` that carries it captures only sequence, not inference. There is currently no way to say "Premise 2 presupposes Premise 1" or "the Conclusion follows from both" inside one record. The Y-statement roundtrip table's ❌ row (*Facing → Decided link*) is precisely this gap.

- Introduce a property (tentatively `adr-o:presupposes` or `adr-o:entails`) between `Claim` nodes, scoped to ADR scope use. This is the ADR scope counterpart of the ADL scope causal predicates already implemented.
- Scope strictly to `Claim → Claim` edges within one record's `hasContext` or `hasDeliberation` list. Do not extend to ADL scope use (that is already covered by ADR-0025 as amended by ADR-0032).
- Design prerequisite: a dedicated ADL entry that resolves whether the direction is `Claim → Claim` (presupposition) or `Claim ← Claim` (entailment), and whether inference edges should also be permitted between `ContextFact`s and a `chosenAlternative` node (the `Facing → Decided` bridge proper).

#### 2.5 Temporal Governance — *Priority: low-medium*

- Add `adr-o:revisitCondition` (a literal or IRI describing the trigger that should prompt re-evaluation of this record). This prevents Zombie Decisions — records that stay `Accepted` indefinitely after the assumptions that justified them have expired. ADR-0005's stance ("log all decisions because you don't know which will matter later") makes temporal governance *more* important, not less: a large log without decay signals becomes noise.
- The simpler `adr-o:revisitDate` (a literal `xsd:date`) is a useful fallback for teams that prefer time-based triggers. Both can coexist.

### Horizon 3: Knowledge System — *Strategic / Long-Term*

**What this horizon is about:** Moving from a record that captures what happened to a system that can reason about why decisions form the patterns they do — across records, across projects, across time. Every item here adds either *relational expressivity* (richer links between records or between records and external artefacts) or *structural enforcement* (SHACL shapes that harden the integrity rules currently living in prose scope notes). None of these are urgent; all of them are load-bearing for the long-term vision.

The governing test for this horizon applies both tests from the [Governing Tests](#the-governing-tests) section above, with a higher bar: the question is not only "can SPARQL answer this?" or "does it close a Y-statement roundtrip gap?" but also "does this move ADR-O from an Index that requires a human to interpret to a Knowledge System that a machine can traverse without ambiguity?" Horizon 3 features are expensive — they change the structural surface area of the ontology significantly — and should only ship when the Horizon 2 model is stable enough to build on.

#### 3.1 Evidence Anchoring — *narrowed from prior "Argumentative Topology" item*

The original roadmap listed "Argumentative Topology" as a single item covering two things: logical-dependency links between `Consideration` nodes, and evidence anchoring from `Consideration` to external artefacts. That item has since been split:

- **The logical-dependency (ADR scope) half** has moved up to **Horizon 2.4**, because the Y-statement roundtrip table shows it as a named ❌ gap, and the ADL scope half already shipped with ADR-0025. ADR scope presupposition/entailment edges belong in Horizon 2, not here.
- **The evidence-anchoring half** remains here as a standalone Horizon 3 item.

The evidence-anchoring gap: `Consideration` nodes carry their rationale as prose literals (`dcterms:description`, `skos:definition`). There is currently no structural link from a `Consideration` to an external artefact that *grounds* the claim — a benchmark result, a specification, a test report, a paper. Without such a link, rationale is asserted but not evidenced; the graph is a Claim-Graph, not a Fact-Graph.

- Introduce an `adr-o:Evidence` class (or align to an existing provenance class such as `prov:Entity`) as the target of a `Consideration → Evidence` link.
- Introduce a property (tentatively `adr-o:groundedIn` or `adr-o:supportedBy`) from `Consideration` to `Evidence`.
- This is a research-grade addition: the shape of `Evidence` (IRI-only? typed? with provenance metadata?) requires a dedicated design sketch and an ADL entry before any `.ttl` change.

#### 3.2 Fine-Grained Evolution (Amendments) — *Direction fixed by ADR-0019/ADR-0020*

The supersession chain (`adr-o:supersedes` / `adr-o:supersededBy`) models full replacement. Partial modifications are now governed by accepted ADL decisions:

- Use a unified amendment model: `adr-o:amends` / `adr-o:amendedBy`.
- Do not split clarifications into a separate predicate; clarifications are modeled as amendments.
- Do not introduce an `Amended` status individual; amendment state is reconstructed from topology (`adr-o:amendedBy` edges), consistent with the reconstructability boundary rule.

Remaining roadmap work here is implementation and tooling hardening, not vocabulary design:

- integrate the accepted amendment predicates into the ontology release line after 0.2.1-draft;
- ensure "Current State" retrieval in tooling follows topology-first merge logic (anchor + all amendments), rather than status-only filtering.

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

SHACL should ship no later than the first Horizon 3 item, because Horizon 3 additions (especially Evidence Anchoring and the ADR scope topology work already in Horizon 2.4) will introduce new structural constraints that cannot be safely deferred to tooling conventions alone.

## 4. Deferred Items (Explicitly Not Scheduled)

These items have been considered in one or more prior iterations and deliberately held back. They are listed here so that the decision to defer them is visible and can be revisited with new evidence.

- **`adr-o:significance`:** Shape unresolved (literal? SKOS concept? numeric?). ADR-0005 recommends it as a post-entry annotation rather than an entry gate. Deferred until a concrete use case drives the shape decision.
- **MADR-style confirmation / fitness predicates:** No first-class predicate yet for "how will we know this decision is being followed?" (e.g., links from `DecisionRecord` to a CI check, test suite, policy rule, or SHACL shape). Deferred pending a concrete shape that adds traversal value beyond prose and aligns with the Horizon 2 "Professional-Grade Record" scope.
- **Kruchten-style directed `prevents` / `isPreventedBy`:** Symmetric `conflictsWith` is sufficient for all observed cases. Can be added as sub-properties later without breaking existing triples.
- **Splitting `Neutral` (deliberation valence):** The single `Neutral` bucket ("noted but didn't push the choice") is sufficient for all observed cases. Refining into orthogonal vs. in-scope-but-non-decisive is non-breaking when needed.
- **Versioning-discipline ADR:** The absence of a dedicated ADR on semantic versioning, release cadence, and deprecation policy is a growing debt. The project has published multiple `-draft` iterations without resolving this. It should be written before anyone depends on ADR-O in production.
- **Additional serializations (JSON-LD, RDF/XML):** Mechanical from the canonical Turtle. Deferred until there is a publishing infrastructure to host them.
- **Domain profile registry:** No registry, no central discovery mechanism. Profiles publish in their own namespaces. Deferred indefinitely — this is an ecosystem problem, not an ontology problem.
- **`rdfs:seeAlso` per-ADR back-references from the ontology header:** A taste call. The ontology cites its public docs but not individual ADR IRIs. Non-required; deferred.
