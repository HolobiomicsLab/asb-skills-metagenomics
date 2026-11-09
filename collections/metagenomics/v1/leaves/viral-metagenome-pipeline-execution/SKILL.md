---
name: viral-metagenome-pipeline-execution
description: Use when analyzing raw shotgun virome reads (e.g., FASTQ from SRA) to
  identify viral contigs, assess their quality, assign taxonomy, and predict phage
  hosts — as in the UC fecal virome validation with 20 samples.
when_to_use_negative:
- Input is already assembled contigs or a feature/abundance table rather than raw
  shotgun reads — the trimming/DECONTAM/metaSPAdes front end would be redundant or
  invalid
- Samples contain isolated (purified) viral fractions only — the article notes ViroProfiler
  is designed for classifying viruses in samples with isolated viruses, but whole
  shotgun metagenomes with excessive environmental contamination increase run time
  and lower precision
- The goal is bacterial (non-viral) community profiling — standardized tools like
  Qiime/Mothur are the article's reference point for bacteria, not this virome pipeline
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: ViroProfiler
  role: Containerized Nextflow (DSL2) pipeline orchestrating the entire viral metagenome
    analysis workflow
  repo: github.com/deng-lab/viroprofiler
- name: FASTP
  role: Adapter and quality trimming of raw shotgun virome reads before host removal
    and assembly
- name: metaSPAdes
  role: De novo metagenomic assembly of quality-trimmed, host-depleted reads (--meta
    mode)
  repo: https://github.com/ablab/spades
- name: VirSorter2
  role: Viral contig identification from assembled scaffolds
  repo: https://github.com/jiarong/VirSorter2
- name: DeepVirFinder
  role: Deep-learning-based viral contig identification
  repo: https://github.com/jessieren/DeepVirFinder
- name: VIBRANT
  role: Viral contig identification and annotation
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: CheckV
  role: Quality and completeness assessment of identified viral contigs
- name: vConTACT2
  role: Viral taxonomy assignment / clustering of viral contigs
- name: MMseqs2
  role: Taxonomy assignment to viral contigs for family-level composition
  repo: https://github.com/soedinglab/MMseqs2
- name: iPHoP
  role: Phage host prediction (VIRALHOST_IPHOP process, 12 CPUs, 300 GB) to count
    condition-specific host-infecting phages
- name: DRAM-v
  role: Gene function annotation of viral genomes
  repo: https://github.com/WrightonLabCSU/DRAM
- name: vRhyme
  role: Binning of viral genomes (vMAGs) from metagenomes within the pipeline
  repo: https://github.com/AnantharamanLab/vRhyme
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:43ffb564644417ebaec42c7b8f886b25766918ccaeeafa8dd92ef05dfb77a3f2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deng-lab/viroprofiler
  tools:
  - CheckV
  - DRAM-v
  - DeepVirFinder
  - FASTP
  - MMseqs2
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - iPHoP
  - iPhoP
  - metaSPAdes
  - vConTACT2
  - vRhyme
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# viral-metagenome-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute the ViroProfiler containerized Nextflow pipeline (deng-lab/viroprofiler) on shotgun viral metagenomic data, chaining quality trimming (FASTP), host removal (DECONTAM), assembly (metaSPAdes --meta), viral identification (VirSorter2, DeepVirFinder, VIBRANT), quality assessment (CheckV), taxonomy assignment (vConTACT2, MMseqs2), and host prediction (iPHoP) to produce annotated viral contigs and community profiles.

## When to use

Use when analyzing raw shotgun virome reads (e.g., FASTQ from SRA) to identify viral contigs, assess their quality, assign taxonomy, and predict phage hosts — as in the UC fecal virome validation with 20 samples.

## When NOT to use

- Input is already assembled contigs or a feature/abundance table rather than raw shotgun reads — the trimming/DECONTAM/metaSPAdes front end would be redundant or invalid
- Samples contain isolated (purified) viral fractions only — the article notes ViroProfiler is designed for classifying viruses in samples with isolated viruses, but whole shotgun metagenomes with excessive environmental contamination increase run time and lower precision
- The goal is bacterial (non-viral) community profiling — standardized tools like Qiime/Mothur are the article's reference point for bacteria, not this virome pipeline

## Inputs

