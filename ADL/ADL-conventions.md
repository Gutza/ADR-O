# ADL Conventions

## ADR Metadata Frontmatter

Every ADR in `ADL/` stores document metadata in YAML frontmatter.

```yaml
---
id: 24
type: Core design
status: Accepted
date: 2026-04-20
author:
  name: Bogdan Stăncescu
  email: bogdan@moongate.ro
amends:
  - ADR-0015
amendedBy:
  - ADR-0026
---
```

### Required keys for finalized ADRs

- `id`: sequential ADR identifier (for example, `ADR-0024`)
- `type`: human-readable ADR type label
- `status`: human-readable status label
- `date`: decision date, `YYYY-MM-DD`
- `author`: author string

### Optional keys

- `amends`: array of ADR IDs this record amends
- `amendedBy`: array of ADR IDs that amend this record

### Draft ADRs

Draft files may temporarily use `id: TBD` until a sequential ADR ID is assigned.

## Ontology Mapping

Frontmatter keys are author-facing. RDF semantics remain canonical through this mapping:

- `id` -> `dcterms:identifier`
- `type` -> `adr-o:hasType`
- `status` -> `adr-o:hasStatus`
- `date` -> `dcterms:date`
- `author` -> `dcterms:creator`
- `amends` -> `adr-o:amends`
- `amendedBy` -> `adr-o:amendedBy`

This keeps author UX simple while preserving alignment with `ontology/adr-o.ttl`.
