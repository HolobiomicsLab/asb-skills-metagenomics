---
name: phage-taxonomy-composition-analysis
description: Use when shotgun viral metagenomic reads (FASTQ) must be turned into
  viral contigs with assigned taxonomy and family-level composition, e.g., comparing
  phage community structure between conditions such as UC vs healthy fecal viromes.
when_to_use_negative:
- Samples are whole-cell metagenomes without viral fraction isolation — ViroProfiler
  is specifically designed for classifying viruses in samples with isolated viruses,
  and excessive contamination lowers precision.
- Input is already a curated viral contig set with CheckV scores and taxonomic labels
  — only the composition/aggregation step is needed, not the full identification pipeline.
- Viruses are RNA viruses or eukaryotic viruses outside the phage-focused annotation
  scope of ViroProfiler's toolchain (VirSorter2/VIBRANT/DeepVirFinder + vConTACT2).
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: ViroProfiler
  role: Containerized Nextflow pipeline orchestrating trimming, host removal, assembly,
    viral identification, and taxonomy assignment
  repo: github.com/deng-lab/viroprofiler
- name: VirSorter2
  role: Viral contig identification from assembled metagenomes
  repo: https://github.com/jiarong/VirSorter2
- name: DeepVirFinder
  role: Deep-learning viral sequence identification
  repo: https://github.com/jessieren/DeepVirFinder
- name: VIBRANT
  role: Virus identification by iterative annotation; consensus with VirSorter2/DeepVirFinder
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: CheckV
  role: Quality/completeness assessment of identified viral contigs before taxonomy
    assignment
- name: vConTACT2
  role: Network-based taxonomy assignment of viral contigs against reference genomes
- name: MMseqs2
  role: Fast sequence search/clustering supplying taxonomy assignments and bin scoring
  repo: https://github.com/soedinglab/MMseqs2
- name: iPhoP
  role: Host prediction (VIRALHOST_IPHOP process, 12 CPUs, 300 GB) for host-class-level
    composition
- name: metaSPAdes
  role: De novo metagenome assembly (--meta) producing contigs for viral identification
  repo: https://github.com/ablab/spades
- name: FASTP
  role: Adapter/quality trimming of shotgun virome reads
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1080/19490976.2023.2192522
    title: 'ViroProfiler: a containerized bioinformatics pipeline for viral metagenomic
      data analysis'
  source_content_hash: sha256:491497945dc5e9fa215822af7186893b0a9e17b9a86f0981691ebd3ad29b4215
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
  - DeepVirFinder
  - FASTP
  - MMseqs2
  - VIBRANT
  - VirSorter2
  - ViroProfiler
  - iPhoP
  - metaSPAdes
  - vConTACT2
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# phage-taxonomy-composition-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assign taxonomy to viral contigs assembled from shotgun virome data with ViroProfiler's vConTACT2 + MMseqs2 workflow, then compute family-level viral composition (e.g., Podoviridae proportion) across sample groups. This is the core composition-profiling purpose of the containerized ViroProfiler pipeline.

## When to use

Use when shotgun viral metagenomic reads (FASTQ) must be turned into viral contigs with assigned taxonomy and family-level composition, e.g., comparing phage community structure between conditions such as UC vs healthy fecal viromes.

## When NOT to use

- Samples are whole-cell metagenomes without viral fraction isolation — ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses, and excessive contamination lowers precision.
- Input is already a curated viral contig set with CheckV scores and taxonomic labels — only the composition/aggregation step is needed, not the full identification pipeline.
- Viruses are RNA viruses or eukaryotic viruses outside the phage-focused annotation scope of ViroProfiler's toolchain (VirSorter2/VIBRANT/DeepVirFinder + vConTACT2).

## Inputs

- Raw shotgun virome FASTQ reads (e.g., SRA downloads for UC fecal virome samples)
- Sample sheet with metadata (healthy vs UC condition labels)
- Custom Nextflow config for ViroProfiler (-c), including resource blocks like VOGDB with 4 CPUs / 40 GB

## Outputs

