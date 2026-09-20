---
name: fda-argos-reference-genome-handling
description: 'Use when benchmarking host-decontamination precision on known bacterial
  content: you need reference-grade complete bacterial assemblies (FDA-ARGOS, BioProject
  231221) to simulate reads or to mask a human index against bacterial sequence.'
when_to_use_negative:
- The goal is removing real human reads from clinical metagenomes rather than benchmarking
  retention on known bacterial content — use Hostile's default human-t2t-hla index
  directly.
- Viral or phage sequence retention is the priority — the human-t2t-hla-argos985 masking
  covers only the 985 bacterial genomes, not the 18,719 RefSeq viral or 26,928 Millard
  Lab phage genomes.
- The input taxa are mycobacteria beyond the FDA-ARGOS set — use human-t2t-hla-argos985-mycob140,
  which additionally masks 140 mycobacterial genomes.
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Hostile
  role: Decontaminates simulated paired Illumina reads and emits JSON logs (reads_in/reads_out)
    used to compute bacterial read retention; its mask utility builds indexes masked
    with 150mers of the FDA-ARGOS genomes
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: Alignment engine used automatically by Hostile in paired short-read mode with
    default --sensitive parameters (-D 15 -R 2 -N 0 -L 22 -i S,1,1.15, -k 1) against
    human-t2t-hla
- name: Minimap2
  role: Aligns 150mers of the supplied FDA-ARGOS target genomes with the reference
    genome during index masking
- name: Bedtools
  role: Masks all Minimap2-aligned regions with N to produce the bacterial-masked
    index
- name: HRRT (ncbi/sra-human-scrubber)
  role: Comparison tool run on the same simulated reads to quantify relative bacterial
    read removal (mean 21x more reads removed than Hostile)
  repo: https://github.com/ncbi/sra-human-scrubber
- name: DWGSIM
  role: Simulates paired Illumina reads from each of the 985 FDA-ARGOS assemblies
    to create ground-truth bacterial read sets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:41046e51cb25a852ccb20e248c93d51b728eaba835cf90758f8d099433abc6a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bedtools
  - Bowtie2
  - DWGSIM
  - HRRT (ncbi/sra-human-scrubber)
  - Hostile
  - Minimap2
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fda-argos-reference-genome-handling

## Summary

Handling of the 985 reference-grade FDA-ARGOS bacterial genomes (BioProject 231221) as ground-truth inputs for benchmarking host read removal tools, including their use in simulating bacterial reads with DWGSIM and in building masked Hostile indexes such as human-t2t-hla-argos985.

## When to use

Use when benchmarking host-decontamination precision on known bacterial content: you need reference-grade complete bacterial assemblies (FDA-ARGOS, BioProject 231221) to simulate reads or to mask a human index against bacterial sequence.

## When NOT to use

- The goal is removing real human reads from clinical metagenomes rather than benchmarking retention on known bacterial content — use Hostile's default human-t2t-hla index directly.
- Viral or phage sequence retention is the priority — the human-t2t-hla-argos985 masking covers only the 985 bacterial genomes, not the 18,719 RefSeq viral or 26,928 Millard Lab phage genomes.
- The input taxa are mycobacteria beyond the FDA-ARGOS set — use human-t2t-hla-argos985-mycob140, which additionally masks 140 mycobacterial genomes.

## Inputs

- 985 reference-grade complete FDA-ARGOS bacterial genome assemblies (NCBI BioProject 231221)
- Paired FASTQ files simulated from assemblies with DWGSIM
- Hostile default index human-t2t-hla (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51)

## Outputs

- Hostile JSON logs with reads_in, reads_out, reads_removed, reads_removed_proportion fields
- HRRT fastq.clean output files
- Bacterial read retention comparison table (e.g. bacterial_retention_comparison.csv)
- Optionally, a masked index (human-t2t-hla-argos985) with 317,973 masked positions

## How to apply

