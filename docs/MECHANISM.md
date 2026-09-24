# BUB1B / MVA Mechanism Background

## The Spindle Assembly Checkpoint

Every time a human cell divides, it must distribute one copy of each of its 46 chromosomes to each daughter cell. The spindle assembly checkpoint (SAC) is the surveillance mechanism that ensures this happens correctly. It monitors the attachment of chromosomes to the mitotic spindle — the protein machinery that pulls chromosomes apart — and holds the cell at metaphase until every kinetochore (the protein complex on each chromosome that connects to spindle fibres) is correctly bi-oriented.

The checkpoint works by inhibiting the Anaphase Promoting Complex/Cyclosome (APC/C), an E3 ubiquitin ligase that triggers anaphase. When any kinetochore is unattached, the SAC generates a **mitotic checkpoint complex (MCC)** that sequesters CDC20 (an APC/C activator), preventing APC/C from degrading securin and cyclin B — the two proteins that must be destroyed before chromosomes can separate.

Once every kinetochore is correctly attached and under tension, the SAC signal is silenced, MCC is disassembled, CDC20 is released, APC/C is activated, securin and cyclin B are degraded, and anaphase proceeds.

## BubR1 (BUB1B)

**BubR1** (Budding Uninhibited by Benzimidazoles Related 1) is a large multi-domain kinase encoded by the **BUB1B** gene (chromosome 15q15.1). It is one of the core components of the MCC and a central effector of the spindle checkpoint.

BubR1's key functions:

1. **MCC component.** BubR1 forms the MCC together with MAD2, BUB3, and CDC20. Within the MCC, BubR1 directly binds and inhibits CDC20, preventing premature APC/C activation.
2. **Kinetochore localisation.** BubR1 localises to unattached kinetochores via BUB3, amplifying the checkpoint signal at the site of attachment failure.
3. **CENP-E interaction.** BubR1 interacts with the kinetochore motor CENP-E (kinesin-7) to monitor chromosome congression and ensure proper tension.
4. **Checkpoint maintenance.** BubR1 has pseudokinase activity (its kinase domain is catalytically inactive in most contexts) but binds APC/C co-activators and contributes to checkpoint strength through scaffolding.

**Key references:**
- Musacchio A, Salmon ED. "The spindle-assembly checkpoint in space and time." *Nat Rev Mol Cell Biol.* 2007;8(5):379–393. PMID: 17426725
- Taylor SS, Ha E, McKeon F. "The human homologue of Bub3 is required for kinetochore localization of Bub1 and a Mad3/Bub1-related protein kinase." *J Cell Biol.* 1998;142(1):1–11. PMID: 9660858
- Bolanos-Garcia VM, Blundell TL. "BUB1 and BUBR1: multifaceted kinases of the cell cycle." *Trends Biochem Sci.* 2011;36(3):141–150. PMID: 20888775

## BUB1B Mutations and MVA

**Mosaic Variegated Aneuploidy (MVA)** is caused by biallelic loss-of-function mutations in BUB1B in most confirmed cases. The condition was first linked to BUB1B by Hanks et al. (2004).

With reduced or absent BubR1:
- The MCC cannot form at full strength
- CDC20 is not adequately sequestered
- APC/C activates prematurely
- Chromosomes separate before all kinetochores are correctly attached
- The result is **aneuploidy** — cells with abnormal chromosome numbers
- Because the mutations are usually hypomorphic (not complete null) and occur in a mosaic pattern, some cells maintain near-normal chromosome counts while others are aneuploid — hence "mosaic variegated"

**Clinical phenotype of MVA:**
- Microcephaly, growth retardation, intellectual disability
- Increased cancer susceptibility (Wilms tumour, rhabdomyosarcoma, leukaemia)
- Variable expressivity — severity correlates with residual BubR1 activity

**Key references:**
- Hanks S et al. "Constitutional aneuploidy and cancer predisposition caused by biallelic mutations in BUB1B." *Nat Genet.* 2004;36(11):1159–1161. PMID: 15475953
- Matsuura S et al. "Monoallelic BUB1B mutations and defective mitotic-spindle checkpoint in seven families with premature chromatid separation (PCS) syndrome." *Am J Med Genet A.* 2006;140(4):358–367. PMID: 16419126
- Snape K et al. "Mutations in CEP57 cause mosaic variegated aneuploidy syndrome." *Nat Genet.* 2011;43(6):527–529. PMID: 21552266

## Pathway Nodes Relevant to Drug Repurposing

The spindle checkpoint pathway contains several proteins that are established drug targets in cancer. These are the nodes where approved drugs have mechanistic entry points:

| Protein | Role in SAC | Approved drugs | Repurposing rationale |
|---|---|---|---|
| **PLK1** (Polo-like kinase 1) | Phosphorylates CDC20, activates APC/C, promotes mitotic exit | Volasertib (Phase III AML) | PLK1 inhibition may slow premature APC/C activation caused by BubR1 loss |
| **Aurora A / AURKA** | Activates CDK1–cyclin B, promotes mitotic entry | Alisertib, Danusertib | Aurora A inhibition delays mitotic entry, giving time for residual checkpoint to act |
| **Aurora B / AURKB** | Error-correction kinase — destabilises incorrect kinetochore–microtubule attachments | Barasertib (AZD1152), Hesperadin | AURKB inhibition affects tension sensing; relevant to kinetochore attachment defects |
| **CDC20** | APC/C co-activator; sequestered by BubR1 in normal SAC | No approved inhibitors yet; apcin and tosyl-l-arginine methyl ester in research | Direct CDC20 inhibition would partially substitute for lost BubR1 inhibitory function |
| **CDK1–Cyclin B** | Master mitotic kinase; substrate of APC/C | RO-3306 (CDK1 inhibitor, research); indirect via CDK inhibitors | Delaying CDK1 activation or maintaining cyclin B artificially may reduce aneuploidy rate |
| **HDAC** (class I/II) | Chromatin remodelling; HDAC inhibitors affect BubR1 expression indirectly | Vorinostat, Panobinostat (approved) | Some HDAC inhibitors upregulate residual BUB1B expression in hypomorphic mutations |

**Note:** None of these represent a cure — MVA has no treatment. These are mechanistic hypotheses for reducing aneuploidy rate in dividing cells with residual BubR1 function. The unknowns are significant: therapeutic window in non-cancerous cells, effect on germline vs. somatic tissue, and interaction with the patient's specific compound heterozygous variant(s) are all genuinely unknown.

## Summary

BubR1's loss causes the spindle checkpoint to fail prematurely, allowing CDC20 to activate APC/C before chromosomes are correctly attached. Drug repurposing targets nodes that either (a) compensate for the lost BubR1 inhibitory function, (b) slow the upstream events that feed into premature checkpoint silencing, or (c) upregulate residual BubR1 expression from the hypomorphic allele.

The pipeline in this project uses this pathway map as its reasoning scaffold — not as a predetermined list of answers, but as the biological context against which retrieved literature and drug database entries are evaluated.
