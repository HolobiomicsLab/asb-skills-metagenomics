---
name: peak-memory-profiling
description: Use when benchmarking the peak memory and throughput of host-sequence
  decontamination of bacterial FASTQ reads with Hostile (Bowtie2/Minimap2, 8 threads)
  and comparing against the NCBI HRRT (sra-human-scrubber).
when_to_use_negative:
- You need to measure read-removal accuracy (e.g. % of real human reads removed vs
  bacterial read retention), not time and memory cost
- Input data is not microbial FASTQ being host-decontaminated (e.g. BAM/CRAM, VCF,
  or already-cleaned reads)
- You are profiling tools other than Hostile/HRRT decontamination and have no comparable
  throughput baseline (32Mbp/s Illumina, 22Mbp/s ONT) to validate against
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Hostile
  role: Decontamination tool being profiled; 'hostile clean' is wrapped to record
    wall-clock time and peak RAM for short- and long-read runs
  repo: https://github.com/bede/hostile
- name: Bowtie2
  role: Short-read aligner invoked by 'hostile clean' for paired Illumina FASTQ decontamination
    at 8 alignment threads
- name: Minimap2
  role: Long-read aligner (map-ont preset, --secondary no) invoked by 'hostile clean'
    for ONT FASTQ decontamination; Hostile builds and reuses .mmi files to speed it
    up
- name: conda
  role: Installs Hostile and its non-Python dependencies (Bowtie2, Minimap2, Samtools,
    Bedtools) from bioconda for the benchmark environment
- name: sra-human-scrubber (HRRT)
  role: NCBI comparator tool whose memory usage (1–71 GB depending on host read quantity)
    provides the contrast to Hostile's stable footprint
  repo: https://github.com/ncbi/sra-human-scrubber
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:4727690036d827ae0914bef2f994173616110553d11fabf27129c71fbad9f301
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# peak-memory-profiling

## Summary

Reproduces the Hostile benchmark of wall-clock time and peak RAM for host read decontamination, comparing Hostile's stable 4GB (Bowtie2 short reads) / 13GB (Minimap2 long reads) footprint against the NCBI HRRT's variable 1–71 GB usage.

## When to use

Use when benchmarking the peak memory and throughput of host-sequence decontamination of bacterial FASTQ reads with Hostile (Bowtie2/Minimap2, 8 threads) and comparing against the NCBI HRRT (sra-human-scrubber).

## When NOT to use

- You need to measure read-removal accuracy (e.g. % of real human reads removed vs bacterial read retention), not time and memory cost
- Input data is not microbial FASTQ being host-decontaminated (e.g. BAM/CRAM, VCF, or already-cleaned reads)
- You are profiling tools other than Hostile/HRRT decontamination and have no comparable throughput baseline (32Mbp/s Illumina, 22Mbp/s ONT) to validate against

## Inputs

- Bacterial Illumina paired-end FASTQ (.fastq.gz) input
- Bacterial ONT long-read FASTQ (.fastq.gz) input
- Cached human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51) for both Bowtie2 and Minimap2
- Environment with 8 threads available and a resource-measuring wrapper (e.g. /usr/bin/time -v)

## Outputs

- Wall-clock elapsed time and peak RAM (peak RSS) per decontamination run
- Throughput in Mbp/s and reads/sec derived from total bases and read counts
- benchmark_results.csv compiling throughput and peak-memory figures for Bowtie2 short-read and Minimap2 long-read runs

## How to apply

Install Hostile with its non-Python dependencies (Bowtie2, Minimap2, Samtools, Bedtools) from bioconda via 'conda create -y -n hostile -c conda-forge -c bioconda hostile', then cache the default human-t2t-hla index ahead of time with 'hostile index fetch --bowtie2' and '--minimap2'. Wrap 'hostile clean' in a resource monitor (e.g. /usr/bin/time -v) to capture elapsed wall-clock time and peak RSS for each run. Run the paired bacterial Illumina FASTQ with the Bowtie2 aligner at '-t 8' alignment threads, and the bacterial ONT long-read FASTQ with Minimap2 (map-ont preset, --secondary no) also at '-t 8'; note that Hostile automatically allocates CPU cores between alignment and compression tasks, so report the thread flag used. Convert elapsed times into throughput in Mbp/s and reads/sec from each dataset's total bases and read counts, and compile the throughput and peak-memory figures into benchmark_results.csv. Accept the run if Illumina short-read throughput is on the order of 32Mbp/s (210k reads/sec), ONT throughput on the order of 22Mbp/s, and peak RAM lands near 4GB for Bowtie2 short reads and 13GB for Minimap2 long reads.

## Related tools

- **Hostile** (Decontamination tool being profiled; 'hostile clean' is wrapped to record wall-clock time and peak RAM for short- and long-read runs) — https://github.com/bede/hostile
- **Bowtie2** (Short-read aligner invoked by 'hostile clean' for paired Illumina FASTQ decontamination at 8 alignment threads)
- **Minimap2** (Long-read aligner (map-ont preset, --secondary no) invoked by 'hostile clean' for ONT FASTQ decontamination; Hostile builds and reuses .mmi files to speed it up)
- **conda** (Installs Hostile and its non-Python dependencies (Bowtie2, Minimap2, Samtools, Bedtools) from bioconda for the benchmark environment)
- **sra-human-scrubber (HRRT)** (NCBI comparator tool whose memory usage (1–71 GB depending on host read quantity) provides the contrast to Hostile's stable footprint) — https://github.com/ncbi/sra-human-scrubber

## Examples

```
/usr/bin/time -v hostile clean --fastq1 reads_R1.fastq.gz --fastq2 reads_R2.fastq.gz --aligner bowtie2 -t 8
```

## Evaluation signals

- Illumina short-read (Bowtie2) throughput lands on the order of the reported 32Mbp/s (210k reads/sec) with 8 alignment threads
- ONT long-read (Minimap2) throughput lands on the order of the reported 22Mbp/s with 8 alignment threads
- Peak RAM for Bowtie2 short reads falls near 4GB; peak RAM for Minimap2 long reads falls near 13GB
- Peak RSS and elapsed wall-clock time were recorded for every run via a wrapper around 'hostile clean', not read post hoc
- Throughput recomputation from total bases and read counts is consistent with the elapsed times recorded in benchmark_results.csv

## Limitations

- The 4GB/13GB figures describe typical use and depend on dataset and thread allocation; Hostile automatically allocates available CPU cores between alignment and compression tasks, which can shift timing results
- HRRT's memory usage varied between 1 and 71 GB depending on host read quantity, so the Hostile-vs-HRRT comparison is input-dependent rather than a single fixed ratio
- Hostile downloads the default human reference/index on first run unless 'hostile index fetch' was run ahead of time, which would otherwise distort timing of the first 'hostile clean' call

## Evidence

- [methods] Bacterial Illumina throughput benchmark: "bacterial Illumina reads were decontaminated at 32Mbp/s (210k reads/sec) and bacterial ONT reads at 22Mbp/s, using 8 alignment threads"
- [readme] Hostile memory footprint: "In typical use, Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)."
- [methods] Benchmark environment installation: "conda create -y -n hostile -c conda-forge -c bioconda hostile"
- [readme] Non-Python dependencies to install before profiling: "Installation with conda/mamba or Docker is recommended due to non-Python dependencies (Bowtie2, Minimap2, Samtools and Bedtools)."
