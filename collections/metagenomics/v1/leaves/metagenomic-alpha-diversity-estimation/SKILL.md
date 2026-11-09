---
name: metagenomic-alpha-diversity-estimation
description: Use when QCed metagenomic profiling tables (phylogenetic clades L2–L7
  or gene abundance) from case–control shotgun data are available and the goal is
  to test whether within-sample diversity differs between cases and controls.
when_to_use_negative:
- Input is raw FASTQ reads that have not passed OMARU read QC and profiling (Steps
  1–3); diversity estimation requires QCed2 abundance tables
- The goal is between-sample (beta-diversity, Bray-Curtis/PERMANOVA) or clade/genic
  association testing rather than within-sample alpha-diversity
- Sample sequencing depths differ and depth normalization cannot be applied, since
  depth-driven bias would confound Shannon diversity comparisons
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_2269
tools:
- name: OMARU
  role: End-to-end workflow providing the diversity assessment step downstream of
    read QC and phylogenetic/functional profiling, including depth normalization and
    Shannon diversity computation
  repo: https://github.com/toshi-kishikawa/OMARU
- name: Snakemake
  role: Executes the OMARU diversity Snakefile (OMARU_QCed2.sm) that produces the
    abundance tables used for diversity estimation
  repo: https://github.com/toshi-kishikawa/OMARU
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqac019
    title: 'OMARU: a robust and multifaceted pipeline for metagenome-wide association
      study'
  source_content_hash: sha256:1833b3472b6f403fa9a294189988b36ce9766f61328a871465e6a1b59cbc1e1b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/toshi-kishikawa/OMARU
  repo_url: https://github.com/toshi-kishikawa/OMARU
  tools:
  - OMARU
  - Snakemake
  - vegan
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metagenomic-alpha-diversity-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates within-sample metagenomic diversity (Shannon alpha-diversity on clade relative abundances at levels L2–L7 and on gene abundance) and tests case–control differences, as implemented downstream of OMARU profiling to detect disrupted diversity such as the significant alpha-diversity increase in T2D.

## When to use

Use when QCed metagenomic profiling tables (phylogenetic clades L2–L7 or gene abundance) from case–control shotgun data are available and the goal is to test whether within-sample diversity differs between cases and controls.

## When NOT to use

- Input is raw FASTQ reads that have not passed OMARU read QC and profiling (Steps 1–3); diversity estimation requires QCed2 abundance tables
- The goal is between-sample (beta-diversity, Bray-Curtis/PERMANOVA) or clade/genic association testing rather than within-sample alpha-diversity
- Sample sequencing depths differ and depth normalization cannot be applied, since depth-driven bias would confound Shannon diversity comparisons

## Inputs

- QCed2-selected sample list (QCed2_sample_list.txt) from OMARU profiling
- Phylogenetic abundance tables at taxonomic levels L2–L7 (e.g. UHGG/CHOCO-based clades)
- Gene abundance tables (e.g. UniRef90/KEGG-based)
- Case–control metadata (original_sample_list.txt with Phenotype, Sex, Age columns)

## Outputs

- Per-sample Shannon alpha-diversity values per taxonomic level (L2–L7) and per gene abundance table
- Case–control diversity test statistics compiled into diversity_statistics.tsv
- Per-level and per-gene P-values for T2D (n=171) and SCZ (n=344) comparisons

## How to apply

First run the OMARU diversity workflow downstream of profiling on the QCed2-selected samples (cd OMARU_project_dir; snakemake -s Snakefiles/OMARU_QCed2.sm) so phylogenetic and gene abundance tables are available. Normalize sequencing-depth bias by down-sampling reads from each sample to a common depth with the OMARU normalization step before diversity estimation. Compute Shannon alpha-diversity per sample on relative abundance of clades at taxonomic levels L2–L7 and on gene abundance, and assess case–control differences against a multiple-testing-corrected threshold (P < 8.3e-3). Compare per-level and per-gene P-values against the published reproduction targets: T2D alpha-diversity significant at L3/L4 (P < 8.3e-3) and gene level (P = 5.1e-3), while SCZ showed no significant diversity differences (gene-level alpha P = 0.134).

## Related tools

- **OMARU** (End-to-end workflow providing the diversity assessment step downstream of read QC and phylogenetic/functional profiling, including depth normalization and Shannon diversity computation) — https://github.com/toshi-kishikawa/OMARU
- **Snakemake** (Executes the OMARU diversity Snakefile (OMARU_QCed2.sm) that produces the abundance tables used for diversity estimation) — https://github.com/toshi-kishikawa/OMARU

## Examples

```
cd OMARU_project_dir && snakemake -s Snakefiles/OMARU_QCed2.sm
```

## Evaluation signals

- T2D alpha-diversity is significantly increased at taxonomic levels L3 and L4 with P < 8.3e-3 after multiple-testing correction
- T2D gene-abundance alpha-diversity reaches P = 5.1e-3
- SCZ shows no significant diversity differences (gene-level alpha P = 0.134), matching the reported null result
- Diversity values are computed after down-sampling reads to a common depth, so depth-correlated diversity trends should be absent
- Compiled statistics written to diversity_statistics.tsv cover all levels L2–L7 and the gene table for both T2D (n=171) and SCZ (n=344)

## Limitations

- Hidden confounding factors would better be adjusted by integration of covariates into the case–control model, but this is not currently implemented in OMARU
- Results depend on the reference database (e.g. UHGG for phylogenetic, UniRef90 for gene); metagenome analysis is highly dependent on reference databases
- Diversity conclusions are dataset-specific: T2D showed significant diversity differences while SCZ showed little difference, so lack of significance in a new cohort is not necessarily a pipeline failure

## Evidence
