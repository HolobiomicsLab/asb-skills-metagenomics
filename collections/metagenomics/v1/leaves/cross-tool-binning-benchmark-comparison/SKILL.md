---
name: cross-tool-binning-benchmark-comparison
description: Use when multiple viral binning tools must be compared on identical scaffold
  inputs with sorted BAM coverage, and a reproducible metrics table (F1, precision,
  redundant-protein counts) is the deliverable.
when_to_use_negative:
- The input is a whole (non-viral) metagenome or predominantly non-viral sequences
  — vRhyme is not meant to bin microbes, and on a synthetic non-viral dataset MetaBat2
  performed better than vRhyme
- Scaffolds are already binned, or the input is a bin/feature table rather than scaffold
  FASTA plus per-sample coverage BAMs
- Only a single coverage sample is available and dataset construction does not support
  coverage variance comparisons — vRhyme performs optimally with multiple input samples
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: vRhyme
  role: 'Virus-centric binner under test: supervised ML (NN + Extra Trees) on sequence
    features plus coverage effect-size networks with protein redundancy scoring'
  repo: https://github.com/AnantharamanLab/vRhyme
- name: MetaBat2
  role: Baseline binner using composite scoring of tetranucleotide frequencies and
    coverage abundance; run on jgi_summarize_bam_contig_depths coverage tables
- name: VAMB
  role: Baseline binner implementing unsupervised variational autoencoders on tetranucleotide
    frequencies and coverage
- name: CONCOCT
  role: Baseline binner clustering scaffolds with Gaussian mixture models on reduced
    tetranucleotide/coverage features
- name: BinSanity
  role: Baseline binner using affinity propagation clustering on coverage abundances
- name: CoCoNet
  role: Baseline virus-specific deep-learning binner for viral genomes from metagenomes
- name: Bowtie2
  role: Short-read aligner producing sorted BAMs (run with --no-unal --no-discordant)
    for coverage calculation
- name: BWA
  role: Alternative aligner ('mem') for producing BAM coverage inputs
  repo: https://github.com/lh3/bwa
- name: Prodigal
  role: ORF prediction on scaffolds (-m -p meta) supplying gene/protein inputs for
    binning and redundancy scoring
  repo: https://github.com/hyattpd/Prodigal
- name: Mmseqs2
  role: Protein redundancy scoring used by vRhyme for bin refinement and contamination
    assessment
  repo: https://github.com/soedinglab/MMseqs2
- name: Python
  role: Implementation language for vRhyme and for aggregating per-tool metrics into
    the benchmark summary table
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:db684f9ff6ea364448f68f54755ba7ee84f7443ad23bf40bcebda69599c9b71e
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
  - Python
  - Samtools
  - VAMB
  - vRhyme
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cross-tool-binning-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark viral metagenome binning tools (vRhyme, MetaBat2, VAMB, CONCOCT, BinSanity, CoCoNet) on the same scaffold sets under equivalent conditions, scoring bins against ground-truth genomes (simulated viromes) or against protein-redundancy/CheckV metrics (natural datasets like GOV2) to rank binners by precision and completeness.

## When to use

Use when multiple viral binning tools must be compared on identical scaffold inputs with sorted BAM coverage, and a reproducible metrics table (F1, precision, redundant-protein counts) is the deliverable.

## When NOT to use

- The input is a whole (non-viral) metagenome or predominantly non-viral sequences — vRhyme is not meant to bin microbes, and on a synthetic non-viral dataset MetaBat2 performed better than vRhyme
- Scaffolds are already binned, or the input is a bin/feature table rather than scaffold FASTA plus per-sample coverage BAMs
- Only a single coverage sample is available and dataset construction does not support coverage variance comparisons — vRhyme performs optimally with multiple input samples

## Inputs

- Viral scaffold FASTA files (e.g., GOV2 scaffolds pre-filtered to ≥5 kb and 98% dereplicated; simulated viromes of 4,324 scaffolds from 999 fragmented complete genomes)
- Sorted BAM files of reads aligned to scaffolds (Bowtie2 or BWA)
- Prodigal-predicted ORF/protein files (-g/-p inputs)
- Curated ground-truth reference genomes for simulated benchmarks (Supplementary Table S1)

## Outputs

