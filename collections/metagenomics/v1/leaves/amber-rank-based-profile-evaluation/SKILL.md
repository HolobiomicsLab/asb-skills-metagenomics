---
name: amber-rank-based-profile-evaluation
description: Use when per-sample taxonomic profiling results (e.g. ganon2, kraken2+bracken,
  kmcp outputs on simulated CAMI/RefSeq samples) must be scored against ground-truth
  taxon assignments to compare F1-score by rank and abundance accuracy.
when_to_use_negative:
- The task is taxonomic binning evaluation rather than profiling — AMBER also reports
  binning metrics (purity, completeness, UniFrac), but this skill targets rank-based
  profiling F1 and L1-norm error
- Input reads are real (non-simulated) samples with no ground-truth taxon assignments
  to compare against
- Raw read FASTQ files that have not yet been classified/profiled by a tool such as
  ganon2 or kraken2+bracken
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: AMBER
  role: Computes rank-based and cumulative evaluation metrics (sens, prec, f1s) of
    predicted vs ground-truth profiles, pinned at version 2.0.21-beta
  repo: https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz
- name: Snakemake
  role: Orchestrates the build and classify pipeline whose classify step invokes AMBER
    evaluation and aggregates results into profiling_benchmark_results.tsv
  repo: github.com/pirovc/ganon_benchmark
- name: ganon2
  role: Profiler under evaluation; its per-sample profiles are scored with AMBER (median
    species F1 ~0.67 on full RefSeq)
  repo: https://github.com/pirovc/ganon
- name: kraken2+bracken
  role: Comparator profiler scored with the same AMBER evaluation (median species
    F1 ~0.31 on full RefSeq)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nargab/lqaf094
    title: 'ganon2: up-to-date and scalable metagenomics analysis'
  source_content_hash: sha256:9b031e7773a8d675e659c150977f4b943fa81dc13ba3519ce90e55ed33ef1bbd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AMBER
  - Snakemake
  - conda
  - ganon2
  - kmcp
  - kraken2+bracken
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# amber-rank-based-profile-evaluation

## Summary

Evaluates taxonomic profiling outputs against ground truth using AMBER 2.0.21-beta, extracting per-rank (tp, tp_direct, tp_indirect, fp_lower, fp_higher, sens, prec, f1s) and cumulative metrics, plus L1-norm abundance error and runtime, to compare profilers such as ganon2 vs kraken2+bracken on simulated samples.

## When to use

Use when per-sample taxonomic profiling results (e.g. ganon2, kraken2+bracken, kmcp outputs on simulated CAMI/RefSeq samples) must be scored against ground-truth taxon assignments to compare F1-score by rank and abundance accuracy.

## When NOT to use

- The task is taxonomic binning evaluation rather than profiling — AMBER also reports binning metrics (purity, completeness, UniFrac), but this skill targets rank-based profiling F1 and L1-norm error
- Input reads are real (non-simulated) samples with no ground-truth taxon assignments to compare against
- Raw read FASTQ files that have not yet been classified/profiled by a tool such as ganon2 or kraken2+bracken

## Inputs

- Per-sample taxonomic profiling outputs (bioboxes-format profiles from ganon2, kraken2+bracken, kmcp)
- Ground-truth readid_assembly_taxid files (readid_assembly_taxid.txt.gz) for the 16 simulated CAMI/RefSeq samples
- Per-sample .stats files with runtimes and peak memory
- NCBI taxdump (nodes.dmp, names.dmp, merged.dmp) used for taxonomy-consistent evaluation

## Outputs

- Per-sample .rank.tsv files with rank-based metrics (tp, tp_direct, tp_indirect, fp_lower, fp_higher, sens, prec, f1s) and cumulative results
- Aggregated profiling_benchmark_results.tsv with per-sample species-level F1-scores, L1-norm abundance errors, runtimes and peak memory

## How to apply

After profiling each of the 16 simulated samples in default profiling mode with the minimum-abundance threshold applied, evaluate every predicted profile against the ground-truth (readid_assembly_taxid-derived) profile with AMBER 2.0.21-beta, pinning the exact commit archive (658a268) for reproducibility. Collect rank-based per-sample results from the .rank.tsv files (columns: tp, tp_direct, tp_indirect, fp_lower, fp_higher, sens, prec, f1s) and the cumulative results, focusing on the species rank. Pull runtimes and peak memory from the .stats files and the L1-norm abundance error per sample. Aggregate species-level F1-scores and L1-norm errors across all samples into profiling_benchmark_results.tsv, then validate the aggregated medians against reported targets (ganon2 median species F1 ~0.67 vs ~0.31 for kraken2+bracken on full RefSeq; ganon2 L1-norm error not exceeding ~100 in all scenarios).

## Related tools

- **AMBER** (Computes rank-based and cumulative evaluation metrics (sens, prec, f1s) of predicted vs ground-truth profiles, pinned at version 2.0.21-beta) — https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz
- **Snakemake** (Orchestrates the build and classify pipeline whose classify step invokes AMBER evaluation and aggregates results into profiling_benchmark_results.tsv) — github.com/pirovc/ganon_benchmark
- **ganon2** (Profiler under evaluation; its per-sample profiles are scored with AMBER (median species F1 ~0.67 on full RefSeq)) — https://github.com/pirovc/ganon
- **kraken2+bracken** (Comparator profiler scored with the same AMBER evaluation (median species F1 ~0.31 on full RefSeq))

## Evaluation signals

- ganon2 median species-level profiling F1 ~0.67 vs ~0.31 for kraken2+bracken on full RefSeq across the 16 samples
- ganon2 L1-norm abundance error does not exceed ~100 in any evaluated scenario
- Each .rank.tsv contains the expected rank-based columns: tp, tp_direct, tp_indirect, fp_lower, fp_higher, sens, prec, f1s, plus cumulative results
- RefSeq RG full-benchmark runtimes fall near ~43 min (ganon2), ~59 min (kraken2+bracken) and ~4 h 47 min (kmcp) for all 16 samples
- Aggregated profiling_benchmark_results.tsv covers exactly the 16 simulated samples with per-sample rows

## Limitations

- Results depend on the 16 simulated samples (including CAMI 1+2 challenge data) and the RefSeq snapshot used; absolute scores may shift with different read sets or database versions
- AMBER is pinned to version 2.0.21-beta at a specific commit; other versions may compute metrics differently
- The minimum-abundance threshold applied in default profiling mode directly affects F1-score and L1-norm error and must be kept consistent across compared tools

## Evidence

- [methods] AMBER pinned version used for evaluation: "AMBER==2.0.21-beta (https://github.com/CAMI-challenge/AMBER/archive/658a268bc86622e6f41304d38c1856b727c995ed.tar.gz)"
- [readme] AMBER purpose for comparative assessment: [evidence span withheld — anchor status: external_doc]
- [readme] Workflow step for unpacking ground-truth files: "# unpack gt files
	gzip -d files/reads/cami/*/*readid_assembly_taxid.txt.gz"
- [abstract] Profiling F1 improvement finding: "In profiling, improvements in the F1-score median are up to 0.35, keeping a balanced L1-norm error in the abundance estimation"
- [readme] Pipeline divided into build and classify steps: "The pipeline is divided into build and classify steps. The folder config/ contains examples of configuration files for building and classifying"
