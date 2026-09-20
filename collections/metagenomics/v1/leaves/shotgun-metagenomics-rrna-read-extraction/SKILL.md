---
name: shotgun-metagenomics-rrna-read-extraction
description: Use when input is raw paired-end or single-end Illumina shotgun metagenomics
  FASTQ (fastq.gz) and the goal is to isolate 16S/18S rRNA fragments prior to reference-targeted
  SSU assembly or taxonomic profiling.
when_to_use_negative:
- Input is already amplicon 16S FASTQ (e.g. from a 16S PCR workflow) where no shotgun
  rRNA extraction step is needed
- The goal is direct taxonomic classification of short reads with fast classifiers
  (Kraken2, Centrifuge) rather than reconstruction of full-length SSU sequences
- The data contain no rRNA-derived reads (e.g. mRNA-only meta-transcriptomic libraries
  depleted of rRNA)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: SortMeRNA
  role: rRNA-specialized tool used to filter 16S/18S reads from trimmed metagenomic
    reads against an indexed SILVA SSU 138.1 database; extracted reads assembled better
    than BBMap-extracted reads
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBMap
  role: General-purpose read mapper used as the alternative extractor of SSU reads
    in the extractor benchmark
  repo: https://github.com/oschakoory/RiboTaxa
- name: BBTools
  role: Upstream adapter removal and quality trimming (ktrim=r, kmer=21, minlength=60,
    trimq=20, qtrim=rl, maxns=1) before SSU extraction
- name: FastQC
  role: Per-read quality control of raw and trimmed metagenomic FASTQ within the RiboTaxa
    pipeline
  repo: https://github.com/oschakoory/RiboTaxa
- name: RiboTaxa
  role: Pipeline that orchestrates trimming (BBTOOLS), 16S/18S filtering (SortMeRNA),
    SSU reconstruction and classification; provides indexed SILVA SSU 138.1 databases
    and setup scripts
  repo: https://github.com/oschakoory/RiboTaxa
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:a58dd0c8c78a2597df912ce3653d8544d218a356625af5ce5e7cd145406571c5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/oschakoory/RiboTaxa
  tools:
  - BBMap
  - BBTools
  - FastQC
  - RiboTaxa
  - SortMeRNA
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# shotgun-metagenomics-rrna-read-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract small-subunit (16S/18S) rRNA reads from shotgun (meta)genomic Illumina FASTQ data using an rRNA-specialized mapper (SortMeRNA) and/or a general-purpose read mapper (BBMap) against the indexed SILVA SSU 138.1 database, producing the filtered read sets needed for full-length SSU reconstruction.

## When to use

Use when input is raw paired-end or single-end Illumina shotgun metagenomics FASTQ (fastq.gz) and the goal is to isolate 16S/18S rRNA fragments prior to reference-targeted SSU assembly or taxonomic profiling.

## When NOT to use

- Input is already amplicon 16S FASTQ (e.g. from a 16S PCR workflow) where no shotgun rRNA extraction step is needed
- The goal is direct taxonomic classification of short reads with fast classifiers (Kraken2, Centrifuge) rather than reconstruction of full-length SSU sequences
- The data contain no rRNA-derived reads (e.g. mRNA-only meta-transcriptomic libraries depleted of rRNA)

## Inputs

- Paired-end or single-end Illumina shotgun metagenomic reads (FASTQ or fastq.gz)
- BBTools/QC-trimmed FASTQ files (adapters removed, minlength=60, trimq=20, maxns=1)
- Indexed SILVA SSU 138.1 reference database for SortMeRNA/BBMap

## Outputs

- Filtered 16S/18S rRNA read FASTQ files from the trimmed (meta)genomic sequence files (SortMeRNA output)
- BBMap-extracted SSU read FASTQ files for extractor comparison
- Read-extraction statistics suitable for tabulating extractor/assembler combinations

## How to apply

First trim adapters and low-quality bases with BBTools (adapter trimming with ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1), optionally checking quality with FastQC/MultiQC. Then extract SSU reads against an indexed SILVA SSU 138.1 database: run SortMeRNA (the rRNA-specialized tool, which in the RiboTaxa benchmark extracted reads that assembled better with EMIRGE, MATAM and MetaRib) and/or BBMap (the general-purpose mapper) to yield filtered 16S/18S FASTQ files. The benchmark design compares both extractors so each can feed separate reference-targeted assemblies (Emirge.py, Emirge_amplicon.py, MATAM, MetaRib); in the mock community study the SortMeRNA-filtered reads assembled better than BBMap-extracted reads. Validate extraction by confirming the filtered folder contains 16S/18S sequences from the trimmed input and that downstream assembly recovers expected full-length variants (e.g. the 41 known mock 16S rRNA variants).

## Related tools

- **SortMeRNA** (rRNA-specialized tool used to filter 16S/18S reads from trimmed metagenomic reads against an indexed SILVA SSU 138.1 database; extracted reads assembled better than BBMap-extracted reads) — https://github.com/oschakoory/RiboTaxa
- **BBMap** (General-purpose read mapper used as the alternative extractor of SSU reads in the extractor benchmark) — https://github.com/oschakoory/RiboTaxa
- **BBTools** (Upstream adapter removal and quality trimming (ktrim=r, kmer=21, minlength=60, trimq=20, qtrim=rl, maxns=1) before SSU extraction)
- **FastQC** (Per-read quality control of raw and trimmed metagenomic FASTQ within the RiboTaxa pipeline) — https://github.com/oschakoory/RiboTaxa
- **RiboTaxa** (Pipeline that orchestrates trimming (BBTOOLS), 16S/18S filtering (SortMeRNA), SSU reconstruction and classification; provides indexed SILVA SSU 138.1 databases and setup scripts) — https://github.com/oschakoory/RiboTaxa

## Examples

```
git clone https://github.com/oschakoory/RiboTaxa.git && cd RiboTaxa && bash conda_virt_env.sh
```

## Evaluation signals

- The filtered output folder contains 16S/18S sequences derived from the trimmed (not raw) input files
- SortMeRNA-extracted reads assemble better than BBMap-extracted reads across EMIRGE (Emirge.py, Emirge_amplicon.py), MATAM and MetaRib, reproducing the paper's qualitative outcome
- Downstream reconstruction on the extracted reads recovers the expected full-length variants (e.g. 41 known mock community 16S rRNA variants from SRA SRR5381736)
- Trimming statistics confirm reads shorter than minlength=60 were discarded and bases with quality below trimq=20 (qtrim=rl) were removed
- Extraction was run against the SILVA SSU 138.1 (indexed) database, not an ad hoc or unindexed reference

## Limitations

- Chimera detection was not included in the 16S rRNA gene reconstruction workflow, and chimeric database sequences could carry over into SSU reconstructions if reads map across the full length of a chimera
- RiboTaxa (which wraps this extraction step) is slower than short-read classifiers such as Kraken2 and Centrifuge
- Extracted reads alone give no species-level answer: inferring taxa directly from short-length Illumina reads resulted in a high rate of false positives, whereas reconstructed sequences were correctly assigned

## Evidence

- [methods] Quality control with BBTOOLS removes adapters and short/low-quality reads.: [evidence span withheld — anchor status: not_found]
