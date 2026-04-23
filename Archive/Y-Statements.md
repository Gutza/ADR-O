# ZIO's Y-Statements — What They Are and Why They Matter

**ZIO** stands for *Zürich, IBM, and Open-source* (the professional trajectory of Dr. Olaf Zimmermann, who coined the format). A **[Y-statement](https://medium.com/olzzio/y-statements-10eb07b5a177)** is a six-part, single-sentence template for capturing an Architectural Decision (AD) — named "Y" because the acronym is pronounced like the English word *"why,"* and because the visual layout of the template elements literally maps to the shape of the letter Y.

## The Letter Y as a Decision Diagram

Picture the letter Y. It has three structural parts: a left branch, a right branch, and a single stem descending from where they meet. Here's how that maps to the problem space:
- The **left branch** represents the *context and problem*: the use case you're in and the non-functional concern you're facing. These are the two inputs that converge into the decision point;
- The **right branch** represents the *options considered*: the chosen option on one side, the rejected alternatives on the other. These are the two roads that diverged — you took one and left the others;
- The **junction** — the center of the Y, where all branches meet — is the decision moment itself. Everything above and to the sides is input; everything below is output;
- The **stem** represents the *outcome flowing downward*: the quality or benefit you aimed to achieve, and the downside you accepted as a consequence. The decision "flows through" the junction and produces these results below.

## The Six-Part Structure

A Y-statement fills in this sentence skeleton:

> *In the context of **[use case / component]**, facing **[non-functional concern]**, we decided for **[option chosen]** and neglected **[options rejected]**, to achieve **[quality / benefit]**, accepting that **[downside / consequence]**.*

Each clause maps to the Y shape:

1. **Context** *(left branch, upper)* — The functional scope: a use case, user story, or architectural component.
2. **Facing** *(left branch, lower)* — The non-functional concern or force that triggered the decision.
3. **We decided for** *(right branch, one side)* — The chosen option. The most important part.
4. **And neglected** *(right branch, other side)* — The alternatives considered but not chosen. Keeping these explicit prevents the same debate from recurring.
5. **To achieve** *(stem, upper)* — The benefits or quality properties gained; the payoff that justifies the choice.
6. **Accepting that** *(stem, lower)* — The known downsides and trade-offs accepted as a result.

## Concrete Example

> *In the context of the Web shop service, facing the need to keep user session data consistent and current across shop instances, we decided for the **Database Session State** pattern and against Client Session State or Server Session State, to achieve data consistency and cloud elasticity, accepting that a session database needs to be designed and implemented.*

## Why This Format Is Useful for an AI Agent

A Y-statement is a **minimal, complete, machine-parseable decision record**. For an agent working within ADR-O, each clause maps naturally to ontology concepts:

- **Context** → the `addresses` predicate target (a requirement or component IRI)
- **Facing** → a `ContextFact`-linked `Consideration` (a non-functional concern)
- **We decided for** → `chosenAlternative`
- **And neglected** → `hasAlternative` nodes *not* selected as `chosenAlternative`
- **To achieve** → `OutcomeFact` nodes with a positive valence
- **Accepting that** → `OutcomeFact` nodes with a negative valence (or `DeliberationFact` nodes flagging known costs)

In other words, a Y-statement is essentially a **human-readable projection** of a `DecisionRecord` subgraph in ADR-O. An agent can parse a Y-statement and hydrate the corresponding triples, or inversely, render a `DecisionRecord` graph as a Y-statement for human consumption.

## Key Design Philosophy

Y-statements deliberately trade completeness for adoptability. They omit process metadata (who decided, when, by what authority), versioning, and inter-decision links — all of which ADR-O *does* model. Think of a Y-statement as a **lossy but human-friendly serialization** of the core rationale slice of an ADR-O record: the part that answers *why* a decision was made, which is also the part most at risk of being lost over time.