- Per-tool bin membership and summary files (e.g., vRhyme_best_bins.#.membership.tsv, vRhyme_best_bins.#.summary.tsv, vRhyme_bin_scoring.tsv)
- Per-tool metrics: recall, precision, accuracy, specificity, F1 score, genome and bin scores
- Redundant-protein totals per tool and low-contamination bin counts (e.g., 11,909 low-contamination bins on GOV2)
- Aggregated cross-tool comparison table (e.g., gov2_vrhyme_benchmark_summary.csv)

## How to apply

Prepare a common input: viral scaffolds (e.g., ≥5 kb, 98% dereplicated for GOV2), sorted BAM files from Bowtie2 ('--no-unal --no-discordant') or BWA 'mem' alignments, and Prodigal ORFs ('-m -p meta'). Run vRhyme with default settings (--iter 20, --bin_size 2, hybrid NN + Extra Trees model, --cov 0.80, dynamic --max_edges 3-6, coverage outlier removal --outliers 4.0), collecting vRhyme_best_bins membership/summary TSVs and vRhyme_bin_scoring.tsv. Run baselines under comparable conditions: MetaBat2 with its 'jgi_summarize_bam_contig_depths' coverage table (geometric mean of tetranucleotide frequencies and coverage), VAMB (variational autoencoder), CONCOCT (Gaussian mixture clustering), BinSanity (affinity propagation), and CoCoNet (virus-specific deep learning). For simulated benchmarks, score bins against fragmented ground-truth genomes (999 complete viral genomes fragmented into 4,324 scaffolds across nine datasets) computing recall, precision, accuracy, specificity, and F1; for natural datasets like GOV2, count total redundant proteins per tool (vRhyme exploits that viruses seldom encode redundant genes) and classify bins with 0-1 redundant proteins as low-contamination, plus CheckV completeness/contamination. Aggregate all per-tool metrics into a single comparison CSV and validate against published values (e.g., vRhyme F1 ≈ 0.87 vs MetaBat2 0.81 and VAMB 0.82; GOV2: 15,106 vRhyme redundant proteins vs 29,334 MetaBat2 and 71,364 CoCoNet).

## Related tools

- **vRhyme** (Virus-centric binner under test: supervised ML (NN + Extra Trees) on sequence features plus coverage effect-size networks with protein redundancy scoring) — https://github.com/AnantharamanLab/vRhyme
- **MetaBat2** (Baseline binner using composite scoring of tetranucleotide frequencies and coverage abundance; run on jgi_summarize_bam_contig_depths coverage tables)
- **VAMB** (Baseline binner implementing unsupervised variational autoencoders on tetranucleotide frequencies and coverage)
- **CONCOCT** (Baseline binner clustering scaffolds with Gaussian mixture models on reduced tetranucleotide/coverage features)
- **BinSanity** (Baseline binner using affinity propagation clustering on coverage abundances)
- **CoCoNet** (Baseline virus-specific deep-learning binner for viral genomes from metagenomes)
- **Bowtie2** (Short-read aligner producing sorted BAMs (run with --no-unal --no-discordant) for coverage calculation)
- **BWA** (Alternative aligner ('mem') for producing BAM coverage inputs) — https://github.com/lh3/bwa
- **Prodigal** (ORF prediction on scaffolds (-m -p meta) supplying gene/protein inputs for binning and redundancy scoring) — https://github.com/hyattpd/Prodigal
- **Mmseqs2** (Protein redundancy scoring used by vRhyme for bin refinement and contamination assessment) — https://github.com/soedinglab/MMseqs2
- **Python** (Implementation language for vRhyme and for aggregating per-tool metrics into the benchmark summary table)

## Examples

```
vRhyme -i gov2_scaffolds.fasta -b bam_folder/*.bam -o gov2_vrhyme_out -t 15
```

## Evaluation signals

- Reproduced vRhyme row on the nine simulated datasets reports average F1 ≈ 0.87, exceeding MetaBat2 (0.81) and VAMB (0.82), with precision 0.94, accuracy 0.90, specificity 0.96
- GOV2 reproduction matches Supplementary Table S7: 15,106 vRhyme redundant proteins vs 29,334 (MetaBat2) and 71,364 (CoCoNet), and 11,909 low-contamination bins
- vRhyme_best_bins membership and summary TSV files exist in the output hierarchy and scaffold counts sum to the input scaffold count (e.g., 56,642 GOV2 scaffolds into 13,175 bins)
- Per-bin contamination classification is internally consistent: bins with 0-1 redundant proteins are labeled low-contamination and are cross-checked with CheckV completeness/contamination
- All six binners ran on the same scaffold/BAM/ORF inputs with documented, comparable parameters recorded in the aggregated CSV

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome
- Binning non-viral genomes is an anticipated limitation; on a synthetic non-viral dataset MetaBat2 performed better than vRhyme
- Performance for a whole (unfiltered) metagenome input has not been fully evaluated; vRhyme is built to run on pre-predicted viral scaffolds
- vRhyme performs optimally with multiple input samples for robust coverage variance comparisons; the optimal number depends on how the dataset was constructed

## Evidence
