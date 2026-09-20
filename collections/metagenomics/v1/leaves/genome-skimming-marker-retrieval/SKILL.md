---
name: genome-skimming-marker-retrieval
description: Use when you need to mine single-copy nuclear markers (e.g., OrthoDB
  USCOs) for phylogenomics from genome-skimming (LC-WGS) data provided as Illumina
  short reads or de novo contigs, especially at coverages of 1x–40x.
when_to_use_negative:
- Your query data is already high-quality, unfragmented annotated genomic sequence
  with full-length genes — assembly-free marker mining of skimming data is unnecessary
- You need trees inferred directly from unassembled reads without marker retrieval
  — a tool like Read2Tree targets that workflow instead
- Your references are not universal single-copy orthologs; the authors recommend limiting
  references to near-USCOs to avoid chimeric sequences
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0622
tools:
- name: Patchwork
  role: 'Primary tool: alignment-based marker retrieval, hit stitching, sliding-window
    trimming, and concatenation from WGS reads or contigs'
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Sequence aligner used by Patchwork for BLASTX similarity search of nucleotide
    query against amino acid references (with --ultra-sensitive --frameshift 15 --masking
    0)
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Implementation language / runtime in which Patchwork is executed
  repo: https://julialang.org/
- name: SPAdes
  role: De novo assembler producing short-read-only contigs used as query when the
    assembly-based Patchwork mode is benchmarked
- name: Subsample.jl
  role: Randomly selects corresponding read pairs to generate coverage-graded data
    sets (1x, 3x, 5x, 10x, 20x, 40x) from paired-end reads
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:07309fd9fdccae75186623308e3ce60ee852748b1d5321db3d646d0f9cbaf439
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - ALiBaSeq
  - BLAST
  - DIAMOND
  - Julia
  - Patchwork
  - QUAST
  - SPAdes
  - Slurm
  - Subsample.jl
  - aTRAM 2
  - exonerate
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# genome-skimming-marker-retrieval

## Summary

Retrieval and concatenation of phylogenetic markers (universal single-copy orthologs, USCOs) from low-coverage whole-genome sequencing data — either fragmented SPAdes short-read assemblies or raw reads — using the Julia/DIAMOND alignment-based tool Patchwork, which stitches overlapping DIAMOND hits, trims noncoding regions via a sliding window, and concatenates remaining regions per reference sequence.

## When to use

Use when you need to mine single-copy nuclear markers (e.g., OrthoDB USCOs) for phylogenomics from genome-skimming (LC-WGS) data provided as Illumina short reads or de novo contigs, especially at coverages of 1x–40x.

## When NOT to use

- Your query data is already high-quality, unfragmented annotated genomic sequence with full-length genes — assembly-free marker mining of skimming data is unnecessary
- You need trees inferred directly from unassembled reads without marker retrieval — a tool like Read2Tree targets that workflow instead
- Your references are not universal single-copy orthologs; the authors recommend limiting references to near-USCOs to avoid chimeric sequences

## Inputs

- Illumina paired-end short reads in FASTQ (e.g., D. gyrociliatus reads, ENA accession PRJEB37657; SRR5088465, SRR5088468, SRR5308129, SRR5308123, SRR5088469, SRR5088471)
- De novo short-read assemblies (SPAdes, e.g., K-mer size 33 or 55)
- Amino acid reference marker sequences, ideally near-USCOs (e.g., 815 D. gyrociliatus USCOs from GCA_904063045.1; 957 H. robusta near-USCOs from GCA_000326865.1)

## Outputs

- One concatenated, continuous marker sequence per reference sequence, recovered from the query data
- Per-marker alignment statistics: percent identity (fraction of identical aligned positions) and query coverage against the reference
- Joint accuracy metric pidentical,cov (percent identical aligned positions times reference coverage) when evaluated against a recoverable USCO set via DIAMOND --ultra-sensitive

## How to apply

Run Patchwork (v0.5.1 under Julia 1.8.2) in a single one-step command, supplying the DNA query (raw paired-end reads or assembled contigs) against amino acid reference sequences (e.g., 815 D. gyrociliatus USCOs from GenBank GCA_904063045.1, or 957 H. robusta near-USCOs from GCA_000326865.1) with --ultra-sensitive --frameshift 15 --masking 0 under default BLOSUM62 scoring (gap open 11, gap extension 1); DIAMOND BLASTX finds homologous regions, then 'hit stitching' merges adjacent/overlapping hits keeping only the highest-scoring segment pair (HSP) per region, unaligned residues/ambiguous amino acids/stop codons are removed, a sliding window trims noncoding stretches, and remaining ordered regions are concatenated and realigned. Use reference sequences restricted to near-USCOs to avoid chimeric output, and keep tantan masking disabled (--masking 0) since it improves query coverage, especially for read-based input. Assemblies are generated from subsampled reads with SPAdes (e.g., K-mer size 33 or 55).

## Related tools

- **Patchwork** (Primary tool: alignment-based marker retrieval, hit stitching, sliding-window trimming, and concatenation from WGS reads or contigs) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Sequence aligner used by Patchwork for BLASTX similarity search of nucleotide query against amino acid references (with --ultra-sensitive --frameshift 15 --masking 0)) — https://github.com/bbuchfink/diamond
- **Julia** (Implementation language / runtime in which Patchwork is executed) — https://julialang.org/
- **SPAdes** (De novo assembler producing short-read-only contigs used as query when the assembly-based Patchwork mode is benchmarked)
- **Subsample.jl** (Randomly selects corresponding read pairs to generate coverage-graded data sets (1x, 3x, 5x, 10x, 20x, 40x) from paired-end reads)

## Examples

```
julia patchwork.jl query_contigs.fasta usco_refs.fasta --ultra-sensitive --frameshift 15 --masking 0
```

## Evaluation signals

- Percent identity of aligned positions: with a high-quality query assembly and 815 same-species USCOs, all 815 markers should be retrieved with ~95.9% mean identical aligned positions and ~92.2% mean query coverage
- Cross-species retrieval: against distant H. robusta references, ~943/957 sequences should be recovered and ~769 should align back uniquely to a single D. gyrociliatus USCO out of the 778 recoverable set
- Coverage gradient: total percent identity (pidentical,cov) for assembly-based runs should reach up to 62% at >=10x coverage, outperforming ALiBaSeq (~7% lower) and aTRAM 2 (max ~22%)
- Most markers should fall well above the 90% aligned-position trimming threshold
- The --masking 0 run must show higher query coverage than the tantan-masked run, with only a slight decrease in percent identity

## Limitations

- At only 1x coverage, read-based query data outperformed assembly-based input, while assemblies performed best at higher coverages — mode choice depends on depth
- aTRAM 2 comparison shows heavy runtime costs on high-coverage read data (40x did not complete within 5 days), a caution for read-based retrieval workflows
- Limit reference sequences to near-USCOs to avoid chimeric sequences in output
- Artifactual noncoding stretches may remain; PREQUAL is recommended downstream to detect and remove such regions

## Evidence

- [abstract] Homologous regions are obtained via a sequence similarity search, followed by a "hit stitching" phase, in which adjacent or overlapping regions are merged into a single unit: "Homologous regions are obtained via a sequence similarity search, followed by a "hit stitching" phase, in which adjacent or overlapping regions are merged into a single unit"
- [results] query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument): "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [results] most markers were well above the 90% aligned position trimming threshold: "most markers were well above the 90% aligned position trimming threshold"
- [readme] Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data.: "Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data."
- [discussion] we recommend that the user limits the use of reference sequences to near-USCOs: "we recommend that the user limits the use of reference sequences to near-USCOs"
