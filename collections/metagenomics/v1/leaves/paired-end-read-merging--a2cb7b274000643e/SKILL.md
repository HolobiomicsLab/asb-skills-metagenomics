---
name: paired-end-read-merging
description: Use when raw paired-end metatranscriptome RNA-seq data in FASTQ form
  must be converted into a single merged, trimmed, rRNA-filtered FASTQ before DIAMOND
  annotation, as in the SAMSA2 preprocessing step.
when_to_use_negative:
- Input is single-end sequencing — PEAR merging is only used when the sequencing was
  paired-end
- Input is already merged, trimmed or rRNA-filtered FASTQ, or a downstream abundance
  summary — re-running preprocessing would waste CPU-intensive compute
- The goal is 16S rRNA amplicon analysis — SortMeRNA would discard the rRNA signal
  these steps are designed to remove
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3308
tools:
- name: PEAR
  role: Merges two paired-end FASTQ sequencing files together to make a single file
    of extended fragments (recommended for paired-end data)
  repo: https://sco.h-its.org/exelixis/web/software/pear/
- name: Trimmomatic
  role: Removes low-quality sequences and adaptor contamination from merged reads
  repo: http://www.usadellab.org/cms/?page=trimmomatic
- name: SortMeRNA
  role: Removes ribosomal reads that do not contribute to the mRNA functional profile
    of the metatranscriptome
  repo: http://bioinfo.lifl.fr/RNA/sortmerna/
- name: SAMSA2
  role: Wraps the three preprocessing steps in master_script.bash (with checkpointing,
    multithreading and an unmerged-preserving variant)
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:c9bed634246c5960becb5ade3edab39d16c9b6a9cbaeacc6e278f57e67209cb5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PEAR
  - SAMSA2
  - SortMeRNA
  - Trimmomatic
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# paired-end-read-merging

## Summary

Merges paired-end metatranscriptomic FASTQ mate pairs with PEAR, then quality/adaptor-trims with Trimmomatic and filters rRNA with SortMeRNA so only non-ribosomal reads feed DIAMOND annotation in the SAMSA2 pipeline.

## When to use

Use when raw paired-end metatranscriptome RNA-seq data in FASTQ form must be converted into a single merged, trimmed, rRNA-filtered FASTQ before DIAMOND annotation, as in the SAMSA2 preprocessing step.

## When NOT to use

- Input is single-end sequencing — PEAR merging is only used when the sequencing was paired-end
- Input is already merged, trimmed or rRNA-filtered FASTQ, or a downstream abundance summary — re-running preprocessing would waste CPU-intensive compute
- The goal is 16S rRNA amplicon analysis — SortMeRNA would discard the rRNA signal these steps are designed to remove

## Inputs

- Raw paired-end metatranscriptome FASTQ read files (forward and reverse mates)
- Adaptor-contamination FASTA file used by Trimmomatic
- SortMeRNA rRNA reference databases

## Outputs

- Merged FASTQ of extended fragments (PEAR output)
- Quality- and adaptor-trimmed FASTQ (Trimmomatic output)
- cleaned_non_rRNA_reads.fastq — ribosome-filtered FASTQ ready for DIAMOND annotation

## How to apply

If the sequencing was paired-end, merge mate pairs with PEAR (the merge tool recommended in the SAMSA2 dependency list) to produce a single file of extended fragments; if unmerged pairs must be retained, use master_script_preserving_unmerged.sh, whose combining_unmerged.R concatenates forward and reverse-complement reads with a string of 20 Ns. Clean the merged reads with Trimmomatic to remove low-quality sequences and adaptor contamination according to a specific adaptor FASTA file. Filter ribosomal reads with SortMeRNA, since rRNA does not contribute to the mRNA functional profile; in SAMSA2 v2.2 the --num_alignments 0 flag was removed from the sortmrna step because it caused problems and slowed things down. PEAR, Trimmomatic and SortMeRNA are multithreaded in v2.2.0, all options and read/program locations are set in the first section of master_script.bash, and a checkpoint file records completed steps so CPU-intensive stages are skipped on rerun. The final ribosome-filtered FASTQ (cleaned_non_rRNA_reads.fastq) is the input for DIAMOND annotation against a local, DIAMOND-indexed database.

## Related tools

- **PEAR** (Merges two paired-end FASTQ sequencing files together to make a single file of extended fragments (recommended for paired-end data)) — https://sco.h-its.org/exelixis/web/software/pear/
- **Trimmomatic** (Removes low-quality sequences and adaptor contamination from merged reads) — http://www.usadellab.org/cms/?page=trimmomatic
- **SortMeRNA** (Removes ribosomal reads that do not contribute to the mRNA functional profile of the metatranscriptome) — http://bioinfo.lifl.fr/RNA/sortmerna/
- **SAMSA2** (Wraps the three preprocessing steps in master_script.bash (with checkpointing, multithreading and an unmerged-preserving variant)) — https://github.com/transcript/samsa2

## Examples

```
bash master_script.bash  # PEAR merge -> Trimmomatic trim -> SortMeRNA rRNA filter, producing cleaned_non_rRNA_reads.fastq
```

## Evaluation signals

- cleaned_non_rRNA_reads.fastq exists and is a valid FASTQ file representing merged, trimmed, rRNA-filtered reads
- rRNA-derived reads are absent (or strongly depleted) in the output compared with the pre-SortMeRNA file
- Adaptor sequences and low-quality tails are removed in the Trimmomatic output per the supplied adaptor FASTA
- When master_script_preserving_unmerged.sh is used, unmerged pairs appear as concatenated reads with a string of 20 Ns between forward and reverse-complement portions
- The checkpoint file lists completed steps so reruns skip the already-finished CPU-intensive PEAR/Trimmomatic/SortMeRNA stages

## Limitations

- SAMSA2 requires Python 2.7 for aggregation scripts (tested on Linux only; Python 3 support may have uncaught errors)
- By default unmerged mate pairs are not carried into the merged file; retaining them requires the v2.2 master_script_preserving_unmerged.sh variant with combining_unmerged.R
- Removing the --num_alignments 0 flag in the SortMeRNA step was needed because it caused problems and slowed the pipeline down

## Evidence

- [readme] PEAR merges paired-end mate pairs before trimming: "If the sequencing was paired-end, PEAR is used to merge mate pairs."
- [readme] PEAR produces extended fragments from paired-end FASTQ: "PEAR - merges two paired-end FASTQ sequencing files together to make a single file of extended fragments."
- [readme] Trimmomatic and SortMeRNA roles in preprocessing: "Trimmomatic is used for the removal of adaptor contamination and low-quality reads.  SortMeRNA removes ribosomal sequences"
- [readme] SortMeRNA rationale — rRNA does not inform mRNA profile: "SortMeRNA - removes ribosomal reads, as these don't contribute to the mRNA profile of the metatranscriptome."
- [readme] SortMeRNA flag removal in v2.2: "The flag --num_alignments 0 in the ribosomal `sortmrna` step has been removed. This caused problems and slowed things down a lot."
- [readme] Unmerged reads concatenated with 20 Ns: "The forward read and the reverse (complement) read are concatenated with a string of 20 Ns in the middle"
- [methods] Preprocessing is the first of four SAMSA2 stages: "the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation)"
