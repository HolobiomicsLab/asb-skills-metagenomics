---
name: confidence-interval-reporting-for-hic-contact-ratios
description: Use when Hi-C library quality must be reported with uncertainty — e.g.
  inter-contig 3D ratios from qc3C k-mer mode across multiple metagenomic Hi-C datasets
  that must be reconciled against published values.
when_to_use_negative:
- The goal is downstream Hi-C-based binning (e.g. ViralCC viral genome recovery) rather
  than Hi-C library quality assessment — the 3D ratio is a QC metric only.
- The input reads are raw unprocessed FASTQ without the specified bbduk/clumpify.sh
  cleaning, since the ratio values will not match the published 23.3%/38.3%/54.9%
  benchmarks.
- Only a single dataset is available and no published reference values exist to validate
  the ratio and CI midpoint against.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: qc3C
  role: Run in k-mer (reference-free) mode on processed Hi-C reads to compute the
    inter-contig 3D ratio and confidence-interval midpoint
- name: BBTools (bbduk)
  role: Quality trimming (trimq=10 qtrim=r ftm=5 minlen=50) and first-10-nucleotide
    trimming (ftl=10) of Hi-C reads prior to qc3C
- name: clumpify.sh
  role: Removal of identical PCR optical and tile-edge duplicates from Hi-C reads
    before qc3C
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:2e23bb9c253eeec09feb534162b6b6231959691240ba2a9fd8894f246284b45d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBTools (bbduk)
  - clumpify.sh
  - qc3C
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# confidence-interval-reporting-for-hic-contact-ratios

## Summary

Quantifies Hi-C library quality by computing the inter-contig (3D) ratio and its confidence-interval midpoint with qc3C in k-mer (reference-free) mode, then compiling per-dataset values into a single summary table (qc3c_3d_ratios.csv) for reproducibility checks.

## When to use

Use when Hi-C library quality must be reported with uncertainty — e.g. inter-contig 3D ratios from qc3C k-mer mode across multiple metagenomic Hi-C datasets that must be reconciled against published values.

## When NOT to use

- The goal is downstream Hi-C-based binning (e.g. ViralCC viral genome recovery) rather than Hi-C library quality assessment — the 3D ratio is a QC metric only.
- The input reads are raw unprocessed FASTQ without the specified bbduk/clumpify.sh cleaning, since the ratio values will not match the published 23.3%/38.3%/54.9% benchmarks.
- Only a single dataset is available and no published reference values exist to validate the ratio and CI midpoint against.

## Inputs

- Cleaned Hi-C paired-end FASTQ reads (quality-trimmed with bbduk trimq=10 qtrim=r ftm=5 minlen=50, first 10 nucleotides removed with ftl=10, PCR optical/tile-edge duplicates removed with clumpify.sh)
- Three metagenomic Hi-C datasets: human gut, cow fecal (e.g. ERR2282092/ERR2530126/ERR2530127 SRA runs), and wastewater

## Outputs

- Per-dataset inter-contig 3D ratio from qc3C k-mer output
- Per-dataset confidence-interval midpoint from qc3C k-mer output
- qc3c_3d_ratios.csv summary table with one 3D ratio and one CI midpoint per dataset

## How to apply

First produce the cleaned Hi-C reads exactly as specified: quality-trim with bbduk (trimq=10 qtrim=r ftm=5 minlen=50), remove the first 10 nucleotides of Hi-C reads (ftl=10), and strip identical PCR optical/tile-edge duplicates with clumpify.sh, since these preprocessing parameters directly affect the computed ratio. Then run qc3C in k-mer (reference-free) mode on each dataset's processed paired-end Hi-C reads and extract two numbers per dataset: the inter-contig (3D) ratio and the midpoint of its confidence interval. Compile the three per-dataset pairs (human gut, cow fecal, wastewater) into qc3c_3d_ratios.csv. Validate by checking the values reproduce the reported 3D ratios of 23.3% (human gut), 38.3% (cow fecal), and 54.9% (wastewater) with CI midpoints 5.938%, 52.07%, and 30.66% respectively. The rationale is that the inter-contig 3D ratio reflects the fraction of Hi-C contacts joining different contigs, a standard proxy for Hi-C ligation quality.

## Related tools

- **qc3C** (Run in k-mer (reference-free) mode on processed Hi-C reads to compute the inter-contig 3D ratio and confidence-interval midpoint)
- **BBTools (bbduk)** (Quality trimming (trimq=10 qtrim=r ftm=5 minlen=50) and first-10-nucleotide trimming (ftl=10) of Hi-C reads prior to qc3C)
- **clumpify.sh** (Removal of identical PCR optical and tile-edge duplicates from Hi-C reads before qc3C)

## Evaluation signals

- Computed 3D ratios reproduce the reported values: 23.3% (human gut), 38.3% (cow fecal), 54.9% (wastewater)
- Computed CI midpoints match 5.938%, 52.07%, and 30.66% respectively (Supplementary Table 7)
- qc3C must be run in k-mer (reference-free) mode, not a reference-based mode, for values to be comparable
- Input FASTQs must show evidence of the specified cleaning (bbduk trimq=10 qtrim=r ftm=5 minlen=50, ftl=10; clumpify.sh dedup) before running qc3C
- qc3c_3d_ratios.csv contains exactly one 3D ratio and one CI midpoint per dataset (three datasets)

## Limitations

- CI midpoints can differ substantially from the 3D ratio point estimate (e.g., human gut ratio 23.3% vs CI midpoint 5.938%), so they should be reported as separate quantities and not used interchangeably.
- Values are sensitive to the exact read-cleaning parameters (ftl=10, trimq=10 qtrim=r ftm=5 minlen=50, clumpify.sh dedup); deviating from them breaks comparability with the published benchmarks.
- The metric was only established for these three metagenomic Hi-C datasets; other datasets may require re-benchmarking rather than direct comparison.

## Evidence

- [methods] reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`: "reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`"
- [methods] Hi-C read first 10 nucleotides trimming: "the first 10 nucleotides of Hi-C reads are trimmed by `bbduk` with parameter `ftl=10`"
- [methods] removal of PCR optical and tile-edge duplicates: "Identical PCR optical and tile-edge duplicates for Hi-C reads were removed by the script `clumpify.sh` from `BBTools` suite."
- [readme] Reproduce_ViralCC raw data acquisition: [evidence span withheld — anchor status: external_doc]
