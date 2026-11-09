---
name: overlap-threshold-parameter-tuning
description: Use when running 'plass assemble' on short-read sequencing data and overlap-detection
  sensitivity/specificity must be adjusted via --min-seq-id and -e before judging
  assembled protein output.
when_to_use_negative:
- The goal is nucleotide-level assembly or strain-resolved assembly of viral genomes
  or 16S rRNAs — PenguiN (penguin nuclassemble / guided_nuclassemble) is the appropriate
  tool, not Plass overlap tuning.
- Input is already assembled contigs or a protein catalog rather than short sequencing
  reads — overlap thresholds apply to assembling translated short reads, not to post-hoc
  catalog processing.
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: Plass
  role: Protein-level assembler whose overlap detection is tuned via --min-seq-id
    and -e; assembles six-frame-translated reads into protein sequences
  repo: https://github.com/soedinglab/plass
- name: conda
  role: Installation channel for Plass (conda install -c conda-forge -c bioconda plass)
  repo: https://github.com/conda-forge/miniforge
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:0247f02503ded30122a3f7e2c971644af2f515e041cfe97a30776d04a8d75264
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/plass
  tools:
  - Plass
  - conda
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# overlap-threshold-parameter-tuning

## Summary

Tunes the overlap-detection thresholds of the Plass protein-level assembler — the overlap sequence identity (--min-seq-id) and overlap E-value (-e) parameters — alongside ORF-level filters, to control how translated reads are merged into assembled protein sequences.

## When to use

Use when running 'plass assemble' on short-read sequencing data and overlap-detection sensitivity/specificity must be adjusted via --min-seq-id and -e before judging assembled protein output.

## When NOT to use

- The goal is nucleotide-level assembly or strain-resolved assembly of viral genomes or 16S rRNAs — PenguiN (penguin nuclassemble / guided_nuclassemble) is the appropriate tool, not Plass overlap tuning.
- Input is already assembled contigs or a protein catalog rather than short sequencing reads — overlap thresholds apply to assembling translated short reads, not to post-hoc catalog processing.

## Inputs

- Paired-end FASTQ read files (e.g. reads_1.fastq.gz, reads_2.fastq.gz) or single-end FASTA/FASTQ reads
- User-chosen values for --min-seq-id (overlap sequence identity threshold) and -e (overlap E-value threshold)
- ORF filter settings: --min-length (default 40 codons) and --filter-proteins flag

## Outputs

- assembly.fas — a FASTA file of assembled protein sequences assembled from six-frame-translated reads

## How to apply

Run 'plass assemble' with the paired-end FASTQ inputs (e.g. reads_1.fastq.gz and reads_2.fastq.gz), the output FASTA, and a tmp working directory; Plass six-frame-translates the reads and assembles them entirely at the protein level. During assembly, overlaps between translated sequences are detected and accepted or rejected based on two thresholds: --min-seq-id, which adjusts the overlap sequence identity threshold, and -e, the E-value threshold for overlaps — loosen or tighten these to trade assembly sensitivity against spurious merges. In parallel, configure the ORF-level filters: --min-length sets the minimum codon length for ORF prediction (default: 40), and --filter-proteins switches the neural network protein filter on. Judge correct application by validating that the output assembly.fas is a valid FASTA of assembled protein sequences honoring the 40-codon minimum ORF length, noting the paper's expectation that ORFs above 100 amino acids are mostly genuine.

## Related tools

- **Plass** (Protein-level assembler whose overlap detection is tuned via --min-seq-id and -e; assembles six-frame-translated reads into protein sequences) — https://github.com/soedinglab/plass
- **conda** (Installation channel for Plass (conda install -c conda-forge -c bioconda plass)) — https://github.com/conda-forge/miniforge

## Examples

```
plass assemble reads_1.fastq.gz reads_2.fastq.gz assembly.fas tmp --min-seq-id 0.9 -e 0.001 --min-length 40 --filter-proteins
```

## Evaluation signals

- assembly.fas exists and parses as a valid FASTA file of protein sequences
- Assembled sequences respect the ORF minimum length filter (--min-length default: 40 codons)
- ORFs longer than 100 amino acids are expected to be mostly genuine — a high fraction of short ORFs may indicate thresholds set too permissively
- Overlap acceptance behavior matches the configured --min-seq-id and -e values (no spurious merges of low-identity overlaps)
- Output scale is consistent with the paper's claim that Plass recovers 2–10 times more protein sequences from complex metagenomes than nucleotide-level assemblers

## Limitations

- The overlap thresholds trade sensitivity against specificity; inappropriate --min-seq-id / -e values can merge incorrect overlaps or fragment true proteins.
- Plass operates entirely at the protein level on six-frame-translated reads, so it is not suitable for nucleotide-level products like 16S rRNA or viral genomes — use PenguiN instead.
- For distributed (MPI) assembly, the tmp folder must be shared between all nodes, which constrains how the run can be scaled.

## Evidence

- [methods] --min-seq-id         Adjusts the overlap sequence identity threshold: "--min-seq-id         Adjusts the overlap sequence identity threshold"
- [methods] -e                   E-value threshold for overlaps: "-e                   E-value threshold for overlaps"
- [abstract] assembles six-frame-translated sequencing reads into protein sequences: "assembles six-frame-translated sequencing reads into protein sequences"
- [methods] plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
- [methods] Both tools can be distributed over several homogeneous computers. However the `tmp` folder has to be shared between all nodes: "Both tools can be distributed over several homogeneous computers. However the `tmp` folder has to be shared between all nodes"
