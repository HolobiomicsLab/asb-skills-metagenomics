---
name: amoa-aob-gene-cluster-analysis
description: Use when a curated set of amoA-AOB (or similar functional gene) reference
  protein+nucleotide sequences must be clustered into representative clusters and
  prioritized by abundance/prevalence across environmental metagenomes before primer
  design.
when_to_use_negative:
- Input is uncurated raw metagenomic reads rather than a curated functional gene reference
  set (Fungene-style HMM filtering has not been applied)
- The goal is primer design or in silico PCR coverage evaluation itself — those belong
  to the downstream MetaFunPrimer/EcoFunPrimer skills that consume this skill's ranked
  gene output
- Reference metagenomes are not environment-relevant, since gene target prioritization
  depends on ranking gene presence in user-defined environment-specific metagenomes
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: CD-HIT
  role: Clusters the curated amoA-AOB protein sequences at 96% similarity (threshold
    optimized via first-order difference) to select representative sequences
  repo: github.com/jialiu232/MetaFunPrimer_paper_info
- name: DIAMOND
  role: Aligns cluster representative protein sequences against 1,550 soil metagenomes
    to compute abundance and prevalence for R-score ranking
- name: MetaFunPrimer
  role: Pipeline embedding the clustering and gene-ranking workflow that prioritizes
    environment-abundant amoA-AOB targets
  repo: github.com/jialiu232/MetaFunPrimer_paper_info
- name: EcoFunPrimer
  role: Consumes the ranked target gene nucleotide sequences for downstream thermodynamically
    stable primer-pair design
  repo: https://github.com/rdpstaff/EcoFunPrimer
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_001
  - task_003
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:934e78fe32b3e30cb5d61ddc15adfc27b8915490369ec6b4409344a492bc4ee1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - DIAMOND
  - EcoFunPrimer
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# amoA-AOB-gene-cluster-analysis

## Summary

Clusters curated FunGene amoA-AOB reference protein sequences into representative gene clusters with CD-HIT and ranks those clusters by an R-score (mean of normalized abundance and prevalence) computed from DIAMOND alignments against 1,550 publicly available soil metagenomes, to identify environment-abundant amoA-AOB targets.

## When to use

Use when a curated set of amoA-AOB (or similar functional gene) reference protein+nucleotide sequences must be clustered into representative clusters and prioritized by abundance/prevalence across environmental metagenomes before primer design.

## When NOT to use

- Input is uncurated raw metagenomic reads rather than a curated functional gene reference set (Fungene-style HMM filtering has not been applied)
- The goal is primer design or in silico PCR coverage evaluation itself — those belong to the downstream MetaFunPrimer/EcoFunPrimer skills that consume this skill's ranked gene output
- Reference metagenomes are not environment-relevant, since gene target prioritization depends on ranking gene presence in user-defined environment-specific metagenomes

## Inputs

- Curated amoA-AOB reference protein sequences (FASTA) from Fungene v9.6 (1,205 sequences)
- Corresponding nucleotide sequences with nucleotide-to-protein ID mapping
- 1,550 publicly available soil metagenomes (Table S1; downloaded via MG-RAST and DOE JGI Genome Portal)
- Cumulative R-score inclusion threshold (e.g., 80%)

## Outputs

- Cluster membership table: one row per input sequence with CD-HIT cluster/representative assignment (60 clusters at 96% identity)
- Ranked table (r_score_ranking.csv) with per-cluster R-score, normalized abundance, prevalence, cumulative R-score, and gene counts
- Set of soil-abundant target genes (720 amoA-AOB genes from top-ranked clusters) for downstream primer design

## How to apply

Start from a curated amoA-AOB set obtained from the Ribosomal Database Project Fungene (version 9.6), filtering for an HMM search score >400 and HMM coverage over 70.2% amino acid similarity. Run CD-HIT on the protein sequences at 96% amino acid similarity — the threshold identified as giving the largest increase in total unique clusters via the first-order difference (symmetric derivative) criterion — yielding 60 representative clusters for the 1,205-sequence amoA-AOB input. Align each of the 60 representative protein sequences to the 1,550 publicly available soil metagenomes using DIAMOND (version 0.9.14), counting as high-quality alignments only hits with 97% sequence identity over the length of the reference gene. Compute normalized abundance and normalized prevalence per representative, average the two into an R-score, rank clusters in descending R-score order, and include clusters until the user-input cumulative R-score threshold (80% in the case study) is reached. The case study found the top-10 ranked clusters capture 720 amoA-AOB genes, i.e. 87% of cumulative abundance, which are the downstream targets for EcoFunPrimer/MetaFunPrimer primer design.

## Related tools

- **CD-HIT** (Clusters the curated amoA-AOB protein sequences at 96% similarity (threshold optimized via first-order difference) to select representative sequences) — github.com/jialiu232/MetaFunPrimer_paper_info
- **DIAMOND** (Aligns cluster representative protein sequences against 1,550 soil metagenomes to compute abundance and prevalence for R-score ranking)
- **MetaFunPrimer** (Pipeline embedding the clustering and gene-ranking workflow that prioritizes environment-abundant amoA-AOB targets) — github.com/jialiu232/MetaFunPrimer_paper_info
- **EcoFunPrimer** (Consumes the ranked target gene nucleotide sequences for downstream thermodynamically stable primer-pair design) — https://github.com/rdpstaff/EcoFunPrimer

## Examples

```
cd-hit -i amoa_aob_proteins.fasta -o amoa_aob_reps.fasta -c 0.96
```

## Evaluation signals

- CD-HIT clustering at 96% similarity of the 1,205 curated amoA-AOB proteins yields exactly 60 distinct representative clusters
- Every input sequence is assigned to exactly one cluster and the membership CSV has one row per input sequence
- R-score equals the arithmetic mean of separately normalized abundance and prevalence for each representative
- The ranked table reproduces the reported top-10 result: 720 amoA-AOB genes and 87% cumulative abundance
- Gene clusters are includable until the 80% cumulative R-score threshold is reached

## Limitations

- Alignment-based ranking depends on reference genes having detectable homologs in metagenomes; hits are only counted at 97% identity over the full reference gene length
- Highly diverse functional genes may cluster into many representatives, requiring numerous primer pairs downstream and possibly exceeding HT-qPCR platform capacity
- The 96% similarity threshold and 80% cumulative R-score cutoff were user-input choices validated for the amoA-AOB case study and may need re-optimization (via first-order difference) for other genes

## Evidence

- [methods] Clustering threshold optimization: "CD-HIT (38, 39) is used to cluster sequences in the range of 80% to 100% (with 1% increments) similarity"
- [methods] Threshold selection criterion: "a similarity threshold that optimizes the first-order difference, a criterion based on the symmetric derivative"
- [results] Alignment stringency for ranking: "with high alignment defined as having 97% sequence identity over the length of the reference gene"
- [results] R-score definition: [evidence span withheld — anchor status: not_found]
- [methods] Cumulative inclusion cutoff: "gene clusters are included until the user-input threshold of the cumulative R-score (i.e., 80% in the case study) is reached"
- [methods] Input curation filters: "filtering for a Hidden Markov Model (HMM) search score of >400 and HMM coverage over 70.2% amino acid similarity"
- [results] Case-study cluster count: "The representatives of the 60 gene clusters found after clustering the input amoA-AOB sequences at 96% amino acid sequence similarity were aligned using MAFFT"
