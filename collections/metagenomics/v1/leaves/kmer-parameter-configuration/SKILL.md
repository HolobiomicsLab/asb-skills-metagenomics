---
name: kmer-parameter-configuration
description: Use when preparing to run the MetaCortex two-stage assembly and you need
  a binary that supports specific k-mer values (e.g. MAXK of 31, 63, 95, 127, 160
  or 192) beyond the fixed bioconda build, or when reproducing an assembly with the
  same k-mer settings as a prior run.
when_to_use_negative:
- If the fixed bioconda MAXK=127 build already supports the k-mer you need — install
  via `conda install -c bioconda metacortex` instead of compiling.
- If the goal is assembly of the reads themselves rather than building/configuring
  the assembler binary — use the compiled executable in the two-stage CTX workflow,
  not the build step.
edam_operation: http://edamontology.org/operation_0525
edam_topics: []
tools:
- name: MetaCortex
  role: Assembler whose source is compiled with a chosen MAXK value to set the maximum
    supported k-mer for CTX-based de Bruijn graph assembly
  repo: https://github.com/SR-Martin/metacortex.git
- name: GCC
  role: Required compiler for building MetaCortex from source (LLVM will not compile
    it)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:2cd61bf06a7bc5594c24ce24198e524b83ebc134b6d436e6f4e671fa9d0ab6d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - GCC
  - MetaCortex
  edam_operation: http://edamontology.org/operation_0525
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# kmer-parameter-configuration

## Summary

Configure the maximum k-mer size (MAXK) when building MetaCortex from source so that the resulting binary supports the k-mer values required for a de Bruijn graph metagenomic assembly. The bioconda build is fixed at MAXK=127, so custom k-mer requirements (31–192) demand source compilation with GCC.

## When to use

Use when preparing to run the MetaCortex two-stage assembly and you need a binary that supports specific k-mer values (e.g. MAXK of 31, 63, 95, 127, 160 or 192) beyond the fixed bioconda build, or when reproducing an assembly with the same k-mer settings as a prior run.

## When NOT to use

- If the fixed bioconda MAXK=127 build already supports the k-mer you need — install via `conda install -c bioconda metacortex` instead of compiling.
- If the goal is assembly of the reads themselves rather than building/configuring the assembler binary — use the compiled executable in the two-stage CTX workflow, not the build step.

## Inputs

- MetaCortex source code from the GitHub repository (git clone https://github.com/SR-Martin/metacortex.git)
- Chosen maximum k-mer size (one of 31, 63, 95, 127, 160, 192 nucleotides)
- GCC toolchain (LLVM is not supported)

## Outputs

- Compiled MetaCortex executable named by k-mer size in the bin directory (e.g. bin/metacortex_k63)
- Binary capable of producing CTX de Bruijn graph files and assembled contigs/sequence graphs

## How to apply

Clone the MetaCortex repository (https://github.com/SR-Martin/metacortex.git) and compile from source with a chosen maximum k-mer via the MAXK make variable, e.g. `make MAXK=63 metacortex`; allowed values include 31, 63, 95, 127, 160 and 192 nucleotides. Use GCC as the compiler — LLVM will not compile MetaCortex, so Mac users must install GCC and set CC to point to the GCC binary. Run the resulting binary from the bin directory (e.g. ./bin/metacortex_k63). The rationale is that MAXK fixes the largest k the de Bruijn graph can use, so the choice must match the k used in the CTX-based two-stage assembly (parallel import of reads into CTX files, then combining CTX files into contigs) and must be reused identically across comparative assemblies (e.g. full-coverage vs subsampled runs) so misassembly counts are comparable.

## Related tools

- **MetaCortex** (Assembler whose source is compiled with a chosen MAXK value to set the maximum supported k-mer for CTX-based de Bruijn graph assembly) — https://github.com/SR-Martin/metacortex.git
- **GCC** (Required compiler for building MetaCortex from source (LLVM will not compile it))

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git; cd metacortex; make MAXK=63 metacortex; ./bin/metacortex_k63
```

## Evaluation signals

- The compiled binary appears in the bin directory with the expected k-mer-suffixed name (e.g. metacortex_k63) and runs without k-mer-size errors.
- The MAXK value matches the k-mer intended for the assembly run; re-running with the same MAXK reproduces CTX graph construction without 'k larger than MAXK' failures.
- For comparative assemblies (e.g. 10% subsampled vs full-coverage S.bongori sim.2 reads), both runs use the identical MAXK/k-mer so misassembled-block counts in misassembly_comparison.tsv are directly comparable.
- Compilation completes with GCC (not LLVM), with no compiler-related build failures.

## Limitations

- The bioconda installation of MetaCortex is fixed at MAXK=127; other k-mer maxima require compiling from source.
- LLVM cannot compile MetaCortex; Mac users must install GCC and point CC at the GCC binary.
- Once compiled, the binary is limited to k-mers up to the chosen MAXK (one of 31, 63, 95, 127, 160, 192); a different maximum requires recompiling.

## Evidence

- [intro] Build MetaCortex from source with a chosen max k-mer value: [evidence span withheld — anchor status: not_found]
- [methods] Compile source code with a chosen maximum kmer size: "make MAXK=63 metacortex"
- [methods] Run the compiled executable from the bin directory: "./bin/metacortex_k63"
- [intro] GCC required for compilation: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
