---
name: paired-end-read-quality-trimming
description: Use when raw paired-end FASTQ reads (e.g. from SRA accessions like ERR008613)
  must be quality-checked and low-quality ends removed before assembly, with surviving
  mate pairs kept as FASTQ.GZ.
when_to_use_negative:
- Input reads are single-end, so Trimmomatic paired-end (PE) mode with surviving mate
  pairs does not apply
- Reads have already been quality-trimmed and a valid FastQC report exists for them
- The goal is assembly quality evaluation rather than read preprocessing — use WebQUAST/QUAST
  on the assemblies instead
tools:
- name: SRA Toolkit (fasterq-dump)
  role: Downloads the raw paired-end reads for accession ERR008613 as FASTQ files
- name: FastQC
  role: Generates the read-quality report (fastqc_report.html) for the raw reads before
    trimming
- name: Trimmomatic
  role: Runs in paired-end (PE) mode to trim low-quality ends, writing surviving pairs
    to trimmed_1P.fastq.gz and trimmed_2P.fastq.gz and discarding orphaned mates
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkad406
    title: 'WebQUAST: online evaluation of genome assemblies'
  source_content_hash: sha256:fdb61631c9566f60bf505a9c3752cd625c6c389b68b6ab353b4488edc7841f9e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastQC
  - SRA Toolkit
  - SRA Toolkit (fasterq-dump)
  - Trimmomatic
  - WebQUAST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# paired-end-read-quality-trimming

## Summary

Quality-controls raw paired-end short reads with FastQC and trims low-quality read ends with Trimmomatic before genome assembly, as done for the E. coli K-12 MG1655 SRA dataset ERR008613 used to prepare sample data for WebQUAST.

## When to use

Use when raw paired-end FASTQ reads (e.g. from SRA accessions like ERR008613) must be quality-checked and low-quality ends removed before assembly, with surviving mate pairs kept as FASTQ.GZ.

## When NOT to use

- Input reads are single-end, so Trimmomatic paired-end (PE) mode with surviving mate pairs does not apply
- Reads have already been quality-trimmed and a valid FastQC report exists for them
- The goal is assembly quality evaluation rather than read preprocessing — use WebQUAST/QUAST on the assemblies instead

## Inputs

- SRA accession (e.g. ERR008613) downloadable via fasterq-dump (SRA Toolkit)
- Raw paired-end FASTQ read files (mate 1 and mate 2)

## Outputs

- trimmed_1P.fastq.gz — surviving forward-mate reads after trimming
- trimmed_2P.fastq.gz — surviving reverse-mate reads after trimming
- fastqc_report.html — FastQC read-quality report for the raw reads

## How to apply

Download the reads for the SRA accession (here ERR008613, E. coli K-12 MG1655) with fasterq-dump from the SRA Toolkit, keeping them as FASTQ files. Run FastQC on the raw reads to generate the read-quality report fastqc_report.html and inspect per-base quality before trimming. Run Trimmomatic in paired-end (PE) mode so both mates are trimmed together: surviving pairs are written to trimmed_1P.fastq.gz and trimmed_2P.fastq.gz while orphaned mates whose partner was discarded are dropped rather than written to singletons. The paired-end mode preserves mate synchronization, which matters because assemblies and downstream QUAST-style read mapping assume coherent pairs. Validate by confirming all three output files exist and that the two trimmed FASTQ.GZ files decompress to valid FASTQ with equal numbers of reads.

## Related tools

- **SRA Toolkit (fasterq-dump)** (Downloads the raw paired-end reads for accession ERR008613 as FASTQ files)
- **FastQC** (Generates the read-quality report (fastqc_report.html) for the raw reads before trimming)
- **Trimmomatic** (Runs in paired-end (PE) mode to trim low-quality ends, writing surviving pairs to trimmed_1P.fastq.gz and trimmed_2P.fastq.gz and discarding orphaned mates)

## Examples

```
trimmomatic PE reads_1.fastq.gz reads_2.fastq.gz trimmed_1P.fastq.gz /dev/null trimmed_2P.fastq.gz /dev/null
```

## Evaluation signals

- trimmed_1P.fastq.gz, trimmed_2P.fastq.gz and fastqc_report.html all exist in the working directory
- Both trimmed files decompress to valid FASTQ
- The two trimmed FASTQ files contain equal numbers of reads (mate pairs remain synchronized)
- The FastQC run completed without errors and produced the HTML quality report
- The Trimmomatic run completed without errors on both mates

## Limitations

- Orphaned mates (whose partner failed trimming thresholds) are discarded, so total read counts can drop below the raw input
- The skill covers read preprocessing only; assembly and its evaluation (e.g. of the four E. coli K-12 MG1655 assemblies via WebQUAST) are separate steps
- QUAST itself is more informative with a reference: the toolkit 'works both with and without reference genomes. However, it is much more informative if at least a close reference genome is provided'

## Evidence

- [supplementary] Supplementary dataset provenance for the sample reads: "The sequencing data for E. coli K-12 MG1655 dataset is available from the National Center for Biotechnology Information (NCBI) Sequence Read Archive under accession number ERR008613"
- [other] Overall preprocessing finding: [evidence span withheld — anchor status: not_found]
- [other] Trimming step and output files: [evidence span withheld — anchor status: not_found]
- [other] Validation criteria for the trimming step: [evidence span withheld — anchor status: not_found]
