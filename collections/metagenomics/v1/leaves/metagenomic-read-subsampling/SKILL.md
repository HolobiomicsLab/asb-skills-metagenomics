---
name: metagenomic-read-subsampling
description: Use when a genome in a simulated community is assembled at extreme coverage
  (e.g. >1000x) and you need to confirm that coverage — not biology — drives the observed
  misassembled blocks.
when_to_use_negative:
- The goal is strain-variation capture or sequence-graph output rather than diagnosing
  coverage-driven misassembly — use the standard MetaCortex two-stage workflow at
  full coverage instead
- Reads are experimental/environmental data (e.g. the SRR961514 tutorial reads) with
  unknown per-genome coverage, where a controlled full-vs-subsampled comparison is
  not meaningful
- Misassemblies are already attributable to causes other than coverage (e.g. strain
  divergence), so downsampling would not test the hypothesis
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: MetaCortex
  role: Performs the two-stage de Bruijn graph assembly (parallel CTX import, then
    CTX combination to contigs) on both full-coverage and 10%-subsampled reads with
    identical parameters
  repo: https://github.com/SR-Martin/metacortex
- name: GCC
  role: Compiles MetaCortex from source so MAXK can be set to the same maximum k-mer
    as the original full-coverage run; LLVM will not compile MetaCortex
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:769badfd16393bdb922d52a40511c1552ee68e87510c4d82e561a3b53bf1d3dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex
  tools:
  - GCC
  - MetaCortex
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# metagenomic-read-subsampling

## Summary

Test whether ultra-high sequencing coverage causes assembly errors by subsampling a simulated metagenomic read set (e.g. to 10% of original coverage) and reassembling with MetaCortex, then comparing misassembled-block counts per genome against the reference.

## When to use

Use when a genome in a simulated community is assembled at extreme coverage (e.g. >1000x) and you need to confirm that coverage — not biology — drives the observed misassembled blocks.

## When NOT to use

- The goal is strain-variation capture or sequence-graph output rather than diagnosing coverage-driven misassembly — use the standard MetaCortex two-stage workflow at full coverage instead
- Reads are experimental/environmental data (e.g. the SRR961514 tutorial reads) with unknown per-genome coverage, where a controlled full-vs-subsampled comparison is not meaningful
- Misassemblies are already attributable to causes other than coverage (e.g. strain divergence), so downsampling would not test the hypothesis

## Inputs

- Simulated bacterial metagenomic FASTQ read files (e.g. simulated community containing the >1000x S.bongori sim.2 genome)
- Reference genome FASTA files for misassembly assessment
- Parameters from the original full-coverage MetaCortex assembly (same MAXK / k-mer settings)

## Outputs

- Contigs from the 10%-coverage MetaCortex two-stage assembly
- Misassembled-block counts per genome for both full-coverage and subsampled assemblies
- Fold-reduction summary written to misassembly_comparison.tsv

## How to apply

Compile MetaCortex from source with the same maximum k-mer used for the full-coverage run (make MAXK=<K>, where K is one of 31, 63, 95, 127, 160 or 192 nucleotides; the bioconda build is fixed at 127), since k-mer size must be held constant for a fair coverage comparison. Subsample the simulated FASTQ reads to 10% of their original coverage, then run the MetaCortex two-stage assembly — parallel import of each reads file into a CTX de Bruijn graph file, followed by combining the CTX files to output contigs — using identical parameters to the full-coverage run. Align both assemblies against the reference genomes and count misassembled blocks per genome (here, the high-coverage S.bongori sim.2 genome). A ~50% reduction in misassembled blocks for the high-coverage genome after downsampling supports the hypothesis that ultra-high coverage caused the misassemblies.

## Related tools

- **MetaCortex** (Performs the two-stage de Bruijn graph assembly (parallel CTX import, then CTX combination to contigs) on both full-coverage and 10%-subsampled reads with identical parameters) — https://github.com/SR-Martin/metacortex
- **GCC** (Compiles MetaCortex from source so MAXK can be set to the same maximum k-mer as the original full-coverage run; LLVM will not compile MetaCortex)

## Examples

```
git clone https://github.com/SR-Martin/metacortex.git && cd metacortex && make MAXK=127 metacortex && ./bin/metacortex_k127 --kmer_size 127 --se_list subsampled_reads.txt --ctx_list ctxs.txt --out_dir subsampled_asm
```

## Evaluation signals

- misassembly_comparison.tsv contains misassembled-block counts for both the full-coverage and 10%-subsampled assemblies and the fold-reduction value
- The reported ~50% reduction in misassembled blocks for S.bongori sim.2 is reproduced after subsampling and reassembly
- The subsampled reassembly used the same MAXK and parameters as the full-coverage run (only coverage differs), so any assembly difference is attributable to coverage
- The MetaCortex binary was compiled with the same maximum k-mer (31–192 range; bioconda fixed at 127) as the original assembly

## Limitations

- The bioconda build of MetaCortex is fixed at MAXK=127, so testing other k-mer sizes requires compiling from source
- LLVM will not compile MetaCortex — Mac users must install GCC and set CC to point to the GCC binary
- A ~50% reduction is supportive but not conclusive; residual misassemblies after downsampling may reflect causes other than coverage

## Evidence

- [intro] Two-stage assembly on subsampled reads uses same CTX workflow: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file. Then secondly, to combine the separate CTX files and output"
- [intro] Compile from source with chosen MAXK: [evidence span withheld — anchor status: not_found]
- [intro] GCC required for compilation; LLVM unsupported: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [methods] run compiled binary from bin directory: "./bin/metacortex_k63"
