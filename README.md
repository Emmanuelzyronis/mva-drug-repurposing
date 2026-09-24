# MVA Rare Disease Hackathon 2026 — Track 2: Drug Repurposing

**Event:** Rare Disease, Real Kid: MVA Hackathon 2026  
**Organizers:** Sage Bionetworks · HuggingFace · MVA Society · BEACON  
**Prize:** $25,000 cash (AWS) + $25,000 Claude API credits (Anthropic) = $50,000 total  
**Track:** Track 2 — Drug Repurposing  
**Deadline:** October 24, 2026  
**HuggingFace Space:** https://huggingface.co/spaces/SageBio/rare-disease-real-kid-mva-hackathon-2026

---

## The Disease

**Mosaic Variegated Aneuploidy (MVA)** is an ultra-rare chromosomal instability syndrome with fewer than 50 confirmed cases worldwide. Patients carry cells with an abnormal number of chromosomes (aneuploidy), distributed unevenly across tissues (mosaic).

The root cause in this patient is **biallelic compound heterozygous mutations in BUB1B** — the gene encoding **BubR1**, a core component of the spindle assembly checkpoint (SAC). The SAC is the cell's quality control gate for chromosome segregation: it delays cell division until every chromosome is correctly attached to the mitotic spindle. When BubR1 is disrupted, the checkpoint fires prematurely, chromosomes missegregate, and aneuploidy accumulates with each cell division.

**No established treatment exists.** This hackathon asks: given what we know about the broken pathway, are there approved drugs that could plausibly correct or compensate for it?

See [`docs/MECHANISM.md`](docs/MECHANISM.md) for the full mechanistic background.

---

## Project: MVA-Repurpose

MVA-Repurpose is a **LangGraph-orchestrated drug repurposing pipeline** that reasons from mechanism to candidate — not from keyword to drug list. It characterizes the BUB1B/BubR1 pathway disruption using retrieved literature, identifies which pathway nodes are druggable by approved compounds, cross-references public drug databases, and synthesizes per-candidate evidence with an explicit confidence taxonomy.

Every claim traces to a source. Every unknown is surfaced rather than collapsed. The pipeline is auditable stage by stage.

### Why not just query DrugBank for "BUB1B"?

Keyword matching finds drugs that have been explicitly studied against the target. Drug repurposing for ultra-rare diseases requires reasoning about the *disrupted pathway*, not the mutated gene alone — because no approved drug targets BUB1B directly. The question is: which nodes *downstream or adjacent* to BubR1 in the spindle checkpoint are druggable, and what approved drugs act on them?

That is a mechanistic reasoning problem, not a lookup.

---

## Pipeline Stages

```
Literature Retrieval → Knowledge Graph → Target Identification → Drug Screening → Evidence Synthesis → Report
```

| # | Stage | What it does |
|---|---|---|
| 1 | **Literature Retrieval** | PubMed/bioRxiv search on BUB1B, BubR1, SAC, MVA, chromosomal instability. Papers embedded and stored in ChromaDB. |
| 2 | **Knowledge Graph** | Extract pathway relationships from retrieved papers. Nodes: proteins (BubR1, MAD2, BUB3, CDC20, PLK1, Aurora A/B, APC/C). Edges: activates, inhibits, recruits, phosphorylates. |
| 3 | **Target Identification** | For each pathway node: is there an approved drug that modulates it? Cross-reference against the knowledge graph edges to identify which modulation direction is therapeutically relevant given BUB1B loss-of-function. |
| 4 | **Drug Candidate Screening** | Query DrugBank, ChEMBL, OpenTargets for approved drugs hitting identified targets. Filter: approved status, human indication, oral/IV availability. |
| 5 | **Evidence Synthesis** | Per candidate: mechanism of action, supporting literature (with citations), clinical context, confidence assessment. Classify each piece of evidence as: **evidence** (directly observed), **inference** (mechanistically reasonable), or **unknown** (not available). |
| 6 | **Report Generation** | Structured JSON + Markdown report. Each candidate has: drug name, target, MoA, evidence bundle, confidence score, open questions. |

See [`docs/PIPELINE.md`](docs/PIPELINE.md) for input/output contracts and documented failure modes.

---

## How This Addresses the Judging Criteria

| Criterion | Weight | How MVA-Repurpose addresses it |
|---|---|---|
| **Rigor** | 35% | Every drug candidate cites retrieved literature. ChromaDB stores the embedding corpus reproducibly. LangGraph stages are individually auditable. Known limitations are documented honestly (see below). |
| **Impact** | 25% | Pathway-level mechanistic reasoning — candidates are grounded in *why* the pathway disruption creates a druggable vulnerability, not just what genes are near BUB1B in a database. |
| **Innovation** | 25% | Evidence/inference/unknown taxonomy applied to drug confidence — the system surfaces what it doesn't know rather than collapsing uncertainty into a score. Novel cross-referencing of spindle checkpoint biology with cancer drug repurposing literature. |
| **Scalability** | 15% | The pipeline is disease-agnostic. Swap the gene (BUB1B → any rare disease gene) and the pathway characterization, target identification, and drug screening stages run identically. |

