---
id: 34
type: Core vocabulary
status: Accepted
date: 2026-04-26
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
---

# ADR-0034 — Timestamping DecisionRecord: Creation, Decision, and Modification

## Context

ADR-O has a temporal model based on the Decision Transaction Principle (ADR-0028): a `DecisionRecord` is a snapshot at t₀. However, the ontology does not currently distinguish between three distinct temporal events relevant to every record:

1. **When the record was first created** (the authorship timestamp).
2. **When the decision was made** (t₀ — the moment of commitment).
3. **When the record was last editorially modified** (the artifact timestamp).

This distinction is not just metadata; it is epistemically relevant. For post-factum ADRs, the decision date precedes the creation date. For amended records, the modification date may be long after the decision date.

## Decision

We introduce three timestamp predicates on `adr-o:DecisionRecord`:

### 1. `dcterms:created` (The Authorship Date)
- **Type:** `owl:AnnotationProperty`
- **Range:** `xsd:date`
- **Semantics:** When the `DecisionRecord` artifact first came into existence. Captures when the team started debating this topic.
- **Why:** Standard Dublin Core predicate. For live ADRs, this is close to `decidedAt`. For post-factum ADRs, this is the date of reconstruction.

### 2. `adr-o:decidedAt` (The Decision Date)
- **Type:** `owl:AnnotationProperty`
- **Range:** `xsd:date` (with scope note allowing `xsd:dateTime`)
- **Semantics:** The moment the decision was reached and committed. This is the **t₀** of the record.
- **Why a new predicate?** No standard vocabulary provides this specific semantic without overloading. It is the temporal anchor for all expected outcomes.

### 3. `dcterms:modified` (The Modification Date)
- **Type:** `owl:AnnotationProperty`
- **Range:** `xsd:date`
- **Semantics:** When the record was last editorially modified (Tier 2 changes per ADR-0028).
- **Constraint:** Modification is editorial and ADR-scoped; modification by *amendment* or *supersession* is recorded as a new `DecisionRecord` with its own timestamps, **not** as a modification of the anchor record.

## Ontology Materialization

```ttl
### https://w3id.org/adr-o#decidedAt
adr-o:decidedAt rdf:type owl:AnnotationProperty ;
                 rdfs:domain adr-o:DecisionRecord ;
                 rdfs:range xsd:date ;
                 rdfs:comment "The date the decision was actually made (t₀). This is the moment of commitment, which may precede or follow record creation."@en ;
                 rdfs:label "decided at"@en ;
                 skos:scopeNote "Range is nominally xsd:date; xsd:dateTime is acceptable for automated tooling."@en ;
                 adr-o:justifiedBy adr-odr:ADR-0034-timestamp-decisionrecord .

### https://w3id.org/adr-o#dcterms:created
dcterms:created rdfs:comment "In ADR-O: the date the artifact was created. For post-factum ADRs this is the honest, post-t₀ reconstruction date."@en ;
                 adr-o:justifiedBy adr-odr:ADR-0034-timestamp-decisionrecord .

### https://w3id.org/adr-o#dcterms:modified
dcterms:modified rdf:type owl:AnnotationProperty ;
                   rdfs:domain adr-o:DecisionRecord ;
                   rdfs:range xsd:date ;
                   rdfs:comment "The date of the last editorial modification (Tier 2 change). Amendment events are recorded as new DecisionRecords, not as modifications here."@en ;
                   rdfs:label "modified"@en ;
                   skos:scopeNote "Pairs with dcterms:version to track artifact evolution without violating the Decision Transaction Principle."@en ;
                   adr-o:justifiedBy adr-odr:ADR-0034-timestamp-decisionrecord .
```

## Alternatives Considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| `prov:generatedAtTime` | Mixes PROV and DC namespaces unnecessarily; `dcterms:created` is more common for document artifacts. |
| `owl:DatatypeProperty` | `AnnotationProperty` is consistent with DC reuse and sufficient for metadata; reasoning is deferred to SHACL. |
| `dcterms:date` for decision | Overloaded in DC; `decidedAt` is semantically precise for t₀. |
| `dcterms:issued` | Publishing semantics don't fit decision records well. |
| `prov:Activity` model | Too verbose; decisions are entities, not processes. |

## Consequences

**Positive:**
- t₀ is explicitly anchored to a timestamp.
- Post-factum records are detectable (created > decided).
- Editorial staleness is queryable via `dcterms:modified`.
- Consistent with `authoredBy`/`decidedBy` social-role symmetry.
- Minimal namespace pollution (one new term).

**Negative / risks:**
- Authors must now provide three dates (though tooling can automate at least two).
- `xsd:date` range may be too coarse for some automated systems (mitigated by scope note).

## References

- [ADR-0015](/ADL/ADR-0015-dublin-core-and-prov-usage.md) — DC Terms usage.
- [ADR-0020](/ADL/ADR-0020-amendments.md) — Amendment topology.
- [ADR-0028](/ADL/ADR-0028-integrating-the-decision-transaction-principle.md) — DTP.
- [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md) — Reconstructability boundary.
