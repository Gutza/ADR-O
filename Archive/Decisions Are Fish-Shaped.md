# Decisions Are Fish-Shaped

## The Shape of Decisions

**A well-formed decision record has the shape of a fish.** This is a structural claim about the logical anatomy of any decision worth recording: decisions begin narrow, they get formalized, then widen further into a space of deliberation, converge on a commitment, and then fan out again into consequences. That profile — point, expanding body, convergence, expanding tail — is the silhouette of a fish, and it maps precisely onto the lifecycle of a `DecisionRecord` up to t₀ when the record is frozen.

The fish shape is also a diagnostic. If your ADR doesn't have this shape — if it begins wide, or converges twice, or has no tail — something is structurally wrong with it, and the shape you ended up with will tell you where your problem lies.

## The Six Parts

### 1. The Snout — The Prime Mover

Every fish begins with its snout. Every ADR begins with a *prime mover*: the raw, pre-analytical impulse that caused the conversation to begin, and ultimately caused the decision record to exist.

The snout is what someone wrote on the meeting agenda. It is the ticket title, the Slack message, the complaint that became a meeting topic. It arrives in ordinary language, at ordinary resolution:

> *"Improve login time"*  
> *"Implementing MFA"*  
> *"The report export takes too long"*

The snout is not yet a need. It has not been sharpened, qualified, or made falsifiable. It is the *name of the situation* that caused deliberation to begin.

This makes the snout the most fundamental element of a `DecisionRecord`. It is the **cause of existence** of the ADR. Without it, there is no fish — there's no conversation, and no basis for asking why this decision record even exists in the first place. The snout is therefore the **only element that is unconditionally immutable**, independent of the freezing rules that govern the rest of the record. Those rules trigger when the record is referenced externally. The snout's immutability is prior: if the snout changes, not only is the decision different – it's a different conversation altogether; it's a different fish.

One fish, one snout. The snout is singular by design, not convention. This follows directly from ADR scope: a `DecisionRecord` is a single epistemic transaction about a single triggering situation. If you find yourself writing two snouts, you axiomatically have to draw two fish.

### 2. The Head — The Sharpened Need

The head is where analysis begins. It is the snout, translated.

The [ADR-O methodology](./ADR-O%20Methodology%20From%20First%20Principles.md) makes a sharp distinction between complaints and needs:

> *"CI is too slow"* is a complaint.
> *"Deployments must complete in under 10 minutes to support five releases per day"* is a need.

The head is where the snout undergoes that translation. Someone — or a group — takes the raw agenda item and asks: *what exactly must be true for this situation to be resolved?* The answer is the head: a falsifiable constraint, a metric, a security requirement, a performance bound. Something that can be satisfied, measured, and made the subject of a decision.

In ADR-O terms, the head is modeled as a `ContextFact` manifesting a `Claim` that represents the primary non-functional concern. It corresponds to the **Facing** clause of the [Y-statement](./Y-Statements.md): *"facing the need to..."*

The head is narrow — ideally, a single sharpened need. A decision that has five competing primary needs in its head has not been scoped; it has been stacked.

### 3. The Body — The Double Diamond

The body is the widest and the messiest part of the fish, and it's where most of the work happens.

The body encompasses everything between the sharpened need and the decision: constraints (legal, regulatory, technological, organizational, cost, etc), alternative approaches, brainstormed options, objections, trade-off analyses, pros and cons, dead ends, and the gradual convergence of the field of possible answers toward the one answer that will be chosen.

