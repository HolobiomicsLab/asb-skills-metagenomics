---
name: cumulative-abundance-rank-thresholding
description: Use when many clustered gene representatives (e.g., 60 amoA-AOB clusters
  from 96% CD-HIT clustering) must be prioritized by environmental abundance/prevalence
  so only a minimal, high-coverage subset is carried forward to primer design.
when_to_use_negative:
- Input sequences are unclustered raw gene reads — cluster representatives must already
  exist (e.g., via 96% CD-HIT clustering) before R-score ranking is meaningful
- No user-defined reference metagenomes are available, since abundance and prevalence
  are estimated against environment-specific references such as the 1,550 soil metagenomes
- The goal is phylogenetic or functional annotation of genes rather than prioritizing
  targets for environment-specific primer design
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaFunPrimer
  role: Pipeline that ranks gene presence in user-defined references and includes
    gene clusters until the cumulative R-score threshold is reached
  repo: https://github.com/jialiu232/MetaFunPrimer_paper_info
- name: DIAMOND
  role: Aligns representative protein sequences to soil metagenomes; hits at 97% identity
    over the reference gene length count as high-quality alignments for abundance/prevalence
    estimation
- name: CD-HIT
  role: Upstream clustering of amoA-AOB sequences at 96% amino acid similarity to
    produce the representative sequences that are ranked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00201-21
    title: 'MetaFunPrimer: an Environment-Specific, High-Throughput Primer Design
      Tool for Improved Quantification of Target Genes'
  source_content_hash: sha256:5b343fdc49c25a12d4c7c771c06ac929abafe91efe60f830408ae4b7843cecaa
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cumulative-abundance-rank-thresholding

## Summary

Ranks gene cluster representatives by an R-score (mean of normalized abundance and normalized prevalence across reference metagenomes) and includes clusters until a cumulative R-score threshold (e.g., 80% in the MetaFunPrimer amoA-AOB case study) is reached, prioritizing environmentally abundant targets for primer design.

## When to use

Use when many clustered gene representatives (e.g., 60 amoA-AOB clusters from 96% CD-HIT clustering) must be prioritized by environmental abundance/prevalence so only a minimal, high-coverage subset is carried forward to primer design.

## When NOT to use

- Input sequences are unclustered raw gene reads — cluster representatives must already exist (e.g., via 96% CD-HIT clustering) before R-score ranking is meaningful
- No user-defined reference metagenomes are available, since abundance and prevalence are estimated against environment-specific references such as the 1,550 soil metagenomes
- The goal is phylogenetic or functional annotation of genes rather than prioritizing targets for environment-specific primer design

## Inputs

- Representative protein sequences from gene clusters (e.g., 60 representatives from 96% CD-HIT clustering of amoA-AOB sequences)
- Reference metagenome set (1,550 publicly available soil metagenomes downloaded from MG-RAST and the DOE JGI Genome Portal)
- User-input cumulative R-score threshold (e.g., 80%)

## Outputs

- Ranked table of gene cluster representatives with R-score, normalized abundance, normalized prevalence, cumulative R-score, and gene counts (r_score_ranking.csv)
- Minimal set of high-priority gene cluster representatives included up to the cumulative R-score threshold (top-10 clusters = 720 genes, 87% cumulative abundance in the case study)

## How to apply

Load the representative protein sequences produced by upstream clustering (in the case study, 60 representatives from 96% amino acid similarity CD-HIT clustering of amoA-AOB sequences). Align each representative to the reference metagenomes (1,550 publicly available soil metagenomes) with DIAMOND (version 0.9.14), counting only high-quality alignments defined as 97% sequence identity over the length of the reference gene. For each representative, compute normalized abundance and normalized prevalence across the metagenomes and average the two to obtain the R-score (R-score = mean of normalized abundance and prevalence). Rank representatives in descending R-score order and accumulate R-scores down the ranked list, including clusters until the user-input cumulative R-score threshold (80% in the case study) is reached. In the reported case, the top-10 ranked clusters captured 720 amoA-AOB genes, i.e., 87% of cumulative abundance of these genes in the soil metagenomes, validating the prioritization before primer design.

## Related tools

- **MetaFunPrimer** (Pipeline that ranks gene presence in user-defined references and includes gene clusters until the cumulative R-score threshold is reached) — https://github.com/jialiu232/MetaFunPrimer_paper_info
- **DIAMOND** (Aligns representative protein sequences to soil metagenomes; hits at 97% identity over the reference gene length count as high-quality alignments for abundance/prevalence estimation)
- **CD-HIT** (Upstream clustering of amoA-AOB sequences at 96% amino acid similarity to produce the representative sequences that are ranked)

## Examples

```
diamond blastp -d 1550_soil_metagenomes.dmnd -q amoA_AOB_60_reps.faa --id 97 --query-cover 100 -o r_score_hits.tsv; python metafunprimer_rank.py --hits r_score_hits.tsv --threshold 0.80 --out r_score_ranking.csv
```

## Evaluation signals

- Ranked table reproduces the reported top-10 result: top-10 clusters account for 720 amoA-AOB genes and 87% of cumulative abundance in the 1,550 soil metagenomes
- Every representative's R-score equals the mean of its separately normalized abundance and prevalence values
- Cluster inclusion stops exactly when the cumulative R-score crosses the user-input threshold (80% in the case study), with no clusters beyond the threshold included
- Only high-quality DIAMOND alignments (97% sequence identity over the full length of the reference gene) contribute to abundance and prevalence estimates
- Output CSV contains all required columns: gene cluster, R-score, normalized abundance, prevalence, cumulative R-score, gene counts

## Limitations

- Ranking depends on the choice of reference metagenomes (environment-specific); results will differ for other environments or metagenome sets
- High alignment stringency (97% identity over full reference length) may miss divergent homologs, underestimating abundance of distant cluster members
- Highly diverse functional genes may yield many included clusters, requiring numerous primer pairs and qPCR runs that can exceed HT-qPCR platform capacity

## Evidence

- [results] R-score definition from results: [evidence span withheld — anchor status: not_found]
- [results] Alignment quality criterion: "with high alignment defined as having 97% sequence identity over the length of the reference gene"
- [methods] Cumulative R-score threshold from methods: "gene clusters are included until the user-input threshold of the cumulative R-score (i.e., 80% in the case study) is reached"
- [discussion] Coverage finding from discussion: "94% coverage of observed genes in 1,550 soil metagenomes"
- [abstract] Gene target prioritization from abstract: "This tool also enables gene target prioritization based on ranking the presence of genes in user-defined references, such as environment-specific metagenomes"
