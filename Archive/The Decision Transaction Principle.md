# The Decision Transaction Principle

## 1. The Atomic Transaction
A `DecisionRecord` is an **epistemic transaction**: it represents *the closed state of deliberation at the moment of commitment*.

> **A decision record is a transaction that commits to a path based on the information, constraints, and values available at the moment of decision.**

Every `DecisionRecord` has a temporal horizon defined by `adr-o:decidedAt` (t₀).
The record artifact itself carries separate editorial timestamps: `dcterms:created` (artifact birth) and `dcterms:modified` (most recent Tier 2 edit).

- **Inside the Horizon (t < t₀):** All information used to justify the decision. This includes context, concerns, alternatives, and the expected effects of the chosen path.
- **Outside the Horizon (t > t₀):** All consequences, realizations, and subsequent learnings, typically captured as `ObservedOutcome` artifacts.

**The boundary is impermeable.** Information from t > t₀ may never enter the `DecisionRecord`.

A `DecisionRecord` is **prescriptive**, never descriptive. It documents:
- What we decided we **will** do.
- Why we **believe** this is the right path.
- What we **expect** to happen.
- What we are **willing to accept** as a cost.

It does **not** document what *will* happen, or what *did* happen in the meanwhile, from the perspective of a future reader. That is a matter for later records.

## 2. The Ledger Metaphor
The ADL is a **ledger of commitments** which chronicles the, not a live feed of current developments:
- A ledger entry is written to capture a state of the world at a specific moment.
- The value of a ledger is *information preservation*:
  1. it encapsulates, freezes, and preserves all inputs and outputs which led to a decision in the past;
  2. it allows us to compare what was committed then against what is known now.

As such, you do not erase a ledger entry to "correct" it; you write a new entry that accounts for the correction. If you edit a decision's logic retrospectively, you are not correcting a record—you are destroying evidence.

## 3. Immutability and the Freezing Trigger

A `DecisionRecord` is mutable during its drafting and analysis phase. It becomes **immutable** when it is **referenced by another record** (the qualifier "another" does heavy duty here).

### The Freezing Rules

### ADR Scope References: No Freeze
**Self-references do not freeze.** A record referencing itself (e.g., in a internal cross-reference or a self-amendment loop if permitted) does not meet the threshold for immutability. Only references from **another** `DecisionRecord` create the referential dependency that locks the original.

### ADL Scope References: Freeze
A `DecisionRecord` is frozen the moment any other `DecisionRecord` asserts a relationship to R: `amendedBy`, `amends`, `conflictsWith`, `dependsOn`, `enables`, `supersededBy`, or `supersedes`.

### Project Scope Reference: Freeze
This extends to the **Project scope**: a `DecisionRecord` is also frozen when it is linked to a system artifact: `affects`, `justifiedBy`, or `materializes`.

Once a record is the stated justification for a piece of the system, it is no longer a draft; it is a premise upon which the system is built.

## 4. Tiers of Editability

Once a record is frozen, changes are restricted by tier.

### Tier 1: The Epistemic Core (Strictly Immutable)
These elements constitute the decision itself. Any change requires a **new `DecisionRecord`** that `amends` or `supersedes` the original.

- `adr-o:decidedAt` (the t₀ of the decision)
- `chosenAlternative`
- `hasContext` / `hasDeliberation` (the premises)
- `intendsToAchieve` / `accepts` (the commitments)
- `hasStatus` (the commitment state)

### Tier 2: The Presentation Layer (Mutable)
These elements are about *how* the decision is communicated, not *what* was decided. They may be edited in place.

- `dcterms:title` (clarification of name)
- `dcterms:description` (refining the summary)
- `skos:prefLabel` on considerations (consistency)
- Typos, grammar, and formatting

### The Amendment Path
A change to a Tier 1 property is not an "edit"; it is a **new decision**.
1. Create a new `DecisionRecord`.
2. Assert the new ADR amends the old one.
3. The original ADR remains in the ledger, unchanged, preserving the record of what was believed at t₀.

## 5. Post-Factum Records

A post-factum ADR (recorded at t > t₀) is still a transaction at t₀.

It's the ADR author's explicit responsibility to **reconstruct the epistemic state at t₀**:
- *"At the time of the decision, we knew X, Y, Z..."*
- *"We believed A would happen..."*

The fact that the author now knows tₙ is metadata (`dcterms:created` vs `adr-o:decidedAt`, plus later `dcterms:modified` if Tier 2 edits occur), not a license to inject future knowledge into the decision logic. The reconstructed t₀ record is then frozen upon publication or reference, just like any other transaction.

## 6. The Learning Delta

The gap between what was **intended** in a `DecisionRecord` (at t₀ via `adr-o:decidedAt`) and what was **observed** in the system (at tₙ via `ObservedOutcome` / `adr-o:observedAt`) is some of the most valuable information in the ADL.

By enforcing the transaction principle, ADR-O preserves this delta. When a later ADR supersedes an earlier one because a cost was higher than accepted, the graph explicitly contains:
1. The original commitment (what we accepted)
2. The subsequent realization (what actually happened)
3. The new decision (how we respond to that delta)
