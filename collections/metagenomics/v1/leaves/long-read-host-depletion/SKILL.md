---
name: long-read-host-depletion
description: Use when the input is a single-end ONT long-read FASTQ(.gz) from a microbial
  sample contaminated with human host sequence, and reads aligning to the host genome
  must be removed or the resource cost benchmarked.
when_to_use_negative:
- Input is paired Illumina short-read data — use the Bowtie2-based short-read decontamination
  path instead (~4GB RAM, 32Mbp/s), not the Minimap2 long-read path
- The goal is to mask host reads with 'N' in place (HRRT/sra-human-scrubber behavior)
  rather than remove them from the FASTQ
- The task is host read removal for a non-human host — a custom index (e.g. mouse-mm39/GRCm39)
  must be used rather than the default human-t2t-hla
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: Hostile
  role: Main decontamination tool; `hostile clean` aligns long reads to the host index
    and removes aligned reads
  repo: github.com/bede/hostile
- name: Minimap2
  role: Long-read aligner invoked by Hostile with the map-ont preset and --secondary
    no for host read identification
- name: conda
  role: Recommended installation channel for Hostile and its non-Python dependencies
    from bioconda
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:4f251d0598c2110c041ee578631396f735c96fa9dac4a1b120892a6d78264e7b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Hostile
  - Minimap2
  - conda
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# long-read-host-depletion

## Summary

Depletes human host reads from single-end long-read (ONT) FASTQ metagenomes using Hostile's Minimap2-based alignment pipeline against the human-t2t-hla index, and benchmarks throughput and peak memory of the run.

## When to use

Use when the input is a single-end ONT long-read FASTQ(.gz) from a microbial sample contaminated with human host sequence, and reads aligning to the host genome must be removed or the resource cost benchmarked.

## When NOT to use

- Input is paired Illumina short-read data — use the Bowtie2-based short-read decontamination path instead (~4GB RAM, 32Mbp/s), not the Minimap2 long-read path
- The goal is to mask host reads with 'N' in place (HRRT/sra-human-scrubber behavior) rather than remove them from the FASTQ
- The task is host read removal for a non-human host — a custom index (e.g. mouse-mm39/GRCm39) must be used rather than the default human-t2t-hla

## Inputs

- Single-end ONT long-read FASTQ or fastq.gz file from a microbial (meta)genomic sample
- Host reference index (default human-t2t-hla: T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51; optionally a masked variant)
- Optionally, dataset metadata (total bases and read counts) for throughput normalization

## Outputs

- Host-depleted FASTQ file (e.g. long.clean.fastq.gz) with reads aligning to the host index removed
- Optional run metrics: wall-clock time, peak RAM (~13GB expected for Minimap2 long reads), and throughput in Mbp/s and reads/sec

## How to apply

Install Hostile with its non-Python dependencies (Bowtie2, Minimap2, Samtools, Bedtools) via conda from bioconda (`conda create -y -n hostile -c conda-forge -c bioconda hostile`), then pre-cache the default human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51) with `hostile index fetch --minimap2`. Run `hostile clean --fastq1 <long.fastq.gz>` with the Minimap2 long-read aligner using the map-ont preset and `--secondary no`, allocating 8 alignment threads with `-t 8`; Hostile automatically builds and reuses `.mmi` files to speed up subsequent Minimap2 runs. The tool consumes FASTQ from files or stdin, drops reads aligning to the host index, and writes a cleaned FASTQ (e.g. `long.clean.fastq.gz`). Optionally specify a masked index (e.g. human-t2t-hla variants masked with 150mers against bacterial/viral/phage genomes) to maximize microbial retention. Wrap the run to record wall-clock time and peak RAM, then normalize to throughput in Mbp/s and reads/sec using the dataset's total bases and read count.

## Related tools

- **Hostile** (Main decontamination tool; `hostile clean` aligns long reads to the host index and removes aligned reads) — github.com/bede/hostile
- **Minimap2** (Long-read aligner invoked by Hostile with the map-ont preset and --secondary no for host read identification)
- **conda** (Recommended installation channel for Hostile and its non-Python dependencies from bioconda)

## Examples

```
hostile clean --fastq1 long.fastq.gz --index human-t2t-hla --aligner minimap2 -t 8
```

## Evaluation signals

- ONT long-read throughput should be on the order of the reported 22Mbp/s with 8 alignment threads
- Peak RAM should fall near 13GB for the Minimap2 long-read path (vs 4GB for Bowtie2 short reads)
- Output FASTQ should be a valid non-empty gzipped FASTQ with no host-aligned reads retained (no truncation/corruption of empty outputs)
- A .mmi file should be built on first run and reused on subsequent runs, visibly reducing Minimap2 startup cost
- With masked indexes, simulated bacterial read retention should be ≥99.997% with ≤0.001% reduction in human read removal

## Limitations

- Long-read decontamination requires ~13GB RAM — substantially more than the 4GB needed for Bowtie2 short reads, so memory-constrained laptops may not suffice
- Earlier Hostile versions could corrupt empty fastq.gz files when no reads remained after decontamination (fixed in #24), and did not verify downloaded index checksums (verification added in #20)
- Throughput/memory figures are benchmarks for bacterial Illumina/ONT reads with 8 alignment threads; other read compositions or thread counts will differ
- HRRT comparison: NCBI HRRT's memory usage varied between 1 and 71 GB depending on host read quantity, and it masks reads with 'N' rather than removing them

## Evidence

- [intro] Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin.: "Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin."
- [methods] Minimap2 .mmi auto-build: "Hostile automatically builds and reuses .mmi files to speed up long read decontamination with Minimap2."
- [methods] Throughput benchmark: "bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads"
- [readme] Memory requirement: "In typical use, Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)."
- [discussion] Empty output corruption caveat: "Prevents corruption of empty `fastq.gz` files when no reads remain after decontamination."
