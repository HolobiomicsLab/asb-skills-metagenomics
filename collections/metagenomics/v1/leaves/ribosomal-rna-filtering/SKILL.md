---
name: ribosomal-rna-filtering
description: Use when raw or quality-trimmed metatranscriptome RNA-seq FASTQ reads
  still contain rRNA sequences that would waste annotation compute and dilute mRNA
  functional profiles.
when_to_use_negative:
- Input is already an annotation or abundance table rather than sequence reads — rRNA
  filtering only applies at the FASTQ stage.
- The analysis targets rRNA itself (e.g., 16S taxonomic profiling), where removing
  ribosomal reads would discard the signal of interest.
- Reads are genomic metagenome reads from which rRNA content is not the concern the
  pipeline is designed to address.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: SortMeRNA
  role: Classifies and removes ribosomal RNA reads from the cleaned FASTQ, producing
    the non-rRNA read file for annotation
  repo: http://bioinfo.lifl.fr/RNA/sortmerna/
- name: Trimmomatic
  role: Upstream step that removes adaptor contamination and low-quality sequences
    before rRNA filtering
  repo: http://www.usadellab.org/cms/?page=trimmomatic
- name: PEAR
  role: Upstream step that merges paired-end FASTQ mates into single extended-fragment
    reads prior to filtering
  repo: https://sco.h-its.org/exelixis/web/software/pear/
- name: SAMSA2
  role: Pipeline wrapper (master_script.bash) that orchestrates multithreaded SortMeRNA
    filtering with checkpointing and cleanup
  repo: https://github.com/transcript/samsa2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-018-2189-z
    title: 'SAMSA2: a standalone metatranscriptome analysis pipeline'
  source_content_hash: sha256:8600623de717bf2b196d803c67c9c6abcd5cd205eb395f011d0877fae9bdf58e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PEAR
  - SAMSA2
  - SortMeRNA
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# ribosomal-rna-filtering

## Summary

Removes ribosomal RNA reads from cleaned metatranscriptomic FASTQ using SortMeRNA within the SAMSA2 preprocessing module, so that only non-rRNA sequences carrying the mRNA functional profile proceed to DIAMOND annotation.

## When to use

Use when raw or quality-trimmed metatranscriptome RNA-seq FASTQ reads still contain rRNA sequences that would waste annotation compute and dilute mRNA functional profiles.

## When NOT to use

- Input is already an annotation or abundance table rather than sequence reads — rRNA filtering only applies at the FASTQ stage.
- The analysis targets rRNA itself (e.g., 16S taxonomic profiling), where removing ribosomal reads would discard the signal of interest.
- Reads are genomic metagenome reads from which rRNA content is not the concern the pipeline is designed to address.

## Inputs

- Adaptor- and quality-trimmed merged FASTQ reads (output of PEAR + Trimmomatic)
- SortMeRNA rRNA reference databases (bundled/installed with SortMeRNA)

## Outputs

- cleaned_non_rRNA_reads.fastq — ribosome-filtered FASTQ ready for DIAMOND annotation
- Discarded rRNA read set (implicitly removed)

## How to apply

After PEAR merges paired-end FASTQ mates and Trimmomatic removes low-quality bases and adaptor contamination, run SortMeRNA on the cleaned FASTQ to classify and remove reads aligning to ribosomal RNA databases. In SAMSA2's master_script this is multithreaded, and the --num_alignments 0 flag was deliberately removed from the sortmrna step because it caused problems and slowed execution — alignment counts to rRNA are irrelevant since any aligned read is discarded regardless. The surviving non-rRNA reads are written to cleaned_non_rRNA_reads.fastq, which is the direct input to the DIAMOND annotation step. Optionally, the companion master_script_preserving_unmerged.sh can keep unmerged reads (concatenated with 20 Ns) in the pipeline before filtering.

## Related tools

- **SortMeRNA** (Classifies and removes ribosomal RNA reads from the cleaned FASTQ, producing the non-rRNA read file for annotation) — http://bioinfo.lifl.fr/RNA/sortmerna/
- **Trimmomatic** (Upstream step that removes adaptor contamination and low-quality sequences before rRNA filtering) — http://www.usadellab.org/cms/?page=trimmomatic
- **PEAR** (Upstream step that merges paired-end FASTQ mates into single extended-fragment reads prior to filtering) — https://sco.h-its.org/exelixis/web/software/pear/
- **SAMSA2** (Pipeline wrapper (master_script.bash) that orchestrates multithreaded SortMeRNA filtering with checkpointing and cleanup) — https://github.com/transcript/samsa2

## Examples

```
bash master_script.bash  # performs PEAR merge → Trimmomatic trim → SortMeRNA rRNA filter → writes cleaned_non_rRNA_reads.fastq
```

## Evaluation signals

- cleaned_non_rRNA_reads.fastq exists and is a valid FASTQ file (4 lines per record, matching IDs, Phred quality lines).
- Read count in cleaned_non_rRNA_reads.fastq is ≤ the input cleaned-read count; the difference corresponds to rRNA hits removed.
- No reads in the output align to the SortMeRNA rRNA reference databases when re-screened.
- The SAMSA2 checkpoint file records the rRNA-filtering step as completed, allowing rerun to skip it.
- Downstream DIAMOND annotation accepts the file directly and produces per-read results.

## Limitations

- The --num_alignments 0 flag in the sortmrna step caused problems and slowed things down; it was removed in SAMSA2 v2.2.0.
- SAMSA2 is tested on Linux and requires Python 2.7 for aggregation scripts; full Python 3 support may have uncaught errors.
- Unmerged paired-end reads are discarded unless the master_script_preserving_unmerged.sh variant is used, which concatenates them with 20 Ns.
- Any read aligning to rRNA is removed regardless of alignment count, so borderline/partial rRNA hits are not recoverable from the main output.

## Evidence

- [readme] SortMeRNA removes ribosomal sequences, as these don't contribute to the mRNA functional profile of the metatranscriptome.: "SortMeRNA removes ribosomal sequences, as these don't contribute to the mRNA functional profile of the metatranscriptome."
- [readme] The flag --num_alignments 0 in the ribosomal `sortmrna` step has been removed. This caused problems and slowed things down a lot.: "The flag --num_alignments 0 in the ribosomal `sortmrna` step has been removed. This caused problems and slowed things down a lot."
- [readme] Filtering of ribosomes for even more speed.: "Filtering of ribosomes for even more speed."
- [methods] preprocessing, annotation, aggregation: "the first 3 of 4 steps in the SAMSA2 pipeline (preprocessing, annotation, aggregation)"
- [readme] Trimmomatic removes adapter contamination according to a specific fasta file.: "Trimmomatic removes adapter contamination according to a specific fasta file."
