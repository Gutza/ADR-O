# ADR-0014 — Ontology Document Shape: Namespace, Version IRIs, No `owl:imports`

| Field        | Value |
|--------------|-------|
| ID           | ADR-0014 |
| Type         | Core vocabulary |
| Status       | Accepted |
| Date         | 2026-04-19 |
| Author       | Bogdan Stăncescu <bogdan@moongate.ro> |

## Context

ADR-O reuses Dublin Core Terms, SKOS, and PROV-O by **namespace prefix**, aligning terms without pulling entire external ontologies into the reasoning closure. The canonical ontology document also needs stable identification and version metadata for publication and tooling.

Repository layout choices (single Turtle file, additional serializations, example folders) are **not** asserted as RDF in the ontology file itself; they belong in project documentation rather than this ADR.

## Decision

**Namespace:** terms live under **`https://w3id.org/adr-o#`** (hash IRIs), reflected in `@base` and the `adr-o:` prefix in [`ontology/adr-o.ttl`](/ontology/adr-o.ttl).

**Ontology resource:** the ontology IRI `https://w3id.org/adr-o` is typed `owl:Ontology` and carries **`owl:versionIRI`**, **`owl:versionInfo`**, and Dublin Core metadata (`dcterms:title`, `dcterms:description`, `dcterms:license`, etc.) as in the current file.

**No `owl:imports`:** the canonical `adr-o.ttl` does **not** import PROV-O, SKOS, or DC Terms as OWL imports; reused predicates and classes are referenced with **declarations only where needed** for annotations and axioms local to ADR-O.

## Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| `owl:imports` of PROV/SKOS/DC | Heavier reasoner load for modest gain at current scope; consumers needing full external axiom sets can import separately. |
| Slash namespace per term | More hosting complexity; hash IRIs suit a compact vocabulary. |

## Consequences

**Positive.** Smaller effective imports footprint; clear versioned ontology IRI for releases.

**Negative / risks.** Downstream tools must not assume full SKOS/PROV inferencing from imports alone.

## References

- [`ontology/adr-o.ttl`](/ontology/adr-o.ttl) — ontology header, `@base`, prefixes (no `owl:imports` in document).
- ADR-0000 — Inception Record (reuse strategy at high level).
