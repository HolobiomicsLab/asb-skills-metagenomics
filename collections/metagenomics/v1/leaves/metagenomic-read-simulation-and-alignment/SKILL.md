---
name: metagenomic-read-simulation-and-alignment
description: 'Use when benchmarking metagenome binners against curated ground-truth
  genomes: simulated/sampled per-sample reads must be converted into sorted BAM coverage
  files and ORF predictions before binning and F1/precision/specificity scoring.'
when_to_use_negative:
- Input is an already-binned set of vMAGs or a non-simulated natural metagenome where
  ground-truth genome membership is unknown (e.g., GOV2 or agricultural soil viromes),
  since no per-bin accuracy scoring is possible
- The target is binning non-viral (microbial) genomes — the article notes MetaBat2
  performed better than vRhyme on a synthetic dataset of predominately non-viral sequences
  and that this was an anticipated limitation
- Only a single coverage sample is available and robust coverage variance comparisons
  cannot be made, as vRhyme performs optimally with multiple input samples
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0196
tools:
- name: vRhyme
  role: Primary viral binner consuming FASTA plus BAM coverage inputs with default
    --iter 20 and -l 2000bp to produce vRhyme_best_bins membership and summary files
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Bowtie2
  role: Short-read aligner used with '--no-unal --no-discordant' to produce coverage
    BAMs from simulated per-sample reads
  repo: http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- name: BWA
  role: Alternative aligner run as 'bwa mem' to generate coverage BAMs
  repo: https://github.com/lh3/bwa
- name: Prodigal
  role: ORF prediction with '-m -p meta' supplying gene/protein inputs (-g/-p) to
    vRhyme
  repo: https://github.com/hyattpd/Prodigal
- name: Samtools
  role: Required dependency for producing and handling sorted BAM coverage files
  repo: http://www.htslib.org/
- name: Mmseqs2
  role: Required vRhyme dependency used for bin scoring and protein redundancy checks
  repo: https://github.com/soedinglab/MMseqs2
- name: MetaBat2
  role: Baseline binner using tetranucleotide frequency and coverage geometric-mean
    scoring for comparison benchmarking
- name: VAMB
  role: Baseline binner using variational autoencoders on tetranucleotide frequencies
    and coverage
- name: CONCOCT
  role: Baseline binner using Gaussian mixture model clustering
- name: BinSanity
  role: Baseline binner using affinity propagation clustering on coverage abundances
- name: CoCoNet
  role: Virus-specific deep learning baseline binner
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:f39562d37af743da20094c92162ac47f086f7790f9ce1772a630ef2240b1a35a
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
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# Metagenomic read simulation and alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates per-sample metagenomic read sets from curated reference genomes (e.g., the paper's nine benchmark datasets of 999 fragmented complete viral genomes as 4,324 scaffolds), aligns them to scaffolds with Bowtie2 or BWA, and supplies coverage BAMs plus Prodigal gene/protein predictions to binners such as vRhyme for supervised benchmarking of viral genome binning.

## When to use

Use when benchmarking metagenome binners against curated ground-truth genomes: simulated/sampled per-sample reads must be converted into sorted BAM coverage files and ORF predictions before binning and F1/precision/specificity scoring.

## When NOT to use

- Input is an already-binned set of vMAGs or a non-simulated natural metagenome where ground-truth genome membership is unknown (e.g., GOV2 or agricultural soil viromes), since no per-bin accuracy scoring is possible
- The target is binning non-viral (microbial) genomes — the article notes MetaBat2 performed better than vRhyme on a synthetic dataset of predominately non-viral sequences and that this was an anticipated limitation
- Only a single coverage sample is available and robust coverage variance comparisons cannot be made, as vRhyme performs optimally with multiple input samples

## Inputs

- Curated benchmark FASTA of fragmented complete viral genomes (e.g., 999 genomes as 4,324 scaffolds, Supplementary Table S1)
- Simulated/sampled per-sample metagenomic read sets (FASTQ)
- Reference scaffold index for Bowtie2 or BWA
- Prodigal gene/protein prediction inputs (optional -g/-p for vRhyme)

## Outputs

- Sorted per-sample BAM alignment files (coverage input for binning)
- Prodigal ORF predictions (GFF genes and .faa proteins)
- Binner outputs (e.g., vRhyme_best_bins membership and summary files)
- Per-bin benchmark metrics: recall, precision, accuracy, specificity, F1 score

