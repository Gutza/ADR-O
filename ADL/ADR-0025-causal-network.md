---
id: 25
type: Core vocabulary
status: Accepted
date: 2026-04-22
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0025 — Causal Topology: Scopes, Modal Constraints, and Materialization

## Context

ADR-0006 introduced relational predicates (`supersedes`, `dependsOn`, `enables`, `conflictsWith`) that allow us to traverse a decision log. But these are **relational**—they tell us that decisions are connected, but they don't tell us **how** or **why**.

A key organizational pain point is understanding why a specific system artifact (e.g., a lock file, a config flag, a CI step) is critical. Currently, this requires reading through ADRs and reconstructing the chain of "this because that" in one's head.

The real-time agent scenario (ADR-0005 vignette) demands that an agent be able to surface relevant history during a live deliberation. This requires that **causality is first-class and machine-traversable**, and that it operates across three distinct scopes of reasoning.

## Decision

**ADR-O establishes a three-scope causal architecture that separates internal reasoning, inter-decision constraints, and deliverable provenance.**

### 1. The Three Scopes of Causation

#### Scope 1: Intra-ADR (The Deliberation)
Within a single record, causality is **argumentative**. A `Consideration` is weighed against an `Alternative` to produce a decision. This scope is designed to be a machine-readable encoding of ZIO's **Y-Statement** structure:

| Y-Statement Clause | ADR-O Construct |
| :--- | :--- |
| *"In the context of..."* | `adr-o:hasContext` $\to$ `ContextFact` $\to$ `Consideration` |
| *"Facing..."* | `adr-o:hasContext` $\to$ `ContextFact` $\to$ `Consideration` (specifically a concern) |
| *"We decided for..."* | `adr-o:chosenAlternative` |
| *"And neglected..."* | `adr-o:hasAlternative` (those not chosen) |
| *"To achieve..."* | `adr-o:hasOutcome` $\to$ `OutcomeFact` with `adr-o:Benefit` |
| *"Accepting that..."* | `adr-o:hasOutcome` $\to$ `OutcomeFact` with `adr-o:AcceptedCost` |

The `adr-o:outcomeValence` predicate is the structural encoding of the Y-Statement's payoff/cost distinction. It transforms a prose "acceptance" into a queryable fact.

#### Scope 2: Intra-ADL (The Causal Network)
Between decisions, causality is **retrospective**. A later decision recognizes that a prior decision's outcome shapes its current options.

- **Causal anchor:** `adr-o:OutcomeFact` (minted by a `DecisionRecord` at the time of acceptance).
- **Predicates:**
    - `adr-o:constrainedBy` (RFC 2119: **MUST**)
    - `adr-o:prohibitedBy` (RFC 2119: **MUST NOT**)
    - `adr-o:recommends` (RFC 2119: **SHOULD**)
    - `adr-o:discourages` (RFC 2119: **SHOULD NOT**)
    - `adr-o:enabledBy` (RFC 2119: **MAY**)
- **Causal direction:** `Consideration` (in later ADR) → `OutcomeFact` (from prior ADR).
- **Nature:** A claim made by the later decision about its own constraints. The prior decision does not "push" constraints forward; the later decision "pulls" them back.

#### Scope 3: Project (The Deliverable)
Between the ADL and the system artifacts it describes.
- **Predicates:**
    - `adr-o:materializes` (`DecisionRecord` $\to$ `rdfs:Resource`)
    - `adr-o:justifiedBy` (`rdfs:Resource` $\to$ `DecisionRecord`)
- **Causal direction:** The deliverable points back to its justification.
- **Nature:** Provenance. The deliverable is a materialized effect of a decision.

---

### 2. The Causation Model

**The Causal Loop:**
An `OutcomeFact` carries an `outcomeValence` (e.g., `adr-o:AcceptedCost`). This valence is an **intra-ADR** statement about the decision's cost. In a subsequent ADR, a `Consideration` may point to that `OutcomeFact` via a **Scope 2** predicate like `adr-o:constrainedBy`.

**The Materialization Bridge:**
`adr-o:materializes` links a `DecisionRecord` to a system artifact (lock file, config, class). The artifact then carries `adr-o:justifiedBy` back to the decision. This is the **Scope 3** entry point for anyone asking "Why does this file exist?".

