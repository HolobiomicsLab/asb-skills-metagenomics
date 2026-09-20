---
name: whole-genome-alignment-variant-comparison
description: Use when you need to quantitatively validate metagenomic strain-specific
  contigs (e.g. walked from a MetaCortex GFA sequence graph) against per-strain references
  by comparing identity and variant counts against a baseline contig.
when_to_use_negative:
- Input is a set of isolate (single-strain) assemblies with no mixed intra-species
  variation — a consensus contig already matches the reference and graph walking adds
  no value
- You only have a final contigs FASTA with no assembly graph (GFA), so strain-specific
  paths cannot be extracted for the per-strain comparison
- Strain references are unavailable, since dnadiff requires a cognate reference genome
  to compute identity and SNP/indel metrics
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0193
- http://edamontology.org/topic_3174
tools:
- name: dnadiff
  role: Performs whole-genome pairwise alignment of each strain-specific contig (and
    the original contig) to its cognate strain reference, reporting average identity
    and SNP/indel counts
- name: MetaCortex
  role: Produces the sequence graph (GFA) from trimmed HIV lab mix reads and supplies
    scripts/strain_assembly.py for per-strain BLAST-scored graph walks
  repo: https://github.com/SR-Martin/metacortex.git
- name: SRA toolkit
  role: Downloads the HIV lab mix reads (fastq-dump SRR961514) that are assembled
    and then validated
  repo: https://github.com/ncbi/sra-tools/wiki
- name: trim-galore
  role: Trims adapters/low-quality bases from the reads before assembly so the graph
    and resulting contigs are clean inputs to dnadiff
  repo: https://github.com/FelixKrueger/TrimGalore
- name: GCC
  role: Required to compile MetaCortex from source with a chosen MAXK (LLVM will not
    compile it)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad020
    title: Capturing variation in metagenomic assembly graphs with MetaCortex
  source_content_hash: sha256:243e896e050e18247e7f74eddb39951b05b4d2eb1bde6bee18a9549d0593ce5f
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0193
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# whole-genome-alignment-variant-comparison

## Summary

Validates reconstructed strain-specific contigs against cognate reference genomes using whole-genome pairwise alignment (dnadiff), quantifying average alignment identity and SNP/indel counts to demonstrate that graph-walked strain assemblies are more accurate per strain than the original consensus contig.

## When to use

Use when you need to quantitatively validate metagenomic strain-specific contigs (e.g. walked from a MetaCortex GFA sequence graph) against per-strain references by comparing identity and variant counts against a baseline contig.

## When NOT to use

- Input is a set of isolate (single-strain) assemblies with no mixed intra-species variation — a consensus contig already matches the reference and graph walking adds no value
- You only have a final contigs FASTA with no assembly graph (GFA), so strain-specific paths cannot be extracted for the per-strain comparison
- Strain references are unavailable, since dnadiff requires a cognate reference genome to compute identity and SNP/indel metrics

## Inputs

- Trimmed HIV lab mix sequencing reads (FASTQ, e.g. SRR961514)
- MetaCortex sequence graph in GFA format containing the ~9.2 kb contig and its graph elements
- Strain-specific contig FASTA files (five, one per HIV strain, from scripts/strain_assembly.py)
- Cognate HIV strain reference genome sequences (FASTA)
- Original consensus contig FASTA (~9.2 kb) as the comparison baseline

## Outputs

- dnadiff output per comparison (average alignment identity, SNP counts, indel counts)
- dnadiff_comparison_table.tsv reproducing Table 4 (identity and variant metrics for strain-specific vs original contigs)
- Validated strain-specific contig FASTA files

## How to apply

First obtain the HIV lab mix reads (e.g. SRR961514) with fastq-dump from the SRA toolkit and trim them with trim-galore before assembly. Build MetaCortex from source with an appropriate maximum k-mer size (make MAXK=63 metacortex) and run ./bin/metacortex_k63 in MC mode to emit the sequence-graph (GFA) assembly containing the ~9.2 kb consensus contig. Run scripts/strain_assembly.py to perform per-strain BLAST-scored graph walks, extracting five strain-specific contig FASTA files from the graph. Then align each strain-specific contig to its cognate strain reference with dnadiff, recording average alignment identity and SNP/indel counts, and repeat the same dnadiff comparison for the original single ~9.2 kb contig to compile a dnadiff_comparison_table.tsv reproducing Table 4. The rationale is that a consensus contig collapses intra-species diversity, so per-strain paths traversed through the de Bruijn/sequence graph should match each cognate reference more closely than the consensus.

## Related tools

- **dnadiff** (Performs whole-genome pairwise alignment of each strain-specific contig (and the original contig) to its cognate strain reference, reporting average identity and SNP/indel counts)
- **MetaCortex** (Produces the sequence graph (GFA) from trimmed HIV lab mix reads and supplies scripts/strain_assembly.py for per-strain BLAST-scored graph walks) — https://github.com/SR-Martin/metacortex.git
- **SRA toolkit** (Downloads the HIV lab mix reads (fastq-dump SRR961514) that are assembled and then validated) — https://github.com/ncbi/sra-tools/wiki
- **trim-galore** (Trims adapters/low-quality bases from the reads before assembly so the graph and resulting contigs are clean inputs to dnadiff) — https://github.com/FelixKrueger/TrimGalore
- **GCC** (Required to compile MetaCortex from source with a chosen MAXK (LLVM will not compile it))

## Examples

```
dnadiff -p strain1_cmp strain1_contig.fasta reference_strain1.fasta
```

## Evaluation signals

- All five strain-specific contig FASTA files exist after running scripts/strain_assembly.py on the MC-mode GFA graph
- dnadiff_comparison_table.tsv shows higher average alignment identity for each strain-specific contig than the original ~9.2 kb contig
- Strain-specific contigs show far fewer SNPs and indels than the original contig, matching the ordering of values in Table 4
- The original single contig is approximately 9.2 kb as expected from the MC-mode assembly of the HIV lab mix
- dnadiff completes without errors for every contig–reference pair, producing per-pair identity and variant reports

## Limitations

- LLVM will not compile MetaCortex, so Mac users must install GCC and set CC to the GCC binary before building with a chosen MAXK
- Validation depends on having cognate per-strain reference genomes; the claim of improved per-strain accuracy is only supported for strains with references (here, five HIV strains)
- The BLAST-scored graph walk selects one path per strain, so accuracy gains rely on correct per-strain branch selection in the sequence graph

## Evidence

- [abstract] MetaCortex captures intra-species diversity via the assembly graph: "MetaCortex, a metagenome assembler that captures intra-species diversity by searching for signatures of local variation along assembled sequences in the underlying assembly graph"
- [results] Strain-specific contigs outperform the original contig: [evidence span withheld — anchor status: not_found]
- [intro] Read acquisition for the HIV lab mix: "Download the reads using the fastq-dump command from the SRA toolkit"
- [methods] Compile with a chosen maximum k-mer size: "make MAXK=63 metacortex"
- [intro] Trim-galore is used before assembly: "we will use `trim-galore <https://github.com/FelixKrueger/TrimGalore>`_ to trim the reads before assembly"
