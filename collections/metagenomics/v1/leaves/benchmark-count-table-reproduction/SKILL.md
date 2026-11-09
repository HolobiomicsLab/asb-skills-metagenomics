---
name: benchmark-count-table-reproduction
description: Use when the goal is to validate a gene predictor's real-read performance
  by reproducing published BLAST-hit count tables from 454 metagenomic reads, e.g.
  the TS28/TS50 human gut twin-study samples.
when_to_use_negative:
- Input data are simulated metagenomes rather than real error-prone 454 reads — the
  paper reports simulated-read results separately from the real-read Table 3 validation
- The aim is novel benchmarking rather than reproducing the published counts; Glimmer-MG
  is 'more computationally expensive than existing software', so avoid it when cost
  dominates
- Reads come from a platform without homopolymer-associated indels (e.g. Illumina),
  where the 454-specific error modeling is not the relevant validation axis
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Glimmer-MG
  role: Primary gene predictor under evaluation; classification (Phymm), clustering
    (Scimm), cluster retraining, and indel-error-aware prediction on the 454 reads
  repo: https://github.com/davek44/Glimmer-MG
- name: Glimmer
  role: Framework on which Glimmer-MG is based; provides IMM-based coding-region identification
  repo: https://github.com/davek44/Glimmer-MG
- name: FragGeneScan
  role: Comparison gene predictor whose BLAST-aligning prediction count serves as
    the baseline in Table 3
- name: 454 Life Sciences
  role: Sequencing platform that produced the real TS28/TS50 human gut reads with
    homopolymer-run indel errors
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkr1067
    title: Gene prediction with Glimmer for metagenomic sequences augmented by classification
      and clustering
  source_content_hash: sha256:765b0210e94b25c81297a5b199a6f4be0bcfe8ffceb224d022fb6833e691bea0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/davek44/Glimmer-MG
  repo_url: https://github.com/davek44/Glimmer-MG
  tools:
  - 454 Life Sciences
  - FragGeneScan
  - Glimmer
  - Glimmer-MG
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# benchmark-count-table-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce the published Table 3 benchmark counts by running three metagenomic gene predictors (Glimmer-MG, FragGeneScan, MetaGeneMark) on the real 454 human gut microbiome reads (TS28, TS50), BLASTing translated predictions against NCBI nr at E < 0.001, and matching counts/proportions to the paper's table.

## When to use

Use when the goal is to validate a gene predictor's real-read performance by reproducing published BLAST-hit count tables from 454 metagenomic reads, e.g. the TS28/TS50 human gut twin-study samples.

## When NOT to use

- Input data are simulated metagenomes rather than real error-prone 454 reads — the paper reports simulated-read results separately from the real-read Table 3 validation
- The aim is novel benchmarking rather than reproducing the published counts; Glimmer-MG is 'more computationally expensive than existing software', so avoid it when cost dominates
- Reads come from a platform without homopolymer-associated indels (e.g. Illumina), where the 454-specific error modeling is not the relevant validation axis

## Inputs

- Real 454 sequencing reads (FASTA/FASTQ) from human gut microbiome samples TS28 and TS50
- FragGeneScan and MetaGeneMark installations for comparison gene predictions
- NCBI non-redundant protein database (nr) for BLAST validation

## Outputs

- Per-predictor, per-sample count of predicted genes (Table 3 format)
- Per-predictor, per-sample counts and proportions of predictions with BLAST hits at E < 0.001
- Relative-difference comparison (Glimmer-MG vs. FragGeneScan, Glimmer-MG vs. MetaGeneMark)

## How to apply

Obtain the real 454 human gut microbiome reads for samples TS28 and TS50 from the obese/lean twin study. Run Glimmer-MG — which classifies reads with Phymm, clusters them with Scimm, retrains models per cluster, and models homopolymer-run indel errors — plus FragGeneScan and MetaGeneMark on the identical FASTA input so comparisons are paired. Translate all predicted coding regions and BLAST them against the NCBI non-redundant protein database with an E-value cutoff of 0.001. Tabulate, per predictor and per sample, the total predicted genes and the number/proportion with BLAST hits, then check qualitative agreement with Table 3 of Kelley et al. (Nucleic Acids Research, 10.1093/nar/gkr1067): Glimmer-MG should show 4.5% more BLAST-aligning predictions than FragGeneScan and 6.5% more than MetaGeneMark, with the highest proportion matching database proteins.

## Related tools

- **Glimmer-MG** (Primary gene predictor under evaluation; classification (Phymm), clustering (Scimm), cluster retraining, and indel-error-aware prediction on the 454 reads) — https://github.com/davek44/Glimmer-MG
- **Glimmer** (Framework on which Glimmer-MG is based; provides IMM-based coding-region identification) — https://github.com/davek44/Glimmer-MG
- **FragGeneScan** (Comparison gene predictor whose BLAST-aligning prediction count serves as the baseline in Table 3)
- **454 Life Sciences** (Sequencing platform that produced the real TS28/TS50 human gut reads with homopolymer-run indel errors)

## Evaluation signals

- Predicted-gene counts per predictor and sample match the values in Table 3 (qualitative agreement with the published table)
- Glimmer-MG yields 4.5% more BLAST-aligning predictions than FragGeneScan and 6.5% more than MetaGeneMark on the real reads
- Glimmer-MG shows the highest proportion of predictions with BLAST hits among the three predictors
- All BLAST searches use a consistent E-value cutoff of 0.001 against the same nr database release
- All three predictors are run on the identical input read sets (TS28 and TS50) so counts are directly comparable

## Limitations

- Predicting genes on error-prone sequences, particularly those with homopolymer-run indels, remains difficult even for Glimmer-MG
- On simulated 454 reads with 2.0% error, many genes are predicted incorrectly (precision 54.7%), so BLAST-hit counts are not a pure accuracy measure
- Glimmer-MG is more computationally expensive than existing software, making full reproduction costly
- No software version for Glimmer-MG or compared tools is stated, so exact count reproduction may drift with tool versions or nr database updates

## Evidence

- [readme] Indel error modeling on 454 reads: [evidence span withheld — anchor status: external_doc]
- [abstract] Glimmer-MG accuracy claim grounding the benchmark expectation: "we developed a metagenomics gene prediction system Glimmer-MG that achieves significantly greater accuracy than previous systems"
- [discussion] Computational cost caveat for reproduction budgeting: "Glimmer-MG has the caveat that it is more computationally expensive than existing software"
- [discussion] Indel prediction difficulty caveat: "Despite Glimmer-MG's contributions, predicting genes on error-prone sequences, particularly those with indels, remains a difficult problem"
