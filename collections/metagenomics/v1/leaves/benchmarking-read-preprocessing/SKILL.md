---
name: benchmarking-read-preprocessing
description: Use when raw paired-end Illumina FASTQ pairs (e.g. Acinetobacter_baumannii_1.fastq.gz/_2.fastq.gz
  from Zenodo 7996690) must be adapter-trimmed and quality-filtered before hybrid
  plasmid assembly with Plassembler.
when_to_use_negative:
- Input reads are long reads (Nanopore/PacBio) — fastp is designed for short reads;
  the Plassembler QC stage targets the paired-end short-read input set only
- The Plassembler run was launched with --skip_qc, so the short-read FASTQs should
  be passed through without fastp trimming
- Reads have already been trimmed/filtered — re-running fastp would only shorten reads
  without benefit
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: 'Performs the paired-end QC: default quality filtering, automatic adapter
    trimming, sliding-window per-read quality cutting, gzip output'
  repo: https://github.com/OpenGene/fastp
- name: Plassembler
  role: Pipeline that invokes the fastp QC stage on its short-read paired-end inputs
    before mapping and hybrid assembly; QC can be skipped with --skip_qc
  repo: https://github.com/gbouras13/plassembler
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:6002e9242c3da33e0f2871cd13f77366a43a009b9a4db7f7b203e4e7e41ca5a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OpenGene/fastp
  tools:
  - Plassembler
  - fastp
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# benchmarking-read-preprocessing

## Summary

Quality-controls paired-end Illumina short-read FASTQ inputs with fastp (>=0.24.2) as the short-read QC stage of the Plassembler pipeline/benchmarking workflow, producing cleaned, gzip-compressed R1/R2 reads for downstream hybrid assembly.

## When to use

Use when raw paired-end Illumina FASTQ pairs (e.g. Acinetobacter_baumannii_1.fastq.gz/_2.fastq.gz from Zenodo 7996690) must be adapter-trimmed and quality-filtered before hybrid plasmid assembly with Plassembler.

## When NOT to use

- Input reads are long reads (Nanopore/PacBio) — fastp is designed for short reads; the Plassembler QC stage targets the paired-end short-read input set only
- The Plassembler run was launched with --skip_qc, so the short-read FASTQs should be passed through without fastp trimming
- Reads have already been trimmed/filtered — re-running fastp would only shorten reads without benefit

## Inputs

- Paired-end Illumina FASTQ R1 file (gzip-compressed, e.g. Acinetobacter_baumannii_1.fastq.gz)
- Paired-end Illumina FASTQ R2 file (gzip-compressed, e.g. Acinetobacter_baumannii_2.fastq.gz)

## Outputs

- cleaned_1.fastq.gz (filtered/adapter-trimmed R1 reads)
- cleaned_2.fastq.gz (filtered/adapter-trimmed R2 reads)
- fastp JSON/HTML QC report of before/after filtering statistics

## How to apply

Run fastp in paired-end mode on the R1/R2 FASTQ pair, writing gzip-compressed cleaned outputs (cleaned_1.fastq.gz, cleaned_2.fastq.gz). Apply fastp's default quality filtering, automatic adapter detection/trimming, and per-read/per-base quality trimming (sliding-window mean-quality cutting at 5'/3' ends). Within Plassembler this QC stage is part of the pipeline and can be skipped via the --skip_qc parameter, so decide upfront whether QC is required for the run. Use fastp >=0.24.2, the version pinned by the Plassembler paper's methods. The cleaned pair is then consumed by the downstream Plassembler steps (minimap2 mapping, Unicycler hybrid assembly).

## Related tools

- **fastp** (Performs the paired-end QC: default quality filtering, automatic adapter trimming, sliding-window per-read quality cutting, gzip output) — https://github.com/OpenGene/fastp
- **Plassembler** (Pipeline that invokes the fastp QC stage on its short-read paired-end inputs before mapping and hybrid assembly; QC can be skipped with --skip_qc) — https://github.com/gbouras13/plassembler

## Examples

```
fastp -i Acinetobacter_baumannii_1.fastq.gz -I Acinetobacter_baumannii_2.fastq.gz -o cleaned_1.fastq.gz -O cleaned_2.fastq.gz
```

## Evaluation signals

- Both cleaned_1.fastq.gz and cleaned_2.fastq.gz exist and are valid gzip FASTQ files
- R1 and R2 cleaned outputs contain matching numbers of reads (paired-end sync preserved)
- fastp report shows adapter content reduced relative to before-filtering statistics
- Cleaned reads are accepted by the downstream Plassembler steps (minimap2 mapping and Unicycler assembly complete without FASTQ parse errors)
- fastp version satisfies the pinned >=0.24.2 requirement from the Plassembler methods

## Limitations

- fastp is designed for short reads (Illumina/MGI); long-read (Nanopore/PacBio) preprocessing requires different tools such as chopper
- The QC stage is optional in Plassembler (--skip_qc), so runs compared with QC skipped are not directly comparable to trimmed runs
- fastp relies on automatic adapter detection unless adapters are supplied, which may miss non-standard adapters

## Evidence

- [methods] fastp version pinned in Plassembler methods: "* [fastp](https://github.com/OpenGene/fastp) >=0.24.2"
- [readme] fastp tool description: [evidence span withheld — anchor status: external_doc]
- [readme] fastp filtering behaviour: [evidence span withheld — anchor status: external_doc]
- [readme] fastp sliding-window trimming: [evidence span withheld — anchor status: external_doc]
