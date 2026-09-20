---
name: classifier-throughput-benchmarking
description: 'Use when comparing taxonomic classifiers on speed and memory: you have
  identical simulated FASTQ read sets (multiple read lengths, single/paired-end),
  25 threads available, and need r.p.s. plus peak-RAM figures per tool/mode.'
when_to_use_negative:
- The goal is classification accuracy/sensitivity rather than runtime and memory —
  the genome-exclusion benchmark in this paper addresses that separately
- Reads must be classified against a database containing the source genomes (nucleotide-level
  classifiers would be more sensitive there); this skill only measures speed/memory
- Only a small read set is available, since wall-clock timing of millions of reads
  (27.24M here) is needed for stable r.p.s. estimates
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Classifier under test, run in MEM mode (-a mem) and Greedy mode with 1 and
    5 allowed mismatches (-e), 25 threads (-z 25); compiled from source and indexed
    with BWT/FM-index
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken
  role: k-mer-based comparator classifier, v0.10.4b, run in default mode with k=31
    for highest precision on the same benchmark reads
- name: Clark
  role: k-mer-based comparator classifier, v1.1.3, run in default mode with k=31 for
    highest precision on the same benchmark reads
- name: ART
  role: Simulates the HTS benchmark reads from whole genomes (including plasmids)
    that feed the runtime benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:b681ac95d5e390bbf826d8f83347f160a9a18eb37f2cb2f65f119d109b74d308
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Clark
  - Kaiju
  - Kraken
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# classifier-throughput-benchmarking

## Summary

Benchmark the runtime throughput (reads per second) and peak resident memory of metagenomic classifiers — Kaiju (MEM and Greedy-1/Greedy-5), Kraken v0.10.4b (k=31) and Clark v1.1.3 (k=31) — on a large simulated read set (27.24M reads) to quantify the speed/memory trade-off between protein-level BWT search and k-mer hash/classifier indexes.

## When to use

Use when comparing taxonomic classifiers on speed and memory: you have identical simulated FASTQ read sets (multiple read lengths, single/paired-end), 25 threads available, and need r.p.s. plus peak-RAM figures per tool/mode.

## When NOT to use

- The goal is classification accuracy/sensitivity rather than runtime and memory — the genome-exclusion benchmark in this paper addresses that separately
- Reads must be classified against a database containing the source genomes (nucleotide-level classifiers would be more sensitive there); this skill only measures speed/memory
- Only a small read set is available, since wall-clock timing of millions of reads (27.24M here) is needed for stable r.p.s. estimates

## Inputs

- 27.24M-read simulated benchmark dataset in FASTQ covering five read types (single-end and paired-end, short and long reads, e.g. 100 nt and 2x250 nt)
- Kaiju reference protein database index (.fmi, BWT + FM-index) built from reference proteins
- NCBI taxonomy files nodes.dmp and names.dmp
- Installed Kraken v0.10.4b and Clark v1.1.3 databases run in default modes with k=31

## Outputs

- Per-run classification output files (tab-separated, e.g. Kaiju's C/U + read name + taxon ID via -o)
- Throughput table in reads per second for every tool, run mode, and read type
- Peak resident memory (GB) per tool/mode combination
- Summary file, e.g. benchmark_results.tsv

## How to apply

Compile Kaiju from source (cd kaiju/src; make, requiring libz-dev) and build the reference protein index (BWT and FM-index via kaiju-mkbwt/kaiju-mkfmi or kaiju-makedb) together with NCBI taxonomy files nodes.dmp and names.dmp. Run Kaiju in MEM mode (-a mem) and in Greedy mode with 1 and 5 allowed mismatches (-e) on each of the five simulated read sets with 25 parallel threads (-z 25), writing tab-separated output with -o; run Kraken (v0.10.4b) and Clark (v1.1.3) in their default modes with k=31 for highest precision on the same reads. Wall-clock time each run to compute throughput in reads per second (total reads / elapsed seconds), and record the peak resident set size per tool/mode/read-type combination. Tabulate all measurements, e.g. as benchmark_results.tsv, and check them against the reported envelope: Kaiju MEM fastest on short reads (~173k r.p.s. for 100 nt single-end vs Kraken 165k and Clark 93k), paired-end 250 nt slowest (MEM 97k, Kraken 24k, Clark 19k r.p.s.), with peak memory of ~5.6 GB for Kaiju versus 72 GB (Kraken) and 65-78 GB (Clark). Rationale: Kaiju's BWT-based protein-level search keeps memory below 6 GB on the benchmark database, while k-mer-based methods trade far larger indexes for their lookup speed.

## Related tools

- **Kaiju** (Classifier under test, run in MEM mode (-a mem) and Greedy mode with 1 and 5 allowed mismatches (-e), 25 threads (-z 25); compiled from source and indexed with BWT/FM-index) — https://github.com/bioinformatics-centre/kaiju
- **Kraken** (k-mer-based comparator classifier, v0.10.4b, run in default mode with k=31 for highest precision on the same benchmark reads)
- **Clark** (k-mer-based comparator classifier, v1.1.3, run in default mode with k=31 for highest precision on the same benchmark reads)
- **ART** (Simulates the HTS benchmark reads from whole genomes (including plasmids) that feed the runtime benchmark)

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i reads.fastq -a mem -o kaiju.out
```

## Evaluation signals

- Kaiju peak memory falls below 6 GB on the benchmark database, while Kraken reaches ~72 GB and Clark 65-78 GB
- Kaiju MEM throughput on short single-end reads is roughly 173k r.p.s., exceeding Kraken (~165k) and Clark (~93k)
- Paired-end 250 nt runs are slowest for all tools: Kaiju MEM ~97k, Kraken ~24k, Clark ~19k r.p.s.
- Kaiju throughput is high enough to classify millions of reads per minute on a standard PC with 25 threads
- Each tool/mode produces a complete classification output file with one line per read (or read pair), containing classified/unclassified status and assigned NCBI taxon identifiers

## Limitations

- Allowing substitutions in the backwards search causes an exponential runtime increase depending on the number of allowed mismatched positions, so Greedy-5 is slower than MEM
- Absolute r.p.s. and memory figures depend on the reference database size and hardware; Kaiju's <6 GB footprint applies to the paper's benchmark database
- Timing results vary with read length and the number of allowed mismatches, so figures are not transferable across read types without re-benchmarking

## Evidence

- [discussion] Kaiju is fast enough for classifying up to millions of reads per minute, depending on the read length and the number of allowed mismatches, and is typically faster than the two k-mer-based methods: "Kaiju is fast enough for classifying up to millions of reads per minute, depending on the read length and the number of allowed mismatches"
- [discussion] Kaiju memory footprint statement: "Kaiju's memory footprint small enough (below 6 GB on our benchmark database) for running it on a standard PC"
- [discussion] Comparator versions and k-mer setting: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [readme] Compile command from README: "For compiling Kaiju and its associated programs, type:
cd kaiju/src
make"
- [readme] Thread option from README: "Kaiju can use multiple parallel threads, which can be specified with the `-z` option, e.g. for using 25 parallel threads"
