---
name: resource-usage-profiling
description: Use when you need to reproduce or verify reported computational resource
  requirements (wall-clock time, peak RAM) of running a tool like DiSCo per genome
  on a standard laptop rather than on HPC infrastructure.
when_to_use_negative:
- The goal is to assess classification accuracy (benchmarking against known annotations)
  rather than computational cost
- The workload is (meta)genomic records or non-FASTA nucleotide data rather than protein
  FASTA genomes
- The tool requires infrastructure beyond a personal computer and the claim under
  test is HPC scalability, not laptop feasibility
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DiSCo
  role: Tool under benchmark; run per genome with default settings on protein FASTA
    input
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: Perl 5
  role: Language in which DiSCo.pl is executed for each genome
- name: hmmsearch
  role: HMMER3 search engine invoked by DiSCo; restricted to a single thread (-n 1)
    during benchmarking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:5d2d7a69b84021e36dec63d34e9fe28b662e4467762b36b311c2cc7e4fc06f7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - DiSCo 1.0
  - Perl 5
  - hmmsearch
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# resource-usage-profiling

## Summary

Benchmark the per-record runtime and peak memory of a standalone bioinformatics tool on a defined dataset using process-level monitoring, to verify claims about computational feasibility (e.g. DiSCo analysing thousands of genomes on a personal computer in hours).

## When to use

Use when you need to reproduce or verify reported computational resource requirements (wall-clock time, peak RAM) of running a tool like DiSCo per genome on a standard laptop rather than on HPC infrastructure.

## When NOT to use

- The goal is to assess classification accuracy (benchmarking against known annotations) rather than computational cost
- The workload is (meta)genomic records or non-FASTA nucleotide data rather than protein FASTA genomes
- The tool requires infrastructure beyond a personal computer and the claim under test is HPC scalability, not laptop feasibility

## Inputs

- 5100 complete genome protein sequence files (4825 bacteria, 253 archaea) downloaded from NCBI in FASTA format
- DiSCo 1.0 Perl scripts and HMM library from the GitHub repository
- HMMER3 hmmsearch (v. 3.1b2) installed and available in $PATH

## Outputs

- Per-genome wall-clock runtime measurements
- Per-genome peak resident memory measurements
- Mean runtime (≈3.4 s/genome) and mean peak memory (≈27 MB/genome) across all 5100 genomes
- Filtered DiSCo hitlist in default tab-separated format for downstream inspection

## How to apply

Download the input dataset — here, 5100 publicly available complete genomic assemblies (4825 bacteria and 253 archaea) from NCBI as protein FASTA files. Run the tool under benchmark conditions, keeping its internal engine single-threaded (hmmsearch with -n 1, the DiSCo default) and all other settings default. Wrap each invocation with standard process-level monitoring such as /usr/bin/time or psutil to record wall-clock runtime and peak resident memory per genome. Aggregate results by computing the mean runtime and mean peak memory across all 5100 genomes, and compare against reported characteristics (≈27 MB RAM and ≈3.4 s per genome for DiSCo).

## Related tools

- **DiSCo** (Tool under benchmark; run per genome with default settings on protein FASTA input) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **Perl 5** (Language in which DiSCo.pl is executed for each genome)
- **hmmsearch** (HMMER3 search engine invoked by DiSCo; restricted to a single thread (-n 1) during benchmarking)

## Examples

```
/usr/bin/time -v perl DiSCo.pl -i genome.faa -n 1
```

## Evaluation signals

- Observed mean peak memory per genome is ≈27 MB
- Observed mean wall-clock runtime per genome is ≈3.4 s in single-thread mode
- Every DiSCo run used the default single hmmsearch thread (-n 1)
- Each genome yields a valid filtered DiSCo hitlist with the default tab-separated separator
- Total benchmark of 5100 genomes completes within hours on a standard laptop, consistent with the paper's claim

## Limitations

- Results are specific to single-thread mode (-n 1) and do not reflect multi-threaded performance
- Figures are averages over complete genomes dataset; metagenomic records or larger assemblies may differ
- Monitoring overhead from wrappers like /usr/bin/time may slightly inflate measured runtimes

## Evidence

- [abstract] DiSCo can analyse thousands of genomes in a matter of hours on a personal computer with no need for high-performance servers.: "DiSCo can analyse thousands of genomes in a matter of hours on a personal computer with no need for high-performance servers."
- [intro] Publicly available complete genomic assemblies of 4825 bacteria and 253 archaea were downloaded from NCBI: "Publicly available complete genomic assemblies of 4825 bacteria and 253 archaea were downloaded from NCBI"
- [readme] -n <number> #number of threads for hmmsearch [optional, default = 1]: "-n <number> 			#number of threads for hmmsearch [optional, default = 1]"
- [readme] The script requires hmmsearch to be in your $PATH.: "The script requires hmmsearch to be in your $PATH."
