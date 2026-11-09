---
name: classification-metrics-computation
description: Reference for comparing a classifier's per-read calls (e.g., SPUMONI
  PML/MS reports) against an aligner's mappings (minimap2) using accuracy, precision,
  recall, F1, throughput, peak RSS, and index size.
when_to_use_negative:
- When the input reads are raw nanopore electrical signal rather than base-called
  sequences — SPUMONI operates on base-called sequences and requires a separate base
  caller upstream (like Readfish).
- When the goal is to obtain mapping coordinates for downstream variant calling or
  assembly — minimap2 alignment, not classification metrics, is the appropriate output.
- When the reference is a small human-genome set where SPUMONI's throughput advantage
  over minimap2 is limited; the expected improvement comes only as the number of human
  genomes in the reference increases.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: SPUMONI
  role: Produces the per-read classification reports (`spumoni run -r <index> -p <reads.fa>
    -P -c`) using matching-statistics and PML indexes whose accuracy, precision, recall,
    F1, RSS, and throughput are measured
  repo: https://github.com/oma219/spumoni
- name: minimap2
  role: Aligner baseline run against the same one-genome and pan-genome references;
    its mappings provide the comparison accuracy and the throughput/peak-RSS contrast
    for the metrics table
- name: PBSIM2
  role: Generates the simulated ZymoMC R9.4 nanopore read sets at the four mean accuracy
    levels (85%, 90%, 95%, 98%) used as labeled benchmark inputs
  repo: https://github.com/yukiteruono/pbsim2
- name: malloc_count
  role: Library used to measure current and peak heap memory allocation (peak RSS)
    of SPUMONI and minimap2 during the benchmark runs
  repo: https://github.com/bingmann/malloc_count
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:77e549010421fdf7141b61a0d46d8a4fd4dd04aceb1af4e13356b84c7e1f065f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - Docker
  - PBSIM2
  - SPUMONI
  - malloc_count
  - minimap2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# classification-metrics-computation

## Summary

Benchmark read classification performance by computing accuracy, precision, recall, and F1 from SPUMONI's per-read classification reports and minimap2 alignments, alongside peak RSS (via malloc_count), index size on disk, and per-base throughput, mirroring the paper's Table 2 comparison over PBSIM2-simulated ZymoMC nanopore reads at 85–98% accuracy.

## When to use

Reference for comparing a classifier's per-read calls (e.g., SPUMONI PML/MS reports) against an aligner's mappings (minimap2) using accuracy, precision, recall, F1, throughput, peak RSS, and index size.

## When NOT to use

- When the input reads are raw nanopore electrical signal rather than base-called sequences — SPUMONI operates on base-called sequences and requires a separate base caller upstream (like Readfish).
- When the goal is to obtain mapping coordinates for downstream variant calling or assembly — minimap2 alignment, not classification metrics, is the appropriate output.
- When the reference is a small human-genome set where SPUMONI's throughput advantage over minimap2 is limited; the expected improvement comes only as the number of human genomes in the reference increases.

## Inputs

- PBSIM2-simulated ZymoMC R9.4 nanopore reads (FASTA/FASTQ) at mean accuracies 85%, 90%, 95%, 98%
- One-genome and pan-genome reference FASTA files with known strain/species labels for ground truth
- SPUMONI per-read classification report files (from `spumoni run ... -c`)
- minimap2 alignment outputs (e.g., PAF/SAM) for the same read sets
- malloc_count peak-RSS measurements, index sizes on disk, and timing/throughput logs

## Outputs

- Per-read classification labels from SPUMONI and per-read mappings from minimap2 for each condition
- A metrics table (table2_benchmark_results.csv) with accuracy, precision, recall, F1, peak RSS, index size, and throughput for 2 classifiers x 4 accuracies x 2 reference types
- Aggregate speed comparisons (e.g., ~19.3x faster with pan-genomic index, ~1.8x faster with genomic index) and F1 comparisons vs minimap2

## How to apply

