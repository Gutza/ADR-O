Please orientate yourself in the project a little, and let's discuss MADR/YADR/JSON ADR lessons we can learn from:

```md
---
# These are optional metadata elements. Feel free to remove any of them.
status: "{proposed | rejected | accepted | deprecated | … | superseded by ADR-0123}"
date: {YYYY-MM-DD when the decision was last updated}
decision-makers: {list everyone involved in the decision}
consulted: {list everyone whose opinions are sought (typically subject-matter experts); and with whom there is a two-way communication}
informed: {list everyone who is kept up-to-date on progress; and with whom there is a one-way communication}
---

# {short title, representative of solved problem and found solution}

## Context and Problem Statement

{Describe the context and problem statement, e.g., in free form using two to three sentences or in the form of an illustrative story. You may want to articulate the problem in form of a question. Consider adding links to collaboration boards or issue management systems. Make the scope of the decision explicit, for instance, by calling out or pointing at structural architecture elements (components, connectors, ...).}

<!-- This is an optional element. Feel free to remove. -->
## Decision Drivers

* {decision driver 1, for instance, a desired software quality, faced concern, constraint or force}
* {decision driver 2}
* … <!-- numbers of drivers can vary -->

## Considered Options

* {title of option 1}
* {title of option 2}
* {title of option 3}
* … <!-- numbers of options can vary -->

## Decision Outcome

Chosen option: "{title of option 1}", because {justification. e.g., only option, which meets k.o. criterion decision driver | which resolves force {force} | … | comes out best (see below)}.

<!-- This is an optional element. Feel free to remove. -->
### Consequences

* Good, because {positive consequence, e.g., improvement of one or more desired qualities, …}
* Bad, because {negative consequence, e.g., compromising one or more desired qualities, …}
* … <!-- numbers of consequences can vary -->

<!-- This is an optional element. Feel free to remove. -->
### Confirmation

{Describe how the implementation / compliance of the ADR can/will be confirmed. Is there any automated or manual fitness function? If so, list it and explain how it is applied. Is the chosen design and its implementation in line with the decision? E.g., a design/code review or a test with a library such as ArchUnit can help validate this. Note that although we classify this element as optional, it is included in many ADRs.}

<!-- This is an optional element. Feel free to remove. -->
## Pros and Cons of the Options

### {title of option 1}

<!-- This is an optional element. Feel free to remove. -->
{example | description | pointer to more information | …}

* Good, because {argument a}
* Good, because {argument b}
<!-- use "neutral" if the given argument weights neither for good nor bad -->
* Neutral, because {argument c}
* Bad, because {argument d}
* … <!-- numbers of pros and cons can vary -->

### {title of other option}

{example | description | pointer to more information | …}

* Good, because {argument a}
* Neutral, because {argument b}
* Bad, because {argument c}
* …

<!-- This is an optional element. Feel free to remove. -->
## More Information

{You might want to provide additional evidence/confidence for the decision outcome here and/or document the team agreement on the decision and/or define when/how this decision the decision should be realized and if/when it should be re-visited. Links to other decisions and resources might appear here as well.}
```


