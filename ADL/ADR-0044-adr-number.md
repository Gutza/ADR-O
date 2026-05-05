---
id: 44
type: Core vocabulary
status: Accepted
date: 2026-05-05
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0007
  - ADR-0024
---

# ADR-0044 — Rename `adr-o:index` to `adr-o:adrNumber`

## Context

ADR-0007 introduced `adr-o:index` as an `xsd:integer` property on `DecisionRecord`, providing a machine-sortable numeric complement to the string `dcterms:identifier`. The name `index` was chosen without close attention to semantics: it describes a data structure (an index in a list) rather than what the property actually represents.

The property's real semantic load is the ADR's **number** — the stable, publicly referenceable identity token that practitioners use every time they cite a record. When a record is referred to as "ADR-0042", the `42` is not incidentally its position in a sorted list; it is how the record is named. It appears in filenames, in inter-record links, in prose, in conversation. That is identity behaviour, not ordering auxiliary behaviour.

The distinction was crystallised by ADR-0043, which introduced `adr-o:consolidationOrder` for the `ConsolidatedOutcome` ordering integer. `consolidationOrder` is explicitly a disposable tooling auxiliary: mutable, rewritable, droppable without any record losing its meaning. The contrast made the semantic gap in `adr-o:index` visible: the two properties look identical in type (`xsd:integer`, functional) but are categorically different in kind. One is a sort key. The other is a name.

A name deserves a name that says so.

## Decision

Rename `adr-o:index` to **`adr-o:adrNumber`**. The semantics, range, and functional constraint are unchanged. Only the IRI is updated.

`adr-o:adrNumber` is self-documenting without context: it identifies what it holds (the ADR's number) without requiring the reader to infer it from the domain restriction or namespace. It also cleanly distinguishes the property from `adr-o:consolidationOrder`: one is the record's identity, the other is a sorting auxiliary.

## Ontology Materialisation

```turtle
###  https://w3id.org/adr-o#adrNumber
adr-o:adrNumber rdf:type owl:DatatypeProperty ,
                         owl:FunctionalProperty ;
    rdfs:domain adr-o:DecisionRecord ;
    rdfs:range xsd:integer ;
    rdfs:comment "The number of a DecisionRecord within its ADL — the stable, publicly referenceable integer identity token used in citations, filenames, and inter-record links. Machine-sortable complement to the string dcterms:identifier."@en ;
    rdfs:label "ADR number"@en ;
    adr-o:justifiedBy adr-odr:ADR-0044-adr-number .
```

The following property is **removed**:

```turtle
adr-o:index   # Renamed to adr-o:adrNumber
```

## Alternatives considered

| Alternative | Reason not chosen |
|---|---|
| Keep `adr-o:index` | Semantically underspecified: names a data structure, not the thing the property represents. The contrast with `adr-o:consolidationOrder` (a genuine ordering auxiliary) makes the gap untenable. |
| `adr-o:number` | Correct in spirit but insufficiently explicit: `number` without qualification could be any number on any resource. `adrNumber` is self-documenting. |
| `adr-o:recordNumber` | Reasonable, but "record" is not how practitioners refer to these in practice. Nobody says "record number 42"; they say "ADR-0042". The name should match colloquial usage. |
| `adr-o:logOrder` | Imports only ordering semantics. The ADR number is not just a sort key — it is the record's name. A name that communicates only ordering would misrepresent the property's identity significance. |
| `adr-o:sequenceNumber` | Same problem as `logOrder`: sequence-flavoured names suggest the value could be reassigned if the sequence were reordered. An ADR number cannot be reassigned; ADR-0042 will never become ADR-0041. |

## Consequences

**Positive.**

- The property name now matches what practitioners already call it: "the ADR number."
- The semantic distinction between identity tokens (`adrNumber`) and ordering auxiliaries (`consolidationOrder`) is legible from the vocabulary alone, without requiring documentation to explain the difference.
- The `rdfs:comment` can now accurately describe what the property represents rather than what it is used for.

**Negative / risks.**

- Breaking rename: any deployed graph using `adr-o:index` requires a migration. At the time of this record the ontology has no external adopters; the cost is accepted.
- `README.md` and any live tooling referencing `adr-o:index` must be updated. These are living documents and are not subject to the enacted-layer immutability constraint.

## References

- [ADR-0007](/ADL/ADR-0007-index-vs-identifier.md) — Introduced `adr-o:index`; amended by this record to replace it with `adr-o:adrNumber`.
- [ADR-0024](/ADL/ADR-0024-sequential-ids-for-all-decision-records.md) — Establishes the sequential assignment convention; the property it references is renamed by this record.
- [ADR-0043](/ADL/ADR-0043-consolidated-adl.md) — Introduced `adr-o:consolidationOrder`; the contrast between that property and `adr-o:index` crystallised the semantic gap this record resolves.
