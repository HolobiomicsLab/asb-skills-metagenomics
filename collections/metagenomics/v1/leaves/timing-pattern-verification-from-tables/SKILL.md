---
name: timing-pattern-verification-from-tables
description: Use when a publication claims a speed ordering across tools on multiple
  datasets and you hold a reproduced runtime table (seconds per pipeline per dataset)
  that must be checked against the claimed pattern.
when_to_use_negative:
- Input is a raw OTU table or FASTQ reads rather than a pre-computed runtime/seconds
  table — this skill ranks reported timings, not measures them
- The benchmark contains more or fewer than the four pipelines (DADA2, deblur, source
  CD-HIT, CDSnake) per row, since validation requires exactly four ranked pipelines
  per dataset row
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0637
tools:
- name: deblur
  role: QIIME2-based pipeline expected to be the fastest in every tested case; its
    runtime is the rank-1 reference
- name: DADA2
  role: QIIME2-based pipeline compared against CDSnake and deblur runtimes per dataset
    row
- name: CD-HIT-OTU-Miseq
  role: Source utilities whose runtime is benchmarked as one of the slowest on Extreme
- name: CDSnake
  role: Snakemake-wrapped CD-HIT pipeline whose runtimes are compared against the
    other three tools
  repo: github.com/ydkondratenko/cdsnake
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-020-03591-6
    title: 'CDSnake: Snakemake pipeline for retrieval of annotated OTUs from paired-end
      reads using CD-HIT utilities'
  source_content_hash: sha256:0a5dc2e178bcccd64c7ffd847c8d105fbaf3cb52f9ce955a4ee373d16feb8b48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-OTU-Miseq
  - CDSnake
  - DADA2
  - deblur
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Timing pattern verification from benchmark tables

## Summary

Verifies claimed runtime rankings of competing pipelines (DADA2, deblur, source CD-HIT-OTU-MiSeq utilities, CDSnake) by loading reproduced Table 2 runtime measurements in seconds for the Balanced, HMP, Extreme mock community datasets and the pooled 'All' set, sorting each row, and checking reported patterns hold.

## When to use

Use when a publication claims a speed ordering across tools on multiple datasets and you hold a reproduced runtime table (seconds per pipeline per dataset) that must be checked against the claimed pattern.

## When NOT to use

- Input is a raw OTU table or FASTQ reads rather than a pre-computed runtime/seconds table — this skill ranks reported timings, not measures them
- The benchmark contains more or fewer than the four pipelines (DADA2, deblur, source CD-HIT, CDSnake) per row, since validation requires exactly four ranked pipelines per dataset row

## Inputs

- Reproduced Table 2 runtime measurements (seconds) with rows for Balanced, HMP, Extreme, and 'All' datasets and columns for DADA2, deblur, source CD-HIT, and CDSnake
- List of claimed runtime patterns to verify (e.g. 'deblur is fastest in every tested case')

## Outputs

- pipeline_runtime_ranking.csv containing a per-dataset ranking of the four pipelines by ascending runtime plus boolean flags indicating whether each claimed pattern holds

## How to apply

Load the reproduced Table 2 runtime measurements in seconds for DADA2, deblur, the source CD-HIT utilities, and CDSnake across the Balanced, HMP, Extreme mock datasets and the pooled 'All' row. For each dataset row, sort the four pipelines by ascending runtime to compute speed ranks, then evaluate each claimed pattern as a boolean flag: deblur attains the minimum runtime in every tested case (e.g. 1371.72 s on HMP); CDSnake (2263.83 s) is slower than DADA2 (4614.81 s) on HMP and takes longer than DADA2 on the pooled All set; the source CD-HIT utilities (2149.59 s on HMP) and CDSnake take substantially longer than the QIIME2-based pipelines on the Extreme dataset. Write the per-dataset ranking table with pattern-hold flags to pipeline_runtime_ranking.csv. Accept the output only if computed ranks match Table 2 values and each row contains exactly four ranked pipelines.

## Related tools

- **deblur** (QIIME2-based pipeline expected to be the fastest in every tested case; its runtime is the rank-1 reference)
- **DADA2** (QIIME2-based pipeline compared against CDSnake and deblur runtimes per dataset row)
- **CD-HIT-OTU-Miseq** (Source utilities whose runtime is benchmarked as one of the slowest on Extreme)
- **CDSnake** (Snakemake-wrapped CD-HIT pipeline whose runtimes are compared against the other three tools) — github.com/ydkondratenko/cdsnake

## Examples

```
python verify_runtime_ranks.py --runtimes table2_seconds.csv --patterns 'deblur_min_all,cdsnake_slower_dada2_hmp,cdhit_longest_extreme' --output pipeline_runtime_ranking.csv
```

## Evaluation signals

- deblur attains the minimum runtime in every dataset row (e.g. 1371.72 s on HMP)
- CDSnake is slower than DADA2 on HMP (2263.83 s vs 4614.81 s per Table 2) and takes longer than DADA2 on the pooled All set
- Source CD-HIT utilities and CDSnake have substantially longer runtimes than DADA2/deblur on the Extreme dataset
- Each dataset row (Balanced, HMP, Extreme, All) contains exactly four ranked pipelines
- Computed ranks match the published Table 2 values for all rows

## Limitations

- Hardware/execution environment for the time benchmarking is not specified in the article, so absolute runtimes may not transfer to other systems
- Software versions are not stated for DADA2, deblur, CD-HIT-OTU-MiSeq, or Snakemake, limiting reproducibility of the timing comparison
- No accessions or availability details are given for the three mock community datasets, so the timing table cannot be independently regenerated from raw data

## Evidence

- [discussion] Table 2 runtime values: "Time, secDADA2deblursource CD-HITCDSnakeHMP4614.81371.722149.592263.83"
- [abstract] CDSnake slower but richer output: "Though generally slower than other pipelines, CDSnake outputted higher total counts, preserving more information from raw data."
- [readme] Pipeline execution command: "To run pipeline with default Snakemake settings, use command snakemake. To modify general parameters of pipeline (such as number of cores to use) use Snakemake parameters"
