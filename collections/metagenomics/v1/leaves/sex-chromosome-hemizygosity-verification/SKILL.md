---
name: sex-chromosome-hemizygosity-verification
description: Use when evaluating diploid-state recovery of a de novo human genome
  assembly and you must verify that single-copy (hemizygous) X regions are reconstructed
  as one contig rather than incorrectly diploid or uncovered.
when_to_use_negative:
- Input is simulated Linked-Read data with known gold-standard haplotypes, where direct
  assembly-vs-truth comparison (LRTK-SIM simulation) is the appropriate accuracy assessment
- Assemblies not generated from 10x Chromium Linked-Read data by Supernova2, since
  no phase-block haplotype scaffold pairs are available for the window coverage classification
- Non-human genomes lacking an hg38-equivalent high-quality reference and known hemizygous
  X coordinate set
edam_topics: []
tools:
- name: Supernova2
  role: De novo assembler whose mkoutput output supplies the haplotype scaffold pairs
    classified per 500-bp window; also the pipelines that accepted the simulated/real
    Linked-Read data
  repo: https://support.10xgenomics.com/de-novo-assembly/software/overview/welcome
- name: Evaluate_diploid_assembly
  role: Notebook-based code (Evaluate_assembly_performance.ipynb) for evaluating and
    comparing diploid assemblies, including the window coverage classification underlying
    the hemizygosity check
  repo: https://github.com/zhanglu295/Evaluate_diploid_assembly
- name: hg38 human reference sequence
  role: Reference to which haplotype scaffolds are aligned to obtain genomic coordinates
    for window sliding and X-chromosome hemizygous region definition
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giz141
    title: Assessment of human diploid genome assembly with 10x Linked-Reads data
  source_content_hash: sha256:a6f586f3125572b9f0b5bcc74e948201eb8d7d50dc1851155ce337204730aed6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Evaluate_diploid_assembly
  - Supernova2
  - hg38 human reference sequence
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Sex-chromosome hemizygosity verification

## Summary

A validation step that checks whether hemizygous regions of the human X chromosome in Supernova2 10x Linked-Read diploid assemblies are spanned by a single haplotype contig, confirming that the window-based diploid/hemizygous classification correctly captures male-like single-copy X coverage.

## When to use

Use when evaluating diploid-state recovery of a de novo human genome assembly and you must verify that single-copy (hemizygous) X regions are reconstructed as one contig rather than incorrectly diploid or uncovered.

## When NOT to use

- Input is simulated Linked-Read data with known gold-standard haplotypes, where direct assembly-vs-truth comparison (LRTK-SIM simulation) is the appropriate accuracy assessment
- Assemblies not generated from 10x Chromium Linked-Read data by Supernova2, since no phase-block haplotype scaffold pairs are available for the window coverage classification
- Non-human genomes lacking an hg38-equivalent high-quality reference and known hemizygous X coordinate set

## Inputs

- Supernova2 haplotype scaffold pairs (megabubble phase blocks) exported via mkoutput from the six real 10x Linked-Read assemblies (NA12878 and NA24385 libraries)
- hg38 human reference sequence for coordinate mapping
- X-chromosome coordinates defining hemizygous regions

## Outputs

- Per-500-bp-window classification (diploid / hemizygous / uncovered) across the genome for each assembly
- Fraction of hemizygous X windows spanned by a single contig (expected 79.9%-87.6%)
- Rows in diploid_assembly_stats.tsv with per-assembly diploid fraction and overall genome coverage

## How to apply

After exporting megabubble haplotype scaffold pairs from phase blocks with 'Supernova2 mkoutput' for each real-data assembly, align the extracted haplotype scaffolds to the hg38 human reference to obtain genomic coordinates. Slide non-overlapping 500-bp windows across the genome and classify each window as diploid (covered by both homologous contigs), hemizygous, or uncovered based on contig alignment coverage. For windows falling in hemizygous X regions, verify that a single contig spans the window in 79.9%-87.6% of cases, consistent with the paper's real-data assemblies. Use the Evaluate_diploid_assembly notebook to compute per-window coverage classes and aggregate the hemizygous-X spanning statistics before writing the assembly statistics table (diploid_assembly_stats.tsv). This check guards against misclassifying single-copy X regions as diploid artifacts of mis-joined phase blocks.

## Related tools

- **Supernova2** (De novo assembler whose mkoutput output supplies the haplotype scaffold pairs classified per 500-bp window; also the pipelines that accepted the simulated/real Linked-Read data) — https://support.10xgenomics.com/de-novo-assembly/software/overview/welcome
- **Evaluate_diploid_assembly** (Notebook-based code (Evaluate_assembly_performance.ipynb) for evaluating and comparing diploid assemblies, including the window coverage classification underlying the hemizygosity check) — https://github.com/zhanglu295/Evaluate_diploid_assembly
- **hg38 human reference sequence** (Reference to which haplotype scaffolds are aligned to obtain genomic coordinates for window sliding and X-chromosome hemizygous region definition)

## Examples

```
jupyter nbconvert --to notebook --execute Evaluate_assembly_performance.ipynb
```

## Evaluation signals

- Hemizygous X regions are spanned by a single contig in 79.9%-87.6% of 500-bp windows per assembly
- Overall genome coverage of the real-data assemblies is ~91%
- Per-assembly diploid fractions fall in the 70%-80% band (R6 library at 58.9%)
- Cross-assembly diploid-region overlap reproduces 50.24% for NA12878 and 67.27% for NA24385 within tolerance
- Every genome window receives exactly one class label (diploid, hemizygous, or uncovered) from non-overlapping 500-bp tiling

## Limitations

- The check is specific to the human X chromosome and hg38; the hemizygous-region boundary definition depends on the reference annotation used
- Only the ∼91% coverage and window-classification results from the six real-data assemblies are supported; simulated-data evaluation instead relies on gold-standard comparison against known haplotypes
- Assembly quality depends on library parameters (CF between ~332x and 823x, WμFL ~50-150 kb); libraries outside the optimal parameter space (e.g. R6, 58.9% diploid) yield weaker validation statistics

## Evidence

- [other] Verify that hemizygous X regions are spanned by a single contig in 79.9-87.6% of windows.: [evidence span withheld — anchor status: not_found]
- [other] Slide non-overlapping 500-bp windows across the genome and classify each window per assembly as diploid (covered by both haplotypes), hemizygous, or uncovered: [evidence span withheld — anchor status: not_found]
- [abstract] When broadly optimal parameters were used for library preparation and sequencing, ∼80% of the genome was assembled in a diploid state.: "When broadly optimal parameters were used for library preparation and sequencing, ∼80% of the genome was assembled in a diploid state."
- [readme] This is a description of how to evaluate and compare diploid assemblies of different parameters.: [evidence span withheld — anchor status: external_doc]
