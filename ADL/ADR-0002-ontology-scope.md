---
id: 2
type: Core design
status: Accepted
date: 2026-04-18
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0002 — ADR = Any Decision Record: Domain-Agnostic Core with Domain-Specific Profiles

## Context

The acronym 'ADR' has traditionally expanded to 'Architecture Decision Record' — a label inherited from the practice's origin in software architecture teams. But neither the label nor the framing is a conceptual requirement: both are historical contingencies, and both are worth examining critically.

This decision addresses two questions that turn out to have the same answer: *what scope should ADR-O carry*, and *what should 'ADR' stand for in this project?*

**The problem ADR-O addresses is the loss of decision rationale outside deliverable proper.** A codebase shows what was implemented, never why; the deliberation evaporates the moment the meeting ends. But this challenge is not unique to software. An HR recruitment procedure, a set of agreed KPIs, a financial governance policy — none of these deliverables include the reasoning behind their individual provisions. The gap between *what was decided* and *why* is structural to white-collar knowledge work, not a pathology of software teams.

The question, then, is whether the ontology should encode software architecture as its scope at the class level (making other domains second-class subclassing exercises) or treat software architecture as one instantiation of a domain-agnostic core. The naming question is the same question, one level up.

### Anticipated objections to domain-agnostic scope

Three objections arise naturally from the ADR tradition and deserve explicit answers before the decision is recorded.

**"The relational vocabulary — `conflictsWith`, `enables`, `prevents` — only makes precise sense in software architecture."**

This conflates the ontology's responsibility with the practitioner's responsibility. The ontology defines that a relationship exists between decisions, and specifies its intended semantics. It does not — and cannot — enforce that practitioners assert it with rigor. That is true of every triple in every ontology: an OWL axiom does not prevent a false assertion, only an inconsistent one. The charge of imprecision outside software is a critique of practice, not of the model. More fundamentally, the claim that `conflictsWith` or `enables` are meaningless between non-software decisions leads to the ludicrous conclusion that decision-making in HR, finance, or strategy would lack coherent structure — which is plainly false. These relationships describe the topology of any decision graph, regardless of what the nodes contain.

**"The word 'architecture' provides a useful significance filter: ADRs are for load-bearing, hard-to-reverse decisions, not for every choice made."**

The filter is real and valuable, but "architecture" is a proxy for it, not the thing itself. This very decision about the scope of ADR-O — whether it applies to software only or to any knowledge work domain — is not a software architecture decision in any technical sense, yet it is obviously load-bearing and irreversible in exactly the way that warrants a formal record. The significance criterion is better captured explicitly (as an optional property or as a guideline in the ontology documentation) than implicitly through domain labeling baked into class IRIs.

**"The `Concern` class is too engineering-centric to generalize cleanly."**

This is a substantive objection, but it dissolves on closer inspection. The engineering-centric appearance of `Concern` comes not from the class itself but from the concept schemes that *instantiate* it in software contexts — quality attributes like performance, security, and maintainability. The class, defined abstractly as "a matter of interest or importance to a stakeholder" (following ISO 42010), is entirely domain-neutral. What varies across domains is the controlled vocabulary of concern instances, not the modeling slot. And critically: this variation exists *within* software as well. A game engine team, a medical imaging team, and a cryptography team each need entirely different concern taxonomies. The extension mechanism that handles intra-software domain variation is identical to the mechanism that would handle inter-domain variation. There is no additional design cost to making it work for HR or finance, because the mechanism must be built regardless.

### The naming question

The scope question and the naming question have the same answer. In 2021 MADR repurposed its own acronym from 'Markdown Architectural Decision Record' to 'Markdown (for) Any Decision Record,' making the same argument explicitly. They did it retroactively, with an established user base to migrate. ADR-O, having no publication history, can make the same move preemptively — before anything is published, before any user base exists, before any IRI is indexed anywhere — and arrive at consistency from the start rather than swimming upstream to it post-factum.

## Decision

**ADR-O will be scoped as a domain-agnostic core ontology, and the acronym 'ADR' in this project expands to 'Any Decision Record' preemptively.** The core will define abstract classes and typed object properties applicable to any decision record in any domain. Domain-specific vocabularies will be expressed as SKOS concept schemes that instantiate the core's abstract anchor points without modifying the core namespace.

