---
name: protein-guided-nucleotide-alignment-execution
description: Use when nucleotide query sequences (SPAdes contigs or raw short reads)
  must be searched against protein reference sequences (e.g., 957 H. robusta near-USCOs)
  to recover homologous phylogenetic markers from fragmented genome-skimming data.
when_to_use_negative:
- The references are not near-USCOs — the article recommends limiting reference sequences
  to near-USCOs to avoid chimeric sequences
- The query is already a high-quality annotated protein set rather than fragmented
  nucleotide WGS data, so translated alignment adds no value
- Coding-region artifacts are expected in output — use PREQUAL to detect and remove
  artifactual noncoding stretches before trusting alignments
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: Patchwork
  role: Wraps the DIAMOND BLASTX search, performs hit stitching, alignment masking,
    sliding-window trimming, and concatenation of retrieved markers
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Sequence aligner executing the translated nucleotide-vs-protein similarity
    search (BLASTX) with --ultra-sensitive, --frameshift 15, --masking 0
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Implementation language of Patchwork; BioSequences.jl and BioAlignments.jl
    provide sequence and alignment data structures
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:cfc8a3d202799f486a3dd7e6a4fe76896254ca55fb6de302b2e7cae1c8b553e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# protein-guided-nucleotide-alignment-execution

## Summary

Executes translated nucleotide-to-protein similarity searches (DIAMOND BLASTX) so that DNA query contigs or reads from fragmented assemblies can be aligned against amino acid reference markers, as implemented in Patchwork for retrieving phylogenetic markers from WGS data.

## When to use

Use when nucleotide query sequences (SPAdes contigs or raw short reads) must be searched against protein reference sequences (e.g., 957 H. robusta near-USCOs) to recover homologous phylogenetic markers from fragmented genome-skimming data.

## When NOT to use

- The references are not near-USCOs — the article recommends limiting reference sequences to near-USCOs to avoid chimeric sequences
- The query is already a high-quality annotated protein set rather than fragmented nucleotide WGS data, so translated alignment adds no value
- Coding-region artifacts are expected in output — use PREQUAL to detect and remove artifactual noncoding stretches before trusting alignments

## Inputs

- DNA query contigs (e.g., SPAdes de novo assembly, K-mer size 55, of D. gyrociliatus) or raw Illumina short reads (e.g., PRJEB37657 / SRR5088465)
- Amino acid reference sequences (e.g., 957 H. robusta near-USCOs from GenBank GCA_000326865.1), as a DIAMOND-formatted database
- Optionally, a 'recoverable' reference USCO set for evaluation (e.g., 778 D. gyrociliatus USCOs from GCA_904063045.1)

## Outputs

- Retrieved marker sequences stitched against each reference protein
- Per-marker alignment metrics (percent identical aligned positions, query coverage)
- Benchmark metrics table (e.g., patchwork_h_robusta_benchmark_metrics.tsv) with combined metric pidentical,cov = nmatch/naligned x cov

## How to apply

Build a DIAMOND-formatted database from the protein reference sequences (e.g., 957 H. robusta near-USCOs from GenBank GCA_000326865.1) and run DIAMOND's BLASTX algorithm with the default BLOSUM62 matrix, gap open penalty 11, and gap extension penalty 1, using Patchwork v.0.5.1 (Julia v.1.8.2, DIAMOND v.2.0.13). For highly divergent query-reference pairs, run with the benchmark options --ultra-sensitive --frameshift 15 and disable tantan masking via --masking 0, which especially improves query coverage for read data. Overlapping or gappy hits are then merged ('hit stitching') so only the highest-scoring segment pair (HSP) per region is retained, and unaligned residues, ambiguous amino acids, and stop codons are removed before a sliding-window trimming step trims noncoding regions. Restricting references to near-USCOs and evaluating against 'recoverable' USCOs (e.g., 778 D. gyrociliatus USCOs that produce a hit under DIAMOND --ultra-sensitive) gives a defensible accuracy readout.

## Related tools

- **Patchwork** (Wraps the DIAMOND BLASTX search, performs hit stitching, alignment masking, sliding-window trimming, and concatenation of retrieved markers) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Sequence aligner executing the translated nucleotide-vs-protein similarity search (BLASTX) with --ultra-sensitive, --frameshift 15, --masking 0) — https://github.com/bbuchfink/diamond
- **Julia** (Implementation language of Patchwork; BioSequences.jl and BioAlignments.jl provide sequence and alignment data structures) — https://github.com/fethalen/Patchwork

## Examples

```
patchwork -q dyro_spades_k55.fasta -r h_robusta_near_usco.fasta --outfmt fasta --ultra-sensitive --frameshift 15 --masking 0
```

## Evaluation signals

- Retrieval count matches the reported benchmark: 943 of 957 H. robusta reference sequences retrieved
- 769 of the recovered markers align back to exactly 1 of the 778 D. gyrociliatus USCOs considered homologous
- Mean percent identical aligned positions is approximately 89% and mean query coverage approximately 74.5% for the divergent reference benchmark
- Combined metric pidentical,cov ≈ 67.2% (identical matches over all reference positions) is reproduced in the metrics table
- No unaligned residues, ambiguous amino acid characters, or stop codons remain in the masked output sequences

## Limitations

- Retrieval against highly divergent references is imperfect: 14 of 957 references were not retrieved and 174 of 778 homologous USCOs lacked a unique back-alignment
- Tantan masking must be disabled (--masking 0) to improve query coverage, particularly for read data
- Noncoding stretches and stop codons can contaminate stitched sequences; the sliding-window trimming and masking steps, or downstream PREQUAL, are needed
- Chimeric sequences can result if references are not restricted to near-USCOs

## Evidence

- [readme] searches the provided DNA query contigs against one or more amino acid reference sequences. Multiple, overlapping hits are merged to derive a single, continuous sequence for each reference sequence.: "searches the provided DNA query contigs against one or more amino acid reference sequences. Multiple, overlapping hits are merged"
- [methods] Build a DIAMOND-formatted database from the 957 H. robusta near-USCO protein sequences (GenBank GCA_000326865.1), which Patchwork then searches with DIAMOND's BLASTX algorithm using the default BLOSUM62 matrix, gap open penalty 11, and gap extension penalty 1.: [evidence span withheld — anchor status: not_found]
- [methods] Run Patchwork v.0.5.1 (Julia v.1.8.2, DIAMOND v.2.0.13) with the D. gyrociliatus SPAdes de novo assembly (K-mer size 55) as query, using the benchmark options --ultra-sensitive --frameshift 15 --masking 0.: [evidence span withheld — anchor status: not_found]
- [results] query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument): "query coverage improves especially for read data when tantan masking in DIAMOND is disabled"
- [discussion] we recommend that the user limits the use of reference sequences to near-USCOs: "we recommend that the user limits the use of reference sequences to near-USCOs"
