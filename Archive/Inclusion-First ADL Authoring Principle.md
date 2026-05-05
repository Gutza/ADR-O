# Inclusion-First ADL Authoring Principle

This article is a lifted transcription of [ADR-0005](/ADL/ADR-0005-log-all-decisions.md), maintained as practitioner-facing guidance under ADR-0042.

## Context

Traditional ADR practice operates with a significance filter. In the most-cited formulation, ADRs are for "architecturally significant" decisions - typically glossed as decisions that are hard to reverse, affect many components, or carry significant risk. The filter is not arbitrary: it reflects a real economic constraint. Writing, reviewing, and maintaining a Markdown ADR in a repository has a non-trivial authoring cost. The markdown-in-folders medium imposes a minimum effort floor that rationally selects for high-stakes decisions and rationally excludes smaller ones.

ADR-0002 widened the filter along the domain axis - from "software architecture" to "any domain" - but explicitly left the significance criterion open as an unresolved question. The nominal expansion from "Architecture Decision Record" to "Any Decision Record" shifted what the filter applies to without changing the filter itself. This ADR builds on previous decisions to resolve it.

**Premise 1: ADR-0000 establishes the ADL as a machine-readable KG.** In a graph, decisions are nodes; their maintenance cost is proportional to what needs to change in the graph, not to how the record was originally authored. A small decision recorded as a handful of triples does not become a maintenance burden because it is small. The graph medium does not price brevity against verbosity the way a folder of Markdown files does.

**Premise 2: ADR-0004 establishes that tooling mediates all human interaction with the KG.** Under the expected deployment, an AI agent can draft a decision record from a design conversation in seconds; the human's role is confirmation and correction, not prose authorship from scratch. The authoring cost floor that motivated the significance filter collapses toward zero under this stack.

### The epistemic problem with load-bearing as an authoring gate

When the economic motivation for the filter dissolves, what remains is a convention in search of a principle. But there is a deeper problem than the missing economic justification. "Load-bearing" is a retrospective property of a decision: it describes how knowing the decision contributes to downstream deliberation or repair, and that contribution is visible only from the future looking back.

At authoring time, the author cannot know. Some of the most consequential decisions in a system's life start as unassuming defaults: a coding convention that spreads quietly through a codebase and later requires a major refactor; an implicit assumption about data shape that becomes architecturally load-bearing once three services depend on it; a one-line constant chosen "for now" that hardens into a contract before anyone notices. These are precisely the decisions that would be rejected at authoring time by a significance filter, and precisely the ones that need to be recovered retroactively for later deliberation (`git blame` wouldn't exist otherwise).

The **post-factum ADR** - the record written after a decision has revealed itself as consequential - exists because retrospective discovery of "load-bearing-ness" is a recognised pattern in ADR practice. Its existence is the field's admission that a forward-looking significance filter is always a guess about the future; the post-factum pattern is the guess's failure mode.

### The ambient-transcription vignette

Consider the directional end state that ADR-0004's tooling stance implies. In a mature deployment of the stack ADR-0004 commits ADR-O to - AI agents that draft records, APIs that enforce shape, MCP servers that expose structured tools, GUIs for navigation - the ADL ceases to be a document humans compose; instead, it becomes an ambient record that tooling instruments on their behalf. Imagine:

Spring 2032, a design meeting is in progress. Speech-to-text transcribes in real time; an AI agent consumes the transcript continuously and maintains the ADL as the conversation unfolds. John says *"hear me out - crazy idea, what if we all migrated to vim for development?"* Before the next sentence is spoken, the agent has instantiated a new `DecisionRecord` with `Status = Proposed`, a title synthesised from the proposal, and a "migrate to vim" `Alternative`. Alice responds *"no, our VSCode plugins are the whole reason onboarding works in a day"*; that turn becomes a `Consideration`, placed into a `DeliberationFact` with `Against` valence on the vim alternative. Bob adds *"and the retraining cost would be weeks per engineer"*: another `Consideration`, another `DeliberationFact`. John concedes within five minutes; the agent transitions the record to `Status = Rejected` and the meeting continues.

That five-minute deliberation, which would never have produced a file in a markdown-in-folders ADL, is now a permanent graph node. Three years later, when some external change (a licensing shift, a tooling deprecation, a security advisory) forces the team to revisit editor choice, the graph returns the 2032 deliberation instead of the team running the discussion cold. The record's existence was cheap; its future utility is a free option on information that would otherwise have evaporated. Meanwhile, day-to-day query surfaces that excavate "current decisions affecting this service" simply do not return the vim record - selective-surfacing tooling (trivial-tagging, relevance ranking, scope filters) can efficiently separate noise from signal in a dense ADL, and none of that work has to happen at authoring time.

