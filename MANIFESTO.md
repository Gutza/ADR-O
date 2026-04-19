# The ADR-O Manifesto

*Decision Records deserve to be more than files.*

## What is lost

Every non-trivial system carries within it a hidden history: the arguments that were made and the arguments that were rejected, the constraints that were obvious to the room but never written down, the moments where the team changed its mind and the forces that moved it. This history is the true specification of a system — more explanatory than any diagram, more honest than any README.

Today, almost all of it is lost.

What survives is a partially-truthful fossil record: markdown files in a `doc/adr/` folder, if the team was disciplined; scattered comments, intra-team messages on Slack or Teams, and git commit messages otherwise. The knowledge that didn't make it into prose — the road not taken, the trade-off reasoning, the assumption that seemed too obvious to state — evaporates the moment the meeting ends.

The next generation of developers inherits the decision baked into the system without the deliberation that produced the system. Chesterton's Fence (*do not remove a fence until you know why it was built*) applies in full – but how do you know why the fence was built if the builder left no record of the wolves? The developers who follow learn *what* was decided but not *why*, and so they cannot know when the reasons have changed. They either tear down fences that were load-bearing or they adopt a cargo cult maintenance mindset. They repeat mistakes that were already made, relitigate questions that were already settled, break constraints they didn't know existed, or preserve atavistic artifacts which incur costs but create no value.

This is not a documentation problem. It is a representation problem.

## What we build

ADR-O is a formal ontology for journalizing decisions. Its scope is deliberately narrow: a single decision, its context, its alternatives, its rationale, and its typed relationships to every other decision in the graph. That is the atom. ADR-O does not try to model all of the team's knowledge — only the unit from which all knowledge that informed the project can be composed.

Consider the neuron. In itself it is almost unremarkable — a cell that fires or does not. What makes it potent is the network: the density of connections, the patterns of activation, the emergent intelligence that no single neuron contains or even hints at. ADR-O is the specification of the atom. The graph is what becomes possible once the atom is right.

The graph can answer questions a folder of markdown files cannot: which decisions constrain this choice? Which past choices conflict with this proposed change? What was the reasoning that led the team away from the approach now being reconsidered? What is the full supersession chain from the founding decision to what is in effect today? These are not keyword searches against prose — they are traversals of an explicit, typed graph, returning consistent answers every time.

ADR-O does not build from nothing. It sits on top of the semantic web ecosystem that the linked data community has spent decades refining — drawing on established vocabularies for provenance, metadata, and controlled concept schemes, and contributing only what is genuinely new: the typed relationships between decisions, the first-class representation of the road not taken, the vocabulary that turns a decision record from a paragraph in a file into a node in a living graph.

## Why now

Kruchten articulated the ontology of architectural decisions in 2004. The concepts were right. The relationships he named — *enables*, *conflictsWith*, *dependsOn*, *supersedes* — are still the right relationships. What was missing was not insight but infrastructure: the tooling, the adoption, the moment.

The moment is now.

In the age of AI, a machine-interpretable graph of decisions is not an academic artefact. It is operational infrastructure. Developers are already feeding their ADR folders to language models — pasting files into context windows, running retrieval over markdown, asking for summaries. They are doing it in the lossiest possible way, asking a model to reconstruct the decision graph from prose, every time, with no consistency guarantee.

ADR-O makes the graph explicit and durable. An agent navigating a codebase with an ADR-O graph does not infer relationships — it traverses them. The `supersedes` edge is a triple, not an implication buried in paragraph three. The rejected alternative and the reason for its rejection are first-class citizens, not a footnote that someone may or may not have thought to include.

The loop that has never been closed can now be closed. Meeting minutes become ADRs, logged by an AI that understands ADR-O. Those ADRs are implemented by agents who navigate the decision graph from any viewpoint, any concern, any layer of the stack. The intellectual history of the system becomes a resource that compounds — every decision enriching the context for every future decision.

This is not a research paper. This is a substrate.

## The north star

A new developer joins a project. They ask: *why does this system look the way it does?*

They do not get a shrug. They do not spend three hours in git blame. They get a coherent narrative — the reasoning, the alternatives that were considered and discarded, the forces that shaped each choice, the moments where the team changed its mind and why — traversable from any angle, queryable by any concern, legible to both human and machine.

The entire intellectual history of the system, preserved and alive.

That is what ADR-O is for.

*ADR-O is released under the Creative Commons Attribution 4.0 International licence.*
*Namespace: `https://w3id.org/adr-o#`*
