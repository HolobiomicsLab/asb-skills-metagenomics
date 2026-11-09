---
name: bioinformatics-tool-benchmark-tabulation
description: Use when a paper reports per-tool benchmark numbers (bases/second throughput,
  index size, peak RSS) in tables and you need to compute and verify cross-tool performance
  ratios across experiments.
when_to_use_negative:
- The paper reports only accuracy metrics (F1 scores, precision/recall) without per-tool
  runtime, index-size, or memory measurements to tabulate
- The benchmark compares tools on a single reference condition with no variation in
  repetitiveness or scale, making ratio-scaling analysis uninformative
- Raw timing logs or profiling output must be produced from scratch rather than extracted
  from published tables (use actual re-benchmarking instead)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: SPUMONI
  role: 'Subject tool: rapid read classification using pseudomatching lengths; its
    throughput, index size, and peak RSS are tabulated and compared'
  repo: https://github.com/oma219/spumoni
- name: minimap2
  role: Baseline aligner against which SPUMONI's throughput, index-size, and memory
    ratios are computed
- name: PBSIM
  role: Simulator used to generate the simulated nanopore reads benchmarked in the
    ZymoMC experiments
  repo: https://github.com/yukiteruono/pbsim2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:016eee6f6ecdd51e5423ff10735761b0ece9717c8d1f365588b5049869923cef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - PBSIM
  - SPUMONI
  - minimap2
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# bioinformatics-tool-benchmark-tabulation

## Summary

Tabulates comparative benchmark metrics (throughput, index size, peak RSS) for two or more bioinformatics tools from published result tables, computing ratio summaries that quantify performance scaling across experimental conditions (e.g., reference repetitiveness).

## When to use

Use when a paper reports per-tool benchmark numbers (bases/second throughput, index size, peak RSS) in tables and you need to compute and verify cross-tool performance ratios across experiments.

## When NOT to use

- The paper reports only accuracy metrics (F1 scores, precision/recall) without per-tool runtime, index-size, or memory measurements to tabulate
- The benchmark compares tools on a single reference condition with no variation in repetitiveness or scale, making ratio-scaling analysis uninformative
- Raw timing logs or profiling output must be produced from scratch rather than extracted from published tables (use actual re-benchmarking instead)

## Inputs

- Published benchmark result tables (e.g., Tables 2-4) reporting throughput (bases/second), index size, and peak RSS per tool per experiment
- Datasets: simulated ZymoMC mock-community reads (generated with PBSIM), real ZymoMC mock-community nanopore reads, and human microbiome 3-genome reference data

## Outputs

- Summary CSV of cross-tool ratios (e.g., spumoni_minimap2_ratios_summary.csv) with throughput, index-size, and peak-RSS ratios ordered by reference repetitiveness
- Verified ratio values (e.g., ~19.3x vs ~1.8x throughput scaling with repetitiveness)

## How to apply

Extract per-dataset reported values — throughput in bases/second, index size, and peak RSS — for each tool (here SPUMONI and minimap2) from the paper's tables covering the simulated ZymoMC, real ZymoMC mock-community, and human microbiome experiments. Load these into a pandas DataFrame, computing throughput advantage ratios (minimap2 time / SPUMONI time), index-size ratios, and peak-RSS ratios. Verify computed ratios against reported claims: ~19.3x throughput advantage on simulated ZymoMC with the many-strain pan-genomic (31 GB/16 GB) index versus ~1.8x with the single-genome (56 MB/28 MB) index, ~11.9x on real ZymoMC, ~12x for the mock community, ~16x smaller many-strain index, ~4x lower peak RSS for microbial references, and ~1.86x larger peak RSS for three human genomes (18 GB vs 21 GB). Assemble all ratios into a single table ordered by reference repetitiveness (few-strain microbial, many-strain microbial, multi-genome human) and export as CSV (e.g., spumoni_minimap2_ratios_summary.csv).

## Related tools

- **SPUMONI** (Subject tool: rapid read classification using pseudomatching lengths; its throughput, index size, and peak RSS are tabulated and compared) — https://github.com/oma219/spumoni
- **minimap2** (Baseline aligner against which SPUMONI's throughput, index-size, and memory ratios are computed)
- **PBSIM** (Simulator used to generate the simulated nanopore reads benchmarked in the ZymoMC experiments) — https://github.com/yukiteruono/pbsim2

## Examples

```
df = pd.read_csv('tables_2_4_extracted.csv'); df['throughput_ratio'] = df['minimap2_time_s'] / df['spumoni_time_s']; df.sort_values('repetitiveness').to_csv('spumoni_minimap2_ratios_summary.csv', index=False)
```

## Evaluation signals

- Computed throughput ratio for simulated ZymoMC pan-genomic index matches the reported ~19.3x
- Single-genome index throughput ratio matches the reported ~1.8x, demonstrating ratio grows with reference repetitiveness
- Index-size ratio for the many-strain microbial reference reproduces the ~16x smaller SPUMONI index (31 GB vs 16 GB)
- Peak-RSS ratios reproduce ~4x lower footprint on microbial references and ~1.86x larger footprint for three human genomes (18 GB vs 21 GB)
- Output CSV contains one row per reference condition, ordered few-strain microbial, many-strain microbial, multi-genome human

## Limitations

- Throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases
- A single KS-stat threshold was used for all SPUMONI experiments; the optimal threshold depends on read accuracy and the reference used, with no per-experiment thresholding provided
- SPUMONI version number is not stated in the paper, complicating exact reproducibility of the benchmarked numbers

## Evidence

- [abstract] Throughput advantage grows with reference repetitiveness: "However SPUMONI is 12 times faster than minimap2."
- [abstract] Index and memory footprint ratios: "SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively."
- [discussion] Human-reference scaling limitation: "we expect our throughput advantage to improve as the number of human genomes in the reference increases"
- [discussion] Human peak-RSS comparison: "SPUMONI's peak RSS for three human genomes is about 1.86X larger than the minimap2's peak RSS"
- [methods] Tool availability for reproduction: "git clone https://github.com/oma219/spumoni"
