---
name: metagenomic-hic-multi-dataset-quality-benchmarking
description: Use when you have cleaned Hi-C paired-end FASTQ reads from several metagenomic
  Hi-C datasets and need a reference-free, quantitative measure of ligation/library
  quality comparable across datasets.
when_to_use_negative:
- You only need Hi-C library quality for a single dataset and no cross-dataset comparison
  — a single qc3C run suffices without the multi-dataset compilation table.
- You need contig-level binning results rather than library-quality metrics — use
  ViralCC (or another Hi-C binning tool) on assembled contigs and aligned BAM files
  instead.
- Only raw, untrimmed FASTQ is available and no access to BBTools is possible — qc3C
  on unprocessed reads will not reproduce the published 3D ratios.
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: qc3C
  role: k-mer (reference-free) mode is used on processed Hi-C reads to compute the
    inter-contig (3D) ratio and confidence intervals for ligation quality
- name: bbduk
  role: quality trimming (trimq=10 qtrim=r ftm=5 minlen=50) and trimming of the first
    10 nucleotides of Hi-C reads (ftl=10) before qc3C
  repo: https://github.com/dyxstat/Reproduce_ViralCC
- name: clumpify.sh
  role: removes identical PCR optical and tile-edge duplicates from Hi-C reads as
    part of the preprocessing step
  repo: https://github.com/dyxstat/Reproduce_ViralCC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:5970c88c527231a9e18301a4fa8e16bd4dff5a049d595f2b33e8ff6ad3128a1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/Reproduce_ViralCC
  tools:
  - bbduk
  - clumpify.sh
  - qc3C
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# metagenomic-hic-multi-dataset-quality-benchmarking

## Summary

Benchmark Hi-C library quality across multiple metagenomic Hi-C datasets by preprocessing paired-end reads (bbduk trimming, clumpify duplicate removal) and running qc3C in k-mer mode to extract inter-contig (3D) ratios and confidence-interval midpoints, as done for the human gut, cow fecal, and wastewater datasets in the ViralCC paper.

## When to use

Use when you have cleaned Hi-C paired-end FASTQ reads from several metagenomic Hi-C datasets and need a reference-free, quantitative measure of ligation/library quality comparable across datasets.

## When NOT to use

- You only need Hi-C library quality for a single dataset and no cross-dataset comparison — a single qc3C run suffices without the multi-dataset compilation table.
- You need contig-level binning results rather than library-quality metrics — use ViralCC (or another Hi-C binning tool) on assembled contigs and aligned BAM files instead.
- Only raw, untrimmed FASTQ is available and no access to BBTools is possible — qc3C on unprocessed reads will not reproduce the published 3D ratios.

## Inputs

- cleaned Hi-C paired-end FASTQ reads (gzip), quality-trimmed and duplicate-removed
- raw Hi-C paired-end FASTQ from SRA (e.g. ERR2282092, ERR2530126, ERR2530127) retrieved with fastq-dump

## Outputs

- qc3C per-dataset output containing the inter-contig (3D) ratio and confidence-interval midpoint
- qc3c_3d_ratios.csv summary table of 3D ratios and CI midpoints for all datasets

## How to apply

First preprocess each dataset's Hi-C FASTQ pairs: quality-trim with bbduk using trimq=10 qtrim=r ftm=5 minlen=50, remove the first 10 nucleotides of Hi-C reads with ftl=10, and strip identical PCR optical and tile-edge duplicates with clumpify.sh from the BBTools suite (as in the ViralCC Reproduce_ViralCC pipeline, BBTools v37.25). Then run qc3C in k-mer (reference-free) mode on each dataset's processed Hi-C reads to assess Hi-C ligation quality without requiring a reference assembly. Extract the inter-contig (3D) ratio and its confidence-interval midpoint from each qc3C output, and compile the per-dataset values into a single summary table (qc3c_3d_ratios.csv). Validation compares computed values against the reported ratios of 23.3% (human gut), 38.3% (cow fecal), and 54.9% (wastewater) with CI midpoints 5.938%, 52.07%, and 30.66%.

## Related tools

- **qc3C** (k-mer (reference-free) mode is used on processed Hi-C reads to compute the inter-contig (3D) ratio and confidence intervals for ligation quality)
- **bbduk** (quality trimming (trimq=10 qtrim=r ftm=5 minlen=50) and trimming of the first 10 nucleotides of Hi-C reads (ftl=10) before qc3C) — https://github.com/dyxstat/Reproduce_ViralCC
- **clumpify.sh** (removes identical PCR optical and tile-edge duplicates from Hi-C reads as part of the preprocessing step) — https://github.com/dyxstat/Reproduce_ViralCC

## Examples

```
bbduk.sh in1=ERR2282092.1_1.fastq.gz in2=ERR2282092.1_2.fastq.gz trimq=10 qtrim=r ftm=5 minlen=50 ftl=10 && clumpify.sh in1=R1.trim.fastq.gz in2=R2.trim.fastq.gz dedup optical && qc3C-kmer -f R1.dedup.fastq.gz -r R2.dedup.fastq.gz -o qc3c_human_gut.
```

## Evaluation signals

- Computed 3D ratios reproduce the reported values 23.3% (human gut), 38.3% (cow fecal), and 54.9% (wastewater) from Supplementary Table 7.
- Confidence-interval midpoints match the reported 5.938%, 52.07%, and 30.66% for the human gut, cow fecal, and wastewater datasets respectively.
- qc3c_3d_ratios.csv contains exactly three per-dataset rows (human gut, cow fecal, wastewater), each with a 3D ratio and a CI midpoint.
- Preprocessing logs confirm all three filtering steps were applied: bbduk trimq=10 qtrim=r ftm=5 minlen=50, ftl=10, and clumpify.sh duplicate removal.
- qc3C ran successfully in k-mer mode (no reference genome required) for all three datasets.

## Limitations

- The 3D ratio depends on the exact preprocessing chain (bbduk trimq=10 qtrim=r ftm=5 minlen=50, ftl=10, clumpify.sh); altering any parameter will change the computed values.
- NCBI download links for raw SRA data may change over time, requiring updated links from the NCBI website.
- CI midpoints (e.g. 5.938% vs 52.07% vs 30.66%) do not scale monotonically with the point 3D ratios, so the midpoint alone is not a reliable proxy for the ratio.

## Evidence

- [methods] reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`: "reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`"
- [methods] the first 10 nucleotides of Hi-C reads are trimmed by `bbduk` with parameter `ftl=10`: "the first 10 nucleotides of Hi-C reads are trimmed by `bbduk` with parameter `ftl=10`"
- [methods] Identical PCR optical and tile-edge duplicates for Hi-C reads were removed by the script `clumpify.sh` from `BBTools` suite.: "Identical PCR optical and tile-edge duplicates for Hi-C reads were removed by the script `clumpify.sh` from `BBTools` suite."
- [readme] We take the cow fecal datasets for example. The other two datasets were processed following the same procedure.: [evidence span withheld — anchor status: external_doc]
- [readme] bbduk.sh and clumpify.sh command from BBTools suite: v37.25: [evidence span withheld — anchor status: external_doc]
