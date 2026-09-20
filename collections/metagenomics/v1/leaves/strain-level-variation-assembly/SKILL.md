---
name: strain-level-variation-assembly
description: Use when assembling metagenomic or mock viral community FASTQ reads with
  high strain-level diversity (e.g. mixed HIV-1 strains at ~2.8% abundance in a host
  background) and you need intra-species diversity preserved instead of a single consensus
  per species.
when_to_use_negative:
- Input is already assembled contigs or a pre-built graph rather than raw/trimmed
  FASTQ reads — MetaCortex imports reads files into the de Bruijn graph itself.
- Goal is a single-isolate reference assembly with no intra-species strain variation
  to capture — a consensus assembler suffices.
- Building on macOS with only LLVM available — LLVM will not compile MetaCortex, so
  the build must use GCC.
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: 'Metagenome assembler run in SW mode (k=127, delta=0.4, min coverage=25) over
    two stages: parallel CTX import then CTX combination into contigs and sequence
    graphs'
  repo: https://github.com/SR-Martin/metacortex.git
- name: SRA toolkit
  role: Provides fastq-dump to download the mock viral community reads (SRR961514)
  repo: https://github.com/ncbi/sra-tools
- name: trim-galore
  role: Adapter/quality trims the reads before de Bruijn graph construction
  repo: https://github.com/FelixKrueger/TrimGalore
- name: MetaQUAST
  role: Evaluates the assembled contigs against HIV-1 reference genomes (genome fraction,
    misassemblies)
- name: GCC
  role: Compiler required to build MetaCortex from source (make MAXK=127), as LLVM
    will not compile it
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:5ac3308e908e1c88a0be89adfff74e048d1a9755190c55275c7417956362d950
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - GCC
  - MetaCortex
  - MetaQUAST
  - SRA toolkit
  - trim-galore
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# strain-level-variation-assembly

## Summary

Assemble metagenomic (environmental) reads with MetaCortex, a de Bruijn graph assembler based on cortex_con, so that signatures of local (strain-level) variation are captured along assembled sequences and output as contigs and sequence graphs rather than collapsed into a consensus.

## When to use

Use when assembling metagenomic or mock viral community FASTQ reads with high strain-level diversity (e.g. mixed HIV-1 strains at ~2.8% abundance in a host background) and you need intra-species diversity preserved instead of a single consensus per species.

## When NOT to use

- Input is already assembled contigs or a pre-built graph rather than raw/trimmed FASTQ reads — MetaCortex imports reads files into the de Bruijn graph itself.
- Goal is a single-isolate reference assembly with no intra-species strain variation to capture — a consensus assembler suffices.
- Building on macOS with only LLVM available — LLVM will not compile MetaCortex, so the build must use GCC.

## Inputs

- FASTQ reads files (e.g. SRR961514 mock viral community reads, ERR2099157 human gut reads)
- Trimmed reads from trim-galore
- Reference genomes for evaluation (five HIV-1 strains: 89.6, HXB2, JR-CSF, NL4-3, YU-2)
- In silico mixed read sets combining viral strains into a host background at ~2.8% total abundance

## Outputs

- CTX binary de Bruijn graph files (one per reads file, stage 1)
- Assembled contigs and sequence-graph format outputs (stage 2)
- MetaQUAST metrics: genome fraction, contig count and misassembly counts (e.g. hiv_mix_assembly_stats.tsv)

## How to apply

Download the reads with fastq-dump from the SRA toolkit (e.g. SRR961514) and trim them with trim-galore before assembly. Build MetaCortex from source with a MAXK that covers your chosen k (make MAXK=127 for the k=127 benchmark) using GCC, since LLVM will not compile MetaCortex. Run MetaCortex SW mode in two stages: first import each reads file in parallel into a de Bruijn graph stored as a CTX binary file, then combine the separate CTX files into contigs and sequence graphs; for the HIV lab mix use k=127, delta=0.4 and min coverage=25. Evaluate the resulting assemblies against reference genomes (e.g. HIV-1 strains 89.6, HXB2, JR-CSF, NL4-3, YU-2) with MetaQUAST, tabulating genome fraction, contig count and misassembly counts per assembler.

## Related tools

- **MetaCortex** (Metagenome assembler run in SW mode (k=127, delta=0.4, min coverage=25) over two stages: parallel CTX import then CTX combination into contigs and sequence graphs) — https://github.com/SR-Martin/metacortex.git
- **SRA toolkit** (Provides fastq-dump to download the mock viral community reads (SRR961514)) — https://github.com/ncbi/sra-tools
- **trim-galore** (Adapter/quality trims the reads before de Bruijn graph construction) — https://github.com/FelixKrueger/TrimGalore
- **MetaQUAST** (Evaluates the assembled contigs against HIV-1 reference genomes (genome fraction, misassemblies))
- **GCC** (Compiler required to build MetaCortex from source (make MAXK=127), as LLVM will not compile it)

## Examples

```
fastq-dump SRR961514; trim_galore reads.fastq.gz; make MAXK=127 metacortex; ./bin/metacortex_k127
```

## Evaluation signals

- On the in silico HIV lab mix, MetaQUAST should report ~82.563% genome fraction across the five HIV-1 references with zero misassemblies, assembled into 31 contigs.
- After stage 1, a separate CTX file must exist for each input reads file; missing CTX files indicate parallel import failed.
- Assemblies from untrimmed reads or wrong k/delta/min-coverage settings should show lower genome fraction or more misassemblies when compared side by side with baselines in the stats table.
- The build must complete with GCC (CC pointing at GCC); a build with LLVM failing confirms the documented limitation rather than a pipeline error.

## Limitations

- LLVM will not compile MetaCortex; Mac users must install GCC and set CC to point to the GCC binary.
- Performance was demonstrated on mock viral communities with high strain-level diversity and simulated communities; results are reported per assembler and must be validated with MetaQUAST against references.
- The exact version used for the published results (Zenodo 10.5281/zenodo.7273627) should be used for reproducibility, and MAXK at compile time must be >= the k used at run time (e.g. MAXK=127 for k=127).

## Evidence

- [abstract] MetaCortex captures intra-species diversity via assembly-graph search: "MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph"
- [intro] Two-stage execution: parallel CTX import then combination into contigs: "MetaCortex is then executed in two stages: Firstly, in parallel, to import each reads file, create a de Bruijn graph and output a CTX file."
- [intro] Read acquisition with fastq-dump: "Download the reads using the fastq-dump command from the SRA toolkit"
- [methods] Build with GCC: "MetaCortex can be installed by compiling the source code with GCC"
- [intro] Pre-assembly trimming requirement: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
