# Dogfooding Report: Semantic Conversion of ADR-0001 and ADR-0005

## 1. Executive Summary
This report documents the results of an exercise in "eating our own dog food": the conversion of two internal Architecture Decision Records (ADR-0001 and ADR-0005) into RDF graphs using the **ADR-O 0.2.1** ontology. 

The primary objective was to test the **completeness** and **consistency** of the current model when faced with two different styles of decision-making: a structured, option-based choice (ADR-0001) and a prose-heavy, philosophical argument (ADR-0005).

## 2. Validated Successes (Model Strengths)
The conversion confirmed that the 0.2.0 "Atom-First" redesign is fundamentally sound for capturing the *intent* and *outcome* of a decision.

*   **Option Pool Management:** The `hasAlternative` and `chosenAlternative` pattern worked seamlessly for both records. Whether it was a list of licenses (ADR-0001) or a set of authoring strategies (ADR-0005), the "Option Pool" approach accurately captured the decision space.
*   **Valence-Based Rationale:** The use of `DeliberationFact` and `OutcomeFact` with valence enums (`Supports`, `Against`, `Benefit`, `Risk`) successfully transformed narrative prose into queryable claims. This allowed the "Why not CC0" or "Why not Apache 2.0" sections of ADR-0001 to be modeled as explicit `Against` weights on specific alternatives.
*   **Metadata Alignment:** The delegation of metadata to Dublin Core Terms (`dcterms:title`, `dcterms:creator`, `dcterms:date`) provided a stable and sufficient shell for the records.

## 3. The "Semantic Gap" (Identified Gaps & Frictions)

The exercise revealed that while the ontology captures the *conclusion* of a decision, it performs a "lossy compression" on the *process* of reaching that conclusion.

### 3.1 The Matrix Problem (Structured Data $\rightarrow$ Claims)
In **ADR-0001**, the author uses a Markdown table to compare licenses across four binary dimensions (Attribution, ShareAlike, Copyleft, and OBOnorm).
*   **The Friction:** ADR-O does not model "Feature Sets" or "Criteria." It models "Considerations" as atoms of prose.
*   **The Result:** To represent the table in RDF, the "data" (e.g., *Apache 2.0 = Attribution Required*) had to be converted into a "claim" (e.g., *The fact that Apache 2.0 requires attribution is a reason to support it*). 
*   **Conclusion:** The model is currently a **Claim-Graph**, not a **Feature-Graph**. If we wish to preserve the "Matrix" view, we need a way to model `Criteria` and the `Evaluation` of an `Alternative` against that `Criterion`.

### 3.2 The Narrative Problem (Stories $\rightarrow$ Facts)
**ADR-0005** relies heavily on a detailed "Ambient-Transcription Vignette" (the 2032 Vim migration scenario) to illustrate the end-state of the project.
*   **The Friction:** The vignette is a narrative arc involving characters (John, Alice, Bob) and a temporal sequence. RDF is a set of unordered assertions.
*   **The Result:** The vignette was collapsed into a single `Consideration` node. The "soul" of the argument—the dialogue and the illustrative storytelling—was stripped away, leaving only the resulting "fact" (that real-time instrumentation is possible).
*   **Conclusion:** The ontology is designed for **knowledge retrieval**, not **narrative preservation**. It captures the *what* and the *why*, but discards the *how* of the storytelling.

### 3.3 The Logical Flow Problem (Syllogisms $\rightarrow$ Lists)
**ADR-0005** is structured as a logical proof: *Premise 1 (Economics of Medium) + Premise 2 (Tooling) $\rightarrow$ Conclusion (Log All).*
*   **The Friction:** The ontology uses `rdf:List` for ordering, but lists are sequences, not logical dependencies.
*   **The Result:** We can list the premises in the `hasContext` list, but we cannot explicitly model the "Therefore" link. We can state that Premise 1 *supports* the decision, but we cannot state that Premise 1 *is a prerequisite for the logic* of the decision.
*   **Conclusion:** The model lacks **Argumentative Topology**. It treats considerations as a "bucket of weights" rather than a "chain of logic."

### 3.4 The Metadata & Social Gap
Both records highlighted a lack of "Organizational Context" predicates.
*   **The "Type" Gap:** Both records use a `Type` field (e.g., "Project Governance" in ADR-0001), but the ontology has no predicate to categorize the *nature* of the record.
*   **The RACI Gap:** While `dcterms:creator` captures who wrote the record, there is no way to distinguish the *Author* from the *Decision Maker* or the *Consulted* parties.

## 4. Final Verdict on Model Completeness

| Task | Status | Note |
| :--- | :--- | :--- |
| **Outcome Recovery** | ✅ Complete | "What was decided?" is perfectly answerable. |
| **Rationale Recovery** | ⚠️ Partial | "Why was it decided?" is answerable as a set of claims, but the logical flow and narrative nuance are lost. |
| **Alternative Analysis** | ⚠️ Partial | Options are captured, but the "Matrix-style" comparative analysis is flattened into prose claims. |
| **Provenance/History** | ✅ Complete | Supersession and authorship are well-handled. |
| **Organizational Context**| ❌ Incomplete | Lack of "Record Type" and "RACI" (Decider vs. Author) roles. |

**Overall Conclusion:** ADR-O 0.2.1 is a high-performance tool for **Decision Indexing**. It is excellent for building a machine-traversable map of "what happened and why." However, it is not yet a tool for **Argument Preservation**. To move from an "Index" to a "Knowledge System," future iterations should consider modeling `Criteria` (to solve the Matrix problem) and `Argument Chains` (to solve the Logical Flow problem).