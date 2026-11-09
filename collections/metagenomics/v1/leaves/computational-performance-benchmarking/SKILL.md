---
name: computational-performance-benchmarking
description: Use when you need to verify claims that a sequence-analysis tool runs
  efficiently on commodity hardware, i.e. quantify wall-clock seconds and peak RAM
  per input record.
when_to_use_negative:
- Input is nucleotide reads or assembled contigs rather than protein FASTA — DiSCo
  takes user-supplied protein sequences only
- The goal is functional benchmarking of classification accuracy (0.98 complete genomes
  / 0.93 metagenomic records) rather than computational resource measurement
- You intend to run hmmsearch multi-threaded, since the reported ~3.4 s/genome figure
  was obtained with the single-thread default (-n 1)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DiSCo 1.0
  role: The genome-mining tool under benchmark; run via DiSCo.pl on each genome's
    protein FASTA with default settings
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: HMMER3 search engine invoked by DiSCo.pl against the DiSCo HMM library, restricted
    to one thread (-n 1) for the benchmark
  repo: http://hmmer.org/
- name: Perl 5
  role: Language in which DiSCo.pl and its filter script are implemented and executed
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:cb03d1217b9098ed2e37e59b77e089e0aa9b896e4cdf98f38933f89f6b3ade62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - DiSCo 1.0
  - Perl 5
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# computational-performance-benchmarking

## Summary

Method for benchmarking the per-genome runtime and peak memory footprint of a genome-mining tool such as DiSCo by wrapping each invocation with process-level monitoring and averaging resource statistics across a large genome set (e.g. 5100 complete bacterial and archaeal genomes).

## When to use

Use when you need to verify claims that a sequence-analysis tool runs efficiently on commodity hardware, i.e. quantify wall-clock seconds and peak RAM per input record.

## When NOT to use

- Input is nucleotide reads or assembled contigs rather than protein FASTA — DiSCo takes user-supplied protein sequences only
- The goal is functional benchmarking of classification accuracy (0.98 complete genomes / 0.93 metagenomic records) rather than computational resource measurement
- You intend to run hmmsearch multi-threaded, since the reported ~3.4 s/genome figure was obtained with the single-thread default (-n 1)

## Inputs

- 5100 complete genome protein sequence files in FASTA format (4825 bacteria, 253 archaea) downloaded from NCBI
- DiSCo 1.0 Perl scripts and DiSCo HMM library from github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- HMMER3 hmmsearch (v. 3.1b2) available in $PATH

## Outputs

- Per-genome wall-clock runtime measurements
- Per-genome peak resident memory (MB) measurements
- Mean runtime (~3.4 s/genome) and mean peak memory (~27 MB/genome) across all 5100 genomes
- Filtered DiSCo hitlist (tab-separated by default) with sequence identifier, HMM name, predicted enzyme type, score, bias, E-value, accuracy, domain coordinates and DsrC-type notes

## How to apply

Download the complete genomes dataset from NCBI (4825 bacteria and 253 archaea) as protein FASTA files, then run DiSCo.pl on each genome's protein FASTA with default settings, keeping hmmsearch restricted to a single thread (-n 1, the default per the README). Wrap each DiSCo invocation with standard process-level monitoring such as /usr/bin/time or psutil to record wall-clock runtime and peak resident memory per genome. Compute the mean runtime and mean peak memory across all 5100 genomes. Collect the filtered DiSCo hitlist produced with the default tab-separated separator (-s 0) for downstream inspection. Validate by confirming the observed averages reproduce the reported performance (about 27 MB RAM and about 3.4 s per genome), consistent with the paper's claim that DiSCo can analyse thousands of genomes in a matter of hours on a personal computer with no need for high-performance servers.

## Related tools

- **DiSCo 1.0** (The genome-mining tool under benchmark; run via DiSCo.pl on each genome's protein FASTA with default settings) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (HMMER3 search engine invoked by DiSCo.pl against the DiSCo HMM library, restricted to one thread (-n 1) for the benchmark) — http://hmmer.org/
- **Perl 5** (Language in which DiSCo.pl and its filter script are implemented and executed) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo

## Examples

```
/usr/bin/time -v perl DiSCo.pl -i genome.faa -n 1 && perl DiSCo.pl -i example.faa -p example -s 0
```

## Evaluation signals

- Mean peak resident memory across all 5100 genomes is approximately 27 MB per genome
- Mean wall-clock runtime is approximately 3.4 seconds per genome in single-thread mode
- hmmsearch thread count is confirmed as 1 (the -n default) in every benchmarked invocation
- The filtered hitlist is emitted in the default tab-separated format (-s 0) with all 20 documented output columns populated
- Aggregate analysis time of thousands of genomes falls within a few hours on a standard laptop, reproducing the paper's performance claim

## Limitations

- Benchmark reflects single-thread hmmsearch mode (-n 1, the default); multi-threaded performance is not characterized here
- Resource figures were measured on a standard laptop, so absolute numbers may differ on other hardware
- Runtime scales with genome size and protein content, so per-genome averages mask variation across large versus small genomes
- The published version states no explicit software version number for DiSCo in some sections, so version pinning must come from the repository (DiSCo 1.0)

## Evidence

- [readme] Single-thread runtime/memory averages: "-n <number> 			#number of threads for hmmsearch [optional, default = 1]"
- [abstract] Personal-computer performance claim: "DiSCo can analyse thousands of genomes in a matter of hours on a personal computer with no need for high-performance servers."
- [readme] Default tab-separated output: "0 = TAB=\t		[default: tab separated]"
- [readme] Implementation and HMMER dependency: "All programs are written in perl (version 5) and dependent on HMMER3. The script requires hmmsearch to be in your $PATH."