## How to apply

Start from curated benchmark datasets (here, Supplementary Table S1: 999 complete viral genomes fragmented into 4,324 scaffolds) and simulate or sample per-sample read sets so each scaffold has multi-sample coverage covariance. Align reads to the fragmented scaffolds with Bowtie2 (default, with '--no-unal --no-discordant') or BWA ('mem'), then sort/coordinate the output with Samtools into sorted BAM files. Predict ORFs with Prodigal using '-m -p meta' so gene/protein inputs can be supplied to vRhyme via its -g/-p flags. Feed the FASTA plus BAM folder to vRhyme ('vRhyme -i fasta -b bam_folder/*.bam') with default --iter 20 and minimum scaffold length 2000bp (-l), or run baseline binners (MetaBat2, VAMB, CONCOCT, BinSanity, CoCoNet) on equivalent inputs. The rationale is that vRhyme exploits coverage variance/effect size comparisons across samples plus supervised machine learning on sequence features, so realistic multi-sample coverage is essential for fair benchmarking.

## Related tools

- **vRhyme** (Primary viral binner consuming FASTA plus BAM coverage inputs with default --iter 20 and -l 2000bp to produce vRhyme_best_bins membership and summary files) — https://github.com/AnantharamanLab/vRhyme
- **Bowtie2** (Short-read aligner used with '--no-unal --no-discordant' to produce coverage BAMs from simulated per-sample reads) — http://bowtie-bio.sourceforge.net/bowtie2/manual.shtml
- **BWA** (Alternative aligner run as 'bwa mem' to generate coverage BAMs) — https://github.com/lh3/bwa
- **Prodigal** (ORF prediction with '-m -p meta' supplying gene/protein inputs (-g/-p) to vRhyme) — https://github.com/hyattpd/Prodigal
- **Samtools** (Required dependency for producing and handling sorted BAM coverage files) — http://www.htslib.org/
- **Mmseqs2** (Required vRhyme dependency used for bin scoring and protein redundancy checks) — https://github.com/soedinglab/MMseqs2
- **MetaBat2** (Baseline binner using tetranucleotide frequency and coverage geometric-mean scoring for comparison benchmarking)
- **VAMB** (Baseline binner using variational autoencoders on tetranucleotide frequencies and coverage)
- **CONCOCT** (Baseline binner using Gaussian mixture model clustering)
- **BinSanity** (Baseline binner using affinity propagation clustering on coverage abundances)
- **CoCoNet** (Virus-specific deep learning baseline binner)

## Examples

```
vRhyme -i viral_scaffolds.fasta -b bam_folder/*.bam -l 2000 --iter 20 -g genes.gff -p proteins.faa
```

## Evaluation signals

- The reproduced vRhyme row reports an average F1 score of approximately 0.87 across the nine benchmark datasets, exceeding MetaBat2 (0.81) and VAMB (0.82)
- vRhyme yields the highest or equal-to-highest average precision (0.94), accuracy (0.90), and specificity (0.96) against the fragmented ground-truth genomes
- Every scaffold's coverage is represented in sorted BAM files with no unmapped-read contamination (Bowtie2 run with --no-unal --no-discordant)
- Prodigal protein predictions (.faa) exist and vRhyme installation test reports 'Success' for required dependencies (Mmseqs2, Samtools, and Prodigal if -p/-g flags are skipped)
- Per-tool metrics are aggregated across all nine datasets into a single comparison table with genome and bin scores consistent with the published evaluation

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- vRhyme is not meant to bin microbes; on a synthetic dataset of predominately non-viral sequences, MetaBat2 performed better than vRhyme
- The optimal number of input coverage samples depends on dataset construction; performance on a whole metagenome (without prior viral prediction) has not been fully evaluated

## Evidence

- [results] On the nine benchmark datasets, vRhyme achieved the highest F1 score, with an average of 0.87 across all nine datasets, outperforming MetaBat2 (0.81) and VAMB (0.82): [evidence span withheld — anchor status: not_found]
- [methods] Align reads to the fragmented scaffolds with Bowtie2 (default, run with '--no-unal --no-discordant') or BWA ('mem') to produce sorted BAM files, and predict ORFs with Prodigal ('-m -p meta'): [evidence span withheld — anchor status: not_found]
