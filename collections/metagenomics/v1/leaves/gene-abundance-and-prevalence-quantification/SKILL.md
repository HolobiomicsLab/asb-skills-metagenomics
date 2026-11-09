---
name: gene-abundance-and-prevalence-quantification
description: Use when a set of clustered gene representatives must be prioritized
  by how abundant and how widespread they are across a defined collection of metagenomes,
  e.g. to select representative amoA-AOB clusters from soil metagenomes for primer
  design.
when_to_use_negative:
- The goal is primer thermodynamics or degeneracy design rather than gene prioritization
  (use EcoFunPrimer/Primer3 instead)
- Input sequences have not yet been clustered into representatives — raw gene sets
  must first be clustered (e.g., CD-HIT at 96%) to avoid redundant ranking
- Reference metagenomes are from a different environment than the target system, since
  R-scores are environment-specific and would not reflect the target habitat's gene
  abundance
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2269
tools:
- name: DIAMOND
  role: Aligns each cluster representative protein sequence to the 1,550 soil metagenomes
    (v0.9.14) to estimate abundance and prevalence via 97% identity full-length hits
- name: CD-HIT
  role: Upstream clustering of amoA-AOB sequences at 96% amino acid similarity to
    produce the 60 representative sequences that are ranked
- name: MetaFunPrimer
  role: Pipeline embedding the R-score ranking of genes of interest against user-defined
    reference metagenomes before primer design
  repo: github.com/jialiu232/MetaFunPrimer_paper_info
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:9b96f2d61b5468d84046175e5009b5949cfc9003c5e2f132ef74366cfcf98156
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - DIAMOND
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Gene abundance and prevalence quantification

## Summary

Ranks representative gene clusters by an R-score — the mean of normalized abundance and normalized prevalence — computed by aligning each representative to a panel of reference metagenomes, here 60 amoA-AOB cluster representatives from 96% CD-HIT clustering searched against 1,550 publicly available soil metagenomes. This prioritizes gene clusters for environment-specific primer design.

## When to use

Use when a set of clustered gene representatives must be prioritized by how abundant and how widespread they are across a defined collection of metagenomes, e.g. to select representative amoA-AOB clusters from soil metagenomes for primer design.

## When NOT to use

- The goal is primer thermodynamics or degeneracy design rather than gene prioritization (use EcoFunPrimer/Primer3 instead)
- Input sequences have not yet been clustered into representatives — raw gene sets must first be clustered (e.g., CD-HIT at 96%) to avoid redundant ranking
- Reference metagenomes are from a different environment than the target system, since R-scores are environment-specific and would not reflect the target habitat's gene abundance

## Inputs

- Representative protein sequences from gene clusters (60 amoA-AOB representatives from 96% CD-HIT clustering, FASTA)
- 1,550 publicly available soil metagenomes (downloaded from MG-RAST and the DOE JGI Genome Portal)
- Cumulative R-score inclusion threshold (80% in the case study)

## Outputs

- Ranked table r_score_ranking.csv with gene cluster, R-score, normalized abundance, prevalence, cumulative R-score, and gene counts per cluster
- Prioritized subset of representative gene clusters meeting the cumulative R-score threshold (top-10 clusters capture 720 genes, 87% of cumulative abundance)

## How to apply

Start from the representative protein sequences produced by upstream CD-HIT clustering of the gene of interest (60 amoA-AOB representatives clustered at 96% amino acid similarity in the case study). Align each representative against all 1,550 soil metagenomes with DIAMOND (version 0.9.14), counting only high-quality hits defined as 97% sequence identity over the length of the reference gene. For each representative, compute normalized abundance and normalized prevalence across metagenomes and average the two to obtain the R-score. Rank representatives in descending R-score order, accumulate R-scores down the ranked list, and include gene clusters until the user-input cumulative R-score threshold is reached (80% in the case study). Write the full ranked table (gene cluster, R-score, normalized abundance, prevalence, cumulative R-score, gene counts) to r_score_ranking.csv.

## Related tools

- **DIAMOND** (Aligns each cluster representative protein sequence to the 1,550 soil metagenomes (v0.9.14) to estimate abundance and prevalence via 97% identity full-length hits)
- **CD-HIT** (Upstream clustering of amoA-AOB sequences at 96% amino acid similarity to produce the 60 representative sequences that are ranked)
- **MetaFunPrimer** (Pipeline embedding the R-score ranking of genes of interest against user-defined reference metagenomes before primer design) — github.com/jialiu232/MetaFunPrimer_paper_info

## Examples

```
diamond blastp -d soil_metagenomes.dmnd -q amoA_representatives_60.faa --id 97 -o r_score_hits.tsv --threads 16
```

## Evaluation signals

- The ranked table reproduces the reported top-10 result: top-10 clusters represent 720 amoA-AOB genes comprising 87% of cumulative abundance in the 1,550 soil metagenomes
- Clusters can be included until the 80% cumulative R-score threshold is reached without any single cluster exceeding it
- Each representative has exactly one R-score equal to the mean of its separately normalized abundance and prevalence
- All counted hits meet the 97% sequence identity over the full reference gene length criterion
- r_score_ranking.csv contains the columns gene cluster, R-score, normalized abundance, prevalence, cumulative R-score, and gene counts

## Limitations

- The ranking is environment-specific: abundance and prevalence reflect the user-defined reference metagenomes (here 1,550 soil metagenomes) and will not generalize to other environments
- Gene clusters with very low relative abundance may fall below the cumulative R-score cutoff and be excluded from primer design
- Highly diverse functional genes may require numerous primer pairs beyond qPCR platform capacity

## Evidence

- [results] High-quality DIAMOND alignment criterion: "with high alignment defined as having 97% sequence identity over the length of the reference gene"
- [results] R-score definition as mean of normalized abundance and prevalence: [evidence span withheld — anchor status: not_found]
- [methods] Cumulative R-score inclusion threshold: "gene clusters are included until the user-input threshold of the cumulative R-score (i.e., 80% in the case study) is reached"
- [methods] Fungene source of amoA-AOB reference set: "filtering for a Hidden Markov Model (HMM) search score of >400 and HMM coverage over 70.2% amino acid similarity"
