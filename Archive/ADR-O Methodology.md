# ADR-O Methodology

This document is a practical guide to authoring decision records in ADR-O. It assumes familiarity with the core concepts but not with the craft. For the theoretical foundations, see:

- [*Decisions Are Fish-Shaped*](./Decisions%20Are%20Fish-Shaped.md) — the structural anatomy of a decision record
- [*The Decision Transaction Principle*](./The%20Decision%20Transaction%20Principle.md) — immutability, tiers, and the learning delta
- [*ADR-O Methodology From First Principles*](./ADR-O%20Methodology%20From%20First%20Principles.md) — the complaint/need distinction and the Y-statement
- [*Y-Statements*](./Y-Statements.md) — the six-clause decision format and its ADR-O mapping

While those documents explain the *why*, this document explains the *how*.

## Before You Open a Record

Not every decision warrants a record. The significance filter — the quality that distinguishes a record-worthy decision from a routine one — is not about domain or scale, it is about reversibility and consequence. A decision is record-worthy when:

- it is **load-bearing**: other decisions, artifacts, or commitments will depend on it;
- it is **hard to reverse**: undoing it later would cost significantly more than making it carefully now;
- it has **alternatives**: there was a genuine choice, not a forced hand.

If all three conditions are met, open a record. **If unsure, err toward opening one.** The cost of an unnecessary record is low; the cost of a missing record — especially when the rationale has evaporated from living memory — can be very high.

One practical test: *would a capable new team member, reading only the codebase or the deliverable, be unable to reconstruct why this was done?* If yes, the record is warranted.

## The Snout: Extracting the Complaint

The snout is the hardest part to get right, because it requires you to reconstruct something that is usually implicit or already forgotten by the time anyone writes anything down.

**The snout is the raw triggering situation** — the complaint, the pain point, the agenda item, the ticket title — that caused deliberation to begin. It arrives in ordinary language, at ordinary resolution, before any analysis has been done:

> *"MFA"*  
> *"Survivor detection in collapsed structures"*  
> *"The report export takes too long"*  
> *"Session handling"*

It is not a solution. It is not a need. It is the name of the situation.

### The meeting agenda test

The most reliable way to extract the snout is to ask: *what would someone have written on the meeting agenda, or in the ticket title, or in the Slack message that started this conversation?* That phrasing — casual, pre-analytical, at ordinary resolution — is the snout.

If the answer you come up with contains a technology, a proposed solution, or a verb like "implement" or "migrate," you have probably slipped past the snout into the head or even the body. A snout names the situation; it does not prescribe a response.

Compare:

| Too solution-contaminated | Likely snout |
|---|---|
| *"WiFi-based survivor detection for disaster scenarios"* | *"Survivor detection in collapsed structures"* |
| *"Implement MFA"* | *"Login strengthening"* or *"Login security"* |
| *"Migrate to PostgreSQL"* | *"Database reliability"* or *"Database capacity"* |
| *"Adopt event-driven architecture"* | *"Service coupling"* |

The left column names the approach. The right column names the problem. The snout is always just the problem.

### Who names it, and why that matters

The person who names the snout is typically the primary stakeholder — the one who felt the pain strongly enough to initiate the conversation. This is not always the author of the record, and is often not the decision-maker. In ADR-O, `adr-o:namedBy` on the `Complaint` node captures this explicitly, and it is worth capturing: knowing who felt the pain tells you whose concern the decision was ultimately in service of.

### Immutability

The snout is the only element of a decision record that is unconditionally immutable — not frozen on external reference like the rest of the record, but frozen from the moment the record is first committed. If the snout changes, you have a different conversation – so draw a new fish (create a new DecisionRecord with its own new snout).

### When the snout is missing in source material

Some records arrive fully formed, starting directly with context and alternatives, with no traceable triggering situation in the surviving source material. This is a structural gap in evidence, not a license to relax the model: a well-formed `DecisionRecord` still requires exactly one snout.

Do not invent a snout. If the currently available source material is insufficient, pause publication, recover additional evidence (ticket history, agenda notes, chat logs, stakeholder interviews), and mint exactly one `Complaint` before considering the record complete.

