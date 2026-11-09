---
name: assembly-misassembly-quantification
description: Use when a metagenomic assembly attributes many misassembled blocks to
  a genome sequenced at ultra-high coverage (>1000x) and you need to test whether
  excess coverage, not biology, caused the errors.
when_to_use_negative:
- The assembly error is suspected to arise from strain diversity or graph variation
  rather than excess read depth — MetaCortex's variation-capturing mode is the intended
  path there, not coverage subsampling
- No reference genome is available for the genome in question, so misassembled blocks
  cannot be counted against a reference
- Input reads are already low-coverage, so a 10% subsample would not test a coverage-driven
  hypothesis
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: MetaCortex
  role: Two-stage de Bruijn graph assembler used to reassemble the 10%-subsampled
    reads under identical parameters to the full-coverage run
  repo: https://github.com/SR-Martin/metacortex
- name: GCC
  role: Compiler required to build MetaCortex from source with a chosen MAXK value
    (LLVM will not compile it)
- name: trim-galore
  role: Optional adapter trimming of reads before assembly
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:51c52fce98c5b9a04dfdbe2318503d73b8b78928a60a0395637a00f173d8dde0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex
  tools:
  - GCC
  - MetaCortex
  - trim-galore
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# assembly-misassembly-quantification

## Summary

Diagnoses coverage-driven assembly errors by subsampling a metagenomic read set to 10% of its original coverage, reassembling with MetaCortex under identical parameters, and comparing misassembled-block counts against a reference between the full-coverage and subsampled assemblies.

## When to use

Use when a metagenomic assembly attributes many misassembled blocks to a genome sequenced at ultra-high coverage (>1000x) and you need to test whether excess coverage, not biology, caused the errors.

## When NOT to use

- The assembly error is suspected to arise from strain diversity or graph variation rather than excess read depth — MetaCortex's variation-capturing mode is the intended path there, not coverage subsampling
- No reference genome is available for the genome in question, so misassembled blocks cannot be counted against a reference
- Input reads are already low-coverage, so a 10% subsample would not test a coverage-driven hypothesis

## Inputs

- Simulated or real metagenomic reads in FASTQ (e.g. simulated bacterial community with S.bongori sim.2 at >1000x coverage)
- Reference genome(s) in FASTA for misassembly evaluation
- Full-coverage MetaCortex assembly output (contigs) for baseline misassembled-block counts
- MetaCortex source code from github.com/SR-Martin/metacortex (or the Zenodo version 10.5281/zenodo.7273627)

## Outputs

- 10%-subsampled FASTQ read set
- CTX de Bruijn graph files from the parallel import stage
- Contigs from the subsampled MetaCortex assembly
- misassembly_comparison.tsv with misassembled-block counts at both coverage levels and the fold-reduction

## How to apply

Compile MetaCortex from source with the same maximum k-mer used for the full-coverage run (`make MAXK=<K>` with K in {31, 63, 95, 127, 160, 192}; the bioconda build is fixed at 127), since LLVM will not compile it and GCC must be set as CC. Subsample the simulated bacterial FASTQ reads to 10% of original coverage. Rerun the MetaCortex two-stage assembly — parallel import of each reads file into a CTX de Bruijn graph file, then combining the CTX files to output contigs — using exactly the same parameters as the full-coverage run. Finally, compare both the full-coverage and subsampled assemblies against the reference to count misassembled blocks per genome, focusing on the high-coverage S.bongori sim.2 genome, and write both counts plus the fold-reduction to misassembly_comparison.tsv. The rationale is that if ultra-high coverage caused the misassemblies, reducing coverage should reduce misassembled blocks (the reported result was a 50% reduction).

## Related tools

- **MetaCortex** (Two-stage de Bruijn graph assembler used to reassemble the 10%-subsampled reads under identical parameters to the full-coverage run) — https://github.com/SR-Martin/metacortex
- **GCC** (Compiler required to build MetaCortex from source with a chosen MAXK value (LLVM will not compile it))
- **trim-galore** (Optional adapter trimming of reads before assembly) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
make MAXK=127 metacortex && ./bin/metacortex_k127 --se subsampled_reads.fastq --output ctx_dir --kmers 63 --sample S.bongori_sub10
```

## Evaluation signals

- misassembly_comparison.tsv contains misassembled-block counts for both the full-coverage and 10%-subsampled assemblies plus the fold-reduction
- The reported ~50% reduction in misassembled blocks for S.bongori sim.2 is reproduced on the subsampled assembly
- Both runs used the same MAXK (one of 31, 63, 95, 127, 160, 192) and identical assembly parameters, so the only variable is coverage
- Stage-1 outputs one CTX file per reads file that are successfully combined in stage 2 into contigs
- Misassembled-block counts are attributed per genome against the reference, with the high-coverage genome (>1000x) as the focal comparison

## Limitations

- The bioconda build is fixed at max k-mer 127, so compiling from source is required to match a MAXK other than 127 between runs
- LLVM will not compile MetaCortex; Mac users must install GCC and set CC to point to the GCC binary
- The finding is specific to ultra-high coverage (>1000x) simulated data; the 50% reduction may not transfer to other coverage regimes or real communities

## Evidence

- [intro] Two-stage assembly: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file."
- [intro] CTX parallelisation rationale: "implements an intermediate binary file format which enables the parallelisation of the process of converting raw reads into a de Bruijn graph structure"
- [intro] Compiler requirement: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [methods] Source installation: "MetaCortex can be installed by compiling the source code with GCC"
- [methods] Build invocation: "make MAXK=63 metacortex"