This is the *diverge-then-converge* structure of the [Double Diamond](https://en.wikipedia.org/wiki/Double_Diamond_(design_process_model)) methodology. First the body widens: you open up the solution space, collect constraints, generate alternatives, surface tensions. Then it narrows: you eliminate options, weigh trade-offs, and approach commitment.

The body is inherently *messy*, it's the relatively unstructured part of the decision that does not easily reduce to a tidy list: constraints interact, alternatives exclude each other, an option that satisfies the head may violate a regulatory constraint in the body. This is where the deliberation happens, and the record should make a reasonable attempt to capture it, while simultaneously accepting that it's a messy and lossy process.

In ADR-O terms, the body is modeled primarily through `DeliberationFact` nodes and the `hasAlternative` pool — the `Claim` nodes placed into deliberation roles, with valences that capture their argumentative direction. But the *shape* of the body — the opening of the solution space followed by its closure — is what the fish metaphor makes explicit that the ontology alone cannot.

#### The Body-Within-the-Body: Spawning New Fish

A critical process rule follows from the fish shape: if, while traversing the body of one fish, you discover that a second distinct decision is required, **you open a new ADR immediately** — mid-deliberation, mid-body.

You do not finish the first fish and then start the second. The second fish's snout is the moment of recognition inside the first fish's body: *"While discussing MFA implementation, we realized that session token expiry policy requires its own decision."* That recognition is a prime mover. It gets its own record, its own scope, and its own fish.

The two records can be linked — likely via `enables` or `dependsOn` — but the records remain separate epistemic transactions, each with a single snout, each with its own convergence point. This rule is not optional. Folding a second decision into the body of the first produces a record that converges on two answers, which is not a convergence at all — it's a tangle.

### 4. The Convergence Point — The Decision

The body terminates when deliberation converges to the decision itself. This is the moment the Double Diamond closes. Of all the alternatives generated and evaluated in the body, one is chosen. The others are explicitly named and set aside — not forgotten, but *neglected*, in the precise sense of the [Y-statement](./Y-Statements.md)'s fourth clause. The chosen alternative, the rejected alternatives, and the trade-offs accepted in making that choice are all recorded here.

The convergence point is the epistemic core of the `DecisionRecord`. It is the moment that defines t₀ — the moment the [Decision Transaction Principle](./The%20Decision%20Transaction%20Principle.md) treats as the sealed boundary of the record. Everything in the snout, head, and body is *input* to this moment. Everything after it is *output*.

In ADR-O terms: `chosenAlternative`, the un-chosen pool via `hasAlternative`, and the accepted costs via `DeliberationFact` nodes with negative valence.

### 5. The Tail — Expected Outcomes

The tail fans out from the convergence point: all the things that must happen, or are expected to happen, as a consequence of the decision.

The tail is directional in time, and it's oriented towards the future, towards events happening after t₀. Although the tail is not made of observations, it's the collection of *commitments about the future*, made at t₀ on the basis of what was known at t₀. The tail may include obligations ("we will need to implement a session database"), anticipated benefits ("authentication latency will fall below 400ms"), and accepted risks ("the session store becomes a single point of failure").

The fan shape is intentional. A single decision typically produces multiple expected outcomes, in multiple directions. Some are gains, some are costs, others are dependencies that will require their own future fish.

In ADR-O terms, the tail is the `ExpectedOutcome` cluster, with `outcomeValence` distinguishing gains, costs, risks, and dependencies.

The tail is still part of the epistemic transaction. It is part of what was believed at t₀, and it must remain as it was believed — even if a later tₙ reveals the beliefs were wrong.

### 6. The Wake — Observed Outcomes

Behind the fish, in the water it has passed through, is the wake: the observed outcomes that materialize at t > t₀.

The wake is not part of the fish, it's outside the `DecisionRecord`. It is the evidence that accumulates in the world after the fish has passed, after the epistemic transaction is sealed — captured in `ObservedOutcome` artifacts with their own timestamps, linked to the original record, but never written back into it.

The wake is where the [learning delta](./The%20Decision%20Transaction%20Principle.md#6-the-learning-delta) lives: the gap between what the tail expected and what the wake reveals. That gap is one of the most valuable things in an ADL. It is what allows an organization to say *"we decided X because we believed Y, and Y turned out to be false"* — and mean it precisely, with evidence.

Erasing the gap by updating the tail to match the wake is the defining act of a non-learning organization.

## Why This Shape Was Previously Implicit

The [Y-statement](./Y-Statements.md) captures five of the six parts: head (Facing), body (deliberation implied), convergence (Decided for / Neglected), and tail (To achieve / Accepting that). The wake is outside the Y-statement by design — it is post-transaction.

But the **snout is absent from the Y-statement entirely**.

The Y-statement begins *"In the context of [A]..."* — and [A] was always doing double duty, simultaneously naming the scope and gesturing at the triggering situation. The fish shape separates those two things. [A] in the Y-statement is the *head* — the sharpened need — not the snout. The snout was always there, motivating every ADR that was ever written, but it had no dedicated place in the model.

This omission has a practical cost. Without a modeled snout, an ADR's reason for existing is implicit. A reader who encounters a record must infer why this particular decision was worth having at all — what situation, whose concern, which moment caused it to open. That inference is lossy. Over time, as the organizational context shifts, the ability to make that inference degrades. The snout is where the ADR's *legitimacy* is recorded: the evidence that a real situation, named by real people, at a real moment, required a decision.

## The Shape as a Diagnostic

The fish is useful not just as a model but as a check.

A record with **no snout** has no traceable cause. It appears fully formed, without an origin. Ask: what actually caused this decision to exist? If you cannot answer in one sentence, the snout was never written down.

A record with **a wide head** — multiple competing primary needs — has not been scoped. The head should be narrow. If it is not, either the snout encompasses too much (it's two fish pretending to be one) or the translation from snout to head was never completed.

A record with **no body** — or a body that jumps immediately to the chosen alternative — has not deliberated. The alternatives were not genuinely considered; they were rationalized after the fact. The body should have guts.

A record that **converges twice** contains two decisions. Split it.

A record with **no tail** has not committed to expectations. It has recorded a choice without recording what was believed about that choice. It cannot generate a learning delta. It is useless to the future.

A record that **rewrites its tail to match its wake** has destroyed the most valuable thing it contained.

And a record with **no wake** is inconsequential, therefore it was useless to begin with.

## Summary

One fish, one snout, one convergence:

| Part | What it is | ADR-O model | Immutability |
|---|---|---|---|
| Snout | The prime mover; raw triggering situation | n/a | Unconditionally immutable as soon as the DecisionRecord is first committed to the ADL |
| Head | The sharpened need; falsifiable constraint | `ContextFact` / primary `Claim` | Tier 1 (frozen on reference) |
| Body | The double diamond; constraints, alternatives, debate | `DeliberationFact`, `hasAlternative` | Tier 1 (frozen on reference) |
| Convergence | The decision; chosen and neglected alternatives | `chosenAlternative`, `hasAlternative` | Tier 1 (frozen on reference) |
| Tail | Expected outcomes; forward-facing commitments | `ExpectedOutcome` | Tier 1 (frozen on reference) |
| Wake | Observed outcomes; post-transaction evidence | `ObservedOutcome` | Outside the record |