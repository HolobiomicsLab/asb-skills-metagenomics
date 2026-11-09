---
name: mag-reference-genome-concordance-analysis
description: Use when you have paired MAG and matching complete isolate genomes (e.g.
  from NCBI/PATRIC) with KEGG KO annotations and need to benchmark HMM-recovered missing
  KOs against a complete-genome reference.
when_to_use_negative:
- You only have MAGs without matching complete reference isolate genomes — there is
  no reference against which to compute concordance.
- Your goal is KEGG Module completeness evaluation alone; run KEMET with --skip_hmm
  instead of the HMM concordance workflow.
- Your annotations are not in a supported KEGG KO format (eggnog / kaas / kofamkoala)
  — KEMET expects one of these annotation formats.
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0196
tools:
- name: KEMET
  role: Executes profile HMM searches for missing KOs (--hmm_mode onebm) on both MAGs
    and reference complete genomes so their recovered KO sets can be compared
  repo: https://github.com/Matteopaluh/KEMET
- name: HMMER
  role: Underlying profile HMM search engine used by KEMET to score candidate ortholog
    hits against the legit-hit threshold
- name: IGGdb
  role: Source of reference complete genomes from PATRIC pointed to when recovering
    isolate genomes matching the MAGs
  repo: https://github.com/snayfach/IGGdb
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.015
    title: KEMET – A python tool for KEGG Module evaluation and microbial genome annotation
      expansion
  source_content_hash: sha256:8edfbaa5cc96d399989de90700ca5912cf820a7bd7fc4e94303802218964b83d
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
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mag-reference-genome-concordance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies agreement between KEGG ortholog (KO) hits recovered by KEMET profile HMM searches on MAGs versus the identical searches run on their corresponding complete reference isolate genomes, yielding precision/sensitivity-style concordance metrics.

## When to use

Use when you have paired MAG and matching complete isolate genomes (e.g. from NCBI/PATRIC) with KEGG KO annotations and need to benchmark HMM-recovered missing KOs against a complete-genome reference.

## When NOT to use

- You only have MAGs without matching complete reference isolate genomes — there is no reference against which to compute concordance.
- Your goal is KEGG Module completeness evaluation alone; run KEMET with --skip_hmm instead of the HMM concordance workflow.
- Your annotations are not in a supported KEGG KO format (eggnog / kaas / kofamkoala) — KEMET expects one of these annotation formats.

## Inputs

- MAG FASTA files (e.g. 12 MAGs per study set)
- Corresponding complete reference isolate genome FASTA files (NCBI/PATRIC)
- KEGG KO annotation files in eggNOG format (.emapper.annotations) for each genome

## Outputs

- Per-genome sets of KOs recovered by KEMET HMM search (onebm mode)
- Hit-classification table: common, MAG-only, complete-genome-only KO counts per pair
- Aggregate concordance fractions and derived precision/sensitivity metrics (e.g. hmm_concordance_metrics.csv)

## How to apply

Collect the MAG FASTA files and their corresponding complete reference genomes (recovered from NCBI or PATRIC, e.g. via IGGdb pointers) together with KO annotation files in eggNOG format (.emapper.annotations). Run kemet.py on each MAG with -a eggnog and --hmm_mode onebm --skip_gsmm to search for missing KOs, keeping the default legit-hit quality threshold of 0.43 and using -q to silence MAFFT and HMMER soft-errors. Repeat the identical KEMET --hmm_mode onebm run on each matching reference complete genome. For each genome pair, compare the sets of HMM-recovered KOs, classifying hits as common, MAG-only, or complete-genome-only, then aggregate counts across pairs (e.g. the reported 84.76% common, 7.62% complete-genome-only, 7.62% MAG-only) and derive precision and sensitivity as common/(common+MAG-only) or common/(common+complete-genome-only), both ≈ 91.75%.

## Related tools

- **KEMET** (Executes profile HMM searches for missing KOs (--hmm_mode onebm) on both MAGs and reference complete genomes so their recovered KO sets can be compared) — https://github.com/Matteopaluh/KEMET
- **HMMER** (Underlying profile HMM search engine used by KEMET to score candidate ortholog hits against the legit-hit threshold)
- **IGGdb** (Source of reference complete genomes from PATRIC pointed to when recovering isolate genomes matching the MAGs) — https://github.com/snayfach/IGGdb

## Examples

```
./kemet.py genomes/bin1.fasta -a eggnog --hmm_mode onebm --skip_gsmm -q --log
```

## Evaluation signals

- The concordance split must sum to ~100% across common, complete-genome-only, and MAG-only hits (e.g. 84.76% / 7.62% / 7.62%).
- Derived precision and sensitivity both equal common/(common + discordant share) ≈ 91.75% in the reported validation.
- Every KO set compared must come from identical KEMET runs (same -a eggnog, --hmm_mode onebm, default 0.43 threshold) on paired MAG/reference FASTA files.
- Summary metrics file (e.g. hmm_concordance_metrics.csv) reproduces the reported fractions within rounding error.

## Limitations

- HMM hits on MAGs may differ from complete genomes due to missing contig regions; in validation ~7.62% of hits were MAG-only and ~7.62% complete-genome-only.
- MAFFT and HMMER soft-errors may occur during the search and should be silenced with -q but reviewed; the default legit-hit threshold of 0.43 governs hit acceptance and can be changed via --threshold_value.
- The mock KO removal test showed only 55% (20/36) and 67% (8/12) true-positive recovery rates for MAGs and complete genomes respectively, so HMM recovery is not guaranteed for every missing KO, particularly truncated sequences.

## Evidence

- [results] reference genomes from NCBI/PATRIC: [evidence span withheld — anchor status: not_found]
