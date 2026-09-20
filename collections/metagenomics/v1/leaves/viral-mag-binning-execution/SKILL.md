---
name: viral-mag-binning-execution
description: Use when input is predicted viral scaffolds (e.g., from VIBRANT or VirSorter)
  plus per-sample sorted BAM alignments, and the goal is to assemble fragmented viral
  genomes into near-complete vMAGs.
when_to_use_negative:
- The goal is binning non-viral (bacterial/archaeal/eukaryotic) genomes — vRhyme is
  not meant to bin microbes, and MetaBat2 outperformed vRhyme on a synthetic non-viral
  dataset
- Only a single sample is available and coverage variance is unreliable — vRhyme performs
  optimally with multiple input samples
- The target is separating multiple inactive (non-replicating) prophages from the
  same host genome — vRhyme had difficulties with this case
edam_operation: http://edamontology.org/operation_0557
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0196
tools:
- name: vRhyme
  role: 'Core binning tool: supervised ML classification of sequence features plus
    coverage effect size networks to construct vMAGs'
  repo: https://github.com/AnantharamanLab/vRhyme
- name: Bowtie2
  role: Read alignment to produce sorted BAM coverage inputs (with --no-unal --no-discordant)
- name: BWA
  role: Alternative read aligner ('mem') producing BAM coverage inputs
  repo: https://github.com/lh3/bwa
- name: Prodigal
  role: ORF prediction ('-m -p meta') supplying gene/protein features for ML and redundancy
    scoring
  repo: https://github.com/hyattpd/Prodigal
- name: Mmseqs2
  role: Protein redundancy checks during bin scoring and refinement
  repo: https://github.com/soedinglab/MMseqs2
- name: Mash
  role: Scaffold dereplication before binning
  repo: https://github.com/marbl/mash
- name: Samtools
  role: Required dependency for coverage calculation from BAM inputs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac341
    title: vRhyme enables binning of viral genomes from metagenomes
  source_content_hash: sha256:1f6fd0e44459ff462f5dbc2de122c6d693d47564aba336d7501861c2c6870377
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
  - Mash
  - MetaBat2
  - Mmseqs2
  - Prodigal
  - Samtools
  - VAMB
  - vRhyme
  edam_operation: http://edamontology.org/operation_0557
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# viral-mag-binning-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute vRhyme to bin viral scaffolds into viral metagenome-assembled genomes (vMAGs) using supervised machine learning (neural network + extra trees) on nucleotide sequence features combined with single- or multi-sample read coverage effect size comparisons, plus protein-redundancy-based bin refinement.

## When to use

Use when input is predicted viral scaffolds (e.g., from VIBRANT or VirSorter) plus per-sample sorted BAM alignments, and the goal is to assemble fragmented viral genomes into near-complete vMAGs.

## When NOT to use

- The goal is binning non-viral (bacterial/archaeal/eukaryotic) genomes — vRhyme is not meant to bin microbes, and MetaBat2 outperformed vRhyme on a synthetic non-viral dataset
- Only a single sample is available and coverage variance is unreliable — vRhyme performs optimally with multiple input samples
- The target is separating multiple inactive (non-replicating) prophages from the same host genome — vRhyme had difficulties with this case

## Inputs

- FASTA file of predicted viral scaffolds (>= 2000 bp, e.g., VIBRANT/VirSorter output)
- Per-sample sorted BAM files of reads aligned to the scaffolds (Bowtie2 or BWA)
- Optional pre-computed gene calls (-g) or protein sequences (-p) from Prodigal in meta mode

## Outputs

- vRhyme_best_bins membership file assigning scaffolds to vMAG bins
- vRhyme_best_bins summary file with bin statistics
- vMAG bin FASTA files (viral metagenome-assembled genomes)

## How to apply

First predict viruses from the metagenome (e.g., with VIBRANT or VirSorter) and use those predictions as the FASTA input, since vRhyme is built to run on viral sequences. Align per-sample reads to scaffolds with Bowtie2 (with '--no-unal --no-discordant') or BWA 'mem' to produce sorted BAMs, and pre-compute ORFs with Prodigal ('-m -p meta') to supply gene (-g) or protein (-p) inputs. Run vRhyme with 'vRhyme -i fasta -b bam_folder/*.bam' using defaults: --iter 20, minimum scaffold length 2000 bp (-l), bin_size 2, and the hybrid neural network + extra trees models; vRhyme computes coverage variance, dereplicates scaffolds (Mash/Nucmer), scores protein redundancy via Mmseqs2 (exploiting that viruses seldom encode redundant genes), and refines bins with Label Propagation on weighted networks. Collect the vRhyme_best_bins membership and summary files as the final vMAGs. Correctness is judged by bin recall/precision against reference genomes or by near-complete, uncontaminated vMAGs on real data.

## Related tools

- **vRhyme** (Core binning tool: supervised ML classification of sequence features plus coverage effect size networks to construct vMAGs) — https://github.com/AnantharamanLab/vRhyme
- **Bowtie2** (Read alignment to produce sorted BAM coverage inputs (with --no-unal --no-discordant))
- **BWA** (Alternative read aligner ('mem') producing BAM coverage inputs) — https://github.com/lh3/bwa
- **Prodigal** (ORF prediction ('-m -p meta') supplying gene/protein features for ML and redundancy scoring) — https://github.com/hyattpd/Prodigal
- **Mmseqs2** (Protein redundancy checks during bin scoring and refinement) — https://github.com/soedinglab/MMseqs2
- **Mash** (Scaffold dereplication before binning) — https://github.com/marbl/mash
- **Samtools** (Required dependency for coverage calculation from BAM inputs)

## Examples

```
vRhyme -i viral_scaffolds.fasta -b bam_folder/*.bam
```

## Evaluation signals

- On the nine simulated virome benchmark datasets, vRhyme should achieve an average F1 of approximately 0.87, average precision 0.94, accuracy 0.90, and specificity 0.96, outperforming MetaBat2 (F1 0.81) and VAMB (F1 0.82)
- test_vRhyme.py reports 'Success' for all Python dependencies, both machine learning models, and required programs (Mmseqs; Samtools for BAM inputs; Prodigal if -p/-g skipped)
- Each output bin's scaffolds should form connected networks post Label Propagation refinement (v1.1.0 fixed non-connected network generation)
- On real metagenomes (e.g., human skin), resulting vMAGs should be near-complete and low-redundancy, e.g., a 22-scaffold Herelleviridae vMAG identified post-binning

## Limitations

- vRhyme had difficulties separating multiple inactive (non-replicating) prophages from the same host genome and in binning non-viral genomes (the latter anticipated)
- Performance on an entire (unfiltered) metagenome as input has not been fully evaluated; typical workflow is to pre-predict viruses with VIBRANT or VirSorter
- vRhyme performs optimally with multiple input samples for robust coverage variance comparisons, though the optimal number depends on dataset construction
- Whole-metagenome performance and certain robustness benchmarks (fragmented NCLDV, megaphage, crAssphage; GOV2/soil viromes; sample-count and parameter effects) are only in Supplementary Tables S6–S9 not present in the main text

## Evidence

- [methods] Run command with BAM inputs grounded in the benchmark workflow.: [evidence span withheld — anchor status: not_found]
- [abstract] Coverage effect size core method.: [evidence span withheld — anchor status: not_found]
