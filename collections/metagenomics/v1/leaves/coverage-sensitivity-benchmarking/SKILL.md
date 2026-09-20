---
name: coverage-sensitivity-benchmarking
description: Use when a genome in a simulated or real metagenomic community is assembled
  at extreme coverage (e.g. >1000x) and shows a high number of misassembled blocks,
  and you need to test whether coverage rather than assembler or sequence properties
  caused the errors.
when_to_use_negative:
- The assembly error you are investigating occurs on a low- or normal-coverage genome,
  not a high-coverage (>1000x) one — coverage reduction will not test that hypothesis
- You need to compare assemblers or k-mer choices rather than isolate the effect of
  coverage; changing any parameter besides coverage would confound the comparison
- Input is a finished assembly without the underlying reads — subsampling and reassembly
  require the original FASTQ files
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: Two-stage de Bruijn graph assembler used to reassemble both full-coverage
    and 10%-subsampled reads into contigs for misassembly comparison
  repo: https://github.com/SR-Martin/metacortex.git
- name: GCC
  role: Compiler required to build MetaCortex from source with a chosen MAXK value
    (LLVM will not compile it)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:c96d3f22b9bbd2316062d288c4506f8f27ed73f2cd49d07472f0446261472cb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - GCC
  - MetaCortex
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# coverage-sensitivity-benchmarking

## Summary

Tests whether ultra-high sequencing coverage drives assembly errors by subsampling reads (e.g. to 10% of original coverage), reassembling with MetaCortex under identical parameters, and comparing misassembled-block counts against the reference.

## When to use

Use when a genome in a simulated or real metagenomic community is assembled at extreme coverage (e.g. >1000x) and shows a high number of misassembled blocks, and you need to test whether coverage rather than assembler or sequence properties caused the errors.

## When NOT to use

- The assembly error you are investigating occurs on a low- or normal-coverage genome, not a high-coverage (>1000x) one — coverage reduction will not test that hypothesis
- You need to compare assemblers or k-mer choices rather than isolate the effect of coverage; changing any parameter besides coverage would confound the comparison
- Input is a finished assembly without the underlying reads — subsampling and reassembly require the original FASTQ files

## Inputs

- Simulated or real metagenomic FASTQ reads with known per-genome coverage levels
- Reference genomes (FASTA) for the community, e.g. the S.bongori sim.2 reference
- Full-coverage MetaCortex assembly contigs (baseline)
- Subsampled reads at 10% of the original coverage

## Outputs

- Subsampled-coverage assembly contigs from the MetaCortex two-stage (CTX import + combine) pipeline
- Misassembled-block counts per genome for both full and subsampled coverage
- Fold-reduction table (e.g. misassembly_comparison.tsv) quantifying the coverage effect

## How to apply

Compile MetaCortex from source with the same maximum k-mer used for the full-coverage run (make MAXK=<K>, where K is one of 31, 63, 95, 127, 160 or 192 nucleotides; the bioconda build is fixed at 127), since the assembly graph k-mer size must match between runs. Subsample the read set to 10% of original coverage, then run the MetaCortex two-stage assembly — parallel import of each reads file into a CTX de Bruijn graph file, then combining the CTX files to output contigs — using exactly the same parameters as the full-coverage assembly. Align both the subsampled and full-coverage assemblies against the reference and count misassembled blocks attributable to the high-coverage genome (e.g. S.bongori sim.2 at >1000x). A large fold-reduction in misassembled blocks at reduced coverage (the source task observed ~50%) supports the hypothesis that ultra-high coverage caused the misassemblies. Record per-genome misassembly counts for both coverage levels and the fold-reduction in a comparison table.

## Related tools

- **MetaCortex** (Two-stage de Bruijn graph assembler used to reassemble both full-coverage and 10%-subsampled reads into contigs for misassembly comparison) — https://github.com/SR-Martin/metacortex.git
- **GCC** (Compiler required to build MetaCortex from source with a chosen MAXK value (LLVM will not compile it))

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && cd metacortex && make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- Misassembled blocks attributed to the high-coverage genome (e.g. S.bongori sim.2) drop by approximately 50% after 10% subsampling and reassembly
- The k-mer size (MAXK) and all MetaCortex parameters are identical between the full-coverage and subsampled runs
- Both assemblies can be aligned to the same reference to produce a valid count of misassembled blocks per genome
- The comparison table (misassembly_comparison.tsv) contains counts for both coverage levels plus the computed fold-reduction

## Limitations

- The bioconda build of MetaCortex is fixed at MAXK=127, so k-mer sizes above 127 require compiling from source with make MAXK=<K>
- LLVM will not compile MetaCortex, so Mac users must install GCC and point CC at the GCC binary
- The ~50% reduction is an observation from one simulated bacterial community; fold-reduction magnitude may vary with genome and community composition

## Evidence

- [other] Compile MetaCortex from source with the same maximum k-mer used for the full-coverage run (make MAXK=<K>, where K is one of 31, 63, 95, 127, 160 or 192 nucleotides; the bioconda build is fixed at 127).: [evidence span withheld — anchor status: not_found]
- [intro] MetaCortex is executed in two stages: parallel import of reads files into CTX files, then combining CTX files to output contigs.: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file. Then secondly, to combine the separate CTX files and output"
- [other] After subsampling the reads at 10% and reassembling with MetaCortex using the same parameters, the number of misassembled blocks belonging to S.bongori sim.2 was reduced by 50%.: [evidence span withheld — anchor status: not_found]
- [intro] LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary.: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
