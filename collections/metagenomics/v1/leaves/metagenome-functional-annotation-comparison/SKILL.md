---
name: metagenome-functional-annotation-comparison
description: Use when you have paired MAG and complete reference genome FASTAs with
  KEGG KO annotations and need to quantify how faithfully HMM-recovered annotations
  on MAGs reproduce isolate-genome annotations.
when_to_use_negative:
- The goal is KEGG Module completeness evaluation only (no HMM hit benchmarking) —
  use --skip_hmm outputs instead of running paired --hmm_mode onebm searches
- No matching reference isolate genome exists for the MAG, since per-pair set comparison
  and concordance fractions cannot be computed
- Evaluating de novo GSMM gapfilling quality — that requires --gsmm_mode and flux
  variability analysis, not HMM hit concordance
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0121
tools:
- name: KEMET
  role: Runs profile HMM searches for missing KOs on both MAGs and reference genomes
    via kemet.py --hmm_mode onebm, whose outputs are compared per pair
  repo: https://github.com/Matteopaluh/KEMET
- name: eggNOG-Mapper
  role: Produces the .emapper.annotations KEGG KO annotation files consumed by kemet.py
    via -a eggnog
- name: HMMER
  role: Underlying profile HMM search engine invoked by KEMET; soft errors silenced
    with -q
- name: IGGdb
  role: Pointer to complete reference genomes recovered from the PATRIC database
  repo: https://github.com/snayfach/IGGdb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:f49bdc2f3cadde50c61c9d75b6b29e458d8f6034088912d1bec4a9c2417e89fe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Matteopaluh/KEMET
  repo_url: https://github.com/Matteopaluh/KEMET
  tools:
  - HMMER
  - IGGdb
  - KEMET
  - eggNOG-Mapper
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metagenome-functional-annotation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that compares KEGG ortholog hits recovered by KEMET HMM profile searches on MAGs against the hits found in matching reference isolate genomes, quantifying annotation-recovery concordance as precision, sensitivity, and shared/unique hit fractions.

## When to use

Use when you have paired MAG and complete reference genome FASTAs with KEGG KO annotations and need to quantify how faithfully HMM-recovered annotations on MAGs reproduce isolate-genome annotations.

## When NOT to use

- The goal is KEGG Module completeness evaluation only (no HMM hit benchmarking) — use --skip_hmm outputs instead of running paired --hmm_mode onebm searches
- No matching reference isolate genome exists for the MAG, since per-pair set comparison and concordance fractions cannot be computed
- Evaluating de novo GSMM gapfilling quality — that requires --gsmm_mode and flux variability analysis, not HMM hit concordance

## Inputs

- MAG FASTA files (e.g. 12 bins)
- Matching complete reference isolate genome FASTAs from NCBI/PATRIC
- KEGG KO annotation files in eggNOG format (.emapper.annotations)
- KEMET installation with HMMER and MAFFT (conda environment per KEMET wiki)

## Outputs

- Per-pair sets of HMM-recovered KOs classified as common, MAG-only, or complete-genome-only
- Aggregated concordance fractions (e.g. 84.76% shared, 7.62% complete-genome-only, 7.62% MAG-only)
- Derived precision and sensitivity metrics (e.g. both 91.75%)
- hmm_concordance_metrics.csv summary table

## How to apply

Collect the 12 MAG FASTA files and their corresponding complete reference genomes (recovered from NCBI or PATRIC, e.g. via the IGGdb pointer) together with their KEGG KO annotations in eggNOG format (.emapper.annotations). Run kemet.py on each MAG with -a eggnog, --hmm_mode onebm and --skip_gsmm to search for missing KOs, keeping the default legit-hit quality threshold of 0.43 and using -q to silence MAFFT and HMMER soft errors. Repeat the identical KEMET run on each matching reference complete genome so the only variable is assembly completeness. For each genome pair, classify HMM-recovered KOs as common, MAG-only, or complete-genome-only, then aggregate counts across the 12 pairs. Compute concordance fractions and derive precision and sensitivity as common/(common+MAG-only) and common/(common+complete-genome-only); write the metrics to hmm_concordance_metrics.csv.

## Related tools

- **KEMET** (Runs profile HMM searches for missing KOs on both MAGs and reference genomes via kemet.py --hmm_mode onebm, whose outputs are compared per pair) — https://github.com/Matteopaluh/KEMET
- **eggNOG-Mapper** (Produces the .emapper.annotations KEGG KO annotation files consumed by kemet.py via -a eggnog)
- **HMMER** (Underlying profile HMM search engine invoked by KEMET; soft errors silenced with -q)
- **IGGdb** (Pointer to complete reference genomes recovered from the PATRIC database) — https://github.com/snayfach/IGGdb

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --skip_gsmm -q --log
```

## Evaluation signals

- Per-pair KO hit sets must partition exactly into common, MAG-only, and complete-genome-only categories with no unclassified hits
- Aggregated fractions must match expected values: 84.76% in common, 7.62% only in complete genomes, 7.62% only in MAGs
- Derived precision and sensitivity must both equal 84.76/(84.76+7.62) ≈ 91.75%
- False positive and false negative rates must each equal 7.62%
- hmm_concordance_metrics.csv must contain the summary metrics matching the reported values

## Limitations

- The default legit-hit HMM quality threshold of 0.43 governs what counts as a hit; changing --threshold_value will shift concordance fractions
- Mock KO removal tests recovered only 55% (20/36) of removed annotations for MAGs and 67% (8/12) for complete genomes, so HMM recovery is not guaranteed for all missing KOs
- MAG-based analyses remain sensitive to assembly gapping, contamination, and short contigs; lower-quality genomes need special care to avoid missingness artifacts

## Evidence

- [results] the corresponding complete genomes recovered from the NCBI or from the PATRIC database: [evidence span withheld — anchor status: not_found]
