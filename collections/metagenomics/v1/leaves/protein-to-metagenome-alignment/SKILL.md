---
name: protein-to-metagenome-alignment
description: Use when you need to prioritize gene cluster representatives by environmental
  relevance — e.g., selecting which amoA-AOB clusters deserve primer design — from
  alignments of proteins against many metagenomes.
when_to_use_negative:
- Input is a nucleotide amplicon/OTU table or already-computed abundance matrix —
  no protein-to-metagenome alignment is needed
- You lack a reference metagenome set for the target environment; environment-specific
  ranking requires user-defined references such as the 1,550 soil metagenomes
- The goal is de novo gene discovery rather than prioritizing already-clustered reference
  genes for primer design
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: DIAMOND
  role: Aligns representative protein sequences to the 1,550 soil metagenomes (v0.9.14)
    with 97% identity over reference length as high-quality hit criterion
- name: CD-HIT
  role: Upstream clustering (96% amino acid similarity) that produces the 60 representative
    sequences fed into this ranking step
- name: MetaFunPrimer
  role: 'Pipeline embedding this ranking: computes normalized abundance/prevalence,
    R-score, and cumulative R-score (80% threshold) to prioritize genes for primer
    design'
  repo: github.com/jialiu232/MetaFunPrimer
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:d0a8a46e1e5272fab1091cfa825d6034855a848c57326e64545072e98484f253
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT
  - DIAMOND
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# protein-to-metagenome-alignment

## Summary

Ranks representative protein sequences of gene clusters (e.g., amoA-AOB) by aligning each to a set of reference metagenomes (1,550 soil metagenomes) with DIAMOND and computing an R-score that averages normalized abundance and normalized prevalence per representative.

## When to use

Use when you need to prioritize gene cluster representatives by environmental relevance — e.g., selecting which amoA-AOB clusters deserve primer design — from alignments of proteins against many metagenomes.

## When NOT to use

- Input is a nucleotide amplicon/OTU table or already-computed abundance matrix — no protein-to-metagenome alignment is needed
- You lack a reference metagenome set for the target environment; environment-specific ranking requires user-defined references such as the 1,550 soil metagenomes
- The goal is de novo gene discovery rather than prioritizing already-clustered reference genes for primer design

## Inputs

- Representative protein sequences from clustered gene targets (e.g., 60 amoA-AOB cluster representatives at 96% similarity, FASTA)
- Reference metagenome set (1,550 publicly available soil metagenomes downloaded from MG-RAST and the DOE JGI Genome Portal)
- Cumulative R-score inclusion threshold (80% in the case study)

## Outputs

- Ranked table of representative genes with R-score, normalized abundance, prevalence, cumulative R-score, and gene counts (e.g., r_score_ranking.csv)
- Ranked gene targets passed downstream to primer design (top clusters capturing e.g. 720 amoA-AOB genes, 87% cumulative abundance)

## How to apply

Load the representative protein sequences produced by upstream clustering (here, 60 representatives from 96% amino acid similarity CD-HIT clustering of amoA-AOB sequences from Fungene 9.6). Align each representative to the 1,550 publicly available soil metagenomes with DIAMOND (version 0.9.14), treating hits with 97% sequence identity over the full length of the reference gene as high-quality alignments. For each representative, compute normalized abundance and normalized prevalence across the metagenomes and average the two to obtain the R-score (representation score). Rank representatives in descending R-score order, track the cumulative R-score, and include clusters until the user-input cumulative threshold (80% in the case study) is reached. In the MetaFunPrimer case study, the top-ranked clusters captured 720 amoA-AOB genes, 87% of cumulative abundance, and the pipeline proceeded to design 78 primer pairs for these targets.

## Related tools

- **DIAMOND** (Aligns representative protein sequences to the 1,550 soil metagenomes (v0.9.14) with 97% identity over reference length as high-quality hit criterion)
- **CD-HIT** (Upstream clustering (96% amino acid similarity) that produces the 60 representative sequences fed into this ranking step)
- **MetaFunPrimer** (Pipeline embedding this ranking: computes normalized abundance/prevalence, R-score, and cumulative R-score (80% threshold) to prioritize genes for primer design) — github.com/jialiu232/MetaFunPrimer

## Examples

```
diamond blastp -d 1550_soil_metagenomes.dmnd -q amoA_representatives.faa -o hits.tsv --id 97 --query-cover 100 --evalue 1e-5
```

## Evaluation signals

- High-quality alignments are defined as hits with 97% sequence identity spanning the full length of the reference gene
- Ranked table must include R-score = mean of separately normalized abundance and prevalence for every representative
- Top-10 ranked clusters should account for 720 amoA-AOB genes and 87% of cumulative abundance in the 1,550 soil metagenomes (paper validation result)
- Gene cluster inclusion must stop when the cumulative R-score reaches the user-input threshold (80% in the case study)
- Number of representatives should match the upstream clustering output (60 clusters at 96% similarity)

## Limitations

- Hits below 97% identity over the reference gene length are discarded as low-quality, potentially missing divergent homologs
- Ranking depends on the user-supplied reference metagenomes; a different environment set would change R-scores and gene prioritization
- Highly diverse functional genes may yield numerous ranked targets requiring many primer pairs, possibly exceeding HT-qPCR platform capacity

## Evidence

- [results] The representative protein sequences from each cluster were aligned to 1,550 publicly available soil metagenomes: "The representative protein sequences from each cluster were aligned to 1,550 publicly available soil metagenomes"
- [results] High-alignment definition: "with high alignment defined as having 97% sequence identity over the length of the reference gene"
- [results] R-score computation: [evidence span withheld — anchor status: not_found]
- [methods] Cumulative R-score threshold: "gene clusters are included until the user-input threshold of the cumulative R-score (i.e., 80% in the case study) is reached"
- [results] Metagenome sources: "Metagenomic data were downloaded from the Metagenomics RAST server (https://www.mg-rast.org/)"
