---
name: virome-assembly-contig-analysis
description: Use when the input is quality-controlled shotgun/VLP virome FASTQ reads
  (e.g., infant gut metagenomes) and the goal is assembled contigs, vOTU counts, per-miner
  overlap comparison, or vOTU-level ORF statistics.
when_to_use_negative:
- Input is already a dereplicated vOTU multifasta or a feature/abundance table rather
  than raw reads or assembled contigs
- The dataset is not a virome (e.g., bulk metagenome with abundant host scaffolds)
  where the --virome sensitivity setting for removing nonviral scaffolds may be inappropriate
- The goal is taxonomic classification/network analysis of known vOTUs via vConTACT2
  rather than vOTU discovery and miner comparison
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaPhage
  role: Nextflow pipeline orchestrating assembly, multi-miner phage detection, dereplication,
    and reporting
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: Megahit
  role: De novo assembly of quality-controlled virome reads into contigs feeding the
    mining step
  repo: https://github.com/voutcn/megahit
- name: VirFinder
  role: k-mer-based machine-learning identification of viral contigs
  repo: https://github.com/jessieren/VirFinder
- name: VirSorter
  role: Homology-based detection of viral hallmark genes in contigs
  repo: https://github.com/simroux/VirSorter
- name: Phigaro
  role: Prophage/viral region detection via pVOG HMMs with smoothing-window algorithm
  repo: https://github.com/bobeobibo/phigaro
- name: VIBRANT
  role: Hybrid annotation-based viral scaffold prediction
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: CD-HIT
  role: cd-hit-est dereplication of merged viral scaffolds into vOTUs at 0.95 identity
    / 0.85 coverage
  repo: https://github.com/weizhongli/cdhit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:284e07928b85ffa90745297d30c3dd40f97c1ddffe63e4f2fd3b83caf3a3d0fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MattiaPandolfoVR/MetaPhage
  tools:
  - CD-HIT
  - Megahit
  - MetaPhage
  - Nextflow
  - Phigaro
  - VIBRANT
  - VirFinder
  - VirSorter
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# virome-assembly-contig-analysis

## Summary

Assemble quality-controlled virome reads into contigs with MEGAHIT and convert them into dereplicated vOTUs using the MetaPhage pipeline's multiple phage miners (VirFinder, VirSorter, Phigaro, VIBRANT) plus CD-HIT-EST clustering, enabling per-miner comparison of viral contig discovery on virome datasets.

## When to use

Use when the input is quality-controlled shotgun/VLP virome FASTQ reads (e.g., infant gut metagenomes) and the goal is assembled contigs, vOTU counts, per-miner overlap comparison, or vOTU-level ORF statistics.

## When NOT to use

- Input is already a dereplicated vOTU multifasta or a feature/abundance table rather than raw reads or assembled contigs
- The dataset is not a virome (e.g., bulk metagenome with abundant host scaffolds) where the --virome sensitivity setting for removing nonviral scaffolds may be inappropriate
- The goal is taxonomic classification/network analysis of known vOTUs via vConTACT2 rather than vOTU discovery and miner comparison

## Inputs

- Quality-controlled paired-end virome FASTQ reads (e.g., Liang et al. 2020 infant shotgun/VLP metagenomes)
- MEGAHIT-assembled metagenomic contigs (FASTA) when starting from the mining step
- MetaPhage databases (Inphared, Kraken2/minikraken, Phigaro, PhiX, VIBRANT, VirSorter)

## Outputs

- Dereplicated vOTU multifasta (final artifact)
- Per-miner vOTU counts and pairwise/all-miner overlap table
- upSet plot of miner comparisons
- Per-vOTU open reading frame counts (e.g., count of vOTUs with >10 ORFs)

## How to apply