```yaml
---
metadata:
  # these are optional metadata elements. feel free to remove any of them, or entire metadata mapping
  status: '{proposed | rejected | accepted | deprecated | … | superseded by ADR-0123}'
  date: '{YYYY-MM-DD when the decision was last updated}'
  decision-makers: '{list everyone involved in the decision}'
  consulted: '{list everyone whose opinions are sought (typically subject-matter experts); and with whom there is a two-way communication}'
  informed: '{list everyone who is kept up-to-date on progress; and with whom there is a one-way communication}'

title: '{short title, representative of solved problem and found solution}'

context-and-problem-statement: |
  {Describe the context and problem statement, e.g., in free form using two to three sentences or in the form of an illustrative story.

  * You may want to articulate the problem in form of a question.
  * Consider adding links to collaboration boards or issue management systems.
  * Make the scope of the decision explicit, for instance, by calling out or pointing at structural architecture elements (components, connectors, ...).}

# this is an optional element
decision-drivers:
- '{decision driver 1, e.g., a force, facing concern, …}'
- '{decision driver 2, e.g., a force, facing concern, …}'
- '… <!-- numbers of drivers can vary -->'

considered-options:
- &id-of-option-1 '{title of option 1}'
- &id-of-option-2 '{title of option 2}'
- &id-of-option-3 '{title of option 3}'
# numbers of options can vary

# this is an optional element
pros-and-cons-of-the-options:

  # the id/title needs to match the one defined in considered-options
  id-of-option-1:
    # this is an optional element
    description: |
      {example | description | pointer to more information | …}

    pros:
    - &argument-a '{argument a}'
    - '{argument b}'
    # use "neutral" if the given argument weights neither for good nor bad.

    neutral:
    - &argument-c '{argument c}'

    cons:
    - &argument-d '{argument d}'
    # numbers of pros and cons can vary

  id-of-option-2:
  # same structure as id-of-option-1
    description: |
      {example | description | pointer to more information | …}

    pros:
    - &argument-a '{argument a}'
    - '{argument b}'
    # use "neutral" if the given argument weights neither for good nor bad.

    neutral:
    - &argument-c '{argument c}'

    cons:
    - &argument-d '{argument d}'
    # numbers of pros and cons can vary

decision-outcome:

  chosen-option:
    link: *id-of-option-1
    justification: |
      {rationale; e.g.,
        - only option, which meets k.o. criterion decision driver, which resolves force {force}, …
        - {why other options were not chosen, e.g., id-of-option-2 fails because …}
        - comes out best (see below)}

  # this is an optional element
  consequences:
    positive:
    # numbers of entries can vary
    - *argument-a
    - '{positive consequence, e.g., improvement of one or more desired qualities, …}'
    neutral:
    - *argument-c
    - '{neutral consequence, e.g., resulting activity, new requirement/decision, …}'
    negative:
    - *argument-d
    - '{negative consequence, e.g., compromising one or more desired qualities, …}'

  # this is an optional element
  confirmation: |
    {Describe how the implementation / compliance of the ADR can/will be confirmed. Is there any automated or manual fitness function? If so, list it and explain how it is applied. Is the chosen design and its implementation in line with the decision? E.g., a design/code review or a test with a library such as ArchUnit can help validate this. Note that although we classify this element as optional, it is included in many ADRs.}

more-information: |
  {You might want to provide additional evidence/confidence for the decision outcome here and/or document the team agreement on the decision and/or define when/how this decision the decision should be realized and if/when it should be re-visited. Links to other decisions and resources might appear here as well.}
```

```json
{
  "metadata": {
    "id": "AD-002",
    "author": "OZ",
    "date": "2026-03-07",
    "version": "1.1",
    "title": "Render Y-Statements in YAML",
    "related": "AD-001"
  },
  "context": "\nY-statement ADRs need a concrete syntax that can be parsed and validated.\n",
  "facing": [
    "Effort to create and maintain",
    "Readability"
  ],
  "decided-for": "Yet another markup language (YAML)",
  "neglected": [
    "Plain text",
    "Markdown",
    "JSON"
  ],
  "to-achieve": [
    "A healthy balance between cost and benefit",
    "Being able to show something new",
    "Include those on the team who are familiar with \"YAML\""
  ],
  "accepting-that": [
    "AD consumers not familiar with YAML might complain"
  ],
  "additional-rationale": "technology exploration | increase visibility and community outreach"
}
```

---

**The argument reuse problem**

