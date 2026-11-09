---
name: hic-read-cleaning-with-bbduk-and-clumpify
description: Use when raw metagenomic Hi-C paired-end FASTQ reads must be cleaned
  before assembly/alignment or before Hi-C library quality assessment such as qc3C
  3D-ratio estimation.
when_to_use_negative:
- Input reads have already been quality-trimmed and deduplicated — re-running clumpify.sh
  or ftl=10 trimming would remove additional legitimate data
- The data are not proximity-ligation (Hi-C) reads and the first-10-nucleotide ftl=10
  trim is not warranted
- The goal is single-end read processing — the pipeline and its tpe/tbo paired-end
  bbduk parameters assume paired-end input
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: BBTools (bbduk)
  role: Adapter removal (ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo), quality
    trimming (trimq=10 qtrim=r ftm=5 minlen=50), and Hi-C first-10-nt trimming (ftl=10)
  repo: https://github.com/dyxstat/Reproduce_ViralCC
- name: BBTools (clumpify.sh)
  role: Removal of identical PCR optical and tile-edge duplicates from Hi-C reads
  repo: https://github.com/dyxstat/Reproduce_ViralCC
- name: SRA Toolkit (fastq-dump)
  role: Converts downloaded SRA files to gzipped paired-end FASTQ inputs for the cleaning
    steps
  repo: https://github.com/dyxstat/Reproduce_ViralCC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:c41bf7ef862d15296c6be651fa44002b9d444e9598ea8b2e1275ca1649809b16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/Reproduce_ViralCC
  tools:
  - BBTools (bbduk)
  - BBTools (bbduk, clumpify.sh)
  - BBTools (clumpify.sh)
  - SRA Toolkit (fastq-dump)
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hic-read-cleaning-with-bbduk-and-clumpify

## Summary

Standardized cleaning of metagenomic Hi-C paired-end reads using BBTools: adapter/quality trimming with bbduk (trimq=10 qtrim=r ftm=5 minlen=50, ftl=10 for Hi-C) followed by removal of identical PCR optical and tile-edge duplicates with clumpify.sh, as used in the ViralCC Hi-C processing pipeline.

## When to use

Use when raw metagenomic Hi-C paired-end FASTQ reads must be cleaned before assembly/alignment or before Hi-C library quality assessment such as qc3C 3D-ratio estimation.

## When NOT to use

- Input reads have already been quality-trimmed and deduplicated — re-running clumpify.sh or ftl=10 trimming would remove additional legitimate data
- The data are not proximity-ligation (Hi-C) reads and the first-10-nucleotide ftl=10 trim is not warranted
- The goal is single-end read processing — the pipeline and its tpe/tbo paired-end bbduk parameters assume paired-end input

## Inputs

- Raw shotgun paired-end FASTQ reads (e.g. *_1.fastq.gz, *_2.fastq.gz from fastq-dump --split-files --gzip)
- Raw metagenomic Hi-C paired-end FASTQ reads (e.g. HiC1.fastq.gz, HiC2.fastq.gz)

## Outputs

- Cleaned Hi-C paired-end FASTQ files (adapters removed, quality-trimmed, first 10 nt trimmed, PCR optical/tile-edge duplicates removed)
- Cleaned shotgun paired-end FASTQ files (adapters removed, quality-trimmed), suitable for MEGAHIT assembly

## How to apply

Remove adapter sequences from reads using bbduk from the BBTools suite with parameters ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo. Then quality-trim with bbduk using trimq=10 qtrim=r ftm=5 minlen=50, dropping reads shorter than 50 nt after trimming. For Hi-C reads specifically, additionally trim the first 10 nucleotides with ftl=10 to remove potential ligation-junction artifacts. Finally, remove identical PCR optical and tile-edge duplicates with clumpify.sh from the BBTools suite. The cleaned paired-end FASTQ files are the input for downstream steps such as alignment to assembled contigs or qc3C k-mer-mode library quality assessment.

## Related tools

- **BBTools (bbduk)** (Adapter removal (ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo), quality trimming (trimq=10 qtrim=r ftm=5 minlen=50), and Hi-C first-10-nt trimming (ftl=10)) — https://github.com/dyxstat/Reproduce_ViralCC
- **BBTools (clumpify.sh)** (Removal of identical PCR optical and tile-edge duplicates from Hi-C reads) — https://github.com/dyxstat/Reproduce_ViralCC
- **SRA Toolkit (fastq-dump)** (Converts downloaded SRA files to gzipped paired-end FASTQ inputs for the cleaning steps) — https://github.com/dyxstat/Reproduce_ViralCC

## Examples

```
fastq-dump --split-files --gzip ERR2282092.1; bbduk.sh in1=ERR2282092.1_1.fastq.gz in2=ERR2282092.1_2.fastq.gz trimq=10 qtrim=r ftm=5 minlen=50 ftl=10
```

## Evaluation signals

- Reproduced Hi-C library quality: qc3C k-mer-mode inter-contig 3D ratios of ~23.3% (human gut), 38.3% (cow fecal), and 54.9% (wastewater), with CI midpoints 5.938%, 52.07%, and 30.66% (Supplementary Table 7)
- Cleaned FASTQ remains paired-end: read counts match between _1 and _2 files after trimming and deduplication
- No remaining adapter sequence in the cleaned reads (bbduk ktrim=r parameters applied)
- Duplicate rate after clumpify.sh is consistent with expectation for the library and identical optical/tile-edge duplicates are removed
- Downstream steps (BWA-MEM alignment, ViralCC) accept the cleaned FASTQ/BAM without read-length errors (min length 50 enforced by minlen=50)

## Limitations

- Version sensitivity: the paper used BBTools v37.25 — results (e.g. qc3C 3D ratios) may not reproduce exactly with other versions
- NCBI may update its SRA download links, so retrieving the raw FASTQ inputs for cleaning may fail and require the latest NCBI links
- ftl=10 trimming discards the first 10 nucleotides of every Hi-C read, which is specific to the proximity-ligation protocol and would degrade non-Hi-C data

## Evidence

- [methods] reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`: "reads are quality-trimmed using `bbduk` with parameters `trimq=10 qtrim=r ftm=5 minlen=50`"
- [methods] Hi-C first 10 nt trim: "the first 10 nucleotides of Hi-C reads are trimmed by `bbduk` with parameter `ftl=10`"
- [methods] duplicate removal with clumpify.sh: "Identical PCR optical and tile-edge duplicates for Hi-C reads were removed by the script `clumpify.sh` from `BBTools` suite."
- [methods] adapter removal with bbduk: "Adaptor sequences are removed by `bbduk` from the `BBTools` suite with parameter `ktrim=r k=23 mink=11 hdist=1 minlen=50 tpe tbo`"
- [readme] software versions in reproduction instructions: [evidence span withheld — anchor status: external_doc]
