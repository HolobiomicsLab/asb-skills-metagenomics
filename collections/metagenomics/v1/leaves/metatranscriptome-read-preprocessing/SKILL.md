---
name: metatranscriptome-read-preprocessing
description: Use when raw paired-end FASTQ metatranscriptome reads must be converted
  into ribosome-filtered, quality- and adapter-trimmed input for DIAMOND annotation,
  e.g. as the first stage of the SAMSA2 master_script pipeline.
when_to_use_negative:
- Input is already cleaned, merged single-end FASTQ with rRNA removed — re-running
  PEAR/Trimmomatic/SortMeRNA wastes CPU on already-preprocessed reads
- Data are not metatranscriptome FASTQ reads but downstream DIAMOND tabular results
  or abundance summary tables that need aggregation instead
- Reads require rRNA-based organism identification — SortMeRNA deliberately discards
  rRNA, so rRNA-dependent approaches like MetaTrans are a better fit
edam_operation: http://edamontology.org/operation_0230
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: SAMSA2
  role: Host pipeline whose master_script.bash orchestrates the preprocessing stage
    (PEAR merge, Trimmomatic trim, SortMeRNA rRNA filter) with multithreading and
    checkpointing
  repo: https://github.com/transcript/samsa2
- name: PEAR
  role: Merges two paired-end FASTQ sequencing files together to make a single file
    of extended fragments
  repo: https://sco.h-its.org/exelixis/web/software/pear/
- name: Trimmomatic
  role: Removes low-quality sequences and adaptor contamination according to a specific
    FASTA adapter file
  repo: http://www.usadellab.org/cms/?page=trimmomatic
- name: SortMeRNA
  role: Removes ribosomal reads, which do not contribute to the mRNA profile of the
    metatranscriptome, for even more speed in annotation
  repo: http://bioinfo.lifl.fr/RNA/sortmerna/
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:549a4a47407b6017c049e2e5d84c30bb22d607f54cb502d0558dc075fdba1e4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/transcript/samsa2
  tools:
  - FLASH
  - MG-RAST
  - PEAR
  - R
  - SAMSA
  - SAMSA2
  - SortMeRNA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0230
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- metatranscriptome-preprocessing
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# metatranscriptome-read-preprocessing

## Summary

SAMSA2's preprocessing module converts raw paired-end FASTQ metatranscriptome reads into annotation-ready input by merging mates with PEAR, trimming low-quality bases and adapters with Trimmomatic, and removing rRNA with SortMeRNA so downstream DIAMOND annotation focuses on mRNA.

## When to use

Use when raw paired-end FASTQ metatranscriptome reads must be converted into ribosome-filtered, quality- and adapter-trimmed input for DIAMOND annotation, e.g. as the first stage of the SAMSA2 master_script pipeline.

## When NOT to use

- Input is already cleaned, merged single-end FASTQ with rRNA removed — re-running PEAR/Trimmomatic/SortMeRNA wastes CPU on already-preprocessed reads
- Data are not metatranscriptome FASTQ reads but downstream DIAMOND tabular results or abundance summary tables that need aggregation instead
- Reads require rRNA-based organism identification — SortMeRNA deliberately discards rRNA, so rRNA-dependent approaches like MetaTrans are a better fit

## Inputs

- Raw paired-end FASTQ metatranscriptome sequencing files (forward and reverse reads)
- FASTA file of adapter sequences used by Trimmomatic
- SAMSA2 master_script.bash configuration section specifying read and program locations

## Outputs

- cleaned_non_rRNA_reads.fastq: ribosome-filtered, quality- and adapter-trimmed merged reads ready for DIAMOND annotation
- SAMSA2 checkpoint file recording which preprocessing steps completed
- Intermediate merged-read files from PEAR

## How to apply

Run the three preprocessing programs in order: PEAR merges the forward/reverse mate-pair FASTQ files into a single file of extended fragments (recommended for paired-end data); Trimmomatic then removes low-quality sequences and adaptor contamination using a specific FASTA file of adapter sequences; SortMeRNA finally filters out ribosomal reads, which do not contribute to the mRNA functional profile. Version 2.2.0 of SAMSA2 runs PEAR, Trimmomatic, and SortMeRNA with multithreading and writes a checkpoint file so finished CPU-intensive steps are skipped on rerun. The final product should be written to cleaned_non_rRNA_reads.fastq, which is the direct input to DIAMOND annotation (DIAMOND 0.8.3 against local RefSeq, SEED Subsystems, or CAZy databases). These steps can be executed through master_script.bash or individually, and all option/read/program locations are set in the first section of the script.

## Related tools

- **SAMSA2** (Host pipeline whose master_script.bash orchestrates the preprocessing stage (PEAR merge, Trimmomatic trim, SortMeRNA rRNA filter) with multithreading and checkpointing) — https://github.com/transcript/samsa2
- **PEAR** (Merges two paired-end FASTQ sequencing files together to make a single file of extended fragments) — https://sco.h-its.org/exelixis/web/software/pear/
- **Trimmomatic** (Removes low-quality sequences and adaptor contamination according to a specific FASTA adapter file) — http://www.usadellab.org/cms/?page=trimmomatic
- **SortMeRNA** (Removes ribosomal reads, which do not contribute to the mRNA profile of the metatranscriptome, for even more speed in annotation) — http://bioinfo.lifl.fr/RNA/sortmerna/

## Examples

```
git clone https://github.com/transcript/samsa2.git && bash master_script.bash
```

## Evaluation signals

- cleaned_non_rRNA_reads.fastq exists and is a syntactically valid FASTQ file
- Reads in the output represent merged (PEAR) fragments that have passed Trimmomatic quality/adapter trimming and SortMeRNA rRNA filtering
- The checkpoint file lists the PEAR, Trimmomatic, and SortMeRNA steps as completed, and reruns skip these steps
- Disk usage stays low because unnecessary intermediate files are removed after each step
- Downstream DIAMOND annotation against RefSeq/SEED Subsystems accepts the file without format errors

## Limitations

- The pipeline requires Python 2 for aggregation scripts and is tested only on Linux despite claiming any OS supporting Python 2.7
- Trimmomatic adapter removal depends on a user-supplied specific FASTA file; wrong adapter sequences leave contamination
- The --num_alignments 0 flag in the ribosomal sortmrna step was removed because it slowed things down considerably
- The script is formatted for a SLURM HPC scheduler and must be edited if run elsewhere; all options and program locations must be set manually in the script's first section
- A 'preserving unmerged' variant exists because the default discards unmerged pairs; unmerged reads are handled via combining_unmerged.R

## Evidence

- [readme] PEAR - merges two paired-end FASTQ sequencing files together to make a single file of extended fragments.: "PEAR - merges two paired-end FASTQ sequencing files together to make a single file of extended fragments."
- [readme] SortMeRNA removes ribosomal sequences, as these don't contribute to the mRNA functional profile of the metatranscriptome.: "SortMeRNA removes ribosomal sequences, as these don't contribute to the mRNA functional profile of the metatranscriptome."
- [readme] Trimmomatic is used for the removal of adaptor contamination and low-quality reads.: "Trimmomatic is used for the removal of adaptor contamination and low-quality reads."
- [readme] Multithreading added for PEAR, Trimmomatic, SortMeRNA: "Multithreading added for PEAR, Trimmomatic, SortMeRNA"
- [methods] the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation): "the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation)"
