---
name: viral-metagenomic-taxonomic-profiling
description: Use when you have shotgun Illumina paired-end virome FASTQ data and need
  viral contig identification, taxonomic assignment and abundance profiling, or when
  benchmarking a profiler against ground-truth mock community composition.
when_to_use_negative:
- Samples contain excessive environmental contamination rather than isolated viral
  fractions — the paper notes contamination increases runtime and lowers precision,
  so isolate viral fractions before sequencing
- Input is already assembled viral contigs or a taxonomic abundance table — assembly/binning
  modules would be redundant
- The goal is profiling bacteria or mixed microbiomes without viral enrichment — ViroProfiler
  is specifically designed for classifying viruses in samples with isolated viruses
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0781
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline orchestrating assembly, binning, viral identification,
    taxonomy and functional annotation
  repo: https://github.com/deng-lab/viroprofiler
- name: metaSPAdes
  role: Genome assembly module within ViroProfiler
  repo: https://github.com/ablab/spades
- name: vRhyme
  role: Binning of viral genomes into vMAGs within ViroProfiler
  repo: https://github.com/AnantharamanLab/vRhyme
- name: phamb
  role: Alternative binning module within ViroProfiler
  repo: https://github.com/RasmussenLab/phamb
- name: VirSorter2
  role: Viral contig identification within ViroProfiler
  repo: https://github.com/jiarong/VirSorter2
- name: DeepVirFinder
  role: Deep-learning viral contig identification within ViroProfiler
  repo: https://github.com/jessieren/DeepVirFinder
- name: VIBRANT
  role: Viral contig identification within ViroProfiler
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: DRAM-v
  role: Gene function annotation of viral sequences
  repo: https://github.com/WrightonLabCSU/DRAM
- name: abricate
  role: AMG/antimicrobial resistance screening within ViroProfiler
  repo: https://github.com/tseemann/abricate
- name: Kraken2/Bracken
  role: Comparator taxonomic profiler run with BrackenSTD and BrackenVPF databases
    for benchmarking
- name: vpfkit
  role: R toolkit converting ViroProfiler output for downstream analysis
  repo: https://github.com/deng-lab/vpfkit
- name: CoverM
  role: Read coverage/abundance calculation
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:0314472ddb03acdb1228022c8e9486deb4002c6bae6d1a6b827065ff3371d19b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  repo_url: https://github.com/deng-lab/viroprofiler
  tools:
  - CoverM
  - DRAM-v
  - DeepVirFinder
  - Kraken2/Bracken
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - abricate
  - metaSPAdes
  - phamb
  - vRhyme
  - vpfkit
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# viral-metagenomic-taxonomic-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end containerized profiling of shotgun virome samples with the ViroProfiler Nextflow pipeline (metaSPAdes assembly, vRhyme/phamb binning, VirSorter2/DeepVirFinder/VIBRANT viral-contig identification, taxonomy and functional annotation), including benchmarking against Kraken2/Bracken with precision, sensitivity and F1 metrics.

## When to use

Use when you have shotgun Illumina paired-end virome FASTQ data and need viral contig identification, taxonomic assignment and abundance profiling, or when benchmarking a profiler against ground-truth mock community composition.

## When NOT to use

- Samples contain excessive environmental contamination rather than isolated viral fractions — the paper notes contamination increases runtime and lowers precision, so isolate viral fractions before sequencing
- Input is already assembled viral contigs or a taxonomic abundance table — assembly/binning modules would be redundant
- The goal is profiling bacteria or mixed microbiomes without viral enrichment — ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses

## Inputs

- Illumina paired-end shotgun metagenomic FASTQ files (e.g., 14 simulated mock virome samples, 13 usable)
- Nextflow config file for compute/container profile
- Kraken2/Bracken reference databases (BrackenSTD standard and BrackenVPF viral-focused)
- Ground-truth taxonomic composition table of the mock community (RefSeq v69-based)

## Outputs

- Assembled metagenomic scaffolds (metaSPAdes)
- Viral metagenome-assembled genome bins (vRhyme/phamb vMAGs)
- Viral contig identifications (VirSorter2/DeepVirFinder/VIBRANT)
- Viral taxonomic profiles and abundance tables per sample at each taxonomic rank
- Functional annotations (DRAM-v, VOGDB, EggNOG5, abricate)
- Benchmark table of precision, sensitivity and F1 per tool, rank and abundance threshold

