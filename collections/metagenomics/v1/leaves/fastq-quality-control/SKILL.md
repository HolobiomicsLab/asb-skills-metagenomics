---
name: fastq-quality-control
description: 'Use when raw paired-end RNA-seq/metatranscriptome FASTQ files must be
  cleaned before annotation: mates need merging, low-quality bases and adapters removed,
  and rRNA reads filtered to speed up annotation.'
when_to_use_negative:
- Input reads are single-end only — the standard SAMSA2 preprocessing begins with
  PEAR merging of paired-end mates (use the master_script_preserving_unmerged variant
  if unmerged reads must be retained)
- Reads are already trimmed and rRNA-filtered and only annotation (DIAMOND), aggregation,
  or DESeq analysis is needed
- The goal is taxonomic identification from 16S rRNA sequencing rather than mRNA functional
  profiling from metatranscriptomes
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: PEAR
  role: Merges two paired-end FASTQ sequencing files together to make a single file
    of extended fragments (used for paired-end data, recommended in the SAMSA2 dependency
    list)
  repo: https://sco.h-its.org/exelixis/web/software/pear/
- name: Trimmomatic
  role: Removes low-quality sequences and adapter contamination from the merged reads,
    using a specific FASTA file of adapter sequences
  repo: http://www.usadellab.org/cms/?page=trimmomatic
- name: SortMeRNA
  role: Removes ribosomal RNA reads from the cleaned FASTQ, since rRNA reads do not
    contribute to the mRNA profile of the metatranscriptome
  repo: http://bioinfo.lifl.fr/RNA/sortmerna/
- name: SAMSA2
  role: Master script that orchestrates the three preprocessing programs with checkpointing,
    multithreading, and disk-usage cleanup
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:a41889780bee695f1a5ba3d7a435a6de9a23ea90b4c6fba876c32c8b30ea004d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PEAR
  - SAMSA2
  - SortMeRNA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fastq-quality-control

## Summary

SAMSA2's preprocessing stage that converts raw paired-end metatranscriptome FASTQ reads into a cleaned, ribosome-filtered FASTQ ready for DIAMOND annotation, by merging mate pairs with PEAR, trimming low-quality bases and adapters with Trimmomatic, and removing rRNA reads with SortMeRNA.

## When to use

Use when raw paired-end RNA-seq/metatranscriptome FASTQ files must be cleaned before annotation: mates need merging, low-quality bases and adapters removed, and rRNA reads filtered to speed up annotation.

## When NOT to use

- Input reads are single-end only — the standard SAMSA2 preprocessing begins with PEAR merging of paired-end mates (use the master_script_preserving_unmerged variant if unmerged reads must be retained)
- Reads are already trimmed and rRNA-filtered and only annotation (DIAMOND), aggregation, or DESeq analysis is needed
- The goal is taxonomic identification from 16S rRNA sequencing rather than mRNA functional profiling from metatranscriptomes

## Inputs

- Raw paired-end metatranscriptome FASTQ read files (forward and reverse mates)

## Outputs

- cleaned_non_rRNA_reads.fastq: ribosome-filtered, quality- and adapter-trimmed merged reads ready for DIAMOND annotation

## How to apply

Load raw paired-end FASTQ metatranscriptome read files, then merge mate pairs with PEAR (the recommended merger for paired-end data in SAMSA2's dependency list) to produce merged read files of extended fragments. Clean the merged reads with Trimmomatic to remove low-quality sequences and adapter contamination, with adapters specified in a dedicated FASTA file. Filter the cleaned FASTQ with SortMeRNA to remove ribosomal reads, since these do not contribute to the mRNA functional profile; in SAMSA2 v2.2.0 the --num_alignments 0 flag in the sortmrna step is removed because rRNA alignments are discarded regardless. Write the result as cleaned_non_rRNA_reads.fastq, which becomes the input for DIAMOND annotation. In the master_script this stage is checkpointed so completed steps are skipped on rerun, and multithreading is enabled for PEAR, Trimmomatic, and SortMeRNA.

## Related tools

- **PEAR** (Merges two paired-end FASTQ sequencing files together to make a single file of extended fragments (used for paired-end data, recommended in the SAMSA2 dependency list)) — https://sco.h-its.org/exelixis/web/software/pear/
- **Trimmomatic** (Removes low-quality sequences and adapter contamination from the merged reads, using a specific FASTA file of adapter sequences) — http://www.usadellab.org/cms/?page=trimmomatic
- **SortMeRNA** (Removes ribosomal RNA reads from the cleaned FASTQ, since rRNA reads do not contribute to the mRNA profile of the metatranscriptome) — http://bioinfo.lifl.fr/RNA/sortmerna/
- **SAMSA2** (Master script that orchestrates the three preprocessing programs with checkpointing, multithreading, and disk-usage cleanup) — https://github.com/transcript/samsa2

## Examples

```
bash master_script.bash  # runs preprocessing (PEAR merge, Trimmomatic trim, SortMeRNA rRNA filter), annotation, aggregation
```

## Evaluation signals

- cleaned_non_rRNA_reads.fastq exists and is a valid FASTQ file
- The file represents ribosome-filtered reads, i.e. rRNA sequences present in the input have been removed by SortMeRNA
- Quality- and adapter-trimming is confirmed: no residual adapter contamination or low-quality tails survive Trimmomatic
- Each preprocessing step wrote its name to the 'checkpoint' file and was skipped on rerun, indicating completion without failure
- The output constitutes the expected input for the next pipeline stage: DIAMOND annotation against a local DIAMOND-indexed database

## Limitations

- Pipeline uses Python 2.7 for aggregation scripts and was only tested on Linux, despite claiming support for any OS with Python 2.7
- Multithreading, SLURM scheduling, checkpointing, and file cleanup assume an HPC-style environment; the SLURM formatting must be changed or removed elsewhere
- If paired-end reads fail to merge, only the master_script_preserving_unmerged.sh variant retains them, by concatenating forward and reverse-complement reads with 20 Ns via combining_unmerged.R

## Evidence

- [readme] PEAR - merges two paired-end FASTQ sequencing files together to make a single file of extended fragments.: "PEAR - merges two paired-end FASTQ sequencing files together to make a single file of extended fragments."
- [readme] Trimmomatic removes adapter contamination according to a specific fasta file.: "Trimmomatic removes adapter contamination according to a specific fasta file."
- [readme] SortMeRNA - removes ribosomal reads, as these don't contribute to the mRNA profile of the metatranscriptome.: "SortMeRNA - removes ribosomal reads, as these don't contribute to the mRNA profile of the metatranscriptome."
- [methods] the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation): "the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation)"
- [readme] The flag --num_alignments 0 in the ribosomal sortmrna step has been removed.: "The flag --num_alignments 0 in the ribosomal `sortmrna` step has been removed."
