# ADR-0001 — Project License: Creative Commons Attribution 4.0 International

| Field        | Value |
|--------------|-------|
| ID           | ADR-0001 |
| Type         | Project governance |
| Status       | Accepted |
| Date         | 2026-04-17 |
| Author       | Bogdan Stăncescu <bogdan@moongate.ro> |

---

## Context

ADR-O is an ontology — a knowledge artefact — not a software library. This distinction matters for licensing because the two dominant open-source software licenses (MIT, Apache 2.0) were designed for code: they speak to executable copying, distribution, and patent grants in ways that map awkwardly onto RDF graphs and OWL axiom sets. A license chosen for an ontology should be legible to the semantic web and linked data communities, not only to software engineers.

Several constraints shaped the decision space:

1. **Attribution.** The ontology represents original modelling work and embeds design rationale. Requiring attribution on reuse is reasonable and common in the field.
2. **Ecosystem alignment.** The OBO Foundry (the most active community for formal ontologies) recommends CC BY 3.0 or CC BY 4.0. OntolAD, the closest prior work surveyed in ADR-0000, is licensed CC BY 4.0. W3C ontologies and schema.org use CC BY or CC0. Apache 2.0 is an outlier in this space.
3. **Machine-readability.** Regardless of which license is chosen, the canonical declaration for an OWL ontology is an annotation on the ontology IRI via `dcterms:license`, not a `LICENSE` file in a repository. The `LICENSE` file is supplementary.

### Licenses considered

The candidate set included both software licenses (from GitHub's picker) and Creative Commons instruments. The shortlist after the ontology-vs-software framing:

| License | Attribution required | ShareAlike | Copyleft | OBO/linked-data norm |
|---------|---------------------|------------|----------|----------------------|
| CC0 1.0 | No | No | No | Preferred by some OBO members |
| CC BY 4.0 | Yes | No | No | Recommended by OBO Foundry; used by OntolAD, schema.org extensions |
| CC BY-SA 4.0 | Yes | Yes | Weak | Uncommon for ontologies |
| Apache 2.0 | Yes (NOTICE file) | No | No | Occasional; includes patent grant irrelevant to ontologies |
| MIT | Yes (header) | No | No | Rare for ontologies; no patent clause |

---

## Decision

**License ADR-O under Creative Commons Attribution 4.0 International (CC BY 4.0).**

### Why not CC0

CC0 maximises reuse friction-free, and some members of the OBO community prefer it precisely because attribution requirements can complicate automated pipelines. However, for a pre-1.0 ontology without an established community, requiring attribution is a lightweight mechanism to keep the provenance chain visible during the period when the design is most likely to be absorbed piecemeal into other works. This can be revisited if attribution becomes a genuine barrier to adoption.

### Why not Apache 2.0

Apache 2.0 is a reasonable choice for ontologies bundled with significant code tooling, because the patent grant matters there. ADR-O has no code artefacts at this stage. Choosing Apache 2.0 would be legible to software engineers but would mark ADR-O as an outlier to the ontology community it is targeting.

---

## Consequences

**Positive.**
- Aligns ADR-O with the dominant license in the formal ontology ecosystem, reducing friction for potential contributors familiar with OBO Foundry or W3C conventions.
- Attribution is required, preserving visible provenance of modelling choices during the early, volatile period of the ontology's design.
- The `dcterms:license` annotation makes the license machine-readable to any SPARQL client or ontology browser without parsing a repository file.

**Negative / risks.**
- If ADR-O is later adopted by a community that prefers CC0 (e.g., an OBO working group), re-licensing would require contacting all prior contributors — a standard but non-trivial process.

---

## References

- Creative Commons. *CC BY 4.0 Deed.* https://creativecommons.org/licenses/by/4.0/
- Creative Commons. *CC BY 4.0 Legal Code.* https://creativecommons.org/licenses/by/4.0/legalcode
- OBO Foundry. *Principles — Licenses.* https://obofoundry.org/principles/fp-001-open.html
- ADR-0000 — Inception Record (this ADL). OntolAD is noted therein as CC BY 4.0; this decision aligns ADR-O's license with that prior work.
