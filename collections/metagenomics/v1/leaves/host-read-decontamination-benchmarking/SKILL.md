---
name: host-read-decontamination-benchmarking
description: Use when microbial FASTQ (short-read Illumina or ONT long-read) data
  must be decontaminated of human host sequences, or when comparing Hostile vs HRRT
  on human removal, microbial retention, throughput, and memory.
when_to_use_negative:
- The input is already a cleaned FASTQ or you need to keep host reads for human genetics
  analysis without an explicit --invert workflow
- You need taxonomic classification of reads rather than binary host/non-host depletion
  — Hostile only discards reads aligning to the index
- You require viral read retention but use the unmasked default index instead of human-t2t-hla.argos-bacteria-985_rs-viral-202401_ml-phage-202401
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Hostile
  role: Primary decontamination tool; aligns FASTQ reads to a human index with Bowtie2
    (short) or Minimap2 (long) and discards aligned reads and mates, emitting JSON
    logs
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: Aligner for paired short-read mode under the --sensitive preset (-D 15 -R
    2 -N 0 -L 22 -i S,1,1.15) with -k 1 and no secondary alignments
- name: Minimap2
  role: Aligner for long-read ONT mode using the map-ont preset with secondary alignments
    disabled; also used in hostile mask to align 150mers for index masking
- name: Samtools
  role: Counts distinct reads during streaming decontamination
- name: Bedtools
  role: Masks aligned 150mer regions with N when building custom masked indexes via
    hostile mask
- name: sra-human-scrubber (HRRT)
  role: NCBI comparator tool using an Eukaryota-derived k-mer database; masks or removes
    reads identified as potentially human
  repo: https://github.com/ncbi/sra-human-scrubber
- name: DWGSIM
  role: Simulates paired Illumina reads from FDA-ARGOS bacterial and mycobacterial
    assemblies for retention benchmarking
- name: PBSIM2
  role: Simulates ONT long reads from the same assemblies for ONT retention benchmarking
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  - task_001
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:cc660963fb262c11624ab4ec57bddfee45d7bf3735911e2e364a3308b0da7937
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bedtools
  - Bowtie2
  - DWGSIM
  - HRRT
  - Hostile
  - Minimap2
  - PBSIM2
  - Samtools
  - conda
  - sra-human-scrubber (HRRT)
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# host-read-decontamination-benchmarking

## Summary

A benchmarking skill for Hostile, an alignment-based host read decontamination tool, against the NCBI Human Read Removal Tool (HRRT), quantifying human read removal and simulated bacterial/mycobacterial read retention on Illumina and ONT FASTQ data.

## When to use

Use when microbial FASTQ (short-read Illumina or ONT long-read) data must be decontaminated of human host sequences, or when comparing Hostile vs HRRT on human removal, microbial retention, throughput, and memory.

## When NOT to use

- The input is already a cleaned FASTQ or you need to keep host reads for human genetics analysis without an explicit --invert workflow
- You need taxonomic classification of reads rather than binary host/non-host depletion — Hostile only discards reads aligning to the index
- You require viral read retention but use the unmasked default index instead of human-t2t-hla.argos-bacteria-985_rs-viral-202401_ml-phage-202401

## Inputs

- Single-end ONT FASTQ or FASTQ.GZ files (e.g. NA12878 long reads, simulated PBSIM2 ONT reads)
- Paired Illumina FASTQ or FASTQ.GZ files (2x100bp or 2x150bp, e.g. 1000 Genomes 30x genomes, DWGSIM-simulated reads)
- Human reference index (default human-t2t-hla: T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51, or masked variants)
- Reference genomes for simulation: 985 FDA-ARGOS bacterial genomes (BioProject 231221) and 140 complete mycobacterial assemblies
- HRRT v2.1.0 k-mer database fetched via init_db.sh for the comparator run

## Outputs

- Host-cleaned FASTQ files named <input>.clean.fastq.gz
- Per-run JSON logs with version, aligner, index, reads_in, reads_out, reads_removed, reads_removed_proportion
- HRRT output fastq.clean files with masked (N) or removed human spots
- Comparison tables of human retention and bacterial/mycobacterial retention (Hostile vs HRRT, e.g. bacterial_retention_comparison.csv, masked_retention_results.csv)
- Benchmark metrics: throughput in Mbp/s and reads/sec, peak RAM per aligner

## How to apply

