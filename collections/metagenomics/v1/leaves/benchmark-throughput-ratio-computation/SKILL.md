---
name: benchmark-throughput-ratio-computation
description: Use when comparing read-classification tools' benchmark tables (bases/second
  throughput, index size, peak RSS) and you need quantitative advantage ratios across
  references of increasing repetitiveness, e.g. single-genome vs many-strain pan-genome
  indexes.
when_to_use_negative:
- The goal is to rerun the raw alignment/classification benchmarks rather than recompute
  or verify ratios from already-published per-tool measurements
- The comparison is between tools other than SPUMONI and minimap2 and no equivalent
  paired throughput/index-size/peak-RSS measurements exist for both
- The input is raw nanopore signal or basecalled FASTQ reads — this skill consumes
  tabular benchmark figures, not sequencing data
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: pandas
  role: Used to compute the throughput-advantage, index-size, and peak-RSS ratios
    from the extracted benchmark values and to assemble the summary CSV
- name: SPUMONI
  role: Tool whose throughput, index size, and peak RSS are compared as the numerator
    of the advantage ratios
  repo: https://github.com/oma219/spumoni
- name: minimap2
  role: Baseline aligner against which SPUMONI's throughput, index size, and peak
    RSS are compared (denominator of the ratios)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2021.102696
    title: Pan-genomic matching statistics for targeted nanopore sequencing
  source_content_hash: sha256:dea64019abd1fac6e6a757957f7b8910f525c9cb609290d16878dd28a8bc62f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oma219/spumoni
  tools:
  - SPUMONI
  - minimap2
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# benchmark-throughput-ratio-computation

## Summary

Computes and cross-checks throughput, index-size, and peak-RSS ratios between SPUMONI and minimap2 from published benchmark tables (simulated ZymoMC, real ZymoMC mock community, and 3-genome human microbiome experiments), assembling the ratios into a single summary CSV ordered by reference repetitiveness.

## When to use

Use when comparing read-classification tools' benchmark tables (bases/second throughput, index size, peak RSS) and you need quantitative advantage ratios across references of increasing repetitiveness, e.g. single-genome vs many-strain pan-genome indexes.

## When NOT to use

- The goal is to rerun the raw alignment/classification benchmarks rather than recompute or verify ratios from already-published per-tool measurements
- The comparison is between tools other than SPUMONI and minimap2 and no equivalent paired throughput/index-size/peak-RSS measurements exist for both
- The input is raw nanopore signal or basecalled FASTQ reads — this skill consumes tabular benchmark figures, not sequencing data

## Inputs

- Published benchmark tables (Tables 2-4) reporting throughput (bases/second), index size, and peak RSS for SPUMONI and minimap2
- Dataset labels distinguishing simulated ZymoMC, real ZymoMC mock community, and human 3-genome experiments
- Index descriptors (single-genome 56 MB/28 MB vs pan-genomic 31 GB/16 GB, human 18 GB vs 21 GB)

## Outputs

- spumoni_minimap2_ratios_summary.csv containing throughput-advantage, index-size, and peak-RSS ratios ordered by reference repetitiveness

## How to apply

Extract per-dataset reported throughput (bases/second), index size, and peak RSS for SPUMONI and minimap2 from Tables 2-4 of the pan-genomic matching statistics paper, covering the simulated ZymoMC, real ZymoMC mock-community, and human 3-genome experiments. Compute throughput advantage ratios as minimap2 time / SPUMONI time with pandas, then index-size and peak-RSS ratios as minimap2 value / SPUMONI value. Verify the ratios against the paper's reported figures: about 19.3x speedup with the pan-genomic (31 GB/16 GB) many-strain index versus about 1.8x with the single-genome (56 MB/28 MB) index on simulated ZymoMC reads, about 11.9x on real ZymoMC, and about 12x on the mock community; about 16x smaller many-strain microbial index; about 4x lower peak RSS for microbial references; and about 1.86x larger peak RSS for three human genomes (18 GB SPUMONI vs 21 GB minimap2). Assemble all ratios into one table ordered by reference repetitiveness (few-strain microbial, many-strain microbial, multi-genome human) and write it as spumoni_minimap2_ratios_summary.csv.

## Related tools

- **pandas** (Used to compute the throughput-advantage, index-size, and peak-RSS ratios from the extracted benchmark values and to assemble the summary CSV)
- **SPUMONI** (Tool whose throughput, index size, and peak RSS are compared as the numerator of the advantage ratios) — https://github.com/oma219/spumoni
- **minimap2** (Baseline aligner against which SPUMONI's throughput, index size, and peak RSS are compared (denominator of the ratios))

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('benchmarks.csv'); df['throughput_advantage']=df.minimap2_time/df.spumoni_time; df.sort_values('reference_repetitiveness').to_csv('spumoni_minimap2_ratios_summary.csv', index=False)"
```

## Evaluation signals

- Simulated ZymoMC throughput advantage must be about 19.3x for the pan-genomic index and about 1.8x for the single-genome index, confirming speedup grows with reference repetitiveness
- Real ZymoMC and mock-community throughput advantages must land near 11.9x and 12x respectively
- Index-size ratio for the many-strain microbial reference must be about 16x in minimap2's disfavor; human 3-genome indexes must compare as 18 GB (SPUMONI) vs 21 GB (minimap2)
- Peak-RSS ratios must be about 4x lower for SPUMONI on microbial references and about 1.86x larger for three human genomes
- Output CSV must contain all three ratio types ordered by reference repetitiveness (few-strain microbial, many-strain microbial, multi-genome human)

## Limitations

- SPUMONI's throughput advantage over minimap2 is limited for human reads; the authors expect improvement only as the number of human genomes in the reference increases
- The advantage ratios are computed from reported aggregate benchmark figures; the paper does not name specific datasets or accessions in the methods, so the underlying per-read data cannot be independently re-derived from the article alone
- Peak-RSS comparison reverses direction for the human 3-genome reference (SPUMONI uses about 1.86x more memory), so a single monotonic 'SPUMONI is smaller/faster' assumption is wrong

## Evidence

- [abstract] However SPUMONI is 12 times faster than minimap2.: "However SPUMONI is 12 times faster than minimap2."
- [abstract] SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively.: "SPUMONI's index and peak memory footprint are also 16 to 4 times smaller than those of minimap2, respectively."
- [discussion] SPUMONI's peak RSS for three human genomes is about 1.86X larger than the minimap2's peak RSS: "SPUMONI's peak RSS for three human genomes is about 1.86X larger than the minimap2's peak RSS"
- [discussion] we expect our throughput advantage to improve as the number of human genomes in the reference increases: "we expect our throughput advantage to improve as the number of human genomes in the reference increases"
