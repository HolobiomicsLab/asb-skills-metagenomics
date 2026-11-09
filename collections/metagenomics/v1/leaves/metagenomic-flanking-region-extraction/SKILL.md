---
name: metagenomic-flanking-region-extraction
description: Use when short-read metagenomic FASTQ data must yield the genomic context
  (flanking regions) around ARGs or other target genes, and full de novo assembly
  would be computationally prohibitive.
when_to_use_negative:
- Input data are long reads (Nanopore/PacBio) — ARGprofiler is designed to only work
  with short-read sequencing data
- You need whole-genome/metagenome-wide assemblies rather than context around specific
  target genes — run full SPAdes de novo assembly instead
- No reference database exists for the target genes — ARGextender requires KMA identification
  of target genes before recruiting reads and skipping assembly otherwise
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_3305
tools:
- name: ARGextender
  role: 'Targeted de novo assembly tool: uses KMA to identify target ARGs and recruit
    reads, then SPAdes to extend the flanking regions around genes of interest'
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: KMA
  role: Recursive k-mer-based mapping to identify target ARGs in reads, recruit reads
    to surrounding regions, and match reference gene sequences to resulting scaffolds
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: SPAdes
  role: De novo assembler run on KMA-recruited reads to build targeted scaffolds around
    ARGs
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: ARGprofiler
  role: Snakemake workflow that orchestrates ARGextender, PanRes, abundance tables
    and Mash sketches for large-scale flanking-region extraction
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: fastq-dl
  role: Downloads FASTQ files from ENA/SRA for the samples to be processed
  repo: https://github.com/rpetit3/fastq-dl
- name: fastp
  role: Quality control and preprocessing of the short-read FASTQ input before assembly
  repo: https://github.com/OpenGene/fastp
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:d7bca64ed43f30c430c57d68f84eb6d91ac2031f55bf889c15bd48691c493b6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/ARGprofiler
  tools:
  - ARGextender
  - ARGprofiler
  - KMA
  - SPAdes
  - fastp
  - fastq-dl
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# metagenomic-flanking-region-extraction

## Summary

Targeted de novo assembly of genomic flanking regions around antimicrobial resistance genes (ARGs) in metagenomic short-read data, as implemented by ARGextender within the ARGprofiler Snakemake pipeline. It combines KMA read recruitment with SPAdes assembly to reconstruct sequence context around ARGs without running full metagenomic assembly.

## When to use

Use when short-read metagenomic FASTQ data must yield the genomic context (flanking regions) around ARGs or other target genes, and full de novo assembly would be computationally prohibitive.

## When NOT to use

- Input data are long reads (Nanopore/PacBio) — ARGprofiler is designed to only work with short-read sequencing data
- You need whole-genome/metagenome-wide assemblies rather than context around specific target genes — run full SPAdes de novo assembly instead
- No reference database exists for the target genes — ARGextender requires KMA identification of target genes before recruiting reads and skipping assembly otherwise

## Inputs

- Short-read metagenomic sequencing data (FASTQ, e.g. Illumina reads from ENA/SRA)
- ARG reference database, e.g. PanRes (14,078 unique ARGs) or ResFinder
- Sample accessions/metadata for downloading reads (e.g. ENA projects PRJEB40798, PRJEB40816, PRJEB40815, PRJEB27621, ERP015409)

## Outputs

- Targeted de novo assembly scaffolds containing ARGs and their genomic flanking regions
- Per-sample counts of ARG-containing scaffolds and distinct ARGs detected
- KMA mapping results matching reference gene sequences to scaffolds
- Abundance tables of genes (via ARGprofiler pipeline outputs)

## How to apply

Run ARGextender, which first uses recursive KMA (e.g. 1.4.12a) against the PanRes database (14,078 unique ARGs) to identify whether the target genes are present in a sample and recruit reads to the surrounding regions; if no target genes are identified, the more expensive assembly step is skipped entirely. Only when targets are present, SPAdes (e.g. 3.15.5) is run on the recruited reads to produce targeted de novo assemblies (scaffolds) around each ARG, with unlimited recursions by default. Match a reference gene database (e.g. ResFinder) back to the resulting scaffolds with KMA to confirm ARG presence, then extract the flanking regions surrounding matched ARGs. This targeted approach was shown to produce scaffolds comparable to full SPAdes assembly (mean ~101 vs ~95 ARG-scaffolds per sample across 951 urban sewage samples) in a much shorter time frame. Assemble the whole procedure inside the ARGprofiler Snakemake workflow so it scales across many publicly available metagenomic datasets.

## Related tools

- **ARGextender** (Targeted de novo assembly tool: uses KMA to identify target ARGs and recruit reads, then SPAdes to extend the flanking regions around genes of interest) — https://github.com/genomicepidemiology/ARGprofiler
- **KMA** (Recursive k-mer-based mapping to identify target ARGs in reads, recruit reads to surrounding regions, and match reference gene sequences to resulting scaffolds) — https://github.com/genomicepidemiology/ARGprofiler
- **SPAdes** (De novo assembler run on KMA-recruited reads to build targeted scaffolds around ARGs) — https://github.com/genomicepidemiology/ARGprofiler
- **ARGprofiler** (Snakemake workflow that orchestrates ARGextender, PanRes, abundance tables and Mash sketches for large-scale flanking-region extraction) — https://github.com/genomicepidemiology/ARGprofiler
- **fastq-dl** (Downloads FASTQ files from ENA/SRA for the samples to be processed) — https://github.com/rpetit3/fastq-dl
- **fastp** (Quality control and preprocessing of the short-read FASTQ input before assembly) — https://github.com/OpenGene/fastp

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- Per-sample scaffold counts should fall in reported ranges when reproduced on the 951 urban sewage samples: ARGextender ~1-350 ARG-scaffolds (mean 101) vs SPAdes full assembly ~1-348 (mean 95)
- Distinct ARGs detected per sample should match full-assembly baselines: ARGextender mean 57 (range 1-141) vs SPAdes mean 58 (range 1-153)
- No assembly should be produced for samples where KMA identified none of the target genes (the expensive SPAdes step is skipped)
- Matched scaffolds should contain the targeted ARG sequence plus flanking sequence on either side when matched against the reference database with KMA
- Runtime per sample should be substantially shorter than full de novo SPAdes assembly while yielding comparable flanking regions

## Limitations

- ARGprofiler is designed to only work with short-read sequencing data, thus not utilizing the advantages of long-read sequencing technologies
- PanRes combines databases of different sources and scopes; which subsets are appropriate must be decided per research question
- Comparability to full assembly depends on sample content — very low-load samples may yield as few as 1 ARG-scaffold

## Evidence

- [abstract] ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly: "ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly"
- [discussion] ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions: "ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions"
- [discussion] This approach also avoids running the more expensive algorithm if none of the target genes is identified: "This approach also avoids running the more expensive algorithm if none of the target genes is identified"
- [discussion] ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [discussion] ARGprofiler is designed to only work with short-read sequencing data, thus not utilizing the advantages of long-read sequencing technologies: "ARGprofiler is designed to only work with short-read sequencing data, thus not utilizing the advantages of long-read sequencing technologies"