## The Head: Sharpening the Need

The head is where the snout undergoes translation. Someone — or a group — takes the raw complaint and asks: *what exactly must be true for this situation to be resolved?* The answer is the head: a falsifiable constraint, a measurable target, a compliance requirement, a performance bound.

> *"MFA"* (snout) → *"All user-facing services must require a second authentication factor to comply with ISO 27001 by Q3 2026."* (head)

The translation is real work. It requires you to understand the constraint well enough to make it falsifiable. A head that cannot be satisfied, measured, or failed is not a head — it is a rephrasing of the snout.

A well-formed head has these properties:

- **Falsifiable**: there is a state of the world that would constitute satisfaction, and a state that would constitute failure.
- **Singular**: the head names one primary concern. A head with multiple competing concerns has not been scoped; it has been stacked. If you cannot identify a single primary need, the snout may encompass multiple fish.
- **Pre-solution**: the head describes what must be achieved, not how. *"We must support five releases per day"* is a head. *"We must use a CI pipeline that completes in under ten minutes"* is drifting toward the body.

In ADR-O, the head is a `Claim` manifested by a `ContextFact`, typically with `adr-o:addresses` pointing at the relevant `Concern`. The `adr-o:satisfies` predicate on the `chosenAlternative` will later close the syllogistic loop: the chosen option was chosen *because* it satisfies this claim.

## When to Split: Recognizing a Second Fish

A critical rule follows from the one-fish-one-snout principle: **if, while traversing the body of one record, you discover that a second distinct decision is required, open a new record immediately.**

Do not finish the first fish and then start the second. The moment of recognition — *"while discussing MFA implementation, we realized that session token expiry policy requires its own decision"* — is itself a prime mover. That recognition is a snout. It gets its own record, its own head, its own deliberation.

The two records can be linked — likely via `adr-o:enables` or `adr-o:dependsOn` — but they are separate epistemic transactions, each with a single snout, each converging on its own answer.

**The diagnostic**: if your record converges on two answers, or if the head contains two primary concerns that cannot be reduced to one, you have two fish. Split it.

## The Body: Deliberating Honestly

The body is the widest part of the fish — the double diamond — and it is where most authors spend the least time and care.

The body should contain:

- **Constraints** that bound the solution space: legal, regulatory, technological, organizational, financial. These are `ContextFact` nodes, placed before the deliberation proper.
- **Quantitative performance requirements**, when present in the source material. These are constraints too — measurable, falsifiable bounds that frame the entire deliberation. Each one deserves its own `Claim` and its own `ContextFact`. Do not collapse a requirements table into a single claim or omit it as "implementation detail." A false negative rate of <1% is a more machine-verifiable commitment than almost anything else in the record; it is the kind of claim that `ObservedOutcome` nodes are built to verify.
- **Alternatives** that were genuinely considered. Each alternative should be named and described as a first-class `Alternative` node in the graph. An alternative that appears only to be shot down is probably a post-hoc rationalization, not a genuine option. The test: *would this alternative have been a reasonable choice at the time, given what was known?* If not, it was never really on the table.
- **Arguments for and against** each alternative, as `DeliberationFact` nodes with `deliberationValence`. A body that lists only supporting arguments for the chosen option has not deliberated — it has rationalized.

The body is inherently messy. Constraints interact with alternatives; an option that satisfies the head may violate a regulatory constraint. Accept the mess. The record should make a reasonable attempt to capture the actual deliberation, not a sanitized reconstruction of it.

## The Convergence: Committing

The convergence point is where the double diamond closes. One alternative is chosen; the others are explicitly named and set aside — not forgotten, but *neglected*, in the precise sense of the Y-statement's fourth clause.

In ADR-O:

- `adr-o:chosenAlternative` names the selected option. It is functional: exactly one per record.
- `adr-o:hasAlternative` names all options including the chosen one. The unchosen alternatives are recoverable from the set difference.
- `adr-o:satisfies` on the chosen alternative closes the syllogism: `chosenAlternative → satisfies → Claim` (the primary facing concern from the head). This is not optional scaffolding — it is the explicit assertion that the chosen option was chosen *because* it addresses the stated need. Without it, the connection is implicit and unqueryable.

