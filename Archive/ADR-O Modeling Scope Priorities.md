# ADR-O Modeling Scope Priorities

## The Primary Use Case: Agent Navigation of the ADL

ADR-O is not a general-purpose knowledge graph, it's not a holistic reasoner, and it's not a repository for telemetry data, metrics, or realized system state. (The Telemetry Tom vignette in ADR-0030 documents that boundary explicitly.)

ADR-O's primary job is to make an **Architecture Decision Log** — the full collection of a project's decision records — efficiently navigable by AI agents and other automated tools.

The core question ADR-O is designed to answer is not *"is this claim true?"* but *"where should I look next?"*

An agent entering an ADL should be able to:

- traverse the graph from any record outward to its dependencies, successors, conflicts, and enablements;
- zoom into a specific record and find its context, deliberation, chosen alternative, verdict, and expected outcomes as first-class, locatable nodes;
- follow typed inter-record links without parsing prose.

That is the scope. Everything else is either in service of that scope or outside it.

The real-time ADR vignette in ADR-0005 and the agent-collaboration scenario in ADR-0025 both illustrate what this means in practice: an AI agent surfaces a constraint from the graph *during* live deliberation, without reading prose, by following typed causal links from a current consideration back to a prior decision's outcome. That traversal is the thing ADR-O exists to enable.

## The Three Scopes and Where Complexity Belongs

ADR-0025 establishes three distinct scopes of causation in the ADR-O model, and the distribution of modeling complexity across them is not accidental.

**ADL scope** is where the richest typed vocabulary lives: `supersedes`, `dependsOn`, `enables`, `conflictsWith`, `amends`, and the causal-modal predicates (`constrainedBy`, `prohibitedBy`, `recommendedBy`, `discouragedBy`, `permittedBy`). This is the layer an agent traverses to understand how decisions relate to one another across time and across the system. It is the primary site of machine-readable reasoning.

**ADR scope** — the interior of a single record — is where *evidence* lives. Context facts, deliberation facts, expected outcomes, and the verdict are all authored at decision time and are primarily for a reader (human or agent) to consume, not traverse in the same graph-navigation sense. The structure here serves locatability and honest authorship, not formal inference.

**Project scope** is the provenance bridge between the ADL and the system artifacts the decisions produced. The `justifiedBy` predicate lets an agent answer "why does this file exist?" by following a link from the artifact back to the decision.

This distribution is a deliberate priority. When a modeling question arises — *should we encode X formally, or leave it to prose?* — the answer depends on which scope X lives in. At ADL scope, formal structure is worth its cost because it enables traversal. At ADR scope, formal structure is worth its cost only when it enables locatability — finding the right node — not when it attempts to encode the internal logic of a human deliberation as a machine-executable syllogism.

## The Consequence: ADR Scope Is Evidence, Not Inference

A direct consequence of the above is that **ADR-O does not attempt to model syllogisms inside decision records**.

The deliberation section of a record contains arguments — `DeliberationFacts` with valences (Supports, Against, Neutral) — but ADR-O does not model argument weight, logical derivation chains, or formal inference from premises to conclusion. Those are operations a human author or an AI agent performs *using* the record, not structure the ontology needs to encode.

This is a deliberate priority, not an oversight. The complexity of a full argumentation vocabulary inside the ADR scope would be significant; the marginal value over well-authored prose at the convergence point is low. ADR-O trusts the author to close the deliberation honestly.

## The Waist of the Fish: `Verdict` as Reified Convergence

The double-diamond / diverge-converge model (sometimes called the fish model in this project) frames every decision as having two phases: a divergent deliberation phase where alternatives and arguments accumulate, and a convergent phase where a choice is made and the deliberation closes.

ADR-O models the divergent phase via `DeliberationFact` nodes with valences. But until the `Verdict` class was introduced, the convergent phase had no first-class representation: the ontology could tell you which alternative was chosen (via `chosenAlternative`), but not *how the deliberation closed* — which arguments weighed more, what trade-offs were consciously accepted, why the balance tipped the way it did.

`Verdict` closes this gap. It is not a description *of* the convergence; it is the convergence itself, reified as a first-class node. This is consistent with the ADR-scope principle above: the `Verdict` does not encode the *logic* of the deliberation — it makes the *act of closure* locatable. An agent navigating the ADL can find the `Verdict`, read it, and know where the decision crystallized, without the ontology pretending to capture why the balance tipped through formal structure.

