# ADR-O Strategic Gap Analysis & Action Plan

This report provides a forward-facing gap analysis and SWOT-style strategic roadmap for ADR-O. It synthesizes the findings from the template audits and an earlier ontology review to define actionable goals for the next iterations (0.2.1 and beyond).

## 1. SWOT Analysis

| **STRENGTHS** (Current Core) | **WEAKNESSES** (Current Gaps) |
| :--- | :--- |
| **Atom-First Identity:** Solves the YADR identity problem via `Consideration` nodes. | **Social Blindness:** No model for the "Human Graph" (Deciders vs. Creators). |
| **Domain Agnostic:** Portable across any professional domain (`ADR-0002`). | **Epistemic Narrowness:** Models "Problems" (`Concern`) but not "Targets" (`Goal`). |
| **Standard Alignment:** Native use of PROV-O, SKOS, and DC Terms. | **Evidence Gap:** Claims exist, but the "Proof" (benchmarks, docs) is not linked. |
| **Tooling-First Philosophy:** Clean substrate designed for AI/MCP mediation. | **Temporal Staticity:** No mechanism for "Revisit" triggers or record versions. |

| **OPPORTUNITIES** (Future Growth) | **THREATS** (Risks) |
| :--- | :--- |
| **Pattern Libraries:** Implementing GADR templates for cross-project reuse. | **Tooling Dependency:** The "substrate" model fails if the tooling layer isn't built. |
| **Real-Time Instrumentation:** Capturing decisions via ambient meeting transcripts. | **Noise Volume:** "Log all" (`ADR-0005`) may create a "data swamp" without `significance` filters. |
| **Automated Compliance:** Using SHACL to enforce decision integrity. | **Complexity Drift:** Adding too many predicates may alienate lightweight users. |

## 2. Compatibility Baseline

Before the roadmap, a summary of the alignment status between `adr-o.ttl` (0.2.0-draft) and each governing ADR:

*   **ADR-0003 (Prose Literals Are Markdown):** Not yet applied — deferred by design. The ontology pre-dates ADR-0003 by one day; the ADR itself says "to be added in the next ontology iteration." Concrete gaps land in the 0.2.1 Markdown rollout (see Horizon 1).
*   **ADR-0004 (The KG Lives Under Tooling):** Compatible; no ontology changes needed. ADR-0004 promotes an existing design assumption to an ADR without introducing new terms. The ontology's `rdf:List` ordering, scope-notes-rather-than-axioms pattern, and deferred SHACL checks are all consistent with it. *Stylistic option (not required):* the ontology currently references its public docs via `rdfs:seeAlso <https://gutza.github.io/ADR-O/>` but does not cite individual ADRs. Adding `rdfs:seeAlso` to each ADR IRI would make the relationship bidirectional — a taste call for a future iteration.
*   **ADR-0005 (Log All Decisions in the ADL):** Compatible; no ontology changes required. Both authoring modes are already supported: real-time ADRs (via `Status=Proposed`, appendable `hasDeliberation` list, eventual `Rejected` transition) and post-factum ADRs (via the `dcterms:date` / `dcterms:created` split, whose scope notes were written precisely for this case). The `adr-o:significance` annotation mentioned in ADR-0005's open questions is correctly absent from the ontology.

## 3. Actionable Goals (The Roadmap)

I have categorized the remaining gaps into three horizons based on the `ONTOLOGY-REVIEW.md` and the template analysis.

### Horizon 1: Tactical Alignment (Immediate / v0.2.1)
*Focus: Closing the gap between the ADLs and the `.ttl` file.*

*   **[x] Markdown Datatype Rollout:** 
    *   Applied `https://www.w3.org/ns/iana/media-types/text/markdown` to `dcterms:description`, `skos:definition`, and `skos:note` (newly declared). `skos:prefLabel` and `*Fact`-level prose properties excluded — see DESIGN-NOTES 0.2.1-draft for the vim-vignette-grounded rationale.
    *   Re-typed the ontology's own `dcterms:description` (line 17 of the `.ttl`).
*   **[x] DESIGN-NOTES Deferral Tracking:**
    *   The 0.2.1 DESIGN-NOTES include a "Decisions from prior drafts applied in this iteration" subsection that records the ADR-0003 deferral application and its two scope-narrowing re-readings.
*   **[x] Semantic Documentation Polish:**
    *   Updated `dcterms:created` scope note to explicitly name post-factum ADRs as a use case.
    *   The `DecisionRecord` `rdfs:comment` ("A record of a single, load-bearing decision…") left as-is per recommendation.
*   **[x] Basic Metadata Expansion:**
    *   Added `dcterms:version` as `owl:AnnotationProperty` to track internal record iterations separately from the supersession chain.

### Horizon 2: Semantic Enrichment (Mid-term)
*Focus: Adding the "Professional Grade" features found in MADR/YADR/JSON.*

*   **[ ] The Social Graph (RACI):**
    *   Introduce `adr-o:decidedBy`, `adr-o:consulted`, and `adr-o:informed` to decouple the *Author* from the *Decision Maker*.
*   **[ ] Epistemic Expansion (Goals & Trade-offs):**
    *   **Targets:** Introduce `adr-o:Goal` (or `intendsToAchieve`) to capture positive targets, not just problem-solving.
    *   **Trade-offs:** Refine `OutcomeFact` or add a predicate to distinguish "Accidental Side-effects" from "Conscious Trade-offs" (`accepting-that`).
*   **[ ] Temporal Governance:**
    *   Add `adr-o:revisitDate` or `adr-o:revisitCondition` to prevent "Zombie Decisions" (decisions that stay in force long after their assumptions have expired).

### Horizon 3: Architectural Evolution (Strategic/Long-term)
*Focus: Moving from a "Record Log" to a "Knowledge System."*

*   **[ ] Pattern-Based Design (GADR):**
    *   Implement `adr-o:DecisionTemplate` and the `adr-o:instantiates` relationship to allow a library of organizational patterns.
*   **[ ] The Evidence Chain:**
    *   Develop a mechanism to link `Consideration` nodes to `Evidence` nodes (e.g., a benchmark result), moving from "Rationale as Prose" to "Rationale as Provenance."
*   **[ ] Fine-Grained Evolution:**
    *   Introduce `adr-o:amends` and `adr-o:clarifies` to model modifications that don't trigger a full supersession.
*   **[ ] Integrity Layer:**
    *   Ship the planned SHACL shapes graph to enforce the constraints currently living in "Scope Notes" (e.g., `Accepted` $\rightarrow$ `chosenAlternative` required).

## 4. Summary of Priority "Quick-Wins"

If the goal is to move from `0.2.0-draft` to a stable `0.2.1`, the immediate priority list is:
1. **Markdown Datatype** (Alignment with `ADR-0003`).
2. **Post-Factum Scope Notes** (Alignment with `ADR-0005`).
3. **RACI Predicates** (Closing the most glaring professional gap).
4. **Goal/Target Class** (Closing the "problem-only" bias).