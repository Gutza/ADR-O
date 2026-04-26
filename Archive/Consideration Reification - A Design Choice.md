# Consideration Reification: A Design Choice

## The Three-Layer Architecture

ADR-O organizes decision data in three layers:

```
DecisionRecord 
  → [hasContext | hasDeliberation | hasExpectedOutcome] → Fact
    → [manifests] → Consideration
```

The `Fact` layer (ContextFact, DeliberationFact, ExpectedOutcome) is the reified link. It is the "placement" of a claim into a role.

The `Consideration` layer is the claim itself—an atomic observation, need, or effect.

## First Principle: Identity is Not Role

The fundamental distinction is this: **a claim is not the same thing as the use of that claim in a decision.**

Consider the claim: *"The system must handle 10,000 requests per second."*

In a single decision record:
- This claim appears in the **Context** as a *need* we are facing.
- This claim appears in an **ExpectedOutcome** as a *goal* we intend to achieve.

If we collapse Considerations into Facts, we have two different facts with the same prose. We can see they are similar via text, but the graph cannot assert they are the same *thing*. The claim's identity becomes a property of its role.

By reifying the Consideration as a first-class node, we decouple the claim from its placement. The claim has a stable IRI. The fact is just a pointer. This allows the graph to express: *"This decision is honest — the thing it claims to achieve is the exact same thing it claimed to be facing."*

## The "Close Call" in ADR-0028

During the design of ADR-0028, we came close to flattening this.

The temptation was real: in most ADRs, most considerations appear only once. Reifying them feels like overhead. We seriously considered collapsing `Consideration` into `Fact`, making the prose a property of the Fact.

The "close call" was resolved when we realized that the *exception*—the case where a consideration appears in multiple roles—is not an edge case. It is the only case that matters for the integrity of the decision.

If you can't structurally link the "Facing" clause to the "To achieve" clause, you can't programmatically verify that a decision is coherent. You can't ask: *"Does this decision actually address the need it claimed to address?"* The reification of Considerations is what enables this query.

## The DTP Constraint

ADR-0028 also taught us that identity is a liability across record boundaries.

If we use the same `Consideration` IRI across two different `DecisionRecords`, an edit to that atom silently mutates the premises of both decisions. This violates the Decision Transaction Principle: a record is a snapshot at t₀.

The solution—`derivedFrom`/`derives`—preserves the *semantic* link (this claim is the same as that claim) without the *identity* link that would allow silent mutation.

**Identity is a tool for intra-record coherence; reference is the tool for inter-record connectivity.**

## Conclusion

The three-layer structure is the minimal shape that respects three distinct categories of information:

1. The **Transaction** (`DecisionRecord`): the commitment and its metadata.
2. The **Role** (`Fact`): the placement of a claim in the decision's structure.
3. The **Claim** (`Consideration`): the atomic proposition being made.

Collapsing any of these layers into another would destroy one of these categories. The structure we chose is not a relic of any particular template format; it is the minimal graph that makes the internal logic of a decision machine-traversable.