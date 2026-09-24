# Judging Criteria Alignment

## How MVA-Repurpose addresses each criterion

---

## Rigor — 35%

*Quality of evidence, methodology, citation, reproducibility.*

| Feature | How it addresses rigor |
|---|---|
| **Citation tracking** | Every evidence claim in Stage 5 carries a PMID. The `evidence_log.json` output maps every drug candidate claim to its source document(s). No claim enters the report without a traceable origin. |
| **Evidence taxonomy** | Claims are classified as `evidence` (directly observed in literature), `inference` (mechanistically derived), or `unknown` (genuinely unavailable). This prevents the conflation of known facts with reasonable speculation. |
| **Reproducible corpus** | ChromaDB persists the embedding corpus to disk. The same literature retrieval queries always produce the same corpus, making results reproducible without re-downloading. |
| **Auditable pipeline** | LangGraph's explicit state machine means each stage can be inspected, re-run in isolation, or rolled back. The pipeline does not skip stages or short-circuit on partial results. |
| **Known limitations documented** | The README's "Known Limitations" section documents scope boundaries honestly — single-patient dataset, literature-only (no wet lab), LLM hallucination risk. Rigor requires acknowledging what the system cannot claim. |
| **Cross-checking** | Knowledge graph relations extracted by the LLM are required to appear in at least 2 source documents before being marked `high_confidence`. Single-source relations are flagged. |

---

## Impact — 25%

*Clinical plausibility, patient benefit.*

| Feature | How it addresses impact |
|---|---|
| **Pathway-level reasoning** | Drug candidates are identified by reasoning about which nodes in the spindle assembly checkpoint pathway are druggable vulnerabilities given BUB1B loss-of-function — not by searching "drugs near BUB1B in a database." This grounds candidates in the actual mechanism of disease. |
| **Directionality** | Stage 3 explicitly reasons about the *direction* of required modulation (inhibit vs. activate) for each target. A drug that activates PLK1 is filtered out even if it hits the right target — because the therapeutic hypothesis requires *inhibition*. |
| **Approved-drugs-only filter** | Only drugs with approved status in any indication are considered. This maximises the plausibility of a near-term clinical path. |
| **Open questions surfaced** | Each candidate's `open_questions` field lists the specific experiments or data that would need to exist before clinical translation. This is the honest assessment of the gap between mechanistic hypothesis and patient benefit. |
| **MVA context maintained** | The reasoning is grounded in BUB1B *loss-of-function* in a *non-cancer* context, which is different from the cancer context where most of these drugs are studied. This distinction is maintained throughout synthesis. |

---

## Innovation — 25%

*Novel mechanism insight, creative drug candidate identification.*

| Feature | How it addresses innovation |
|---|---|
| **Evidence/inference/unknown taxonomy** | Applying a structured confidence taxonomy to drug repurposing candidates is novel in this context. Instead of a ranked list, each candidate has a decomposable evidence bundle that shows exactly *why* the confidence is what it is. |
| **Pathway graph as reasoning scaffold** | Rather than treating the drug repurposing problem as a similarity search (find drugs approved for diseases similar to MVA), the pipeline reasons over the biochemical pathway graph — asking which nodes, when modulated, partially restore the function lost by BUB1B disruption. |
| **Cross-domain literature** | The corpus spans spindle checkpoint biology, cancer mitotic therapy, and rare chromosomal instability — connecting insights from cancer drug development to a paediatric rare disease context where these drugs have not been studied. |
| **Scalable architecture** | The pipeline is a generalisation, not a one-off. Swapping the gene (BUB1B → another rare disease gene) and re-running produces a mechanistically grounded drug repurposing report for any disease with a known gene and pathway. This demonstrates that the *approach* is novel, not just the application. |

---

## Scalability — 15%

*Could this approach work for other rare diseases?*

| Feature | How it addresses scalability |
|---|---|
| **Disease-agnostic pipeline** | The six stages (literature retrieval, knowledge graph, target identification, drug screening, evidence synthesis, report generation) are not MVA-specific. The only MVA-specific inputs are the initial query terms and the known phenotype (BUB1B loss → premature CDC20 release). Any gene can be substituted. |
| **Parameterized queries** | Stage 1 takes a configurable list of search queries. Changing `gene: "BUB1B"` to `gene: "BRCA2"` or `gene: "CFTR"` adapts the entire retrieval. |
| **Synonym table** | `data/synonyms.json` maps gene/protein aliases. This is the main source of disease-specific configuration — a small, maintainable file per disease target. |
| **Demonstrated with a hard case** | MVA is an ideal scalability demonstration: it has no known treatment, fewer than 50 cases, and no direct drug target. If the pipeline produces mechanistically grounded candidates for MVA, it is likely to produce stronger results for more common rare diseases with richer literature. |
