---
name: qc3c-kmer-mode-ligation-quality-assessment
description: Use when you have cleaned Hi-C paired-end FASTQ reads (no reference assembly
  yet) and need a reference-free metric of Hi-C ligation/library quality, such as
  the inter-contig 3D ratio.
when_to_use_negative:
- You want Hi-C binning output (viral genomes or host pairs) rather than a library-quality
  metric — use the ViralCC pipeline instead.
- You already have aligned Hi-C BAM files mapped to assembled contigs; qc3C k-mer
  mode is reference-free and operates on cleaned FASTQ reads.
- Reads are raw and unprocessed — the 3D ratio should be computed on reads cleaned
  with the bbduk (trimq=10 qtrim=r ftm=5 minlen=50, ftl=10) and clumpify.sh steps
  first.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: qc3C
  role: Run in k-mer (reference-free) mode on processed Hi-C reads to assess Hi-C
    ligation quality and extract the inter-contig (3D) ratio with confidence intervals
- name: BBTools (bbduk)
  role: Quality-trims Hi-C reads (trimq=10 qtrim=r ftm=5 minlen=50), removes the first
    10 nucleotides (ftl=10), and removes adaptor sequences to produce cleaned input
    for qc3C
- name: clumpify.sh
  role: Removes identical PCR optical and tile-edge duplicate Hi-C reads prior to
    qc3C quality assessment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:3e16c64325196a192ef643eb21cebbe3b88b3c9f28d4cbfe5828c8ddeed59c82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBTools (bbduk)
  - clumpify.sh
  - qc3C
  - qc3C (k-mer mode)
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# qc3C k-mer mode Hi-C ligation quality assessment

## Summary

Runs qc3C in reference-free k-mer mode on quality-cleaned Hi-C paired-end FASTQ reads to compute the inter-contig (3D) ratio and its confidence-interval midpoint as a measure of Hi-C library ligation quality. In the ViralCC paper this quantifies library quality for the human gut, cow fecal, and wastewater metagenomic Hi-C datasets.

## When to use

Use when you have cleaned Hi-C paired-end FASTQ reads (no reference assembly yet) and need a reference-free metric of Hi-C ligation/library quality, such as the inter-contig 3D ratio.

## When NOT to use

- You want Hi-C binning output (viral genomes or host pairs) rather than a library-quality metric — use the ViralCC pipeline instead.
- You already have aligned Hi-C BAM files mapped to assembled contigs; qc3C k-mer mode is reference-free and operates on cleaned FASTQ reads.
- Reads are raw and unprocessed — the 3D ratio should be computed on reads cleaned with the bbduk (trimq=10 qtrim=r ftm=5 minlen=50, ftl=10) and clumpify.sh steps first.

## Inputs

- Cleaned Hi-C paired-end FASTQ reads (quality-trimmed with bbduk trimq=10 qtrim=r ftm=5 minlen=50, ftl=10, and duplicate-removed with clumpify.sh)
- Per-dataset raw Hi-C FASTQ files (e.g. SRA runs ERR2282092, ERR2530126, ERR2530127 from the ViralCC study)

## Outputs

- Per-dataset qc3C k-mer mode report containing the inter-contig (3D) ratio and confidence-interval midpoint
- Summary CSV (qc3c_3d_ratios.csv) with 3D ratios and CI midpoints per dataset (human gut 23.3%/5.938%, cow fecal 38.3%/52.07%, wastewater 54.9%/30.66%)

## How to apply

Start from Hi-C paired-end reads that were quality-trimmed with bbduk (trimq=10 qtrim=r ftm=5 minlen=50), had the first 10 nucleotides removed (ftl=10), and had identical PCR optical/tile-edge duplicates removed with clumpify.sh from the BBTools suite. Run qc3C in k-mer (reference-free) mode on each dataset's processed Hi-C reads so that ligation quality can be assessed without aligning to an assembly. From each qc3C output, extract the inter-contig (3D) ratio and its confidence-interval midpoint. Compile the per-dataset 3D ratios and CI midpoints into a summary table (e.g. qc3c_3d_ratios.csv). For the ViralCC datasets the expected reproducibility targets are 3D ratios of 23.3% (human gut), 38.3% (cow fecal), and 54.9% (wastewater) with CI midpoints 5.938%, 52.07%, and 30.66%, reported in Supplementary Table 7.

## Related tools

- **qc3C** (Run in k-mer (reference-free) mode on processed Hi-C reads to assess Hi-C ligation quality and extract the inter-contig (3D) ratio with confidence intervals)
- **BBTools (bbduk)** (Quality-trims Hi-C reads (trimq=10 qtrim=r ftm=5 minlen=50), removes the first 10 nucleotides (ftl=10), and removes adaptor sequences to produce cleaned input for qc3C)
- **clumpify.sh** (Removes identical PCR optical and tile-edge duplicate Hi-C reads prior to qc3C quality assessment)

## Examples

```
bbduk.sh in1=HiC_R1.fastq.gz in2=HiC_R2.fastq.gz trimq=10 qtrim=r ftm=5 minlen=50 ftl=10; clumpify.sh in=trimmed.fastq.gz out=cleaned.fastq.gz dedupe; qc3C kmer -f cleaned.fastq.gz
```

## Evaluation signals

- Reproduced 3D ratios match 23.3% (human gut), 38.3% (cow fecal), and 54.9% (wastewater) as reported in Supplementary Table 7
- Confidence-interval midpoints match 5.938%, 52.07%, and 30.66% for the human gut, cow fecal, and wastewater datasets respectively
- Input reads were cleaned with the exact bbduk parameters (trimq=10 qtrim=r ftm=5 minlen=50, ftl=10) and clumpify.sh duplicate removal before running qc3C
- qc3C was run in k-mer (reference-free) mode, not reference mode
- All three per-dataset 3D ratios and CI midpoints are present in the compiled qc3c_3d_ratios.csv summary table

## Limitations

- The 3D ratio is a library-quality metric only; it does not by itself recover viral genomes or virus-host pairs.
- Results depend on the exact read-cleaning protocol (bbduk trimq=10 qtrim=r ftm=5 minlen=50 with ftl=10, plus clumpify.sh duplicate removal); deviations change the computed 3D ratio.
- qc3C k-mer mode is reference-free, so it does not leverage assembly contigs or alignment information.

## Evidence

- [methods] Read cleaning precedes qc3C: "reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`"
- [methods] First 10 nt trimming of Hi-C reads: "the first 10 nucleotides of Hi-C reads are trimmed by `bbduk` with parameter `ftl=10`"
- [methods] Duplicate removal with clumpify.sh: "Identical PCR optical and tile-edge duplicates for Hi-C reads were removed by the script `clumpify.sh` from `BBTools` suite."
- [readme] Software versions used in the reproducibility workflow: [evidence span withheld — anchor status: external_doc]
