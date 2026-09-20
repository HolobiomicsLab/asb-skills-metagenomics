---
name: sequence-hit-stitching-and-region-concatenation
description: Use when a DIAMOND BLASTX search of fragmented short-read assembly contigs
  (or raw reads) against amino acid reference sequences produces multiple scattered
  hits per marker that must be merged into one continuous sequence for phylogenomic
  retrieval.
when_to_use_negative:
- When the query is a single complete, well-annotated genome where markers can be
  extracted directly with annotation tools like BUSCO — stitching is designed for
  fragmented short-read assemblies or reads
- When references are not (near-)single-copy orthologs — the article warns that chimeric
  sequences can arise and recommends limiting reference sequences to near-USCOs
- When a de novo gene predictor or read-to-tree tool like Read2Tree is preferred and
  no reference-guided stitching is desired
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3293
tools:
- name: Patchwork
  role: Implements hit stitching (HSP-per-region retention), alignment masking, sliding-window
    trimming, and concatenation/realignment of remaining regions
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Provides the BLASTX translated search whose HSPs feed the stitching phase;
    run with --ultra-sensitive --frameshift 15 --masking 0
  repo: https://github.com/bbuchfink/diamond
- name: BioAlignments.jl
  role: Julia library providing the alignment algorithms and data structures used
    inside Patchwork
  repo: https://github.com/BioJulia/BioAlignments.jl
- name: BioSequences.jl
  role: Julia library providing DNA/amino-acid sequence data types used to manipulate
    stitched regions
  repo: https://github.com/BioJulia/BioSequences.jl
- name: Julia
  role: Implementation language of Patchwork, chosen for speed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:1bcb751ffcac997ee64d9e770a679666f751366fe95ebca385e1395b420e6cb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - BioAlignments.jl
  - BioSequences.jl
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Sequence hit stitching and region concatenation

## Summary

Merges multiple overlapping or gappy alignment hits (HSPs) from a translated nucleotide-to-protein similarity search into a single continuous query sequence per reference, then trims noncoding stretches and concatenates/realigns remaining regions to recover full-length phylogenetic markers from fragmented WGS data.

## When to use

Use when a DIAMOND BLASTX search of fragmented short-read assembly contigs (or raw reads) against amino acid reference sequences produces multiple scattered hits per marker that must be merged into one continuous sequence for phylogenomic retrieval.

## When NOT to use

- When the query is a single complete, well-annotated genome where markers can be extracted directly with annotation tools like BUSCO — stitching is designed for fragmented short-read assemblies or reads
- When references are not (near-)single-copy orthologs — the article warns that chimeric sequences can arise and recommends limiting reference sequences to near-USCOs
- When a de novo gene predictor or read-to-tree tool like Read2Tree is preferred and no reference-guided stitching is desired

## Inputs

- DIAMOND BLASTX alignment hits of translated nucleotide query contigs against pooled amino acid reference protein FASTA (e.g. 815 D. gyrociliatus USCO reference sequences)
- Fragmented de novo query assembly (e.g. SPAdes v3.15.3, K=55, from Illumina short reads) or raw WGS reads
- Reference amino acid sequences of phylogenetic markers (e.g. USCOs from OrthoDB-derived genomes such as GenBank GCA_904063045.1)

## Outputs

- One stitched, continuous query sequence per reference marker with unaligned residues, ambiguous amino acids, and stop codons removed
- Trimmed, concatenated and realigned marker alignments suitable for phylogenetic matrix assembly
- Accuracy/completeness statistics (percent identical aligned positions, query coverage, combined (pidentical, cov) measure)

## How to apply

After running DIAMOND BLASTX (e.g. with --ultra-sensitive --frameshift 15 --masking 0, default BLOSUM62, gap open 11, gap extension 1) against a pooled reference protein FASTA, feed the hits to Patchwork's hit-stitching phase: merge adjacent or overlapping alignments such that only the highest-scoring segment pair (HSP) per region is retained, resolving duplicates and overlaps against the reference coordinates. Next apply alignment masking to remove unaligned residues, ambiguous amino acid characters, and stop codons from the resulting query sequence. Then use the sliding-window-based alignment trimming algorithm to cut away noncoding stretches that survive in the stitched sequence. Finally, concatenate the ordered, nonoverlapping remaining regions into one continuous sequence and realign it to the reference. The pipeline is implemented in Julia (BioAlignments.jl/BioSequences.jl) within Patchwork v0.5.1 and is intended specifically for retrieving single-copy markers like OrthoDB USCOs from fragmented LC-WGS assemblies.

## Related tools

- **Patchwork** (Implements hit stitching (HSP-per-region retention), alignment masking, sliding-window trimming, and concatenation/realignment of remaining regions) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Provides the BLASTX translated search whose HSPs feed the stitching phase; run with --ultra-sensitive --frameshift 15 --masking 0) — https://github.com/bbuchfink/diamond
- **BioAlignments.jl** (Julia library providing the alignment algorithms and data structures used inside Patchwork) — https://github.com/BioJulia/BioAlignments.jl
- **BioSequences.jl** (Julia library providing DNA/amino-acid sequence data types used to manipulate stitched regions) — https://github.com/BioJulia/BioSequences.jl
- **Julia** (Implementation language of Patchwork, chosen for speed)

## Examples

```
diamond blastx -d usco_refs.dmnd -q spades_k55_contigs.fasta --ultra-sensitive --frameshift 15 --masking 0 && julia patchwork.jl stitch-and-concatenate hits.tsv
```

## Evaluation signals

- Benchmark invariant: with a high-quality D. gyrociliatus assembly and 815 same-species USCO references, all 815 markers should be retrieved with ~95.9% mean identical aligned positions, ~92.2% mean query coverage, and 88.4% identical matches over all reference positions
- Each reference marker should map to exactly one stitched continuous sequence — no duplicated or overlapping regions remain after concatenation of ordered nonoverlapping regions
- Stitched sequences should be free of unaligned residues, ambiguous amino acid characters, and stop codons (all removed by the masking step)
- Most recovered markers should sit above the 90% aligned-position trimming threshold
- For distant references (H. robusta near-USCOs), expect ~943/957 retrieved with 769 aligning uniquely back to a single D. gyrociliatus USCO — multiple or ambiguous back-alignment indicates stitching failure

## Limitations

- Noncoding stretches can persist in stitched sequences; a sliding-window trimming algorithm removes them, and PREQUAL is suggested as a downstream check, but residual artifacts are possible
- Using references that are not near-USCOs risks chimeric sequences; the authors recommend limiting references to near-single-copy orthologs
- At very low coverage (e.g. 1x), stitched marker recovery degrades; read-based retrieval may actually outperform assembly-based stitching at 1x coverage
- No specific cutoff threshold values are given for the identity-based trimming step — choosing a cutoff is easiest only when identity approaches 100%

## Evidence

- [methods] hit stitching retains HSP per region: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [methods] masking removes bad residues: "unaligned residues, ambiguous amino acid characters, and stop codons (also known as “termination codons”) are all removed from the resulting query sequence."
- [methods] sliding window trimming: "we have implemented a sliding window–based alignment trimming approach to rid the alignments from these unwanted regions"
- [methods] concatenation of regions: "the resulting set of ordered, nonoverlapping sequence regions are concatenated into 1 continuous sequence"
- [abstract] abstract stitching description: "followed by a "hit stitching" phase, in which adjacent or overlapping regions are merged into a single unit"
- [readme] README feature statement: "The program searches the provided DNA query contigs against one or more amino acid reference sequences. Multiple, overlapping hits are merged to derive a single, continuous sequence for each"
