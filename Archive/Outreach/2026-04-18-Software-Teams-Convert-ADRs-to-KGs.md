---
reviewer notes: See `2026-04-18-Software-Teams-Convert-ADRs-to-KGs.notes for the editor.md`
---

# Software Teams, Convert your ADRs to Knowledge Graphs
# *Low-hanging fruit: from markdown ADRs to queryable graph in one day*

Your **Architecture Decision Records** (*ADRs*) are already semi-structured and relatively uniform across a single repo. Converting them to a machine-readable knowledge graph can be done relatively painlessly using modern LLMs, and a good knowledge graph yields emergent value way beyond the information it contains. Give me 10 minutes to convince you this is worthwhile.

## The State of the Art

Wikipedia's collaborative editorial process is not conducive to producing memorable quotes—but [suffering transcends process](https://www.reddit.com/r/physicsmemes/comments/dx9y72/the_opening_paragraph_to_goodsteins_textbook/):

> Software architecture design is a [wicked problem](https://en.wikipedia.org/wiki/Wicked_problem), therefore architectural decisions are difficult to get right. – [anonymous](https://en.wikipedia.org/w/index.php?title=Architectural_decision&diff=prev&oldid=735828233), _[Architectural decision](https://en.wikipedia.org/wiki/Architectural_decision)_

Not only are architectural decisions difficult to get right, but the industry is prone to the infamous "build it now, fix it later" mindset under delivery pressure. The final, tangible deliverable is the binary, which is built from source, which is written per the implementation spec, which is derived from the functional spec... magically!

It's not that the team doesn't understand what they're doing while they're doing it; the illusion of a magical black box step appears retroactively. More often than not, the road from "what we need to accomplish" to "code passes tests" is a blur after the fact: we remember the original elevator pitch (A) and we know the product we ended up with two years later (B), but there's an organizational knowledge chasm between A and B—two manifestations of what should be the same thing. We need a shared, machine-readable way to relate decisions, alternatives, and concerns. I propose **ADR-O**, a formal ontology for ADRs as a natural way to close that gap.

Software development teams have long since recognized this particular pain point, and many adopted ADRs enthusiastically in the wake of Michael Nygard's seminal 2011 [blog post](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) popularizing the concept (if you're interested in the history of ADRs I highly recommend Michael Keeling's wonderful [Love Unrequited: The Story of Architecture, Agile, and How Architecture Decision Records Brought Them Together](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=9801811) from 2022). The way people implement this today is by writing text or markdown files (these are the ADRs) documenting technical decisions; the collection of all ADRs is the project's **Architecture Decision Log** (*ADL*).

ADRs are the standard solution to the original challenge—but they're not enough. ADRs encourage you to turn decisions into a running log of artifacts documenting the journey, and that is the crucial first step in capturing the *why* and the *how* of the transition from functional specs to code; without ADRs, that initial moment is lost forever. What ADRs don't do so well is the indexing required for recollecting information after the fact. Even if the information is present *somewhere* in the ADL, **a repository of text files is something of a [one-way function](https://en.wikipedia.org/wiki/One-way_function)**: it's easy to start from an ADR and see how those decisions manifest in the code, but it's much harder to start from something in the code/database/architecture and find the relevant ADR—and real life needs are even more challenging than that.

In practice, an ADL has to answer several orthogonal classes of questions:
- **Targeted questions**: why has that been implemented this way? Why was the data organized this way? What is this synthetic column in the database for?
- **System constraints/limitations**: what are the soft/hard *emergent* boundaries of size, performance or complexity that have accrued in the system for a particular type of concern? (Number of concurrent requests, users, record size, posts per user, number of username changes for denormalized data, etc.)
- **Unused capacity**: the opposite of the previous question: where did we provision too much capacity which incurs costs but is currently unused?
- **Filtering by area of responsibility**: what security risks have we accumulated? Which regulatory gaps did we gloss over? What cost reduction opportunities did we choose to miss for development speed? What are the operational inefficiencies we accepted "temporarily"? Why do we have to respect this particular procedure when we interact with that system?
- **Filtering by system**: how did we end up with this particular set of hardware needs? Or software needs? How did we choose this technological stack? Georedundancy configuration? Cryptographic algorithms? Software frameworks, libraries, tooling, logging, IDE, the list is endless.
- **Organizational history**: this might feel academic, but organizations/teams/projects sometimes flip-flop between option A and B simply because there's no organizational memory to recollect that *we tried that before, here's what happened*. An ADL which makes it easy to recount the history of past decisions would naturally solve this class of frustrations.

Note how targeted questions are answerable using full-text search or standard RAG techniques, others need labeling (filtering by system or area of responsibility), history traversal (organizational history), or natural language processing (constraints or unused capacity). For flat markdown files in a folder, full-text search and RAG work well, but all of the others require cumbersome conventions baked into freeform markdown files. For a graph database, those are just natural features of the medium.

## Introducing ADR-O

**[ADR-O](https://w3id.org/adr-o)** is a formal [OWL 2](https://www.w3.org/TR/owl2-overview/) ontology for ADRs. Its scope is deliberately narrow: a single ADR, the context that produced it, the alternatives considered (including the alternatives that were rejected, which become first-class nodes rather than footnotes), the rationale, and the typed relationships this decision bears to every other decision in the log. That last set is a small, principled vocabulary: `supersedes`/`supersededBy`, `dependsOn`, `enables`, `conflictsWith`, `seeAlso`. Little more is needed to express the structure that already lives, implicitly and unreliably, inside a well-run text ADL.

That vocabulary is not chosen at random—it maps cleanly onto the question taxonomy above. *Targeted* questions remain targeted: they always worked, and they still do. *Filtering by area of responsibility* and *filtering by system* become lookups against nodes labelled by concern or affected component, where the labels are entries in a formally controlled vocabulary defined by your team, rather than strings that one ADR spells "auth" and the next ADR spells "authentication". *Organizational history*—the flip-flop problem, where the team risks relitigating a question it had already settled—becomes a graph traversal along `supersedes`/`supersededBy`, visiting every rejected alternative on the way. *System constraints* and *unused capacity* hang off the same typed edges, pinned to the decisions that introduced them, and are therefore retrievable by walking into a node rather than by grepping across a folder.

The point to land is this: **the ontology is not adding complexity, it is removing ambiguity**. The relationships were always there—buried in paragraph three of an ADR, implied by a "(see also ADR-0042)" aside, lost to the reorg that moved the referenced file. ADR-O turns them into edges, and the ADL becomes a graph you can actually traverse.

## Consuming the Graph

Once the graph exists, the interrogation layer that sits on top of it is an ontology-aware graph RAG [MCP](https://modelcontextprotocol.io/). Humans and agents both query through the same interface, and the six question categories become six natural kinds of traversal rather than six conventions baked into freeform markdown.

Targeted questions resolve the way they always did, but now the answer arrives with its graph neighbourhood attached: "what was this synthetic column meant to do" comes back with the concern it addresses, the alternatives that were rejected, and the supersession chain behind the current state. System and responsibility filters are predictable, machine-generated SPARQL-level lookups against controlled concepts. Organizational history is a walk along `supersedes`/`supersededBy` that returns the chain together with the rejected alternatives on every step. Constraints and unused capacity sit on the nodes they belong to and come back the same way.

The distinction between an ontology-aware MCP and a generic RAG pipeline matters. A generic pipeline reconstructs the decision graph in the LLM's context window from text confetti on every query, at every hop, with no consistency guarantee: the same question can return different shapes of answer depending on retrieval jitter. The ontology-aware MCP traverses meaningfully typed edges and returns the same shape every time. And the implicit, emergent relationships—the ones nobody thought to encode at conversion time—do not require another pass: they are discoverable at query time against the full graph context. The structure enables what was never explicitly encoded.

## Updating the Graph

Speech-to-text transcribes the design meeting. The transcript is handed to an LLM ADR-O writer service, which understands the transcript and—before it writes a single line—consults the existing graph through the same MCP. The new decision is therefore born with full context: it knows which prior decisions it touches, what the existing supersession chains look like, which concerns are already tracked and which ones are new.

That context allows the LLM to keep the entire graph consistent at all times. If the new decision supersedes an earlier one, the writer records `supersedes` in one direction and `supersededBy` in the other, and transitions the prior ADR's status from `Accepted` to `Superseded`. If the new decision introduces a dependency, the corresponding `dependsOn` edge is written. If it enables something downstream or conflicts with a prior position, `enables` and `conflictsWith` are wired in as triples rather than as promises in prose. The graph remains consistent by construction—a guarantee no folder of markdown files can give you, at any level of team discipline.

And the interface? A single CRUD MCP over the [RDF triplestore](https://en.wikipedia.org/wiki/Triplestore). Creation, conversion, consumption, ongoing maintenance—an ADR writer working from a meeting transcript, a human editor making a manual correction, an agent pulling context for an unrelated task—everything talks to the graph through the same primitive.

## The Bootstrap

Notice what we have not described: how your existing markdown ADRs are converted to a graph ADL in the first place—and that's because there's nothing to describe. The CRUD primitive from the previous section does not care whether its input is a meeting transcript or the markdown body of an ADR written in 2019. Feed it the historical log in chronological order (ADR-0001 first, then ADR-0002 with ADR-0001 already in graph context, and so on) and it writes [RDF triples](https://www.w3.org/TR/rdf11-primer/) in [Turtle](https://www.w3.org/TR/turtle/) the same way it will every day after.

ADR-0001 lands with no prior context—which is exactly the context its original author had. Each subsequent record then completes the graph that already contains its predecessors, so the contextual relationships the author *implied but did not spell out*—the dependency on a datastore chosen three months earlier, the constraint accepted in a security review, the position now being quietly walked back—can even land as typed edges at conversion time, not only at query time. [OWL 2 consistency checking](https://www.w3.org/TR/owl2-overview/) runs after each insertion, exactly as it would in steady state. A few dozen ADRs convert in an afternoon.

The elegance is in what is not there. There is no separate bootstrap. No migration tool to keep current with the production writer. No second schema for the importer. No risk that a converter's interpretation of the ontology drifts, six months in, from the writer's. One ontology, one MCP, one CRUD primitive, one workflow. The bootstrap and the steady state are the same system, run once on yesterday's prose and then daily on tomorrow's transcripts.

The promise in the title is no longer a project plan. It's just what happens organically when you point the system at your existing ADL.

## The Vision
A new developer joins the team. They open the codebase and ask the question every new developer asks: *why does this system look the way it does?*

They do not get a shrug. They do not spend a week with `git blame`. They get a coherent narrative—the reasoning, the alternatives that were considered and discarded, the forces that shaped each choice, the moments the team changed its mind and why—traversable from any angle, queryable by any concern, legible to both human and machine.

That is the view from the top of the stack you just built. Every step from here to there was incremental: an ontology in place of markdown convention, a one-pass LLM extraction into triples, an ontology-aware MCP over a triplestore, the same MCP carried forward into every new ADR. No single component was heroic. The result is the one worth wanting—the intellectual history of the system, preserved and alive, not as an aspiration but as the default shape of your decision log.

It's still early days, but instead of an ending I'd like to propose a beginning: imagine the future of AI coding in a world where all decisions in your dev meeting that ended five minutes ago have already been filed semantically to a machine-readable graph knowledge database and reconciled with all other historical constraints of the project.