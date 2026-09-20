---
name: high-throughput-genome-scalability-testing
description: Use when you need to verify that a core-genome estimation pipeline (mOTUpan/mOTUlizer
  with mmseqs2 gene clustering) scales to thousands of MAGs/genomes before committing
  to a full production run, or to reproduce the published resource benchmarks.
when_to_use_negative:
- The genome set is small (tens of genomes) — scalability benchmarking adds no information
  over simply running the pipeline
- You lack the *.faa inputs and your own gene clusters are not in the --cog_file JSON-compatible
  format expected by mOTUpan.py
- The goal is read-based OTU table generation from the unrelated mOTUs tool rather
  than core-genome partitioning
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan
  role: Iterative Bayesian core-genome partitioning method whose scalability is being
    benchmarked
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUpan.py
  role: Command-line entry point; --faas triggers mmseqs2 easy_cluster gene clustering
    and -o writes the TSV output
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUlizer
  role: Software package implementing mOTUpan and its companion scripts (mOTUlize.py,
    mOTUconvert.py)
  repo: https://github.com/moritzbuck/mOTUlizer
- name: MMseqs2
  role: Performs the parallelizable easy_cluster gene-clustering stage (~4 min on
    24 threads for 9443 genomes)
  repo: https://github.com/soedinglab/MMseqs2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:d119db1d19772389d4e695c3d4a44abf8ae80cec3d0d23e3a9a4afc360dce45c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/moritzbuck/mOTUlizer
  tools:
  - MMseqs2
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# high-throughput-genome-scalability-testing

## Summary

Benchmark the runtime and memory footprint of mOTUpan-based core-genome partitioning at the scale of thousands of genomes, using the reported 9443 Staphylococcus aureus GTDB run (mmseqs2 clustering in ~4 min on 24 threads; mOTUpan in ~2 h 15 min single-threaded at ~3 GB RAM on a Ryzen 9 3900X) as the reference envelope.

## When to use

Use when you need to verify that a core-genome estimation pipeline (mOTUpan/mOTUlizer with mmseqs2 gene clustering) scales to thousands of MAGs/genomes before committing to a full production run, or to reproduce the published resource benchmarks.

## When NOT to use

- The genome set is small (tens of genomes) — scalability benchmarking adds no information over simply running the pipeline
- You lack the *.faa inputs and your own gene clusters are not in the --cog_file JSON-compatible format expected by mOTUpan.py
- The goal is read-based OTU table generation from the unrelated mOTUs tool rather than core-genome partitioning

## Inputs

- Protein FASTA files (*.faa) for the genome set (e.g. 9443 S. aureus GTDB genomes)
- Optional: precomputed gene clusters passed via --cog_file instead of --faas

## Outputs

- TAB-separated per-gene-cluster core/accessory partitioning (output.tsv via -o)
- Wall-clock runtime and peak-memory measurements for the mmseqs2 clustering stage and the single-threaded mOTUpan stage

## How to apply

Collect the protein FASTA files (*.faa) for the genome set (e.g. the 9443 S. aureus GTDB genomes) and run mOTUpan.py with --faas so that gene clustering is computed with mmseqs2 easy_cluster parallelized across threads (24 threads in the benchmark, ~4 min). The mOTUpan stage then runs iteratively on a single thread, recomputing the likelihood of core vs accessory assignment for each gene cluster and updating the completeness prior until convergence, using only ~3 GB RAM (~2 h 15 min total on a Ryzen 9 3900X). Write results with -o output.tsv as a TAB-separated core/accessory partitioning per gene cluster. Instrument both stages with wall-clock timing and peak-memory measurement (e.g. /usr/bin/time -v) and compare against the reported envelope to judge scalability.

## Related tools

- **mOTUpan** (Iterative Bayesian core-genome partitioning method whose scalability is being benchmarked) — https://github.com/moritzbuck/mOTUlizer
- **mOTUpan.py** (Command-line entry point; --faas triggers mmseqs2 easy_cluster gene clustering and -o writes the TSV output) — https://github.com/moritzbuck/mOTUlizer
- **mOTUlizer** (Software package implementing mOTUpan and its companion scripts (mOTUlize.py, mOTUconvert.py)) — https://github.com/moritzbuck/mOTUlizer
- **MMseqs2** (Performs the parallelizable easy_cluster gene-clustering stage (~4 min on 24 threads for 9443 genomes)) — https://github.com/soedinglab/MMseqs2

## Examples

```
/usr/bin/time -v mOTUpan.py --faas *.faa -o output.tsv
```

## Evaluation signals

- Gene-clustering stage completes in ~4 min on 24 threads for 9443 S. aureus GTDB genomes
- Full mOTUpan run finishes in ~2 h 15 min single-threaded on a Ryzen 9 3900X with peak RAM around 3 GB
- output.tsv contains a core/accessory assignment for every gene cluster in the input set
- Iterative recomputation converged (partitioning stable) before the run terminated
- Measured runtime/memory stay within the reported envelope when the benchmark is reproduced with equivalent thread counts

## Limitations

- Reference figures are hardware-dependent (Ryzen 9 3900X, 24 threads); other CPUs will shift the envelope
- mOTUpan's core partitioning runs single-threaded, so parallelism only accelerates the mmseqs2 clustering stage
- Benchmarked with a single species (S. aureus); much more pangenome-diverse taxa may exceed ~3 GB RAM

## Evidence

- [results] Runtime/memory reference for mOTUpan stage: [evidence span withheld — anchor status: not_found]
- [methods] Command form for mOTUpan.py: "Simplest command to run (needs mmseqs2 installed), but many options:

``nmOTUpan.py --faas *.faa -o output.tsv"
- [methods] Gene clustering via mmseqs2: "it computes gene clusters using [`mmseqs2`](https://github.com/soedinglab/MMseqs2)"
- [methods] Iterative convergence of the core partitioning: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [abstract] Scalability claim from abstract: "The core-genome prediction is computationally efficient and can be scaled up to thousands of genomes."
- [methods] Alternative to --faas input: "`--cog_file` : can be used as an alternative to `--faas`, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
