---
name: metagenomic-read-recruitment-targeted-assembly
description: Use when you need genomic flanking regions around antimicrobial resistance
  genes (or other target genes) from short-read metagenomic FASTQ data, and full metagenomic
  de novo assembly is too computationally expensive.
when_to_use_negative:
- Input is long-read sequencing data — ARGprofiler/ARGextender is designed to only
  work with short-read sequencing data
- You already have a full metagenomic de novo assembly or finished genomes to mine
  for flanks — direct sequence extraction is cheaper than read recruitment
- No target genes are identified in the pre-screen — running the assembly step is
  wasted compute since the KMA filter step exists precisely to avoid this
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: KMA
  role: Recruits reads to target ARG sequences, filters samples via mapstat thresholds
    (>90% identity, mean depth >6), and extracts target-containing scaffolds (within
    70% of best alignment score) each recursion round
- name: SPAdes
  role: De novo assembly of the KMA-recruited reads in each targeted-assembly round
    (version 3.15.5 in the benchmark)
- name: ARGextender
  role: Implements the recursive KMA-SPAdes targeted-assembly loop and writes flanking
    FASTA, scaffolds, and scaffold-target tables
- name: ARGprofiler
  role: Snakemake workflow wrapping ARGextender and the PanRes database for large-scale
    short-read metagenomic processing
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:eb70d523dd6809d8610ac067881214216055f7405f3c2dc94462c40af2bbc969
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# metagenomic-read-recruitment-targeted-assembly

## Summary

ARGextender-style targeted de novo assembly: iteratively recruit reads to ARG targets with KMA, assemble the recruited reads with SPAdes, and re-target scaffolds until read recruitment saturates — recovering ARG flanking regions without full metagenomic de novo assembly.

## When to use

Use when you need genomic flanking regions around antimicrobial resistance genes (or other target genes) from short-read metagenomic FASTQ data, and full metagenomic de novo assembly is too computationally expensive.

## When NOT to use

- Input is long-read sequencing data — ARGprofiler/ARGextender is designed to only work with short-read sequencing data
- You already have a full metagenomic de novo assembly or finished genomes to mine for flanks — direct sequence extraction is cheaper than read recruitment
- No target genes are identified in the pre-screen — running the assembly step is wasted compute since the KMA filter step exists precisely to avoid this

## Inputs

- Paired-end short-read FASTQ files from metagenomic samples (e.g. downloaded from ENA/SRA with fastq-dl)
- PanRes ARG reference collection (combined ARG database, doi 10.5281/zenodo.8055115)
- KMA mapstat files from pre-screening (for sample-level filters: >90% query identity, >90% global consensus identity, mean read depth >6)

## Outputs

- FASTA file of assembled genomic flanking sequences around ARGs (typical flank lengths 100–5000 bp)
- ARG-containing scaffolds and SPAdes assembly graph
- Table mapping which target ARG sequences occur in each scaffold
- Per-sample counts of ARG-scaffolds and distinct ARGs detected

## How to apply

Pre-screen reads against the PanRes ARG reference collection (14,078 unique ARGs) with KMA, filtering mapstat results so only samples passing >90% query identity, >90% global consensus identity, and mean read depth >6 proceed to assembly — this avoids running the expensive assembly algorithm when no target genes are identified. Recruit reads mapping to target ARG sequences with KMA and assemble only those reads de novo with SPAdes. After each round, extract scaffolds containing target sequences using KMA, retaining targets whose alignment score falls within 70% of the best-scoring target, and set these scaffolds as the new targets. Repeat the KMA-SPAdes recursion until no more reads are included in the assembly or the user-defined maximum number of recursions is reached (unlimited by default; the benchmark used KMA 1.4.12a and SPAdes 3.15.5). On saturation, save scaffolds plus the assembly graph and a table mapping which target sequences occur in each scaffold, and write the extracted genomic flanking sequences around the ARGs to a FASTA file.

## Related tools

- **KMA** (Recruits reads to target ARG sequences, filters samples via mapstat thresholds (>90% identity, mean depth >6), and extracts target-containing scaffolds (within 70% of best alignment score) each recursion round)
- **SPAdes** (De novo assembly of the KMA-recruited reads in each targeted-assembly round (version 3.15.5 in the benchmark))
- **ARGextender** (Implements the recursive KMA-SPAdes targeted-assembly loop and writes flanking FASTA, scaffolds, and scaffold-target tables)
- **ARGprofiler** (Snakemake workflow wrapping ARGextender and the PanRes database for large-scale short-read metagenomic processing) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
kma -ipe reads_R1.fq.gz reads_R2.fq.gz -t_db PanRes -ont -o argextender_round1
```

## Evaluation signals

- Verify the recursion terminated by the saturation criterion — no new reads incorporated into the assembly — or by the user-defined maximum recursion count
- Check the output FASTA contains assembled flanks for detected ARGs, with typical flank lengths between 100 and 5000 bp
- Confirm the scaffold-target table records which ARGs are present in each scaffold
- Benchmark against full SPAdes metagenomic assembly: expect comparable output (mean ~101 ARG-scaffolds, range 1–350, vs 95, range 1–348; mean ~57 distinct ARGs, range 1–141, vs 58, range 1–153) at much lower compute cost
- Confirm samples failing the pre-screen thresholds (>90% query identity, >90% global consensus identity, mean depth >6) never reach assembly

## Limitations

- ARGprofiler is designed to only work with short-read sequencing data, so long-read-derived structural context around ARGs is missed
- Unlimited recursions by default means runtime depends on read-recruitment saturation; users must set a maximum recursion count to bound cost
- PanRes combines different sources and scopes, so it is up to the individual research question which subsets are appropriate for consideration

## Evidence

- [abstract] ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly: "ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly"
- [discussion] ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions: "ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions"
- [discussion] This approach also avoids running the more expensive algorithm if none of the target genes is identified: "This approach also avoids running the more expensive algorithm if none of the target genes is identified"
- [discussion] ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [readme] ARGextender, an assembly tool for extending the genomic flanking region around genes of interest: "ARGextender, an assembly tool for extending the genomic flanking region around genes of interest"
