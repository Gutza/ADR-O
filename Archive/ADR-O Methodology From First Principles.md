# ADR-O Methodology From First Principles

## The Pathology of Complaint-Driven Design

Most ADR practices begin with a complaint: *"Our CI pipeline is too slow,"* *"The API is hard to use,"* or *"I don't like how we handle sessions."*

This seems natural, but it is a category error. A complaint is an observation of a symptom, not a decision context. When we record a complaint as the "Context" of an ADR, we have already surrendered. We are reacting to a feeling rather than responding to a constraint.

The real work of architecture happens in the translation from *complaint* to *need*.

> "CI is too slow" is a complaint.
> "Deployments must complete in under 10 minutes to support five releases per day" is a need.

The latter is a falsifiable claim. It can be satisfied, it can be measured, and it can be the subject of a decision. The former is a mood.

## The Principle: Decision as Transaction

A decision is not a Word document, it is an **[epistemic transaction](./The%20Decision%20Transaction%20Principle.md)**.

At a specific moment in time ($t_0$), a set of actors with a specific information set reached a commitment to a path. That moment is the transaction. Everything that happens after ($t_n$) is **verification** or **consequence**, not part of the decision.

A common temptation (and frequent failure mode) in ADR practice is *retrospective smearing*: updating a decision record with what we later learned. When we change "we expect 100ms latency" to "we achieved 38ms latency," we are not correcting or improving the accuracy of a record; we are destroying information regarding a past transaction.

The gap between what was expected at $t_0$ and what was realized at $t_n$ is where organizational learning lives; when you erase the gap, you erase the learning.

## The Y-Statement as the Irreducible Unit

If a decision is a transaction, what is the minimum viable record of that transaction? The [Y-statement](./Y-Statements.md) provides the answer:

> *In the context of **[A]**, facing **[B]**, we decided for **[C]** and neglected **[D]**, to achieve **[E]**, accepting that **[F]**.*

This is not a template; it is a logical form. Each clause performs a distinct epistemic function:

1.  **Context [A]**: The boundaries. What is this decision *about*?
2.  **Facing [B]**: The need. What is the *falsifiable constraint* we are aiming for?
3.  **Decided for [C]**: The commitment. What path are we taking?
4.  **Neglected [D]**: The opportunity cost. What did we reject, and why? (This is frequently omitted, leading to information loss.)
5.  **To achieve [E]**: The intended outcome. What do we believe this will produce?
6.  **Accepting that [F]**: The explicit trade-off. What downside are we knowingly incurring?

If any clause is missing, the decision is incomplete. If any clause is "TBD," the transaction hasn't happened yet.

## The Discipline of the Loop

The most important structural property of a decision is the **feedback loop between [B] and [E]**.

The "Facing" clause (B) and the "To achieve" clause (E) should, in a coherent decision, refer to the same consideration.

> **Facing:** "...the need to keep user session data consistent..."
> **To achieve:** "...data consistency..."

When these two clauses manifest the same consideration, the decision is honest: it is solving the problem it set out to solve. When they diverge, the decision has either pivoted or is delusional.

Most ADRs never make this explicit. They state a problem, then state a solution, and the reader is expected to infer the connection. The discipline of the Y-statement makes the connection a first-class claim that can be questioned.

## The Graph as Enforcer

The reason ADR-O reifies `Consideration` as a node rather than prose is not for the sake of the ontology, but for the sake of the practitioner.

When "consistency" is a node, the graph can tell you:
- This consideration was a *need* in ADR-0042.
- It was an *expected gain* in ADR-0042.
- It is now a *constraint* on ADR-0115.
- It was *rejected* as a goal in ADR-0087.

This is not "documentation." This is a **trace of reasoning**.

Reification prompts the author to ask: *"Am I introducing a new consideration, or am I referring to an existing consideration instance?"* The very act of repeating that check enforces discipline and reduces drifting terminology and duplicated reasoning, two patterns that make large ADLs hard to read.

## The Moral of the Transaction

The hardest part of this methodology is the refusal to be "accurate" in the way people usually mean.

Your ADR will contain "wrong" predictions. It will contain "naive" expectations. It will contain "incorrect" understandings of the options.

**Leave them there;** resist the urge to retroactively "fix" the decision record log as to become a perfectly accurate representation of history *in hindsight*. A proper decision record is a snapshot of what was known and believed at the moment of commitment, warts and all.

The value of an ADR is not in the correctness of the decision — many correct decisions are made for the wrong reasons, and many wrong decisions are made for the right reasons. The value is in the **traceability of the reasoning**.

When you can say, *"We decided X because we believed Y, and Y turned out to be false,"* you have a learning organization. When you can only say *"We decided X, and it works,"* you have a lucky organization.

The difference between the two is whether you had the discipline to record the transaction, or whether you only recorded the result.