## How to apply

Run 'nextflow run deng-lab/viroprofiler' with a docker or singularity profile and a custom config passed via '-c' on each input FASTQ pair; the pipeline chains genome assembly (metaSPAdes), binning (vRhyme or phamb), viral-contig identification (VirSorter2, DeepVirFinder, VIBRANT) and taxonomy/functional annotation (DRAM-v, abricate, VOGDB, EggNOG5). For benchmarking, run Kraken2/Bracken on the same samples in two database configurations (BrackenSTD standard and BrackenVPF viral-focused, the latter derived from ViroProfiler's database), then match predicted viral taxa and abundances against known mock-sample ground truth at each taxonomic rank. Sweep abundance thresholds and compute precision = TP/(TP+FP), sensitivity = TP/(TP+FN), and F1 per tool and rank; on the 13 usable simulated Illumina mock virome samples (RefSeq v69-based), expect ViroProfiler to exceed 95% precision and sensitivity at phylum level while BrackenVPF reaches only ~50% sensitivity. Containerization via docker/singularity is the reproducibility mechanism, so profiles should be pinned rather than run bare. Downstream R analysis of ViroProfiler output can use the deng-lab/vpfkit toolkit.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline orchestrating assembly, binning, viral identification, taxonomy and functional annotation) — https://github.com/deng-lab/viroprofiler
- **metaSPAdes** (Genome assembly module within ViroProfiler) — https://github.com/ablab/spades
- **vRhyme** (Binning of viral genomes into vMAGs within ViroProfiler) — https://github.com/AnantharamanLab/vRhyme
- **phamb** (Alternative binning module within ViroProfiler) — https://github.com/RasmussenLab/phamb
- **VirSorter2** (Viral contig identification within ViroProfiler) — https://github.com/jiarong/VirSorter2
- **DeepVirFinder** (Deep-learning viral contig identification within ViroProfiler) — https://github.com/jessieren/DeepVirFinder
- **VIBRANT** (Viral contig identification within ViroProfiler) — https://github.com/AnantharamanLab/VIBRANT
- **DRAM-v** (Gene function annotation of viral sequences) — https://github.com/WrightonLabCSU/DRAM
- **abricate** (AMG/antimicrobial resistance screening within ViroProfiler) — https://github.com/tseemann/abricate
- **Kraken2/Bracken** (Comparator taxonomic profiler run with BrackenSTD and BrackenVPF databases for benchmarking)
- **vpfkit** (R toolkit converting ViroProfiler output for downstream analysis) — https://github.com/deng-lab/vpfkit
- **CoverM** (Read coverage/abundance calculation) — https://github.com/wwood/CoverM

## Examples

```
nextflow run deng-lab/viroprofiler --input 'samples/*_{1,2}.fq.gz' -profile docker -c viroprofiler.config
```

## Evaluation signals

- At phylum level, ViroProfiler sensitivity and precision both exceed 95% on the mock virome benchmark, matching Figure 2a
- BrackenVPF sensitivity lands around 50% at phylum level, confirming the reference configuration was reproduced correctly
- All 13 usable mock samples (of 14 simulated RefSeq v69-based samples) yield complete output through assembly, binning, and taxonomy modules
- Benchmark table contains a precision/sensitivity/F1 cell for every combination of tool (ViroProfiler, BrackenSTD, BrackenVPF), taxonomic rank, and abundance threshold, and metrics are internally consistent (precision, sensitivity ≤ 1; F1 is the harmonic mean)
- Pipeline runs reproducibly under a pinned docker or singularity profile via the '-c' Nextflow config

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; unenriched metagenomes may underperform
- Excessive environmental contaminations, usually found in metagenome sequences, could increase the running time of the pipeline and result in lower precision
- BrackenVPF (Kraken2/Bracken with the ViroProfiler-derived viral database) achieves only ~50% sensitivity at phylum level, so Kraken2/Bracken is a poor substitute for viral profiling
- Benchmarking detail (datasets, comparison tools, validation data) is not fully enumerated in the article discussion, limiting independent verification

## Evidence

- [results] Benchmark performance claim: [evidence span withheld — anchor status: not_found]
