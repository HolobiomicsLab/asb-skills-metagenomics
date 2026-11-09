---
name: iterative-scaffold-extension-loop-implementation
description: Use when de novo assembly of genomic flanking regions around target genes
  (e.g. ARGs) is needed from short-read metagenomic FASTQ data but full metagenomic
  de novo assembly is too computationally expensive.
when_to_use_negative:
- Input is long-read sequencing data — ARGprofiler/ARGextender is designed to only
  work with short-read sequencing data and does not exploit long-read technologies
- The goal is full metagenomic de novo assembly of all contigs rather than extension
  of genomic flanks around pre-specified target genes
- The goal is taxonomic profiling (microbial abundance) rather than ARG detection
  and flanking-region reconstruction — use the mOTUs-based profiling path instead
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: ARGextender
  role: Implements the recursive KMA-SPAdes targeted de novo assembly loop that extends
    flanking regions around target genes
  repo: https://github.com/genomicepidemiology/ARGprofiler
- name: KMA
  role: Recruits reads mapping to target sequences and extracts scaffolds containing
    targets each recursion round
- name: SPAdes
  role: Performs the de novo assembly of KMA-recruited reads in each recursion round
- name: fastq-dl
  role: Downloads paired-end FASTQ input runs from ENA/SRA by accession
  repo: https://github.com/rpetit3/fastq-dl
- name: ARGprofiler
  role: Snakemake workflow that orchestrates the loop and data flow within the larger
    ARG analysis pipeline
  repo: https://github.com/genomicepidemiology/ARGprofiler
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae086
    title: ARGprofiler—a pipeline for large-scale analysis of antimicrobial resistance
      genes and their flanking regions in metagenomic datasets
  source_content_hash: sha256:f7c436984c6262cd516fcb084f8923e07d5e94b7342b6f1557a5ee8f905a5df3
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
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# iterative-scaffold-extension-loop-implementation

## Summary

Implements ARGextender's recursive KMA-SPAdes targeted-assembly loop, which recruits reads mapping to ARG targets, assembles them de novo with SPAdes, and iteratively promotes recovered scaffolds to new targets to extend genomic flanking regions while avoiding costly full metagenomic de novo assembly.

## When to use

Use when de novo assembly of genomic flanking regions around target genes (e.g. ARGs) is needed from short-read metagenomic FASTQ data but full metagenomic de novo assembly is too computationally expensive.

## When NOT to use

- Input is long-read sequencing data — ARGprofiler/ARGextender is designed to only work with short-read sequencing data and does not exploit long-read technologies
- The goal is full metagenomic de novo assembly of all contigs rather than extension of genomic flanks around pre-specified target genes
- The goal is taxonomic profiling (microbial abundance) rather than ARG detection and flanking-region reconstruction — use the mOTUs-based profiling path instead

## Inputs

- Paired-end short-read metagenomic FASTQ files (e.g. Copenhagen sewage sample PRJEB34633 downloaded via fastq-dl)
- PanRes ARG reference collection (Zenodo doi 10.5281/zenodo.8055115)
- KMA mapstat files used as a pre-assembly filter
- Initial target sequences (ARG references) in FASTA format

## Outputs

- Scaffolds and SPAdes assembly graph from the saturated targeted assembly
- Table mapping which target sequences (ARGs) occur in each scaffold
- FASTA file of assembled genomic flanking sequences around the ARGs (typical flank lengths 100–5000 bp in the paper's evaluation)

## How to apply

Start with samples pre-screened by KMA against the PanRes ARG collection (PanRes holds 14,078 unique ARGs), keeping only samples passing >90% query identity, >90% global consensus identity, and mean read depth >6 based on KMA mapstat files. Recruit reads mapping to target ARG sequences with KMA and assemble them de novo with SPAdes (targeted assembly rather than whole-metagenome assembly). After each round, extract scaffolds containing target sequences using KMA, retaining targets whose alignment score is within 70% of the best-scoring target, and set these scaffolds as the new targets. Repeat the KMA-SPAdes recursion until no additional reads are incorporated into the assembly or a user-defined maximum number of recursions is reached (unlimited by default). At saturation, save the scaffolds and assembly graph plus a table mapping which target sequences occur in each scaffold, and write the extracted genomic flanking sequences around the ARGs to a FASTA file. This design also avoids running the more expensive assembly algorithm if none of the target genes is identified.

## Related tools

- **ARGextender** (Implements the recursive KMA-SPAdes targeted de novo assembly loop that extends flanking regions around target genes) — https://github.com/genomicepidemiology/ARGprofiler
- **KMA** (Recruits reads mapping to target sequences and extracts scaffolds containing targets each recursion round)
- **SPAdes** (Performs the de novo assembly of KMA-recruited reads in each recursion round)
- **fastq-dl** (Downloads paired-end FASTQ input runs from ENA/SRA by accession) — https://github.com/rpetit3/fastq-dl
- **ARGprofiler** (Snakemake workflow that orchestrates the loop and data flow within the larger ARG analysis pipeline) — https://github.com/genomicepidemiology/ARGprofiler

## Examples

```
snakemake --cores 40 --use-envmodules --profile profile/ --configfile profile/config.yaml
```

## Evaluation signals

- Verify the recursion terminated by the no-new-reads saturation criterion or the user-defined maximum recursion count
- Confirm the output FASTA contains assembled flanks for detected ARGs, with typical flank lengths falling between 100 and 5000 bp as in the paper's evaluation
- Check the scaffold-target table records which ARG targets are present in each scaffold
- Confirm only samples passing the KMA pre-filter (>90% query identity, >90% global consensus identity, mean read depth >6) reached assembly
- Expect extension quality comparable to running SPAdes on the full sample but in a much shorter time frame, per the paper's sewage-sample comparison

## Limitations

- Recursion is unlimited by default, so a user-defined maximum should be set to bound runtime on hard-to-saturate samples
- Only supports short-read sequencing data; long-read support is planned but not available
- Scaffold-to-target assignment keeps targets with an alignment score within 70% of the best-scoring target, which may admit borderline matches
- If none of the target genes is identified in a sample, the expensive assembly step is skipped entirely — no flanks can be produced

## Evidence

- [abstract] ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly: "ARGextender is a bioinformatic approach combining KMA and SPAdes to recruit reads for a targeted de novo assembly"
- [discussion] ARGextender uses KMA and SPAdes to create targeted de novo assemblies: "ARGextender uses KMA and SPAdes to create targeted de novo assemblies by identifying if the targeted ARGs are present in a sample and then recruiting reads to the surrounding regions"
- [methods] Repeat the KMA-SPAdes recursion until no additional reads are incorporated: [evidence span withheld — anchor status: not_found]
- [discussion] ARGextender could extract comparable flanking regions to SPAdes: "ARGextender could extract comparable flanking regions to SPAdes but in a much shorter time frame"
- [readme] ARGextender, an assembly tool for extending the genomic flanking region around genes of interest: "ARGextender, an assembly tool for extending the genomic flanking region around genes of interest."
