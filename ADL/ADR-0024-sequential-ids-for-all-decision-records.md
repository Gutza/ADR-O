---
id: 24
type: Core design
status: Accepted
date: 2026-04-21
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0007
  - ADR-0020
---

# ADR-0024 — Sequential IDs for All Decision Records

## Context

ADR-0020 introduced amendments as first-class `DecisionRecord` individuals, and ADR-0007 established `adr-o:index` as a sequential integer for chronological ordering. However, the internal discussion around implementation raised a critical ambiguity: whether amendment records should carry derived IDs (e.g., `ADR-0042-A`) or independent IDs (e.g., `ADR-1337`).

The suffix pattern (`ADR-0042-A`) is a holdover from file-system-based ADLs where physical proximity in a directory serves as a proxy for relationship. In a graph, this is not only unnecessary but structurally misleading.

The real-time authoring model (ADR-0005) compounds this: an AI agent instrumenting a deliberation in real-time cannot know at the moment of record creation whether a decision will eventually be a standalone record, an amendment, or a supersession. Forcing a dependent ID scheme would require the agent to either guess and potentially rename (breaking IRI stability), or defer ID assignment until the relationship is resolved (breaking real-time logging).

## Decision

**Every `DecisionRecord` receives a unique, sequential `adr-o:index` and corresponding `dcterms:identifier` at the moment of creation, regardless of its eventual relationship to other records.**

### Specifics

1.  **No Suffixes.** Amendment records do not use suffixes like `-A`, `-amendment`, or `-1`. They receive the next available sequential ID in the log.
2.  **Relationship via Edge.** The relationship between an amendment and its anchor is encoded exclusively via the `adr-o:amends` / `adr-o:amendedBy` predicates.
3.  **Identity is Independent.** A record's identity is established at the moment it is recognized as record-worthy, not when its relationship to other records is determined.

### Example

```turtle
# Created at T=1
:ADR-0042 a adr-o:DecisionRecord ;
          dcterms:identifier "ADR-0042" ;
          adr-o:index 42 ;
          dcterms:title "Use PostgreSQL" .

# Created at T=100, during a meeting
# The agent creates it as a first-class record
:ADR-1337 a adr-o:DecisionRecord ;
           dcterms:identifier "ADR-1337" ;
           adr-o:index 1337 ;
           dcterms:title "Add read-replica requirement to PostgreSQL" ;
           adr-o:amends :ADR-0042 .
```

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| **Hierarchical IDs** (`ADR-0042-A`) | Couples identity to relationship; requires knowing the relationship at creation time; breaks sequential indexing; encodes graph topology in a string (anti-pattern). |
| **Deferred ID assignment** | Violates real-time logging requirement; creates "unidentified" decisions in the graph; complicates agent workflows. |
| **Template-based IDs** (e.g., `AMEND-001`) | Creates multiple ID sequences to track; complicates chronological sorting across the whole log; still couples identity to role. |

## Consequences

**Positive.**
- **Stable IRIs from T=0.** Records can be referenced immediately, even before their full relationship to the existing graph is understood.
- **Chronological honesty.** `adr-o:index` reflects the actual sequence of deliberation events in the project's history.
- **Query simplicity.** "What is the current state of decision X?" is a graph traversal starting at the anchor and following `adr-o:amendedBy` edges, not a string-parsing exercise on identifiers.
- **Tooling-friendly.** Agents can mint records confidently; the relationship is just another triple asserted as it becomes known.

**Negative / risks.**
- Humans who are used to scanning filenames may find it harder to "visually group" amendments with their anchors without tooling. This is a tooling problem (e.g., the renderer should show amendments grouped under anchors), not an ontology problem.

## References

- ADR-0004 — The KG Lives Under Tooling.
- ADR-0005 — Log All Decisions in the ADL (real-time authoring case).
- ADR-0007 — `adr-o:index` vs `dcterms:identifier`.
- ADR-0020 — Amendment Model.
