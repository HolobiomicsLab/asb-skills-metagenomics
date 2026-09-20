---
name: nanopore-read-simulation
description: Use when you need ground-truth nanopore read sets of known origin and
  adjustable accuracy to benchmark read classification or alignment accuracy (F1,
  precision, recall) and throughput.
when_to_use_negative:
- You already have real sequencer reads with verified labels (e.g. a SRA set like
  SRX6602475 spiked with simulated host reads); simulation is only needed for the
  ground-truth component
- You need to model raw nanopore electrical signal or Read Until API signal batches
  — PBSIM2 operates at the base-sequence level and SPUMONI-class workflows require
  a separate upstream base caller
- You need exact reproduction of a specific real sequencing run's error profile rather
  than controlled accuracy sweeps
tools:
- name: PBSIM2
  role: Model-based simulator of nanopore long reads at specified mean accuracies,
    providing ground-truth read sets for the benchmark
  repo: https://github.com/yukiteruono/pbsim2
- name: SPUMONI
  role: Read classifier under evaluation, run in PML mode (spumoni run -r <index>
    -p <reads.fa> -P -c) on the simulated read sets
  repo: https://github.com/oma219/spumoni
- name: minimap2
  role: Alignment baseline run on the same simulated read sets to compute comparative
    accuracy and throughput
- name: malloc_count
  role: Runtime library used to measure peak heap memory (RSS) of classifier/aligner
    runs on simulated data
  repo: https://github.com/bingmann/malloc_count
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:0325141b4d4d0d47f76d55742827059a6c886f61dc4f9db4c40e46f5a907ce10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yukiteruono/pbsim2
  tools:
  - Docker
  - PBSIM2
  - SPUMONI
  - malloc_count
  - minimap2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nanopore-read-simulation

## Summary

Simulate Oxford-Nanopore-like long reads with PBSIM2 at controlled mean base-call accuracies (e.g. 85%, 90%, 95%, 98%) from reference FASTAs, to generate benchmark read sets for evaluating read classifiers such as SPUMONI against alignment baselines like minimap2.

## When to use

Use when you need ground-truth nanopore read sets of known origin and adjustable accuracy to benchmark read classification or alignment accuracy (F1, precision, recall) and throughput.

## When NOT to use

- You already have real sequencer reads with verified labels (e.g. a SRA set like SRX6602475 spiked with simulated host reads); simulation is only needed for the ground-truth component
- You need to model raw nanopore electrical signal or Read Until API signal batches — PBSIM2 operates at the base-sequence level and SPUMONI-class workflows require a separate upstream base caller
- You need exact reproduction of a specific real sequencing run's error profile rather than controlled accuracy sweeps

## Inputs

- Reference genome FASTA(s) to sample reads from (e.g. ZymoMC microbial genomes, CHM13 human assembly, multi-FASTA pan-genomes)
- PBSIM2 HMM model file for Nanopore chemistry (e.g. data/P6C4.model, R9.4)
- Target mean read accuracy levels (85%, 90%, 95%, 98%) and desired sequencing depth (e.g. --depth 20)

## Outputs

- Simulated nanopore FASTA/FASTQ read files with known true origin per read, one set per accuracy level and reference
- Ground-truth labels enabling per-read classification accuracy, precision, recall, and F1 computation
- Benchmark comparison table (e.g. table2_benchmark_results.csv, table4_spumoni_vs_minimap2.csv) with F1, throughput (kbp/s), peak RSS, and index size

## How to apply

Run PBSIM2 in model-based mode against a chosen reference (e.g. ZymoMC genomes with R9.4 chemistry, or CHM13 human at 90% base-call accuracy), tuning the simulator's accuracy parameter to produce read sets at each target mean accuracy (85%, 90%, 95%, 98%). Because the simulation samples reads from known reference loci, every read's true origin is known, enabling exact computation of classification accuracy, precision, recall, and F1 against classifier outputs (SPUMONI PML/matching-statistics reports) or alignment outputs (minimap2). Generate one read set per accuracy level and per reference scenario (one-genome vs pan-genome/multi-strain) so throughput (kbp/s) and memory (peak RSS, e.g. via malloc_count) can be compared under identical inputs. Note that higher simulated base-call accuracy generally favors SPUMONI, which is most advantageous at higher accuracy, so sweeping accuracy is essential rather than testing a single level.

## Related tools

- **PBSIM2** (Model-based simulator of nanopore long reads at specified mean accuracies, providing ground-truth read sets for the benchmark) — https://github.com/yukiteruono/pbsim2
- **SPUMONI** (Read classifier under evaluation, run in PML mode (spumoni run -r <index> -p <reads.fa> -P -c) on the simulated read sets) — https://github.com/oma219/spumoni
- **minimap2** (Alignment baseline run on the same simulated read sets to compute comparative accuracy and throughput)
- **malloc_count** (Runtime library used to measure peak heap memory (RSS) of classifier/aligner runs on simulated data) — https://github.com/bingmann/malloc_count

## Examples

```
pbsim --depth 20 --hmm_model data/P6C4.model sample/sample.fasta
```

## Evaluation signals

- Each simulated read maps back to exactly one known reference locus, so confusion-matrix counts (TP/FP/FN) sum to the total number of simulated reads per condition
- The empirical mean base-call accuracy of each simulated set matches the requested level (85%, 90%, 95%, 98%) within simulator tolerance
- Reproduced metrics fall near reported values: e.g. three-genome human host-depletion F1 of 97.08% (SPUMONI) vs 99.17% (minimap2), throughput 27.0 vs 13.5 kbp/s
- SPUMONI's simulated-ZymoMC speedup over minimap2 should land near ~1.8x (one-genome index) and ~19.3x (pan-genome index) across the accuracy sweep
- All 16 conditions (2 classifiers x 4 accuracies x 2 reference types) produce complete rows in the output CSV before aggregation

## Limitations

- Simulated reads only approximate real nanopore error profiles; a single KS-stat threshold tuned on real nanopore data was used across all experiments, and the optimal threshold is expected to depend on read accuracy and reference
- Simulations do not capture Read Until signal-level behavior: the classifier operates on base-called sequences, not the raw signal batches delivered by the Read Until API
- Throughput conclusions may not transfer when reference similarity differs — e.g. SPUMONI's advantage over minimap2 is limited for human reads and expected to improve only as more human genomes are added to the reference

## Evidence

- [methods] Simulate ZymoMC R9.4 nanopore R9.4 reads with PBSIM2 at four mean accuracy levels (85%, 90%, 95%, 98%).: [evidence span withheld — anchor status: not_found]
- [readme] We have developed a long reads simulater (called PBSIM) in which sampling-based and model-based simulations are implemented.: [evidence span withheld — anchor status: external_doc]
- [discussion] SPUMONI is well positioned for these trends because it delivers its most advantageous combinations of speed and F1 score at higher base-calling accuracy.: "SPUMONI is well positioned for these trends because it delivers its most advantageous combinations of speed and F1 score at higher base-calling accuracy."
- [methods] Generate simulated human nanopore reads from CHM13 at 90% base-call accuracy using PBSIM2.: [evidence span withheld — anchor status: not_found]