YADR is the most structurally revealing of the three formats. The YAML anchor/alias mechanism (`&argument-a` → `*argument-a`) encodes a genuinely important semantic claim: the argument you made in favour of Option 1 during evaluation *is the same argument* that appears as a positive consequence in the decision outcome. Not a copy of it — the same thing. MADR handles this with copy-paste, which means the "Pros" section and the "Consequences" section can silently drift. YADR makes the identity explicit within the document.

ADR-O could solve this *properly* — with IRI identity across the graph — but only if arguments or forces are first-class nodes. Right now they're buried in prose (`context`, `rationale`, `consequences`). This is one of the deferred items in DESIGN-NOTES, and I think YADR makes the strongest possible case for promoting it. A first-class `Force` or `Argument` class is the thing that lets you say in SPARQL: "show me all decisions where the same force appeared as a pro for the chosen option and as a con for a rejected one."

**The "accepting-that" gap**

The JSON Y-statement format has the most honest structural element of the three: `accepting-that`. It forces the author to name what they gave up by choosing the winner. ADR-O models `wasRejectedBecause` on `Alternative` — the rejection rationale for losers — but has no parallel predicate for the chosen option's trade-offs. The design notes mention splitting `consequences` into positive/negative (MADR pattern), but that's not quite the same thing. `accepting-that` is specifically about the costs of the *chosen path*, which is more precise than a general negative-consequence list. Worth considering as a named predicate rather than just a subsection of prose.

**The chosen-alternative marking problem**

All three formats struggle with this differently. MADR names the winner inline in prose. YADR links via anchor. JSON has `decided-for` and `neglected` as parallel first-class slots. ADR-O currently has `hasAlternative` and `wasRejectedBecause`, but the chosen alternative is only identifiable by absence — it's the one without a `wasRejectedBecause`. That's an inferential gap. An explicit predicate — something like `adr-o:implements` or `adr-o:chosenAlternative` from `DecisionRecord` to `Alternative` — would close it and make the decision outcome machine-traversable without text parsing.

**Neutral arguments**

YADR's three-way split (pros / neutral / cons) captures something real that the other two formats ignore: some considerations weigh neither for nor against a choice but still belong in the record. A factor that's neutral today may become decisive later when conditions change. MADR loses this entirely. JSON doesn't model it. ADR-O's prose fields lose it too. If forces ever become first-class nodes, the weight or valence of a force relative to an alternative (positive, negative, neutral) is a natural property on the relationship, not on the node itself.

**Confirmation / fitness functions**

MADR is the only format that explicitly asks *how will you know this decision is being followed?* The "Confirmation" section — whether there's an automated fitness function or a manual review process — is structurally absent from YADR and the JSON format, and from ADR-O entirely. For a machine-interpretable ADR graph, this is a significant gap: compliance is exactly the kind of question an agent should be able to answer by traversal, not by reading prose. A `conformanceCheck` predicate pointing to a test, a CI rule, or a SHACL shape would be a meaningful addition.

**Decision drivers vs concerns**

MADR has "Decision Drivers" as a first-class list. The JSON format has `facing` as its parallel. ADR-O has `addresses` linking a `DecisionRecord` to a `Concern`. These are close but not identical: a "decision driver" in MADR is the force that pushed the decision, while `addresses` in ADR-O is the concern the decision resolves. The Y-statement's `facing` is actually closer to ADR-O's model than MADR's `decision-drivers`, because `facing` frames the concern as a problem being responded to rather than a quality attribute being optimised for. Worth being explicit in ADR-O documentation that `addresses` is meant to cover both senses — or flagging whether they should be split.

**What the JSON format uniquely reveals**

The Y-statement strips the ADR down to its irreducible core: *context + forces + choice + rejected-alternatives + goals + accepted-costs*. Everything else in MADR and YADR is elaboration of those six slots. That's a useful diagnostic for ADR-O: every class and predicate in the ontology should be traceable back to one of those six slots, or it's a candidate for deferral. `adr-o:context`, `adr-o:rationale`, `adr-o:decision`, `adr-o:consequences`, `Alternative`, `Concern` all map cleanly. What's missing in ADR-O's core is the *goals* slot (`to-achieve` in the JSON) — the intended outcomes that the choice was meant to achieve, which is distinct from the concerns being addressed and distinct from the consequences. That's currently prose in `rationale`.

