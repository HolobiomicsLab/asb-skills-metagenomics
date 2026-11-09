---
name: multiqc-report-aggregation
description: Use when a metagenomics QC stage has produced per-sample FastQC HTML/zip
  reports for raw and trimmed FASTQ pairs and you need one aggregated overview to
  verify adapter removal and Q20 trimming.
when_to_use_negative:
- No per-sample QC reports exist yet — run FastQC on the raw and trimmed FASTQ files
  before aggregating
- Input is not FASTQ-based Illumina (meta)genomic data processed by the RiboTaxa QC
  stage, so before/after FastQC comparison is meaningless
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiQC
  role: Aggregates the per-file FastQC reports generated before and after bbduk.sh
    trimming into a single multiqc overview of all (meta)genomics files
  repo: https://github.com/MultiQC/MultiQC
- name: FastQC
  role: Produces the per-file .html quality reports (before_fastqc and after_fastqc)
    that MultiQC aggregates
  repo: http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- name: BBTools
  role: 'Upstream trimming step (bbduk.sh: ktrim=r, kmer=21, qtrim=rl, trimq=20, maxns=1,
    minlength=60) whose effect the aggregated FastQC reports verify'
  repo: https://jgi.doe.gov/data-and-tools/bbtools/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqac070
    title: 'RiboTaxa: combined approaches for rRNA genes taxonomic resolution down
      to the species level from metagenomics data revealing novelties'
  source_content_hash: sha256:69d6af5f395eab2af87c645158c4d2928fa1ed2a7f2305ac2202bf522ec83b00
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/oschakoory/RiboTaxa
  repo_url: https://github.com/MultiQC/MultiQC
  tools:
  - BBTools
  - FastQC
  - MultiQC
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# multiqc-report-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregate per-file FastQC quality reports (before and after bbduk.sh adapter/quality trimming) from the RiboTaxa quality-control stage into a single MultiQC HTML report, so read-quality improvements across all (meta)genomic FASTQ files can be reviewed at a glance.

## When to use

Use when a metagenomics QC stage has produced per-sample FastQC HTML/zip reports for raw and trimmed FASTQ pairs and you need one aggregated overview to verify adapter removal and Q20 trimming.

## When NOT to use

- No per-sample QC reports exist yet — run FastQC on the raw and trimmed FASTQ files before aggregating
- Input is not FASTQ-based Illumina (meta)genomic data processed by the RiboTaxa QC stage, so before/after FastQC comparison is meaningless

## Inputs

- FastQC HTML/zip reports for raw input FASTQ files (before_fastqc)
- FastQC HTML/zip reports for bbduk.sh-trimmed FASTQ files (after_fastqc)
- Trimmed FASTQ(.gz) pairs in the quality_control directory

## Outputs

- Single MultiQC aggregate report (HTML) summarizing all FastQC reports
- MultiQC parsed data (fastqc per-module metrics) underlying the report

## How to apply

In the RiboTaxa quality-control directory, run FastQC separately on the raw input FASTQ files (before_fastqc) and on the bbduk.sh-trimmed outputs (after_fastqc), where trimming used ktrim=r, kmer=21, qtrim=rl, trimq=20, maxns=1 and minlength=60. Then run MultiQC over the directory containing all per-file FastQC reports so it parses the fastqc_data.txt content and emits a single multiqc report covering every (meta)genomics file. Compare the before/after aggregates to confirm that adapter content, per-base quality and sequence-length distributions improved and that no read below 60 bp survived. Treat a missing or empty aggregate report as a QC-stage validation failure.

## Related tools

- **MultiQC** (Aggregates the per-file FastQC reports generated before and after bbduk.sh trimming into a single multiqc overview of all (meta)genomics files) — https://github.com/MultiQC/MultiQC
- **FastQC** (Produces the per-file .html quality reports (before_fastqc and after_fastqc) that MultiQC aggregates) — http://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- **BBTools** (Upstream trimming step (bbduk.sh: ktrim=r, kmer=21, qtrim=rl, trimq=20, maxns=1, minlength=60) whose effect the aggregated FastQC reports verify) — https://jgi.doe.gov/data-and-tools/bbtools/

## Examples

```
multiqc quality_control/ -o quality_control/multiqc
```

## Evaluation signals

- Trimmed FASTQ pair exists in the quality_control directory and FastQC/MultiQC reports were generated for both before_fastqc and after_fastqc
- Aggregated MultiQC report covers every input (meta)genomics file, not a subset
- Per-base quality and adapter-content modules in the after-trim aggregate show Q20 trimming and adapter removal took effect relative to the before aggregate
- No surviving reads shorter than 60 bp or with more than 1 N in the trimmed output, per bbduk parameters
- Reads with regions below Q20 were trimmed from both ends (qtrim=rl) as reflected in the FastQC quality distributions

## Limitations

- MultiQC only summarizes existing FastQC output; it does not itself trim or filter reads — trimming failures must be fixed at the bbduk.sh step
- The aggregate report is descriptive: detecting residual adapter or low-quality signal still requires manual inspection of the before/after modules
- This aggregation applies to the RiboTaxa QC stage for Illumina (meta)genomic FASTQ input; it does not cover later SSU extraction, assembly or classification QC

## Evidence

- [readme] - For quality control :FastQC, MultiQC: [evidence span withheld — anchor status: not_found]
