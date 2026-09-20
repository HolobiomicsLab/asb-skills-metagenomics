---
name: minimap2-long-read-alignment
description: Use when decontaminating Oxford Nanopore (ONT) long reads from (meta)genomic
  FASTQ input against a human reference index such as human-t2t-hla, or when benchmarking
  long-read host depletion against tools like NCBI's HRRT.
when_to_use_negative:
- Input is paired Illumina short-read data — Hostile auto-selects Bowtie2 (--sensitive,
  -k 1) instead of Minimap2 in that case
- Goal is exhaustive host read removal rather than precise microbial retention — HRRT
  masks more aggressively but removes 21-43 times more bacterial reads
- Reference is mouse (mouse-mm39) or a custom non-human genome requiring different
  index construction
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Hostile
  role: Wraps Minimap2 in a streaming long-read decontamination workflow, handling
    index caching, .mmi building, thread allocation, and JSON logging
  repo: https://github.com/bede/hostile
- name: Minimap2
  role: Long-read aligner invoked with the map-ont preset and secondary alignments
    disabled to identify host-derived reads
- name: Samtools
  role: Counts distinct reads in the alignment stream so aligned reads and their mates
    can be discarded
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_008
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:4fc9f94b66505b92c8caa013cc05f1d6283b1f0166382fef856e8614d30a021b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - DWGSIM
  - HRRT
  - Hostile
  - Minimap2
  - PBSIM2
  - Samtools
  - conda
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# minimap2-long-read-alignment

## Summary

Applies Minimap2 with the map-ont preset (secondary alignments disabled) as the long-read aligner inside Hostile's streaming host-read decontamination pipeline, aligning single-end ONT FASTQ reads against a human reference index so aligned reads can be counted with Samtools and discarded.

## When to use

Use when decontaminating Oxford Nanopore (ONT) long reads from (meta)genomic FASTQ input against a human reference index such as human-t2t-hla, or when benchmarking long-read host depletion against tools like NCBI's HRRT.

## When NOT to use

- Input is paired Illumina short-read data — Hostile auto-selects Bowtie2 (--sensitive, -k 1) instead of Minimap2 in that case
- Goal is exhaustive host read removal rather than precise microbial retention — HRRT masks more aggressively but removes 21-43 times more bacterial reads
- Reference is mouse (mouse-mm39) or a custom non-human genome requiring different index construction

## Inputs

- Single-end ONT long-read FASTQ file (.fastq or .fastq.gz), passed via --fastq1
- Host reference index (e.g. human-t2t-hla, human-t2t-hla-argos985-mycob140) fetched with 'hostile index fetch' or downloaded automatically

## Outputs

- Gzip-compressed cleaned FASTQ named <input>.clean.fastq.gz
- JSON log with version, aligner, index, fastq1_in_name, reads_in, reads_out, reads_removed, reads_removed_proportion

## How to apply

Run Hostile in long-read mode by passing only --fastq1, which auto-selects Minimap2 with the -x map-ont preset and secondary alignments disabled (--secondary no) against the default human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51) or a masked index like human-t2t-hla-argos985-mycob140. Hostile streams the gzip-compressed FASTQ through alignment, counts distinct reads with Samtools, and discards aligned reads; it automatically builds and reuses .mmi files to speed up repeated Minimap2 runs and allocates available CPU cores between alignment and compression. Plan resources accordingly: Minimap2 long-read mode requires 13GB of RAM (versus 4GB for Bowtie2 short reads), and bacterial ONT reads were decontaminated at 22Mbp/s using 8 alignment threads. Capture the emitted JSON log (reads_in, reads_out, reads_removed, reads_removed_proportion) to compute retention; in benchmarks this configuration retained 99.98918% of simulated bacterial ONT reads and removed only 0.038% of real ONT human reads.

## Related tools

- **Hostile** (Wraps Minimap2 in a streaming long-read decontamination workflow, handling index caching, .mmi building, thread allocation, and JSON logging) — https://github.com/bede/hostile
- **Minimap2** (Long-read aligner invoked with the map-ont preset and secondary alignments disabled to identify host-derived reads)
- **Samtools** (Counts distinct reads in the alignment stream so aligned reads and their mates can be discarded)

## Examples

```
hostile clean --fastq1 long.fastq.gz --index human-t2t-hla  # Minimap2 map-ont, creates long.clean.fastq.gz + JSON log
```

## Evaluation signals

- JSON log parses and satisfies reads_in = reads_out + reads_removed with reads_removed_proportion in [0, 1]
- Simulated bacterial ONT read retention ≥ 99.989% (benchmark: 99.98918% on 985 FDA-ARGOS assemblies, 891 reads removed)
- Real human ONT read removal ≈ 0.038% retained (Hostile) vs 0.037% (HRRT) on ONT data
- Reused .mmi file is built/cached so subsequent long-read runs skip index rebuilding
- Masked index human-t2t-hla-argos985-mycob140 shifts ONT human retention only marginally (0.038029% to 0.038069%)

## Limitations

- Minimap2 long-read mode needs 13GB of RAM, which may exceed laptop constraints compared with 4GB for Bowtie2 short reads
- Hostile aims for precision — removing an order of magnitude fewer microbial reads — so it removes fewer human reads than k-mer-based tools like HRRT in some settings
- Earlier Hostile versions did not verify downloaded index checksums (fixed in latest release, issue #20)

## Evidence

- [methods] Long-read ONT alignment parameters: "Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)"
- [methods] Index composition: "The default index `human-t2t-hla` comprises [T2T-CHM13v2.0](https://www.ncbi.nlm.nih.gov/assembly/11828891) and [IPD-IMGT/HLA](https://www.ebi.ac.uk/ipd/imgt/hla/) v3.51"
- [methods] Automatic .mmi caching: "Hostile automatically builds and reuses .mmi files to speed up long read decontamination with Minimap2."
- [abstract] ONT benchmark retention: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [readme] README long-read invocation and throughput: "# Long reads
hostile clean --fastq1 long.fastq.gz  # Creates long.clean.fastq.gz"