GADR is a short paper (ZEUS 2019, Kopp & Armbruster) but it introduces a structurally distinct idea that none of the other formats touch.

**What GADR actually is**

GADR strips MADR down to everything that is *cross-project*: title, context, decision drivers, and the options with their pros and cons. It removes everything that is *project-specific*: status, deciders, date, and the decision outcome (justification, positive/negative consequences). The result is a reusable template. A team facing a decision copies the GADR, adds their project-specific weighting and outcome, and produces a concrete MADR. The JabRef/Eclipse Winery example shows this in practice — both projects used the same Java build-tool GADR but reached opposite decisions (Gradle vs Maven) because they weighted the same shared pros and cons differently.

The UML diagram is what matters for ADR-O: MADR *inherits from* GADR, which means GADR is the superclass. A concrete decision record is a specialization of a decision template, not the other way around.

**The gap this exposes in ADR-O**

ADR-O currently has one first-class record type: `adr-o:DecisionRecord`. GADR argues there is a meaningful second type — call it `DecisionTemplate` or `DecisionPattern` — that is neither a concrete record nor an `Alternative`. It's a cross-project archetype: a standing problem context with a known option space, not yet instantiated into any particular decision. Modeling this as a `Proposed` record doesn't work, because a `Proposed` record is still project-specific and is expected to resolve. A template is deliberately left open.

The practical graph query this enables: "which of our project's decision records were derived from shared organizational or community patterns?" That's a genuinely useful governance question that no other format even recognizes, and ADR-O has no predicate for it. An `adr-o:instantiates` property from `DecisionRecord` to `DecisionTemplate`, and a `DecisionTemplate` class that omits status/deciders/date while sharing context and options, would close it.

**The amended/clarified link types**

GADR introduces a typed link vocabulary — `SupersededBy`, `Supersedes`, `AmendedBy`, `Amends`, `ClarifiedBy`, `Clarifies` — and reflects it in MADR's status enumeration (`Accepted`, `Superseded`, `Amended`, `Clarified`). ADR-O currently has only `supersedes`/`supersededBy`. The missing relationships are:

`amends` — a later decision partially modifies an earlier one without replacing it. The original record remains in force; the amending record changes only a part of it. This is genuinely distinct from supersession. A record can be `Amended` and still active; a `Superseded` record is no longer in force.

`clarifies` — no change to the decision at all, just additional explanation. The target record's content is unchanged; the clarifying record adds interpretive context.

Both are weak enough relationships that they're commonly left as prose links ("see also ADR-0017"), but making them typed predicates would allow SPARQL to distinguish "this decision is a clean replacement" from "this decision modifies a still-active record" from "this decision only explains an unchanged record" — three completely different governance situations.

**The status corollary**

If `amends` and `clarifies` are first-class predicates, then `Amended` and `Clarified` are natural extensions to the status scheme — a record that has been partially amended should reflect that in its status, so a query asking "which records are fully current as-written?" can filter them out. ADR-O's status scheme currently has no way to express "still in force, but partially modified." This is a gap even if the `DecisionTemplate` concept is deferred.

**What's weaker in GADR**

The paper is honest that it's an early idea. The workflow (copy GADR → specialize into MADR) is manual and version-control-dependent, with no mechanism for tracking which MADRs descended from which GADRs. The authors acknowledge the discovery problem — how do teams find the GADR they need? — and leave it open. ADR-O could solve both of these cleanly: `adr-o:instantiates` handles provenance, and the graph itself handles discovery via SPARQL. That's the strongest argument for absorbing GADR's insight into the ontology rather than treating it as a separate file format.