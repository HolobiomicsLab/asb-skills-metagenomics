---
name: assembly-quality-assessment-metrics
description: Use when assemblies from multiple de novo assemblers must be scored per
  contig against known reference replicons, requiring a Phred-like quality metric
  derived from alignment identity rather than raw sequence features.
when_to_use_negative:
- When no reference replicons are available for the sample — the metric is reference-based
  and requires aligning contigs to known community replicons.
- When the goal is reference-free assembly validation based only on contig features
  themselves (e.g. QUAST-style metrics without reference alignment).
- When contigs below the --minLength threshold (e.g. <1 kbp) are the object of study
  — they are excluded before scoring.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: LMAS
  role: Automated Nextflow workflow that aligns assembler contigs to reference replicons
    and computes per-contig PLS quality scores within its evaluation pipeline
  repo: https://github.com/B-UMMI/LMAS
- name: QUAST
  role: 'Contrast baseline: feature-based de novo assembly validation without reference
    alignment'
- name: MetaQUAST
  role: Related reference-alignment-based evaluation approach that LMAS-style reference-specific
    metrics extend
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac122
    title: 'LMAS: evaluating metagenomic short <i>de novo</i> assembly methods through
      defined communities'
  source_content_hash: sha256:7bfbe20c50bec437dd17adc6e277bac4c1186329cdabfd61135f3e1d221577bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/LMAS
  tools:
  - LMAS
  - MetaQUAST
  - QUAST
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# assembly-quality-assessment-metrics

## Summary

Computes per-contig Phred-like assembly quality scores (Pls) in LMAS by converting contig-to-reference alignment identity into an error-based score, so assembly quality of individual contigs can be compared across assemblers against known reference replicons.

## When to use

Use when assemblies from multiple de novo assemblers must be scored per contig against known reference replicons, requiring a Phred-like quality metric derived from alignment identity rather than raw sequence features.

## When NOT to use

- When no reference replicons are available for the sample — the metric is reference-based and requires aligning contigs to known community replicons.
- When the goal is reference-free assembly validation based only on contig features themselves (e.g. QUAST-style metrics without reference alignment).
- When contigs below the --minLength threshold (e.g. <1 kbp) are the object of study — they are excluded before scoring.

## Inputs

- contig-to-reference alignment results per assembler (identity of contigs over the minimum contig size threshold, e.g. 1000 bp per --minLength)
- reference replicon sequences for the defined mock community (e.g. ZymoBIOMICS Microbial Community Standards, BMock12, Gut-Mix-RR/Gut-Mix-HiLo)
- assembler-generated contigs (FASTA) from tools such as ABySS, MetaHipmer2, minia, VelvetOptimiser, SPAdes, MEGAHIT

## Outputs

- pls_scores.csv containing per-contig PLS values tagged with assembler and reference replicon
- per-contig assembly quality scores capped at a maximum Phred-like value of 60

## How to apply

Load the contig-to-reference alignment results for each assembler, restricted to contigs above the minimum contig size threshold defined by the --minLength parameter (default 1000 bp) that map to a reference replicon. For each aligned contig, compute the error rate as E = 1 - Identity, where Identity is the fraction of the alignment matching the reference replicon. Apply the Phred-like transformation PLS = -10 * log10(E), following Equation 1, with a special-case cap of 60 when E = 0 (perfect identity). Tag each score with its assembler and reference replicon and write the per-contig values to pls_scores.csv.

## Related tools

- **LMAS** (Automated Nextflow workflow that aligns assembler contigs to reference replicons and computes per-contig PLS quality scores within its evaluation pipeline) — https://github.com/B-UMMI/LMAS
- **QUAST** (Contrast baseline: feature-based de novo assembly validation without reference alignment)
- **MetaQUAST** (Related reference-alignment-based evaluation approach that LMAS-style reference-specific metrics extend)

## Examples

```
nextflow run B-UMMI/LMAS --reads 'sample_{1,2}.fastq.gz' --references reference.fasta --assemblers spades,megahit --minLength 1000
```

## Evaluation signals

- pls_scores.csv exists and contains a PLS value for every aligned contig above the --minLength threshold
- No score exceeds the cap of 60 (the special case when E = 0)
- PLS = -10 * log10(1 - Identity) reproduces the expected value for a given alignment identity (e.g. Identity = 0.99 → PLS = 20)
- Every score is tagged with both its source assembler and its reference replicon
- Contigs below the minimum contig size threshold (1000 bp) are absent from pls_scores.csv

## Limitations

- Scores depend on the completeness/accuracy of the reference replicon set; undefined or poorly represented community members cannot be scored.
- The cap of 60 for E = 0 truncates the dynamic range for perfect alignments, so top scores saturate.
- LMAS is computationally intensive due to the de novo assembly process, and assembler usability may be limited with default parameters; results may not generalize across community compositions.
- Default --minLength of 1000 bp means shorter contigs are excluded from the quality assessment entirely.

## Evidence

- [methods] Pls definition: [evidence span withheld — anchor status: not_found]
- [methods] Cap at 60: [evidence span withheld — anchor status: not_found]
- [methods] minLength parameter: "The **X** in the **Contigs > X** section corresponds to the minimum contig size as defined in the ``--minLength`` parameter"
- [discussion] Integrated pipeline rationale: "LMAS is the only software-integrating assembly and its evaluation into a single pipeline, guaranteeing the same conditions are met for all tools."
- [intro] Reference-based evaluation: "MetaQUAST [11], a modification of QUAST, extends the original software by performing assembly evaluation based on aligning contigs to a reference"
