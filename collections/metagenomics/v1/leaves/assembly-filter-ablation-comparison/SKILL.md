---
name: assembly-filter-ablation-comparison
description: Use when you must quantify how an assembly-stage filter (e.g. Plass's
  --filter-proteins neural network) changes the retained sequence set, by comparing
  filtered vs unfiltered candidate ORFs.
when_to_use_negative:
- The candidate sequences are already known-genuine proteins (e.g. from curated catalogs
  like UniProt), so an on/off ablation of a coding-potential filter yields no information
- You are assembling nucleotide contigs rather than protein sequences (use PenguiN's
  guided_nuclassemble/nuclassemble instead of the Plass protein filter)
- The goal is redundancy reduction of a finished catalog (Plass catalogs are redundancy
  reduced to 90% sequence identity at 90% coverage, a clustering step distinct from
  the neural-network ORF filter)
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: Plass
  role: De novo protein-level assembler producing candidate ORFs; its --filter-proteins
    option toggles the neural-network protein filter on/off for the ablation
  repo: https://github.com/soedinglab/plass
- name: C++
  role: Language in which Plass is implemented and in which the reimplemented filter
    is reconstructed from the Plass source
  repo: https://github.com/soedinglab/plass
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-019-0437-4
    title: Protein-level assembly increases protein sequence recovery from metagenomic
      samples manyfold
  source_content_hash: sha256:a9edba7cf6597263dc528385c5476af5d37d5fc99f618a9a629c4b3a80962411
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/soedinglab/plass
  tools:
  - C++
  - Plass
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# assembly-filter-ablation-comparison

## Summary

Ablation-comparison skill for Plass protein-level assembly: run the neural-network protein-coding-potential filter (--filter-proteins) in on and off modes over the same assembled candidate ORFs and compare retention to quantify which assemblies the filter keeps or rejects.

## When to use

Use when you must quantify how an assembly-stage filter (e.g. Plass's --filter-proteins neural network) changes the retained sequence set, by comparing filtered vs unfiltered candidate ORFs.

## When NOT to use

- The candidate sequences are already known-genuine proteins (e.g. from curated catalogs like UniProt), so an on/off ablation of a coding-potential filter yields no information
- You are assembling nucleotide contigs rather than protein sequences (use PenguiN's guided_nuclassemble/nuclassemble instead of the Plass protein filter)
- The goal is redundancy reduction of a finished catalog (Plass catalogs are redundancy reduced to 90% sequence identity at 90% coverage, a clustering step distinct from the neural-network ORF filter)

## Inputs

- Paired-end FASTQ short reads (e.g. reads_1.fastq.gz, reads_2.fastq.gz) from complex metagenomic samples
- Candidate ORF protein sequences from 'plass assemble' output (assembly.fas), including ORFs below 60 amino acids

## Outputs

- filtered_proteins.fas: protein set retained after the filter-on run (and the filter-off counterpart)
- filter_report.tsv: ORF retention counts by length bin (<60, 60-100, >100 aa) for filter-on vs filter-off modes

## How to apply

First run 'plass assemble' on paired-end FASTQ reads with the default ORF prediction minimum length of 40 codons (--min-length 40) to produce candidate protein sequences in assembly.fas; Plass emits all ORFs with a start and end codon, including very short ORFs below 60 amino acids. Then reconstruct or invoke the neural-network protein filter, which Plass exposes via the --filter-proteins option that 'switches the neural network protein filter off/on'. Apply the filter to the same candidate ORFs in two modes (filter on, filter off), retaining the survivors as filtered_proteins.fas. Tabulate ORF retention counts by length bin (<60, 60-100, >100 amino acids) for both modes into filter_report.tsv. Judge correctness by the paper's expectation that assemblies above 100 amino acids are mostly genuine protein sequences and that spurious short ORFs (<60 aa) are preferentially rejected when the filter is on.

## Related tools

- **Plass** (De novo protein-level assembler producing candidate ORFs; its --filter-proteins option toggles the neural-network protein filter on/off for the ablation) — https://github.com/soedinglab/plass
- **C++** (Language in which Plass is implemented and in which the reimplemented filter is reconstructed from the Plass source) — https://github.com/soedinglab/plass

## Examples

```
plass assemble reads_1.fastq.gz reads_2.fastq.gz assembly.fas tmp --min-length 40 --filter-proteins
```

## Evaluation signals

- The filter-on output is dominated by assemblies above 100 amino acids, which the paper states are mostly genuine protein sequences
- Spurious short ORFs (<60 aa) are preferentially rejected in the filter-on run relative to the filter-off run
- Both ablation modes are run over the identical candidate ORF set from assembly.fas, so differences in filtered_proteins.fas are attributable only to the filter
- filter_report.tsv contains retention counts for all three length bins (<60, 60-100, >100 aa) for both modes
- The assembly step used the documented default ORF prediction minimum length (--min-length 40 codons)

## Limitations

- Plass emits all ORFs having a start and end codon, including very short ORFs below 60 amino acids, which inflate the unfiltered candidate set
- The neural-network filter must be reimplemented from the Plass source repository (github.com/soedinglab/plass) when reproducing its behavior outside the tool
- Both tools (Plass/PenguiN) can be distributed over several computers, but the tmp folder has to be shared between all nodes

## Evidence

- [methods] Plass provides a neural network protein filter controlled through --filter-proteins: "--filter-proteins    Switches the neural network protein filter off/on"
- [methods] ORF minimum length default: "--min-length         minimum codon length for ORF prediction (default: 40)"
- [abstract] Plass assembles six-frame-translated reads into protein sequences: "assembles six-frame-translated sequencing reads into protein sequences"
- [methods] Plass accepts paired-end FASTQ and single reads: "Plass and PenguiN can assemble both paired-end reads (FASTQ) and single reads (FASTA or FASTQ)"
- [readme] Example assemble command from the README: "plass assemble examples/reads_1.fastq.gz examples/reads_2.fastq.gz assembly.fas tmp"
- [readme] Plass is open-source software in C++: "GPL-licensed open source software implemented in C++ and available for Linux and macOS"
