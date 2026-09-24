# Pipeline Architecture

## Overview

MVA-Repurpose is a six-stage LangGraph pipeline. Each stage has a defined input contract, output contract, and documented failure modes. The state machine is explicit — failure at any stage is surfaced immediately rather than silently propagated.

```
┌─────────────────────┐
│  1. Literature      │  PubMed / bioRxiv → ChromaDB
│     Retrieval       │
└────────┬────────────┘
         │
┌────────▼────────────┐
│  2. Knowledge       │  Paper excerpts → Pathway graph (nodes + edges)
│     Graph           │
└────────┬────────────┘
         │
┌────────▼────────────┐
│  3. Target          │  Pathway nodes → Druggable targets (with directionality)
│     Identification  │
└────────┬────────────┘
         │
┌────────▼────────────┐
│  4. Drug Candidate  │  Targets → Approved drug candidates (DrugBank/ChEMBL/OpenTargets)
│     Screening       │
└────────┬────────────┘
         │
┌────────▼────────────┐
│  5. Evidence        │  Candidates → Per-candidate evidence bundles
│     Synthesis       │     (evidence / inference / unknown taxonomy)
└────────┬────────────┘
         │
┌────────▼────────────┐
│  6. Report          │  Evidence bundles → report.md + report.json + evidence_log.json
│     Generation      │
└─────────────────────┘
```

---

## Stage 1: Literature Retrieval

**Purpose:** Build the corpus of domain knowledge. Every downstream claim must be traceable to a document retrieved here.

**Input:**
```python
{
    "queries": [
        "BUB1B BubR1 spindle assembly checkpoint",
        "Mosaic Variegated Aneuploidy BUB1B",
        "BubR1 drug target mitotic checkpoint",
        "spindle checkpoint CDC20 APC/C inhibitor",
        "PLK1 Aurora kinase chromosomal instability"
    ],
    "max_results_per_query": 100,
    "sources": ["pubmed", "biorxiv"]
}
```

**Output:**
```python
{
    "documents": [
        {
            "pmid": "17426725",
            "title": "The spindle-assembly checkpoint in space and time",
            "abstract": "...",
            "full_text_url": "...",
            "source": "pubmed",
            "embedding_id": "chroma_doc_abc123"
        }
    ],
    "corpus_size": 347,
    "chroma_collection": "mva_literature_v1"
}
```

**Implementation notes:**
- Uses `Bio.Entrez` (Biopython) for PubMed queries with polite rate limiting (3 req/sec)
- bioRxiv via their REST API
- Text chunked at 512 tokens with 64-token overlap using `RecursiveCharacterTextSplitter`
- Embeddings via Claude `claude-3-haiku` (fast, cheap for embedding workloads)
- ChromaDB collection persisted to `$CHROMA_PERSIST_DIRECTORY` — deterministic re-runs skip re-download

**Failure modes:**
- NCBI rate limit → exponential backoff, max 5 retries
- Paper unavailable (paywall) → abstract-only embedding, flagged in document metadata
- ChromaDB write failure → pipeline halts, state preserved, resumable

---

## Stage 2: Knowledge Graph

**Purpose:** Extract structured pathway relationships from the literature corpus. Produce a graph of biological entities (proteins, complexes, processes) and their relationships.

**Input:** ChromaDB collection from Stage 1

**Output:**
```python
{
    "nodes": [
        {"id": "BUB1B", "type": "gene", "aliases": ["BubR1", "MAD3L"]},
        {"id": "CDC20", "type": "protein", "role": "APC/C co-activator"},
        {"id": "APC_C", "type": "complex", "function": "ubiquitin ligase, triggers anaphase"}
    ],
    "edges": [
        {"from": "BUB1B", "to": "CDC20", "relation": "inhibits", "confidence": 0.95, "sources": ["17426725", "9660858"]},
        {"from": "PLK1", "to": "CDC20", "relation": "phosphorylates_activates", "confidence": 0.88, "sources": ["..."]}
    ]
}
```

**Implementation notes:**
- Claude API with structured output (tool-use) to extract entity-relation triples from paper chunks
- Retrieved chunks ranked by MMR (maximum marginal relevance) to reduce redundancy
- Confidence score = fraction of retrieved papers supporting the relation
- Graph stored as adjacency list JSON; visualised with networkx if needed

**Failure modes:**
- LLM extraction hallucination → cross-check extracted relations against at least 2 source documents; relations with only 1 source are marked `low_confidence`
- Ambiguous entity resolution (e.g., "BUB1" vs "BUB1B") → synonym table maintained manually in `data/synonyms.json`

---

## Stage 3: Target Identification

**Purpose:** Given the pathway graph and the known consequence of BUB1B loss (premature CDC20 release → premature APC/C activation → aneuploidy), identify which pathway nodes represent druggable vulnerabilities.

