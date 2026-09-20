---
name: orf-coding-potential-classification
description: Use when working with six-frame-translated short-read assemblies (e.g.
  'plass assemble' output FASTA) where short ORFs below 60 aa may be spurious and
  a coding-potential decision is needed per candidate protein.
when_to_use_negative:
- Input is already a curated, redundancy-reduced protein catalog (e.g. SRC/MERC released
  at 90% identity, 90% coverage) rather than raw assembly candidates containing all
  start/end-codon ORFs
- The goal is nucleotide-level contig assembly (use PenguiN's nuclassemble/guided_nuclassemble
  instead of Plass's protein filter)
- ORFs are already length-filtered above 100 amino acids, where the paper states assemblies
  are mostly genuine and the filter contributes little discrimination
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Plass
  role: Provides the neural-network protein filter (--filter-proteins) applied to
    candidate ORFs from six-frame-translated assembly; also runs the ORF prediction
    with --min-length
  repo: https://github.com/soedinglab/plass
- name: C++
  role: Implementation language used to reimplement the neural-network protein-coding-potential
    filter from the Plass source repository
  repo: https://github.com/soedinglab/plass
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:e7891966c4d9ab454a3e5cedd6c4db39b9952a48eb2794631e9b0d50e37fbd26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/plass
  tools:
  - C++
  - Plass
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# orf-coding-potential-classification

## Summary

Classifies candidate open reading frames produced during Plass protein-level assembly as coding or spurious using Plass's neural network protein filter, toggled by the --filter-proteins option, so that only genuine protein sequences are retained in the assembled catalog.

## When to use

Use when working with six-frame-translated short-read assemblies (e.g. 'plass assemble' output FASTA) where short ORFs below 60 aa may be spurious and a coding-potential decision is needed per candidate protein.

## When NOT to use

- Input is already a curated, redundancy-reduced protein catalog (e.g. SRC/MERC released at 90% identity, 90% coverage) rather than raw assembly candidates containing all start/end-codon ORFs
- The goal is nucleotide-level contig assembly (use PenguiN's nuclassemble/guided_nuclassemble instead of Plass's protein filter)
- ORFs are already length-filtered above 100 amino acids, where the paper states assemblies are mostly genuine and the filter contributes little discrimination

## Inputs

- Candidate protein/ORF sequences in FASTA (e.g. assembly.fas from 'plass assemble')
- Paired-end metagenomic reads in FASTQ (e.g. reads_1.fastq.gz, reads_2.fastq.gz)

## Outputs

- Filtered protein set in FASTA (filtered_proteins.fas) after the filter-on run
- ORF retention counts tabulated by length bin (<60, 60-100, >100 aa) for filter-on vs filter-off modes (e.g. filter_report.tsv)

## How to apply

Run 'plass assemble' on paired-end FASTQ reads with the default ORF prediction minimum length of 40 codons (--min-length 40), which emits all ORFs with a start and end codon, including very short ORFs below 60 amino acids. Parse the candidate ORFs from assembly.fas, then apply the neural-network protein-coding-potential filter, which is controlled in Plass by the --filter-proteins option that switches the filter off or on. Compare filter-on and filter-off retention behavior to characterize the filter: bin retained ORFs by length (<60, 60-100, >100 amino acids) since the paper states assemblies above 100 aa are mostly genuine protein sequences while short ORFs are preferentially rejected. If reimplementing the filter in C++, reconstruct its behavior from the Plass source repository (github.com/soedinglab/plass), where it is toggled by --filter-proteins, and validate against the two-mode comparison.

## Related tools

- **Plass** (Provides the neural-network protein filter (--filter-proteins) applied to candidate ORFs from six-frame-translated assembly; also runs the ORF prediction with --min-length) — https://github.com/soedinglab/plass
- **C++** (Implementation language used to reimplement the neural-network protein-coding-potential filter from the Plass source repository) — https://github.com/soedinglab/plass

## Examples

```
plass assemble --filter-proteins examples/reads_1.fastq.gz examples/reads_2.fastq.gz filtered_proteins.fas tmp
```

## Evaluation signals

- Filtered output (filter-on run) is dominated by assemblies above 100 amino acids, which the paper states are mostly genuine protein sequences
- Spurious short ORFs (<60 amino acids) are preferentially rejected in the filter-on run relative to the filter-off run
- Retention counts by length bin (<60, 60-100, >100 aa) differ measurably between filter-on and filter-off modes, and are tabulated in filter_report.tsv
- Reimplemented filter behavior is consistent with the --filter-proteins toggle semantics in the Plass source repository (github.com/soedinglab/plass)

## Limitations

- Plass emits all ORFs having a start and end codon, including very short ORFs below 60 amino acids, so unfiltered candidate sets contain many spurious sequences
- The article documents the filter only as an on/off CLI toggle; the paper does not describe filter architecture details, so reimplementation requires reconstructing behavior from the source repository
- ORF prediction minimum length defaults to 40 codons; changing --min-length alters the candidate pool the filter must classify

## Evidence

- [methods] --filter-proteins    Switches the neural network protein filter off/on: "--filter-proteins    Switches the neural network protein filter off/on"
- [methods] --min-length         minimum codon length for ORF prediction (default: 40): "--min-length         minimum codon length for ORF prediction (default: 40)"
- [readme] Plass and PenguiN ... assemble protein sequences or DNA/RNA contigs from short read sequencing data: "Plass (Protein-Level ASSembler) and PenguiN (Protein guided nucleotide assembler) are software to assemble protein sequences or DNA/RNA contigs from short read sequencing data"
- [methods] plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