The convergence point defines t₀. Everything before it was input to the decision; everything after it is output.

## The Tail: Committing to Expectations

The tail fans out from the convergence point: all the things expected to happen as a consequence of the decision, made at t₀ on the basis of what was known at t₀.

In ADR-O, the tail is the `ExpectedOutcome` cluster. Four valences are available:

- `ExpectedGain` — a positive outcome you intend to produce
- `ExpectedCost` — a downside you are knowingly accepting (*"accepting that..."* in the Y-statement)
- `ExpectedRisk` — a possible negative outcome you acknowledge but cannot fully control
- `ExpectedDependency` — work or further deliberation that this decision triggers

The distinction between `ExpectedCost` and `ExpectedRisk` matters: a cost is accepted voluntarily as a trade-off (you could have chosen differently); a risk is acknowledged but not chosen (it may or may not materialize regardless).

When the source material includes a risk-mitigation table, transcribe both columns into each `ExpectedRisk` claim's `dcterms:description`. The ontology does not yet have a dedicated `mitigatedBy` predicate; until it does, the mitigation belongs in the prose of the claim itself — not discarded. A claim that reads *"Metal debris in collapse zones may block RF signals. Mitigation: multi-angle scanning and adaptive frequency selection."* is queryable as prose and preserves the full intent. Dropping the mitigation silently loses half the information.

**The tail is immutable at t₀.** The beliefs recorded in the tail are what was believed at the moment of commitment. If reality later proves them wrong, that divergence is captured in `ObservedOutcome` artifacts — never by editing the tail. Editing the tail to match what actually happened is the defining act of a non-learning organization.

## The Wake: Observing Outcomes

The wake is not part of the record. It lives outside the epistemic transaction.

`ObservedOutcome` artifacts are minted at tₙ > t₀, linked to the record via `adr-o:realizedFrom`, and carry their own timestamps (`adr-o:observedAt`), attribution (`adr-o:observedBy`), and evidence (`adr-o:evidence`). They may optionally verify an `ExpectedOutcome` via `adr-o:verifies`, and carry a verdict (`Satisfied`, `Violated`, or `Inconclusive`).

The gap between the tail and the wake — between what was believed at t₀ and what was observed at tₙ — is the learning delta. It is where a learning organization discovers whether it is making decisions for the right reasons. Preserving that gap requires that the tail remain unchanged. Erasing it destroys the most valuable thing the record contains.

## Transcribe Everything: Prose Is Not Loss

When converting a source document to RDF, the instinct to produce clean, sparse, well-structured Turtle is correct. That instinct does not extend to the content of literals. **Clean RDF and full prose are not in tension.** A graph with 350 triples whose claims faithfully carry directory trees, struct definitions, triage tables, and performance bounds is better than a graph with 250 triples whose claims are summaries.

The failure mode is subtle: it presents itself as editorial judgment. The converter decides that a directory tree is "implementation detail," that a requirements table is "not part of the rationale," that a struct definition belongs in code, not in the record. Each individual decision feels defensible. Collectively, they destroy the information that makes the tail verifiable and the deliberation honest.

The correct rule is simpler: **transcribe the source material aggressively. Mark ontology gaps as gaps. Never resolve a gap by omission.**

Concretely:

- If the source has a requirements table with six rows, mint six claims — one per row — each carrying the full numeric target in its `dcterms:description`. Do not summarize them into one.
- If the source has a directory tree, put it in the `dcterms:description` of the relevant `Alternative` or `DeliberationFact` claim. The prose datatype is Markdown; trees, tables, and code blocks are all valid content.
- If the source has a struct definition, include it. A future `ObservedOutcome` or downstream record may need to reference the precise shape of the chosen design.
- If the source has a risk-mitigation table, capture both columns in every `ExpectedRisk` claim. The mitigation is not a separate concern — it is part of what was believed at t₀.

When you find yourself omitting something because it "doesn't fit neatly," that is the signal to stop and ask whether the ontology has a gap, not whether the content can be dropped.

## Wiring to the Graph: Practical Turtle Notes

