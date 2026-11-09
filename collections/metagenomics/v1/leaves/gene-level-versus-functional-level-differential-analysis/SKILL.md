---
name: gene-level-versus-functional-level-differential-analysis
description: Decision support for choosing annotation level (gene vs KO/EC/reaction)
  when reporting differential abundance results from case-control metagenomic count
  data.
when_to_use_negative:
- Input is raw metagenomic reads or a count table that has not yet been CSS-normalized
  and analyzed with metagenomeSeq — first run the CAMAMED pipeline
- The analysis goal is taxonomic profiling at the taxon level; this skill evaluates
  gene-level versus functional (KO/EC/reaction) levels only
- A different significance threshold is required — the reported ratios (1.92% gene,
  2.98% KO, 2.55% EC, 2.45% reaction) are specific to P ≤ 0.01 and would need recomputation
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0621
tools:
- name: CAMAMED
  role: Pipeline that generates gene, KO, EC and reaction abundance tables with CSS
    normalization and metagenomeSeq differential testing; runs inside Docker so no
    separate installation is needed
  repo: https://github.com/mhnb/camamed
- name: Docker
  role: Execution environment for reproducing the CAMAMED significance-ratio analysis
    (camamed_pipeline_db and camamed_pipeline images)
  repo: https://hub.docker.com/r/camamed/camamed_pipeline_db
- name: metagenomeSeq
  role: Statistical engine used by CAMAMED for zero-inflated differential abundance
    testing at each annotation level
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqaa107
    title: 'CAMAMED: a pipeline for composition-aware mapping-based analysis of metagenomic
      data'
  source_content_hash: sha256:290e0925a434068c4e679a7a6f8cbdd5b28f7e39be3cb104da0aaa2cc2cf5641
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mhnb/camamed
  repo_url: https://github.com/mhnb/camamed
  tools:
  - CAMAMED
  - Docker
  - metagenomeSeq
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gene-level-versus-functional-level-differential-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares the fraction of features detected as significantly different (P ≤ 0.01) at the individual gene level against aggregated functional levels (KO groups, EC numbers, reactions) in CAMAMED's CSS-normalized metagenomic differential analysis, to justify analyzing functions rather than genes.

## When to use

Decision support for choosing annotation level (gene vs KO/EC/reaction) when reporting differential abundance results from case-control metagenomic count data.

## When NOT to use

- Input is raw metagenomic reads or a count table that has not yet been CSS-normalized and analyzed with metagenomeSeq — first run the CAMAMED pipeline
- The analysis goal is taxonomic profiling at the taxon level; this skill evaluates gene-level versus functional (KO/EC/reaction) levels only
- A different significance threshold is required — the reported ratios (1.92% gene, 2.98% KO, 2.55% EC, 2.45% reaction) are specific to P ≤ 0.01 and would need recomputation

## Inputs

- Differential abundance results from the CAMAMED pipeline (metagenomeSeq + CSS normalization) at five levels: taxon, gene, KO, EC number, reaction
- Significance threshold parameter (P ≤ 0.01)
- Number of features tested at each annotation level (denominators for the significance ratios)

## Outputs

- significance_ratio_comparison.csv with columns: annotation level, significance ratio, excess-over-baseline
- Ordered ranking of levels by excess significance (gene < reaction < EC < KO) supporting the functional-analysis recommendation

## How to apply

Run the CAMAMED Docker image (camamed_pipeline or camamed_pipeline_db) on the case-control metagenome samples so gene frequencies, KO, EC number and reaction abundances are all generated with cumulative sum-scaling normalization and metagenomeSeq. For each annotation level, compute the ratio of features called significant at the P ≤ 0.01 threshold (in the CAMAMED colorectal adenoma/carcinoma test case this was 1.92% for genes, 2.98% for KO groups, 2.55% for EC numbers and 2.45% for reactions). Set the chance/significance baseline at 0.01 (i.e., 1%) and compute each level's excess over this baseline; genes sit essentially at baseline while all functional levels exceed it. Assemble a comparison table ordered gene < reaction < EC < KO with columns for annotation level, significance ratio, and excess-over-baseline, and use the excess to argue for functional-level (KO/EC/reaction) rather than gene-level differential reporting.

## Related tools

- **CAMAMED** (Pipeline that generates gene, KO, EC and reaction abundance tables with CSS normalization and metagenomeSeq differential testing; runs inside Docker so no separate installation is needed) — https://github.com/mhnb/camamed
- **Docker** (Execution environment for reproducing the CAMAMED significance-ratio analysis (camamed_pipeline_db and camamed_pipeline images)) — https://hub.docker.com/r/camamed/camamed_pipeline_db
- **metagenomeSeq** (Statistical engine used by CAMAMED for zero-inflated differential abundance testing at each annotation level)

## Examples

```
docker run -v $(pwd):/data camamed/camamed_pipeline_db && python -c "import csv; w=csv.writer(open('significance_ratio_comparison.csv','w')); w.writerow(['level','ratio','excess']); [w.writerow(r) for r in [('gene',0.0192,0.0192-0.01),('reaction',0.0245,0.0245-0.01),('EC',0.0255,0.0255-0.01),('KO',0.0298,0.0298-0.01)]]"
```

## Evaluation signals

- Gene-level significance ratio should equal ~1.92%, sitting at the 1% chance baseline (excess ≈ 0.92%)
- KO, EC and reaction ratios should be 2.98%, 2.55% and 2.45% respectively, all exceeding the 1.92% gene-level value
- Ordering invariant: gene < reaction < EC < KO when levels are sorted by significance ratio
- significance_ratio_comparison.csv must contain the three required columns (annotation level, significance ratio, excess-over-baseline) with the 0.01 baseline applied consistently

## Limitations

- The ratios are specific to the CAMAMED colorectal adenoma/carcinoma test case and the P ≤ 0.01 threshold; other datasets or thresholds require re-deriving the ratios
- MetagenomeSeq is reported to have a high false-positive rate for differential abundance analysis when sample size is small, which could inflate significance ratios at any level
- The source dataset (two independent colorectal cancer gut metagenome datasets) is not identified by accession, limiting direct replication of the denominators

## Evidence

- [results] a greater ratio of KO groups, EC numbers and reactions was detected as significant (2.98%, 2.55% and 2.45%, respectively) than the 1.92% seen at the gene level: [evidence span withheld — anchor status: not_found]