See [`docs/JUDGING_ALIGNMENT.md`](docs/JUDGING_ALIGNMENT.md) for the detailed mapping.

---

## Tech Stack

| Tool | Role | Why |
|---|---|---|
| **Python 3.12** | Runtime | Typed domain model, async workers |
| **LangGraph** | Pipeline orchestration | Explicit state machine — each stage is auditable, resumable, and testable independently |
| **LangChain** | Document retrieval + splitting | PubMed integration, text chunking for ChromaDB ingestion |
| **ChromaDB** | Literature embedding store | Persistent, reproducible — the same corpus produces the same retrieval results |
| **Anthropic Claude API** | Primary reasoning model | Prize sponsor; strongest at structured reasoning and evidence synthesis |
| **Google Gemini** | Fallback model | Deterministic fallback if Claude unavailable; interface identical |
| **Biopython** | VCF/FASTQ parsing | For variant context from the WGS dataset if needed |
| **HuggingFace datasets** | Dataset access | Downloads the 85GB WGS data from `SageBio/mva-hackathon-2026-data` |
| **FastAPI** | Report API | Serve the structured drug candidate report |
| **Prometheus + Grafana** | Pipeline monitoring | Track stage durations and retrieval coverage |

---

## Setup

### Prerequisites
- Python 3.12+
- 100–150 GB free disk space (for the WGS dataset, optional for Track 2)
- HuggingFace account with `HF_TOKEN` (for dataset download)
- Anthropic API key

### Install

```bash
git clone https://github.com/Emmanuelzyronis/mva-drug-repurposing.git
cd mva-drug-repurposing
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### Configure

```bash
cp .env.example .env
# Fill in ANTHROPIC_API_KEY, HF_TOKEN, etc.
```

### Download dataset (optional — Track 2 is literature-based)

See [`docs/DATASET.md`](docs/DATASET.md) for the full download guide.

### Run

```bash
python -m mva_repurpose.pipeline
```

The pipeline runs all six stages and writes:
- `output/report.md` — human-readable drug candidate report
- `output/report.json` — structured JSON for downstream use
- `output/evidence_log.json` — full citation trail per candidate

---

## Output Format

Each drug candidate in the report:

```json
{
  "drug": "Volasertib",
  "target": "PLK1",
  "mechanism": "Competitive ATP-site inhibitor of Polo-like kinase 1",
  "pathway_rationale": "PLK1 phosphorylates and activates CDC20, which activates APC/C to degrade securin and cyclin B — the same downstream effectors that BubR1 normally holds in check. PLK1 inhibition may partially compensate for checkpoint bypass.",
  "evidence": [
    {"type": "evidence", "claim": "PLK1 phosphorylates CDC20 at T70 and S114, promoting APC/C activation", "source": "PMC3090..."},
    {"type": "inference", "claim": "PLK1 inhibition may slow premature APC/C activation in BUB1B-deficient cells"},
    {"type": "unknown", "claim": "Whether PLK1 inhibition is tolerated in non-cancer cells with high SAC deficiency"}
  ],
  "confidence_score": 0.62,
  "approved_indications": ["AML (Phase III)"],
  "open_questions": ["Therapeutic window in post-mitotic vs. proliferating tissues", "Germline vs. somatic cell targeting"]
}
```

---

## Timeline

| Period | Milestone |
|---|---|
| Sep 24 – Oct 1 | Literature retrieval pipeline working. ChromaDB populated with BUB1B/SAC papers from PubMed. |
| Oct 1 – Oct 8 | Knowledge graph construction. Pathway nodes and edges extracted. Target identification complete. |
| Oct 8 – Oct 15 | Drug candidate screening. DrugBank, ChEMBL, OpenTargets integrations working. |
| Oct 15 – Oct 22 | Evidence synthesis and report generation. Per-candidate confidence scoring. |
| Oct 22 – Oct 24 | Validation, known-limitations documentation, final report polish, submission. |

---

## Known Limitations

*These are documented here deliberately — rigor requires honest scope boundaries.*

1. **Single-patient dataset.** All reasoning is grounded in one patient's WGS. Drug candidates may not generalise to all BUB1B variants.
2. **Literature coverage.** PubMed retrieval is comprehensive but not exhaustive. Preprints without PMID indexing may be missed.
3. **In silico only.** No wet-lab validation. Drug candidates are mechanistically plausible, not experimentally confirmed.
4. **Approved-drug filter.** The pipeline considers only approved drugs to maximise clinical plausibility — this excludes investigational compounds that may be mechanistically relevant.
5. **Model reasoning errors.** The synthesis stage uses Claude/Gemini. LLM outputs are grounded in retrieved documents, but hallucinations are possible — every claim should be independently verified against the cited source before clinical consideration.

---

## Submission Links

- **HuggingFace Challenge Space:** https://huggingface.co/spaces/SageBio/rare-disease-real-kid-mva-hackathon-2026
- **Dataset:** https://huggingface.co/datasets/SageBio/mva-hackathon-2026-data
- **Submission period:** August 24 – October 24, 2026
