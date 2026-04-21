# ADR-0007 — Numeric `adr-o:index` vs String `dcterms:identifier`

| Field        | Value |
|--------------|-------|
| ID           | ADR-0007 |
| Type         | Core vocabulary |
| Status       | Accepted |
| Date         | 2026-04-19 |
| Author       | Bogdan Stăncescu <bogdan@moongate.ro> |
| Amended by   | ADR-0024 |

## Context

Human-readable decision identifiers (for example `ADR-0042`) are essential for documents and conversation, but they do not sort reliably in SPARQL for organisational-history and log-order queries. A separate numeric index was introduced during ontology design to support native ordering without parsing identifier strings.

## Decision

**Use both**, with distinct roles:

- **`dcterms:identifier`** — string identifier of a `DecisionRecord` (e.g. `ADR-0042`), as usual in Dublin Core.
- **`adr-o:index`** — `xsd:integer`, `owl:FunctionalProperty` on `adr-o:DecisionRecord`, for sortable ordering within a log and for traversing supersession chains in intended sequence.

The ontology documents this split in the `rdfs:comment` on `adr-o:index` and in the `skos:scopeNote` on `dcterms:identifier` in [`ontology/adr-o.ttl`](/ontology/adr-o.ttl).

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Only string identifiers | Sorting and numeric filters become string hacks; error-prone across locales and ID formats. |
| Only integers | Loses human-facing IDs that match repository filenames and team vocabulary. |

## Consequences

**Positive.** SPARQL `ORDER BY ?index` and range filters work without parsing labels.

**Negative / risks.** Authors must maintain two fields consistently; tooling can assist.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — `adr-o:index`; `dcterms:identifier` scope note.
