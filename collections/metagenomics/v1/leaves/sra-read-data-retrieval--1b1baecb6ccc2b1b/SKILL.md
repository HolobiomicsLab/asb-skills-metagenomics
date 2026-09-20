---
name: sra-read-data-retrieval
description: Use when a task requires reproducing the MTBC_B (n=1,609) Mykrobe resistance-prediction
  validation and the raw isolate FASTQ reads must first be downloaded from ENA/NCBI
  SRA using accession codes in Supplementary Data.
when_to_use_negative:
- The sequencing reads are already available locally as FASTQ — proceed directly to
  running Mykrobe predictor rather than re-downloading.
- No accession codes or phenotyping tables are available (Supplementary Data 1–10
  not accessible), so isolates cannot be traced to SRA entries.
- The task targets S. aureus or other species validation sets (St_A1, St_B1) whose
  accessions and QC thresholds differ from the MTBC workflow.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Mykrobe predictor
  role: Downstream consumer of the retrieved SRA reads; reads are downloaded so that
    `mykrobe predict --species tb` can be run and validated against phenotypic DST
    labels
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe parser
  role: Parses the JSON output produced from the downloaded reads; applies QC by removing
    samples Mykrobe detects as non-tuberculous mycobacteria
  repo: https://github.com/phac-nml/mykrobe-parser
- name: European Nucleotide Archive / NCBI SRA
  role: Source repository from which the MTBC isolate sequencing data are downloaded
    using Supplementary Data accessions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:971a1755623d4439f4a81f04061b1ef3cc3ab87ca5b36e46abb2c26215ddc236
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - European Nucleotide Archive / NCBI SRA
  - Mykrobe parser
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# sra-read-data-retrieval

## Summary

Retrieves raw MTBC clinical-isolate sequencing reads from ENA/NCBI SRA using accession codes listed in the paper's Supplementary Data, paired with per-isolate phenotypic DST resistance labels, to enable reproduction of Mykrobe predictor validation results (82.6%/98.5% sensitivity/specificity on n=1,609 MTBC_B samples).

## When to use

Use when a task requires reproducing the MTBC_B (n=1,609) Mykrobe resistance-prediction validation and the raw isolate FASTQ reads must first be downloaded from ENA/NCBI SRA using accession codes in Supplementary Data.

## When NOT to use

- The sequencing reads are already available locally as FASTQ — proceed directly to running Mykrobe predictor rather than re-downloading.
- No accession codes or phenotyping tables are available (Supplementary Data 1–10 not accessible), so isolates cannot be traced to SRA entries.
- The task targets S. aureus or other species validation sets (St_A1, St_B1) whose accessions and QC thresholds differ from the MTBC workflow.

## Inputs

- ENA/NCBI SRA accession codes for MTBC clinical isolates, listed with phenotyping data in Supplementary Data 1–10
- Phenotypic DST resistance labels per isolate and drug (from the same supplementary tables)
- Reference MTBC panel definitions used by Mykrobe predictor

## Outputs

- Per-isolate FASTQ read files (e.g. reads.fq.gz) for the 1,609 MTBC_B validation isolates
- Tabulated per-isolate, per-drug phenotypic DST labels keyed to the same accessions
- Sample inventory recording which accessions were retrieved and which were later QC-excluded (contaminated or non-MTBC)

## How to apply

Locate the accession codes together with the phenotyping data in the article's Supplementary Data 1–10, filtering for the MTBC_B validation isolates. Download the paired-end reads from the European Nucleotide Archive or NCBI SRA for each accession and assemble a per-isolate, per-drug table of phenotypic DST resistance labels alongside the FASTQ files. Note that the original training/validation sets (e.g. MTBC_A1 with 338 isolates, Myco_SRA with 380 SRA samples) are also SRA-deposited and can be retrieved the same way if extending the analysis. After download, prepare the data for Mykrobe predictor by keeping HiSeq 2000-derived reads (mean read length 87 bp after cutting at bases with quality below Q10) and plan to apply QC downstream: removal of samples detected as non-tuberculous mycobacteria and of samples with >40 confident heterozygous SNPs (contaminated by an unrelated strain; 20 such samples were removed from the validation set).

## Related tools

- **Mykrobe predictor** (Downstream consumer of the retrieved SRA reads; reads are downloaded so that `mykrobe predict --species tb` can be run and validated against phenotypic DST labels) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe parser** (Parses the JSON output produced from the downloaded reads; applies QC by removing samples Mykrobe detects as non-tuberculous mycobacteria) — https://github.com/phac-nml/mykrobe-parser
- **European Nucleotide Archive / NCBI SRA** (Source repository from which the MTBC isolate sequencing data are downloaded using Supplementary Data accessions)

## Evaluation signals

- Retrieved sample count matches the validation set size (n=1,609 for MTBC_B, or n=338 for MTBC_A1 training set).
- Each downloaded accession pairs with a phenotypic DST label from Supplementary Data 1–10, allowing per-drug confusion matrices to be built.
- After downstream QC (non-MTBC removal plus the 40-heterozygous-SNP contamination threshold, ~20 validation samples removed), reproducing Mykrobe on the retained reads yields the reported 82.6% sensitivity and 98.5% specificity.
- Per-drug sensitivities from the retrieved data match rifampicin 93.7%, isoniazid 84.3%, ethambutol 71.6%.
- All read files are valid paired FASTQ compatible with the HiSeq 2000 characteristics described (mean read length 87 bp).

## Limitations

- Accession codes and phenotyping data are only available in the supplementary materials (Supplementary Data 1–10); without them the isolates cannot be retrieved or labelled.
- Some validation samples are contaminated (20 removed from validation, 6 from training) and will fail the 40-heterozygous-SNP QC threshold, so retrieved reads do not all contribute to the reported metrics.
- False-negative rates were only estimated for drugs with >10 resistant samples in the validation set, limiting which per-drug metrics the retrieved data can support.

## Evidence

- [discussion] All the sequencing data is available at the European Nucleotide Archive and National Center for Biotechnology Information's SRA: "All the sequencing data is available at the European Nucleotide Archive and National Center for Biotechnology Information's SRA"
- [discussion] Accession codes with phenotyping data in supplementary materials: "Accession codes are listed, together with phenotyping data, in Supplementary Data 1–10."
- [readme] QC removal of non-tuberculous mycobacteria samples: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
- [discussion] Removal of contaminated samples from training and validation sets: "We removed 6 samples from the training data set and 20 samples from the validation set that were contaminated"
