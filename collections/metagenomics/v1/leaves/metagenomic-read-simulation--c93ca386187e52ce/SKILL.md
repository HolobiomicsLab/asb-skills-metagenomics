---
name: metagenomic-read-simulation
description: Use when benchmarking HGT breakpoint detection or local strain reconstruction
  pipelines against known ground truth, so sensitivity and FDR can be computed from
  simulated reads at defined coverages.
when_to_use_negative:
- Analyzing real shotgun metagenomics samples where no ground-truth breakpoints exist
  — sensitivity/FDR against true positions cannot be computed
- Evaluating tools whose input requirements differ from paired-end shotgun reads (e.g.,
  long-read or assembly-based HGT callers)
- Simulations requiring realistic community abundance profiles or sequencing error
  models beyond WGSIM's defaults when the benchmark design does not specify them
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: WGSIM
  role: Generates paired-end simulated reads from the true local strains at specified
    coverage levels (2X, 5X, 30X, 70X)
  repo: https://github.com/lichen2018/LEMON
- name: BWA
  role: Aligns simulated reads against the reference genomes to produce the BAM input
    for breakpoint callers
  repo: https://github.com/lichen2018/LEMON
- name: LEMON
  role: 'Downstream consumer of the simulated data: detects raw/accurate HGT breakpoints
    and reconstructs local strains from the unique- and split-read BAMs'
  repo: https://github.com/lichen2018/hgt-detection
- name: LUMPY
  role: Comparison caller run with default parameters on the same simulated BAM inputs
    to compute baseline sensitivity/FDR
- name: Samtools/HTSlib
  role: Extracts unique reads (samtools view -q 20) and prepares BAM/coverage files
    required by the LEMON workflow
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:61b0845bc466073ac6c0e2f8d52e6fdf07cb3ed25216e5251e8f9f480312dbbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - LEMON
  - LUMPY
  - Samtools/HTSlib
  - WGSIM
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# metagenomic-read-simulation

## Summary

Simulate paired-end shotgun reads from constructed true local strains (with HGT-inserted segments) using WGSIM across multiple coverage levels to create controlled benchmark data for evaluating HGT breakpoint detection and strain reconstruction tools like LEMON and LUMPY.

## When to use

Use when benchmarking HGT breakpoint detection or local strain reconstruction pipelines against known ground truth, so sensitivity and FDR can be computed from simulated reads at defined coverages.

## When NOT to use

- Analyzing real shotgun metagenomics samples where no ground-truth breakpoints exist — sensitivity/FDR against true positions cannot be computed
- Evaluating tools whose input requirements differ from paired-end shotgun reads (e.g., long-read or assembly-based HGT callers)
- Simulations requiring realistic community abundance profiles or sequencing error models beyond WGSIM's defaults when the benchmark design does not specify them

## Inputs

- True local strain sequences (reference genomes with 10 kbp transferred HGT segments inserted)
- Metagenomic reference set (e.g., 16,093 species / 1,246,881 scaffolds from NCBI bacterial genomes)
- Target coverage levels (2X, 5X, 30X, 70X)
- WGSIM paired-end read simulation parameters

## Outputs

- Simulated paired-end FASTQ read files at each coverage level
- BWA alignment BAM against the reference set
- Unique-read BAM (samtools view -q 20) and split-read BAM (lumpy-sv extractSplitReads_BwaMem)
- Ground-truth breakpoint list for sensitivity/FDR and RA scoring (e.g., sensitivity_fdr_table.csv, reconstruction_accuracy_comparison.csv)

## How to apply

First construct true local strains by randomly selecting receptor and donor genomes from the reference set (e.g., 100 microbial genomes yielding 60 local strains with 4260 HGT breakpoints, each with 10 kbp transferred segments inserted). Then generate paired-end reads from each true local strain with WGSIM at multiple coverage levels (e.g., 2X, 5X, 30X, 70X) to probe low- and high-coverage performance. Align the simulated reads against the reference with BWA (bwa mem -M) and derive the unique-read BAM (samtools view -q 20), split-read BAM (lumpy-sv extractSplitReads_BwaMem), and coverage file that downstream callers require. Finally, score detected breakpoints against the simulated ground truth using a 20 bp tolerance and compute sensitivity (true detected / all true breakpoints) and FDR (false discovered / all discovered). The rationale is that only with simulated reads can per-breakpoint sensitivity/FDR and reconstruction accuracy (RA via Smith-Waterman, match_score = 1, mismatch_score = -1) be computed exactly, as done for LEMON vs LUMPY.

## Related tools

- **WGSIM** (Generates paired-end simulated reads from the true local strains at specified coverage levels (2X, 5X, 30X, 70X)) — https://github.com/lichen2018/LEMON
- **BWA** (Aligns simulated reads against the reference genomes to produce the BAM input for breakpoint callers) — https://github.com/lichen2018/LEMON
- **LEMON** (Downstream consumer of the simulated data: detects raw/accurate HGT breakpoints and reconstructs local strains from the unique- and split-read BAMs) — https://github.com/lichen2018/hgt-detection
- **LUMPY** (Comparison caller run with default parameters on the same simulated BAM inputs to compute baseline sensitivity/FDR)
- **Samtools/HTSlib** (Extracts unique reads (samtools view -q 20) and prepares BAM/coverage files required by the LEMON workflow) — https://github.com/samtools/htslib

## Examples

```
wgsim -1 150 -2 150 -d 350 -s 100 true_local_strain.fa sim_5x_1.fq sim_5x_2.fq; bwa mem -M ref.fa sim_5x_1.fq sim_5x_2.fq | samtools view -bS - > aln.bam; samtools view -q 20 aln.bam > unique.bam
```

## Evaluation signals

- Sensitivity and FDR at each coverage fall close to the reported values: LEMON 22.39%/51.19% sensitivity at 2X/5X, 0.002 FDR at 2X, 0.0034 FDR at 70X; LUMPY 6.86%/31.12% sensitivity, 0.016/0.010 FDR
- Every true breakpoint (4260 across 60 local strains) is recoverable from the simulation setup, so a detected breakpoint within 20 bp of the true position counts as a true positive
- Reconstruction accuracy matches reported figures: LEMON RA 0.6087 (5X) and 1.0 (30X); LUMPY RA 0.5238 (5X) and 0.9565 (30X), scored by Smith-Waterman with match_score = 1, mismatch_score = -1
- The comparison tables (sensitivity_fdr_table.csv, reconstruction_accuracy_comparison.csv) show LEMON with higher sensitivity and lower FDR than LUMPY across all coverage levels
- Unique-read BAM is non-empty after samtools view -q 20 filtering, confirming alignment quality sufficient for downstream breakpoint detection

## Limitations

- Simulations use randomly selected receptors/donors and fixed 10 kbp transferred segments, so results may not generalize to real HGT fragment size distributions or community structures
- Third-party tools were run with default parameters (BWA, LUMPY), so benchmark outcomes depend on those default settings rather than tuned configurations
- Detection scoring depends on the 20 bp tolerance; sensitivity/FDR values are only comparable under the same tolerance convention

## Evidence

- [methods] All true local strains were used to generated 20X paired-end reads with WGSIM [28] as an input of LEMON.: "All true local strains were used to generated 20X paired-end reads with WGSIM [28] as an input of LEMON."
- [methods] Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA).: "Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA)."
- [abstract] The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome.: "The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome."
- [readme] It is a software takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred genome segments, and reconstructs the inserted local strain.: "It is a software takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred genome segments, and reconstructs the inserted local strain."