- Viral contig FASTA with per-contig taxonomy assignments (vConTACT2 clusters + MMseqs2 taxonomy)
- CheckV quality/completeness table for viral contigs
- Family-level composition table (e.g., Podoviridae proportion per sample group)
- iPHoP host-prediction table and per-group host-class counts (e.g., Bacteroidia-infecting phages)
- Summary table comparing reproduced values against published validation values

## How to apply

Run the Nextflow workflow deng-lab/viroprofiler with a custom config passed via -c: trim adapters/quality with FASTP, remove host reads, and assemble de novo with metaSPAdes (--meta). Identify viral contigs by intersecting VirSorter2, DeepVirFinder, and VIBRANT calls, then assess quality/completeness with CheckV so only adequately complete contigs enter composition analysis. Assign taxonomy with vConTACT2 (network clustering against reference viral genomes) supplemented by MMseqs2 taxonomy, then aggregate assignments to family-level composition — the concrete deliverable being per-group proportions such as Podoviridae in healthy versus UC samples. Optionally run iPHoP (VIRALHOST_IPHOP process, 12 CPUs, 300 GB memory) to predict phage hosts and enrich the composition picture with host-class counts (e.g., Bacteroidia-infecting phages per group). Compile contig counts, family proportions, and host counts into one summary table for direct comparison against published values.

## Related tools

- **ViroProfiler** (Containerized Nextflow pipeline orchestrating trimming, host removal, assembly, viral identification, and taxonomy assignment) — github.com/deng-lab/viroprofiler
- **VirSorter2** (Viral contig identification from assembled metagenomes) — https://github.com/jiarong/VirSorter2
- **DeepVirFinder** (Deep-learning viral sequence identification) — https://github.com/jessieren/DeepVirFinder
- **VIBRANT** (Virus identification by iterative annotation; consensus with VirSorter2/DeepVirFinder) — https://github.com/AnantharamanLab/VIBRANT
- **CheckV** (Quality/completeness assessment of identified viral contigs before taxonomy assignment)
- **vConTACT2** (Network-based taxonomy assignment of viral contigs against reference genomes)
- **MMseqs2** (Fast sequence search/clustering supplying taxonomy assignments and bin scoring) — https://github.com/soedinglab/MMseqs2
- **iPhoP** (Host prediction (VIRALHOST_IPHOP process, 12 CPUs, 300 GB) for host-class-level composition)
- **metaSPAdes** (De novo metagenome assembly (--meta) producing contigs for viral identification) — https://github.com/ablab/spades
- **FASTP** (Adapter/quality trimming of shotgun virome reads)

## Examples

```
nextflow run deng-lab/viroprofiler -profile docker -c viroprofiler.config --input samplesheet.csv
```

## Evaluation signals

- Contig counts fall in the expected range of the validation: ViroProfiler recovers 761 viral contigs vs 183 from the original analysis of the same 20 UC fecal virome samples
- Every contig entering composition analysis has a CheckV quality/completeness estimate
- Family-level proportions (e.g., Podoviridae in healthy vs UC) reproduce the published Figure 3 values when rerun on the same samples
- vConTACT2 clusters are consistent with MMseqs2 taxonomy calls (no family-level contradictions on shared contigs)
- iPHoP host predictions yield coherent host-class counts (e.g., Bacteroidia-infecting phages) that match the reported healthy-vs-UC comparison

## Limitations

- ViroProfiler is specifically designed for classifying viruses in samples with isolated viruses; results degrade on contaminated metagenomes.
- Excessive environmental contaminations in metagenome sequences could increase pipeline running time and result in lower precision.
- It is recommended to isolate the viral fractions before sequencing for an accurate estimation of viruses in the samples.
- Profiling with this multi-tool consensus (VirSorter2 + DeepVirFinder + VIBRANT + vConTACT2 + MMseqs2 + CheckV + iPHoP) is resource-intensive and time-consuming relative to single-tool approaches.

## Evidence

- [readme] MMseqs2 role in the pipeline: [evidence span withheld — anchor status: external_doc]
- [readme] vRhyme role described in its README: [evidence span withheld — anchor status: external_doc]