The parallel with `Complaint` is intentional but asymmetric:

| | `Complaint` | `Verdict` |
|---|---|---|
| **Reifies** | The mouth of the fish — the raw triggering act | The waist of the fish — the convergence act |
| **Epistemic type** | Witnessed act; not a verifiable proposition | Witnessed act; not a verifiable proposition |
| **Mutability** | Ontologically immutable: constitutive of the record's identity | Tier 1 epistemic core: mutable during drafting, frozen on external reference |
| **Content** | Ordinary language; the named situation before analytical sharpening | Authored prose; the author's account of how deliberation closed |
| **Linked from** | `DecisionRecord` via `promptedBy` (functional) | `DecisionRecord` via `closedBy` (functional) |

The mutability asymmetry is important and worth stating plainly. A `Complaint` is the anchor of the fish's identity — the entire deliberation exists because that specific pain was named. Changing the complaint doesn't revise the record; it dissolves it. There is no grace period tied to external reference, because the complaint is constitutive of the record from the moment it is stated: a new mouth is a new fish.

> **Example**  
> Consider a colleague who files a complaint because they can't figure out how to sort rows in a spreadsheet. That complaint is, objectively, a training problem — not a software problem, not an architectural decision, probably not even worth an ADR. And yet: the moment it is named, it anchors a real deliberation. People will discuss *that specific pain*, not some other pain. The team might decide to write a macro, or a quick-reference guide, or simply to explain the built-in sort function — and the record will end up `Rejected` or trivially `Accepted` in thirty minutes. But here is the point: you cannot change the complaint mid-deliberation to something more interesting without changing what the deliberation is *about*. If someone says "actually, let's make this about our general Excel literacy gap across the team," that is a different complaint, a different fish, and a different record. The original record — the one anchored to the hapless colleague's confusion about row sorting — remains exactly as it was named, regardless of how embarrassing or trivial it looks in hindsight.

A `Verdict`, by contrast, is the current state of convergence, and convergence is genuinely provisional until it produces effects. The deliberation can re-open, the author can change their mind, the balance can tip differently — all of this is normal deliberative behavior, not record corruption. The Decision Transaction Principle's external-reference freeze is exactly the right trigger here, because that's the moment the verdict stops being an internal deliberative position and starts being a premise that others depend on. A slight move of the tail doesn't always mean a new fish.

Neither `Complaint` nor `Verdict` is backed by `Claim`. A `Claim` is a verifiable proposition; both of these are witnessed acts, not propositions. These are different epistemic types and must remain separate.

The `Verdict`'s prose (typed `^^adr-o:md` per the established convention) is where the Y-statement's *"accepting that [F]"* clause lives in machine-locatable form: the authored record of what was consciously traded off, which arguments proved decisive, and what costs or risks were knowingly accepted. It is not a formal encoding of the deliberation's logic; it is a locatable, authored statement of an act of judgment.

## Why `closedBy` and Not Something Else

The predicate `closedBy` was chosen over alternatives (`resolvedBy`, `settledBy`, `concludedBy`) for three reasons:

1. **Neutrality**: `closedBy` says the deliberation is closed; it carries no positive valence about the quality of the conclusion. `resolvedBy` implies the problem was solved satisfactorily. A verdict is passed regardless of whether the outcome is good.

2. **Finality**: "Closed" is unambiguous and irreversible in ordinary English in a way that "concluded" is not. A conclusion can be revisited; a closed deliberation cannot.

3. **Fish model alignment**: The fish model's waist is literally the moment the open space closes. `closedBy` maps directly to that geometric intuition.

## The Broader Principle

These specific decisions — not encoding argument weight, modeling the verdict as a prose-carrying reification rather than a formal reasoning structure, choosing `closedBy` for its neutrality — all follow from the same priority: **ADR-O optimizes for efficient agent navigation and honest authorship, not for formal completeness inside the record**.

The graph structure lives at the ADL level. The ADR interior is evidence and authored judgment. ADR-O provides the hooks that make both locatable and traversable; it does not attempt to replace the author's reasoning with a formal system.