Download the 985 reference-grade bacterial FDA-ARGOS genomes from NCBI BioProject 231221 (including common human pathogens). To benchmark bacterial read retention, simulate paired Illumina reads from each assembly with DWGSIM, then run `hostile clean --fastq1 <r1> --fastq2 <r2>` in paired short-read mode (Bowtie2 with default --sensitive parameters against the human-t2t-hla index) plus HRRT on the same reads, computing retention as reads_out/reads_in from the Hostile JSON logs and HRRT output. Hostile retained 99.99989% of simulated bacterial reads versus 99.99752% for HRRT, with HRRT removing a mean of 21 times (95% CI: 9, 30) more bacterial reads. Alternatively, use the assemblies to build masked indexes: the `human-t2t-hla-argos985` index masks `human-t2t-hla` with 150mers from the 985 genomes (317,973 masked positions, 0.010%), and masking uses Minimap2 to align 150mers of the target genomes to the reference with bedtools masking aligned regions with N.

## Related tools

- **Hostile** (Decontaminates simulated paired Illumina reads and emits JSON logs (reads_in/reads_out) used to compute bacterial read retention; its mask utility builds indexes masked with 150mers of the FDA-ARGOS genomes) — https://github.com/bede/hostile
- **Bowtie2** (Alignment engine used automatically by Hostile in paired short-read mode with default --sensitive parameters (-D 15 -R 2 -N 0 -L 22 -i S,1,1.15, -k 1) against human-t2t-hla)
- **Minimap2** (Aligns 150mers of the supplied FDA-ARGOS target genomes with the reference genome during index masking)
- **Bedtools** (Masks all Minimap2-aligned regions with N to produce the bacterial-masked index)
- **HRRT (ncbi/sra-human-scrubber)** (Comparison tool run on the same simulated reads to quantify relative bacterial read removal (mean 21x more reads removed than Hostile)) — https://github.com/ncbi/sra-human-scrubber
- **DWGSIM** (Simulates paired Illumina reads from each of the 985 FDA-ARGOS assemblies to create ground-truth bacterial read sets)

## Examples

```
hostile clean --fastq1 r1.fastq.gz --fastq2 r2.fastq.gz --index human-t2t-hla-argos985
```

## Evaluation signals

- Hostile must retain at least 99.989% of simulated bacterial reads (reads_out/reads_in from JSON logs), matching the reported figure of 99.99989%
- HRRT retention on the same reads should be ~99.99752%, with HRRT removing a mean of 21x (95% CI: 9, 30) more bacterial reads than Hostile
- Each Hostile run must produce a valid JSON log containing reads_in, reads_out, reads_removed, and reads_removed_proportion fields
- Masked index construction should report 317,973 masked positions (0.010%) for human-t2t-hla-argos985
- Masked-index usage should raise bacterial retention to ≥99.997% with ≤0.001% reduction in human read removal

## Limitations

- Retention figures apply to simulated reads (DWGSIM) and may not perfectly reflect real bacterial communities
- The default unmasked human-t2t-hla index is not masked against any bacterial sequence (0 masked positions); maximum retention requires explicitly using or building a masked index
- Earlier Hostile versions did not verify downloaded index checksums, and could corrupt empty fastq.gz files when no reads remained after decontamination (issues #20, #24)

## Evidence

- [abstract] our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [methods] ARGOS dataset description: "985 reference grade bacterial genomes](https://www.ncbi.nlm.nih.gov/bioproject/231221) including common human pathogens"
- [methods] Masking method: "Masking uses Minimap2 to align 150mers of the supplied target genomes with the reference genome, and bedtools to mask all aligned regions with N."
- [abstract] Masked index retention: "Using Hostile with a masked reference genome further increases bacterial read retention (≥99.997%) with negligible (≤0.001%) reduction in human read removal performance"
- [readme] README index table: "`human-t2t-hla` masked with 150mers for [985](https://github.com/bede/hostile/blob/main/paper/supplementary-table-2.tsv) [FDA-ARGOS](https://www.ncbi.nlm.nih.gov/bioproject/231221) **bacterial**"
