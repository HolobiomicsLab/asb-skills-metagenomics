---
name: index-size-memory-footprint-comparison
description: Use when comparing read-classification or alignment tools on repetitive
  references and you need to quantify how index size, peak RSS, and bases/second scale
  with reference repetitiveness.
when_to_use_negative:
- The comparison target is a single non-repetitive reference genome, where SPUMONI's
  advantage over minimap2 is only marginal (~1.8x) and the pan-genome index benefit
  does not materialize
- The analysis concerns classification accuracy (F1 scores) rather than resource efficiency
  metrics like index size and peak memory
- Input is raw nanopore signal rather than per-tool benchmark statistics — SPUMONI
  operates on base-called sequences only
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0102
tools:
- name: SPUMONI
  role: Compressed pan-genome index tool whose throughput, index size, and peak RSS
    are measured against minimap2
  repo: github.com/oma219/spumoni
- name: minimap2
  role: Read aligner baseline against which SPUMONI's resource-efficiency ratios are
    computed
- name: MONI
  role: Batch matching-statistics algorithm that SPUMONI replaces with an online PML
    algorithm; relevant as an efficiency baseline
  repo: github.com/maxrossi91/moni
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:eda8bdcc53c2bc1874bf7c6b484367f5478c9facecd19f36bb4ecd676b7d1c9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MONI
  - SPUMONI
  - minimap2
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# index-size-memory-footprint-comparison

## Summary

Quantifies the resource-efficiency advantage of SPUMONI's pan-genomic compressed indexes over minimap2 by computing throughput, index-size, and peak-RSS ratios across experiments of increasing reference repetitiveness (single-genome vs many-strain microbial pan-genomes vs multi-genome human references).

## When to use

Use when comparing read-classification or alignment tools on repetitive references and you need to quantify how index size, peak RSS, and bases/second scale with reference repetitiveness.

## When NOT to use

- The comparison target is a single non-repetitive reference genome, where SPUMONI's advantage over minimap2 is only marginal (~1.8x) and the pan-genome index benefit does not materialize
- The analysis concerns classification accuracy (F1 scores) rather than resource efficiency metrics like index size and peak memory
- Input is raw nanopore signal rather than per-tool benchmark statistics — SPUMONI operates on base-called sequences only

## Inputs

- Per-dataset tool benchmark tables (Tables 2-4): throughput (bases/second), index size, and peak RSS for SPUMONI and minimap2
- Simulated ZymoMC mock-community nanopore reads
- Real ZymoMC mock-community nanopore reads
- Human microbiome (3-genome) reference experiment metrics

## Outputs

- spumoni_minimap2_ratios_summary.csv — table of throughput, index-size, and peak-RSS ratios ordered by reference repetitiveness
- Verified speedup ratios (~19.3x simulated, ~11.9x real ZymoMC, ~12x mock community)

## How to apply

Extract per-dataset throughput (bases/second), index size, and peak RSS for SPUMONI and minimap2 from the results tables (Tables 2-4) covering simulated ZymoMC, real ZymoMC mock-community, and human 3-genome experiments. Compute throughput advantage ratios (minimap2 time / SPUMONI time), index-size ratios, and peak-RSS ratios with pandas, verifying reported values: ~19.3x speedup with the many-strain pan-genomic index (31 GB / 16 GB) versus ~1.8x with the single-genome index (56 MB / 28 MB), ~11.9x on real ZymoMC, ~12x on the mock community, a ~16x smaller many-strain microbial index, 18 GB vs 21 GB for three human genomes, ~4x lower RSS for microbial references, and ~1.86x larger SPUMONI RSS for three human genomes. Assemble all ratios into a single table ordered by reference repetitiveness (few-strain microbial, many-strain microbial, multi-genome human) and write it as spumoni_minimap2_ratios_summary.csv.

## Related tools

- **SPUMONI** (Compressed pan-genome index tool whose throughput, index size, and peak RSS are measured against minimap2) — github.com/oma219/spumoni
- **minimap2** (Read aligner baseline against which SPUMONI's resource-efficiency ratios are computed)
- **MONI** (Batch matching-statistics algorithm that SPUMONI replaces with an online PML algorithm; relevant as an efficiency baseline) — github.com/maxrossi91/moni

## Examples

```
import pandas as pd; df = pd.read_csv('tables2_4_metrics.csv'); df['throughput_ratio'] = df['minimap2_time'] / df['spumoni_time']; df.sort_values('reference_repetitiveness').to_csv('spumoni_minimap2_ratios_summary.csv', index=False)
```

## Evaluation signals

- Simulated ZymoMC throughput ratio should be ~19.3x with the many-strain pan-genomic (31 GB/16 GB) index and ~1.8x with the single-genome (56 MB/28 MB) index
- Real ZymoMC throughput ratio should verify to ~11.9x and mock-community to ~12x
- Index-size ratio for the many-strain microbial reference should verify to ~16x smaller than minimap2's
- Human 3-genome comparison should show 18 GB (SPUMONI) vs 21 GB (minimap2) index sizes and SPUMONI peak RSS ~1.86x larger than minimap2's
- Peak-RSS ratio for microbial references should be ~4x lower for SPUMONI; output CSV must contain one row per reference-repetitiveness tier (few-strain microbial, many-strain microbial, multi-genome human)

## Limitations

- Throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases
- SPUMONI's peak RSS for three human genomes is about 1.86X larger than minimap2's, so the memory advantage does not hold for multi-genome human references
- SPUMONI version number is not stated in the methods, complicating exact reproduction of benchmarked ratios

## Evidence

- [abstract] However SPUMONI is 12 times faster than minimap2.: "However SPUMONI is 12 times faster than minimap2."
- [abstract] SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively.: "SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively."
- [discussion] SPUMONI's peak RSS for three human genomes is about 1.86X larger than the minimap2's peak RSS: "SPUMONI's peak RSS for three human genomes is about 1.86X larger than the minimap2's peak RSS"
- [discussion] we expect our throughput advantage to improve as the number of human genomes in the reference increases: "we expect our throughput advantage to improve as the number of human genomes in the reference increases"