**Input:** Knowledge graph from Stage 2

**Output:**
```python
{
    "targets": [
        {
            "protein": "PLK1",
            "rationale": "PLK1 phosphorylates CDC20 at T70/S114 to promote APC/C activation. Inhibiting PLK1 would slow the same downstream event that BubR1 normally prevents by sequestering CDC20.",
            "modulation": "inhibition",
            "druggability": "high",
            "approved_drugs_known": True
        }
    ]
}
```

**Implementation notes:**
- Reasoning step: Claude prompted with the graph + known BUB1B loss phenotype → asks "which nodes, when modulated, would partially compensate for loss of BubR1's CDC20-inhibitory function?"
- Druggability assessment cross-referenced with OpenTargets tractability annotations
- Only targets with `druggability: high | medium` passed to Stage 4

**Failure modes:**
- No druggable targets identified → pipeline completes with `targets: []` and a documented explanation; report states honest null result

---

## Stage 4: Drug Candidate Screening

**Purpose:** For each identified target, find approved drugs that act on it with the correct modulation direction.

**Input:** Target list from Stage 3

**Output:**
```python
{
    "candidates": [
        {
            "drug": "Volasertib",
            "target": "PLK1",
            "modulation": "inhibition",
            "approved": True,
            "indications": ["AML (Phase III)"],
            "chembl_id": "CHEMBL1079846",
            "drugbank_id": "DB11642",
            "mechanism": "Competitive ATP-site inhibitor of PLK1",
            "clinical_stage": "Phase III (not approved for MVA indication)"
        }
    ]
}
```

**APIs queried:**
- ChEMBL REST API: `https://www.ebi.ac.uk/chembl/api/data/` — mechanism of action, approval status
- OpenTargets GraphQL: `https://api.platform.opentargets.org/api/v4/graphql` — known drug-target associations, clinical evidence
- DrugBank (via public XML download) — drug details, interaction data

**Failure modes:**
- API unavailable → cached response used if available; otherwise stage retries up to 3 times
- Drug found but wrong modulation direction (e.g., PLK1 activator instead of inhibitor) → filtered out with explanation logged

---

## Stage 5: Evidence Synthesis

**Purpose:** For each drug candidate, build a structured evidence bundle. Classify each supporting claim as evidence, inference, or unknown.

**Input:** Drug candidates from Stage 4 + ChromaDB corpus from Stage 1

**Output per candidate:**
```python
{
    "drug": "Volasertib",
    "evidence_bundle": [
        {
            "type": "evidence",
            "claim": "PLK1 phosphorylates CDC20 at T70 and S114, promoting APC/C activation",
            "source_pmid": "PMC3090439",
            "source_excerpt": "..."
        },
        {
            "type": "inference",
            "claim": "PLK1 inhibition may reduce premature APC/C activation in BUB1B-deficient cells",
            "reasoning": "If PLK1 cannot phosphorylate CDC20, CDC20 is less able to activate APC/C prematurely. This partially mimics the CDC20-sequestration function of BubR1."
        },
        {
            "type": "unknown",
            "claim": "Whether PLK1 inhibition is tolerated at therapeutic doses in non-cancer proliferating cells with constitutive SAC deficiency"
        }
    ],
    "confidence_score": 0.62,
    "open_questions": [...]
}
```

**Implementation notes:**
- Evidence: claims directly supported by retrieved literature, with PMID citation
- Inference: mechanistically reasonable claims derived from the pathway graph; labelled as such
- Unknown: gaps that would need experimental validation; surfaced explicitly rather than omitted
- Confidence score: weighted average of evidence type (evidence: 1.0, inference: 0.5, unknown: 0.0) across the bundle
- Claude `claude-opus` for synthesis (highest reasoning quality for the most important stage)

**Failure modes:**
- LLM produces claim not supported by retrieved documents → cross-check against ChromaDB; unsupported claims downgraded to `inference` or `unknown`
- All claims for a candidate resolve to `unknown` → candidate flagged as `speculative`, included in report with explicit warning

---

## Stage 6: Report Generation

**Purpose:** Produce the final structured output.

**Outputs:**
- `output/report.md` — human-readable Markdown report for submission
- `output/report.json` — machine-readable structured data
- `output/evidence_log.json` — complete citation trail, one entry per evidence claim

**Report structure:**
1. Executive summary (3–5 drug candidates, ranked by confidence score)
2. Per-candidate sections (full evidence bundle, open questions, clinical context)
3. Methodology (pipeline description, corpus statistics, known limitations)
4. Appendix (full evidence log with citations)

**Failure modes:**
- Stage completes even if upstream stages produced partial results — partial results are documented as such rather than silently omitted