### Prefixes

Every ADR-O Turtle file should declare these prefixes at minimum:

```turtle
@prefix adr-o:   <https://w3id.org/adr-o#> .
@prefix dcterms: <http://purl.org/dc/terms/> .
@prefix skos:    <http://www.w3.org/2004/02/skos/core#> .
@prefix prov:    <http://www.w3.org/ns/prov#> .
@prefix rdf:     <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:     <http://www.w3.org/2001/XMLSchema#> .
```

### Prose literals

All prose-carrying literals use `^^adr-o:md`, the ergonomic alias for the IANA Markdown datatype (ADR-0038):

```turtle
dcterms:description "The raw triggering situation."^^adr-o:md ;
skos:definition     "A reusable atomic proposition."^^adr-o:md ;
```

**Apostrophes and special characters in literals**: Turtle parsers treat the single-quote character (`'`) as a string delimiter. Any `skos:prefLabel` or `dcterms:description` containing an apostrophe (e.g. *"K9 units have limited availability"* rewritten as *"K9's availability..."*) will silently break a single-quoted literal, consuming everything up to the next `'` as part of the string. The safe practice is to author all prose literals using double-quoted strings (`"..."`) and avoid single-quoted Turtle literals entirely. For multiline content, prefer `\n` escape sequences inside a double-quoted string over triple-quoted literals (`"""..."""`): some parsers (including older rdflib releases) mishandle backtick characters (`` ` ``) inside triple-quoted strings, producing cryptic `string index out of range` errors. When in doubt, generate Turtle programmatically via rdflib or a similar library and let the serializer handle escaping — this eliminates the entire class of encoding errors.

### File-level ordering: fish first, DecisionRecord last

The RDF graph is order-independent by definition — a triple is a triple regardless of where it appears in the file. But the Turtle serialization is read by humans too, and the order of subject blocks in the file shapes how easily a reader can reconstruct the reasoning.

The temptation is to open with the `DecisionRecord` block, since it is the hub of the graph and the most recognizable landmark. Resist it. The `DecisionRecord` block is a collection of outward arcs to nodes that must already exist in the reader's mental model for those arcs to be meaningful. Introducing the hub before its spokes makes every pointer a forward reference.

The natural order follows the fish:

1. **Agents** — the people, before anything they participate in is mentioned.
2. **Complaint** — the snout; the cause of existence of the record.
3. **Concerns** — the domain anchors that claims address.
4. **Claims** — the atomic propositions; organized in fish order within this section: context claims first (sensing limitations, the primary need, performance requirements), then deliberation claims (supporting, then against the chosen alternative, then against unchosen alternatives), then expected outcome claims (gains, costs, risks, dependencies).
5. **Alternatives** — after the claims that argue for or against them, so `adr-o:satisfies` points backward, not forward.
6. **DecisionRecord** — last, when every node it references is already defined. Its inline blank nodes (`ContextFact`, `DeliberationFact`, `ExpectedOutcome`) are the only forward references remaining, and those are self-contained.
7. **Referenced resources** — bibliographic annotations, after the record that references them.

This ordering means a reader scanning the file top-to-bottom encounters the vocabulary of the decision before the decision itself — the same order in which the deliberation actually unfolded. It also means that tooling doing a linear parse never needs to resolve a named-node forward reference.

### The Complaint node

```turtle
:complaint-001 a adr-o:Complaint ;
    dcterms:description "Session handling"^^adr-o:md ;
    dcterms:created "2026-01-10"^^xsd:date ;
    adr-o:namedBy :agent-cso .

:ADR-001 adr-o:promptedBy :complaint-001 .
```

`promptedBy` is functional and normatively required: exactly one complaint per record. If you cannot identify the complaint from currently available source material, recover additional evidence and complete the record; do not invent text and do not publish a snoutless record as complete.

### Ordered lists for context, deliberation, outcomes

`hasContext`, `hasDeliberation`, and `hasExpectedOutcome` take `rdf:List` values. Use Turtle's parenthesis syntax:

```turtle
adr-o:hasContext (
    [ a adr-o:ContextFact ; adr-o:manifests :claim-a ]
    [ a adr-o:ContextFact ; adr-o:manifests :claim-b ]
) ;
```

The list order reflects author intent and is used by tooling to reconstruct narrative order. Blank nodes are appropriate for fact individuals that will not be referenced from elsewhere; name them if you need to point at them from another record.

### The satisfies bridge

`adr-o:satisfies` belongs on the `Alternative`, not on the `DecisionRecord`:

```turtle
:alt-chosen a adr-o:Alternative ;
    adr-o:satisfies :claim-primary-need .
```

This closes the syllogism: `DecisionRecord →chosenAlternative→ Alternative →satisfies→ Claim →addresses→ Concern`.

### Bibliographic references

Use `dcterms:references` on the record for external citations, and annotate each cited resource as an independent subject:

```turtle
:ADR-001 dcterms:references <https://arxiv.org/abs/2004.03661> .

<https://arxiv.org/abs/2004.03661>
    dcterms:title "CSI-based Human Activity Recognition" ;
    dcterms:description "Surveys CSI signal processing for classifying human activities; informs the movement and breathing pattern classifiers."^^adr-o:md .
```

Resource annotation blocks are shared across records: if a second record cites the same IRI, only the `dcterms:references` arc need be added. The annotation is already in the graph.

For very large ADLs, consider a dedicated `references.ttl` file to hold resource descriptions, keeping individual record files focused on the decision.

## Diagnostics: The Fish as a Check

The fish shape is not just a description of a well-formed record — it is a diagnostic. Every malformed record has a characteristic shape, and the shape tells you where the problem lies.

| Symptom | Diagnosis | Remedy |
|---|---|---|
| No `promptedBy` | The record is structurally incomplete: the cause of existence is unrecorded | Reconstruct the triggering situation from source material before considering the record complete; do not invent complaint text |
| Snout contains a solution | The complaint has been contaminated by the answer; the snout is actually the head or body | Strip the technology or approach from the snout; name the problem, not the response |
| Wide head — multiple primary concerns | The decision has not been scoped; two or more distinct needs are competing for primacy | Identify the primary concern; demote secondary concerns to body constraints, or split into multiple records |
| No body, or body jumps to chosen option | The alternatives were not genuinely considered; the record rationalizes rather than deliberates | Go back to the divergence phase; name the real alternatives and argue them honestly |
| `chosenAlternative` has no `satisfies` arc | The syllogistic connection between the chosen option and the primary need is implicit | Add `adr-o:satisfies` from the chosen alternative to the primary `Claim` from the head |
| No tail | The record commits to a path without committing to any beliefs about what that path will produce | Add `ExpectedOutcome` nodes; if there are truly no expected gains, costs, or risks, the decision was not thought through |
| Record converges twice | Two decisions are tangled in one record | Split at the second convergence point; the moment of recognition of the second decision is its snout |
| Tail edited to match wake | The learning delta has been destroyed | Restore the original tail from version history; capture the realized outcome as an `ObservedOutcome` instead |
| No `ObservedOutcome` on an old record | The wake has not been captured; the learning delta is unmeasured | This is not a structural error, but it is a missed opportunity; prioritize observation for high-stakes or high-uncertainty decisions |
| Requirements table in source, single claim in graph | Quantitative bounds have been collapsed; the tail is unverifiable | Mint one `Claim` per requirement row; each numeric target deserves its own `ContextFact` so `ObservedOutcome` nodes can verify it individually |
| Risk present in source, mitigation absent from graph | Half the t₀ belief has been dropped; the record cannot reconstruct what was known | Include the mitigation in the `dcterms:description` of the `ExpectedRisk` claim; if the ontology gains a `mitigatedBy` predicate, migrate then |
| Source content omitted because it "doesn't fit neatly" | An ontology gap has been resolved by silence rather than noted as a gap | Transcribe the content into prose; never drop information to make the graph look cleaner |
| `DecisionRecord` block appears first in the file | Every arc in the hub is a forward reference; the reader cannot follow the reasoning top-to-bottom | Reorder: Agents → Complaint → Concerns → Claims → Alternatives → DecisionRecord → Referenced resources |
