---
name: bowtie2-alignment-based-read-classification
description: 'Use when paired or single-end Illumina FASTQ reads from a microbial
  sample must be screened for human origin: aligning to a host reference index is
  needed to remove human contamination while retaining bacterial reads.'
when_to_use_negative:
- Input is long-read (ONT/PacBio) data — Hostile routes long reads to Minimap2 (13GB
  RAM) instead of Bowtie2
- Maximum viral or phage sequence retention is the priority — use a masked index such
  as human-t2t-hla.argos-bacteria-985_rs-viral-202401_ml-phage-202401 rather than
  the unmasked default
- The goal is masking rather than removing suspected host reads (e.g. NCBI HRRT-style
  'N' masking) — alignment-based removal discards aligned reads entirely
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Hostile
  role: Wraps Bowtie2 alignment in paired short-read mode to classify and remove host-aligned
    reads from FASTQ input, emitting cleaned FASTQ and a JSON log
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: Alignment engine for short-read classification against the human-t2t-hla index
    with default --sensitive parameters and -k 1
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:5e468f1827bd4f6806af64b5190bec95319eafed88c10b8a5942807fa0c04ec0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bowtie2
  - Hostile
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Bowtie2 alignment-based read classification

## Summary

Classify sequencing reads as host or non-host by aligning them with Bowtie2 against a human reference index (e.g. human-t2t-hla, T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51) and discarding aligned reads, as implemented in Hostile for short-read decontamination.

## When to use

Use when paired or single-end Illumina FASTQ reads from a microbial sample must be screened for human origin: aligning to a host reference index is needed to remove human contamination while retaining bacterial reads.

## When NOT to use

- Input is long-read (ONT/PacBio) data — Hostile routes long reads to Minimap2 (13GB RAM) instead of Bowtie2
- Maximum viral or phage sequence retention is the priority — use a masked index such as human-t2t-hla.argos-bacteria-985_rs-viral-202401_ml-phage-202401 rather than the unmasked default
- The goal is masking rather than removing suspected host reads (e.g. NCBI HRRT-style 'N' masking) — alignment-based removal discards aligned reads entirely

## Inputs

- Paired-end FASTQ or FASTQ.gz files (short reads, e.g. simulated DWGSIM reads from FDA-ARGOS bacterial assemblies)
- Single-end FASTQ/FASTQ.gz files
- Prebuilt human reference index (human-t2t-hla, comprising T2T-CHM13v2.0 and IPD-IMGT/HLA v3.51), downloaded automatically on first run

## Outputs

- Cleaned FASTQ.gz files with host-aligned reads removed
- JSON decontamination log with reads_in, reads_out, reads_removed, and reads_removed_proportion fields

## How to apply

Download or cache the default human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51) with 'hostile index fetch', or let Hostile fetch it automatically. Run 'hostile clean --fastq1 <r1> --fastq2 <r2>' in paired short-read mode, which invokes Bowtie2 with its default --sensitive parameters (-D 15 -R 2 -N 0 -L 22 -i S,1,1.15) and -k 1 to align reads against the human index. Reads that align to the host index are removed from the FASTQ stream; unaligned reads are written as cleaned FASTQ.gz. Capture the JSON log (fields reads_in, reads_out, reads_removed, reads_removed_proportion) and compute bacterial retention as reads_out/reads_in — Hostile was benchmarked retaining at least 99.989% of simulated bacterial reads from 985 FDA-ARGOS genomes while removing at least 99.6% of real human reads from 1000 Genomes Project samples.

## Related tools

- **Hostile** (Wraps Bowtie2 alignment in paired short-read mode to classify and remove host-aligned reads from FASTQ input, emitting cleaned FASTQ and a JSON log) — https://github.com/bede/hostile
- **Bowtie2** (Alignment engine for short-read classification against the human-t2t-hla index with default --sensitive parameters and -k 1)

## Examples

```
hostile clean --fastq1 r1.fastq.gz --fastq2 r2.fastq.gz --index human-t2t-hla
```

## Evaluation signals

- Hostile JSON log shows reads_out/reads_in retention of at least 99.989% for simulated bacterial reads (e.g. DWGSIM reads from the 985 FDA-ARGOS assemblies)
- Retention comparison against HRRT shows Hostile removing an order of magnitude fewer microbial reads (HRRT removed a mean 21x more bacterial reads; 95% CI 9-30)
- Human read removal exceeds 99.5-99.6% on 1000 Genomes Project samples
- Cleaned FASTQ.gz files are produced without corruption, including valid (non-empty) output when no reads are removed
- Index downloads pass manifest checksum verification (verification added in the latest release, issue #20)

## Limitations

- Short-read decontamination requires ~4GB RAM (Bowtie2); long reads use Minimap2 with ~13GB RAM
- The unmasked default index removes some microbial reads; a masked index (e.g. human-t2t-hla-argos985, 317,973 masked positions, 0.010%) increases bacterial retention to at least 99.997% with a negligible (at most 0.001%) reduction in human read removal
- Hostile treats aligned reads as removed; it does not mask them in place like HRRT's 'N' replacement

## Evidence

- [methods] Remove reads aligning to an index from fastq[.gz] input files or stdin.: "Remove reads aligning to an index from fastq[.gz] input files or stdin."
- [methods] Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2): "Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)"
- [abstract] our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [readme] In typical use, Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2).: "In typical use, Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)."
