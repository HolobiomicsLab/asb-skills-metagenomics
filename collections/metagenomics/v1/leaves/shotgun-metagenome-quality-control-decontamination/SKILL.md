---
name: shotgun-metagenome-quality-control-decontamination
description: Use when raw paired-end shotgun metagenome FASTQ files (e.g. from NCBI
  SRA human gut, infant, or patient cohorts) must be cleaned of low-quality bases
  and host/contaminant reads before fungal taxonomic or functional profiling.
when_to_use_negative:
- Input is already a QC'd feature/abundance table rather than raw FASTQ reads
- The goal is fungal functional profiling with FunOMIC-P, which requires DIAMOND blastx
  thresholds (coverage >95%, identity >99%, e-value <10e-10) rather than read-level
  QC and Bowtie2 mapping
- Input is amplicon (16S/ITS) data rather than shotgun metagenomic reads
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: KneadData
  role: Quality control and decontamination of raw shotgun metagenomic FASTQ reads
    before mapping
- name: FunOMIC
  role: Main pipeline (FunOMIC.sh) orchestrating read merging, mapping to FunOMIC-T,
    and abundance aggregation
  repo: https://github.com/ManichanhLab/FunOMIC
- name: FLASH2
  role: Merging overlapping paired-end reads within the FunOMIC pipeline
  repo: https://github.com/dstreett/FLASH2
- name: Bowtie2
  role: Mapping cleaned reads to the FunOMIC-T fungal single-copy marker-gene database
    (v2.3.4.3)
  repo: http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: Samtools
  role: Filtering alignments, retaining reads with >80% aligned length and q-score
    >30 (v1.9)
  repo: http://www.htslib.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.07.010
    title: 'FunOMIC: Pipeline with built-in fungal taxonomic and functional databases
      for human mycobiome profiling'
  source_content_hash: sha256:650591be1233affc477d7c9e390904b5294af3b9af6661ed5e7d1ebb1b5e6ef4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ManichanhLab/FunOMIC
  repo_url: https://github.com/ManichanhLab/FunOMIC
  tools:
  - Bowtie2
  - FLASH2
  - FunOMIC
  - KneadData
  - Samtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# shotgun-metagenome-quality-control-decontamination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality-control and decontamination of human shotgun metagenomic FASTQ reads (e.g. the 2,679 public human metagenomes processed with KneadData v0.7.7-alpha) so that only cleaned reads are mapped against the FunOMIC-T fungal marker-gene database for taxonomic profiling.

## When to use

Use when raw paired-end shotgun metagenome FASTQ files (e.g. from NCBI SRA human gut, infant, or patient cohorts) must be cleaned of low-quality bases and host/contaminant reads before fungal taxonomic or functional profiling.

## When NOT to use

- Input is already a QC'd feature/abundance table rather than raw FASTQ reads
- The goal is fungal functional profiling with FunOMIC-P, which requires DIAMOND blastx thresholds (coverage >95%, identity >99%, e-value <10e-10) rather than read-level QC and Bowtie2 mapping
- Input is amplicon (16S/ITS) data rather than shotgun metagenomic reads

## Inputs

- Paired-end shotgun metagenomic FASTQ files from NCBI SRA (e.g. 2,679 public human metagenome samples)
- FunOMIC-T taxonomic database (1.69 million fungal single-copy marker genes from https://manichanh.vhir.org/funomic/)
- Bacterial reference database path supplied via the FunOMIC.sh -a flag for decontamination/subtraction

## Outputs

- Cleaned (QC'd, decontaminated) paired-end FASTQ files per sample
- Filtered SAM/BAM alignments of reads to FunOMIC-T marker genes
- Per-sample fungal species abundance/prevalence table produced by the in-house FunOMIC script

## How to apply

Download the raw shotgun datasets from NCBI SRA and run each sample through KneadData v0.7.7-alpha to trim low-quality bases and remove host/contaminant reads, yielding cleaned FASTQ pairs per sample. Feed the cleaned pairs into the FunOMIC pipeline's main program (FunOMIC.sh), which merges overlapping pairs with FLASH2 and maps reads with Bowtie2 v2.3.4.3 to the FunOMIC-T database of 1.69 million fungal single-copy marker genes (with a bacterial database supplied via -a for host/bacterial subtraction). Filter alignments with Samtools v1.9, keeping only mapped reads with more than 80% of read length aligned and a q-score above 30. The rationale is that strict read-level QC plus the >80% alignment-length and q>30 filters are what enable the reported specificity (0.9999 in the bacterial mock community with ~1000:1 bacterial:fungal read ratio) and the low 4.72E-05 mean mapping rate for fungal detection.

## Related tools

- **KneadData** (Quality control and decontamination of raw shotgun metagenomic FASTQ reads before mapping)
- **FunOMIC** (Main pipeline (FunOMIC.sh) orchestrating read merging, mapping to FunOMIC-T, and abundance aggregation) — https://github.com/ManichanhLab/FunOMIC
- **FLASH2** (Merging overlapping paired-end reads within the FunOMIC pipeline) — https://github.com/dstreett/FLASH2
- **Bowtie2** (Mapping cleaned reads to the FunOMIC-T fungal single-copy marker-gene database (v2.3.4.3)) — http://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **Samtools** (Filtering alignments, retaining reads with >80% aligned length and q-score >30 (v1.9)) — http://www.htslib.org/

## Examples

```
FunOMIC.sh -1 read1.fastq -2 read2.fastq -p out -o results/ -a bacterialDB_path -b FunOMIC-T_folder -c FunOMIC-P_folder -t 8
```

## Evaluation signals

- Reproduced fungal detection rate on the 2,679 samples matches the reported 72.9% of samples with detectable fungal DNA
- Mean FunOMIC-T mapping rate matches the reported 4.72E-05
- Per-species prevalence is consistent with reported patterns, e.g. Saccharomyces cerevisiae present in 52.4% of samples
- Filtered alignments retain only reads with >80% of read length aligned and q-score above 30
- Bacterial mock-community check: none of 999,021 simulated bacterial reads align to FunOMIC-T (specificity 0.9999)

## Limitations

- Extraction and quality of single-copy marker genes rely on the completeness of available fungal genomes, which may result in lower coverage of fungal taxonomies compared with amplicon databases
- High inter-kingdom conservation of protein-coding genes means bacterial contamination was not totally preventable, particularly for functional annotation
- No bacterial read filtration step is included before functional annotation in the current tool; it is only proposed for a future update

## Evidence
