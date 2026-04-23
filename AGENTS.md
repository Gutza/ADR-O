# AGENTS.md — ADR-O

## Core Project
- **Ontology:** [`ontology/adr-o.ttl`](ontology/adr-o.ttl) — canonical OWL 2 ontology.
- **Decision Records:** [`ADL/*.md`](ADL/) — human-readable ADRs with YAML frontmatter.
- **ADR Registry:** [`ADL/ADL.yaml`](ADL/ADL.yaml) — **must be kept in sync** with all ADR files.
- **Design History:** [`ontology/DESIGN-NOTES.md`](ontology/DESIGN-NOTES.md) — versioned design rationale.
- **Namespace:** `https://w3id.org/adr-o#`

## Adding/Updating ADRs
1. Create `ADL/ADR-XXXX-title.md` with YAML frontmatter:
   ```yaml
   ---
   id: XX
   type: <Core design|Project governance|etc>
   status: <Proposed|Accepted|Deprecated|Superseded|Rejected>
   date: YYYY-MM-DD
   author:
     name: <Name>
     email: <Email>
   amends: [optional, IDs]
   amendedBy: [optional, IDs]
   ---
   ```
2. Add/update entry in `ADL/ADL.yaml` with matching `index`, `title`, `filename`, `status`, `summary`.
3. Use sequential IDs; amendments get new IDs, not suffixes (ADR-0024).
4. Link `amends`/`amendedBy` in both frontmatter and `ADL.yaml`.

## Ontology Contributions
- **Standard:** Turtle (`.ttl`) format.
- **Markdown literals:** Use datatype `^^<https://www.w3.org/ns/iana/media-types/text/markdown>` for `dcterms:description`, `skos:definition`, `skos:note` (and sub-properties).
- **Short labels:** `skos:prefLabel` is a plain string label, **not** Markdown.
- **Prose policy:** No Nygard body literals on records (ADR-0011). Prose lives in `Consideration` nodes.
- **Process:**
  - Substantive changes → new ADR in `ADL/` + `DESIGN-NOTES.md` entry.
  - Minor/tactical changes → `DESIGN-NOTES.md` entry only.
  - Update `owl:versionInfo` and `owl:versionIRI` in `adr-o.ttl` when advancing versions.

## Causal Scope Naming Canon

ADR-0025 establishes three canonical causal scope names. These names **must** be used consistently in all normative files (ADRs, ontology, roadmap). Any other form is drift.

| Canonical name | Banned aliases |
|:---|:---|
| **ADR scope** | `intra-ADR`, `Scope-1`, `within-record` |
| **ADL scope** | `inter-ADR`, `cross-ADR`, `intra-ADL`, `Scope-2` |
| **Project scope** | `Project-level scope`, `Scope-3` |

## Verification
- No automated test suite exists (SHACL deferred).
- **Manual verification:**
  - Ensure `ADL/ADL.yaml` matches `ADL/*.md` files.
  - Validate Turtle syntax with any RDF validator.
  - Verify `owl:AllDifferent` blocks over status and valence schemes (ADR-0008, ADR-0009).
  - Check that `chosenAlternative` is present for `Accepted` records.
