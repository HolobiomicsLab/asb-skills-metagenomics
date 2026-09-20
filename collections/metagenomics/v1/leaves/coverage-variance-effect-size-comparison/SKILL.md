---
name: coverage-variance-effect-size-comparison
description: Use when binning predicted viral scaffolds into vMAGs from multi-sample
  metagenomes where scaffolds share similar sequence features but differ in read-coverage
  abundance across samples, and sorted BAM mappings are available.
when_to_use_negative:
- The input is a whole metagenome rather than virus-predicted scaffolds, or contains
  non-viral microbial genomes — vRhyme is built for viral sequences and is not meant
  to bin microbes, and performance on whole metagenomes has not been fully evaluated
- The community contains multiple inactive (non-replicating) prophages from the same
  host genome — vRhyme had difficulties separating these despite coverage comparisons
- Only one coverage sample is available for a highly redundant dataset where coverage
  variance is the sole discriminating signal — vRhyme performs optimally with multiple
  input samples
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: vRhyme
  role: Implements the coverage effect size comparisons, network construction, and
    ML-based binning of viral scaffolds
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Bowtie2
  role: Read alignment option producing the BAM mappings from which coverage variance
    is calculated
  repo: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- name: BWA
  role: Alternative read aligner for generating scaffold coverage BAMs
  repo: https://github.com/lh3/bwa
- name: Samtools
  role: Required for handling BAM coverage inputs other than -c during coverage calculation
  repo: http://www.htslib.org/
- name: pysam
  role: Python SAM/BAM wrapper used by vRhyme for coverage parsing (version >= 0.15.0)
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:40ccf6103ec6ce791d2b2dd06d48decfd932f45ed90cec8c5e0496458e1f2082
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/AnantharamanLab/vRhyme
  repo_url: https://github.com/AnantharamanLab/vRhyme
  tools:
  - BWA
  - Bowtie2
  - CoCoNet
  - MetaBat2
  - Python
  - Samtools
  - pysam
  - vRhyme
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# coverage-variance-effect-size-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

vRhyme's core binning signal: single- or multi-sample read coverage abundance effect sizes are compared between viral scaffolds to weight binning networks, separating scaffolds with similar nucleotide features (GC, tetranucleotide frequencies) that are actually different genomes.

## When to use

Use when binning predicted viral scaffolds into vMAGs from multi-sample metagenomes where scaffolds share similar sequence features but differ in read-coverage abundance across samples, and sorted BAM mappings are available.

## When NOT to use

- The input is a whole metagenome rather than virus-predicted scaffolds, or contains non-viral microbial genomes — vRhyme is built for viral sequences and is not meant to bin microbes, and performance on whole metagenomes has not been fully evaluated
- The community contains multiple inactive (non-replicating) prophages from the same host genome — vRhyme had difficulties separating these despite coverage comparisons
- Only one coverage sample is available for a highly redundant dataset where coverage variance is the sole discriminating signal — vRhyme performs optimally with multiple input samples

## Inputs

- Nucleotide FASTA of predicted viral scaffolds (vRhyme -i), e.g., ≥5 kb, 98%-dereplicated VIBRANT/VirSorter predictions
- Sorted BAM files of metagenomic reads mapped to the scaffolds, one per sample (vRhyme -b), or an alternate coverage input via -c
- Optional FASTQ reads (vRhyme -r/-v/-u) with aligner choice (--aligner: Bowtie2 or BWA)

## Outputs

- Weighted coverage-similarity networks per binning iteration, refined into bins
- vRhyme_best_bins.#.membership.tsv (scaffold-to-bin membership) and vRhyme_best_bins.#.summary.tsv
- vRhyme_bin_scoring.tsv with Mmseqs2-based protein redundancy scores per bin
- vMAG bin FASTA files for downstream CheckV completeness/contamination assessment

## How to apply

Provide virus-predicted (e.g., VIBRANT or VirSorter), dereplicated scaffolds as the -i nucleotide FASTA, removing circular (complete) scaffolds pre-binning. Map metagenomic reads to the scaffolds with Bowtie2 or BWA and supply sorted BAM files via -b (or a precomputed coverage table via -c); vRhyme computes coverage automatically using default alignment quality filters (--read_id 0.97, --mask 150) and coverage outlier removal (--outliers 4.0). For each scaffold pair, vRhyme compares single- or multi-sample coverage effect sizes — scaffolds with concordant abundance profiles across samples gain weighted network edges (dynamic --max_edges 3-6) — because coverage variance separates scaffolds that share sequence features but belong to different genomes. These effect-size edges are combined with supervised machine learning classification of sequence feature compositions (neural network + Extra Trees, --model hybrid) and bins are refined over --iter 20 iterations using Label Propagation. Use as many input samples as the dataset supports, since vRhyme performs optimally with multiple samples for more robust coverage variance comparisons; Samtools/pysam handle BAM parsing.

## Related tools

- **vRhyme** (Implements the coverage effect size comparisons, network construction, and ML-based binning of viral scaffolds) — https://github.com/AnantharamanLab/vRhyme
- **Bowtie2** (Read alignment option producing the BAM mappings from which coverage variance is calculated) — http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- **BWA** (Alternative read aligner for generating scaffold coverage BAMs) — https://github.com/lh3/bwa
- **Samtools** (Required for handling BAM coverage inputs other than -c during coverage calculation) — http://www.htslib.org/
- **pysam** (Python SAM/BAM wrapper used by vRhyme for coverage parsing (version >= 0.15.0)) — https://github.com/pysam-developers/pysam

## Examples

```
vRhyme -i viral_scaffolds.fna -b sample1.sorted.bam sample2.sorted.bam -m -p meta
```

## Evaluation signals

- Coverage calculation ran with default alignment quality filters (--read_id 0.97, --mask 150) and outlier removal (--outliers 4.0); circular scaffolds were flagged and removed pre-binning
- Output hierarchy contains vRhyme_best_bins.#.membership.tsv, vRhyme_best_bins.#.summary.tsv, and vRhyme_bin_scoring.tsv
- On the GOV2 benchmark, coverage-variance-driven binning yields 15,106 redundant proteins and 11,909 low-contamination bins, matching Supplementary Table S7 and beating MetaBat2 (29,334) and CoCoNet (71,364)
- aux/test_vRhyme.py reports Success for Samtools, which is required whenever a coverage input other than -c is used
- Scaffolds with similar tetranucleotide/GC features but distinct genomes are assigned to separate bins rather than merged

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- vRhyme performs optimally with multiple input samples for more robust coverage variance comparisons, though the optimal number depends on how the dataset or metagenome was constructed (Supplementary Table S8, Figure S6)
- vRhyme is built for viral sequences; performance on a whole metagenome has not been fully evaluated, and binning non-viral genomes is an anticipated limitation (MetaBat2 outperformed vRhyme on a non-viral synthetic dataset, Supplementary Table S11)

## Evidence
