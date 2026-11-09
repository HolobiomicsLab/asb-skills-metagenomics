---
name: viral-genome-benchmark-dataset-preparation
description: Use when building simulated virome benchmarks with fragmented complete
  viral genomes as ground truth to evaluate viral binners such as vRhyme, MetaBat2,
  VAMB, CONCOCT, BinSanity, or CoCoNet.
when_to_use_negative:
- The input is a natural metagenome (e.g., GOV2 or agricultural soil virome) rather
  than a simulated dataset with fragmented ground-truth genomes, so metrics cannot
  be computed against known references
- The goal is binning microbes or non-viral genomes; vRhyme is explicitly not meant
  for this and MetaBat2 outperformed vRhyme on a non-viral synthetic dataset
- Input scaffolds have not been predicted as viral (the typical workflow is to pre-predict
  viruses with VIBRANT or VirSorter before binning)
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: vRhyme
  role: Benchmarked viral binning tool consuming the prepared FASTA, BAM, and protein
    inputs; collect vRhyme_best_bins membership and summary files
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Bowtie2
  role: Aligns simulated per-sample reads to fragmented scaffolds with '--no-unal
    --no-discordant' to generate coverage BAMs
  repo: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- name: BWA
  role: Alternative aligner ('mem') for producing sorted BAM coverage inputs
  repo: https://github.com/lh3/bwa
- name: Prodigal
  role: Predicts ORFs with '-m -p meta' to supply gene/protein inputs to the binners
  repo: https://github.com/hyattpd/Prodigal
- name: Samtools
  role: Sorts alignment output into sorted BAM files and is required for coverage-based
    vRhyme runs
  repo: http://www.htslib.org/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:22d2e6955a5e1556936cd432db9fe5236f5982a0a4fff74cd41ba3b4b3ac17fb
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
  - BinSanity
  - Bowtie2
  - CONCOCT
  - CoCoNet
  - MetaBat2
  - Mmseqs2
  - Prodigal
  - Samtools
  - VAMB
  - vRhyme
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# viral-genome-benchmark-dataset-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare simulated virome benchmark datasets from curated reference viral genomes (999 complete genomes fragmented into 4,324 scaffolds across nine datasets) with per-sample read sets, sorted BAM alignments, and Prodigal ORF predictions so that viral binning tools can be scored against ground truth with recall, precision, accuracy, specificity, and F1.

## When to use

Use when building simulated virome benchmarks with fragmented complete viral genomes as ground truth to evaluate viral binners such as vRhyme, MetaBat2, VAMB, CONCOCT, BinSanity, or CoCoNet.

## When NOT to use

- The input is a natural metagenome (e.g., GOV2 or agricultural soil virome) rather than a simulated dataset with fragmented ground-truth genomes, so metrics cannot be computed against known references
- The goal is binning microbes or non-viral genomes; vRhyme is explicitly not meant for this and MetaBat2 outperformed vRhyme on a non-viral synthetic dataset
- Input scaffolds have not been predicted as viral (the typical workflow is to pre-predict viruses with VIBRANT or VirSorter before binning)

## Inputs

- Curated reference datasets of complete viral genomes (Supplementary Table S1: 999 genomes)
- Fragmented scaffold FASTA files derived from the complete genomes (4,324 scaffolds, min length 2000 bp for vRhyme)
- Simulated per-sample metagenomic read sets for coverage covariance
- Sorted BAM alignment files of simulated reads against scaffolds

## Outputs

- Sorted BAM files per simulated sample (Bowtie2/BWA alignments)
- Prodigal gene (.gff) and protein (.faa) predictions from '-m -p meta'
- Per-tool bin membership files (e.g., vRhyme_best_bins membership and summary files)
- Benchmark comparison table of recall, precision, accuracy, specificity, and F1 per tool per dataset

## How to apply

Obtain the nine curated benchmark datasets (Supplementary Table S1: 999 complete viral genomes fragmented into 4,324 scaffolds) and simulate the per-sample read sets needed for coverage covariance. Align reads to the fragmented scaffolds with Bowtie2 (run with '--no-unal --no-discordant') or BWA ('mem') to produce sorted BAM files. Predict ORFs with Prodigal using '-m -p meta' to supply gene (-g) and protein (-p) inputs to the binners. Run vRhyme with BAM inputs ('vRhyme -i fasta -b bam_folder/*.bam') at default --iter 20, minimum scaffold length 2000 bp (-l), bin_size 2, and the hybrid neural-network + extra-trees models, and run baselines (MetaBat2, VAMB, CONCOCT, BinSanity, CoCoNet) under equivalent conditions. The rationale is that ground-truth fragmentation plus simulated multi-sample coverage enables exact computation of genome- and bin-level metrics against known reference genomes.

## Related tools

- **vRhyme** (Benchmarked viral binning tool consuming the prepared FASTA, BAM, and protein inputs; collect vRhyme_best_bins membership and summary files) — https://github.com/AnantharamanLab/vRhyme
- **Bowtie2** (Aligns simulated per-sample reads to fragmented scaffolds with '--no-unal --no-discordant' to generate coverage BAMs) — http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- **BWA** (Alternative aligner ('mem') for producing sorted BAM coverage inputs) — https://github.com/lh3/bwa
- **Prodigal** (Predicts ORFs with '-m -p meta' to supply gene/protein inputs to the binners) — https://github.com/hyattpd/Prodigal
- **Samtools** (Sorts alignment output into sorted BAM files and is required for coverage-based vRhyme runs) — http://www.htslib.org/

## Examples

```
bowtie2 -x bt2_idx -1 sample_R1.fq -2 sample_R2.fq --no-unal --no-discordant | samtools sort -o sample.sorted.bam
```

## Evaluation signals

- Reproduced vRhyme row reports F1 of approximately 0.87 averaged over the nine benchmark datasets, exceeding VAMB (0.82) and MetaBat2 (0.81)
- vRhyme achieves highest or equal-to-highest average precision (0.94), accuracy (0.90), and specificity (0.96) across datasets
- Every sample produces a non-empty sorted BAM, and ORF files were generated with Prodigal '-m -p meta' for the 4,324-scaffold inputs
- Per-tool bins score against the known fragmented ground-truth genomes, yielding defined genome and bin scores for all six tools
- test_vRhyme.py prints 'Success' for Python dependencies, both machine learning models, and required program dependencies (Mmseqs; Samtools for coverage inputs; Prodigal if -p/-g skipped)

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- vRhyme is not meant to bin microbes; performance on a whole metagenome has not been fully evaluated, and MetaBat2 outperformed vRhyme on a non-viral synthetic dataset
- vRhyme performs optimally with multiple input samples for more robust coverage variance comparisons, though the optimal number depends on dataset construction
- Only Mmseqs2 is a required dependency for testing; Bowtie2/BWA, Prodigal, Mash, and Nucmer are optional and depend on which coverage, ORF, or dereplication functions are used

## Evidence

- [methods] fragmented ground truth and alignment setup: [evidence span withheld — anchor status: not_found]
- [methods] aligner flags: [evidence span withheld — anchor status: not_found]
- [methods] vRhyme run parameters: [evidence span withheld — anchor status: not_found]