Take MEGAHIT-assembled contigs from the preprocessed virome reads as input to the mining step, running the four phage miners in parallel: VirFinder (k-mer machine learning), VirSorter (viral hallmark gene homology), Phigaro (pVOG HMM prophage detection with smoothing window), and VIBRANT (hybrid). Merge predicted viral scaffolds and dereplicate with CD-HIT-EST using a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85, matching the phage-species-level vOTU convention. Set the minimal viral consensus scaffold length with --minlen (use 3000 to mirror the Liang et al. DNA virome reanalysis rather than the 1000 default) and enable the --virome option, which increases sensitivity by removing nonviral scaffolds. Compute per-miner vOTU counts and pairwise/all-miner overlaps, render them as an upSet plot, and optionally count open reading frames per vOTU (e.g., vOTUs with >10 ORFs). Decision point: raising --minlen trades sensitivity for confidence in near-complete viral genomes; the default 1000 bp applies to generic use, while 3000 bp reproduces the published infant virome result of 5,284 vOTUs (2,507 with >10 ORFs).

## Related tools

- **MetaPhage** (Nextflow pipeline orchestrating assembly, multi-miner phage detection, dereplication, and reporting) — https://github.com/MattiaPandolfoVR/MetaPhage
- **Megahit** (De novo assembly of quality-controlled virome reads into contigs feeding the mining step) — https://github.com/voutcn/megahit
- **VirFinder** (k-mer-based machine-learning identification of viral contigs) — https://github.com/jessieren/VirFinder
- **VirSorter** (Homology-based detection of viral hallmark genes in contigs) — https://github.com/simroux/VirSorter
- **Phigaro** (Prophage/viral region detection via pVOG HMMs with smoothing-window algorithm) — https://github.com/bobeobibo/phigaro
- **VIBRANT** (Hybrid annotation-based viral scaffold prediction) — https://github.com/AnantharamanLab/VIBRANT
- **CD-HIT** (cd-hit-est dereplication of merged viral scaffolds into vOTUs at 0.95 identity / 0.85 coverage) — https://github.com/weizhongli/cdhit

## Examples

```
cd-hit-est -i merged_viral_scaffolds.fasta -o votus.fasta -c 0.95 -n 9 -aS 0.85 -d 0 ; nextflow run MattiaPandolfoVR/MetaPhage --input 'reads/*_{1,2}.fastq.gz' --minlen 3000 --virome
```

## Evaluation signals

- Per-miner vOTU counts and overlap distribution show high concordance with published Liang et al. results ('high concordance in the number of vOTUs predicted')
- With --minlen 3000 on the Liang et al. DNA virome dataset, expect 5,284 vOTUs of which 2,507 contain more than 10 ORFs
- cd-hit-est parameters confirm 0.95 identity threshold, word size 9, and 0.85 alignment coverage were applied
- upSet plot of miner overlaps renders without error and covers all four miners (VirFinder, VirSorter, Phigaro, VIBRANT)
- No consensus scaffold shorter than the --minlen threshold remains in the dereplicated vOTU multifasta

## Limitations

- Users should be mindful of biases inherent to virome analyses, including recall/precision trade-offs in virus prediction
- Viral prediction tools disagree; using multiple miners produces partial overlaps and singletons that inflate/deflate per-miner counts
- Using the default --minlen of 1000 bp will yield different vOTU counts than studies (e.g., Liang et al.) that used a 3,000 bp cutoff
- The pipeline is validated primarily on virome datasets; the --virome option's nonviral scaffold removal is tuned for virome sensitivity

## Evidence

- [discussion] CD-HIT-EST dereplication at 0.95 identity, 0.85 coverage: "with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85"
- [intro] vOTU clustering as species-level gold standard: "clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity"
- [methods] --minlen parameter for consensus scaffold length: "Specify the minimal length in bp for a viral _consensus_ scaffold. Default is `1000`."
- [discussion] --virome option for nonviral scaffold removal: "The option –virome increases sensitivity for virome data sets by removing nonviral scaffolds"
- [readme] MetaPhage as a nextflow pipeline for automatic phage discovery: "This is MetaPhage, a nextflow pipeline for automatic phage discovery. 
MetaPhage requires a Linux environment."
