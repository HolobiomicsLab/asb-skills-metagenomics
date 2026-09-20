---
name: kma-mapstat-alignment-filtering
description: Use when you have paired-end short reads and a gene reference collection
  (e.g. PanRes ARGs) and need to decide which samples/genes warrant expensive de novo
  assembly, or to recruit reads around target genes for reconstructing flanking regions.
when_to_use_negative:
- Input is long-read sequencing data — ARGprofiler is designed to only work with short-read
  sequencing data
- No target genes are identified in the KMA pre-screen — the expensive assembly step
  is deliberately avoided in that case
- You need a full metagenomic assembly rather than flanks around specific gene targets
  — the KMA-SPAdes loop is a targeted, not whole-community, assembly
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0798
tools:
- name: KMA
  role: Maps reads against the PanRes ARG database to pre-screen samples via mapstat
    files, recruits reads to targets, and extracts scaffolds containing target sequences
    after each assembly round
- name: SPAdes
  role: Performs targeted de novo assembly of the KMA-recruited reads in each recursion
    round
- name: ARGextender
  role: Orchestrates the KMA-SPAdes recursion to extend genomic flanking regions around
    genes of interest
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: ARGprofiler
  role: Snakemake workflow wrapping KMA/ARGextender for large-scale ARG and flanking-region
    analysis
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
  source_content_hash: sha256:41290d7582ed272069e986879c528ba442f3f6183c409e8b1b838bc4955c85b5
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0798
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# kma-mapstat-alignment-filtering

## Summary

A targeted-alignment filtering approach from ARGprofiler/ARGextender that uses KMA to map short reads against the PanRes ARG reference collection, applies identity/depth cutoffs to KMA mapstat output, and recycles recruited reads into SPAdes for targeted de novo assembly instead of a full metagenomic assembly.

## When to use

Use when you have paired-end short reads and a gene reference collection (e.g. PanRes ARGs) and need to decide which samples/genes warrant expensive de novo assembly, or to recruit reads around target genes for reconstructing flanking regions.

## When NOT to use

- Input is long-read sequencing data — ARGprofiler is designed to only work with short-read sequencing data
- No target genes are identified in the KMA pre-screen — the expensive assembly step is deliberately avoided in that case
- You need a full metagenomic assembly rather than flanks around specific gene targets — the KMA-SPAdes loop is a targeted, not whole-community, assembly

## Inputs

- Paired-end short-read FASTQ files (e.g. from Copenhagen sewage sample PRJEB34633, downloaded with fastq-dl)
- PanRes ARG reference collection (doi 10.5281/zenodo.8055115)
- KMA mapstat files from the pre-screen
- Scaffolds from the previous SPAdes assembly round (used as new targets)

## Outputs

- Filtered set of samples/genes passing identity and depth thresholds (from KMA mapstat files)
- Scaffolds and assembly graph containing target ARG sequences
- Table mapping which target sequences occur in each scaffold
- FASTA file of assembled genomic flanking sequences around ARGs (typical flank lengths 100–5000 bp in the paper's evaluation)

## How to apply

First pre-screen each sample by mapping reads with KMA against the PanRes database (14,078 unique ARGs) and inspect the KMA mapstat files, retaining only samples passing >90% query identity, >90% global consensus identity, and mean read depth >6; this avoids running the more expensive assembly algorithm when no target genes are identified. Next, recruit reads mapping to target ARG sequences with KMA and assemble the recruited reads de novo with SPAdes (targeted assembly rather than full metagenome assembly). After each SPAdes round, extract scaffolds containing target sequences using KMA, retaining targets with an alignment score within 70% of the best-scoring target, and set these scaffolds as the new targets. Repeat the KMA-SPAdes recursion until no more reads are included in the assembly or the user-defined maximum number of recursions is met (unlimited by default); at saturation, save scaffolds and the assembly graph, write a scaffold-to-target table, and extract genomic flanking sequences around the ARGs to a FASTA file.

## Related tools

- **KMA** (Maps reads against the PanRes ARG database to pre-screen samples via mapstat files, recruits reads to targets, and extracts scaffolds containing target sequences after each assembly round)
- **SPAdes** (Performs targeted de novo assembly of the KMA-recruited reads in each recursion round)
- **ARGextender** (Orchestrates the KMA-SPAdes recursion to extend genomic flanking regions around genes of interest) — https://github.com/genomicepidemiology/ARGprofiler
- **ARGprofiler** (Snakemake workflow wrapping KMA/ARGextender for large-scale ARG and flanking-region analysis) — https://github.com/genomicepidemiology/ARGprofiler
- **fastq-dl** (Downloads the paired-end FASTQ inputs from ENA/SRA by accession) — https://github.com/rpetit3/fastq-dl

## Evaluation signals

- Recursion terminated by the no-new-reads saturation criterion (or the user-defined maximum number of recursions), not by error
- Output FASTA contains assembled flanks for detected ARGs, with typical flank lengths between 100 and 5000 bp in the paper's evaluation
- The scaffold-target table records which ARG target sequences are present in each scaffold
- Pre-screen filter: only samples with >90% query identity, >90% global consensus identity, and mean read depth >6 in KMA mapstat files proceed to assembly
- Post-recursion target retention: scaffolds kept have an alignment score within 70% of the best-scoring target

## Limitations

- ARGprofiler/ARGextender supports only short-read data; long-read support is planned but not available
- PanRes combines different sources and scopes, so which subsets are appropriate is left to individual research questions
- Typical flanking-region lengths fall between 100 and 5000 bp in the paper's evaluation, so very distant flanks may not be recovered
- Recursion defaults to unlimited, which can extend run time if a saturation point is never reached

## Evidence

- [abstract] ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly: "ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly"
- [discussion] Targeted de novo assembly loop described in the workflow: "ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions"
- [discussion] Avoiding expensive assembly when no target genes identified: "This approach also avoids running the more expensive algorithm if none of the target genes is identified"
- [discussion] Benchmark result supporting the targeted-assembly approach: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [abstract] PanRes database size: "providing the PanRes database of 14 078 unique ARGs that combines several existing collections into one"
