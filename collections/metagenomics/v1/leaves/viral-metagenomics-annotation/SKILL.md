---
name: viral-metagenomics-annotation
description: Use when shotgun metagenomic reads or assembled contigs must be classified
  as viral, annotated for taxonomy/function, binned into vMAGs, or assigned hosts
  and lytic/lysogenic/chronic replication cycles.
when_to_use_negative:
- Input is already a feature table or isolate virus genomes rather than shotgun metagenomic
  data — ViroProfiler 'is specifically designed for classifying viruses in samples
  with isolated viruses' and expects metagenomic input
- You only need bacterial community profiling (e.g., Qiime/Mothur amplicon workflows),
  not viral discovery and annotation
- Heavy environmental contamination without prior viral-fraction isolation is expected,
  which the authors warn increases runtime and lowers precision
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0196
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline orchestrating all viral identification, annotation,
    host and replication-cycle prediction steps
  repo: github.com/deng-lab/viroprofiler
- name: metaSPAdes
  role: Assembles shotgun metagenomic reads into contigs for viral discovery
  repo: https://github.com/ablab/spades
- name: vRhyme
  role: Bins viral contigs into vMAGs using coverage variance and supervised machine
    learning
  repo: https://github.com/AnantharamanLab/vRhyme
- name: VirSorter2
  role: Identifies viral contigs in the assembled metagenome
  repo: https://github.com/jiarong/VirSorter2
- name: DeepVirFinder
  role: Deep-learning-based identification of viral sequences
  repo: https://github.com/jessieren/DeepVirFinder
- name: VIBRANT
  role: Iterative annotation-based virus identification
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: DRAM-v
  role: Gene function annotation of viral genes
  repo: https://github.com/WrightonLabCSU/DRAM
- name: abricate
  role: AMR gene screening within the pipeline
  repo: https://github.com/tseemann/abricate
- name: iPhoP
  role: Host prediction for viral contigs (process VIRALHOST_IPHOP)
- name: BACPHLIP
  role: Scores lytic/lysogenic lifestyle using --multi_fasta on predicted viral contigs
  repo: https://github.com/adamhockenberry/bacphlip
- name: Replidec
  role: Alternative replication-cycle detector classifying contigs as lytic, lysogenic,
    or chronic
  repo: https://github.com/deng-lab/Replidec
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:5fc08177feda5eddf06750e75c1aa44fbea12a0a57c97c50c43ae32d09dd5f32
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  tools:
  - BACPHLIP
  - DRAM-v
  - DeepVirFinder
  - Replidec
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - abricate
  - iPhoP
  - metaSPAdes
  - vRhyme
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# viral-metagenomics-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate shotgun viral metagenomic data end-to-end with ViroProfiler: assemble reads, identify viral contigs (VirSorter2/VIBRANT/DeepVirFinder), bin viral genomes (vRhyme/phamb), assign taxonomy and gene function, and predict hosts and replication cycles (iPhoP, BACPHLIP, Replidec).

## When to use

Use when shotgun metagenomic reads or assembled contigs must be classified as viral, annotated for taxonomy/function, binned into vMAGs, or assigned hosts and lytic/lysogenic/chronic replication cycles.

## When NOT to use

- Input is already a feature table or isolate virus genomes rather than shotgun metagenomic data — ViroProfiler 'is specifically designed for classifying viruses in samples with isolated viruses' and expects metagenomic input
- You only need bacterial community profiling (e.g., Qiime/Mothur amplicon workflows), not viral discovery and annotation
- Heavy environmental contamination without prior viral-fraction isolation is expected, which the authors warn increases runtime and lowers precision

## Inputs

- Shotgun metagenomic reads (FASTQ)
- Assembled contigs/scaffolds from metaSPAdes
- Predicted viral contigs (multi-FASTA) from VirSorter2/VIBRANT/DeepVirFinder
- Reference databases: VOGDB, EggNOG5

## Outputs

- Classified viral contig set with taxonomy assignments
- Viral genome bins (vMAGs) from vRhyme/phamb
- Gene functional annotations (DRAM-v) and AMG/AMR calls
- Per-contig table of host predictions (iPhoP) and replication-cycle calls (BACPHLIP + Replidec)

## How to apply

Run ViroProfiler, a Nextflow DSL2 (≥21.10.3) containerized workflow, starting from shotgun metagenomic FASTQ: assemble with metaSPAdes, optionally bin viral genomes with vRhyme or phamb, then identify viral contigs with VirSorter2, VIBRANT, and DeepVirFinder. Annotate gene function with DRAM-v and AMR genes with abricate, using databases such as VOGDB (4 CPUs, 40 GB in custom.config) and EggNOG5. For lifestyle/host annotation, run iPhoP (12 CPUs, 300 GB) for host prediction, BACPHLIP with --multi_fasta (4 CPUs, 80 GB) for lytic/lysogenic scoring, and Replidec (4 CPUs, 80 GB) for lytic/lysogenic/chronic classification, reconciling both calls per contig with iPhoP hosts into one per-contig table. Containerization (Docker) is used deliberately so results are computationally reproducible and shareable across collaborators.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline orchestrating all viral identification, annotation, host and replication-cycle prediction steps) — github.com/deng-lab/viroprofiler
- **metaSPAdes** (Assembles shotgun metagenomic reads into contigs for viral discovery) — https://github.com/ablab/spades
- **vRhyme** (Bins viral contigs into vMAGs using coverage variance and supervised machine learning) — https://github.com/AnantharamanLab/vRhyme
- **VirSorter2** (Identifies viral contigs in the assembled metagenome) — https://github.com/jiarong/VirSorter2
- **DeepVirFinder** (Deep-learning-based identification of viral sequences) — https://github.com/jessieren/DeepVirFinder
- **VIBRANT** (Iterative annotation-based virus identification) — https://github.com/AnantharamanLab/VIBRANT
- **DRAM-v** (Gene function annotation of viral genes) — https://github.com/WrightonLabCSU/DRAM
- **abricate** (AMR gene screening within the pipeline) — https://github.com/tseemann/abricate
- **iPhoP** (Host prediction for viral contigs (process VIRALHOST_IPHOP))
- **BACPHLIP** (Scores lytic/lysogenic lifestyle using --multi_fasta on predicted viral contigs) — https://github.com/adamhockenberry/bacphlip
- **Replidec** (Alternative replication-cycle detector classifying contigs as lytic, lysogenic, or chronic) — https://github.com/deng-lab/Replidec

## Examples

```
nextflow run deng-lab/viroprofiler -profile docker
```

## Evaluation signals

- All pipeline processes (VIRALHOST_IPHOP, BACPHLIP, REPLIDEC and upstream viral-discovery steps) complete without error
- The host/replication-cycle output table contains exactly one row per input viral contig with populated host and replication-cycle columns
- BACPHLIP and Replidec calls reconcile per contig (lytic/lysogenic/chronic) and merge cleanly with iPhoP host assignments
- Pipeline is executed via containerized Nextflow (Docker profile) so a re-run on the same inputs reproduces identical outputs
- Mock-dataset benchmark output is comparable to existing virus classification tools, as used by the authors for validation

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; whole-metagenome runs with excess contamination increase runtime and reduce precision
- Isolating the viral fraction before sequencing is recommended for accurate virus estimation
- The discussion names mock-dataset benchmarking but does not identify specific benchmark datasets, comparison tool settings, or published validation datasets, limiting reproducibility verification

## Evidence

- [readme] vRhyme input expectation: [evidence span withheld — anchor status: external_doc]
