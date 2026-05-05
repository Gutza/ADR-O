# ADR-O Tooling-Mediation Principle

This article is a lifted transcription of [ADR-0004](/ADL/ADR-0004-kg-under-tooling.md), maintained as practitioner-facing guidance under ADR-0042.

## Context

Several decisions in the ADR-O 0.2.0-draft iteration were made with reference to a shared background assumption: that ADR-O graphs are not user-facing artifacts but substrate - mediated by a tooling layer before any human author or reader encounters them.

This assumption deserves its own record because its scope is non-trivial: it is not a claim about how the tooling should be structured, or which layer enforces which constraint, but a claim about the expected operating environment of ADR-O graphs and the design trade-offs that become available in that environment.

The expected tooling stack includes some combination of: AI agents that draft, summarize, and answer natural-language questions about the KG; APIs that mediate programmatic access with shape enforcement; MCP servers that expose structured tools to AI agents; and GUIs for authoring, visualization, and navigation. The assumption does not require that all of these be present in every deployment. It asserts that at least one mediation layer is expected, and that designing for zero-mediation is not the primary case.

## Decision

**ADR-O graphs are authoritative substrate, not user surface.** The expected operating environment of an ADR-O deployment includes some automated mediation between the RDF graph and human authors or readers. The ontology is designed for this layered world, not for a hypothetical use case in which someone writes and reads Turtle without further tooling.

A load-bearing design heuristic follows from this stance: when choosing between two ontology shapes, prefer the one that pushes incidental complexity into the tooling layer in exchange for a cleaner, more traversable, and more complete KG - *unless* the complexity in question is something that no reasonable tooling implementation can reconstruct from the graph alone.

This is the **reconstructability boundary rule**: if a representation detail can be reconstructed from the graph by reasonable tooling, keep it out of the ontology shape; if it cannot be reconstructed reliably, it belongs in the graph as first-class structure. Future ADRs that cite this decision should state explicitly which side of that boundary the contested detail falls on.

One concrete corollary is already on record. ADR-0003 commits prose-carrying literals to the IANA-rooted Markdown datatype; the tooling layer to which this ADR commits ADR-O is expected to render that datatype natively. Renderers, GUIs, and AI-agent-facing surfaces alike are responsible for turning Markdown literals into the human-facing prose they encode, not for surfacing them as raw text. ADR-0003 documents the convention on the graph side; this ADR supplies the tooling-side half that closes the loop and makes "Markdown-typed literal" an operationally meaningful signal rather than a metadata annotation no layer acts on.

### Alternatives considered

| Alternative | Reason not chosen |
|-------------|-------------------|
| Keep the assumption implicit | Implicit assumptions cannot be cited by later decisions; they must be re-derived or re-argued each time, and are invisible to anyone checking choices for design-intentionality consistency. The fact that the 0.2.0-draft notes had to name it mid-iteration confirms it was doing hidden work without being on record. |
| Assume raw consumption as the primary use case | Forecloses atom-first design: consumers seeing raw `Consideration` IRIs without materialised prose would need the KG to carry redundant human-facing copies of all content. Also makes SHACL a prerequisite rather than a companion and puts the ontology in the impossible position of encoding both structured graph and prose representation. |
| Treat tooling mediation as a domain-profile concern (core-neutral) | The core ontology's own design choices already embody the assumption - no Nygard literals, `rdf:List` ordering, conventions in scope notes over axioms. Declaring the core neutral on this point would misrepresent its actual design stance and create false expectations for any implementer who tries to consume the core without mediation. |

## Consequences

**Positive.**
- Future ADRs and design notes can cite this decision by ID rather than re-arguing the assumption from scratch; design-intentionality consistency becomes checkable.
- The "push incidental complexity to tooling" heuristic becomes a named, citable principle rather than an informal rule of thumb applied ad hoc.
- The design boundary ("what tooling can reconstruct vs. what must be in the graph") is on record as a distinction future iterations should address explicitly when it arises.
- ADR-0003's commitment of prose literals to a Markdown datatype is retroactively reinforced: the mediation layer this ADR names is the same layer expected to render those literals. The two ADRs form a coherent pair - ADR-0003 on the graph side, ADR-0004 on the tooling side - and neither stands fully on its own.

**Negative / risks.**
- The assumption fails for consumers who attempt to use ADR-O graphs raw, with no tooling. They encounter a more skeletal model than the tooling-mediated human-facing view suggests: `Consideration` nodes carrying text payloads rather than Nygard-shaped prose, Markdown literals surfaced as raw text rather than rendered, `rdf:List` ordering that some lightweight readers handle awkwardly, integrity rules absent rather than enforced. This is the deliberate trade-off, not an oversight - but it must be stated plainly so that implementers choosing a raw-consumption deployment are not surprised.

**Open questions.**
- No single follow-up ADR is queued. The reconstructability boundary rule named in the Decision is a recurring live question rather than a point deferral: each future occurrence is expected to be resolved in context, citing this ADR and naming why the detail is or is not reconstructable from the graph.

## References

- ADR-0000 - Inception Record. Establishes the machine-traversable RDF graph as the design goal; this ADR specifies the access model under which that graph is intended to operate.
- ADR-0002 - Scope. Establishes the domain-agnostic core; this ADR is orthogonal to scope but shares the same design posture: prefer clean, general structure over domain-baked conventions.
- ADR-0003 - Prose Literals Are Markdown. Defines the Markdown datatype for prose-carrying literals. This ADR commits the tooling layer to rendering that datatype natively and thereby supplies the tooling-side half of ADR-0003's convention; the two ADRs should be read as a pair.