The concrete implications for class naming and IRI design:

- The primary class will be `adr-o:DecisionRecord`, not `adr-o:ArchitectureDecisionRecord`. Encoding domain into the primary class IRI would make non-software uses structurally second-class and is inconsistent with the design goal of a portable core.
- `adr-o:ArchitectureDecisionRecord` will **not** be defined in the core ontology. A named subclass is only warranted when the ontology can state non-trivial axioms specific to that subclass; no such axioms have been identified, and defining a class whose constraints are TBD is defining a label, not a concept. Domain profiles that need this term may define `ex:ArchitectureDecisionRecord rdfs:subClassOf adr-o:DecisionRecord` in their own namespaces — which is exactly the extension mechanism this decision establishes.
- `adr-o:Concern` will be defined as an abstract anchor class. The core ontology will populate no concept scheme for it. Domain profiles — published independently, in their own namespaces — will define SKOS concept schemes whose members are valid values for `adr-o:addresses`. Software teams may publish `swdev:concerns`; HR teams may publish `hrpolicy:concerns`; neither touches the core.
- The same extension pattern applies to any other point where domain-specific controlled vocabularies are needed: additional status values beyond the core scheme, domain-specific relationship subtypes, and so on.

### Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Software architecture scope only | Encodes an artificial constraint at the IRI level; the design challenge of domain-specific concept schemes must be solved anyway for variation within software; the underlying problem is domain-agnostic |
| Domain-agnostic scope with `ArchitectureDecisionRecord` as a core subclass | Violates the design principle being established: domain-specific named classes belong in domain namespaces, not the core. Privileges software architecture over every other domain for no reason other than historical sentiment. Adds complexity and decision fatigue without any axioms to justify it — a class with no non-trivial constraints is a label, not a concept. |
| Multiple first-class domain profiles in the core namespace | Pollutes the core namespace with domain-specific terms; defeats the purpose of a stable, portable core; domain profiles belong in their own namespaces |
| Defer the scope decision until after initial ontology publication | IRIs are effectively irreversible once published; deferral is not neutral — publishing `adr-o:ArchitectureDecisionRecord` as the primary class *is* a scope decision, just an implicit and harder-to-reverse one |

## Consequences

**Positive.**
- The ontology's core is portable across any domain that produces decisions with rationale, alternatives, and typed relationships to other decisions.
- The software ADR community can define `ArchitectureDecisionRecord` in their own domain profile namespace if they need the term, without burdening the core.
- The SKOS concept-scheme extension mechanism, which must be built regardless, now serves both intra-software domain variation and inter-domain variation with no additional design cost.
- Adoption outside software becomes possible without forking or re-namespacing the ontology.

**Negative / risks.**
- A domain-agnostic core places a higher burden on the abstract definitions: `adr-o:DecisionRecord`, `adr-o:Concern`, and the relational properties must be defined with enough precision to be genuinely useful, and enough generality to remain honest across domains.

**Open questions.**
- What is the right way to express the significance criterion — the quality that distinguishes a record-worthy decision from a routine one — if not through the word "architecture"? A datatype property (`adr-o:significance`)? A scope note in the documentation? A SHACL constraint that profiling ontologies may tighten?
- How should domain profiles be discovered and registered? An informal registry? A SKOS collection maintained in the ADR-O repository? No central registry at all?

## References

- ADR-0000 — Inception Record (this ADL). Establishes the greenfield approach and the reuse strategy.
- ISO/IEC/IEEE 42010:2011. *Systems and Software Engineering — Architecture Description.* Source of the domain-neutral definition of `Concern` as "a matter of interest or importance to a stakeholder."
- W3C SKOS. https://www.w3.org/TR/skos-reference/ — The extension mechanism for domain-specific concept schemes.
- Guessi et al., OntolAD (2015) — surveyed in ADR-0000; the abstract treatment of `Concern` in ISO 42010 informed the decision to keep the class domain-neutral.
- Zimmermann, O. (2021). *ADR = Any Decision Record? Architecture, Design and Beyond.* https://medium.com/olzzio/adr-any-decision-record-916d1b64b28d — Makes the 'Any Decision Record' argument and records MADR's decision to repurpose its own acronym in v3.0.0.