```turtle
# The causal chain
mr-adl:ADR-0001  adr-o:materializes  mr-res:package-lock.json .
mr-res:package-lock.json            adr-o:justifiedBy  mr-adl:ADR-0001 .

mr-adl:outcome-A  a adr-o:OutcomeFact ;
                  adr-o:outcomeValence adr-o:AcceptedCost .

mr-adl:cons-B  a adr-o:Consideration ;
               adr-o:constrainedBy  mr-adl:outcome-A .
```

---

### 3. The Supersession Corollary

**A superseded `DecisionRecord` has no active causal footprint.**

When `ADR-B` `supersedes` `ADR-A`, `ADR-A`'s `OutcomeFact`s become **semantically void**. They still exist in the graph as historical facts, but they no longer act as constraints or enablers for future decisions.

**Tooling implication:** Any `Consideration` that is `constrainedBy` an `OutcomeFact` of a superseded decision is flagged as having a **stale constraint**. This is a primary value of the graph—it reveals when a decision is relying on a premise that is no longer in force.

---

### 4. Reasoning and Inferences

By making causal links first-class, an OWL reasoner can discover emergent properties of the decision network:

**Transitive Causality**
If `ADR-C` is `constrainedBy` an outcome of `ADR-B`, and `ADR-B` was `enabledBy` an outcome of `ADR-A`, then `ADR-C` has an implicit causal dependency on `ADR-A`. The reasoner can expose this chain.

**Contradiction Detection**
If a `Consideration` is simultaneously `enabledBy` and `prohibitedBy` outcomes from the same decision (or related decisions), the reasoner flags a logical inconsistency in the deliberation.

**Impact Analysis**
"If I supersede ADR-X, which downstream Considerations in the ADL now have voided constraints?" This becomes a simple SPARQL query over the causal graph, rather than a manual audit.

---

## Real-time ADR Agent

Starting from the real-time ADR concept in ADR-0005, ADR-0025 enables the upgraded version of that scenario:

> **John:** "I think we should migrate to vim. The team is already familiar with it."
>
> **Agent (interrupts):** "I've found a relevant constraint. You're currently discussing the editor choice, but `cons-vscode-plugin-dependency` is a Consideration in your current deliberation. It is `adr-o:constrainedBy` the `outcome-plugin-existence` of ADR-0723 ('Install VSCode extensions for project-specific linting')."
>
> **John:** "What does that mean specifically?"
>
> **Agent:** "ADR-0723 materializes the `.vscode/extensions.json` file. That file contains the linting plugin that enforces our project's style guide. If you migrate to vim, you'll need to find a vim equivalent for that plugin or the CI will fail."
>
> **John:** "Fine, we'll just find a vim plugin."
>
> **Agent:** "Before you do—ADR-0723 is also a causal anchor for **four other decisions**: the CI pipeline (ADR-0112), the onboarding guide (ADR-0201), the contributor contract (ADR-0415), and the security scan workflow (ADR-0503). All four of those decisions are `constrainedBy` that same outcome. If you change the linting mechanism, you're not just changing an editor—you're pulling the rug out from under four downstream decisions. Would you like me to map the impact chain?"

---

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| **Forward constraints** | Predicting the future is not the ADL's job; retrospective assertion is honest. |
| **Decision-to-Decision causal links** | Too coarse; doesn't capture *which* outcome matters. |
| **Lumping all into `dependsOn`** | Loses the modal strength (MUST vs MAY) and the materialization bridge. |
| **Ontology terms as decisions** | Category error; terms are materializations, not decisions. |

## Consequences

**Positive.**
- **Causality is queryable.** The "lock file" problem is answered by graph traversal.
- **Agent collaboration is grounded.** Agents can surface specific constraints during deliberation.
- **Syllogisms are preserved.** Logical chains are explicit, not buried in prose.
- **Supersession is meaningful.** Stale constraints become detectable.
- **Reasoners can help.** Impact analysis and contradiction detection become possible.

**Negative / risks.**
- The graph becomes denser; requires tooling to surface relevance.
- Authors (or agents) must assert the links.

## References

- ADR-0004 — The KG Lives Under Tooling (agents as active participants).
- ADR-0005 — Log All Decisions (the real-time vignette, now expanded).
- ADR-0010 — Atom-first `Consideration` (the node that carries the causal link).
- ADR-0011 — Strict Graph (no Nygard body literals to hide constraints in).
- RFC 2119 — Requirement levels (MUST, SHOULD, MAY).
- ADR-0026 — Ontology provenance (the specific application of this model).

You are exactly right. I was treating the agent like a search engine ("let me find the document") rather than a graph-navigator. The real value of the network-of-atoms is **transversal**.
