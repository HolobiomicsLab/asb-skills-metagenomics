---
name: inter-contig-3d-ratio-estimation
description: Use when you have cleaned Hi-C paired-end FASTQ reads (e.g. quality-
  and adapter-trimmed with bbduk, duplicates removed with clumpify.sh) and need a
  reference-free metric of Hi-C ligation quality before binning.
when_to_use_negative:
- Input is raw, untrimmed Hi-C FASTQ — the 3D ratio must be computed on reads preprocessed
  with bbduk (trimq=10 qtrim=r ftm=5 minlen=50, ftl=10) and clumpify.sh deduplication,
  otherwise the metric is not comparable to reported values
- You already have a reference assembly/BAM and want alignment-based Hi-C QC rather
  than the reference-free k-mer mode of qc3C
- The goal is downstream viral genome binning itself (ViralCC pipeline) rather than
  assessing Hi-C library quality
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: qc3C
  role: Run in k-mer (reference-free) mode on processed Hi-C reads to compute the
    inter-contig (3D) ratio and its confidence interval, quantifying Hi-C ligation
    quality
- name: BBTools (bbduk)
  role: Quality-trims Hi-C reads (trimq=10 qtrim=r ftm=5 minlen=50) and trims the
    first 10 nucleotides (ftl=10) prior to qc3C
- name: BBTools (clumpify.sh)
  role: Removes identical PCR optical and tile-edge duplicates from Hi-C reads before
    qc3C
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:9dafbf290a1aa2da2a603274a3fca83679e518f3280a3c7dcc3ec5963a3ccc50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BBTools (bbduk)
  - BBTools (clumpify.sh)
  - qc3C
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# inter-contig-3d-ratio-estimation

## Summary

Estimates the inter-contig (3D) ratio of metagenomic Hi-C libraries with qc3C in k-mer (reference-free) mode, as a quantitative measure of Hi-C ligation/library quality. In the ViralCC paper this was applied to human gut, cow fecal, and wastewater Hi-C datasets after bbduk/clumpify preprocessing.

## When to use

Use when you have cleaned Hi-C paired-end FASTQ reads (e.g. quality- and adapter-trimmed with bbduk, duplicates removed with clumpify.sh) and need a reference-free metric of Hi-C ligation quality before binning.

## When NOT to use

- Input is raw, untrimmed Hi-C FASTQ — the 3D ratio must be computed on reads preprocessed with bbduk (trimq=10 qtrim=r ftm=5 minlen=50, ftl=10) and clumpify.sh deduplication, otherwise the metric is not comparable to reported values
- You already have a reference assembly/BAM and want alignment-based Hi-C QC rather than the reference-free k-mer mode of qc3C
- The goal is downstream viral genome binning itself (ViralCC pipeline) rather than assessing Hi-C library quality

## Inputs

- Cleaned Hi-C paired-end FASTQ reads (quality-trimmed with bbduk: trimq=10 qtrim=r ftm=5 minlen=50, ftl=10)
- Duplicate-removed Hi-C reads (clumpify.sh, PCR optical and tile-edge duplicates)
- Per-dataset identifiers (human gut, cow fecal, wastewater metagenomic Hi-C libraries)

## Outputs

- Per-dataset inter-contig (3D) ratio from qc3C k-mer mode output
- Per-dataset confidence-interval midpoint for the 3D ratio
- Summary table qc3c_3d_ratios.csv compiling the three 3D ratios and CI midpoints

## How to apply

Start from Hi-C paired-end FASTQ reads that have been quality-trimmed with bbduk (trimq=10 qtrim=r ftm=5 minlen=50), had the first 10 nucleotides removed (ftl=10), and had identical PCR optical/tile-edge duplicates removed with clumpify.sh. Run qc3C in k-mer (reference-free) mode on each dataset's processed Hi-C reads to assess ligation quality without needing an assembly or reference. Extract the inter-contig (3D) ratio and its confidence-interval midpoint from the qc3C output for each dataset. Compile the per-dataset 3D ratios and CI midpoints into a single summary table (e.g. qc3c_3d_ratios.csv). Validate by comparing against reported values — for the ViralCC datasets these were 23.3% (human gut), 38.3% (cow fecal), and 54.9% (wastewater), with CI midpoints 5.938%, 52.07%, and 30.66%.

## Related tools

- **qc3C** (Run in k-mer (reference-free) mode on processed Hi-C reads to compute the inter-contig (3D) ratio and its confidence interval, quantifying Hi-C ligation quality)
- **BBTools (bbduk)** (Quality-trims Hi-C reads (trimq=10 qtrim=r ftm=5 minlen=50) and trims the first 10 nucleotides (ftl=10) prior to qc3C)
- **BBTools (clumpify.sh)** (Removes identical PCR optical and tile-edge duplicates from Hi-C reads before qc3C)

## Examples

```
bbduk.sh in1=ERR2282092.1_1.fastq.gz in2=ERR2282092.1_2.fastq.gz trimq=10 qtrim=r ftm=5 minlen=50 ftl=10
```

## Evaluation signals

- The computed inter-contig 3D ratios reproduce the reported values of 23.3% (human gut), 38.3% (cow fecal), and 54.9% (wastewater) (Supplementary Table 7)
- The extracted confidence-interval midpoints match 5.938% (human gut), 52.07% (cow fecal), and 30.66% (wastewater)
- qc3C was run in k-mer (reference-free) mode, not alignment/reference mode
- The three per-dataset 3D ratios and CI midpoints are compiled into a single qc3c_3d_ratios.csv summary table
- Reads fed to qc3C trace back to the documented preprocessing chain: bbduk quality trimming + ftl=10 first-nucleotide trimming + clumpify.sh duplicate removal (BBTools v37.25 in the reproduction)

## Limitations

- The 3D ratio is a library-quality metric only; it does not by itself validate downstream binning results and must be paired with other QC or benchmarking checks
- Results are dataset-dependent: the three libraries spanned 23.3%–54.9%, so 3D ratios are not comparable across studies without the same preprocessing and qc3C mode
- Values were reported with confidence-interval midpoints (e.g. 5.938%, 52.07%, 30.66%), indicating uncertainty in the estimates that should be reported alongside the point estimate

## Evidence

- [methods] reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`: "reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`"
- [methods] Identical PCR optical and tile-edge duplicates for Hi-C reads were removed by the script `clumpify.sh` from `BBTools` suite.: "Identical PCR optical and tile-edge duplicates for Hi-C reads were removed by the script `clumpify.sh` from `BBTools` suite."
- [methods] the first 10 nucleotides of Hi-C reads are trimmed by `bbduk` with parameter `ftl=10`: "the first 10 nucleotides of Hi-C reads are trimmed by `bbduk` with parameter `ftl=10`"
- [other] Run qc3C in k-mer (reference-free) mode on each dataset's processed Hi-C reads to assess Hi-C ligation quality.: [evidence span withheld — anchor status: not_found]
- [readme] bbduk.sh and clumpify.sh command from BBTools suite: v37.25: [evidence span withheld — anchor status: external_doc]
