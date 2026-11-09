---
name: cpu-parallelization-runtime-benchmarking
description: Use when you need to quantify parallel-scaling behaviour of a CPU-parallelized
  bioinformatics tool (e.g. chewBBACA AlleleCall) by wall-clock timing the same workload
  across several --cpu values and identifying an I/O bottleneck.
when_to_use_negative:
- The tool you are benchmarking uses a single CPU core (e.g. Genome Profiler), which
  'making it unsuitable for large-scale analyses' — a CPU-scaling benchmark would
  be meaningless.
- You only need one allelic profile matrix and already know the machine's optimal
  core count — simply run AlleleCall with that --cpu value instead of a full sweep.
- Input is raw sequencing reads rather than draft/complete genome FASTA assemblies
  — AlleleCall takes FASTA assemblies, not FASTQ.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: chewBBACA
  role: Provides the AlleleCall module being benchmarked; performs BLAST Score Ratio
    (BSR, default threshold 0.6) based allele calling using multiprocessing over the
    --cpu parameter
  repo: https://github.com/B-UMMI/chewBBACA
- name: BLAST
  role: Underlying alignment engine used by AlleleCall for allele assignment; its
    subprocess cost contributes to the measured scaling curve
- name: Python 3.4
  role: Runtime environment required by chewBBACA; used on both laptops and HPC clusters
    for the benchmark runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1099/mgen.0.000166
    title: 'chewBBACA: A complete suite for gene-by-gene schema creation and strain
      identification'
  source_content_hash: sha256:4d5f18b275ec3a17f03e25e2d687bcec573c98575c4487222b97e4fe49b09e74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/B-UMMI/chewBBACA
  tools:
  - BLAST
  - Python 3.4
  - chewBBACA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# cpu-parallelization-runtime-benchmarking

## Summary

Benchmarks how the runtime of chewBBACA's BLAST-based AlleleCall module scales with CPU count (multiprocessing) by timing repeated allele-calling runs on 100 S. agalactiae draft assemblies, revealing near-linear speedup up to 32 CPUs followed by a disk/I/O-bound plateau.

## When to use

Use when you need to quantify parallel-scaling behaviour of a CPU-parallelized bioinformatics tool (e.g. chewBBACA AlleleCall) by wall-clock timing the same workload across several --cpu values and identifying an I/O bottleneck.

## When NOT to use

- The tool you are benchmarking uses a single CPU core (e.g. Genome Profiler), which 'making it unsuitable for large-scale analyses' — a CPU-scaling benchmark would be meaningless.
- You only need one allelic profile matrix and already know the machine's optimal core count — simply run AlleleCall with that --cpu value instead of a full sweep.
- Input is raw sequencing reads rather than draft/complete genome FASTA assemblies — AlleleCall takes FASTA assemblies, not FASTQ.

## Inputs

- 100 draft genome FASTA files of Streptococcus agalactiae assemblies (FigShare zip: https://figshare.com/s/9cbe1d422805db54cd52)
- cgMLST schema directory (schema_seed with reference loci FASTA files) produced by a prior CreateSchema step
- List of CPU counts to benchmark (e.g. 1, 2, 4, 8, 16, 32, 64) passed as the --cpu parameter of chewBBACA.py AlleleCall

## Outputs

- allelecall_runtime_scaling.tsv — runtime-versus-CPU-count table with per-sample wall-clock times
- Allelic profile matrix (results_alleles.tsv) from each AlleleCall run
- Wall-clock timing log per CPU configuration

## How to apply

Download the zip file of S. agalactiae draft genome FASTA assemblies deposited on FigShare (https://figshare.com/s/9cbe1d422805db54cd52) and extract 100 draft genomes. Load a pre-built cgMLST schema directory (schema_seed loci) as the reference loci, then invoke chewBBACA's AlleleCall module once per tested CPU count, passing the count via the --cpu parameter and recording wall-clock time for each run. Parse AlleleCall outputs (allelic profile matrix) to compute per-sample runtime for each configuration, and tabulate runtime versus CPU count. Judge success by whether runtimes show near-linear per-sample decrease up to 32 CPUs, an I/O-bound plateau beyond that, and per-strain times in the seconds-to-minutes range — the article reports peak performance of ~2.5 s per sample with 32 CPUs on an HPC cluster. Note the rationale: chewBBACA is 'easily scalable for the analysis of large datasets' via multiprocessing, but disk I/O ultimately limits throughput, so the --cpu value should be matched to the storage subsystem.

## Related tools

- **chewBBACA** (Provides the AlleleCall module being benchmarked; performs BLAST Score Ratio (BSR, default threshold 0.6) based allele calling using multiprocessing over the --cpu parameter) — https://github.com/B-UMMI/chewBBACA
- **BLAST** (Underlying alignment engine used by AlleleCall for allele assignment; its subprocess cost contributes to the measured scaling curve)
- **Python 3.4** (Runtime environment required by chewBBACA; used on both laptops and HPC clusters for the benchmark runs)

## Examples

```
chewBBACA.py AlleleCall -i complete_genomes/ -g tutorial_schema/schema_seed -o results32_wgMLST --cpu 6
```

## Evaluation signals

- Per-sample runtime decreases near-linearly with CPU count up to 32 CPUs, then plateaus (disk/I/O-bound) on the HPC cluster.
- Every per-strain runtime falls in the seconds-to-minutes range (article: 'a matter of seconds to minutes per strain on a laptop').
- Peak measured performance is approximately 2.5 s per sample using 32 CPUs on the HPC cluster.
- Each AlleleCall run yields a valid allelic profile matrix (results_alleles.tsv) covering all input genomes and reference loci.
- The final TSV contains one row per CPU configuration with matching wall-clock and derived per-sample runtimes.

## Limitations

- Beyond 32 CPUs, disk/I/O becomes the bottleneck, so adding more cores yields no further per-sample speedup.
- Reported execution times are hardware-dependent: the tutorial notes 'Using a computer with less powerful specifications can greatly increase the duration of the analyses.'
- Novel alleles identified during AlleleCall are added to the schema (tutorial: 14,720 novel alleles added), so repeated benchmark runs against a mutating schema require a fresh schema copy to keep workloads comparable.

## Evidence

- [abstract] The software performs allele calling in a matter of seconds to minutes per strain on a laptop but is easily scalable for the analysis of large datasets: "The software performs allele calling in a matter of seconds to minutes per strain on a laptop but is easily scalable for the analysis of large datasets"
- [discussion] The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input: "The allele-calling engine of chewBBACA uses FASTA files with draft assemblies or complete genomes as input"
- [readme] took approximately 17 minutes to complete (an average of 32 seconds per genome): [evidence span withheld — anchor status: external_doc]
- [readme] Using a computer with less powerful specifications can greatly increase the duration of the analyses.: [evidence span withheld — anchor status: external_doc]
