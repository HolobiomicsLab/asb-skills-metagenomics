---
name: alignment-throughput-measurement
description: Use when comparing resource footprints (peak memory, throughput) of read-decontamination
  or alignment pipelines across tools, e.g. Hostile vs HRRT on Illumina and ONT FASTQ
  inputs.
when_to_use_negative:
- The goal is accuracy evaluation (human read removal rate or bacterial read retention),
  not resource benchmarking — use the paper's accuracy evaluation workflow instead
- The aligner or decontamination tool does not support streaming FASTQ input or thread-count
  control, making wall-clock and peak-RAM comparison non-comparable
- The dataset is not microbial host-contaminated reads (e.g. pure human samples),
  where throughput-to-decontamination mapping is meaningless
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Hostile
  role: Decontamination tool under benchmark; run via 'hostile clean' with 8 alignment
    threads to measure time and peak RAM
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: Short-read (Illumina) aligner invoked by Hostile; its RAM footprint (~4GB)
    is one benchmark target
- name: Minimap2
  role: Long-read (ONT) aligner invoked by Hostile with map-ont preset; its RAM footprint
    (~13GB) is one benchmark target
- name: sra-human-scrubber (HRRT)
  role: Comparison tool whose k-mer database loading and variable memory usage (1–71
    GB) are measured alongside Hostile
  repo: https://github.com/ncbi/sra-human-scrubber
- name: conda
  role: Installation channel used to resolve Hostile and its non-Python dependencies
    reproducibly
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:77946105c0608d1fac77a057e4648cd0af08fe7b8ecb697e68715aada5e00c84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bowtie2
  - Hostile
  - Minimap2
  - conda
  - sra-human-scrubber (HRRT)
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# alignment-throughput-measurement

## Summary

Benchmarking skill for measuring wall-clock throughput (Mbp/s, reads/sec) and peak RAM of host-read decontamination aligners (Hostile with Bowtie2/Minimap2 vs NCBI HRRT) on short and long microbial reads.

## When to use

Use when comparing resource footprints (peak memory, throughput) of read-decontamination or alignment pipelines across tools, e.g. Hostile vs HRRT on Illumina and ONT FASTQ inputs.

## When NOT to use

- The goal is accuracy evaluation (human read removal rate or bacterial read retention), not resource benchmarking — use the paper's accuracy evaluation workflow instead
- The aligner or decontamination tool does not support streaming FASTQ input or thread-count control, making wall-clock and peak-RAM comparison non-comparable
- The dataset is not microbial host-contaminated reads (e.g. pure human samples), where throughput-to-decontamination mapping is meaningless

## Inputs

- Bacterial Illumina paired FASTQ (.fastq.gz) short-read dataset
- Bacterial ONT long-read FASTQ (.fastq.gz) dataset
- Hostile default human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51), Bowtie2 and Minimap2 variants
- Total base counts and read counts per dataset (for throughput conversion)

## Outputs

- benchmark_results.csv with per-run wall-clock time, throughput (Mbp/s, reads/sec), and peak RAM (GB)
- Peak-memory comparison table for Hostile (Bowtie2 short reads, Minimap2 long reads) vs HRRT

## How to apply

Install Hostile via conda from bioconda so its non-Python dependencies (Bowtie2, Minimap2, Samtools, Bedtools) are resolved, then cache the default human-t2t-hla index with 'hostile index fetch --minimap2' and '--bowtie2'. Wrap 'hostile clean' runs so that wall-clock time and peak RAM are recorded: run the bacterial Illumina paired FASTQ with the Bowtie2 aligner and '-t 8' alignment threads, and the bacterial ONT long-read FASTQ with Minimap2 (map-ont preset, --secondary no) and '-t 8'. Convert elapsed times into throughput (Mbp/s and reads/sec) using each dataset's total bases and read counts, and compile figures into benchmark_results.csv. Repeat the same measurement protocol for the NCBI HRRT (aligns_to via scrub.sh), which loads a Eukaryota-derived k-mer database, to contrast its variable memory usage with Hostile's stable footprint.

## Related tools

- **Hostile** (Decontamination tool under benchmark; run via 'hostile clean' with 8 alignment threads to measure time and peak RAM) — https://github.com/bede/hostile
- **Bowtie2** (Short-read (Illumina) aligner invoked by Hostile; its RAM footprint (~4GB) is one benchmark target)
- **Minimap2** (Long-read (ONT) aligner invoked by Hostile with map-ont preset; its RAM footprint (~13GB) is one benchmark target)
- **sra-human-scrubber (HRRT)** (Comparison tool whose k-mer database loading and variable memory usage (1–71 GB) are measured alongside Hostile) — https://github.com/ncbi/sra-human-scrubber
- **conda** (Installation channel used to resolve Hostile and its non-Python dependencies reproducibly)

## Examples

```
hostile clean --fastq1 long.fastq.gz --minimap2 map-ont --secondary no -t 8  # wrap with /usr/bin/time -v to record wall-clock time and peak RAM
```

## Evaluation signals

- Illumina short-read throughput is on the order of the reported 32Mbp/s (210k reads/sec) with 8 alignment threads
- Bacterial ONT throughput is on the order of 22Mbp/s
- Peak RAM falls near 4GB for Bowtie2 short reads and near 13GB for Minimap2 long reads
- HRRT memory usage varies with host read quantity (expected range roughly 1–71 GB), contrasting with Hostile's stable footprint
- Output benchmark_results.csv contains throughput and peak-memory figures for both tools on both read types

## Limitations

- Hostile memory figures are for 'typical use'; the benchmark depends on 8 alignment threads and Hostile's automatic allocation of CPU cores between alignment and compression tasks
- Earlier Hostile versions did not verify downloaded index checksums and could corrupt empty fastq.gz files when no reads remained after decontamination
- HRRT by default masks identified human reads with 'N' rather than removing them, so throughput comparisons measure different operations unless -x (remove spots) is used
- HRRT binary aligns_to is compiled only for x86_64 GNU/Linux, constraining where the comparison can run

## Evidence

- [methods] Illumina throughput 32 Mbp/s: "bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads"
- [methods] RAM requirements: "Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)"
- [readme] installation: "Installation with conda/mamba or Docker is recommended due to non-Python dependencies (Bowtie2, Minimap2, Samtools and Bedtools)."
- [methods] index fetch: "To optionally download and cache the default index (`human-t2t-hla`) ahead of time, run `hostile index fetch`."
- [readme] HRRT description: [evidence span withheld — anchor status: not_found]
