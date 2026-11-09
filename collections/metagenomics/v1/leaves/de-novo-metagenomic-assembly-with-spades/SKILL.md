---
name: de-novo-metagenomic-assembly-with-spades
description: Use when short paired-end metagenomic reads contain ARGs (pre-screened
  via KMA against PanRes) and you need the surrounding genomic context assembled without
  assembling the entire metagenome.
when_to_use_negative:
- Input data are long reads — ARGprofiler/ARGextender is designed to only work with
  short-read sequencing data
- None of the target ARGs are identified in the sample by KMA — the more expensive
  assembly algorithm should then be skipped entirely
- You need whole-community metagenomic assemblies rather than flanks around specific
  target genes
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3837
- http://edamontology.org/topic_3305
tools:
- name: ARGextender
  role: 'Orchestrates the targeted assembly: recruits reads around target ARGs and
    iterates assembly until saturation'
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: KMA
  role: Recruits reads mapping to target ARG/scaffold sequences and extracts scaffolds
    containing targets after each round
- name: SPAdes
  role: Performs the de novo assembly of the recruited target reads in each recursion
    round
- name: ARGprofiler
  role: Snakemake workflow wrapping the whole process, including PanRes pre-screening
    and ARGextender invocation
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: fastq-dl
  role: Downloads the paired-end FASTQ inputs from ENA/SRA by accession
  repo: https://github.com/rpetit3/fastq-dl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:6114109980335f7eae71bd625c3e418d214f413a9beaf1d263b5e07e529e6f21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/genomicepidemiology/ARGprofiler
  tools:
  - ARGextender
  - ARGprofiler
  - ContigExtender
  - KMA
  - SPAdes
  - fastq-dl
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3837
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metagenomic-de-novo-assembly
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# de-novo-metagenomic-assembly-with-spades

## Summary

Reconstruct genomic flanking regions around antimicrobial resistance genes (ARGs) by iterating KMA read recruitment and SPAdes targeted de novo assembly, instead of performing computationally expensive full metagenomic assembly. This is the core of ARGextender within the ARGprofiler Snakemake workflow.

## When to use

Use when short paired-end metagenomic reads contain ARGs (pre-screened via KMA against PanRes) and you need the surrounding genomic context assembled without assembling the entire metagenome.

## When NOT to use

- Input data are long reads — ARGprofiler/ARGextender is designed to only work with short-read sequencing data
- None of the target ARGs are identified in the sample by KMA — the more expensive assembly algorithm should then be skipped entirely
- You need whole-community metagenomic assemblies rather than flanks around specific target genes

## Inputs

- Paired-end short-read metagenomic FASTQ files (e.g. Copenhagen sewage sample PRJEB34633 obtained via fastq-dl)
- PanRes ARG reference collection (14,078 unique ARGs, Zenodo doi 10.5281/zenodo.8055115)
- KMA mapstat files used to filter samples before assembly

## Outputs

- Scaffolds and assembly graph from the saturated targeted assembly
- Table mapping which target ARG sequences occur in each scaffold
- FASTA file of extracted genomic flanking sequences around the ARGs

## How to apply

First pre-screen samples with KMA against the PanRes database (14,078 unique ARGs) and use the KMA mapstat files as a filter, proceeding only with samples passing >90% query identity, >90% global consensus identity, and mean read depth >6. Recruit reads mapping to the target ARG sequences with KMA and assemble only those reads de novo with SPAdes, deliberately avoiding full metagenomic assembly. After each SPAdes round, extract scaffolds containing the targets using KMA, retaining targets with an alignment score within 70% of the best-scoring target, and set these scaffolds as the new targets for the next round. Repeat the KMA-SPAdes recursion until no more reads are included in the assembly or a user-defined maximum number of recursions is reached (unlimited by default). On saturation, save the scaffolds and assembly graph plus a table mapping which target sequences occur in each scaffold, and write the extracted genomic flanking sequences to a FASTA file.

## Related tools

- **ARGextender** (Orchestrates the targeted assembly: recruits reads around target ARGs and iterates assembly until saturation) — https://github.com/genomicepidemiology/ARGprofiler
- **KMA** (Recruits reads mapping to target ARG/scaffold sequences and extracts scaffolds containing targets after each round)
- **SPAdes** (Performs the de novo assembly of the recruited target reads in each recursion round)
- **ARGprofiler** (Snakemake workflow wrapping the whole process, including PanRes pre-screening and ARGextender invocation) — https://github.com/genomicepidemiology/ARGprofiler
- **fastq-dl** (Downloads the paired-end FASTQ inputs from ENA/SRA by accession) — https://github.com/rpetit3/fastq-dl

## Examples

```
fastq-dl -a PRJEB34633 --ena; snakemake --use-conda --cores 40 --configfile profile/config.yaml
```

## Evaluation signals

- The recursion terminated by the no-new-reads saturation criterion (or a user-defined maximum number of recursions)
- Output FASTA contains assembled flanks for detected ARGs, with typical flank lengths between 100 and 5000 bp as in the paper's evaluation
- Scaffold-target table correctly records which ARGs are present per scaffold
- Samples that failed the pre-assembly filter (>90% query identity, >90% global consensus identity, mean read depth >6) did not enter assembly
- ARGextender flanks are comparable to a full SPAdes assembly of the same sample but produced in a much shorter time frame

## Limitations

- Only supports short-read sequencing data; long-read support is planned but not available
- Assembly may never saturate unless capped — the maximum number of recursions is unlimited by default, so a user-defined cap may be needed
- Avoids running the expensive assembly algorithm only if target genes are first identified by KMA; samples without detectable targets yield nothing

## Evidence

- [abstract] ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly: "ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly"
- [discussion] ARGextender recursion until saturation: "ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions"
- [discussion] Skip expensive assembly when no targets found: "This approach also avoids running the more expensive algorithm if none of the target genes is identified"
- [discussion] ARGextender comparable to SPAdes but faster: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [discussion] ARGprofiler short-read scope: "ARGprofiler is designed to only work with short-read sequencing data, thus not utilizing the advantages of long-read sequencing technologies"
