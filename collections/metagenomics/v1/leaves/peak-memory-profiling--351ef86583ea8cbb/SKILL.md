---
name: peak-memory-profiling
description: Use when comparing runtime and RAM footprint of taxonomic classifiers
  or similar read-processing tools on a fixed simulated read set, to decide which
  tool fits available hardware.
when_to_use_negative:
- The goal is classification accuracy (sensitivity/precision) rather than resource
  consumption — use the genome-exclusion benchmark workflow instead
- Input is a real metagenome sample where read counts vary per run, making reads-per-second
  throughput comparisons across tools non-comparable
- Only a single tool is being profiled with no cross-tool comparison target, so a
  full benchmark table is unnecessary
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Classifier under test, run in MEM mode (-a mem) and Greedy mode with 1 and
    5 allowed mismatches (-e) using 25 threads (-z 25); its BWT/FM-index design keeps
    memory below 6 GB
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken
  role: k-mer-based classifier baseline (v0.10.4b, default mode, k=31) whose ~72 GB
    memory footprint is contrasted with Kaiju
- name: Clark
  role: k-mer-based classifier baseline (v1.1.3, default mode, k=31) requiring 65-78
    GB peak memory in the benchmark
- name: ART
  role: Read simulator used to generate the five simulated benchmark read sets from
    the reference genomes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:cf56d153f484eb6c73d73731cc7846a9226076c4baa86c7027116e2eca8463b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Clark
  - Kaiju
  - Kraken
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# peak-memory-profiling

## Summary

Benchmarking skill for measuring classification throughput (reads per second) and peak resident memory of metagenome classifiers — here Kaiju (MEM, Greedy-1, Greedy-5), Kraken v0.10.4b and Clark v1.1.3 — across read types on a fixed 27.24M-read benchmark dataset with 25 parallel threads, reproducing Fig. 4 of Menzel et al. 2016.

## When to use

Use when comparing runtime and RAM footprint of taxonomic classifiers or similar read-processing tools on a fixed simulated read set, to decide which tool fits available hardware.

## When NOT to use

- The goal is classification accuracy (sensitivity/precision) rather than resource consumption — use the genome-exclusion benchmark workflow instead
- Input is a real metagenome sample where read counts vary per run, making reads-per-second throughput comparisons across tools non-comparable
- Only a single tool is being profiled with no cross-tool comparison target, so a full benchmark table is unnecessary

## Inputs

- Five simulated HTS benchmark read sets (27.24M reads total, e.g. single-end and paired-end 100 nt/250 nt generated with ART)
- Kaiju reference protein database index (.fmi, BWT + FM-index)
- NCBI taxonomy files nodes.dmp and names.dmp
- Kraken v0.10.4b and Clark v1.1.3 k=31 indexes built on the same reference genomes

## Outputs

- benchmark_results.tsv tabulating reads-per-second throughput and peak resident memory for each tool (Kaiju MEM/Greedy-1/Greedy-5, Kraken k=31, Clark k=31), run mode, and read type
- Per-run classification output files (tab-separated, via kaiju -o)

## How to apply

Compile Kaiju from source (cd kaiju/src; make) and build the reference protein database index (Burrows-Wheeler transform and FM-index) together with NCBI taxonomy files nodes.dmp and names.dmp. Run Kaiju in MEM mode (-a mem) and in Greedy mode with 1 and 5 allowed mismatches (-e) on each of the five simulated read sets using 25 parallel threads (-z 25), writing tab-separated output with -o; run Kraken (v0.10.4b) and Clark (v1.1.3) in default modes with k=31 for highest precision on the same reads. Wall-clock each run on the 27.24M-read dataset to compute reads-per-second throughput and record the peak resident memory for every tool/mode combination. Tabulate throughput and memory per tool, mode, and read type and save as benchmark_results.tsv. Rationale: Kaiju's BWT/FM-index protein-level search is expected to stay below 6 GB and process millions of reads per minute, whereas the k-mer indexes of Kraken and Clark demand tens of GB.

## Related tools

- **Kaiju** (Classifier under test, run in MEM mode (-a mem) and Greedy mode with 1 and 5 allowed mismatches (-e) using 25 threads (-z 25); its BWT/FM-index design keeps memory below 6 GB) — https://github.com/bioinformatics-centre/kaiju
- **Kraken** (k-mer-based classifier baseline (v0.10.4b, default mode, k=31) whose ~72 GB memory footprint is contrasted with Kaiju)
- **Clark** (k-mer-based classifier baseline (v1.1.3, default mode, k=31) requiring 65-78 GB peak memory in the benchmark)
- **ART** (Read simulator used to generate the five simulated benchmark read sets from the reference genomes)

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i inputfile.fastq -o kaiju.out
```

## Evaluation signals

- Kaiju peak memory falls below 6 GB on the benchmark database, versus ~72 GB for Kraken and 65-78 GB for Clark
- Kaiju MEM reaches ~173k reads/s on single-end 100 nt reads (vs Kraken 165k, Clark 93k), the fastest short-read configuration
- Paired-end 250 nt is the slowest configuration for all tools: MEM 97k, Kraken 24k, Clark 19k reads/s
- Throughput scales with read length and number of allowed mismatches (Greedy-5 slower than Greedy-1, which is slower than MEM)
- benchmark_results.tsv contains one row per tool/mode/read-type combination across all five read types with both r.p.s. and peak RAM columns populated

## Limitations

- Memory and speed depend on the chosen reference database; the reported footprint (below 6 GB) applies to the paper's benchmark database, while current prebuilt indexes such as refseq require ~111 GB RAM
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches, so Greedy-5 timings will not extrapolate to higher mismatch counts
- Measured values are hardware- and thread-count-dependent (25 parallel threads in the paper); results are not directly transferable to different machines

## Evidence

- [discussion] Kaiju's memory footprint small enough (below 6 GB on our benchmark database) for running it on a standard PC: "Kaiju's memory footprint small enough (below 6 GB on our benchmark database) for running it on a standard PC"
- [discussion] Kaiju fast enough for millions of reads per minute, faster than k-mer methods: "Kaiju is fast enough for classifying up to millions of reads per minute, depending on the read length and the number of allowed mismatches"
- [discussion] Kraken v0.10.4b and Clark v1.1.3 run in default modes with k=31 for highest precision: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [methods] Compiling Kaiju from source: cd kaiju/src; make: "For compiling Kaiju and its associated programs, type:
``ncd kaiju/src
make
"
- [readme] Kaiju can use multiple parallel threads, which can be specified with the -z option, e.g. for using 25 parallel threads: "Kaiju can use multiple parallel threads, which can be specified with the `-z` option, e.g. for using 25 parallel threads"