Simulate or obtain base-called reads with known species-of-origin labels (here PBSIM2-simulated ZymoMC R9.4 nanopore reads at mean accuracies of 85%, 90%, 95%, and 98%), then classify each read set with both SPUMONI (`spumoni run ... -c` per-read reports) and minimap2 against one-genome and pan-genome references. For each condition (2 classifiers x 4 accuracies x 2 reference types), derive confusion-matrix counts from the classification/alignment outputs and compute accuracy, precision, recall, and F1 per read. In parallel, record peak resident set size using malloc_count, index size on disk, and per-base throughput (bases processed per unit time) for each run. Rationale: SPUMONI uses matching statistics/PML plus a null-index hypothesis test to classify, so its binary per-read calls must be judged against an aligner baseline on identical inputs; the paper reports SPUMONI has similar accuracy to minimap2 while being ~12x faster with a ~16x smaller index, so the speed/accuracy/memory trade-off is the quantity of interest. Assemble all metrics into a single table (table2_benchmark_results.csv) with columns mirroring Table 2.

## Related tools

- **SPUMONI** (Produces the per-read classification reports (`spumoni run -r <index> -p <reads.fa> -P -c`) using matching-statistics and PML indexes whose accuracy, precision, recall, F1, RSS, and throughput are measured) — https://github.com/oma219/spumoni
- **minimap2** (Aligner baseline run against the same one-genome and pan-genome references; its mappings provide the comparison accuracy and the throughput/peak-RSS contrast for the metrics table)
- **PBSIM2** (Generates the simulated ZymoMC R9.4 nanopore read sets at the four mean accuracy levels (85%, 90%, 95%, 98%) used as labeled benchmark inputs) — https://github.com/yukiteruono/pbsim2
- **malloc_count** (Library used to measure current and peak heap memory allocation (peak RSS) of SPUMONI and minimap2 during the benchmark runs) — https://github.com/bingmann/malloc_count

## Examples

```
spumoni run -r <index_prefix> -p <reads.fa> -P -c > spumoni_report.txt
```

## Evaluation signals

- The metrics table contains exactly 16 condition rows (2 classifiers x 4 mean read accuracies x 2 reference types), each with accuracy, precision, recall, F1, peak RSS, index size, and throughput populated.
- SPUMONI's F1 scores should be comparable to minimap2's when both are run against the pan-genome (many-strains-per-species) index, per the paper's central claim.
- Throughput ratios should show SPUMONI faster than minimap2 — about 19.3x on the pan-genomic index and about 1.8x on the genomic index for the simulated ZymoMC benchmark.
- SPUMONI's index size and peak RSS should be smaller than minimap2's (paper reports index ~1/16 the size and peak footprint under 1/4); flag any condition where SPUMONI's peak RSS exceeds minimap2's (only the three-human-genome case reported ~1.86x larger).
- Simulated reads have known ground-truth species labels, so precision/recall can be recomputed independently from the raw classification reports and must match the table entries.

## Limitations

- A single KS-stat threshold is used for all experiments; the optimal threshold depends on read accuracy and the reference used, and no per-experiment thresholding method is provided.
- SPUMONI operates on base-called sequences, not raw signal, and cannot yet directly interact with the Read Until API; users must run a separate base caller upstream of SPUMONI.
- The throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases.
- The null index may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads (e.g., conserved genes between species).
- SPUMONI version number is not stated in the methods (only `docker pull oma219/spumoni:latest`), complicating exact reproduction.

## Evidence

- [abstract] SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species.: "SPUMONI has similar accuracy as minimap2 when both are run against an index containing many strains per species."
- [abstract] SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively.: "SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively."
- [discussion] SPUMONI's peak RSS for three human genomes is about 1.86X larger than the minimap2's peak RSS: "SPUMONI's peak RSS for three human genomes is about 1.86X larger than the minimap2's peak RSS"
- [discussion] we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
- [discussion] SPUMONI is well positioned for these trends because it delivers its most advantageous combinations of speed and F1 score at higher base-calling accuracy.: "SPUMONI is well positioned for these trends because it delivers its most advantageous combinations of speed and F1 score at higher base-calling accuracy."
- [readme] measure the current and peak heap memory allocation, and write a memory profile for plotting: [evidence span withheld — anchor status: not_found]