This scenario is not speculation about arbitrary AI capabilities. It is entailed by ADR-0004's commitment to a tooling-mediated substrate: once agents draft, APIs enforce shape, and MCP servers expose the graph, the ambient-authoring case follows as a natural deployment. We remain in charted waters. The tooling is not yet built, but the ontology and the authoring conventions must admit this creation model without restructuring when it is.

We will coin term **real-time ADR** to mean *records created by tooling at or near the moment of deliberation*, which have the corollary property that *no human author decided that this particular decision was worth capturing*. Real-time ADRs and post-factum ADRs are the two limiting cases of the same authoring model - the one where the significance judgement is replaced by completeness of ambient record plus selective surfacing. Most records in a pre-ambient deployment will sit between the two limits.

### The cost asymmetry

Authoring cost, already low under ADR-0004's expected deployment, is approaching zero in the real-time ADR limit.

By contrast, the omission cost is high and deferred. An unrecorded decision is invisible to graph traversal. A future decision that would have cited an unrecorded one as context, flagged a conflict with it, or depended on it cannot do so. The decision graph is sparser than the actual history of deliberation; queries return incomplete answers; agents operating over the graph reason from an incomplete model. The cost is paid not at authoring time but at every future point where the missing node would have been relevant. This is the KB equivalent of technical debt.

Retrospective recovery (post-factum ADRs) are possible but lossy. The deliberation context is stale, alternatives are harder to reconstruct honestly, and the original authoring moment's reasoning is already gone.

Under this asymmetry, the rational authoring policy is to default to inclusion. Exclusion requires positive confidence that a decision is outside the project's decision domain - Janet's pink dress, John's mustard at lunch. Within the domain, log.

### The entry-gate metaphor is the thing to reject

In markdown-in-folders, the entry gate is load-bearing infrastructure. It has to be: every record costs reviewer time and directory noise, and every record surfaces with equal visual weight in a file listing. A significance filter is how that medium scales.

In a tooling-mediated graph, the entry gate is a vestigial metaphor. Record creation is cheap; tooling handles selective surfacing (demotion, trivial-tagging, query-layer filtering); and the cost asymmetry runs the other way. A false positive costs a tagged-trivial node that everyday query surfaces ignore by default. A false negative costs a missing node that later deliberation has to reason without.

Framed with ADR-0004's **reconstructability boundary rule**: relevance ranking, demotion, and "hide trivial unless asked" views are representation concerns that reasonable tooling can reconstruct from a dense graph, so they should stay out of the ontology shape. What cannot be reconstructed after omission is the decision node itself; that is why this ADR places the default on logging in-scope decisions.

The traditional filter transplanted onto the new medium is not a refinement; it is a convention inherited from a different medium whose economics no longer hold.

### The relationship to ADR-0002

This ADR expands ADR-0002 orthogonally. ADR-0002 addressed the domain axis: not "software architecture only" but "any domain." This ADR addresses the magnitude axis: not "only consequential decisions" but "all decisions within scope." The two expansions are independent - accepting one does not entail the other - but they are mutually reinforcing: a domain-agnostic, magnitude-inclusive ADL is more useful as a cross-context deliberation record than either expansion alone.

## Decision

**The ADL has no entry gate beyond domain relevance.** All decisions within the project's deliberative scope should be logged, regardless of their apparent magnitude at authoring time. The only exclusion is what the author can confidently rule out as outside the project's decision domain. When in doubt, log.

Three corollaries make the policy operational:

**Load-bearing-ness is the ontological scope of ADR-O, not the authoring criterion.** ADR-O models load-bearing decisions - that is what the ADL ends up containing, visible from the future looking back at what actually mattered. But that is not something the author is able to predict at decision time. Conflating the two is the specific mistake this ADR corrects: the forward-looking version of the test is always a guess, and the cost asymmetry argues the guess should default to inclusion.

**Post-factum ADRs are first-class.** A decision discovered to be load-bearing only in retrospect - the quietly-spread coding convention that becomes a refactor, the implicit default that locks in architecture - gets a record written after the fact. Such records carry the same ontological shape as any other; they differ only in their temporal relationship to the decision moment. Recording a post-factum ADR is not an admission of earlier failure; it is the ADL operating as designed under the retrospective-discovery case.

**Real-time ADRs are first-class.** Under the tooling stack ADR-0004 commits ADR-O to, records may be instrumented in real time - an AI agent filing a `Proposed` record before any human author has consciously decided the proposal is worth filing, and resolving the status as deliberation unfolds. The vim vignette above is not an edge case and not a reductio. It is the target creation model for a mature deployment. The ontology is designed to accommodate this; ADR-0005 codifies that the authoring conventions treat it as normal, not aberrant. A five-minute proposal-and-rejection cycle produces a valid record; its triviality for day-to-day query purposes is a tooling surface concern, not an ADL-entry concern.

