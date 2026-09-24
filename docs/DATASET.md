# Dataset Guide

## What's in the dataset

**Dataset:** `SageBio/mva-hackathon-2026-data`  
**URL:** https://huggingface.co/datasets/SageBio/mva-hackathon-2026-data  
**Size:** ~85 GB compressed (100–150 GB uncompressed — plan storage accordingly)

The dataset contains whole-genome sequencing (WGS) data from a single paediatric patient with confirmed Mosaic Variegated Aneuploidy caused by biallelic BUB1B compound heterozygous mutations. The NHS-confirmed diagnosis and variant calls are included.

**Contents:**
- **FASTQ lanes** — raw paired-end sequencing reads (gzip-compressed, ~75 GB)
- **VCF** — variant calls (SNVs, indels, structural variants)
- **Clinical phenotype** — structured phenotype description in HPO terms
- **QC metrics** — sequencing depth, coverage, alignment statistics

---

## Track 2 and the dataset

**Track 2 (Drug Repurposing) is primarily literature-based.** You do not need to process the full 85 GB WGS dataset to complete a Track 2 submission.

What Track 2 uses from the dataset:
- **VCF** — to confirm the specific BUB1B variant(s) in this patient (compound heterozygous)
- **Clinical phenotype** — to ground the mechanistic reasoning in the patient's observed presentation
- **FASTQ** — not required for Track 2

The VCF and phenotype files are a small fraction of the total dataset size. If storage is limited, download selectively.

---

## How to download

### Prerequisites

```bash
pip install huggingface_hub
huggingface-cli login  # enter your HF_TOKEN
```

Or set the token directly:

```bash
export HF_TOKEN=your_token_here
```

### Download the full dataset (Track 1 / complete analysis)

```bash
huggingface-cli download SageBio/mva-hackathon-2026-data \
  --repo-type dataset \
  --local-dir ./data/mva-wgs
```

**Storage required:** 100–150 GB. Run this on a machine with sufficient disk.

### Download VCF + phenotype only (Track 2 minimum)

```bash
huggingface-cli download SageBio/mva-hackathon-2026-data \
  --repo-type dataset \
  --local-dir ./data/mva-wgs \
  --include "*.vcf*" "*.json" "*.tsv" "*.txt"
```

This downloads the variant calls and clinical data without the large FASTQ files.

### Using the Python API

```python
from huggingface_hub import snapshot_download

snapshot_download(
    repo_id="SageBio/mva-hackathon-2026-data",
    repo_type="dataset",
    local_dir="./data/mva-wgs",
    ignore_patterns=["*.fastq.gz", "*.fq.gz"],  # skip raw reads for Track 2
    token=os.environ["HF_TOKEN"]
)
```

---

## Using the VCF in the pipeline

The pipeline reads the patient's VCF to:
1. Confirm the exact BUB1B variants (which compound heterozygous alleles are present)
2. Assess whether the mutations are predicted to be loss-of-function (frameshift, nonsense, canonical splice site) or hypomorphic (missense, partial splice)
3. This determines whether the drug repurposing hypothesis should target "restore partial function" (hypomorphic) vs. "compensate for complete loss" (null)

```python
import cyvcf2

vcf = cyvcf2.VCF("./data/mva-wgs/patient.vcf.gz")
for variant in vcf:
    if variant.CHROM == "15" and 40700000 < variant.POS < 40900000:  # BUB1B locus
        print(variant.CHROM, variant.POS, variant.REF, variant.ALT)
```

---

## Storage requirements summary

| Use case | Data needed | Approx size |
|---|---|---|
| Track 2 only (drug repurposing) | VCF + phenotype | ~500 MB |
| Track 1 only (variant prediction) | VCF + FASTQ | ~85 GB |
| Both tracks | Full dataset | ~85 GB |

Always have at least **25–30% headroom** beyond the dataset size for intermediate files, ChromaDB index, pipeline checkpoints, and output.