Install Hostile with its non-Python dependencies (Bowtie2, Minimap2, Samtools, Bedtools) via bioconda (`conda create -y -n hostile -c conda-forge -c bioconda hostile`) and cache the default `human-t2t-hla` index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51) with `hostile index fetch`. Run `hostile clean` in paired short-read mode (Bowtie2 --sensitive, -k 1, no secondary alignments) or single-end long-read mode (Minimap2 map-ont preset, --secondary no), parsing the emitted JSON log fields reads_in, reads_out, reads_removed, and reads_removed_proportion. Benchmark retention by decontaminating simulated bacterial reads (DWGSIM Illumina, PBSIM2 ONT) from 985 FDA-ARGOS assemblies (BioProject 231221) and 140 mycobacterial assemblies, computing retention as reads_out/reads_in, and run HRRT v2.1.0 (ncbi/sra-human-scrubber) on the identical read sets as the comparator. For maximum microbial retention, rerun against a masked index such as `human-t2t-hla-argos985-mycob140`, created by masking the human reference with 150mers via `hostile mask` (Minimap2 alignment + Bedtools N-masking). Validate against reported figures: ≥99.6% removal of real human reads, ≥99.989% retention of simulated bacterial reads, 32Mbp/s Illumina and 22Mbp/s ONT throughput with 8 alignment threads, and 4GB (Bowtie2) / 13GB (Minimap2) peak RAM.

## Related tools

- **Hostile** (Primary decontamination tool; aligns FASTQ reads to a human index with Bowtie2 (short) or Minimap2 (long) and discards aligned reads and mates, emitting JSON logs) — https://github.com/bede/hostile
- **Bowtie2** (Aligner for paired short-read mode under the --sensitive preset (-D 15 -R 2 -N 0 -L 22 -i S,1,1.15) with -k 1 and no secondary alignments)
- **Minimap2** (Aligner for long-read ONT mode using the map-ont preset with secondary alignments disabled; also used in hostile mask to align 150mers for index masking)
- **Samtools** (Counts distinct reads during streaming decontamination)
- **Bedtools** (Masks aligned 150mer regions with N when building custom masked indexes via hostile mask)
- **sra-human-scrubber (HRRT)** (NCBI comparator tool using an Eukaryota-derived k-mer database; masks or removes reads identified as potentially human) — https://github.com/ncbi/sra-human-scrubber
- **DWGSIM** (Simulates paired Illumina reads from FDA-ARGOS bacterial and mycobacterial assemblies for retention benchmarking)
- **PBSIM2** (Simulates ONT long reads from the same assemblies for ONT retention benchmarking)

## Examples

```
hostile clean --fastq1 r1.fastq.gz --fastq2 r2.fastq.gz --index human-t2t-hla-argos985-mycob140 --threads 8
```

## Evaluation signals

- JSON log invariant: reads_in equals reads_out plus reads_removed, and reads_removed_proportion is within [0, 1]
- Human retention targets: Hostile 0.132% vs HRRT 0.160% at 10x Illumina; Hostile 0.594% vs HRRT 0.729% at 30x Illumina; Hostile 0.038% vs HRRT 0.037% on ONT reads
- Simulated bacterial retention: Hostile ≥99.989% (Illumina 99.99989%, ONT 99.98918% with 891 reads removed); masked index reaches 99.99994% with exactly 251 ONT bacterial reads removed
- Throughput and memory: Illumina ~32Mbp/s (210k reads/sec) and ONT ~22Mbp/s at 8 alignment threads; peak RAM ~4GB (Bowtie2) and ~13GB (Minimap2)
- For the bundled tuberculosis_1_1.fastq.gz test sample, expect reads_removed_proportion of 0.0 with reads_out equal to reads_in

## Limitations

- HRRT's memory usage varied between 1 and 71 GB depending on host read quantity, so large-scale comparator runs need careful resource planning
- Masked indexes trade a small amount of human removal performance (≤0.001% reduction) for microbial retention; human retention changed marginally from 0.131738% to 0.131979% (Illumina) and 0.038029% to 0.038069% (ONT)
- Earlier Hostile versions could corrupt empty fastq.gz files when no reads remain after decontamination and did not verify downloaded index checksums (issues #20, #24)
- The test-human-mit index (human mitochondrial genome NC_012920.1) is for testing purposes only, not real decontamination

## Evidence

- [abstract] abstract: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [methods] methods: "Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)"
- [readme] readme: "bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads"
- [discussion] discussion: "Prevents corruption of empty `fastq.gz` files when no reads remain after decontamination. (#24)"