**This is not a mandate to record arbitrary trivia, and the domain filter still applies:** decisions outside the project's deliberative scope (personal choices, chat banter, unrelated observations) do not belong in the ADL. Within scope, however, the threshold is low: if the decision might plausibly inform a future one, err towards logging it. Selective surfacing of what matters day-to-day is a tooling responsibility, per ADR-0004's push-incidental-complexity-to-tooling heuristic, not a burden the ADL absorbs through upfront filtering.

### Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Retain the traditional significance filter with an updated gloss | The filter was economically motivated, and that motivation has dissolved. Retaining it produces an arbitrary rather than principled threshold - a convention inherited from a different medium applied to a medium where its rationale no longer holds. |
| Retain a load-bearing entry gate with a lower threshold | The interim version of the fix that kept "load-bearing" as the authoring gate and merely lowered the bar. Rejected because load-bearing-ness is a retrospective property; any forward-looking gate is a guess, and the cost asymmetry argues the guess should default to inclusion. A lower-but-extant gate keeps the entry-gate metaphor alive and reintroduces the same authoring-time judgement burden the rest of the analysis dissolves. |
| Introduce a formal `adr-o:significance` property to encode the threshold machine-readably | A significance property remains potentially useful as a relative annotation on already-recorded decisions - for prioritisation, filtering, or navigation in a dense ADL - but it addresses a different question from what gets recorded in the first place. The gate this ADR addresses is prior to any annotation. |
| Leave the question open (status quo from ADR-0002) | The open question in ADR-0002 noted the missing premises. ADR-0000 (machine-readable KG) and ADR-0004 (tooling mediation) now supply them. Continued deferral would be failing to close a question whose conditions for closure have been met. |

## Consequences

**Positive.**
- The KG's decision graph approaches completeness with respect to actual deliberation history. Graph traversal and agent reasoning operate over a denser and more honest substrate.
- Small decisions - which often explain constraints that later appear inexplicable - are captured where previously they would evaporate between sessions or get buried in commit messages and chat logs.
- The ADL's value compounds over time: a dense decision history is more useful for AI-assisted review than a sparse one, because agents can recover not just the high-level choices but the granular deliberation that shaped them.
- The ontology's design and the authoring conventions accommodate real-time ADRs and post-factum ADRs as first-class creation models, not special cases. Future tooling that instruments ADLs directly from meeting transcripts, or that back-fills decisions discovered to be consequential after the fact, does not need to restructure the ontology or the conventions to do so.
- The open question on `adr-o:significance` from ADR-0002 is partially resolved: the property, if introduced, should annotate relative importance *within* the ADL, not gate entry to it.

**Negative / risks.**
- Volume management: a lower threshold produces more records. This is reframed here as a tooling responsibility - selective surfacing, trivial-tagging, and query-layer demotion are the mechanisms that keep a dense ADL navigable - rather than as a concern the ADL absorbs through upfront filtering. Until that tooling exists, human navigation of a dense ADL will be coarser than it will eventually be.
- Forward-compatibility dependency: the strongest version of this ADR's argument - the real-time ADR end state - rests on tooling that ADR-0004 commits the project to envisioning but has not yet built. The waters are charted (ADR-0004 names the tooling stack explicitly), but the present-day version of the policy leans on the forgiving cost asymmetry rather than on tooling completeness. In the pre-ambient transition period, authors apply a best-guess inclusion heuristic; the heuristic is forgiving but still a judgement, and coverage across authors may be uneven until tooling assists the judgement directly.
- Naming real-time ADRs as first-class writes a check that tooling has not yet cashed. This ADR does not commit the project to building that tooling on any timeline; it commits the ontology's design and the authoring conventions to admit it without restructuring when it is built.

**Open questions.**
- Whether `adr-o:significance` as a relative-importance annotation (not an entry gate) has value for navigation in a dense ADL; deferred to a future ADR.
- Whether post-factum ADRs and real-time ADRs deserve distinct provenance markers beyond being ordinary records - a status-scheme addition, a `dcterms:provenance` pattern, or similar. Currently treated as indistinguishable from contemporaneously hand-authored records; if future tooling needs the distinction, it can be introduced non-breakingly.
- What authoring conventions or tooling prompts best operationalise the "default to logging" discipline in practice - an empirical question that will be answered by use.

## References

- ADR-0000 - Inception Record. Establishes the machine-readable, graph-traversable KG as the ADL substrate; the low-maintenance-cost premise argued here depends on this foundation.
- ADR-0002 - Scope. Introduces the "Any Decision Record" expansion along the domain axis and explicitly leaves the significance criterion as an open question (see "Open questions" in ADR-0002). This ADR resolves that open question along the orthogonal magnitude axis.
- ADR-0004 - The KG Lives Under Tooling. Provides the tooling-mediation premise that collapses the authoring cost floor, grounds the real-time ADR end state as a charted-waters extrapolation rather than speculation about arbitrary AI capabilities, and supplies the push-complexity-to-tooling heuristic under which volume management is a tooling responsibility rather than an ADL-level concern.
