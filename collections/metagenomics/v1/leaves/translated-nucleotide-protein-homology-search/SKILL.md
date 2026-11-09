---
name: translated-nucleotide-protein-homology-search
description: Use when the query data are unassembled or fragmented nucleotide sequences
  (contigs, raw reads) and homologs must be found against protein reference sequences,
  tolerating frameshifts in fragmented assemblies.
when_to_use_negative:
- The query is already protein sequence, so translated search (BLASTX) is unnecessary
  — use a protein-protein search instead
- High-quality annotated gene models are already available for the query genome and
  a gene annotation/annotation-transfer tool (e.g., GeMoMa) would be more appropriate
- The target references are not near-universal single-copy orthologs; the article
  recommends limiting reference sequences to near-USCOs to avoid chimeric sequences
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3293
tools:
- name: DIAMOND
  role: Sequence aligner executing the BLASTX translated nucleotide-vs-protein search
    with --ultra-sensitive --frameshift 15 --masking 0
  repo: https://github.com/bbuchfink/diamond
- name: Patchwork
  role: Pipeline wrapper that builds the DIAMOND database, runs the search, and performs
    hit stitching, masking, and trimming of the hits
  repo: https://github.com/fethalen/Patchwork
- name: Julia
  role: Programming language in which Patchwork (and its use of BioAlignments.jl/BioSequences.jl
    for stitching and alignment handling) is implemented
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:f486aaba8f70a126160d710a4575ec74b8e7a6788f1847a9139b1317459ec9fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bbuchfink/diamond
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# translated-nucleotide-protein-homology-search

## Summary

Perform a DIAMOND BLASTX search of translated nucleotide query contigs (or reads) against amino acid reference sequences to recover homologous phylogenetic markers, as implemented in the Patchwork pipeline for WGS/genome-skimming data.

## When to use

Use when the query data are unassembled or fragmented nucleotide sequences (contigs, raw reads) and homologs must be found against protein reference sequences, tolerating frameshifts in fragmented assemblies.

## When NOT to use

- The query is already protein sequence, so translated search (BLASTX) is unnecessary — use a protein-protein search instead
- High-quality annotated gene models are already available for the query genome and a gene annotation/annotation-transfer tool (e.g., GeMoMa) would be more appropriate
- The target references are not near-universal single-copy orthologs; the article recommends limiting reference sequences to near-USCOs to avoid chimeric sequences

## Inputs

- Nucleotide query sequences (assembled contigs, e.g., SPAdes v3.15.3 K-mer 55 assembly, or raw WGS reads) in FASTA/FASTQ
- Amino acid reference sequences (e.g., 815 USCO protein FASTA from GCA_904063045.1)
- DIAMOND database built from the pooled reference protein FASTA

## Outputs

- DIAMOND BLASTX alignment hits (HSPs) between translated query contigs and reference proteins
- Per-marker stitched query sequences feeding into Patchwork's masking, trimming, concatenation, and accuracy/completeness assessment

## How to apply

Pool the amino acid reference sequences (e.g., the 815 D. gyrociliatus USCOs from GenBank GCA_904063045.1) into a single FASTA and build a DIAMOND database. Run DIAMOND BLASTX with the nucleotide query (SPAdes-assembled contigs, e.g., K-mer 55, or raw reads) using --ultra-sensitive --frameshift 15 --masking 0, keeping default BLOSUM62 scoring (gap open 11, gap extension 1). Disabling tantan masking (--masking 0) is deliberate because query coverage improves especially for read data when masking is disabled. The --frameshift 15 option allows alignment across frameshift errors typical of short-read assemblies. Downstream, Patchwork performs hit stitching that retains only the highest-scoring segment pair (HSP) per region before masking, trimming, and concatenation.

## Related tools

- **DIAMOND** (Sequence aligner executing the BLASTX translated nucleotide-vs-protein search with --ultra-sensitive --frameshift 15 --masking 0) — https://github.com/bbuchfink/diamond
- **Patchwork** (Pipeline wrapper that builds the DIAMOND database, runs the search, and performs hit stitching, masking, and trimming of the hits) — https://github.com/fethalen/Patchwork
- **Julia** (Programming language in which Patchwork (and its use of BioAlignments.jl/BioSequences.jl for stitching and alignment handling) is implemented) — https://github.com/fethalen/Patchwork

## Examples

```
diamond blastx -d usco_refs.dmnd -q spades_k55_contigs.fasta --ultra-sensitive --frameshift 15 --masking 0 -o hits.tab
```

## Evaluation signals

- Benchmark reproduction: against 815 same-species USCO references, all 815 markers should be retrieved
- Mean identical aligned positions of 95.9% across recovered markers
- Mean query coverage of 92.2% (and 88.4% identical matches over all reference positions)
- Most markers should fall well above the 90% aligned-position trimming threshold
- Stitched output should contain one continuous sequence per reference with only the highest-scoring segment pair retained per region

## Limitations

- Coverage matters: below ~10x coverage total percent identity drops (only ~62% for >=10x assemblies), and at 1x coverage read-based data may perform better than assembly data
- Retrieval degrades for distant references: only 943 of 957 H. robusta references were retrieved, with 89% average identity and 74.5% mean query coverage
- Non-near-USCO references risk producing chimeric sequences; noncoding stretches may need post-hoc removal (e.g., PREQUAL)

## Evidence

- [methods] DIAMOND BLASTX with ultra-sensitive mode and frameshift handling on fragmented assemblies: [evidence span withheld — anchor status: not_found]
- [results] Masking disabled improves read coverage: "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [abstract] Patchwork uses DIAMOND for homologous region retrieval: "Patchwork is an alignment-based tool that utilizes the sequence aligner DIAMOND"
- [methods] Hit stitching retains best HSP per region: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [results] Same-species benchmark accuracy metrics: "We retrieved all of the initial 815 markers. On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
- [readme] Patchwork works with contigs or raw reads against protein references: "The program searches the provided DNA query contigs against one or more amino acid reference sequences. Multiple, overlapping hits are merged to derive a single, continuous sequence"
