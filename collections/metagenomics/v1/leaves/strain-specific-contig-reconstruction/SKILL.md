---
name: strain-specific-contig-reconstruction
description: Use when a metagenomic assembly graph (GFA sequence graph from MetaCortex
  MC mode) encodes intra-species/strain-level variation and you need to extract per-strain
  paths and quantify their accuracy against strain references.
when_to_use_negative:
- The input is a finished consensus contig or standard FASTA assembly with no accompanying
  GFA sequence graph — the per-strain walk requires graph elements (branching paths)
  that only MetaCortex MC mode emits
- The community has low or no strain-level diversity, so there are no local-variation
  signatures in the graph to walk
- The goal is generic metagenomic contig assembly without strain resolution — other
  popular metagenomic assemblers may be simpler
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaCortex
  role: Assembles trimmed reads into a de Bruijn graph and outputs the sequence graph
    (GFA) whose per-strain paths are walked; source code at github.com/SR-Martin/metacortex
  repo: https://github.com/SR-Martin/metacortex.git
- name: SRA toolkit
  role: Obtains the HIV lab mix reads via the fastq-dump command
  repo: https://github.com/ncbi/sra-tools/wiki
- name: trim-galore
  role: Trims the reads before assembly
  repo: https://github.com/FelixKrueger/TrimGalore
- name: dnadiff
  role: Aligns each strain-specific contig (and the original consensus contig) to
    strain references and reports average identity and SNP/indel counts for Table
    4
- name: GCC
  role: Compiles MetaCortex from source (LLVM will not compile it); build with a chosen
    MAXK such as 63
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:fca7b72d9462022ef733830ab10b12e7b9357f382fe80f925a772b7383ede7a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SR-Martin/metacortex.git
  tools:
  - GCC
  - MetaCortex
  - SRA toolkit
  - dnadiff
  - trim-galore
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# strain-specific-contig-reconstruction

## Summary

Reconstruct strain-specific contigs by walking the MetaCortex MC sequence graph (GFA) with per-strain BLAST-scored branch selection, then validate each reconstructed contig against its cognate reference with dnadiff to show per-strain accuracy exceeds the consensus contig.

## When to use

Use when a metagenomic assembly graph (GFA sequence graph from MetaCortex MC mode) encodes intra-species/strain-level variation and you need to extract per-strain paths and quantify their accuracy against strain references.

## When NOT to use

- The input is a finished consensus contig or standard FASTA assembly with no accompanying GFA sequence graph — the per-strain walk requires graph elements (branching paths) that only MetaCortex MC mode emits
- The community has low or no strain-level diversity, so there are no local-variation signatures in the graph to walk
- The goal is generic metagenomic contig assembly without strain resolution — other popular metagenomic assemblers may be simpler

## Inputs

- Metagenomic raw reads from an SRA run (e.g. HIV lab mix, SRR961514), FASTQ format
- Per-strain reference genome sequences (five HIV strain references), FASTA
- MetaCortex sequence graph in GFA format containing the ~9.2 kb consensus contig and its graph elements
- MetaCortex CTX binary graph files from the parallel reads-import stage

## Outputs

- Five strain-specific contig FASTA files extracted by the BLAST-scored graph walk
- dnadiff comparison metrics (average identity, SNP counts, indel counts) per strain-specific contig vs. its reference
- dnadiff_comparison_table.tsv reproducing Table 4, comparing strain-specific contigs against the original consensus contig

## How to apply

Download the mixed-community reads with fastq-dump (e.g. SRR961514 from the HIV lab mix dataset) and trim them with trim-galore before assembly. Build MetaCortex from source with an appropriate maximum k-mer size (e.g. 'make MAXK=63 metacortex'; LLVM cannot compile it, use GCC) and run './bin/metacortex_kX' in MC mode so the second assembly stage (combining CTX binary files) emits contigs plus a sequence graph in GFA format containing the ~9.2 kb contig and its variant graph elements. Run scripts/strain_assembly.py to perform the per-strain BLAST-scored graph walk, selecting branches by BLAST score for each of the five HIV strains and writing each strain-specific path as a FASTA. Validate with dnadiff: align each strain-specific contig to its cognate strain reference and record average alignment identity plus SNP and indel counts, then repeat for the original single consensus contig and compile the metrics into dnadiff_comparison_table.tsv (Table 4). The rationale is that MetaCortex captures signatures of local variation in the underlying de Bruijn assembly graph, so walking the graph per strain recovers strain-resolved sequence that a consensus contig collapses.

## Related tools

- **MetaCortex** (Assembles trimmed reads into a de Bruijn graph and outputs the sequence graph (GFA) whose per-strain paths are walked; source code at github.com/SR-Martin/metacortex) — https://github.com/SR-Martin/metacortex.git
- **SRA toolkit** (Obtains the HIV lab mix reads via the fastq-dump command) — https://github.com/ncbi/sra-tools/wiki
- **trim-galore** (Trims the reads before assembly) — https://github.com/FelixKrueger/TrimGalore
- **dnadiff** (Aligns each strain-specific contig (and the original consensus contig) to strain references and reports average identity and SNP/indel counts for Table 4)
- **GCC** (Compiles MetaCortex from source (LLVM will not compile it); build with a chosen MAXK such as 63)

## Examples

```
make MAXK=63 metacortex && ./bin/metacortex_k63
```

## Evaluation signals

- The five strain-specific FASTA files each exist and correspond to a distinct HIV strain path in the sequence graph
- dnadiff reports higher average alignment identity for each strain-specific contig against its cognate reference than for the original ~9.2 kb consensus contig
- Strain-specific contigs show far fewer SNPs and indels than the original contig, matching the ordering of values in Table 4
- The assembled GFA sequence graph contains the ~9.2 kb contig and its variant graph elements emitted from the MC-mode run
- dnadiff_comparison_table.tsv reproduces Table 4 metrics (average identity, SNP and indel counts) with strain-specific rows outperforming the consensus row

## Limitations

- Requires compiling MetaCortex from source with GCC — LLVM will not compile it, so Mac users must install GCC and set CC accordingly (Conda install via bioconda is an alternative)
- The maximum k-mer size must be chosen at build time (e.g. MAXK=63) and must be large enough to resolve the strain variation; the executable is k-specific (./bin/metacortex_kX)
- Accuracy is demonstrated on a mock viral community with high strain-level diversity (HIV lab mix); results depend on variation signatures being present in the underlying assembly graph

## Evidence

- [abstract] MetaCortex captures intra-species diversity via graph signatures: "MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph"
- [intro] Two-stage assembly with GFA sequence graph output: "Then secondly, to combine the separate CTX files and output"
- [methods] Build MetaCortex from source with a chosen MAXK: "make MAXK=63 metacortex"
- [methods] Run the k-specific compiled executable: "./bin/metacortex_k63"
- [intro] GCC required to compile MetaCortex: "LLVM will not compile MetaCortex, so Mac users will also need to install GCC, and set CC to point to the GCC binary"
- [intro] fastq-dump download of the HIV lab mix reads: "Download the reads using the fastq-dump command from the SRA toolkit::

	fastq-dump SRR961514"
