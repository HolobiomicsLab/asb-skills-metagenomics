---
name: protein-assembly-orf-length-distribution-analysis
description: Use when metagenomic short reads are six-frame-translated and assembled
  with Plass and you need to quantify ORF length retention (e.g. <60, 60-100, >100
  aa bins) with the neural-network protein filter on versus off.
when_to_use_negative:
- Input is already a redundancy-filtered protein catalog (e.g. the 90% sequence identity
  / 90% coverage Soil Reference Catalog) rather than raw assembled ORF candidates
  needing length-distribution filtering.
- The goal is nucleotide-level assembly of contigs (PenguiN nuclassemble/guided_nuclassemble)
  rather than protein-level ORF filtering.
- Only ORFs above 100 amino acids are of interest, since the length-bin comparison
  below 60 aa is specifically aimed at detecting spurious short ORF rejection by the
  filter.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Plass
  role: six-frame-translates short reads and assembles candidate protein ORFs (plass
    assemble); its --filter-proteins option toggles the neural-network protein filter
    and --min-length 40 sets the ORF prediction minimum
  repo: https://github.com/soedinglab/plass
- name: C++
  role: implementation language for reconstructing the neural-network protein-coding-potential
    filter behavior from the Plass source repository
  repo: https://github.com/soedinglab/plass
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:13ede39693cd4f3b68c57cae150f5e62e9128204793adec6c8dfb96bda7005a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/plass
  tools:
  - C++
  - Plass
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# protein-assembly-orf-length-distribution-analysis

## Summary

Analyzes the length distribution of ORFs produced by Plass protein-level assembly and evaluates how the neural-network protein filter (--filter-proteins) reshapes that distribution, particularly the retention of spurious short ORFs (<60 aa) versus assemblies above 100 amino acids.

## When to use

Use when metagenomic short reads are six-frame-translated and assembled with Plass and you need to quantify ORF length retention (e.g. <60, 60-100, >100 aa bins) with the neural-network protein filter on versus off.

## When NOT to use

- Input is already a redundancy-filtered protein catalog (e.g. the 90% sequence identity / 90% coverage Soil Reference Catalog) rather than raw assembled ORF candidates needing length-distribution filtering.
- The goal is nucleotide-level assembly of contigs (PenguiN nuclassemble/guided_nuclassemble) rather than protein-level ORF filtering.
- Only ORFs above 100 amino acids are of interest, since the length-bin comparison below 60 aa is specifically aimed at detecting spurious short ORF rejection by the filter.

## Inputs

- paired-end FASTQ metagenomic reads (reads_1.fastq.gz, reads_2.fastq.gz)
- candidate ORF protein sequences in FASTA format (assembly.fas) produced by plass assemble with --min-length 40

## Outputs

- filtered_proteins.fas: retained protein set after the neural-network filter (filter-on and filter-off modes)
- filter_report.tsv: ORF retention counts by length bin (<60, 60-100, >100 amino acids) for filter-on and filter-off runs

## How to apply

Run 'plass assemble' on paired-end FASTQ reads (e.g. reads_1.fastq.gz and reads_2.fastq.gz into assembly.fas) with the default ORF prediction minimum length of 40 codons (--min-length 40), which emits all ORFs having a start and end codon, including very short ORFs below 60 amino acids. Reimplement or invoke the neural-network protein-coding-potential filter from the Plass source (github.com/soedinglab/plass), which is toggled by --filter-proteins (switches the filter off/on). Apply the filter in both modes to the candidate ORFs to produce the retained protein set filtered_proteins.fas. Tabulate ORF retention counts by length bin (<60, 60-100, >100 amino acids) for both filter modes into filter_report.tsv. The rationale is that Plass assemblies above 100 amino acids are mostly genuine protein sequences, so the filter-on run should preferentially reject spurious short ORFs while retaining long assemblies.

## Related tools

- **Plass** (six-frame-translates short reads and assembles candidate protein ORFs (plass assemble); its --filter-proteins option toggles the neural-network protein filter and --min-length 40 sets the ORF prediction minimum) — https://github.com/soedinglab/plass
- **C++** (implementation language for reconstructing the neural-network protein-coding-potential filter behavior from the Plass source repository) — https://github.com/soedinglab/plass

## Examples

```
plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp --min-length 40 --filter-proteins
```

## Evaluation signals

- filter_report.tsv shows the filter-on run preferentially rejects ORFs in the <60 amino acid bin relative to the filter-off run.
- The filtered output is dominated by assemblies above 100 amino acids, which the paper states are mostly genuine protein sequences.
- assembly.fas contains ORFs whose lengths respect the --min-length 40 codon default for ORF prediction.
- The filter-on and filter-off runs are reproducible end-to-end from the same paired-end FASTQ input via the same plass assemble command.
- Plass recovers 2-10 times more protein sequences from complex metagenomes than prior nucleotide-level assembly, so the filtered set size should remain consistent with protein-level assembly gains.

## Limitations

- Plass emits all ORFs having a start and end codon, including very short ORFs below 60 amino acids, so unfiltered candidate sets are heavily contaminated with spurious short ORFs.
- The exact behavior of the neural-network protein filter must be reconstructed from the Plass source repository (github.com/soedinglab/plass); the CLI only exposes an off/on switch.
- Both paired-end and single-read modes are supported (FASTQ or FASTA), so input format inconsistencies between paired-end and single-end workflows can affect the ORF candidate pool.

## Evidence

- [methods] --filter-proteins    Switches the neural network protein filter off/on: "--filter-proteins    Switches the neural network protein filter off/on"
- [methods] --min-length         minimum codon length for ORF prediction (default: 40): "--min-length         minimum codon length for ORF prediction (default: 40)"
- [methods] plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
- [abstract] It recovers 2-10 times more protein sequences from complex metagenomes and can assemble huge datasets.: [evidence span withheld — anchor status: not_found]
- [readme] software to assemble protein sequences or DNA/RNA contigs from short read sequencing data meant to work best for complex metagenomic or metatranscriptomic datasets: "software to assemble protein sequences or DNA/RNA contigs from short read sequencing data meant to work best for complex metagenomic or metatranscriptomic datasets"
