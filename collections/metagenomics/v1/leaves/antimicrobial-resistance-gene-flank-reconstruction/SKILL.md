---
name: antimicrobial-resistance-gene-flank-reconstruction
description: Use when short-read metagenomic FASTQ data must yield assembled genomic
  context (100–5000 bp flanks) around ARGs, but full metagenomic de novo assembly
  is computationally prohibitive or unnecessary.
when_to_use_negative:
- Input is long-read sequencing data — ARGprofiler is designed to only work with short-read
  data
- 'No target ARGs are detected by the KMA pre-screen (filters not met: >90% query
  identity, >90% global consensus identity, mean depth >6) — assembly is skipped to
  avoid expense'
- A full, unbiased metagenomic assembly is required rather than targeted assembly
  around known ARG references
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: ARGextender
  role: Core assembly tool driving the recursive KMA-SPAdes targeted de novo assembly
    to extend flanking regions around genes of interest
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: KMA
  role: Maps short reads to PanRes ARG targets; provides mapstat files for pre-screening
    filters and read recruitment in each recursion round
- name: SPAdes
  role: De novo assembler for reads recruited to ARG targets in each recursion round
- name: ARGprofiler
  role: Snakemake workflow that orchestrates KMA, SPAdes/ARGextender, abundance tables,
    and Mash sketches
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: fastq-dl
  role: Downloads paired-end FASTQ inputs from ENA/SRA by accession
  repo: https://github.com/rpetit3/fastq-dl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:c213393bf95fff78ee0b891accbec8cf8357391a5553cef0bdf918924f21198e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/ARGprofiler
  tools:
  - ARGextender
  - ARGprofiler
  - KMA
  - SPAdes
  - fastq-dl
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# antimicrobial-resistance-gene-flank-reconstruction

## Summary

Reconstructs genomic flanking regions around antimicrobial resistance genes (ARGs) from metagenomic short reads via iterative targeted de novo assembly (ARGextender: KMA read recruitment + SPAdes assembly), avoiding the cost of full metagenomic assembly. Implemented within the Snakemake-based ARGprofiler pipeline with the PanRes database of 14,078 unique ARGs.

## When to use

Use when short-read metagenomic FASTQ data must yield assembled genomic context (100–5000 bp flanks) around ARGs, but full metagenomic de novo assembly is computationally prohibitive or unnecessary.

## When NOT to use

- Input is long-read sequencing data — ARGprofiler is designed to only work with short-read data
- No target ARGs are detected by the KMA pre-screen (filters not met: >90% query identity, >90% global consensus identity, mean depth >6) — assembly is skipped to avoid expense
- A full, unbiased metagenomic assembly is required rather than targeted assembly around known ARG references

## Inputs

- Paired-end short-read metagenomic FASTQ files (e.g. downloaded from ENA/SRA via fastq-dl)
- PanRes ARG reference collection (14,078 unique ARGs, Zenodo doi 10.5281/zenodo.8055115)
- KMA mapstat files used for sample pre-screening filters

## Outputs

- FASTA file of assembled genomic flanking sequences around ARGs (typical flank lengths 100–5000 bp)
- Scaffolds and assembly graph (SPAdes output) from the final recursion round
- Table mapping which target ARG sequences occur in each scaffold
- Abundance tables of genes (from KMA) and microbes

## How to apply

Obtain paired-end short reads with fastq-dl (e.g. ENA project PRJEB34633) and the PanRes reference collection from Zenodo (doi 10.5281/zenodo.8055115). Pre-screen samples with KMA against PanRes and use the KMA mapstat files to retain only samples passing >90% query identity, >90% global consensus identity, and mean read depth >6, avoiding assembly when no target genes are identified. Run the KMA-SPAdes recursion: KMA recruits reads mapping to target ARG sequences, SPAdes assembles them de novo, and scaffolds containing targets (alignment score within 70% of the best-scoring target) become the new targets for the next round. Repeat until no additional reads are incorporated or the user-defined maximum number of recursions is met (unlimited by default). On saturation, save scaffolds and the assembly graph, record which target sequences occur in each scaffold, and write the flanking sequences to a FASTA file.

## Related tools

- **ARGextender** (Core assembly tool driving the recursive KMA-SPAdes targeted de novo assembly to extend flanking regions around genes of interest) — https://github.com/genomicepidemiology/ARGprofiler
- **KMA** (Maps short reads to PanRes ARG targets; provides mapstat files for pre-screening filters and read recruitment in each recursion round)
- **SPAdes** (De novo assembler for reads recruited to ARG targets in each recursion round)
- **ARGprofiler** (Snakemake workflow that orchestrates KMA, SPAdes/ARGextender, abundance tables, and Mash sketches) — https://github.com/genomicepidemiology/ARGprofiler
- **fastq-dl** (Downloads paired-end FASTQ inputs from ENA/SRA by accession) — https://github.com/rpetit3/fastq-dl

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- The KMA-SPAdes recursion terminates by the saturation criterion: no more reads are included in the de novo assembly, or the user-defined maximum recursion count is reached
- Output FASTA contains assembled flanking sequences for detected ARGs, with typical flank lengths between 100 and 5000 bp as in the paper's evaluation
- The scaffold-target table correctly records which PanRes ARG targets occur in each scaffold
- Samples entering assembly passed the KMA pre-screen: >90% query identity, >90% global consensus identity, and mean read depth >6
- Flank reconstruction matches SPAdes full-assembly flanks on benchmark samples (ARGextender extracted comparable flanking regions to SPAdes in sewage samples) but runs in much less time

## Limitations

- ARGprofiler is designed to only work with short-read sequencing data, not utilizing long-read technologies
- PanRes combines different sources and scopes, so the appropriateness of subsets is left to individual research questions
- Real datasets may contain many more unknowns than the in silico evaluation data suggested

## Evidence

- [abstract] ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly: "ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly"
- [discussion] ARGextender creates targeted de novo assemblies by detecting ARGs then recruiting reads to surrounding regions: "ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions"
- [discussion] ARGextender matched SPAdes flanks in much shorter time: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [readme] Pipeline designed to analyze ARG flanking regions in metagenomic data: "ARGprofiler is a newly developed Snakemake pipeline designed to analyze ARGs' read distances, abundances, and genomic flanking regions in metagenomic sequencing data."
- [intro] Pipeline can be repurposed for other tasks: "while we have targeted antimicrobial resistance (AMR), the pipeline can be repurposed for other tasks"
