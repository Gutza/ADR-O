# AGENTS.md — ADR-O

## Core Project
- **Ontology:** [`adr-o.ttl`](adr-o.ttl) — canonical OWL 2 ontology.
- **Decision Records:** [`ADL/*.md`](ADL/) — human-readable ADRs with YAML frontmatter.
- **ADR Registry:** [`ADL/ADL.yaml`](ADL/ADL.yaml) — **must be kept in sync** with all ADR files.
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
4. Mirror relationship fields in both frontmatter and `ADL.yaml` where applicable: `amends`, `amendedBy`, `supersedes`, `supersededBy`.

## Ontology Contributions
- **Standard:** Turtle (`.ttl`) format.
- **Literals:** Use datatype `^^adr-o:md` for `dcterms:description`, `skos:definition`, `skos:note` (and sub-properties) to write full Markdown (see ADR-0038) .
- **Short labels:** `skos:prefLabel` is a plain string label, **not** Markdown.
- **Prose policy:** No Nygard body literals on records (ADR-0011). Prose lives in `Claim` nodes.
- **Current core terms:** Use `ExpectedOutcome` (not `OutcomeFact`), `manifests` (not `consideration`), and `derivedFrom`/`derives` (identity reuse is not the cross-record model).
- **Observation loop:** Use `ObservedOutcome`, `verifies`, and `hasVerdict` for tₙ validation of t₀ expected outcomes (ADR-0030, ADR-0033).
- **Controlled vocabularies:** Keep status and valence terms aligned with ontology schemes: status (`Accepted`, `Proposed`, `Rejected`, `Deprecated`, `Superseded`), deliberation valence (`Supports`, `Against`, `Neutral`), expected outcome valence (`ExpectedGain`, `ExpectedCost`, `ExpectedRisk`, `ExpectedDependency`), observation verdict (`Satisfied`, `Violated`, `Inconclusive`).
- **Process:**
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
  - Verify `owl:AllDifferent` blocks over status, valence, and observation verdict schemes (ADR-0008, ADR-0009, ADR-0030).
  - Check that `chosenAlternative` is present for `Accepted` records.
