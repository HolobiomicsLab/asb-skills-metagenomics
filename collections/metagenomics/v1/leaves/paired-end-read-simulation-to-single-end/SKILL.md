---
name: paired-end-read-simulation-to-single-end
description: Use when deciding whether paired-end sequencing is necessary for metatranscriptome
  annotation, or when quantifying how much annotation information is lost by processing
  paired-end reads as single-end.
when_to_use_negative:
- Input reads are already single-end — no simulation is possible or meaningful.
- Annotation is done locally with SAMSA2/DIAMOND against a custom database rather
  than via MG-RAST, since the comparison workflow is tied to MG-RAST v3.6 submission
  and its queue.
- The goal is organism abundance estimation from already-annotated summary tables
  rather than a raw-read preprocessing comparison.
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: Trimmomatic
  role: Trims each paired-end read to 100 bp to create simulated single-end files
    and removes adaptor contamination during preprocessing
  repo: http://www.usadellab.org/cms/?page=trimmomatic
- name: FLASH
  role: Joins paired-end reads in the paired-end preprocessing branch of SAMSA
- name: MG-RAST
  role: Annotation server (v3.6) producing organism-level annotations for both paired-end
    and simulated single-end read sets
- name: SAMSA
  role: Pipeline scripts (analysis_counter, RefSeq_output_reducer.py) that aggregate
    MG-RAST results into comparable organism summary files
  repo: github.com/transcript/SAMSA
- name: DESeq2
  role: R package used in R_organism_script.Rmd to correlate organism-level activity
    between paired-end and single-end results
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1270-8
    title: 'SAMSA: a comprehensive metatranscriptome analysis pipeline'
  source_content_hash: sha256:ccbf8398af63e0726196a0ed422ee58935c8ecf828ba2f518a59b4921e0eb06f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - FLASH
  - MG-RAST
  - R
  - SAMSA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# paired-end-read-simulation-to-single-end

## Summary

Simulates single-end metatranscriptome data from paired-end FASTQ files by truncating each read to 100 bp with Trimmomatic, enabling a controlled comparison of annotation loss between paired-end and single-end processing in the SAMSA/MG-RAST pipeline.

## When to use

Use when deciding whether paired-end sequencing is necessary for metatranscriptome annotation, or when quantifying how much annotation information is lost by processing paired-end reads as single-end.

## When NOT to use

- Input reads are already single-end — no simulation is possible or meaningful.
- Annotation is done locally with SAMSA2/DIAMOND against a custom database rather than via MG-RAST, since the comparison workflow is tied to MG-RAST v3.6 submission and its queue.
- The goal is organism abundance estimation from already-annotated summary tables rather than a raw-read preprocessing comparison.

## Inputs

- raw paired-end metatranscriptome FASTQ files (pilot fecal gut microbiome samples)
- adaptor contamination FASTA file for Trimmomatic
- MG-RAST account and annotation server access (v3.6)

## Outputs

- simulated single-end FASTQ files (each read trimmed to 100 bp)
- preprocessed paired-end and single-end FASTQ sets for MG-RAST submission
- organism summary files from analysis_counter and RefSeq_output_reducer.py
- table_2_pe_se_comparison.csv with QC removal counts, reference match percentages, unique transcript counts, and DESeq2 correlation of organism activity

## How to apply

Load raw paired-end metatranscriptome FASTQ files and use Trimmomatic to trim each read to a fixed 100 bp, producing simulated single-end versions of every sample. Run both the original paired-end set (with adaptor contamination removal by Trimmomatic and paired-end joining by FLASH) and the simulated single-end set through SAMSA preprocessing, then submit both to MG-RAST (v3.6) for organism-level annotation. Download annotation results and aggregate them into organism summary files with analysis_counter and RefSeq_output_reducer.py. Finally, compute comparison metrics — reads removed by QC, percentage of reads matching the reference database, unique transcripts identified — and correlate organism-level activity summaries between versions in R with DESeq2 to quantify how much paired-end joining improves annotation. Sequences shorter than 100 bp or unjoined paired-end reads fail annotation, which is the rationale for testing the 100 bp paired-end format.

## Related tools

- **Trimmomatic** (Trims each paired-end read to 100 bp to create simulated single-end files and removes adaptor contamination during preprocessing) — http://www.usadellab.org/cms/?page=trimmomatic
- **FLASH** (Joins paired-end reads in the paired-end preprocessing branch of SAMSA)
- **MG-RAST** (Annotation server (v3.6) producing organism-level annotations for both paired-end and simulated single-end read sets)
- **SAMSA** (Pipeline scripts (analysis_counter, RefSeq_output_reducer.py) that aggregate MG-RAST results into comparable organism summary files) — github.com/transcript/SAMSA
- **DESeq2** (R package used in R_organism_script.Rmd to correlate organism-level activity between paired-end and single-end results)

## Examples

```
python SAMSA_pre_annotation_pipeline.py -input sample_R1.fastq -input2 sample_R2.fastq -trimmomatic /path/trimmomatic.jar -c PE ; java -jar trimmomatic.jar SE -phred33 sample.fastq sample_trimmed_100bp.fastq CROP:100
```

## Evaluation signals

- Reads removed by QC: expected ~51,507 (0.3%) for paired-end vs ~123,518 (0.6%) for simulated single-end — single-end should show roughly double the QC losses.
- Reference database match percentage: expected 77.2% for paired-end vs 65.9% for single-end.
- Unique transcripts identified: expected ~270,384 paired-end vs ~215,599 single-end.
- Average correlation of organism-level activity summaries between paired-end and single-end results should be approximately 0.72.
- All simulated single-end reads must be exactly 100 bp after Trimmomatic trimming; shorter sequences would fail the annotation-length requirement.

## Limitations

- Sequences must be either longer than 100 bp or joined paired-end reads to be annotated, so the simulated single-end 100 bp reads sit at the minimum length and lose joining-derived extension.
- MG-RAST annotation servers require queue waits of days to weeks, so the paired comparison may be slow to complete.
- Findings are based on one pilot fecal metatranscriptome and MG-RAST v3.6; MG-RAST is not tailored to eukaryotic organisms, which may affect mismatch rates.
- Microbial population size shifts cannot be measured as accurately as with shotgun metagenomics regardless of read format.

## Evidence

- [results] Paired-end reads outperform single-end in annotation: [evidence span withheld — anchor status: not_found]
- [abstract] Read length requirement: "Sequences need to be either long reads (longer than 100 bp) or joined paired-end reads."
- [abstract] Sequencing format recommendation: "We further recommend that stool metatranscriptomes be ribodepleted and sequenced in a 100 bp paired end format with a minimum of 40 million reads per sample."
- [methods] Preprocessing steps in SAMSA: "it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified)"
- [methods] Analysis scripts used for aggregation and statistics: "R_organism_script.Rmd - imports organism summary files and performs statistical significance correlations"
- [readme] Pipeline flow from README: "Starting with a raw sequenced file in fastq format, it performs cleaning and removal of adaptor contamination, and then aligns paired-end files (if specified). The files must then be submitted to"
