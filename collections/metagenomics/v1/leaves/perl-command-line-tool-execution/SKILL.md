---
name: perl-command-line-tool-execution
description: Use when you need to reproduce or verify the resource requirements (seconds
  and MB RAM per genome) of a Perl-based CLI genome-mining tool like DiSCo on FASTA
  protein inputs, or to demonstrate a tool runs on a personal computer without HPC.
when_to_use_negative:
- Input is raw nucleotide contigs or reads rather than FASTA-formatted protein sequences
  — DiSCo.pl takes user-supplied protein sequences only
- The goal is classification accuracy evaluation against curated labels rather than
  measuring runtime/memory cost
- Only a handful of genomes are at stake and no performance claim needs verification
  — a single ad-hoc run suffices without benchmarking overhead
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DiSCo 1.0
  role: Perl 5 standalone genome-mining tool executed per genome FASTA to detect Dsr-dependent
    dissimilatory sulphur proteins and predict enzyme type
  repo: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- name: hmmsearch 3.1b2
  role: HMMER3 search engine invoked by DiSCo.pl (single thread, -n 1 default) against
    the DiSCo HMM library; the main compute cost being benchmarked
- name: Perl 5
  role: Runtime environment in which DiSCo.pl and its filter script are executed from
    the command line
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1099/mgen.0.000603
    title: 'DiSCo: a sequence-based type-specific predictor of Dsr-dependent dissimilatory
      sulphur metabolism in microbial data'
  source_content_hash: sha256:46c6ef7dc723e64ab536d921fbe2aa7cb615829e1bff9fde69c9884ccd6a6de6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
  tools:
  - DiSCo
  - DiSCo 1.0
  - Perl 5
  - hmmsearch 3.1b2
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# perl-command-line-tool-execution

## Summary

Benchmark the computational footprint of DiSCo.pl, a Perl 5 command-line tool, by running it single-threaded against 5100 complete genome protein FASTA files and recording wall-clock runtime and peak resident memory per genome with process-level monitors such as /usr/bin/time or psutil.

## When to use

Use when you need to reproduce or verify the resource requirements (seconds and MB RAM per genome) of a Perl-based CLI genome-mining tool like DiSCo on FASTA protein inputs, or to demonstrate a tool runs on a personal computer without HPC.

## When NOT to use

- Input is raw nucleotide contigs or reads rather than FASTA-formatted protein sequences — DiSCo.pl takes user-supplied protein sequences only
- The goal is classification accuracy evaluation against curated labels rather than measuring runtime/memory cost
- Only a handful of genomes are at stake and no performance claim needs verification — a single ad-hoc run suffices without benchmarking overhead

## Inputs

- 5100 complete genome protein sequence files in FASTA format (4825 bacteria, 253 archaea) downloaded from NCBI
- DiSCo.pl script and DiSCo HMM library from the GitHub repository
- HMMER3 (hmmsearch) installed and on $PATH

## Outputs

- Per-genome wall-clock runtime and peak resident memory measurements
- Mean runtime (~3.4 s) and mean peak memory (~27 MB) across the 5100-genome dataset
- Default tab-separated filtered DiSCo hitlist per genome (sequence ID, HMM name, predicted enzyme type, score, bias, E-value, accuracy, domain coordinates, DsrC cysteine note)

## How to apply

Download the 5100 complete genome protein sets (4825 bacteria, 253 archaea) from NCBI in FASTA format and ensure hmmsearch (HMMER3, v3.1b2) is in $PATH. Run 'perl DiSCo.pl -i <genome>.faa' with default settings so hmmsearch uses a single thread (-n 1 is the default), wrapping each invocation with /usr/bin/time or psutil to capture wall-clock time and peak resident set size. Collect the default tab-separated hitlist (-s 0) produced for each genome for downstream inspection. Average runtime and peak memory across all 5100 genomes and compare against the reported ~3.4 s and ~27 MB per genome, which substantiates the claim that thousands of genomes can be analysed in hours on a personal computer.

## Related tools

- **DiSCo 1.0** (Perl 5 standalone genome-mining tool executed per genome FASTA to detect Dsr-dependent dissimilatory sulphur proteins and predict enzyme type) — https://github.com/Genome-Evolution-and-Ecology-Group-GEEG/DiSCo
- **hmmsearch 3.1b2** (HMMER3 search engine invoked by DiSCo.pl (single thread, -n 1 default) against the DiSCo HMM library; the main compute cost being benchmarked)
- **Perl 5** (Runtime environment in which DiSCo.pl and its filter script are executed from the command line)

## Examples

```
/usr/bin/time -v perl DiSCo.pl -i genome.faa -s 0
```

## Evaluation signals

- Mean peak resident memory across all 5100 genomes is approximately 27 MB (order of tens of MB, not hundreds)
- Mean wall-clock runtime per genome is approximately 3.4 s in single-thread mode, implying the full dataset completes in hours on a laptop
- Each run produces a tab-separated hitlist (default -s 0) whose columns match the documented schema (sequence identifier, HMM name, predicted enzyme type, score, bias, E-value, accuracy, domain fields, DsrC note)
- Total runtime scales roughly linearly with genome count, consistent with one ~3.4 s hmmsearch pass per genome
- hmmsearch thread count confirmed at 1 (-n 1) so measurements reflect the documented single-thread benchmark

## Limitations

- Benchmark applies to complete genomes of protein sequences; per-genome cost may differ for large metagenomic records
- Measurements depend on the host laptop's hardware and on HMMER3 being correctly installed and in $PATH; DiSCo requires hmmsearch available as a dependency
- The article provides no software version metadata or explicit benchmark dataset accessions in the discussion section, so exact reproducibility relies on the GitHub distribution and NCBI download snapshot

## Evidence

- [abstract] DiSCo can analyse thousands of genomes in a matter of hours on a personal computer with no need for high-performance servers.: "DiSCo can analyse thousands of genomes in a matter of hours on a personal computer with no need for high-performance servers."
- [intro] Publicly available complete genomic assemblies of 4825 bacteria and 253 archaea were downloaded from NCBI: "Publicly available complete genomic assemblies of 4825 bacteria and 253 archaea were downloaded from NCBI"
- [readme] All programs are written in perl (version 5) and dependent on HMMER3. The script requires hmmsearch to be in your $PATH.: "All programs are written in perl (version 5) and dependent on HMMER3. 
The script requires hmmsearch to be in your $PATH."
- [readme] -n <number> #number of threads for hmmsearch [optional, default = 1]: "-n <number> 			#number of threads for hmmsearch [optional, default = 1]"
