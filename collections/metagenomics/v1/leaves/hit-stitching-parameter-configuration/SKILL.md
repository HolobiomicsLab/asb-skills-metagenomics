---
name: hit-stitching-parameter-configuration
description: Use when retrieving phylogenetic markers (e.g. USCOs/near-USCOs) from
  fragmented WGS assemblies or reads with Patchwork, and DIAMOND search sensitivity,
  frameshift, or tantan masking parameters must be set to maximize retrieval and coverage.
when_to_use_negative:
- Reference sequences are not near-USCOs — the article recommends limiting references
  to near-USCOs to avoid chimeric sequences
- The query is already high-quality annotated sequence rather than fragmented WGS
  assembly or reads, where marker retrieval from WGS is unnecessary
- Tantan masking removal is undesirable because target sequences contain low-complexity
  regions that would produce spurious hits
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: Patchwork
  role: Implements hit stitching (merging adjacent/overlapping DIAMOND HSPs), alignment
    masking, sliding-window trimming, and concatenation of retrieved markers
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Performs the BLASTX sequence similarity search (BLOSUM62, gap open 11, extend
    1) whose hits Patchwork stitches; benchmark flags --ultra-sensitive --frameshift
    15 --masking 0
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Programming language in which Patchwork is written and executed
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:5cdfc0dd8e053afcd9d4700e468ee60fadb21fbd59fd388bd00df3e4fd34bea0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hit-stitching-parameter-configuration

## Summary

Configure Patchwork's hit-stitching phase, in which adjacent or overlapping DIAMOND alignments to a protein reference are merged into one continuous sequence, and tune the benchmark search options (--ultra-sensitive --frameshift 15 --masking 0) so that homologous markers can be recovered even between highly divergent species.

## When to use

Use when retrieving phylogenetic markers (e.g. USCOs/near-USCOs) from fragmented WGS assemblies or reads with Patchwork, and DIAMOND search sensitivity, frameshift, or tantan masking parameters must be set to maximize retrieval and coverage.

## When NOT to use

- Reference sequences are not near-USCOs — the article recommends limiting references to near-USCOs to avoid chimeric sequences
- The query is already high-quality annotated sequence rather than fragmented WGS assembly or reads, where marker retrieval from WGS is unnecessary
- Tantan masking removal is undesirable because target sequences contain low-complexity regions that would produce spurious hits

## Inputs

- DNA query contigs from a de novo assembly (e.g. SPAdes K-mer-55 assembly of D. gyrociliatus) or raw WGS reads
- DIAMOND-formatted database of amino acid reference sequences (e.g. 957 H. robusta near-USCO protein sequences from GenBank GCA_000326865.1)
- Optional set of 'recoverable' homologous USCO references for evaluation (e.g. 778 D. gyrociliatus USCOs from GCA_904063045.1)

## Outputs

- One stitched, trimmed, and concatenated query sequence per reference sequence
- Per-marker alignment statistics (percent identical aligned positions, query coverage) aggregated into a metrics table such as patchwork_h_robusta_benchmark_metrics.tsv

## How to apply

First run Patchwork's underlying DIAMOND BLASTX search (default BLOSUM62 matrix, gap open penalty 11, gap extension penalty 1) between the DNA query (e.g. a D. gyrociliatus SPAdes K-mer-55 assembly) and DIAMOND-formatted amino acid reference sequences (e.g. 957 H. robusta near-USCOs from GenBank GCA_000326865.1). During hit stitching, all alignments between the query region and a target are merged so that only the highest-scoring segment pair (HSP) per region is retained, and adjacent or overlapping regions are merged into a single unit. For the divergence benchmark, pass the options --ultra-sensitive --frameshift 15 --masking 0: ultra-sensitivity plus 15-frame-shift compensation recovers remote homologs across the D. gyrociliatus/H. robusta divergence, and disabling tantan masking (--masking 0) improves query coverage, especially for read data. A sliding-window trimming step then removes noncoding stretches, and remaining ordered, nonoverlapping regions are concatenated into one continuous sequence per reference. Restrict downstream evaluation to 'recoverable' markers (e.g. the 778 D. gyrociliatus USCOs that produce a hit under DIAMOND --ultra-sensitive) and score them with pidentical,cov = nmatch/naligned x cov.

## Related tools

- **Patchwork** (Implements hit stitching (merging adjacent/overlapping DIAMOND HSPs), alignment masking, sliding-window trimming, and concatenation of retrieved markers) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Performs the BLASTX sequence similarity search (BLOSUM62, gap open 11, extend 1) whose hits Patchwork stitches; benchmark flags --ultra-sensitive --frameshift 15 --masking 0) — https://github.com/bbuchfink/diamond
- **Julia** (Programming language in which Patchwork is written and executed) — https://github.com/fethalen/Patchwork

## Examples

```
patchwork -q d_gyrociliatus_spades_k55.fasta -r h_robusta_nearusco.dmnd --ultra-sensitive --frameshift 15 --masking 0
```

## Evaluation signals

- Retrieval count matches expectation: e.g. 943 of 957 H. robusta reference sequences retrieved when querying with the D. gyrociliatus assembly
- Uniquely aligned markers match: 769 of the recovered markers align to exactly 1 of the 778 recoverable D. gyrociliatus USCOs
- Mean percent identical aligned positions is approximately 89% and mean query coverage approximately 74.5% for the divergence benchmark
- Combined metric pidentical,cov ≈ 67.2% identical matches over all reference positions
- Each reference produces exactly one continuous stitched sequence, with only the highest-scoring segment pair retained per region

## Limitations

- For highly divergent query–reference pairs only 943/957 references were retrieved and 769/778 align back uniquely, so distant homologs may be missed or ambiguously stitched
- Stitching merges only adjacent or overlapping regions; gappy or nonoverlapping hits are not unified by this step
- Disabling tantan masking (--masking 0) can raise coverage but is a benchmark-specific choice; unmasked low-complexity regions may yield spurious alignments
- Downstream artifacts (noncoding stretches) should be further screened with tools such as PREQUAL, and cutoff thresholds for trimming become harder to choose as identity drops far below 100%

## Evidence

- [methods] all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained: "all alignments made between the query region and the target sequence are merged in a way such that only the highest-scoring segment pair (HSP) for each region is retained"
- [abstract] followed by a "hit stitching" phase, in which adjacent or overlapping regions are merged into a single unit: "followed by a "hit stitching" phase, in which adjacent or overlapping regions are merged into a single unit"
- [results] query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument): "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [methods] the resulting set of ordered, nonoverlapping sequence regions are concatenated into 1 continuous sequence: "the resulting set of ordered, nonoverlapping sequence regions are concatenated into 1 continuous sequence"
- [readme] Stitch overlapping or gappy sequences together based on a reference: "Stitch overlapping or gappy sequences together based on a reference"
