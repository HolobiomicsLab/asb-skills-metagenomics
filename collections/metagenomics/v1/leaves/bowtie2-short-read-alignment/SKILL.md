---
name: bowtie2-short-read-alignment
description: Use when decontaminating paired or single short reads (e.g. Illumina
  2x150bp) from microbial samples by removing human host reads with Hostile's Bowtie2
  mode, or when benchmarking retention of bacterial reads versus human read removal.
when_to_use_negative:
- Input is long-read (ONT) data — use Hostile's Minimap2 mode (-x map-ont preset,
  --secondary no) instead, which requires ~13GB RAM
- The goal is masking human reads in place rather than removal — HRRT masks identified
  spots with 'N' via its k-mer database, a different output paradigm
- Retention of specific microbial sequences is the priority and the unmasked default
  index would falsely deplete them — instead use a masked index (e.g. human-t2t-hla-argos985
  or the viral/phage-masked index)
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Hostile
  role: Primary decontamination wrapper that invokes Bowtie2 in --sensitive mode with
    -k 1 and no secondary alignments to remove host reads from short-read FASTQ
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: Short-read aligner performing the actual alignment of reads to the human-t2t-hla
    index; requires ~4GB RAM
- name: conda
  role: Installation channel for Hostile and its non-Python dependencies (Bowtie2,
    Minimap2, Samtools, Bedtools)
- name: HRRT
  role: Comparison tool (NCBI SRA human scrubber) used to benchmark human removal
    and bacterial retention against Hostile's Bowtie2 mode
  repo: https://github.com/ncbi/sra-human-scrubber
provenance:
  source_task_ids:
  - task_004
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:751d3ed67944c0cc29000aad0a7d70209331aa18d33ded0578e0618ef33d1209
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bowtie2
  - DWGSIM
  - HRRT
  - Hostile
  - PBSIM2
  - conda
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bowtie2-short-read-alignment

## Summary

Bowtie2-based short-read host decontamination mode of Hostile, aligning paired or single Illumina FASTQ reads against the human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51) and removing aligned reads to clean microbial metagenomes.

## When to use

Use when decontaminating paired or single short reads (e.g. Illumina 2x150bp) from microbial samples by removing human host reads with Hostile's Bowtie2 mode, or when benchmarking retention of bacterial reads versus human read removal.

## When NOT to use

- Input is long-read (ONT) data — use Hostile's Minimap2 mode (-x map-ont preset, --secondary no) instead, which requires ~13GB RAM
- The goal is masking human reads in place rather than removal — HRRT masks identified spots with 'N' via its k-mer database, a different output paradigm
- Retention of specific microbial sequences is the priority and the unmasked default index would falsely deplete them — instead use a masked index (e.g. human-t2t-hla-argos985 or the viral/phage-masked index)

## Inputs

- Paired Illumina FASTQ(.gz) files from microbial or metagenomic samples (e.g. 2x150bp 1000 Genomes reads at 30x depth)
- Single-end short-read FASTQ(.gz) files or FASTQ streams from stdin
- Pre-built or masked Bowtie2 index (default human-t2t-hla = T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51; masked variants e.g. human-t2t-hla-argos985-mycob140)

## Outputs

- Cleaned FASTQ(.gz) files with reads aligning to the human index removed
- JSON decontamination log reporting reads_in, reads_out, reads_removed, and reads_removed_proportion
- Retention/depletion percentages for downstream benchmark comparison tables (e.g. masked_retention_results.csv)

## How to apply

Install Hostile via conda (`conda create -y -n hostile -c conda-forge -c bioconda hostile`), then fetch and cache the default human-t2t-hla index with `hostile index fetch`. Run `hostile clean` in Bowtie2 short-read mode, which uses the implicit --sensitive preset with -k 1 and no secondary alignments, to align paired FASTQ input against the index and emit cleaned FASTQs plus a JSON log with reads_in, reads_out, reads_removed and reads_removed_proportion. Compute retention as reads_out/reads_in; validation targets from the paper include removal of at least 99.6% of real human reads (Hostile retained 0.594% on 1000 Genomes 30x Illumina data versus HRRT's 0.729%) and retention of at least 99.989% of simulated bacterial reads (99.99998% for simulated mycobacterial Illumina reads). For maximum microbial retention, switch to a masked index such as human-t2t-hla-argos985-mycob140, which raised bacterial retention to 99.99994% with human removal degraded only marginally (0.131738% to 0.131979%). Bowtie2 mode requires about 4GB of RAM and processed bacterial Illumina reads at 32Mbp/s (210k reads/sec) with 8 alignment threads.

## Related tools

- **Hostile** (Primary decontamination wrapper that invokes Bowtie2 in --sensitive mode with -k 1 and no secondary alignments to remove host reads from short-read FASTQ) — https://github.com/bede/hostile
- **Bowtie2** (Short-read aligner performing the actual alignment of reads to the human-t2t-hla index; requires ~4GB RAM)
- **conda** (Installation channel for Hostile and its non-Python dependencies (Bowtie2, Minimap2, Samtools, Bedtools))
- **HRRT** (Comparison tool (NCBI SRA human scrubber) used to benchmark human removal and bacterial retention against Hostile's Bowtie2 mode) — https://github.com/ncbi/sra-human-scrubber

## Examples

```
hostile clean --fastq1 sample_R1.fastq.gz --fastq2 sample_R2.fastq.gz --index human-t2t-hla-argos985-mycob140
```

## Evaluation signals

- JSON log from `hostile clean` must show human read removal of at least 99.6% on real human data (Hostile retained 0.594% on 1000 Genomes 30x Illumina, 0.132% at 10x)
- Simulated bacterial Illumina retention must exceed the abstract-level guarantee of 99.989% (reported 99.99998% for simulated mycobacterial reads)
- With the masked index human-t2t-hla-argos985-mycob140, bacterial Illumina retention should rise to 99.99994% while human retention changes stay within ~0.001% (0.131738% to 0.131979%)
- Hostile should remove roughly 21-23% more human short reads and 21-43 times fewer bacterial reads than HRRT
- Throughput check: bacterial Illumina decontamination at ~32Mbp/s (210k reads/sec) with 8 alignment threads

## Limitations

- Unmasked default index removes more bacterial reads than necessary (15x more mycobacterial reads than HRRT on simulated Illumina data); use a masked index when microbial retention is critical
- Bowtie2 mode is unsuitable for long reads; ONT data requires Minimap2 mode with ~13GB RAM
- Earlier Hostile versions did not verify downloaded index checksums (issue #20) and could corrupt empty fastq.gz files when no reads remained after decontamination (issue #24)

## Evidence

- [methods] Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2): "Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)"
- [methods] Default index composition: "The default index `human-t2t-hla` comprises [T2T-CHM13v2.0](https://www.ncbi.nlm.nih.gov/assembly/11828891) and [IPD-IMGT/HLA](https://www.ebi.ac.uk/ipd/imgt/hla/) v3.51"
- [abstract] Abstract retention/removal guarantee: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [readme] Readme install instructions: "conda create -y -n hostile -c conda-forge -c bioconda hostile"
- [readme] Readme throughput benchmark: "bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads"
