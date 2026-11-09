---
name: contig-alignment-identity-metric-implementation
description: Use when you have contig-to-reference alignment results (identity per
  aligned contig ≥1000 bp mapping to known reference replicons) and need per-contig
  Phred-like assembly quality scores for assembler benchmarking.
when_to_use_negative:
- Input is raw assembly contigs without reference alignments — identity cannot be
  computed without mapping contigs to reference replicons first
- The goal is reference-free assembly quality assessment (e.g. QUAST-style feature-based
  metrics) rather than reference-based accuracy scoring
- Evaluating contigs shorter than the 1000 bp --minLength threshold, which are excluded
  before scoring
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: LMAS
  role: automated Nextflow workflow that aligns assembled contigs to reference replicons
    and computes the per-contig PLS score as part of its evaluation pipeline
  repo: https://github.com/B-UMMI/LMAS
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:5a65afa33d2b36f36a9d842bb13e6f1705cc3f629d4c995e20c7eb21b998f5fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# contig-alignment-identity-metric-implementation

## Summary

Implements the LMAS Pls (Phred-like assembly score) per contig, converting contig-to-reference replicon alignment identity into a Phred-like quality value with a cap of 60, so contig-level assembly accuracy can be compared across assemblers on defined mock communities.

## When to use

Use when you have contig-to-reference alignment results (identity per aligned contig ≥1000 bp mapping to known reference replicons) and need per-contig Phred-like assembly quality scores for assembler benchmarking.

## When NOT to use

- Input is raw assembly contigs without reference alignments — identity cannot be computed without mapping contigs to reference replicons first
- The goal is reference-free assembly quality assessment (e.g. QUAST-style feature-based metrics) rather than reference-based accuracy scoring
- Evaluating contigs shorter than the 1000 bp --minLength threshold, which are excluded before scoring

## Inputs

- contig-to-reference alignment results per assembler, including per-alignment identity of each contig to the reference replicon
- reference replicon sequences (FASTA) of the defined mock community
- minimum contig length threshold (1000 bp via --minLength)

## Outputs

- pls_scores.csv with one row per aligned contig, tagged with assembler and reference, containing the per-contig PLS value (capped at 60)

## How to apply

Load the contig-to-reference alignment results for each assembler, retaining only contigs over the minimum contig size threshold (1000 bp, set by the --minLength parameter) that map to each reference replicon. For each aligned contig, compute the error rate as E = 1 - Identity, where Identity is the fraction of the alignment matching the reference. Then compute the Phred-like score as -10 * log10(E), applying the special-case cap of 60 when E = 0 (a perfect alignment would otherwise yield an infinite value). Write the per-contig PLS values tagged with assembler and reference to pls_scores.csv so scores can be aggregated per assembler and per reference replicon in the LMAS report.

## Related tools

- **LMAS** (automated Nextflow workflow that aligns assembled contigs to reference replicons and computes the per-contig PLS score as part of its evaluation pipeline) — https://github.com/B-UMMI/LMAS

## Examples

```
python scripts/compute_pls.py --identity alignments.tsv --min-length 1000 --cap 60 --out pls_scores.csv
```

## Evaluation signals

- pls_scores.csv exists and contains a PLS value for every aligned contig passing the 1000 bp --minLength threshold
- No score exceeds the cap of 60; any value above 60 indicates a bug in the E = 0 handling
- Perfectly identical alignments (Identity = 1.0, E = 0) must yield exactly 60, not infinity or NaN
- Scores should be reproducible across the 3 LMAS runs per sample with default parameters (deterministic given the same alignment input)
- Per-contig PLS values should be traceable to the corresponding alignment identity: PLS = -10*log10(1 - Identity) for 0 < Identity < 1

## Limitations

- Score depends entirely on the availability and quality of reference replicons; it is not reference-free and assumes contigs are aligned to the correct replicon
- The E = 0 cap of 60 bounds perfect-align scores, so truly flawless contigs cannot be distinguished from those with very small errors near the cap
- Contigs below the --minLength threshold (1000 bp) are excluded, so short-contig assembly quality is invisible to this metric

## Evidence

- [methods] The Pls is a Phred-like score computed per contig using as the error function the identity of each aligned contig to the reference replicon (E = 1 - Identity), following Equation 1 with a cap of 60 when E = 0: [evidence span withheld — anchor status: not_found]
- [methods] minimum contig size threshold (--minLength): "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [discussion] LMAS integrates assembly and evaluation: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [results] workflow run with default parameters: "the LMAS workflow was run 3 times for all samples using default parameters"
