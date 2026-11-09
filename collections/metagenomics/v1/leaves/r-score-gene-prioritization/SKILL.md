---
name: r-score-gene-prioritization
description: Use when a set of gene cluster representatives must be prioritized by
  how abundant and widespread they are across user-defined reference metagenomes,
  e.g. to select which gene clusters to target with primers.
when_to_use_negative:
- When the goal is de novo gene discovery rather than prioritizing already-clustered
  reference genes for primer targeting
- When no user-defined reference metagenomes/genomes are available, since the R-score
  depends entirely on abundance and prevalence in those references
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0622
tools:
- name: DIAMOND
  role: Aligns cluster representative protein sequences against reference soil metagenomes
    (97% identity over reference length) to estimate abundance and prevalence
- name: CD-HIT
  role: Produces the gene clusters and representative sequences (96% amino acid similarity)
    that are the input to R-score ranking
- name: MetaFunPrimer
  role: Pipeline embedding the R-score-based gene target prioritization step
  repo: https://github.com/jialiu232/MetaFunPrimer_paper_info
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:fb7555c21db12ecfa2b8388d625924965859f12907eb74eea74a7efb069eb230
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jialiu232/MetaFunPrimer_paper_info
  tools:
  - CD-HIT
  - DIAMOND
  - MetaFunPrimer
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# r-score-gene-prioritization

## Summary

Ranks gene cluster representatives by combining normalized abundance and prevalence across reference metagenomes into a single R-score, enabling prioritization of environmentally abundant gene targets for primer design (as done for amoA-AOB clusters against 1,550 soil metagenomes in MetaFunPrimer).

## When to use

Use when a set of gene cluster representatives must be prioritized by how abundant and widespread they are across user-defined reference metagenomes, e.g. to select which gene clusters to target with primers.

## When NOT to use

- When the goal is de novo gene discovery rather than prioritizing already-clustered reference genes for primer targeting
- When no user-defined reference metagenomes/genomes are available, since the R-score depends entirely on abundance and prevalence in those references

## Inputs

- Representative protein sequences from gene clusters (e.g. 60 amoA-AOB representatives at 96% amino acid similarity)
- Reference metagenomes (1,550 publicly available soil metagenomes from MG-RAST and DOE JGI Genome Portal)

## Outputs

- Ranked table of gene cluster representatives with R-score, normalized abundance, prevalence, cumulative R-score, and gene counts (r_score_ranking.csv)
- Minimal set of top-ranked gene clusters meeting the cumulative R-score threshold (80%)

## How to apply

Load the representative protein sequences produced by upstream clustering (e.g. 60 amoA-AOB representatives after 96% CD-HIT clustering). Align each representative to the reference metagenomes (1,550 publicly available soil metagenomes) with DIAMOND (version 0.9.14), defining a high-quality alignment as 97% sequence identity over the length of the reference gene. For each representative, compute normalized abundance and normalized prevalence across the metagenomes and average the two to obtain the R-score. Rank representatives in descending R-score order and include gene clusters until a user-input cumulative R-score threshold (80% in the case study) is reached; validate that the top-ranked clusters reproduce reported results (e.g. top-10 clusters capturing 720 amoA-AOB genes and 87% cumulative abundance).

## Related tools

- **DIAMOND** (Aligns cluster representative protein sequences against reference soil metagenomes (97% identity over reference length) to estimate abundance and prevalence)
- **CD-HIT** (Produces the gene clusters and representative sequences (96% amino acid similarity) that are the input to R-score ranking)
- **MetaFunPrimer** (Pipeline embedding the R-score-based gene target prioritization step) — https://github.com/jialiu232/MetaFunPrimer_paper_info

## Examples

```
diamond blastp -d 1550_soil_metagenomes.dmnd -q amoA_AOB_reps.faa --id 97 -o r_score_hits.tsv
```

## Evaluation signals

- Top-10 ranked clusters reproduce the reported result: 720 amoA-AOB genes and 87% cumulative abundance
- Gene clusters are included until the 80% cumulative R-score threshold is reached, then inclusion stops
- Each R-score equals the mean of separately normalized abundance and prevalence values
- High-quality alignments meet 97% sequence identity over the full length of the reference gene
- Ranked table contains all required columns (gene cluster, R-score, normalized abundance, prevalence, cumulative R-score, gene counts)

## Limitations

- Ranking quality depends on the chosen reference metagenome set; it reflects abundance in user-defined references, not global distributions
- Genes of very low relative abundance may be deprioritized, complicating detection without HT-qPCR or conventional qPCR approaches combined with MetaFunPrimer

## Evidence

- [results] Alignment and R-score computation: "The representative protein sequences from each cluster were aligned to 1,550 publicly available soil metagenomes"
- [results] High-quality alignment definition: "with high alignment defined as having 97% sequence identity over the length of the reference gene"
- [results] R-score formula: [evidence span withheld — anchor status: not_found]
- [methods] Cumulative R-score inclusion threshold: "gene clusters are included until the user-input threshold of the cumulative R-score (i.e., 80% in the case study) is reached"
- [results] Clustering upstream of ranking: "The representatives of the 60 gene clusters found after clustering the input amoA-AOB sequences at 96% amino acid sequence similarity were aligned using MAFFT"
