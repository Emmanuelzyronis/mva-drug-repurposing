# Implementation Plan

**Deadline:** October 24, 2026  
**Time available:** 4 weeks from September 24

---

## Week 1: Sep 24 – Oct 1 — Literature Foundation

**Goal:** Stage 1 (Literature Retrieval) fully working. ChromaDB populated and queryable.

### Tasks

- [ ] Set up Python project structure (`mva_repurpose/` package, `pyproject.toml`, `requirements.txt`)
- [ ] Configure `.env` with ANTHROPIC_API_KEY, HF_TOKEN, PUBMED_EMAIL
- [ ] Implement PubMed retrieval via `Bio.Entrez` with rate limiting and retry logic
- [ ] Implement bioRxiv retrieval via REST API
- [ ] Set up ChromaDB with persistent storage
- [ ] Implement text chunking (512 tokens, 64 overlap) and embedding ingestion
- [ ] Run initial retrieval for core queries:
  - `"BUB1B BubR1 spindle assembly checkpoint mitotic"`
  - `"Mosaic Variegated Aneuploidy chromosomal instability"`
  - `"spindle checkpoint CDC20 APC inhibitor drug"`
  - `"PLK1 Aurora kinase mitotic therapy"`
  - `"chromosomal instability rare disease drug repurposing"`
- [ ] Verify corpus: 200+ documents ingested, ChromaDB queryable
- [ ] Download VCF + phenotype from HuggingFace dataset
- [ ] Parse BUB1B variants from VCF, determine loss-of-function vs. hypomorphic

**Milestone check:** Query ChromaDB for "BubR1 CDC20 inhibition" and get relevant paper chunks back.

---

## Week 2: Oct 1 – Oct 8 — Knowledge Graph + Target Identification

**Goal:** Stages 2 and 3 working. Pathway graph built. Druggable targets identified with rationale.

### Tasks

- [ ] Implement LangGraph state machine (graph definition, node functions, edges)
- [ ] Stage 2: Claude tool-use prompt for entity-relation extraction from paper chunks
- [ ] Implement synonym resolution (`data/synonyms.json` for BUB1B / BubR1 / MAD3L etc.)
- [ ] Build pathway graph: nodes (proteins, complexes) + edges (inhibits, activates, phosphorylates)
- [ ] Validate graph manually: confirm BubR1→CDC20 inhibition edge present with high confidence
- [ ] Stage 3: Reason over graph to identify druggable vulnerabilities
- [ ] Query OpenTargets tractability annotations for identified targets
- [ ] Produce ranked target list with rationale and druggability scores
- [ ] Write tests for graph construction (at least: correct node count, BubR1→CDC20 edge present)

**Milestone check:** `targets.json` contains PLK1, Aurora A, Aurora B, CDK1 with documented rationale.

---

## Week 3: Oct 8 – Oct 15 — Drug Candidate Screening

**Goal:** Stage 4 working. Approved drug candidates identified for each target with full metadata.

### Tasks

- [ ] Integrate ChEMBL REST API (mechanism of action, approval status)
- [ ] Integrate OpenTargets GraphQL (drug-target associations, clinical evidence)
- [ ] Download and parse DrugBank public XML (offline, most complete approval data)
- [ ] Implement modulation-direction filter (inhibitor vs. activator)
- [ ] Implement approval-status filter (approved in any indication)
- [ ] Cross-reference all three sources per target; merge deduplicated candidate list
- [ ] Produce `candidates.json` with full metadata per drug
- [ ] Manual review: verify top candidates against known literature
- [ ] Expected candidates: Volasertib (PLK1), Alisertib (Aurora A), Barasertib (Aurora B), Vorinostat/Panobinostat (HDAC)

**Milestone check:** `candidates.json` has at least 4 candidates with ChEMBL ID, DrugBank ID, mechanism, approved indications.

---

## Week 4: Oct 15 – Oct 22 — Evidence Synthesis + Report

**Goal:** Stages 5 and 6 complete. Full report generated and self-reviewed.

### Tasks

- [ ] Implement Stage 5: evidence synthesis with Claude Opus
- [ ] For each candidate: retrieve relevant ChromaDB chunks (MMR-ranked)
- [ ] Classify each claim as evidence / inference / unknown
- [ ] Compute confidence score per candidate
- [ ] Document open questions per candidate
- [ ] Implement Stage 6: generate `report.md`, `report.json`, `evidence_log.json`
- [ ] Self-review: read the full report and check every claim against its cited source
- [ ] Add honest acknowledgements of limitations to each candidate section
- [ ] Test deterministic fallback: run pipeline with `ANTHROPIC_API_KEY=invalid`, confirm Gemini fallback produces a coherent report

**Milestone check:** Full report generated. Top candidate section reads cleanly with evidence/inference/unknown breakdown visible.

---

## Final Days: Oct 22 – Oct 24 — Polish + Submit

**Goal:** Submission-ready.

### Tasks

- [ ] Run full pipeline end-to-end from clean state (no cached ChromaDB)
- [ ] Verify reproducibility: second run produces identical candidates
- [ ] Update README with actual output statistics (corpus size, candidate count)
- [ ] Record a short demo showing pipeline run and report output
- [ ] Write submission description for HuggingFace Space
- [ ] Submit to: https://huggingface.co/spaces/SageBio/rare-disease-real-kid-mva-hackathon-2026
- [ ] Final check: all five judging criteria explicitly addressed in the submission write-up

---

## Risk Register

| Risk | Likelihood | Mitigation |
|---|---|---|
| Literature retrieval too slow (100+ papers) | Medium | Pre-run retrieval in Week 1; cache ChromaDB |
| ChromaDB scaling issues at 300+ docs | Low | Tested at similar scale in FreshIndex |
| Claude API rate limits during synthesis | Low | Gemini fallback; batch requests |
| DrugBank XML too large to parse efficiently | Low | Filter to relevant targets before full parse |
| Candidate list empty (no approved drugs on pathway) | Low | Pathway has 4+ known drug targets in oncology |
| Submission format mismatch | Low | Read HuggingFace Space submission guide in Week 4 |
