---
name: pangenome-reference-repetitiveness-analysis
description: Use when comparing read-classification tool performance (bases/second
  throughput, index size, peak RSS) across references of differing repetitiveness,
  e.g. single-genome vs many-strain pan-genome indexes on mock-community reads.
when_to_use_negative:
- The task is raw read classification or alignment itself rather than quantifying
  resource/performance scaling between existing benchmark runs
- Only a single reference of fixed repetitiveness is available, so the throughput-vs-repetitiveness
  trend cannot be computed
- The goal is accuracy evaluation (F1 score, KS-stat thresholding) rather than throughput,
  index-size, or memory-footprint ratios
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SPUMONI
  role: 'Tool under evaluation: its throughput, index size, and peak RSS are measured
    against minimap2 across references of increasing repetitiveness'
  repo: https://github.com/oma219/spumoni
- name: minimap2
  role: Baseline read aligner whose runtime, index size, and peak RSS are the denominators
    for all computed ratios
- name: pbsim2
  role: Simulator used to generate the simulated ZymoMC mock-community nanopore reads
    underlying the per-dataset benchmark metrics
  repo: https://github.com/yukiteruono/pbsim2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:3acc432b5e0ffd74820f9e6032472ee82e94063de149d391590a1cec2cba4299
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - SPUMONI
  - minimap2
  - pbsim2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pangenome-reference-repetitiveness-analysis

## Summary

Quantifies how SPUMONI's throughput, index-size, and peak-RSS advantages over minimap2 scale with reference repetitiveness, using benchmark metrics from simulated ZymoMC, real ZymoMC mock-community, and human 3-genome nanopore experiments.

## When to use

Use when comparing read-classification tool performance (bases/second throughput, index size, peak RSS) across references of differing repetitiveness, e.g. single-genome vs many-strain pan-genome indexes on mock-community reads.

## When NOT to use

- The task is raw read classification or alignment itself rather than quantifying resource/performance scaling between existing benchmark runs
- Only a single reference of fixed repetitiveness is available, so the throughput-vs-repetitiveness trend cannot be computed
- The goal is accuracy evaluation (F1 score, KS-stat thresholding) rather than throughput, index-size, or memory-footprint ratios

## Inputs

- Benchmark metrics (bases/second throughput, index size, peak RSS) from Tables 2-4 for SPUMONI and minimap2
- Simulated ZymoMC mock-community nanopore reads (e.g. generated with pbsim2)
- Real ZymoMC mock-community nanopore read dataset
- Single-genome (56 MB/28 MB), many-strain pan-genomic (31 GB/16 GB), and three-human-genome (18 GB vs 21 GB) reference indexes

## Outputs

- spumoni_minimap2_ratios_summary.csv — throughput, index-size, and peak-RSS ratios ordered by reference repetitiveness

## How to apply

Extract the per-dataset reported throughput (bases/second), index size, and peak RSS for SPUMONI and minimap2 from Tables 2-4, covering simulated ZymoMC, real ZymoMC mock-community, and human 3-genome experiments. Compute throughput advantage ratios (minimap2 time / SPUMONI time) in pandas, verifying the ~19.3x simulated speedup with the many-strain (31 GB/16 GB) pan-genomic index versus only ~1.8x with the single-genome (56 MB/28 MB) index, plus ~11.9x (real ZymoMC) and ~12x (mock community). Compute index-size ratios, checking the ~16x smaller many-strain microbial index and the 18 GB vs 21 GB human 3-genome comparison. Compute peak-RSS ratios, verifying ~4x lower footprint on microbial references and ~1.86x larger footprint for three human genomes. Assemble all ratios into one table ordered by reference repetitiveness (few-strain microbial, many-strain microbial, multi-genome human) and write it as spumoni_minimap2_ratios_summary.csv.

## Related tools

- **SPUMONI** (Tool under evaluation: its throughput, index size, and peak RSS are measured against minimap2 across references of increasing repetitiveness) — https://github.com/oma219/spumoni
- **minimap2** (Baseline read aligner whose runtime, index size, and peak RSS are the denominators for all computed ratios)
- **pbsim2** (Simulator used to generate the simulated ZymoMC mock-community nanopore reads underlying the per-dataset benchmark metrics) — https://github.com/yukiteruono/pbsim2

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('spumoni_tables_2_4.csv'); df['speedup']=df.minimap2_time/df.spumoni_time; df.to_csv('spumoni_minimap2_ratios_summary.csv',index=False)"
```

## Evaluation signals

- Throughput ratio (minimap2 time / SPUMONI time) reproduces ~19.3x on simulated ZymoMC reads with the many-strain pan-genomic index and only ~1.8x with the single-genome index
- Real ZymoMC ratio reproduces ~11.9x and the mock-community ratio ~12x, matching the paper's claimed ~12x speedup
- Index-size ratio confirms the many-strain microbial index is about one-sixteenth the size of minimap2's, and the human 3-genome comparison is 18 GB vs 21 GB
- Peak-RSS ratios confirm ~4x lower footprint for microbial references and ~1.86x larger footprint for three human genomes
- Output CSV spumoni_minimap2_ratios_summary.csv exists, contains all three ratio types, and is ordered by reference repetitiveness (few-strain microbial, many-strain microbial, multi-genome human)

## Limitations

- Throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases
- A single KS-stat threshold is used for all experiments; the optimal threshold is a function of read accuracy and the reference used, and no per-experiment thresholding method is provided
- The null index (reverse of positive-index sequences) may be insufficient when there is substantial sequence similarity between depletion-target reads and non-target reads
- SPUMONI version number is not stated in the methods, complicating exact reproducibility

## Evidence

- [abstract] SPUMONI is 12 times faster than minimap2.: "However SPUMONI is 12 times faster than minimap2."
- [abstract] SPUMONI's index and peak memory footprint are 16 to 4 times smaller.: "SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively."
- [discussion] SPUMONI peak RSS for three human genomes is 1.86X larger.: "SPUMONI's peak RSS for three human genomes is about 1.86X larger than the minimap2's peak RSS"
- [discussion] Throughput advantage expected to improve with more human genomes.: "we expect our throughput advantage to improve as the number of human genomes in the reference increases"
- [discussion] Same KS-stat threshold used across all experiments.: "we use the same KS-stat threshold for all the experiments which was optimized to perform well on real nanopore data sets"