- Raw shotgun virome reads in FASTQ format (e.g., downloaded from the SRA archive of a published fecal virome study)
- Sample metadata sheet mapping samples to conditions (e.g., UC patients vs healthy individuals)
- Nextflow config file for resource allocation (e.g., -c custom.config with per-process CPU/memory settings)

## Outputs

- Set of identified viral contigs with per-tool identifications from VirSorter2, DeepVirFinder, and VIBRANT
- CheckV quality/completeness assessments for each viral contig
- Taxonomic assignments per contig (vConTACT2 clusters and MMseqs2 taxonomy) and family-level composition table
- iPHoP host predictions per viral contig (e.g., counts of Bacteroidia-infecting phages per condition)
- Compiled summary table of viral contig counts, family proportions, and host-prediction counts for comparison against published values

## How to apply

Download raw shotgun virome reads (e.g., from SRA) and run the Nextflow pipeline 'deng-lab/viroprofiler' with a custom config passed via -c, performing adapter/quality trimming with FASTP, host read removal with DECONTAM, and de novo assembly with metaSPAdes (--meta). Identify viral contigs with VirSorter2, DeepVirFinder, and VIBRANT, then assess quality/completeness with CheckV. Assign taxonomy with vConTACT2 and MMseqs2 taxonomy to compute family-level composition (e.g., Podoviridae proportions across conditions). Predict phage hosts with iPHoP (VIRALHOST_IPHOP process, 12 CPUs, 300 GB) and compile all reproduced values (viral contig count, family proportions, host-assigned phage counts) into a summary table. In the UC fecal virome validation this workflow identified 761 viral contigs versus 183 from the original published analysis, demonstrating improved viral discovery.

## Related tools

- **ViroProfiler** (Containerized Nextflow (DSL2) pipeline orchestrating the entire viral metagenome analysis workflow) — github.com/deng-lab/viroprofiler
- **FASTP** (Adapter and quality trimming of raw shotgun virome reads before host removal and assembly)
- **metaSPAdes** (De novo metagenomic assembly of quality-trimmed, host-depleted reads (--meta mode)) — https://github.com/ablab/spades
- **VirSorter2** (Viral contig identification from assembled scaffolds) — https://github.com/jiarong/VirSorter2
- **DeepVirFinder** (Deep-learning-based viral contig identification) — https://github.com/jessieren/DeepVirFinder
- **VIBRANT** (Viral contig identification and annotation) — https://github.com/AnantharamanLab/VIBRANT
- **CheckV** (Quality and completeness assessment of identified viral contigs)
- **vConTACT2** (Viral taxonomy assignment / clustering of viral contigs)
- **MMseqs2** (Taxonomy assignment to viral contigs for family-level composition) — https://github.com/soedinglab/MMseqs2
- **iPHoP** (Phage host prediction (VIRALHOST_IPHOP process, 12 CPUs, 300 GB) to count condition-specific host-infecting phages)
- **DRAM-v** (Gene function annotation of viral genomes) — https://github.com/WrightonLabCSU/DRAM
- **vRhyme** (Binning of viral genomes (vMAGs) from metagenomes within the pipeline) — https://github.com/AnantharamanLab/vRhyme

## Examples

```
nextflow run deng-lab/viroprofiler -c custom.config --input samplesheet.csv --outdir results -with-docker
```

## Evaluation signals

- Viral discovery rate exceeds the original published analysis: the UC validation identified 761 viral contigs with ViroProfiler vs only 183 in the original study on the same 20 fecal virome samples
- All three viral identification tools (VirSorter2, DeepVirFinder, VIBRANT) produce output on the assembled contigs and their union is consistent with CheckV quality assessments
- Family-level composition (e.g., Podoviridae proportions in healthy vs UC samples) matches the reported validation values and Figure 3
- iPHoP host-prediction counts (e.g., Bacteroidia-infecting phages in UC vs healthy) reproduce the published validation numbers
- Pipeline completes reproducibly under containerization (Docker) with the supplied Nextflow config, honoring per-process resource limits such as VIRALHOST_IPHOP at 12 CPUs / 300 GB

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses
- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision; isolating viral fractions before sequencing is recommended
- Some viral-discovery tools are highly difficult to install for inexperienced users, which containerization mitigates but does not eliminate for downstream interpretation
- No benchmark datasets, comparison tools, or published datasets used for validation are named in the discussion section, complicating third-party reproduction checks

## Evidence

- [other] iPHoP host prediction resources in workflow: [evidence span withheld — anchor status: not_found]
